# sandbox-runtime (srt) を WSL2 で試す

[`@anthropic-ai/sandbox-runtime`](https://github.com/anthropic-experimental/sandbox-runtime) (srt) は、任意のプロセスに OS レベルのファイルシステム制限とネットワーク制限を課すサンドボックスツール。Claude Code の実行プロセス全体（MCP サーバーや hooks を含む）を丸ごと包んで隔離できる。

Linux では bubblewrap によるコンテナ化とプロキシ経由のネットワークフィルタリングで実現されるため、本リポジトリでは WSL2 (Ubuntu-24.04) 上で動かす。なお srt には Windows ネイティブ対応（alpha、専用ローカルアカウント + WFP 方式）も追加されているが、本ディレクトリのスコープは WSL2 での検証とする。

## 前提条件

- Windows 11 + WSL2 (Ubuntu-24.04)
- Node.js（nvm 等で導入。検証時は v25.7.0 / npx 11.11.0）
- 以下の Linux 依存パッケージ（**要 sudo**）
  - `bubblewrap` — コンテナ化ランタイム
  - `socat` — プロキシブリッジ用ソケットリレー
  - `ripgrep` — mandatory deny パス検出

## セットアップ

WSL2 のシェルで実行する（Windows 側からは `wsl -d Ubuntu-24.04 -- bash -lc '<cmd>'`）。

```bash
# 1. 依存パッケージの導入（sudo が必要）
sudo apt-get update
sudo apt-get install -y bubblewrap socat ripgrep

# 2. Ubuntu 24.04 の userns 制限について
#    通常の Ubuntu 24.04 では kernel.apparmor_restrict_unprivileged_userns=1 が
#    既定で有効なため、次の無効化が必要になる:
#      sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
#    ただし WSL2 カーネル (6.18.33.2-microsoft-standard-WSL2) にはこの sysctl 自体が
#    存在せず、非特権 user namespace は最初から利用できるため、この手順は不要（検証済み）。
```

srt のインストールは不要。`npx` で直接実行できる。

## 設定ファイル

設定例は [`srt-settings.json`](./srt-settings.json)。srt は既定で `~/.srt-settings.json` を読むが、本リポジトリでは `--settings` フラグでこのファイルを明示指定して使う（`~/.srt-settings.json` にコピーしても同じ）。

スキーマは srt リポジトリの README を正とする。要点:

- **network（allow-only）**: 既定で全ネットワーク遮断。`allowedDomains` に列挙したドメインだけ許可される（`*.github.com` のようなワイルドカード可）。設定例では Anthropic API・GitHub・npm registry を許可している。
- **filesystem read（deny-then-allow）**: 既定で全読み取り許可。`denyRead` で広く拒否し、`allowRead` で再許可する（read は `allowRead` が優先）。設定例では `/home` 全体を拒否した上で、リポジトリ（`.`）・`~/.claude`・`~/.claude.json`・Node.js ツールチェーン（`~/.nvm`, `~/.npm`）だけを再許可している。
- **filesystem write（allow-only）**: 既定で全書き込み拒否。`allowWrite` に列挙したパスだけ許可され、`denyWrite` が優先で除外される。設定例ではリポジトリ・`/tmp`・`~/.claude`・`~/.claude.json` のみ許可し、`.git/hooks`・`.git/config`・`.env` を明示的に拒否している。
  - `.git/hooks/`・`.git/config`・`.bashrc` などは srt が **mandatory deny path** として常に書き込み拒否するため、`denyWrite` への記載は明示のための多重防御。
- 相対パス（`.` など）は **srt 実行時のカレントディレクトリ基準**。必ずリポジトリルートから実行すること。
- Linux 実装はパスの glob パターン非対応（リテラルパスのみ）。

## 実行

リポジトリルートで:

```bash
# 動作確認
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json 'echo hello'

# Claude Code 全体をサンドボックス内で起動する例
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json claude
```

## 検証手順

セットアップ完了後、以下で Acceptance criteria を確認する。

```bash
# (1) 許可ドメイン → 応答が返る
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json \
  'curl -sS --max-time 20 https://api.github.com/zen'

# (2) 許可外ドメイン → 接続が遮断される
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json \
  'curl -sS --max-time 20 https://example.com'

# (3) 許可外パスへの書き込み → 拒否される (Operation not permitted)
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json \
  'touch ~/srt-deny-test.txt'

# (4) 許可パスへの書き込み → 成功する
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json \
  'touch /tmp/srt-allow-test.txt && echo write-ok'

# (5) mandatory deny → 拒否される
npx -y @anthropic-ai/sandbox-runtime --settings srt/srt-settings.json \
  'echo x >> .git/config'
```

## 検証結果

2026-08-27、WSL2 (Ubuntu-24.04, カーネル 6.18.33.2-microsoft-standard-WSL2) で実施。

**実行できた範囲:**

- Node.js v25.7.0 / npx 11.11.0 (nvm) を確認。`npx -y @anthropic-ai/sandbox-runtime` でのパッケージ取得と CLI 起動は成功。
- 非特権 user namespace の利用可否を確認: `unshare --user --map-root-user true` は成功。`kernel.apparmor_restrict_unprivileged_userns` sysctl は WSL2 カーネルに存在しないため、Ubuntu 24.04 で通常必要な無効化手順は不要。
- 上記「検証手順」(1)〜(5) 相当のコマンドを `--settings srt/srt-settings.json` 付きで実行したところ、srt は 5 件ともハングせず即座に次のエラーで終了した（exit code 1）:

  ```
  Error: Sandbox dependencies not available: ripgrep (rg) not found, bubblewrap (bwrap) not installed, socat not installed
  ```

**実行できなかった理由:**

- この環境では `bubblewrap` / `socat` / `ripgrep` が未導入で、`sudo -n` がパスワードを要求して失敗するため（非対話環境）、`sudo apt-get install` による依存導入ができなかった。サンドボックス本体の起動（許可外ドメインの遮断・許可外パスへの書き込み拒否の実測）まで到達していない。

**残りの手順:**

1. WSL2 の対話シェルで `sudo apt-get update && sudo apt-get install -y bubblewrap socat ripgrep` を実行する。
2. 上記「検証手順」(1)〜(5) を再実行し、期待どおり (1)(4) が成功、(2)(3)(5) が拒否されることを確認する。
3. 結果をこの節に追記する。
