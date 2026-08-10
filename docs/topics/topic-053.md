# 053. カテゴリの頻度と割合 — `table` / `prop.table` → `value_counts` / `crosstab`

!!! abstract "この項目の R→Python 対応"
    - **R**: `table()`（度数）／`prop.table()`（割合）／`addmargins()`（合計）
    - **Python（推奨）**: pandas **`value_counts()`**（度数）／**`value_counts(normalize=True)`**（割合）
    - **要注意**: R の `prop.table` は「全体に対する割合」。行%・列%は `margin=` 指定。pandas は crosstab の `normalize=`

（sex = M,F,M,F,M,M,F,M,F,F → M:5, F:5）

---

## R ではこう書く

```r
table(dm3$sex)                        # 度数
prop.table(table(dm3$sex))            # 割合（全体）
addmargins(table(dm3$sex))            # 合計付き
```

出力:

```text
F M
5 5

  F   M
0.5 0.5
```

!!! note "R の勘所"
    - `table()` は度数、`prop.table()` は割合（既定は全体に対して）。
    - `table()` は **NA を既定で除外**。含めるなら `table(x, useNA = "ifany")`。
    - 2 変数のクロスは `table(a, b)`（→ [054](topic-054.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm3["sex"].value_counts()                 # 度数（件数降順）
    dm3["sex"].value_counts(normalize=True)   # 割合
    dm3["sex"].value_counts().sort_index()    # キー順に並べ替え

    # 度数と割合を1表に
    vc = dm3["sex"].value_counts()
    pd.DataFrame({"n": vc, "pct": (vc / vc.sum() * 100).round(1)})
    ```

    出力（`value_counts`）:

    ```text
    {'M': 5, 'F': 5}
    {'M': 0.5, 'F': 0.5}
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p["sex"].value_counts()                      # sex, count
    dm3p["sex"].value_counts(normalize=True)        # 割合列
    ```

!!! tip "実務ではこれ"
    - 度数は `value_counts()`、割合は `normalize=True`。**並びは件数降順**なので、カテゴリ順に見せるなら `sort_index()`。
    - **"n (%)" 表**は度数と割合を数値で作り、最後に f-string 化（→ [057](topic-057.md)）。
    - 欠損も数えるなら `value_counts(dropna=False)`（R の `useNA="ifany"` 相当）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 度数 | `table(x)` | `x.value_counts()` | `x.value_counts()` |
| 割合 | `prop.table(table(x))` | `value_counts(normalize=True)` | `value_counts(normalize=True)` |
| 合計付き | `addmargins()` | `vc.sum()` を追加 | — |
| キー順 | `table` は水準順 | `value_counts().sort_index()` | `.sort("...")` |
| 欠損も | `useNA="ifany"` | `value_counts(dropna=False)` | 既定で含む |

## つまずきポイント

!!! warning "R と Python の差"
    - **並び順**：`table` は水準（因子）順、`value_counts` は件数降順。TFL のカテゴリ順（No/Yes、Grade 1/2/3…）を出すなら Categorical＋`sort_index`。
    - **欠損の既定**：`table` も `value_counts` も既定で NA を除外。含めるなら明示。
    - **割合の分母**：`normalize=True` は全体が分母。群内%が欲しいなら crosstab の `normalize="index"`（→ [054](topic-054.md)）。
    - **factor の全水準**：データに現れない水準を 0 で出すには Categorical にして `value_counts()`（現れない水準も 0 で残る）。

## 関連項目

- [022. 件数と頻度](topic-022.md)
- [054. クロス集計表](topic-054.md)
- [057. n (%) 整形のパターン](topic-057.md)
