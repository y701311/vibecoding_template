# AI Development Template Blueprint

## 0. このドキュメントの位置づけ

本ドキュメントは、AIエージェントを用いた開発を、開発知識の少ないユーザーでも一定品質以上で進められるようにするための、軽量な開発テンプレート／ガバナンス設計の最新版である。

対象は主に Databricks 上で Genie Code 等のAIエージェントと対話しながら進める、小〜中規模のデータ・分析・ML・AI Application 開発を想定する。

本設計は、独自の巨大な開発フレームワークを作ることを目的としない。
既存の Spec-Driven Development、Databricks 標準機能、Agent Skills、GitHub 等の既存エコシステムを優先的に利用し、その上に必要最小限のガバナンスと非エンジニア向け対話レイヤーを追加することを基本思想とする。

最終的な利用UXは、次を目標とする。

```text
1つの Databricks Starter ZIP を配置
↓
Genie Code と自然言語で会話開始
↓
AIが既存資産を調査
↓
不足する重要要件だけ質問
↓
Project Type / Preset / Risk / 必要Skillを判断
↓
project.md を具体化
↓
必要なフォルダ・ファイルだけ追加
↓
実装・検証・Converge
↓
必要なら本番化
```

後続エージェントは、本ドキュメントをもとに以下を具体化してよい。

- `AGENTS.md` の生成
- `project.md` テンプレートの生成
- Project Type / Preset の具体化
- Databricks 向け配布Starterの生成
- 必要最小限の Skill 定義
- 実案件への適用
- 実開発を通じた問題点の抽出
- フィードバックをもとにしたテンプレート改善

ただし、既存標準・既存Skill・Databricksネイティブ機能で代替可能なものを安易に自作してはならない。


---

# 1. 目的

本テンプレートの目的は以下の5つに限定する。

1. 開発知識の少ないユーザーから、暗黙的な要件を含めて必要な情報を引き出す。
2. AIエージェントが勝手にスコープを広げたり、未検証の状態で完了と誤認したりすることを防ぐ。
3. Databricks標準、Spec-Driven Development、既存Skill、既存OSSを最大限利用する。
4. 小規模案件では軽量に、本番・高リスク案件では厳格に運用できるようにする。
5. 1つのStarter ZIPから、会話と既存資産調査によって案件固有のプロジェクトへ自然に育てられるようにする。

最終的に目指すのは、

> 非エンジニアの曖昧な要求を、AIが必要最小限の質問で構造化し、既存の開発標準・Databricks機能を優先して実装し、要求と実行結果の両方を確認してから完了とするための軽量ガバナンステンプレート。

である。


---

# 2. 非目的

以下は本テンプレートの目的ではない。

- 独自のSDLCをゼロから作ること
- 独自のTask Managerを作ること
- 独自のIssue管理基盤を作ること
- 独自のTest Frameworkを作ること
- 独自のRisk Engineを作ること
- 独自のConvergence Engineを作ること
- 独自のDatabricks wrapperを作ること
- 独自のML frameworkを作ること
- 独自のDeployment toolを作ること
- Project Typeごとに別ZIPを大量配布すること
- 利用者にComposition処理を意識させること
- GitHub利用を全案件で必須にすること
- すべての案件で厳密な仕様書・ADR・PR運用を要求すること

必要なものは既存機能を利用し、必要になったタイミングで後付けする。


---

# 3. 基本思想

## 3.1 Understand before Build

AIは、依頼を受けてすぐに実装を開始してはならない。

ただし、すべての案件で重い要件定義を行う必要もない。

AIはまず依頼の目的、影響範囲、リスク、既存資産を確認し、その案件に必要な深さで理解を行う。


## 3.2 Ask only decision-relevant questions

質問は、回答によって以下のいずれかが変わる場合にのみ行う。

- 設計
- 実装
- 出力
- 例外時動作
- 成功条件
- セキュリティ
- 本番運用
- 破壊的変更の可否

AIが合理的かつ安全に判断できるものは、原則としてユーザーへ質問しない。

これを Decision-Relevance Gate とする。


## 3.3 Inspect before asking

質問する前に、利用可能な既存資産を調査する。

例:

- 既存コード
- Notebook
- Unity Catalog
- Tables / Views
- Jobs / Pipelines
- README / project.md
- 既存テスト
- 既存Dashboard
- MLflow資産

既存資産から判明することをユーザーに再質問しない。


## 3.4 Separate business intent from technical implementation

非エンジニアのユーザーに、技術方式を選ばせない。

悪い例:

- SCD Type 2 にしますか？
- Merge と Append のどちらにしますか？
- Structured Streaming にしますか？

良い例:

- 過去時点の値も後から確認できる必要がありますか？
- 同じデータを再投入したとき、二重登録されない必要がありますか？
- データはほぼリアルタイムで更新される必要がありますか？

技術方式の選択はAI側の責任とする。


## 3.5 Prefer the simplest design

要求を満たす中で、最も単純な設計を優先する。

以下を避ける。

- 将来使うかもしれないという理由だけの共通化
- 不要なレイヤー
- 不要な抽象化
- 不要なMedallion 3層化
- 不要なSkill追加
- 不要な設定ファイル
- 不要なフォルダ
- 不要なPreset


## 3.6 Control Scope

AIは、依頼された作業とは関係の薄い改善を勝手に行わない。

作業中に別の問題を発見した場合:

- Acceptance Criteria 達成に必要 → 現タスクで対応してよい
- 関連するが不要 → 発見事項として記録
- 無関係 → 変更しない


## 3.7 Verify behavior

コードを書いたことは完了を意味しない。

可能な範囲で実際に、

- 実行
- テスト
- データ品質確認
- 結果確認

を行う。


## 3.8 Converge against requirements

Verify と Converge を分ける。

Verify:
> 作ったものは正しく動作するか？

Converge:
> そもそも要求したものを全部作ったか？余計なものを作っていないか？

Converge では少なくとも以下を確認する。

- Missing
- Partial
- Contradiction
- Unrequested change
- Documentation drift


## 3.9 Never claim unperformed verification

実行していないテストを「成功」と報告してはならない。

確認できていない場合は、

- 未確認
- 実行不能
- 手動確認が必要

と明示する。


## 3.10 Prefer adoption over invention

既存機能・既存標準・既存Skill・Databricksネイティブ機能を、自作より優先する。

優先順位:

1. Platform Native
2. Existing Standard / Existing Skill
3. Configure
4. Compose
5. Thin Wrapper
6. Custom Build

Custom Build は最後の手段とする。

独自実装を行う場合は、最低限以下を説明できる必要がある。

> Custom implementation justification: 既存の○○では△△要件を満たせないため。


## 3.11 Grow only when needed

プロジェクト構造、Skill、Preset、ドキュメントは最初から完成形を生成しない。

会話・既存資産調査・Risk判断によって必要性が明らかになったものだけ追加する。

追加と同様に、不要になったものを削除することもAIの責任とする。


---

# 4. 配布モデル

## 4.1 基本は1つのDatabricks Starter ZIP

案件種類ごとにZIPを分けない。

基本配布物:

```text
ai-dev-databricks-starter.zip
```

これ1つで以下を扱う。

- Analytics
- Data Engineering
- Machine Learning
- AI Application


## 4.2 ZIPを分ける場合

Project TypeではなくPlatform差が大きい場合のみ検討する。

例:

```text
databricks-starter.zip
generic-python-starter.zip
```

現時点ではDatabricks版のみを主対象とする。


## 4.3 配布時に確定するもの

Starter ZIP作成時に確定・Materializeしておく。

- Core Principles
- Databricks向け基本Adapter
- Bootstrap instruction
- project.md テンプレート
- Project Type guidance
- Preset guidance
- 最低限のテンプレート


## 4.4 実行時に決定するもの

AIとの会話・既存資産調査の中で決定する。

- Primary Project Type
- Supporting Project Type
- Preset
- Risk Level
- Optional Skill
- Catalog / Schema等の案件設定
- 必要な追加フォルダ
- 必要な追加ドキュメント
- 必要なJob / Pipeline / Bundle定義


---

# 5. Bootstrap

ZIPをDatabricks上に配置後、ユーザーは基本的に自然言語でやりたいことを伝えるだけでよい。

`AGENTS.md` には初回起動時の動作を明示する。

概念例:

```text
If this project has not been initialized:

1. Read AGENTS.md and docs/project.md.
2. Inspect existing project files and Databricks assets.
3. Understand the user's goal.
4. Determine likely Project Type and Risk.
5. Ask only decision-relevant questions.
6. Update docs/project.md.
7. Select applicable guidance / preset.
8. Create only the files and folders that are currently necessary.
9. Do not begin substantial implementation until requirements are sufficiently clear for the selected risk level.
```


---

# 6. Runtime Composition

実行時には概念上、以下が行われる。

```text
Conversation
+
Existing assets
+
Core
+
Databricks guidance
+
Selected Project Type
+
Selected Preset
+
Selected Optional Skills
↓
Project-specific implementation context
```

ただし、専用Composition Engineはv1では作らない。

AIエージェント自身に選択・統合を行わせる。


## 6.1 実行時操作の3分類

### Select

既存のProject Type / Preset / Skillを選ぶ。

### Configure

案件固有値を設定する。

例:

- Catalog
- Schema
- Input Table
- Output Table
- Forecast horizon
- KPI
- SLA

### Generate

本当に必要な成果物だけ生成する。

例:

- spec
- src
- tests
- Job
- Pipeline
- Bundle resource


## 6.2 重要判断の可視化

AIがProject Type、Preset、Risk等を選択した場合、ユーザーへ簡潔に理由を説明する。

例:

> 時系列予測が中心のためPrimary TypeをML、データ整備をSupportingのData Engineeringとします。現時点は検証段階なのでStandard扱いとし、本番Job化時にProductionへ再評価します。

ユーザーに選択させる必要はないが、重要判断をブラックボックス化しない。


---

# 7. Business Alignment

## 7.1 IPO+C

非エンジニアとの認識合わせには IPO+C を利用する。

ただし、IPO+C は正式仕様言語ではなく、会話用の中間表現である。

- Purpose
- Input
- Process
- Output
- Conditions


### Purpose

何のために行うのか。
誰が何を判断・自動化するためのものか。


### Input

何を受け取るのか。


### Process

業務的に何をするのか。


### Output

最終的に何が得られるのか。


### Conditions

Input / Process / Output だけでは表現できない条件。

例:

- 実行頻度
- データ量
- 品質条件
- SLA
- 権限
- 履歴保持
- 再実行時動作
- エラー時動作
- 保持期間


## 7.2 暗黙要件チェック観点

AIは内部的に以下の観点をチェックする。

- Purpose
- Users
- Input
- Process
- Output
- Timing
- Volume
- Quality
- History
- Failure
- Recovery
- Security
- SLA

ただし、すべてを質問してはならない。


## 7.3 Requirement State

必要な場合のみ、以下の状態を使う。

- Known
- Assumed
- Unknown
- Decision

専用管理ツールは作らない。


---

# 8. Risk-Based Development

厳密な点数制のRisk Engineは作らない。

3段階だけ使う。


## 8.1 Small

例:

- 列名変更
- SQL条件修正
- 表示変更
- 簡単なバグ修正

フロー:

```text
UNDERSTAND
→ BUILD
→ VERIFY
```


## 8.2 Standard

例:

- 新規集計
- Dashboard追加
- Excel処理
- 新しいNotebook
- 中規模ロジック追加

フロー:

```text
UNDERSTAND
→ SPECIFY
→ BUILD
→ VERIFY
→ CONVERGE
```


## 8.3 Production / High Risk

例:

- 本番Pipeline
- 本番データ更新
- Gold仕様変更
- 金額ロジック
- KPI定義変更
- 権限変更
- 外部公開
- 外部連携
- MLモデル本番投入
- Schema contract変更

フロー:

```text
UNDERSTAND
→ CLARIFY
→ SPECIFY
→ PLAN
→ CRITIQUE
→ BUILD
→ VERIFY
→ CONVERGE
→ OPERATE
```


## 8.4 High Risk Trigger

以下を含む場合は原則 Production / High Risk 扱い。

- Production mutation
- Delete / destructive operation
- Money / KPI
- Permissions / Security
- External publication
- Sensitive data
- Model deployment
- Schema contract change


## 8.5 Riskは途中で変わってよい

例:

```text
PoC / 精度検証
Standard
↓
毎日自動実行したい
Production
```

Risk再評価に応じて必要なSkill・DoD・ファイルを追加する。


---

# 9. 基本開発フロー

人間が覚える基本フローは5工程だけ。

```text
UNDERSTAND
→ SPECIFY
→ BUILD
→ VERIFY
→ CONVERGE
```

必要に応じて追加:

- CLARIFY
- PLAN
- CRITIQUE
- OPERATE


---

# 10. Scope / Constraints

基本:

- Scope
- Acceptance Criteria

必要な場合のみ:

- Out of Scope
- Must
- Must Not
- Prefer
- Escalate


---

# 11. Business Scenario

非エンジニアとのテスト合意には Business Scenario を使う。

例:

```text
Scenario: 同じ月のExcelを再投入する

Given:
既に同じデータが処理済みである

When:
同じファイルを再投入する

Then:
売上が二重計上されない
```

流れ:

```text
Business Scenario
→ Acceptance Criteria
→ Technical Test
```


---

# 12. Definition of Done

## 12.1 Global DoD

- Acceptance Criteriaを満たす
- 必要なVerificationを実施
- RequirementとのConvergenceを確認
- Scope外変更がない
- 未検証事項を明示


## 12.2 Production Data DoD

- Idempotent
- Failure detectable
- Recovery method defined
- Monitoring defined


## 12.3 ML DoD

- Baseline comparison
- Evaluation split appropriate
- Leakage check
- Metrics reported


## 12.4 AI Application DoD

必要に応じて:

- Evaluation scenarios
- Response quality
- Retrieval quality
- Tool selection / execution
- Hallucination確認
- Prompt injection等
- Trace / latency / cost


---

# 13. Project Types

大分類のみとし、4類型で固定する。

## 13.1 Analytics

- SQL分析
- Excel処理
- Dashboard
- Notebook分析
- レポート
- EDA


## 13.2 Data Engineering

- Ingestion
- Transformation
- ETL / ELT
- Pipeline
- Job
- Gold dataset
- Reporting dataset


## 13.3 Machine Learning

- Forecast
- Classification
- Regression
- Anomaly Detection
- Feature Engineering
- Training
- Inference


## 13.4 AI Application

- LLM Application
- RAG
- AI Agent
- AI Chat
- AI Function
- Tool Calling
- Agentic Workflow


---

# 14. 複数Project Type

1案件1Typeに限定しない。

例:

```text
Demand Forecasting
Primary: Machine Learning
Supporting:
- Data Engineering
- Analytics
```

```text
RAG
Primary: AI Application
Supporting:
- Data Engineering
```


---

# 15. Preset

典型案件をPresetとして扱う。

候補:

- demand-forecasting
- monthly-excel-report
- dashboard
- batch-ingestion
- rag
- ai-agent

Presetは「完成プロジェクトのコピー元」ではなく差分ガイダンスである。

主に以下を追加する。

- 推奨Project Type
- Supporting Type
- 要件確認観点
- Business Scenario観点
- DoD追加項目
- 推奨Skill
- 典型検証項目


## Preset追加条件

- 複数案件で再利用する
- 既存Presetとの差が有意

案件固有のものをPreset化しない。


---

# 16. Skill

Skillは能力単位とする。

候補:

- data-analysis
- data-engineering
- ml-development
- ai-application
- productionize
- github-development

ただし公式Skill・既存Skillで代替できる場合は自作しない。


## 16.1 RuntimeでのSkill追加

「Skillが増える」とは原則、

> 既存Skillを案件に応じて有効化・参照する

ことを意味する。

毎案件新しいSkillを生成することではない。


## 16.2 新規Skill生成

以下を満たす場合のみ。

- 複数案件で再利用可能
- 既存Skillに統合しにくい
- 既存公式Skillで不足
- そのSkillによりAIの判断・実装・検証が実際に変わる


---

# 17. Databricks 方針

## 17.1 Databricks Native First

```text
Orchestration
→ Lakeflow Jobs

Pipeline
→ Lakeflow Pipelines

Governance
→ Unity Catalog

Experiment Tracking
→ MLflow

Deployment
→ Declarative Automation Bundles

BI
→ AI/BI
```

標準機能を独自Pythonフレームワークで再実装しない。


## 17.2 Databricks固有指示

配布版StarterではDatabricks Adapterの内容を最終的な `AGENTS.md` にMaterializeしてよい。

利用者に `adapters/databricks/AGENTS.append.md` 等の内部構造を意識させない。

フレームワーク開発側ではAdapterを部品として管理してよいが、配布物では原則統合済みとする。


## 17.3 Notebook Policy

Notebook完成 = 開発完了とはしない。

探索:

```text
notebooks/exploration/
```

本番ロジックは必要に応じて:

- `src/`
- SQL
- Job
- Pipeline
- Bundle resource

へ分離する。


## 17.4 Medallion

標準パターンとして優先的に検討するが必須ではない。

目的:

- データ品質境界
- 責務分離
- 再処理境界
- 利用契約

単純案件では簡略化可能。


## 17.5 Production Ready

重視する順:

- Correctness
- Reliability
- Recoverability
- Observability
- Cost

必ず検討:

1. 同じ入力を2回処理するとどうなるか
2. 途中失敗後に再実行するとどうなるか
3. Backfillするとどうなるか
4. 部分成功時にどう復旧するか


---

# 18. GitHub

Optional Skill。

有効時のみ:

- Issue
- Branch
- Commit
- PR
- Review
- CI

小規模Databricks案件では必須ではない。


---

# 19. ドキュメント方針

最初から大量のドキュメントを生成しない。

中心:

```text
docs/project.md
```

推奨構造:

```markdown
# Project

## Purpose

## Risk Level

## Selected Context
- Primary Type
- Supporting Types
- Preset
- Active Skills

## IPO+C

### Input
### Process
### Output
### Conditions

## Scope

## Acceptance Criteria

## Business Scenarios

## Constraints

## Architecture Summary

## Verification

## Current Status
```

大きくなった場合のみ分離:

- requirements.md
- architecture.md
- operations.md
- data-contracts.md
- specs/


---

# 20. project.md は会話で育つ

初期:

```text
Purpose: TBD
Risk: TBD
Input: TBD
...
```

会話・既存資産調査によってAIが更新していく。

ユーザーが最初に全部埋めるフォームではない。

`project.md` は、

> AIとユーザーの間で徐々に具体化される軽量なプロジェクト契約

として扱う。


---

# 21. Spec-Driven Developmentとの関係

Spec Kit等と互換性を持つが特定CLIを必須にしない。

既存SDDが利用可能なら優先。

Small案件では重いSDDフローを省略してよい。


---

# 22. Critique

Production / High Riskのみ原則実施。

観点:

1. 要求漏れ
2. 過剰設計
3. より単純な方法
4. Databricks標準で代替できないか
5. 運用上の重大リスク


---

# 23. Evidence-Based Completion

推奨報告:

```text
Result
- 実装内容

Verification
- 実施した確認
- 結果

Convergence
- Acceptance Criteriaとの突合

Not Verified
- 未確認事項

Remaining
- 残課題

Status
- VERIFIED / DONE
```

Production案件ではRuntime Verificationを必要に応じて追加。


---

# 24. 状態管理

必要なら:

- TODO
- IMPLEMENTING
- VERIFIED
- DONE

Productionのみ必要に応じて:

- PRODUCTION-VERIFIED

複雑なState Machineは作らない。


---

# 25. プロジェクト構造は実行時に成長する

## 25.1 Starter初期状態

極力小さくする。

例:

```text
project/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
└── guidance/
    ├── project-types/
    ├── presets/
    └── templates/
```


## 25.2 会話後のStandard案件例

```text
project/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
├── guidance/
├── src/
├── notebooks/
│   └── exploration/
└── tests/
```


## 25.3 Production化後

必要に応じて:

```text
resources/
├── jobs/
└── pipelines/

databricks.yml
```

などを追加。


## 25.4 Hygiene

Converge時に以下も確認する。

- 未使用Skill
- 空フォルダ
- 不要Preset依存
- 重複ドキュメント
- 一時Notebook
- 不要な生成物

追加だけでなく削除も行う。


---

# 26. 配布ZIPの推奨構造

Databricks利用者向けの配布物では、内部のAdapter/Composition構造を隠す。

```text
ai-dev-databricks-starter/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
└── guidance/
    ├── project-types/
    │   ├── analytics.md
    │   ├── data-engineering.md
    │   ├── ml.md
    │   └── ai-application.md
    ├── presets/
    │   ├── demand-forecasting.md
    │   ├── monthly-excel-report.md
    │   ├── batch-ingestion.md
    │   ├── dashboard.md
    │   ├── rag.md
    │   └── ai-agent.md
    └── templates/
        ├── business-scenario.md
        └── production-readiness.md
```

`src/`, `tests/`, `resources/` 等は必要になったときAIが追加してもよい。

空フォルダを配布時点で大量生成しない。


---

# 27. フレームワーク開発側の構造

利用者向けZIPとは別に、テンプレート自体を保守するソース側では部品化してよい。

例:

```text
framework-source/
├── core/
├── adapters/
│   └── databricks/
├── project-types/
├── presets/
├── templates/
└── packaging/
```

配布時:

```text
Core
+
Databricks Adapter
+
Guidance
↓
Materialize
↓
Databricks Starter ZIP
```

利用者はこのCompositionを意識しない。


---

# 28. Composition Engineは作らない

論理概念としては、

```text
Resolve
→ Compose
→ Materialize
```

を持つ。

ただしv1では専用CLI・専用Engineを作らない。

- 配布時Materialization → 手作業/単純処理/AIで十分
- 実行時Composition → AI自身に任せる

複数案件で同じ機械的変換が繰り返されると確認できた場合のみ、自動化を検討する。


---

# 29. Relevance Gate

ルール、Skill、Preset、ドキュメント、質問、フォルダを追加する際:

> これを削除した場合、合理的なAIの実装・レビュー・検証判断が実際に変わるか？

変わらなければ追加しない。


---

# 30. v1 実装計画

最初の配布版として作るもの:

```text
ai-dev-databricks-starter/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
└── guidance/
    ├── project-types/
    │   ├── analytics.md
    │   ├── data-engineering.md
    │   ├── ml.md
    │   └── ai-application.md
    ├── presets/
    │   ├── demand-forecasting.md
    │   ├── monthly-excel-report.md
    │   ├── batch-ingestion.md
    │   ├── dashboard.md
    │   ├── rag.md
    │   └── ai-agent.md
    └── templates/
        ├── business-scenario.md
        └── production-readiness.md
```

ただしPreset数は実装時レビューでさらに削減してよい。


---

# 31. v1で作らないもの

- CLI
- GUI
- Task Manager
- Risk Engine
- Convergence Engine
- Issue Manager
- Test Framework
- Databricks wrapper
- 独自ML framework
- Deployment tool
- Runtime Composition Engine
- 自動Skill installer
- 大量のPreset
- 大量の空フォルダ


---

# 32. 実案件フィードバック項目

## Usability

- ZIP配置後すぐ会話を始められるか
- 初期説明が必要すぎないか
- 質問が多すぎないか
- 非エンジニアが回答できるか


## Requirement Quality

- 暗黙要件を拾えたか
- IPO+Cが役立ったか
- project.mdが自然に育ったか


## Runtime Selection

- Type選択は適切だったか
- Preset選択は適切だったか
- Risk判断は適切だったか
- Skillを読みすぎていないか
- 重要判断の説明は十分か


## Project Growth

- 必要なファイルだけ増えたか
- 空/不要フォルダが増えなかったか
- SkillやPresetが増殖しなかったか


## Development Quality

- Scope creepを防げたか
- 過剰設計を防げたか
- Native-before-Customが守られたか


## Verification

- Business Scenarioが有効だったか
- VerifyとConvergeの分離が有効だったか
- 未検証事項が正しく報告されたか


## Production

- Idempotency
- Recovery
- Monitoring
- Cost
- Job / Pipeline化

が適切だったか。


---

# 33. 改善判断ルール

改善時の優先順位:

1. 不要なものを削除
2. 既存標準へ置換
3. 既存要素へ統合
4. 設定で対応
5. 薄いWrapper
6. 最後に新規Skill / 新規仕組み

「機能追加」より「複雑性削減」を優先する。


---

# 34. 批判的レビュー

ここまでの設計に対する主要な懸念と対応を以下に示す。


## 34.1 懸念: 1ZIPに全ガイダンスを入れるとコンテキストが肥大化する

### 問題

Analytics / DE / ML / AI Application / 多数Presetを全部AIが毎回読むと、Starterが軽量でもコンテキストは重くなる。

### 改善

- `AGENTS.md` には全ガイダンスを直接埋め込まない
- 最初はCore + Bootstrapだけ読む
- Project Type推定後に関連ガイダンスだけ参照させる
- Presetも候補が絞れた時だけ読む
- 「全ファイルを最初に読む」指示は禁止する

配布ZIPは1つでも、**Runtime Contextは選択的にロードする**。


## 34.2 懸念: AIが勝手にファイルを増やしすぎる

### 改善

ファイル/フォルダ生成にもRelevance Gateを適用する。

生成条件:

- 実装に必要
- 検証に必要
- 運用に必要
- 今後のAI判断に必要

「標準構造だから」という理由だけで作らない。


## 34.3 懸念: Skillの「有効化」が実装上曖昧

### 問題

各AgentのSkillロード方式は異なる可能性がある。

### 改善

Blueprintでは「Active Skill」を論理概念として扱う。

物理的な、

- コピー
- 参照
- Agent Skillディレクトリ配置
- built-in skill利用

はAgent/Platform Adapterに任せる。

つまり、**Logical Skill Selection と Physical Skill Installationを分離**する。


## 34.4 懸念: Presetが増殖する

### 改善

v1ではPreset数を最小限にする。

最初から6個全部必要とは限らない。

推奨初期候補:

- demand-forecasting
- monthly-excel-report
- batch-ingestion
- rag

DashboardはAnalytics Typeだけで十分ならPreset化しない。
AI AgentもAI Application Type + RAG等との差が固まるまでは追加を保留してよい。


## 34.5 懸念: `project.md` が何でも入る巨大ファイルになる

### 改善

小〜中規模では単一ファイルを維持するが、分割Triggerを定める。

分割候補:

- 仕様が複数独立機能に分かれる
- Architecture説明が長大
- Operations手順が独立して必要
- Data Contractが複数存在
- AIが必要情報を見つけにくくなる

「行数がNを超えたら」のような形式的ルールにはしない。


## 34.6 懸念: Runtime Type / Preset選択がブラックボックス化する

### 改善

重要選択は短く説明する。
ただしユーザー承認を毎回要求しない。

High Riskまたは不可逆判断のみ明示確認する。


## 34.7 懸念: ZIP更新後、既存案件との乖離が起きる

### 改善

Starterはプロジェクト作成時の起点と位置づける。

既存案件へ毎回最新Starterを上書き適用しない。

将来的に共通ルール更新が必要になった場合も、まずは差分確認を行う。
自動マイグレーション機構はv1では作らない。


## 34.8 懸念: 配布時MaterializeとRuntime Compositionの二重概念が複雑

### 改善

利用者向けには隠す。

利用者から見えるのは:

```text
ZIPを置く
→ 会話する
→ Projectが育つ
```

のみ。

Resolve / Compose / Materializeはフレームワーク保守者向けの設計概念とする。


## 34.9 懸念: 「即開発開始」が「即実装開始」と誤解される

### 改善

即開発開始の定義:

> ZIP配置後すぐ、既存資産調査・要件理解・必要な質問・project.md具体化を開始できる。

必ずしも即コード生成を意味しない。


## 34.10 懸念: Databricks以外へ広げすぎる

### 改善

v1ではDatabricks専用に集中する。

Platform abstractionは論理的には残すが、Generic Pythonや他Platform向けZIPを先に作らない。


---

# 35. レビュー反映後のv1最終案

レビューを踏まえ、v1では以下に絞る。

```text
ai-dev-databricks-starter/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
└── guidance/
    ├── project-types/
    │   ├── analytics.md
    │   ├── data-engineering.md
    │   ├── ml.md
    │   └── ai-application.md
    ├── presets/
    │   ├── demand-forecasting.md
    │   ├── monthly-excel-report.md
    │   ├── batch-ingestion.md
    │   └── rag.md
    └── templates/
        ├── business-scenario.md
        └── production-readiness.md
```

重要:

- `src/` は最初から作らない
- `tests/` は最初から作らない
- `resources/` は最初から作らない
- `databricks.yml` もBundle利用が必要になるまで作らない
- 追加Skillファイルも最初から大量同梱しない
- Databricks公式Skillが使えるならそちらを優先する


---

# 36. v1の最重要Bootstrap動作

AIは初回会話時に以下を行う。

```text
1. Core rulesを読む
2. project.mdを読む
3. ユーザー目的を理解
4. 既存Databricks資産を調査
5. Primary / Supporting Typeを仮決定
6. 必要ならPreset候補を選ぶ
7. Riskを仮決定
8. 重要な不明点だけ質問
9. project.mdを更新
10. 必要なガイダンスだけ読む
11. 必要なファイルだけ生成
12. 開発開始
```

これをStarterの中心UXとする。


---

# 37. 最終的なCore

1. Understand before build
2. Inspect before asking
3. Ask only decision-relevant questions
4. Separate business intent from technical implementation
5. Prefer the simplest sufficient design
6. Control scope
7. Verify behavior
8. Converge against requirements
9. Never claim unperformed verification
10. Native / Existing before Custom
11. Grow only when needed


---

# 38. 最終要約

このテンプレートはAIに開発方法を細かく固定するためのものではない。

利用者から見える理想形は、

```text
1つのZIPを置く
↓
やりたいことを話す
↓
AIが既存環境を調べる
↓
必要なことだけ質問する
↓
案件に合うType / Preset / Riskを判断する
↓
project.mdが育つ
↓
必要なフォルダ・ファイル・既存Skillだけ追加する
↓
Databricks標準を優先して開発する
↓
実際に検証する
↓
要求とのズレを確認する
↓
不要物を整理する
↓
証拠付きで完了を報告する
```

というものである。

フレームワーク自身も「自作しすぎない」「最初から作りすぎない」「必要になったら育てる」を最重要原則とする。

---

# 39. 開発ユーザーから見たUX

本テンプレートは、内部的には Risk、Project Type、Preset、Skill、SDD、Databricks Adapter など複数の概念を持つが、
開発ユーザーにそれらを意識させることを目的としない。

ユーザーから見た理想UXは、

```text
ZIPを置く
↓
やりたいことを自然言語で話す
↓
AIが既存環境を調査する
↓
必要なことだけ聞かれる
↓
認識合わせ
↓
開発が進む
↓
確認済み／未確認が分かる形で結果が返る
```

という単純な流れである。


## 39.1 ユーザーが最初に行うこと

原則として必要な操作は以下だけとする。

1. Databricks Starter ZIPを取得する
2. ZIPを解凍し、Databricks上の作業フォルダへ配置する
3. Genie Codeを開く
4. 自然言語でやりたいことを伝える

例:

> 毎月届く店舗別のExcelをまとめて、店舗ごとの売上を確認できるようにしたい。

または、

> このSalesデータから来月の需要予測を作りたい。まず精度を確認したい。

ユーザーが最初に以下を決める必要はない。

- Project Type
- Preset
- Skill
- Risk Level
- Medallion構成
- Job / Pipeline方式
- テストフレームワーク
- MLflow利用方法
- Bundle構成
- フォルダ構造


## 39.2 初回会話時にAIが行うこと

AIはユーザーへ大量の質問を返す前に、利用可能な既存資産を調査する。

例:

- 現在のプロジェクトファイル
- Notebook
- Unity Catalog
- 既存Table / View
- 既存Job / Pipeline
- 既存テスト
- MLflow資産
- Dashboard
- README / project.md

その結果から、以下を仮判断する。

- ユーザーのPurpose
- Primary Project Type
- Supporting Project Type
- Preset候補
- Risk Level
- 既存資産の再利用可否

AIは重要な判断について短く説明する。

例:

> 今回は需要予測が中心なのでPrimary TypeをMachine Learning、
> 既存のSilver売上データを利用するためSupporting TypeをData Engineeringとします。
> 現時点では精度検証が目的なのでStandard扱いで進めます。
> 本番自動実行に進む場合はProductionとして再評価します。

これは説明であり、通常はユーザーに選択作業を要求しない。


## 39.3 要件ヒアリングのUX

AIは「質問票」をユーザーに渡してはならない。

以下の順序を基本とする。

```text
既存情報から推測
↓
安全に仮定可能なら仮定
↓
設計や成功条件が変わる不明点だけ質問
```

質問は業務言語で行う。

悪い例:

> MERGEでUpsertしますか？

良い例:

> 同じデータをもう一度取り込んだ場合、二重登録されない必要がありますか？

悪い例:

> SCD Type 2にしますか？

良い例:

> マスタの値が後から変わったとき、当時の値も後から確認できる必要がありますか？


## 39.4 認識合わせのUX

重要な案件では、実装前にAIがユーザーの要求を簡潔にまとめる。

ユーザーに見せる表現は、内部用語より分かりやすさを優先する。

例:

```text
目的
店舗ごとの翌月需要を予測し、発注判断に利用する。

入力
- 日次売上
- 商品マスタ
- 店舗マスタ

処理
- データ確認
- 必要な特徴量生成
- 過去データを使った精度検証
- 翌月予測

出力
- 店舗×商品ごとの予測数量
- 精度指標

重要条件
- 過去情報を未来予測へ混ぜない
- 現行方式より精度が悪い場合は採用しない
```

必要に応じてAIは、

> この理解で進めます。違う点があれば教えてください。

と確認する。

High Risk案件では、重要な不可逆判断のみ明示的な承認を求める。


## 39.5 ユーザーが見る主なファイル

非エンジニアユーザーが日常的に意識するファイルは極力少なくする。

原則:

### `README.md`

- このStarterの使い方
- Genie Codeへの最初の依頼例
- プロジェクトの再開方法

### `docs/project.md`

- 何を作っているか
- 現在の要求
- 入力／処理／出力
- 成功条件
- 現在の状態

### `AGENTS.md`

基本的にはAI向け。
ユーザーが毎回編集することを前提としない。

その他の技術ファイルは、必要になった場合にAIが生成・管理する。


## 39.6 project.mdのUX

`project.md` はユーザーが最初に記入する申請フォームではない。

初期状態ではTBDがあってよい。

会話と既存資産調査によってAIが更新する。

ユーザーから見ると、

> 会話内容をAIが整理して残してくれるプロジェクトの共通認識

として扱う。

AIは重要な要求変更が発生した場合、実装だけでなく `project.md` も更新する。


## 39.7 開発開始後のUX

通常、ユーザーは細かな技術ステップを指示する必要はない。

例:

> その方針で進めて。

だけでよい。

AIは必要に応じて、

- フォルダ追加
- ソースコード生成
- Notebook作成
- Test作成
- Job / Pipeline定義
- Bundle定義

を行う。

ただし、新しい構造を作ること自体を目的化しない。


## 39.8 途中確認のUX

AIは低価値な確認で作業を頻繁に止めない。

ユーザー確認が必要なのは主に、

- 要求に複数の合理的解釈があり結果が大きく変わる
- 本番データを破壊的に変更する
- Gold / KPI等の意味を変更する
- 権限や公開範囲を変更する
- セキュリティ上重要な判断
- 大きな追加コストが発生する
- 要求同士が矛盾している

場合とする。

それ以外は、合理的な仮定を明示して進めてよい。


## 39.9 Risk昇格時のUX

案件は途中で性質が変わることがある。

例:

```text
「まず精度を見たい」
↓
Standard

「良さそうなので毎朝自動実行したい」
↓
Productionへ昇格
```

この場合AIは、

> 本番自動実行へ進むため、ここからProduction扱いに切り替えます。
> 再実行時の安全性、失敗時の復旧、監視、実行コストを追加で確認します。

のように、何が変わるかを短く説明する。

ユーザーに内部Skill選択をさせる必要はない。


## 39.10 完了時のUX

「できました」だけで終了しない。

ユーザーに分かる形で以下を報告する。

```text
実装したこと
- 需要予測処理を作成
- 精度評価を追加
- 結果テーブルを作成

確認したこと
- 学習処理の実行: 成功
- Backtest: 成功
- Baseline比較: 実施済み

まだ確認していないこと
- 毎日の自動Job実行
- 本番データ量での処理時間

残っていること
- 本番化する場合はJob化と監視設定が必要

状態
VERIFIED
```

ユーザーが「何が本当に確認済みか」を一目で理解できることを優先する。


## 39.11 再開時のUX

数日後や別の担当者が作業を再開するときも、長い説明を要求しない。

ユーザーは例えば、

> このプロジェクトを続けたい。今どこまで進んでいる？

と聞けばよい。

AIは、

- `project.md`
- 現在のコード
- Tests
- Job / Pipeline
- 残課題

を確認し、現在地を説明してから再開する。


## 39.12 ユーザーが求められないこと

このStarterでは、通常の開発ユーザーに以下を要求しない。

- AGENTS.mdの手動編集
- Skillファイルの手動コピー
- Presetの手動選択
- Project Typeの手動選択
- Riskスコアの入力
- 複雑な要件定義フォームの記入
- フォルダ構造の設計
- Databricks機能の選定
- テスト種類の選定
- Job定義の手書き
- SDD用CLIの利用

必要ならAIが提案・実施する。


## 39.13 良いUXの会話例

ユーザー:

> このExcelを毎月集計して、部署ごとの実績帳票を作りたい。

AI:

> 既存ファイルと現在のDatabricks資産を確認します。
> 今回はExcel集計・帳票生成が中心なのでAnalyticsをPrimaryとして扱います。
> 1点だけ、同じ月のExcelを再投入した場合に既存結果を置き換える想定か、追加する想定かで処理が変わります。どちらが正しいですか？

ユーザー:

> 置き換えたい。

AI:

> 了解です。再実行しても二重計上されないことを成功条件に含めます。
> まず入力・処理・出力を `project.md` に整理し、その後実装とテストを進めます。


## 39.14 避けるべきUX

以下は避ける。

### 質問攻め

```text
Q1. Catalogは？
Q2. Schemaは？
Q3. Mergeですか？
Q4. SLAは？
Q5. Computeは？
Q6. Bronzeは？
...
```

### 技術選択の丸投げ

> Lakeflow PipelineとJobのどちらにしますか？

### 書類作成の強制

> requirements.md、architecture.md、test-plan.mdを先に全部埋めてください。

### 毎回承認待ち

> このファイルを作ってよいですか？
> testsフォルダを作ってよいですか？
> Notebookを作ってよいですか？

### ブラックボックスな自動判断

> Productionにしました。Skillを5つ追加しました。

理由を簡潔に説明すべきである。


## 39.15 UX成功基準

StarterのUXは以下を満たすことを目標とする。

- ユーザーがZIP配置後すぐ自然言語で開始できる
- 初回に専門用語を理解する必要がない
- AIの質問が意思決定に必要なものへ絞られている
- AIの重要判断が簡潔に説明される
- 技術方式をユーザーに丸投げしない
- ユーザーが主に見るドキュメントは少ない
- 本番化時だけ追加確認が増える
- 完了時に確認済み／未確認が分かる
- 数日後でも会話だけで開発を再開できる
- フレームワークそのものの操作が目的化しない


---

# 40. UX観点での実案件フィードバック

実案件でStarterを試す際は、技術品質だけでなくUXも評価する。

## 40.1 初期導入

- ZIP配置から最初の有用な会話まで迷わなかったか
- READMEを長く読む必要がなかったか
- 初回プロンプトが自然だったか


## 40.2 ヒアリング

- 質問数は適切だったか
- 技術用語を要求されなかったか
- 既に分かる情報を再質問されなかったか
- 本当に重要な暗黙要件を拾えたか


## 40.3 自動判断

- Project Type選択が自然だったか
- Risk判断の説明が理解できたか
- 不要なPreset / Skillが有効化されなかったか


## 40.4 開発中

- 確認のために頻繁に作業が止まらなかったか
- AIが勝手にスコープを広げなかったか
- project.mdが現在の認識を正しく表していたか


## 40.5 完了

- 実装結果を非エンジニアでも理解できたか
- 確認済み／未確認が区別できたか
- 次に何をすべきか分かったか


## 40.6 再開

- 後日、過去の会話を詳細に説明し直さず再開できたか
- AIがプロジェクトの現在地を正しく復元できたか


UX上の問題が見つかった場合、まず以下の順で改善する。

1. 不要な質問・操作を削除
2. AIによる調査・推測へ移す
3. 表現を業務言語へ変更
4. 既存ファイルへ情報を統合
5. 最後に新しいUX機構を追加する

---

# 41. Genie Code Skill Integration

本Blueprintでは、Databricks Genie Code の正式な Skill 機構を利用可能な前提で設計する。

ただし、以下を明確に分離する。

```text
Project Guidance
≠
Genie Code Skill
```

Project Guidance は、そのプロジェクトで参照する設計・Preset・テンプレート等である。

Genie Code Skill は、Genie Code がタスクとの関連性に応じてロードする正式な専門ワークフロー／知識単位である。


## 41.1 AGENTS.md と Skill の責務分担

### AGENTS.md

常時守るべきプロジェクト原則を置く。

例:

- Understand before build
- Inspect before asking
- Ask only decision-relevant questions
- Scope control
- Native / Existing before Custom
- Verify / Converge
- Evidence-based completion
- Medallionを標準パターンとして検討する
- プロジェクト固有Convention

AGENTS.md は、AIに常に適用してほしいルールを中心とする。


### SKILL.md

特定タスクで必要になる専門ワークフローを置く。

例:

```text
demand-forecasting
- time-series split
- backtesting
- baseline comparison
- leakage check
- forecast horizon確認

production-readiness
- idempotency
- retry
- recovery
- monitoring
- production verification

organization-specific-data-quality
- 組織固有の品質基準
- 必須チェック
- エスカレーション条件
```

Skill は、タスクとの関連性がある場合のみ利用する。


## 41.2 Genie Code Skill の正式配置

Genie Code の正式Skillとして利用する場合、各Skillは専用ディレクトリと `SKILL.md` を持つ。

概念例:

```text
.assistant/
└── skills/
    ├── demand-forecasting/
    │   └── SKILL.md
    └── organization-data-quality/
        └── SKILL.md
```

Workspace共有Skillと個人用Skillでは配置場所が異なるため、
物理的な配置はDatabricksの現行仕様に従うこと。

Blueprintは配置先を独自抽象化して上書きしない。


## 41.3 Starter ZIP内の `guidance/` と Skill を混同しない

Starter ZIPに含める、

```text
guidance/
├── project-types/
├── presets/
└── templates/
```

は、原則としてプロジェクト内ガイダンスであり、
置いただけでGenie Code正式Skillになることを前提としない。

したがって、

```text
guidance/presets/demand-forecasting.md
```

と、

```text
.assistant/skills/demand-forecasting/SKILL.md
```

は別の責務を持つ。

Preset:

> この種類の案件で何を考慮するべきか

Skill:

> この専門タスクをどのような手順・基準で実行するか


## 41.4 Skill利用時の優先順位

Skillが必要と思われる場合、AIは以下の順番で確認する。

```text
1. Genie Code built-in capability で十分か
↓
2. Databricks公式 / 既存Skillが存在するか
↓
3. Workspace / User Skillとして既に存在するか
↓
4. Starter内の再利用可能Skillを利用できるか
↓
5. 既存Skillの設定・組み合わせで対応できるか
↓
6. 本当に必要な場合のみ新規Skillを作成
```

これは `Native / Existing before Custom` 原則のSkill版である。


## 41.5 自作しないSkill

Genie Code / Databricks が既に十分理解している一般的な操作を、
単に説明し直すだけのSkillは作らない。

原則として以下のようなSkillは不要。

```text
unity-catalog-basic
notebook-basic
spark-basic
sql-basic
mlflow-basic
jobs-basic
pipeline-basic
```

既存能力で不足する具体的理由がある場合のみ検討する。


## 41.6 自作Skillに向いているもの

独自Skillは、AIの判断・実装・検証方法を実際に変えるものに絞る。

例:

- 組織固有のData Quality基準
- 独自の需要予測評価基準
- 特定業務の会計／集計ルール
- 組織固有の本番化チェック
- 特定の再利用可能な専門ワークフロー

案件固有の一度きりの情報はSkill化せず、`project.md` 等に置く。


## 41.7 Runtimeで「Skillが増える」の意味

会話中にSkillが増えるとは、原則として、

```text
案件理解が進む
↓
必要な能力が判明
↓
既存Skillを選択・有効化
```

することを意味する。

毎案件、新しいSkillファイルをAIが生成することを意味しない。


## 41.8 会話途中で新規Skillが必要になった場合

会話途中で、既存Skillでは満たせない再利用価値の高い専門ワークフローが必要と判明した場合:

```text
必要性を確認
↓
既存Skillで代替できない理由を記録
↓
Skillを作成
↓
正しいSkill配置先へ配置
↓
project.md に利用予定Skillを記録
↓
新しいGenie Codeチャットで継続
```

Genie Codeでは、Skillを新規作成・編集した場合、
現在のチャットがその変更を確実に再ロードすることを前提としない。

したがって、Skill変更後は新しいGenie Codeチャットを開始して継続する運用を標準とする。


## 41.9 Skill追加時のユーザーUX

ユーザーにSkillの詳細操作を要求しない。

悪い例:

> `.assistant/skills/` にこのファイルをコピーしてください。

良い例:

> 今回は既存のSkillでは不足する、再利用可能な需要予測評価ルールが必要です。
> Skillとして追加しました。変更を反映するため、新しいGenie Codeチャットでこのプロジェクトを続けてください。

必要な物理配置をAIが実行可能な場合はAIが行う。


## 41.10 Skill選択の可視化

Skillの内部ロードを逐一報告する必要はない。

ただし、開発方針に大きく影響するSkillを利用する場合は簡潔に説明してよい。

例:

> 本番化に入るため、Production Readiness のチェックを追加して進めます。

ユーザーにSkill名そのものを覚えさせることを目的としない。


## 41.11 Skill肥大化防止

Skill追加時にも Relevance Gate を適用する。

以下の質問に Yes と言えないなら、新しいSkillを作らない。

> このSkillが存在しない場合、合理的なAIの実装・レビュー・検証判断は実際に変わるか？

さらに以下を確認する。

- 複数案件で再利用可能か
- 既存Skillへ統合できないか
- Databricks公式能力と重複していないか
- 単なる案件固有情報ではないか


---

# 42. Starter ZIP と Skill の最終方針

v1のStarter ZIPは、Skillを大量同梱する「Skill Pack」にしない。

基本構成:

```text
ai-dev-databricks-starter/
├── AGENTS.md
├── README.md
├── docs/
│   └── project.md
└── guidance/
    ├── project-types/
    ├── presets/
    └── templates/
```

Genie Code正式Skillについては、

1. まず既存Workspace/User Skillを利用
2. Databricks公式・既存Skillを利用
3. Starterに同梱する場合も、再利用価値が明確な少数Skillに限定
4. 新規Skillは実案件で必要性が確認されてから追加

とする。


## 42.1 将来的な配布候補

実運用で繰り返し必要になることが確認できた場合のみ、
Starter ZIPにSkill sourceを追加してよい。

例:

```text
skill-sources/
└── organization-demand-forecasting/
    └── SKILL.md
```

ただし、`skill-sources/` に置いただけで有効になるとは扱わない。

Bootstrap時に、

- 既存Skill確認
- 必要性確認
- 正式配置
- 新チャットで反映

という手順を取る。


## 42.2 最終UX

利用者から見える理想形は変えない。

```text
ZIPを置く
↓
やりたいことを話す
↓
AIが既存環境と既存Skillを確認する
↓
必要なSkillだけ利用する
↓
不足があれば必要最小限だけ追加する
↓
必要なら新チャットへ引き継ぐ
↓
開発を続ける
```

ユーザーがSkill管理者になる必要はない。

---

# 43. Alignment and Solution Validation

本Blueprintでは、技術的に正しく動くことだけではなく、

> AIが理解した要求・選んだ解決方法・生成した成果物が、ユーザーの頭の中にある期待と一致しているか

を継続的に確認する。

この目的のため、以下の3つの考え方を導入する。

- Expectation Playback
- Solution Validation
- Alignment Check

ただし、これらを毎回独立した重い工程として実行してはならない。
Riskに応じて軽量化し、既存の UNDERSTAND / VERIFY / CONVERGE に統合して運用する。


## 43.1 Expectation Playback

AIがユーザーの要求を理解した後、その理解を業務言語で短く言い直す。

目的:

- 暗黙的な認識差を早期に発見する
- 技術用語を使わずにユーザーと認識を合わせる
- 実装前に大きな方向違いを防ぐ

例:

> 私の理解では、毎月の店舗別Excelを取り込み、重複を除き、部署別に集計して、
> 前月との差が分かる帳票を出力します。
> 同じ月を再実行した場合は、二重計上せず最新結果へ置き換える想定です。

重要:

- `project.md` の内容をそのまま読み上げるものではない
- ユーザーが理解できる業務言語へ翻訳する
- Small案件では省略してよい
- Standard案件では重要な認識点だけ確認
- High Risk案件では実装前に明示的なPlaybackを行う


## 43.2 Implementation Validation

実装が技術的に妥当かを確認する。

主な観点:

- コードが意図どおり動くか
- SQL結果が正しいか
- Technical Testが通るか
- Data Quality条件を満たすか
- エラー処理が機能するか
- 再実行時に壊れないか

これは主に VERIFY の責務である。


## 43.3 Solution Validation

技術的に動いていても、その解決方法自体が妥当とは限らない。

Solution Validationでは以下を確認する。

- この方法で本当にユーザーの目的を達成できるか
- より単純な方法で同じ目的を達成できないか
- 過剰設計になっていないか
- Databricks標準機能を適切に利用しているか
- 想定利用方法に対して運用可能か
- 出力粒度・形式が実利用に合っているか
- 必要以上に独自実装していないか

重要な原則:

```text
Implementation correctness
≠
Solution correctness
```

コードとして正しくても、問題解決として間違っている可能性がある。


## 43.4 Alignment Check

Alignment Checkは、以下の間にズレがないかを確認する。

```text
User Intent
↔
AI Understanding
↔
Specification
↔
Implementation
↔
Actual Output
↔
Real Usage
```

最低限の確認観点:

### Intent Alignment

作成物が、本来の目的に寄与しているか。

例:

ユーザーの目的:
> 発注量を決めたい

AIの成果物:
> 予測モデルの精度レポートだけ

であれば、技術的に正しくてもAlignment不足である。


### Input Alignment

AIが想定した入力と、実際に利用するデータが一致しているか。


### Process Alignment

業務上期待される処理と、実装ロジックが一致しているか。


### Output Alignment

以下がユーザー期待と一致しているか。

- 粒度
- 形式
- 項目
- 更新タイミング
- 見せ方


### Exception Alignment

異常時、再実行時、欠損時等の挙動がユーザー期待と一致しているか。


### Usage Alignment

実際の利用シーンで使えるか。

例:

「毎朝会議前に確認したい」

という要求に対し、
毎回Notebookを手動操作しなければ結果が得られないなら、
技術的には動いていても利用目的とのAlignmentが不足している。


## 43.5 Verify / Align / Converge の役割

概念を重複させないため、以下のように整理する。

### VERIFY

> 作ったものは技術的に正しく動くか？

対象:

- Test
- Runtime
- Data Quality
- Error handling


### ALIGN

> 作ったもの・選んだ解決方法は、ユーザーの期待と実利用に合っているか？

対象:

- Intent
- Usage
- Output
- Business behavior
- Solution appropriateness


### CONVERGE

> Spec / Acceptance Criteria / 実装 / ドキュメントの残差がなくなっているか？

対象:

- Missing
- Partial
- Contradiction
- Unrequested change
- Documentation drift

この3つを明確に分ける。


## 43.6 Risk別運用

### Small

原則:

```text
UNDERSTAND
→ BUILD
→ VERIFY
```

Alignment確認は会話の中で暗黙的に行う。
独立したAlignment工程は不要。


### Standard

原則:

```text
UNDERSTAND
→ EXPECTATION PLAYBACK
→ SPECIFY
→ BUILD
→ VERIFY
→ ALIGN
→ CONVERGE
```

ただし、EXPECTATION PLAYBACK と ALIGN は短く行う。


### Production / High Risk

原則:

```text
UNDERSTAND
→ CLARIFY
→ EXPECTATION PLAYBACK
→ SPECIFY
→ PLAN
→ CRITIQUE
→ BUILD
→ VERIFY
→ SOLUTION VALIDATION
→ ALIGN
→ CONVERGE
→ OPERATE
```

不可逆変更や重要KPI等では、重要なAlignment差分についてユーザー確認を行う。


## 43.7 project.md への反映

`project.md` に新しい巨大セクションを追加しない。

既存の以下のセクションを活用する。

- Purpose
- IPO+C
- Acceptance Criteria
- Business Scenarios
- Architecture Summary
- Verification

必要な場合のみ以下を追加してよい。

```markdown
## Alignment Notes

- User expectation:
- AI interpretation:
- Confirmed differences:
- Remaining uncertainty:
```

Alignment Notesはズレが実際に見つかった場合のみ使用する。


## 43.8 Alignment Gap の扱い

Alignment Checkでズレが見つかった場合:

```text
Minor wording / presentation gap
→ 修正して継続

Requirement interpretation gap
→ project.md / Acceptance Criteriaを修正

Architecture / solution gap
→ Solution Validationへ戻る

High Risk / irreversible gap
→ ユーザー確認
```

実装を仕様に合わせるだけでなく、
仕様自体がユーザー意図を誤っている場合は仕様側も修正する。


## 43.9 完了条件への統合

Global DoD に以下を追加する。

- 重要なユーザー期待と成果物のAlignmentを確認済み
- 実際の利用目的に反する既知のギャップがない
- Alignment未確認事項がある場合は明示済み


---

# 44. 批判的レビュー: Alignment機能追加後

Alignment関連の考え方は有効だが、そのまま追加すると以下の問題が起こり得る。


## 44.1 問題: 工程が増えすぎる

以下をすべて独立工程にすると重い。

```text
Expectation Playback
Implementation Validation
Solution Validation
Alignment Check
Verify
Converge
```

### 改善

これらを3つに統合する。

```text
VERIFY
ALIGN
CONVERGE
```

- Expectation Playback → UNDERSTAND / SPECIFY 内の会話技法
- Implementation Validation → VERIFY
- Solution Validation → ALIGN の一部
- Alignment Check → ALIGN
- Converge → 残差確認

つまり、最終的なユーザーに見せる工程名を増やさない。


## 44.2 問題: ユーザー確認が増えすぎる可能性

Alignmentを理由に毎回、

> この理解で合っていますか？

と聞くとUXが悪化する。

### 改善

Alignmentの大部分はAI自身が、

- 会話
- project.md
- 実装
- 出力
- 既存資産

を比較して確認する。

ユーザー確認は、

- 解釈によって結果が大きく変わる
- High Risk
- 不可逆
- 実利用の期待をAIだけで確認できない

場合に限定する。


## 44.3 問題: Alignmentが主観的になりやすい

「ユーザーの頭の中」は直接観測できない。

### 改善

Alignmentを推測だけで判断せず、以下の観測可能な根拠を優先する。

1. 明示されたPurpose
2. IPO+C
3. Business Scenario
4. Acceptance Criteria
5. ユーザーの具体例
6. 実際の利用手順
7. 生成された出力例
8. ユーザーフィードバック

「ユーザーはこう思っているはず」と断定しない。


## 44.4 問題: Specへの過信

Specと実装が一致していても、Spec自体が間違っている可能性がある。

### 改善

Convergeだけで完了にせず、ALIGNで、

```text
User Intent
↔
Spec
```

も確認する。

このためALIGNはConvergeより前に行う。


## 44.5 問題: 実装妥当性チェックがコードレビューと重複する

### 改善

コード品質の細部は既存Lint/Test/Reviewへ任せる。

BlueprintのSolution Validationでは、

- 解決方式
- 過剰設計
- Native vs Custom
- 運用可能性
- 実利用適合性

だけを見る。

低レベルなコードレビュー規約を自作しない。


## 44.6 問題: Alignment Notesが新たなドキュメント負債になる

### 改善

常設しない。

ズレが見つかった場合だけ記録する。
通常案件では `Purpose / IPO+C / Acceptance Criteria` の更新だけで済ませる。


## 44.7 問題: 「ALIGN」が新しい独自フレームワーク化する危険

### 改善

ALIGNは独自Engine・独自Skill・独自DSLにしない。

単なるチェック観点として保持する。

専用ツールは作らない。


---

# 45. レビュー反映後の最終開発フロー

Blueprint上の主要工程は、以下に整理する。

## Small

```text
UNDERSTAND
→ BUILD
→ VERIFY
```

## Standard

```text
UNDERSTAND
→ SPECIFY
→ BUILD
→ VERIFY
→ ALIGN
→ CONVERGE
```

## Production / High Risk

```text
UNDERSTAND
→ CLARIFY
→ SPECIFY
→ PLAN
→ CRITIQUE
→ BUILD
→ VERIFY
→ ALIGN
→ CONVERGE
→ OPERATE
```

Expectation Playbackは独立工程名にせず、
UNDERSTAND / SPECIFY の中で必要に応じて行う。

Solution Validationは独立工程名にせず、
ALIGNの一部として行う。


---

# 46. ALIGN の最小チェック

Standard以上では、最低限以下を確認する。

```text
1. Purpose
   成果物は本来の目的に寄与しているか

2. Behavior
   ユーザーが期待する動作になっているか

3. Output
   粒度・形式・タイミングが実利用に合うか

4. Solution
   過剰設計や不要な自作になっていないか

5. Usage
   実際の業務フローで利用可能か
```

これ以上の詳細チェックは、
案件・Project Type・Riskに応じて追加する。


---

# 47. UX上の最終原則

ユーザーから見た体験は複雑化させない。

内部的には、

```text
Verify
Align
Converge
```

を行っていても、ユーザーには例えば、

> 実装とテストは完了しています。
> さらに、当初の目的・想定していた使い方・出力形式とも一致していることを確認しました。
> 未確認なのは本番スケジュール実行だけです。

程度の説明でよい。

フレームワークの内部工程をユーザーに学ばせない。

