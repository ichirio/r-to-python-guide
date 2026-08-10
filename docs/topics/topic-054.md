# 054. クロス集計表 — `xtabs` / `table` → `crosstab` / `pivot_table`

!!! abstract "この項目の R→Python 対応"
    - **R**: `table(a, b)` / `xtabs(~ a + b)`（度数）／`prop.table(, margin=)`（行%・列%）
    - **Python（推奨）**: pandas **`crosstab(a, b)`**（度数・割合・合計を一手に）
    - **要注意**: 行%・列%の指定方向。R の `margin=1` は行、pandas の `normalize="index"` は行

2 変数の分割表。sex × arm の人数、行%・列%、合計。

---

## R ではこう書く

```r
table(dm3$sex, dm3$arm)                        # 度数
xtabs(~ sex + arm, data = dm3)                 # 式で書く版
prop.table(table(dm3$sex, dm3$arm), margin = 1)# 行%（各行が100%）
addmargins(table(dm3$sex, dm3$arm))            # 行・列の合計
```

出力（`table(sex, arm)`）:

```text
    A B
  F 2 3
  M 3 2
```

!!! note "R の勘所"
    - `table(a, b)`：a が行、b が列。`xtabs(~a+b)` は同じ結果を式で。
    - `prop.table(, margin=1)`＝行%、`margin=2`＝列%、省略＝全体%。
    - 3 変数以上は `ftable()` で見やすく。

---

## Python ではこう書く

=== "pandas"

    ```python
    pd.crosstab(dm3["sex"], dm3["arm"])                       # 度数
    pd.crosstab(dm3["sex"], dm3["arm"], margins=True)         # 合計付き
    pd.crosstab(dm3["sex"], dm3["arm"], normalize="index")    # 行%
    pd.crosstab(dm3["sex"], dm3["arm"], normalize="columns")  # 列%

    # 値を集計するクロス（度数でなく平均など）
    pd.crosstab(dm3["sex"], dm3["arm"], values=dm3["age"], aggfunc="mean")
    ```

    出力（`crosstab(sex, arm)`）:

    ```text
    arm  A  B
    sex
    F    2  3
    M    3  2
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p.pivot(on="arm", index="sex", values="subjid", aggregate_function="len")
    ```

!!! tip "実務ではこれ"
    - **度数のクロス表** → `pd.crosstab(row, col)`。合計は `margins=True`、割合は `normalize=`。
    - **セルに平均などを入れる**（連続値のクロス）→ `crosstab(..., values=, aggfunc=)` か `pivot_table`。
    - カテゴリ順は Categorical で固定（TFL の群順・水準順）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 度数クロス | `table(a, b)` / `xtabs(~a+b)` | `crosstab(a, b)` |
| 合計付き | `addmargins()` | `crosstab(..., margins=True)` |
| 行% | `prop.table(, margin=1)` | `crosstab(..., normalize="index")` |
| 列% | `prop.table(, margin=2)` | `crosstab(..., normalize="columns")` |
| 全体% | `prop.table()` | `crosstab(..., normalize="all")` |
| 値の集計 | `tapply` / `xtabs(v~a+b)` | `crosstab(..., values=, aggfunc=)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **%の方向**：R `margin=1`＝行%、`margin=2`＝列%。pandas は `normalize="index"`＝行%、`"columns"`＝列%。**「行=各行が100%」を取り違えない**。
    - **欠損**：`crosstab` は既定で NaN を除外。含めるなら `dropna=False`。
    - **カテゴリ順**：`crosstab` はキーをソート。TFL の群順は Categorical 水準で固定。
    - **χ² 検定へ**：クロス表を `scipy.stats.chi2_contingency` に渡す（→ [061](../roadmap.md)）。`crosstab` の結果をそのまま使える。

## 関連項目

- [053. カテゴリの頻度と割合](topic-053.md)
- [037. 横持ち化（pivot_wider）](topic-037.md)
- [061. カイ二乗・Fisher](../roadmap.md)
