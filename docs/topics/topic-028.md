# 028. 型変換 — `as.numeric` / `as.character` → `astype` / `to_numeric`

!!! abstract "この項目の R→Python 対応"
    - **R**: `as.numeric()` / `as.integer()` / `as.character()` / `as.factor()`（変換不能は `NA`＋警告）
    - **Python（推奨）**: 一括は **`astype()`**、失敗を欠損にするなら **`pd.to_numeric(errors="coerce")`**
    - **要注意**: `astype(int)` は**欠損があると失敗**。欠損を保ちたいなら nullable な `"Int64"`

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
as.numeric(c("1", "2", "x"))   # "x" は NA（警告付き）
as.character(c(1L, 2L))        # "1" "2"
as.integer("3")                # 3
as.factor(c("A", "B", "A"))    # 水準 A, B
```

出力:

```text
[1]  1  2 NA
Warning: NAs introduced by coercion
[1] "1" "2"
```

!!! note "R の勘所"
    - `as.numeric()`：変換できない文字は **NA＋警告**。黙って欠損化するので警告は見逃さない。
    - factor↔数値は要注意：`as.numeric(factor)` は**水準番号**を返す。値そのものが欲しければ `as.numeric(as.character(f))`。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd

    # 文字→数値。変換不能を欠損にする（as.numeric 相当）
    pd.to_numeric(pd.Series(["1", "2", "x"]), errors="coerce")  # [1.0, 2.0, NaN]

    # 一括の型指定
    dm["age"].astype("Int64")     # nullable 整数（欠損を保持できる）
    dm["subjid"].astype(str)      # 文字列に
    dm["arm"].astype("category")  # factor 相当

    # 失敗時に例外にしたい（既定）
    # pd.to_numeric(s)            # "x" があると ValueError
    ```

    出力（`to_numeric(..., "coerce")`）:

    ```text
    [1.0, 2.0, nan]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series(["1", "2", "x"])
    print(s.cast(pl.Int64, strict=False).to_list())   # [1, 2, None]（strict=False で欠損化）
    dmp.with_columns(pl.col("subjid").cast(pl.Utf8))
    ```

!!! tip "実務ではこれ"
    - **文字→数値で汚れたデータ**（`"NA"`, `""`, `"x"` 混在）は `pd.to_numeric(s, errors="coerce")`。`as.numeric` と同じく変換不能を NaN に。
    - **整数で欠損を保ちたい**なら `astype("Int64")`（大文字 I の nullable 型）。素の `int` は欠損不可。
    - **factor 相当**は `astype("category")`。水準の順序を付けるなら `pd.Categorical(..., ordered=True)`（→ [091](../roadmap.md)）。
    - polars は `cast(dtype, strict=False)` で「変換できないものは null」。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 文字→数値（失敗は欠損） | `as.numeric(x)` | `pd.to_numeric(x, errors="coerce")` | `cast(Float64, strict=False)` |
| 文字→数値（失敗は例外） | — | `pd.to_numeric(x)` | `cast(Float64)` |
| →文字列 | `as.character(x)` | `x.astype(str)` | `cast(pl.Utf8)` |
| →整数（欠損可） | `as.integer(x)` | `x.astype("Int64")` | `cast(pl.Int64, strict=False)` |
| →factor/カテゴリ | `as.factor(x)` | `x.astype("category")` | `cast(pl.Categorical)` |
| 日付へ | `as.Date(x)` | `pd.to_datetime(x)` | `str.to_date` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`astype(int)` は欠損で失敗**：NaN を含む float 列を `astype(int)` すると `IntCastingNaNError`／`ValueError`。**nullable の `"Int64"`** を使う。
    - **factor→数値の罠**：R の `as.numeric(factor)` は水準番号。pandas の Categorical→数値も `cat.codes` はコード番号。値が欲しければ一度 `astype(str)` を挟む。
    - **静かな欠損化**：`errors="coerce"` は変換不能を黙って NaN に。件数が想定と合うか必ず確認（R の警告に相当するものは自分で数える）。
    - **真偽の変換**：`as.numeric(TRUE)` は 1。pandas は `astype(int)` で True→1。`"TRUE"`/`"True"` のような**文字列**はそのままでは 1 にならない点に注意。

## 関連項目

- [005. 欠損値の扱い](topic-005.md)
- [091. 因子とラベル](../roadmap.md)
- [092. 日付・時刻の計算](../roadmap.md)
