# Claude Code サンドボックス方式の比較

Claude Code を隔離して実行する主な 4 方式（ネイティブ sandbox / sandbox-runtime / Dev Container / Claude Code on the web）の仕組み・対応 OS・隔離範囲・セットアップコスト・推奨用途を比較する。

本ドキュメントは 2026-08 時点の公式ドキュメント（[参考リンク](#参考リンク)）に基づく。設定スキーマは変わりうるため、実際に設定する際は最新の公式ドキュメントを確認すること。

## 比較表

| 項目 | ネイティブ sandbox<br>(sandboxed Bash tool) | sandbox-runtime<br>(`@anthropic-ai/sandbox-runtime`) | Dev Container | Claude Code on the web |
| :--- | :--- | :--- | :--- | :--- |
| 仕組み | OS のサンドボックス機構で Bash コマンドを隔離。macOS は Seatbelt、Linux / WSL2 は bubblewrap + socat（プロキシ経由でネットワーク制御） | ネイティブ sandbox と同じ Seatbelt / bubblewrap で Claude Code の**プロセス全体**をラップ | Docker コンテナ内で Claude Code を実行。参照実装は iptables / ipset によるデフォルト拒否のファイアウォール（`init-firewall.sh`）付き | Anthropic 管理の隔離 VM 上でセッションを実行。ネットワークプロキシが許可リストを強制し、GitHub トークンは VM 外のプロキシが保持 |
| 隔離範囲 | Bash コマンドとその子プロセスのみ（Read / Edit などの組み込みツール、MCP サーバー、フックはホスト上で動く） | Claude Code プロセス全体（ファイルツール、MCP サーバー、フックを含む） | 開発環境全体（コンテナ内で動くものすべて） | OS 全体（VM 単位） |
| 対応 OS | macOS / Linux / WSL2。**Windows ネイティブは非対応**（WSL1 も非対応） | macOS / Linux（WSL2 含む）。Windows は alpha サポート（専用ユーザー + WFP） | Docker が動くホスト（Windows ネイティブでも利用可） | ブラウザまたは CLI（`--cloud`）から利用。ローカル環境不問 |
| Docker 要否 | 不要 | 不要 | 必要 | 不要 |
| セットアップコスト | macOS はゼロ。Linux / WSL2 は低（`bubblewrap` と `socat` のインストール） | 低（npm パッケージ + `~/.srt-settings.json`） | 中（Docker、devcontainer 対応エディタ、設定ファイル） | なし（Claude サブスクリプションが必要。Web から起動する場合は GitHub 連携も必要） |
| 推奨用途 | 日常の開発で許可プロンプトを減らす | Docker なしで MCP サーバーやフックまで隔離したい。`--dangerously-skip-permissions` での無人実行 | チームで環境を標準化しつつ無人実行もしたい | 信頼できないリポジトリの検証、ローカル環境のないデバイスからのタスク委任 |

前者 2 方式はホスト OS 上でコンテナなしに動き、後者 2 方式はコンテナ / VM の中に Claude Code 全体を入れる。公式ドキュメントではこのほかにカスタムコンテナと専用 VM も比較されている（[Sandbox environments](https://code.claude.com/docs/en/sandbox-environments) 参照）。

## 各方式の詳細

### 1. ネイティブ sandbox（sandboxed Bash tool）

Claude Code に組み込みのサンドボックスで、Bash コマンドとその子プロセスのファイルシステム / ネットワークアクセスを OS レベルで制限する。境界内で完結するコマンドは許可プロンプトなしで実行できる（auto-allow モード）。

- **仕組み**
  - macOS: 標準の Seatbelt フレームワーク（追加インストール不要）
  - Linux / WSL2: [bubblewrap](https://github.com/containers/bubblewrap)（ファイルシステム隔離）+ [socat](http://www.dest-unreach.org/socat/)（サンドボックス外のプロキシへネットワークを中継）
  - ネットワークはサンドボックス外のプロキシサーバー経由で制御され、ドメイン許可リストで絞る
- **対応 OS**: macOS / Linux / WSL2。**Windows ネイティブは非対応**。Windows では WSL2 ディストリビューション内で Claude Code を実行する必要がある。WSL1 は bubblewrap が必要とするカーネル機能がないため非対応。
- **デフォルトの境界**: 書き込みは作業ディレクトリとセッション一時ディレクトリのみ。読み取りは一部の保護パスを除きホスト全体（`~/.ssh` などは既定では読めるため、`sandbox.credentials` や `denyRead` で明示的に保護する）。
- **注意**: 隔離されるのは Bash コマンドのみ。MCP サーバーやフックはホスト上で無制限に動くため、無人実行の境界としては単体では不十分。

セッション内の `/sandbox` コマンドで有効化・設定確認ができる。設定は `settings.json` の `sandbox` キーで行う（[設定例](#settingsjson-の-sandbox-キー設定例)参照）。

### 2. sandbox-runtime（`@anthropic-ai/sandbox-runtime`）

ネイティブ sandbox と同じ OS プリミティブ（macOS は `sandbox-exec` / Seatbelt、Linux は bubblewrap）を使って、任意のプロセス全体をラップするスタンドアロンパッケージ。`npx @anthropic-ai/sandbox-runtime claude` として起動すると、Bash だけでなくファイルツール・MCP サーバー・フックまで含むセッション全体が境界の内側に入る。

- **状態**: beta の research preview。設定フォーマットは変わりうる。
- **対応 OS**: macOS / Linux（WSL2 含む）。Windows は alpha サポート（専用の `srt-sandbox` ローカルユーザーと Windows Filtering Platform によるネットワーク制御）。
- **セットアップ**: Linux / WSL2 では `bubblewrap`、`socat`、`ripgrep` が必要。設定は `~/.srt-settings.json`（または `--settings` で指定するファイル）に書く。既定ではネットワーク拒否・書き込みはごく一部のパスのみなので、最低限プロジェクトディレクトリ、`~/.claude` と `~/.claude.json`、`/tmp` への書き込みと、`api.anthropic.com` などの必要ドメインを許可する。
- **自動保護**: `.git/hooks`、`.mcp.json`、`.claude/commands` など、次回起動時に非サンドボックスで実行される設定を書き換えられるパスへの書き込みは、設定なしでも拒否される。
- **推奨用途**: Docker を使わずに `--dangerously-skip-permissions` での無人実行の境界を作りたい場合。

```bash
npx @anthropic-ai/sandbox-runtime claude
```

### 3. Dev Container

VS Code などの Dev Containers 対応エディタが管理する Docker コンテナ内で Claude Code を実行する。プロジェクトはコンテナにマウントされ、編集はローカルリポジトリに反映される。

- **仕組み**: リポジトリの `.devcontainer/` に環境を定義する。anthropics/claude-code リポジトリが参照実装を公開しており、次の 3 ファイルで構成される。
  - [`devcontainer.json`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/devcontainer.json): ボリュームマウント、`runArgs`（ファイアウォール用の `NET_ADMIN` / `NET_RAW` ケーパビリティ）、拡張機能、環境変数
  - [`Dockerfile`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/Dockerfile): ベースイメージ、開発ツール、Claude Code のインストール
  - [`init-firewall.sh`](https://github.com/anthropics/claude-code/blob/main/.devcontainer/init-firewall.sh): iptables で許可ドメイン以外の外向き通信をすべて遮断（デフォルト拒否）
- **対応 OS**: Docker が動けばよいので、Windows ネイティブホストでも使える（ネイティブ sandbox が使えない Windows での代替手段の一つ）。
- **推奨用途**: チームで隔離環境を標準化する。デフォルト拒否のファイアウォール構成なら、非 root ユーザーで `--dangerously-skip-permissions` を使った無人実行にも対応する。
- **注意**: `--dangerously-skip-permissions` 実行時、コンテナ内からアクセスできるもの（`~/.claude` の認証情報を含む）の持ち出しは防げない。信頼できるリポジトリでのみ使い、`~/.ssh` などのホストの秘密情報はマウントしない。

### 4. Claude Code on the web

各セッションを Anthropic 管理の隔離 VM で実行するホスト型の方式。

- **仕組み**: ネットワークプロキシが既定の許可リストを強制する。GitHub トークンはサンドボックス外の専用プロキシが保持し、VM 内にはスコープを絞った一時的な認証情報だけを渡す。
- **要件**: Claude サブスクリプション。Web インターフェースから起動する場合は GitHub アカウント連携が必要。CLI から `claude --cloud` で起動する場合はローカルリポジトリのアップロードも可能。
- **推奨用途**: インフラを自分で用意せずに VM 隔離が欲しい場合、信頼できないリポジトリの検証、開発環境のないデバイスからのタスク委任。

## パーミッションモードとの関係

パーミッションモードは「ツール呼び出しを実行するか・事前に確認するか」を決め、サンドボックスは「実行されたコマンドが何にアクセスできるか」を制限する。両者は補完関係にある。

- ネイティブ sandbox の auto-allow モードは、境界内で完結するコマンドをプロンプトなしで実行する。deny ルールや `Bash(git push *)` のような ask ルールは引き続き適用される。
- `--dangerously-skip-permissions` での無人実行は、必ずコンテナ・VM・sandbox-runtime のいずれかの中で行う（ネイティブ sandbox 単体では Bash 以外が無防備なため不十分）。Linux / macOS では root 実行時にこのフラグは拒否される。

詳細は [Permission modes](https://code.claude.com/docs/en/permission-modes) を参照。

## settings.json の `sandbox` キー設定例

ネイティブ sandbox は `settings.json`（プロジェクトの `.claude/settings.json`、ユーザーの `~/.claude/settings.json`、または管理者配布の managed settings）の `sandbox` キーで設定する。全キーは [Settings reference](https://code.claude.com/docs/en/settings-reference#sandbox-settings) を参照。

基本形。サンドボックスを有効化し、作業ディレクトリ外への書き込み先とネットワークの許可ドメインを絞る:

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"]
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

- `enabled`: サンドボックスの有効化。`~/.claude/settings.json` に置けば全プロジェクトに適用される
- `filesystem.allowWrite`: 既定（作業ディレクトリ + セッション一時ディレクトリ）の外で書き込みを許可するパス。`~/` はホーム相対、`/` は絶対パス
- `network.allowedDomains`: 事前に許可するドメイン。`*.example.com` 形式のワイルドカードが使える。未指定のドメインへの初回アクセス時はプロンプトが出る

認証情報の保護。既定では `~/.aws/credentials` なども読めてしまうため、`sandbox.credentials` で明示的に遮断する:

```json
{
  "sandbox": {
    "enabled": true,
    "credentials": {
      "files": [
        { "path": "~/.aws/credentials", "mode": "deny" },
        { "path": "~/.ssh", "mode": "deny" }
      ],
      "envVars": [
        { "name": "GITHUB_TOKEN", "mode": "deny" },
        { "name": "NPM_TOKEN", "mode": "deny" }
      ]
    }
  }
}
```

組織で強制する場合の managed settings 例。サンドボックスが初期化できなければ起動を失敗させ、サンドボックス外での再実行（`dangerouslyDisableSandbox`）も禁止する:

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false
  }
}
```

サンドボックスと互換性のないツール（例: `docker`）は `excludedCommands` でサンドボックス外実行に切り出せる。

なお、ネイティブ sandbox は Windows ネイティブでは動かないため、Windows ホストにこの設定を配る場合は WSL2 内での実行を前提にするか、コンテナ / VM 方式を使う。

## 参考リンク

- [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) — ネイティブ sandbox の仕組みと全設定
- [Choose a sandbox environment](https://code.claude.com/docs/en/sandbox-environments) — 各隔離方式の公式比較
- [Development containers](https://code.claude.com/docs/en/devcontainer) — Dev Container の構成と参照実装
- [Choose a permission mode](https://code.claude.com/docs/en/permission-modes) — パーミッションモードと隔離の関係
- [anthropic-experimental/sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) — sandbox-runtime のリポジトリと設定スキーマ
- [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) — ホスト型サンドボックスの詳細
- [Settings reference: Sandbox settings](https://code.claude.com/docs/en/settings-reference#sandbox-settings) — `sandbox` キーの全設定項目
