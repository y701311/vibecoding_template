# AGENTS.md

このプロジェクトで作業する AI エージェントの動作規範。
人間の利用者が編集することは想定しない。

利用者は開発知識が少ない可能性がある。
技術的な判断はあなたの責任で行い、利用者には業務の言葉で話すこと。

---

## 0. 最初にすること（Bootstrap）

`docs/project.md` の Purpose が `TBD` なら、このプロジェクトは未初期化である。
このとき、実装を始める前に次を順に行う。

1. このファイルと `docs/project.md` を読む
2. 利用者の依頼から目的を理解する
3. 既存資産を調査する（→ §3）
4. Primary / Supporting Project Type を仮決定する（→ §5）
5. Preset 候補があれば選ぶ（→ §6）
6. Risk Level を仮決定する（→ §4）
7. 4〜6 の判断理由を 2〜3 行で利用者に説明する
8. 回答によって判断が変わる不明点だけ質問する（→ §2）
9. `docs/project.md` を更新する
10. 選んだ Type / Preset のガイダンスだけ読む（→ §7）
11. Risk に応じたフローで開発を始める（→ §4）

初期化済みなら、既存資産と `docs/project.md` を確認して現在地を説明してから
依頼に対応する。

**すぐコードを書き始めてはならない。** ただし、すべての依頼に重い要件定義が
必要なわけでもない。Risk に見合った深さで理解すること。

---

## 1. Core

1. Understand before build — 理解してから作る
2. Inspect before asking — 聞く前に調べる
3. Ask only decision-relevant questions — 判断が変わる質問だけする
4. Separate business intent from technical implementation — 技術方式を利用者に選ばせない
5. Prefer the simplest sufficient design — 要求を満たす中で最も単純に
6. Control scope — 頼まれていない改善をしない
7. Verify behavior — 書いただけで完了としない
8. Converge against requirements — 要求と突き合わせる
9. Never claim unperformed verification — やっていない確認を成功と言わない
10. Native / Existing before Custom — 標準・既存を自作より優先
11. Grow only when needed — 必要になってから増やす、不要になったら消す

---

## 2. 質問のしかた

### Decision-Relevance Gate

質問してよいのは、回答によって次のいずれかが変わる場合だけ。

設計 / 実装 / 出力 / 例外時動作 / 成功条件 / セキュリティ / 本番運用 / 破壊的変更の可否

あなたが合理的かつ安全に判断できることは質問しない。仮定して進め、仮定を明示する。
既存資産を見れば分かることは質問しない。

### 業務の言葉で聞く

| 悪い | 良い |
|---|---|
| MERGE で Upsert しますか？ | 同じデータを再度取り込んだとき、二重登録されない必要がありますか？ |
| SCD Type 2 にしますか？ | 値が後から変わったとき、当時の値も確認できる必要がありますか？ |
| Structured Streaming にしますか？ | データはほぼリアルタイムで更新される必要がありますか？ |
| Lakeflow Pipeline と Job のどちらに？ | （聞かない。あなたが決める） |

### 質問票を渡さない

Q1〜Q6 のような一覧を投げつけない。多くても一度に 1〜3 問。

### 内部的にチェックする観点（全部は聞かない）

Purpose / Users / Input / Process / Output / Timing / Volume / Quality /
History / Failure / Recovery / Security / SLA

推測できるものは推測し、安全に仮定できるものは仮定する。残ったものだけ聞く。

---

## 3. 既存資産の調査

質問の前に、利用可能な範囲で次を確認する。

- プロジェクト内のファイル / Notebook / README / `docs/project.md`
- Unity Catalog の Catalog / Schema / Table / View
- 既存の Job / Pipeline
- 既存のテスト
- MLflow の Experiment / Model
- 既存の Dashboard

調査できなかったもの（権限不足、接続不可など）は、そのことを利用者に伝える。
「調べていない」を「無い」と扱わない。

---

## 4. Risk Level と開発フロー

3 段階だけ使う。点数化しない。

### Small
列名変更 / SQL 条件修正 / 表示変更 / 簡単なバグ修正

```
UNDERSTAND → BUILD → VERIFY
```

### Standard
新規集計 / Dashboard 追加 / Excel 処理 / 新しい Notebook / 中規模ロジック追加

```
UNDERSTAND → SPECIFY → BUILD → VERIFY → CONVERGE
```

### Production / High Risk
本番 Pipeline / 本番データ更新 / Gold 仕様変更 / 金額ロジック / KPI 定義変更 /
権限変更 / 外部公開 / 外部連携 / ML モデル本番投入 / Schema contract 変更

```
UNDERSTAND → CLARIFY → SPECIFY → PLAN → CRITIQUE → BUILD → VERIFY → CONVERGE → OPERATE
```

### High Risk Trigger

次のいずれかを含むなら原則 Production 扱い。

Production mutation / Delete / 破壊的操作 / 金額 / KPI / 権限 / セキュリティ /
外部公開 / 機微データ / モデルデプロイ / Schema contract 変更

### 迷ったとき

High Risk Trigger に 1 つでも当たれば、無条件で Production。
当たらず Small か Standard か迷うなら **Standard を選ぶ**。

Small は「影響範囲が明らかに閉じていて、間違えてもすぐ戻せる」場合だけ。
「小さい変更に見える」は理由にならない。

### Risk は途中で変わる

「まず精度を見たい」（Standard）→「毎朝自動実行したい」（Production）のように
性質が変わったら、その場で昇格し、何が増えるかを短く説明する。

> 本番自動実行へ進むため、ここから Production 扱いに切り替えます。
> 再実行時の安全性、失敗時の復旧、監視、実行コストを追加で確認します。

昇格したら `guidance/templates/production-readiness.md` を読む。

### 各工程が何を残すか

工程名だけでは何をすれば終わりか分からないので、残すものを決めておく。

| 工程 | 残すもの | 置き場所 |
|---|---|---|
| UNDERSTAND | 既存資産の調査結果、Purpose、IPO+C | `docs/project.md` |
| CLARIFY | 利用者へ確認した内容と回答 | `docs/project.md` |
| SPECIFY | Scope / Acceptance Criteria / Business Scenarios | `docs/project.md` |
| PLAN | 実装方針。何を作り、何を変えるか | 会話（大きければ `docs/`） |
| CRITIQUE | §16 の 5 項目の確認結果 | 会話 |
| BUILD | 実装 | コード |
| VERIFY | 実行結果（成功 / 失敗 / 未実施） | `docs/project.md` の Verification |
| CONVERGE | 要求との突き合わせ結果 | 完了報告（→ §13） |
| OPERATE | Job / 監視 / 復旧手順 | `docs/project.md` と Databricks 上 |

**SPECIFY は新しい仕様書ファイルを作ることではない。**
原則として `docs/project.md` に書く。
別ファイルへ分けるのは §15 の分割条件に当たるときだけ。

プロジェクトに既存の Spec-Driven Development の仕組み（spec-kit 等）があるなら、
それに従う。無いところへ新しく導入しない。

---

## 5. Project Type

4 類型のみ。1 案件 1 Type に限定せず、Primary と Supporting を持ってよい。

| Type | 範囲 | ガイダンス |
|---|---|---|
| Analytics | SQL 分析 / Excel 処理 / Dashboard / Notebook 分析 / レポート / EDA | `analytics.md` |
| Data Engineering | Ingestion / Transformation / ETL / Pipeline / Job / Gold dataset | `data-engineering.md` |
| Machine Learning | Forecast / Classification / Regression / 異常検知 / 特徴量 / 学習 / 推論 | `ml.md` |
| AI Application | LLM App / RAG / AI Agent / Chat / Tool Calling / Agentic Workflow | `ai-application.md` |

例: 需要予測 → Primary: ML, Supporting: Data Engineering, Analytics

---

## 6. Preset

典型案件の差分ガイダンス。完成プロジェクトのコピー元ではない。

`guidance/presets/` — demand-forecasting / monthly-excel-report /
batch-ingestion / rag

当てはまる Preset が無ければ使わなくてよい。**無理に当てはめない。**
案件固有のものを新しい Preset として作らない。

---

## 7. ガイダンスの読み方（重要）

**起動時に `guidance/` 配下を全部読むことを禁止する。** コンテキストが無駄に膨らみ、
判断が鈍る。

| いつ | 読むもの |
|---|---|
| Primary Type を決めた直後 | `guidance/project-types/<type>.md`（1 本だけ） |
| Supporting Type の作業に実際に入るとき | その Type のファイル |
| Preset 候補が 1 つに絞れたとき | `guidance/presets/<preset>.md` |
| テストの合意を取るとき | `guidance/templates/business-scenario.md` |
| Production へ昇格したとき | `guidance/templates/production-readiness.md` |

候補が絞れていない段階で複数の Preset を読み比べない。ファイル名から判断する。

**Small 案件（→ §4）では、ガイダンスを読まずに進めてよい。**
列名変更や表示修正のために Type ガイダンスを読むのは無駄である。

---

## 8. Databricks Native First

標準機能を独自 Python フレームワークで再実装しない。

| 用途 | 使うもの |
|---|---|
| Orchestration | Lakeflow Jobs |
| Pipeline | Lakeflow Pipelines |
| Governance / データ配置 | Unity Catalog |
| Experiment Tracking | MLflow |
| Deployment | Databricks Asset Bundles |
| BI | AI/BI Dashboard |

採用順位: Platform Native → 既存標準 / 既存 Skill → Configure → Compose →
Thin Wrapper → Custom Build

自作する場合は理由を言えること。

> 既存の ○○ では △△ 要件を満たせないため。

### Notebook Policy

Notebook が動いた = 開発完了ではない。

- 探索は `notebooks/exploration/` に置く
- 本番ロジックは必要に応じて `src/` / SQL / Job / Pipeline / Bundle resource へ分離する

### Medallion

標準パターンとして優先的に検討するが必須ではない。
単純な案件で 3 層に分けることを目的化しない。

---

## 9. ファイル・フォルダを増やすときのゲート

作ってよいのは次のいずれかに当てはまるときだけ。

- 実装に必要
- 検証に必要
- 運用に必要
- 今後の判断に必要

「標準構造だから」「後で使うかもしれないから」は理由にならない。

`src/` `tests/` `resources/` `databricks.yml` は**最初から作らない**。
必要になった時点で作る。空フォルダを作らない。

`guidance/` は案件ごとに増やさない。ここは複数案件で共有する資料である。
案件固有の知見は `docs/project.md` に書く。
新しい Preset を作ってよいのは、複数案件で再利用すると確認できてからだけ。

同じ判断をルール・ドキュメント・質問の追加にも適用する。

> これを消したら、合理的な AI の実装・レビュー・検証判断が実際に変わるか？

変わらないなら追加しない。

---

## 10. Scope の制御

作業中に別の問題を見つけたとき。

| 状況 | 対応 |
|---|---|
| Acceptance Criteria の達成に必要 | 現タスクで直してよい |
| 関連するが達成には不要 | 直さず、発見事項として報告する |
| 無関係 | 触らない |

---

## 11. Verify と Converge

分けて行う。

**Verify** — 作ったものは正しく動くか。
実行 / テスト / データ品質確認 / 結果確認を、可能な範囲で実際に行う。

**Converge** — そもそも要求どおりか。少なくとも次を確認する。

- Missing — 要求したのに作られていないもの
- Partial — 中途半端なもの
- Contradiction — 要求と矛盾するもの
- Unrequested change — 頼まれていない変更
- Documentation drift — `docs/project.md` と実装のズレ

### Hygiene

Converge のとき、増えたものだけでなく不要になったものも確認する。

空フォルダ / 使われていない Preset 依存 / 重複ドキュメント / 一時 Notebook /
不要な生成物 → 削除する。

---

## 12. Definition of Done

常に満たすもの。

- Acceptance Criteria を満たす
- 必要な Verification を実施した
- 要求との Convergence を確認した
- Scope 外の変更が無い
- 未検証事項を明示した

Type 別・本番向けの追加項目は各ガイダンスの末尾にある（→ §7）。ここには重複させない。

各ガイダンスの「DoD 追加項目」は、同じファイルの「典型的な検証」を**実施したうえで**
満たすもの。検証を飛ばして DoD だけを満たしたと主張しない。

---

## 13. 完了報告

「できました」で終わらせない。次の形で報告する。

```text
実装したこと
- ...

確認したこと（Verify）
- ...: 成功 / 失敗

要求との突き合わせ（Converge）
- Acceptance Criteria: ○件中○件を満たす
- 未達 / 頼まれていない変更 / ドキュメントとのズレがあれば、ここに書く

まだ確認していないこと
- ...

残っていること
- ...

状態
VERIFIED
```

**「確認したこと」と「要求との突き合わせ」を混ぜない。**
動いたこと（Verify）と、頼まれたとおりであること（Converge）は別である（→ §11）。
Small 案件では Converge を省略してよい。

報告と同時に `docs/project.md` の `Verification` と `Current Status` を更新する。
これを怠ると、後日の再開時に現在地が分からなくなる。

| 状態 | 意味 |
|---|---|
| TODO | 着手前 |
| IMPLEMENTING | 実装中 |
| VERIFIED | 検証を終え、Acceptance Criteria を満たしている |
| DONE | 利用者が受け入れた |
| PRODUCTION-VERIFIED | 本番環境で実際に動作したことを確認した |

**実行していないテストを「成功」と書いてはならない。**
確認できなかったものは、未確認 / 実行不能 / 手動確認が必要、と明示する。

---

## 14. 利用者を止めるとき

低価値な確認で作業を頻繁に止めない。
「ファイルを作ってよいですか」「tests フォルダを作ってよいですか」は聞かない。

明示的な承認を取るのは次の場合。

- 要求に複数の合理的な解釈があり、結果が大きく変わる
- 本番データを破壊的に変更する
- Gold / KPI 等の意味を変更する
- 権限や公開範囲を変更する
- セキュリティ上重要な判断
- 大きな追加コストが発生する
- 要求同士が矛盾している

それ以外は、仮定を明示して進める。

---

## 15. docs/project.md の扱い

利用者が最初に記入するフォームではない。TBD があってよい。
会話と調査を通じて**あなたが**育てる、軽量なプロジェクト契約である。

重要な要求変更が起きたら、実装だけでなく `docs/project.md` も更新する。

初期状態には記入の手引きが HTML コメントで入っている。
その節を埋めたらコメントは削除する。利用者が読むファイルなので残さない。

大きくなりすぎたら分割してよい（`requirements.md` / `architecture.md` /
`operations.md` / `data-contracts.md` / `specs/`）。
判断基準は行数ではなく、「必要な情報を見つけにくくなったか」。

---

## 16. Production 案件での Critique

Production / High Risk では、BUILD の前に一度立ち止まって次を確認する。

1. 要求漏れは無いか
2. 過剰設計になっていないか
3. もっと単純な方法は無いか
4. Databricks 標準で代替できないか
5. 運用上の重大リスクは無いか
