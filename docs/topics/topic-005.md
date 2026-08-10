# 005. 欠損値の扱い — `NA` → `NaN` / `None` / `pd.NA`

!!! abstract "この項目の R→Python 対応"
    - **R**: 欠損は `NA`（型別に `NA_real_` 等）、集計は既定で **NA を伝播**（`na.rm=TRUE` で無視）
    - **Python（推奨）**: 数値は `np.nan`、汎用は `None`、pandas の統一欠損は `pd.NA`。集計は既定で **欠損を無視**
    - **要注意**: ここが R↔Python で**最も事故る**差。「既定で伝播 or 無視」が逆

---

## R ではこう書く

```r
v <- c(1, 2, NA, 4)

mean(v)              # NA が伝播する
mean(v, na.rm = TRUE)# 無視して計算
is.na(v)             # 欠損位置
sum(is.na(v))        # 欠損数
NA == 1              # 比較しても NA
```

出力:

```text
[1] NA
[1] 2.333333
[1] FALSE FALSE  TRUE FALSE
[1] 1
[1] NA
```

!!! note "R の欠損まわりの語彙"
    - `NA`：値が欠けている（長さ1の欠損）。型別に `NA_integer_` `NA_real_` `NA_character_`。
    - `NULL`：そもそも存在しない（長さ0）。`NA` とは別物。
    - `NaN`：非数（`0/0`）。`is.na(NaN)` は `TRUE`。
    - 判定は必ず `is.na(x)`。**`x == NA` は使えない**（結果が `NA` になる）。

---

## Python ではこう書く

Python では欠損が**文脈で複数の型**になります。まず対応を押さえます。

| R | Python | 備考 |
|---|---|---|
| `NA`（数値） | `np.nan`（float） | NumPy/pandas の数値欠損の実体 |
| `NA`（一般） | `None` | Python の「値なし」。オブジェクト列に入る |
| `NULL` | `None` | 引数の未指定など |
| `NaN` | `np.nan` | 同じ非数 |
| （新） | `pd.NA` | pandas の型に依らない統一欠損 |

=== "pandas"

    ```python
    import numpy as np
    import pandas as pd

    v = pd.Series([1, 2, np.nan, 4])

    print(v.mean())        # 既定で欠損を無視（skipna=True）
    print(v.sum())
    print(v.isna().tolist())
    print(v.isna().sum())
    print(np.nan == 1)     # 比較は False（NA ではない）
    ```

    出力:

    ```text
    2.3333333333333335
    7.0
    [False, False, True, False]
    1
    False
    ```

=== "polars"

    ```python
    import polars as pl

    s = pl.Series([1, 2, None, 4])
    print(s.mean())          # 欠損を無視
    print(s.is_null().sum()) # 欠損数
    print(s.null_count())
    ```

    polars は欠損を **`None`（null）** で一貫して扱い、`NaN`（非数）とは**区別**します。`is_null()` と `is_nan()` が別物である点は pandas より厳密です。

!!! tip "実務ではこれ"
    - 欠損判定は **`isna()` / `is_null()`** を使う。`x == np.nan` は**常に False**で、R の `x == NA`（→ NA）とも挙動が違う。
    - 「NA を除いて計算」は pandas では**既定でそうなっている**ので `na.rm=TRUE` 相当は不要。逆に**含めたい**なら `skipna=False`。

---

## `na.rm=TRUE` の対応（既定が逆なので要注意）

| 計算 | R | pandas |
|---|---|---|
| 欠損を無視して平均 | `mean(v, na.rm = TRUE)` | `v.mean()`（既定） |
| 欠損があれば NA/NaN | `mean(v)`（既定） | `v.mean(skipna=False)` |
| 欠損数 | `sum(is.na(v))` | `v.isna().sum()` |
| 行の欠損を落とす | `na.omit(df)` / `drop_na()` | `df.dropna()` |
| 欠損を埋める | `coalesce()` / `replace_na()` | `df.fillna(...)` |

---

## つまずきポイント

!!! warning "R と Python の差"
    - **既定が逆**：R の集計は NA を**伝播**、pandas は**無視**。SAS/R と数値を突き合わせるときは、pandas 側で欠損の扱いを**明示**（`skipna=`）して意図を固定する。
    - **整数列に欠損 → float 化**：pandas の素の整数列は欠損を持てず `float64` に格上げされる。整数を保ちたいなら nullable 型 `astype("Int64")`（大文字 I）。
    - **`None` と `np.nan` の混在**：数値列に `None` を入れると `np.nan` に変換されるが、オブジェクト列では `None` のまま残る。判定は型を問わない `isna()` で。
    - **polars は `null` と `NaN` を区別**：欠損は `null`、`0/0` は `NaN`。`drop_nulls()` と `fill_nan()` を取り違えない。
    - **文字列の "NA"**：CSV 読み込みで `"NA"` `""` が欠損になるか否かは `na_values` / `null_values` 次第。R の `read_csv` と既定が違うことがある。

## 関連項目

- [003. データフレーム入門](topic-003.md)
- [035. 欠損の補完（coalesce / replace_na / fill）](../roadmap.md)
- [056. 欠損数の集計](../roadmap.md)
