# 069. シフトテーブル

!!! abstract "この項目の R→Python 対応"
    - **R**: ベースラインと後観測のカテゴリを `table()` / `count()` でクロス（factor 水準を固定）
    - **Python（推奨）**: **`pd.crosstab(baseline, post)`**（Categorical で水準順を固定）
    - **要注意**: 出現しない水準（0 セル）も出すため、**Categorical で全水準を宣言**する

「ベースライン Normal → 後観測 High」のような**状態遷移の人数表**。検査値グレードのシフトで多用。

（Normal / High の2水準）

---

## R ではこう書く

```r
lab$base <- factor(lab$base, levels = c("Normal","High"))
lab$post <- factor(lab$post, levels = c("Normal","High"))

table(lab$base, lab$post)                 # 行=baseline, 列=post
# addmargins() で合計、prop.table(margin=1) で行%（baseline 内%）
```

!!! note "R の勘所"
    - **factor の水準**を固定しないと、出現しないグレードが表から落ちる。
    - 行=baseline、列=post に取り、対角が「不変」、非対角が「悪化/改善」。
    - 行%（`prop.table(margin=1)`）で「baseline ○○のうち後に△△へ」を出す。

---

## Python ではこう書く

=== "pandas"

    ```python
    order = ["Normal", "High"]
    shift = pd.crosstab(
        pd.Categorical(lab["base"], order),
        pd.Categorical(lab["post"], order),
        rownames=["baseline"], colnames=["post"],
    )
    print(shift)

    # 行%（baseline 内）
    # pd.crosstab(..., normalize="index")
    ```

    出力:

    ```text
    post      Normal  High
    baseline
    Normal         1     2
    High           1     1
    ```

=== "polars"

    ```python
    import polars as pl
    (labp.group_by(["base","post"]).len()
         .pivot(on="post", index="base", values="len").fill_null(0))
    ```

!!! tip "実務ではこれ"
    - **`pd.crosstab(base, post)`**。水準順・0 セルを出すため **`pd.Categorical(x, order)`** で全水準を宣言。
    - 合計は `margins=True`、baseline 内%は `normalize="index"`。
    - グレードが多い（G0〜G4）ときも同じ。順序付き Categorical にしておけば並びが揃う。
    - 群別シフトは群でループ、または `groupby(arm)` してから各群でクロス。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| シフト表 | `table(base, post)` | `pd.crosstab(base, post)` |
| 水準固定 | `factor(x, levels=)` | `pd.Categorical(x, order)` |
| 合計 | `addmargins()` | `crosstab(..., margins=True)` |
| 行%（baseline 内） | `prop.table(margin=1)` | `crosstab(..., normalize="index")` |
| 0 セルも出す | factor 水準 | Categorical 水準 |
| 群別 | `by(arm)` | `groupby("arm")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **0 セルの欠落**：出現しないグレードの組は、水準を宣言しないと表から消える。R は factor、pandas は Categorical で全水準を持たせる。
    - **順序**：グレードは順序付き（G0<G1<…）。`Categorical(ordered=True)` で並びを固定（→ [091](../roadmap.md)）。
    - **欠損の扱い**：ベースラインか後観測が欠測の被験者をどう扱うか（別カテゴリ「Missing」を立てる等）を仕様で決める。
    - **行%の方向**：baseline 内%は `normalize="index"`（各行が 100%）。列%と取り違えない（→ [054](topic-054.md)）。

## 関連項目

- [054. クロス集計表](topic-054.md)
- [046. 欠けた組合せを補完](topic-046.md)
- [091. 因子とラベル](../roadmap.md)
