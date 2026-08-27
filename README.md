# claude-code-sandbox-lab

Claude Code の堅牢なサンドボックス構成を試す実験場。隔離方式ごとの設定例と検証手順・検証結果をまとめ、Windows (WSL2 / Docker) 環境で実際に動かして確かめることを目的とする。

## 構成

| パス | 内容 |
| :--- | :--- |
| [`.devcontainer/`](./.devcontainer/) | Dev Container 構成。anthropics/claude-code の公式リファレンス実装（`devcontainer.json` / `Dockerfile` / `init-firewall.sh`）準拠 |
| [`srt/`](./srt/) | [`@anthropic-ai/sandbox-runtime`](https://github.com/anthropic-experimental/sandbox-runtime) (srt) の設定例と WSL2 での検証手順 |
| [`docs/sandbox-comparison.md`](./docs/sandbox-comparison.md) | 公式 6 方式（ネイティブ sandbox / sandbox-runtime / Dev Container / カスタムコンテナ / 専用 VM / Claude Code on the web）の比較ドキュメント |
| [`docs/risks-and-mitigations.md`](./docs/risks-and-mitigations.md) | Claude Code 利用時のリスク一覧と対策、リスク × サンドボックス 6 方式の対策可否マトリクス |

## 各構成の概要と検証結果

### Dev Container（[`.devcontainer/`](./.devcontainer/)）

Docker コンテナ内で Claude Code を実行し、`init-firewall.sh` の iptables + ipset で許可ドメイン以外の外向き通信をデフォルト拒否する構成。ネイティブサンドボックスが使えない Windows ネイティブホストでも利用できる。リファレンス実装からの調整は、DNS の重複 A レコードで起動が中断する実測の問題への `ipset add -exist` 対応、LF を保証する `.gitattributes` の追加など最小限。

**検証結果（検証済み）:**

- `npx @devcontainers/cli up` でビルド・起動が成功（`NET_ADMIN` / `NET_RAW` 付き、`postStartCommand` でファイアウォール適用、スクリプト内の自己検証も通過）
- コンテナ内から許可ドメイン（`api.anthropic.com`）への HTTPS 接続が成功
- 許可外ドメイン（`example.com`）への接続が遮断されることを確認

### sandbox-runtime（[`srt/`](./srt/)）

srt は bubblewrap + プロキシ経由のネットワークフィルタリングで、Claude Code の実行プロセス全体（MCP サーバーや hooks を含む）を丸ごと隔離するツール。設定例 [`srt/srt-settings.json`](./srt/srt-settings.json) は、ネットワークを許可ドメインのみ（allow-only）、書き込みをリポジトリ・`/tmp`・`~/.claude` のみに制限する。手順の詳細は [`srt/README.md`](./srt/README.md) を参照。

**検証結果（部分的）:**

- WSL2 (Ubuntu-24.04) で `npx` によるパッケージ取得と CLI 起動は成功。非特権 user namespace が利用可能なこと（Ubuntu 24.04 で通常必要な AppArmor sysctl の無効化が WSL2 では不要なこと）も確認済み
- ただし検証環境に依存パッケージ（`bubblewrap` / `socat` / `ripgrep`）が未導入かつ sudo が非対話で使えなかったため、サンドボックス本体の起動（遮断・拒否の実測）には未到達
- 残りの手順（依存導入と検証コマンドの再実行）は [`srt/README.md`](./srt/README.md) の「検証結果」節に記録している

### 方式比較（[`docs/sandbox-comparison.md`](./docs/sandbox-comparison.md)）

ネイティブ sandbox（sandboxed Bash tool）、sandbox-runtime、Dev Container、カスタムコンテナ、専用 VM、Claude Code on the web の公式 6 方式について、仕組み・隔離範囲・対応 OS・セットアップコスト・推奨用途を比較する。パーミッションモードとの関係や、ネイティブ sandbox の `settings.json` 設定例も含む。どの方式を選ぶかの出発点として参照する。

### リスクと対策（[`docs/risks-and-mitigations.md`](./docs/risks-and-mitigations.md)）

Claude Code 利用時に発生しうるリスク（プロンプトインジェクション、認証情報の流出、設定改ざんによる永続化、無人実行の暴走など 11 項目）を整理し、それぞれに必要な対策をまとめる。あわせて、サンドボックス 6 方式ごとにどのリスクを対策できるかをマトリクスで可視化する。サンドボックスでは守れない領域（プロジェクトコードの改変、API へのデータ送信）も明示している。
