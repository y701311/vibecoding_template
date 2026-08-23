# Skill の使い方

Genie Code の正式な Skill を、使う・作る・変更するときに読む。
既存の Skill をそのまま使うだけなら、読む必要はない。

---

## guidance と Skill は別物

| | 中身 | 置き場所 |
|---|---|---|
| `guidance/` | この種類の案件で**何を考慮すべきか** | このプロジェクト内 |
| Skill | この専門タスクを**どの手順・基準で実行するか** | Genie Code の Skill 配置先 |

`guidance/presets/demand-forecasting.md` と
`demand-forecasting` という Skill は、名前が同じでも責務が違う。

前者は「需要予測案件で何を確認すべきか」、
後者は「時系列分割・backtest・Baseline 比較をどう実行するか」である。

**`guidance/` にファイルを置いても Skill にはならない。**

---

## 使う前に確認する順序

1. Genie Code の標準能力で足りるか
2. Databricks 公式 / 既存の Skill があるか
3. Workspace / User Skill として既に存在するか
4. 既存 Skill の設定・組み合わせで足りるか
5. 本当に必要な場合だけ新規作成

AGENTS.md §8 の Native / Existing before Custom を Skill に適用したものである。
**ほとんどの案件は 1〜3 で終わる。**

---

## 作らない Skill

Genie Code と Databricks が既に理解している一般操作を、説明し直すだけのものは作らない。

`unity-catalog-basic` / `notebook-basic` / `spark-basic` / `sql-basic` /
`mlflow-basic` / `jobs-basic` / `pipeline-basic`

いずれも不要である。既存能力で不足する具体的な理由を言えない限り作らない。

---

## 作ってよい Skill

複数案件で再利用でき、AI の判断・実装・検証が実際に変わるものだけ。

- 組織固有のデータ品質基準
- 組織固有の評価基準（例: 需要予測の合否ライン）
- 特定業務の会計・集計ルール
- 組織固有の本番化チェック

**案件固有の一度きりの情報を Skill にしない。** それは `docs/project.md` に書く。

作る前に、これに Yes と言えるか確認する。

> この Skill が無い場合、合理的な AI の実装・レビュー・検証判断は実際に変わるか？

加えて次を確認する。

- 複数案件で再利用できるか
- 既存の Skill へ統合できないか
- Databricks 公式の能力と重複していないか
- 単なる案件固有情報ではないか

---

## Skill を追加・変更したとき

**Genie Code は、実行中のチャットが Skill の変更を再読み込みすることを保証しない。**

そのため次の順で行う。

1. 既存 Skill で代替できない理由を記録する
2. Skill を作成し、正しい配置先へ置く
3. `docs/project.md` の Selected Context に、利用する Skill を記録する
4. 利用者へ、新しいチャットで続けるよう伝える

配置先は Workspace 共有か個人用かで異なる。**Databricks の現行仕様に従うこと。**
このドキュメントで配置先を独自に決めない。

### 利用者に配置作業をさせない

悪い例。

> `.assistant/skills/` にこのファイルをコピーしてください。

良い例。

> 今回は既存の Skill では不足する、再利用可能な需要予測の評価ルールが必要でした。
> Skill として追加しています。反映のため、新しいチャットでこのプロジェクトを
> 続けてください。ここまでの経緯は `docs/project.md` に残してあります。

AI が実行できる配置は AI が行う。

---

## 利用者に見せること

Skill のロードを逐一報告しない。Skill 名を覚えさせない。

開発方針が大きく変わるときだけ、業務の言葉で一言添える。

> 本番化に入るため、Production Readiness のチェックを追加して進めます。

利用者を Skill の管理者にしない。
