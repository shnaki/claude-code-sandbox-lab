# AI 利用の脅威カタログとマトリクス設計案

Claude 利用時の脅威を体系的にマトリクス化するための土台として、脅威カタログとして利用できる業界標準資料をカテゴリ別に整理し、複数カタログから脅威リストを統合する方針と、脅威 × 3 ユースケースのマトリクス設計案をまとめる。既存の[リスクと対策](./risks-and-mitigations.md)が「サンドボックスで防げるか」の観点でリスク 11 項目を扱うのに対し、本ドキュメントはその上流にあたる「そもそも何が脅威か」の出典を業界標準フレームワークに求める。

本ドキュメントは 2026-08 時点の各資料（[参考リンク](#参考リンク)）に基づく。各カタログは改訂が続いているため、実際にマトリクスを作成・更新する際は最新版を確認すること。

## 対象ユースケース

マトリクスの列となる 3 ユースケース。後段の資料は、このいずれか（または複数）の脅威の出典として使う。

| ユースケース | 内容 |
| :--- | :--- |
| チャットのみ | Claude（Web / デスクトップアプリ）との対話のみ。ローカルでのコード実行・ツール実行なし |
| Claude Code（顧客資産なし） | 自社・個人のリポジトリを対象に Claude Code を利用する。壊れて困るのは自分たちの資産のみ |
| Claude Code（顧客資産あり） | 顧客から預かったコード・データを含むリポジトリを対象に Claude Code を利用する。セキュリティ脅威に加えて契約・守秘義務の観点が加わる |

## 資料一覧

### 網羅系カタログ

攻撃・脅威を体系的に分類した網羅志向のカタログ。統合脅威リストの母集団と、漏れがないかのチェックに使う。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [MITRE ATLAS](https://atlas.mitre.org/) v5.1.0（2025-11） | ATT&CK 流の形式で AI への攻撃を整理。16 tactics / 84 techniques / 32 mitigations / 42 case studies。2026-02 にエージェント技法が追加された | 攻撃手法の正規 ID（ATLAS ID）の出典。ケーススタディで脅威の現実性を裏付ける |
| [OWASP AI Exchange](https://owaspai.org/) | 脅威を開発時 / 入力（利用時）/ ランタイムの 3 段階に分類。対策は Manage / Resilient models / Watch / Limit の 4 系統 50+ 項目。ISO/IEC 27090 に寄与。[日本語訳](https://coky-t.gitbook.io/owasp-ai-security-and-privacy-guide-ja/)あり | 脅威 → 対策の対応付けの裏付け。網羅性チェックのクロスウォーク先 |
| [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final)（2025-03） | Adversarial Machine Learning の攻撃・対策の分類と用語集。GenAI・RAG・エージェントへの攻撃を含む | 攻撃分類の学術的裏付けと用語の正規化 |
| [BIML: An Architectural Risk Analysis of LLM](https://berryvilleiml.com/docs/BIML-LLM24.pdf) | LLM のアーキテクチャリスク 81 項目（うちブラックボックス基盤モデルリスク 23 項目）。[インタラクティブ版](https://berryvilleiml.com/interactive/)あり | 利用者から見えない基盤モデル側リスクの整理。提供者責任範囲への縮約（統合方針の第 2 層）の判断材料 |
| [Microsoft: Taxonomy of Failure Modes in Agentic AI Systems v2.0](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/bade/documents/products-and-services/en-us/security/Taxonomy-of-Failure-Modes-in-Agentic-AI-Systems-v2-0.pdf)（2026-06） | エージェント固有の failure mode 分類。v2.0 で新 failure mode 7 種（Agentic Supply Chain Compromise / Goal Hijacking / Inter-Agent Trust Escalation / CUA Visual Attack / Session Context Contamination / MCP・Plugin Abuse / Capability Disclosure）を追加。mitigation は 5 families | エージェント（Claude Code）固有脅威の補完源。新 failure mode 7 種を網羅性チェックに使う |
| [ENISA FAICP](https://www.enisa.europa.eu/publications/a-multilayer-framework-for-good-cybersecurity-practices-for-ai) | ICT 基盤 / ML ライフサイクル / セクター別の 3 層でセキュリティプラクティスを整理 | 組織導入時にどのレイヤの対策かを整理する際の参考 |

### Top 10 系

優先度付きの上位リスト。項目数が絞られており ID 体系も安定しているため、統合脅威リストの主軸に据える。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [OWASP GenAI LLM Top 10 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) | LLM アプリケーションの上位リスク 10 項目（LLM01〜LLM10）。NIST・ATLAS・CWE へのマッピング付き | 統合脅威リストの主軸その 1。LLM## を正規 ID として使う |
| [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | エージェントアプリケーションの上位リスク 10 項目。ASI01 Agent Goal Hijack / ASI02 Tool Misuse & Exploitation / ASI03 Identity & Privilege Abuse / ASI04 Agentic Supply Chain Vulnerabilities / ASI05 Unexpected Code Execution / ASI06 Memory & Context Poisoning / ASI07 Insecure Inter-Agent Communication / ASI08 Cascading Failures / ASI09 Human-Agent Trust Exploitation / ASI10 Rogue Agents | 統合脅威リストの主軸その 2。Claude Code 系ユースケースの中心。ASI## を正規 ID として使う |
| [OWASP Agentic AI – Threats and Mitigations v1.0](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) | エージェント脅威 T1〜T15 と対策。本文は PDF 配布のみのため資料ページを参照 | ASI01〜10 より粒度の細かい T## を出典 ID の補助として使う |
| [Agentic Security Initiative](https://genai.owasp.org/initiatives/agentic-security-initiative/) | OWASP GenAI 配下のエージェントセキュリティ 5 文書スイート | 上記 2 資料の親イニシアチブ。続編・改訂の追跡窓口 |

### リスク台帳・メタ分析

個別カタログを横断的に集約した台帳。統合脅威リストそのものではなく、網羅性チェックと現実性の裏付けに使う。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [MIT AI Risk Repository](https://airisk.mit.edu/) | 74 フレームワーク・1,700+ リスクを統合した台帳。7 ドメイン / 24 サブドメイン + Causal Taxonomy（Entity / Intent / Timing）。7 ドメインは 1 Discrimination & Toxicity / 2 Privacy & Security / 3 Misinformation / 4 Malicious Actors / 5 Human-Computer Interaction / 6 Socioeconomic & Environmental / 7 AI System Safety, Failures, & Limitations | 24 サブドメインを網羅性チェックのチェックリストに使う。非セキュリティ利用リスク（誤情報・過度依存・IP）の出典 |
| [AI Incident Database](https://incidentdatabase.ai/) | 実際に起きた AI インシデント 759+ 件を CSET 分類で整理 | 脅威の現実性・発生頻度の裏付け。マトリクスの該当度判定の参考 |
| [AVID – AI Vulnerability Database](https://avidml.org/) | AI の脆弱性を CVE 風に採番・蓄積するデータベース | 個別脆弱性レベルの事例参照 |

### 対策・統制側

脅威側ではなく対策・統制のカタログ。マトリクスの「主な対策」列の裏付けに使う。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [CSA AI Controls Matrix](https://cloudsecurityalliance.org/research/working-groups/ai-controls) | 243 統制 / 18 ドメイン。ISO/IEC 42001・NIST AI 600-1 等へのマッピング付き | 対策列の統制 ID 参照。組織的統制（ガバナンス・契約カテゴリ）の裏付け |
| [Google SAIF + SAIF Map](https://saif.google/secure-ai-framework/saif-map) | AI リスクと統制の対応マップ | 脅威 → 統制の対応付けのセカンドオピニオン |
| [NIST AI 600-1 Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)（2024-07） | 生成 AI 固有の 12 リスクカテゴリ（CBRN / Confabulation / Dangerous Content / Data Privacy / Environmental / Harmful Bias / Human-AI Configuration / Information Integrity / Information Security / IP / Obscene Content / Value Chain）と対策 | 非セキュリティ利用リスクの追加元（統合方針の第 2 層）。Confabulation・IP・Value Chain が特に該当 |
| [NCSC/CISA Guidelines for Secure AI System Development](https://www.ncsc.gov.uk/collection/guidelines-secure-ai-system-development) | 設計・開発・展開・運用の 4 フェーズ別のセキュア開発ガイドライン | ライフサイクル観点の対策の裏付け |
| [ISO/IEC 42001](https://www.iso.org/standard/81230.html)・[ISO/IEC 23894](https://www.iso.org/standard/77304.html)・[ISO/IEC 27090](https://www.iso.org/standard/56581.html) | それぞれ AI マネジメントシステム（AIMS）、AI リスクマネジメント、AI セキュリティ（策定中）の国際規格 | 組織的統制の参照先。CSA AICM 経由でマッピングを利用する |

### 日本語資料

国内の公的ガイドライン・調査。組織のポリシー策定と、契約・守秘観点の補完に使う。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [総務省・経産省「AI 事業者ガイドライン 第 1.2 版」](https://www.meti.go.jp/shingikai/mono_info_service/ai_shakai_jisso/pdf/20260331_1.pdf)（2026-03） | 開発者 / 提供者 / 利用者の主体分類でリスクと対応を整理 | 「Claude を利用する側」のスコープ定義の裏付け。契約・守秘観点の補完元 |
| [IPA「AI 利用時のセキュリティ脅威・リスク調査報告書」](https://www.ipa.go.jp/digital/ai/security/index.html)（2024-07） | 国内向けの AI 利用時セキュリティ脅威・リスクの調査報告 | 国内文脈での脅威の裏付け |
| [AISI「AI セーフティに関する評価観点ガイド 第 1.20 版」](https://aisi.go.jp/output/output_framework/guide_to_evaluation_perspective_on_ai_safety/)（2026-07） | AI セーフティの評価観点を整理。エージェント対応で改訂 | 出典 ID の一つ（AISI 評価観点）。評価・検証項目の設計参考 |
| [AISI「AI セーフティに関するレッドチーミング手法ガイド 第 1.10 版」](https://aisi.go.jp/output/output_framework/guide_to_red_teaming_methodology_on_ai_safety/)（2025-03） | レッドチーミングの計画・実施・報告の手法ガイド | 本ラボでの攻撃検証（レッドチーミング）の手順設計参考 |
| [デジタル庁「テキスト生成 AI 利活用におけるリスクへの対策ガイドブック（α版）」](https://www.digital.go.jp/assets/contents/node/basic_page/field_ref_resources/c1959599-efad-472e-a640-97ae67617219/fe843dc6/20240610_resources_generalitve-ai-guidebook_01.pdf)（2024-06） | ユースケース別のリスク × 軽減策という本ラボと同型の構成 | マトリクス構成（ユースケース別 × 対策）の先行例 |
| [デジタル庁 DS-920「行政の進化と革新のための生成 AI の調達・利活用に係るガイドライン」](https://www.digital.go.jp/assets/contents/node/basic_page/field_ref_resources/e2a06143-ed29-4f1d-9c31-0f06fca67afc/80419aea/20250527_resources_standard_guidelines_guideline_01.pdf)（2025-05） | 行政機関向けの生成 AI 調達・利活用ガイドライン | 組織導入時の統制・調達要件の参考 |
| [JDLA「生成 AI の利用ガイドライン 第 1.1 版」](https://www.jdla.org/document/) | 組織内の生成 AI 利用規程のひな形。入力データ（機密・個人情報）と生成物（著作権・虚偽）の注意点を整理 | 「顧客資産あり」ユースケースの契約・守秘観点の補完元 |
| [IPA「AI 利用者のためのセキュリティ豆知識」](https://www.ipa.go.jp/security/ai/index.html)（2026） | 利用者向けの平易なセキュリティ注意点集 | 利用者教育・啓発の参考 |

### コーディングエージェント固有

Claude Code そのものと、コーディングエージェント特有の脅威に関する資料。

| 資料 | 特徴 | 本ラボでの使いどころ |
| :--- | :--- | :--- |
| [Anthropic 公式 Security](https://code.claude.com/docs/en/security)・[Sandboxing](https://code.claude.com/docs/en/sandboxing) docs | Claude Code のセキュリティモデルとサンドボックスの公式ドキュメント。既存の[リスクと対策](./risks-and-mitigations.md)・[サンドボックス方式の比較](./sandbox-comparison.md)で参照済み | マトリクスの「主な対策」列の一次出典 |
| [CSA research note: Slopsquatting](https://labs.cloudsecurityalliance.org/research/csa-research-note-slopsquatting-ai-supply-chain-20260419-csa/)（2026-04） | ハルシネーションで生成された実在しないパッケージ名を突く AI 由来のサプライチェーン攻撃 | サプライチェーンカテゴリへの補完脅威（統合方針の第 3 層） |

## 脅威リストの統合方針（4 層アプローチ）

上記の資料群から、マトリクスの行となる統合脅威リストを次の 4 層で作る。

### 第 1 層: OWASP LLM Top 10 + Agentic ASI を主軸に統合

OWASP GenAI LLM Top 10 2026（LLM01〜LLM10）と OWASP Top 10 for Agentic Applications 2026（ASI01〜ASI10）・Agentic AI Threats and Mitigations（T1〜T15）を突き合わせ、重複を正規化して約 20 行の統合脅威リストに整理する。いずれも NIST・ATLAS へのマッピングを持つため、他カタログの正規 ID との連結にもこの主軸を使う。

### 第 2 層: 「Claude を利用する側」観点のスコープフィルタ

本ラボの主体は Claude を利用する開発者・組織であり、モデルを開発・提供する側ではない。そこで開発時脅威（学習データポイズニング、モデル抽出等）は「提供者（Anthropic）側の責任範囲」として 1 行に縮約する。かわりに NIST AI 600-1 の 12 リスクカテゴリと MIT AI Risk Repository のドメイン 3（誤情報）/ サブドメイン 5.1（過度依存）/ ドメイン 6（IP・契約）から、非セキュリティの利用リスク（ハルシネーション起因の業務ミス、機密情報の入力、著作権）を行として追加する。この層が「チャットのみ」ユースケースの主戦場になる。

### 第 3 層: クロスウォークによる網羅性チェック

MIT AI Risk Repository の 24 サブドメイン、OWASP AI Exchange の脅威一覧、Microsoft Taxonomy v2.0 の新 failure mode 7 種と統合脅威リストをクロスウォークし、漏れを補完する。補完が確定している脅威は次のとおり: MCP / プラグイン悪用、CUA 視覚攻撃、セッション文脈汚染、slopsquatting。また「Claude Code（顧客資産あり）」向けには、JDLA ガイドラインと AI 事業者ガイドラインから契約・守秘観点の行を補完する。

### 第 4 層: 出典 ID マッピング

各脅威行に LLM## / ASI## / T## / ATLAS ID / AISI 評価観点等の出典 ID を併記し、どのカタログに由来する脅威かを追跡できるようにする。対策列は Anthropic 公式ドキュメントと既存の[リスクと対策](./risks-and-mitigations.md)（サンドボックス層）を一次出典とし、OWASP AI Exchange / CSA AI Controls Matrix の統制で裏付ける。

## マトリクス設計案

統合脅威リストが確定した後に作るマトリクスの設計。マトリクス本体（全セル埋め）の作成は本ドキュメントのスコープ外とし、別途行う。

- **行**: 統合脅威リスト約 20 行。次の 6 カテゴリでグルーピングし、各行に出典 ID を併記する
  - 入力系 / 実行・ツール系 / データ・機密系 / 出力・品質系 / サプライチェーン / ガバナンス・契約
- **列**: 3 ユースケース（チャットのみ / Claude Code 顧客資産なし / Claude Code 顧客資産あり）の該当度 + 主な対策 + 出典
- **セル凡例**: 既存の[リスクと対策](./risks-and-mitigations.md)の `◎○✕−` スタイルを踏襲しつつ、意味は本ドキュメントの文脈で再定義する
  - **◎** そのユースケースの主要な脅威
  - **○** 条件付きで該当（構成・運用・扱うデータ次第）
  - **−** そのユースケースには該当しない

形とセルの書きぶりのイメージ（行の内容は例であり確定ではない）:

| カテゴリ | 脅威（出典 ID） | チャットのみ | Claude Code<br>（顧客資産なし） | Claude Code<br>（顧客資産あり） | 主な対策 | 出典 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 入力系 | 間接プロンプトインジェクション（LLM01 / ASI01 / T6 / ATLAS ID） | ○ | ◎ | ◎ | 許可プロンプトでのレビュー、サンドボックス（[リスク 1](./risks-and-mitigations.md#1-プロンプトインジェクション) に委譲） | OWASP / ATLAS |
| 出力・品質系 | ハルシネーション起因の業務ミス（LLM09 / NIST AI 600-1 Confabulation / MIT ドメイン 3） | ◎ | ○ | ○ | 出力レビュー、根拠の一次確認 | NIST / MIT |
| ガバナンス・契約 | 顧客データの守秘義務・契約違反 | − | − | ◎ | 契約条件の確認、入力データの管理 | JDLA / AI 事業者ガイドライン |

## 既存リスク 1〜11 との対応（サンドボックス層への委譲）

Claude Code の実行に関わる脅威については、既存の[リスクと対策](./risks-and-mitigations.md)のリスク 1〜11 と[リスク × サンドボックス方式マトリクス](./risks-and-mitigations.md#リスク--サンドボックス方式マトリクス)が「サンドボックス 6 方式でどこまで防げるか」の詳細を持っている。本マトリクスの「主な対策」列ではこれを重複記述せず、該当リスク番号への参照で委譲する。カテゴリ単位の対応は次のとおり。

| 統合脅威リストのカテゴリ | 対応する既存リスク（[リスクと対策](./risks-and-mitigations.md#リスク一覧と対策)） |
| :--- | :--- |
| 入力系 | 1（プロンプトインジェクション） |
| 実行・ツール系 | 2（破壊的コマンドの実行）、6（MCP サーバー・フックの無制限実行）、8（許可プロンプト疲れ）、9（無人実行での暴走） |
| データ・機密系 | 3（認証情報・機密ファイルの読み取り）、4（ネットワーク経由のデータ流出）、11（モデル・API へのデータ送信） |
| 出力・品質系 | 10（プロジェクトコードの意図しない改変）の一部（バグ混入）。ハルシネーション起因の業務ミス等はサンドボックスの守備範囲外 |
| サプライチェーン | 5（設定改ざんによる永続化）、7（信頼できないコード・依存関係の実行）、10（プロジェクトコードの意図しない改変） |
| ガバナンス・契約 | 対応する既存リスクなし（サンドボックスの守備範囲外。契約・運用レイヤで対策する） |

「チャットのみ」ユースケースはコード実行を伴わないため、既存リスク 1〜11 の大半（実行系）が該当せず、第 2 層で追加した非セキュリティ利用リスクが中心になる。逆に Claude Code 系ユースケースでは、サンドボックスで防げる範囲は既存ドキュメントに委譲し、本マトリクスは「どの脅威がどのユースケースで効くか」と「サンドボックス以外の対策」に集中する。

## 参考リンク

- [MITRE ATLAS](https://atlas.mitre.org/) — AI への攻撃 tactics / techniques / mitigations / case studies のカタログ（v5.1.0）
- [OWASP AI Exchange](https://owaspai.org/) — 脅威を開発時 / 入力 / ランタイムの 3 段階で整理した網羅カタログ
- [OWASP AI Exchange 日本語訳](https://coky-t.gitbook.io/owasp-ai-security-and-privacy-guide-ja/) — coky-t 氏による GitBook 日本語訳
- [NIST AI 100-2 E2025](https://csrc.nist.gov/pubs/ai/100/2/e2025/final) — Adversarial Machine Learning の攻撃・対策分類と用語集
- [BIML: An Architectural Risk Analysis of LLM](https://berryvilleiml.com/docs/BIML-LLM24.pdf) — LLM のアーキテクチャリスク 81 項目の分析（PDF）
- [BIML インタラクティブ版](https://berryvilleiml.com/interactive/) — 上記 81 リスクのインタラクティブ閲覧版
- [Microsoft: Taxonomy of Failure Modes in Agentic AI Systems v2.0](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/bade/documents/products-and-services/en-us/security/Taxonomy-of-Failure-Modes-in-Agentic-AI-Systems-v2-0.pdf) — エージェント AI の failure mode 分類（PDF）
- [ENISA FAICP](https://www.enisa.europa.eu/publications/a-multilayer-framework-for-good-cybersecurity-practices-for-ai) — AI サイバーセキュリティプラクティスの多層フレームワーク
- [OWASP GenAI LLM Top 10 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) — LLM アプリケーションの上位リスク 10 項目
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) — エージェントアプリケーションの上位リスク 10 項目（ASI01〜ASI10）
- [OWASP Agentic AI – Threats and Mitigations v1.0](https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/) — エージェント脅威 T1〜T15 と対策（本文は PDF 配布）
- [Agentic Security Initiative](https://genai.owasp.org/initiatives/agentic-security-initiative/) — OWASP GenAI のエージェントセキュリティ 5 文書スイート
- [MIT AI Risk Repository](https://airisk.mit.edu/) — 74 フレームワーク・1,700+ リスクの統合台帳
- [AI Incident Database](https://incidentdatabase.ai/) — 実インシデント 759+ 件のデータベース
- [AVID – AI Vulnerability Database](https://avidml.org/) — AI 脆弱性データベース
- [CSA AI Controls Matrix](https://cloudsecurityalliance.org/research/working-groups/ai-controls) — 243 統制 / 18 ドメインの AI 統制マトリクス
- [Google SAIF Map](https://saif.google/secure-ai-framework/saif-map) — SAIF の AI リスク・統制マップ
- [NIST AI 600-1 Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) — 生成 AI 固有の 12 リスクカテゴリ（PDF）
- [NCSC/CISA Guidelines for Secure AI System Development](https://www.ncsc.gov.uk/collection/guidelines-secure-ai-system-development) — セキュア AI 開発ガイドライン
- [ISO/IEC 42001](https://www.iso.org/standard/81230.html) — AI マネジメントシステム（AIMS）の国際規格
- [ISO/IEC 23894](https://www.iso.org/standard/77304.html) — AI リスクマネジメントの国際規格
- [ISO/IEC 27090](https://www.iso.org/standard/56581.html) — AI セキュリティの国際規格（策定中）
- [AI 事業者ガイドライン 第 1.2 版](https://www.meti.go.jp/shingikai/mono_info_service/ai_shakai_jisso/pdf/20260331_1.pdf) — 総務省・経産省による開発者 / 提供者 / 利用者の主体分類ガイドライン（PDF）
- [IPA AI 利用時のセキュリティ脅威・リスク調査報告書](https://www.ipa.go.jp/digital/ai/security/index.html) — 国内向け AI 利用時セキュリティ脅威の調査報告
- [AISI AI セーフティに関する評価観点ガイド 第 1.20 版](https://aisi.go.jp/output/output_framework/guide_to_evaluation_perspective_on_ai_safety/) — AI セーフティ評価観点の整理（エージェント対応改訂版）
- [AISI AI セーフティに関するレッドチーミング手法ガイド 第 1.10 版](https://aisi.go.jp/output/output_framework/guide_to_red_teaming_methodology_on_ai_safety/) — レッドチーミングの計画・実施・報告の手法ガイド
- [デジタル庁 テキスト生成 AI 利活用におけるリスクへの対策ガイドブック（α版）](https://www.digital.go.jp/assets/contents/node/basic_page/field_ref_resources/c1959599-efad-472e-a640-97ae67617219/fe843dc6/20240610_resources_generalitve-ai-guidebook_01.pdf) — ユースケース別のリスク × 軽減策ガイドブック（PDF）
- [デジタル庁 DS-920 行政の進化と革新のための生成 AI の調達・利活用に係るガイドライン](https://www.digital.go.jp/assets/contents/node/basic_page/field_ref_resources/e2a06143-ed29-4f1d-9c31-0f06fca67afc/80419aea/20250527_resources_standard_guidelines_guideline_01.pdf) — 行政機関向け生成 AI ガイドライン（PDF）
- [JDLA 生成 AI の利用ガイドライン](https://www.jdla.org/document/) — 組織内利用規程のひな形（第 1.1 版）
- [IPA AI 利用者のためのセキュリティ豆知識](https://www.ipa.go.jp/security/ai/index.html) — 利用者向けセキュリティ注意点集
- [Security](https://code.claude.com/docs/en/security) — Claude Code のセキュリティモデル公式ドキュメント
- [Sandboxing](https://code.claude.com/docs/en/sandboxing) — Claude Code のサンドボックス公式ドキュメント
- [CSA research note: Slopsquatting](https://labs.cloudsecurityalliance.org/research/csa-research-note-slopsquatting-ai-supply-chain-20260419-csa/) — ハルシネーション由来のサプライチェーン攻撃に関する研究ノート
