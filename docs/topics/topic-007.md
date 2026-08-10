# 007. 列選択 — `select` → `[]` / polars `select`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `select()`（`starts_with()` などのヘルパが強力）
    - **Python（推奨）**: pandas は **`df[[...]]`**、条件選択は `df.filter(...)` / `columns.str`；polars は **`select()`**
    - **要注意**: pandas の `df.filter()` は dplyr の `filter`（行抽出）ではなく**列選択**。名前が紛らわしい

（データは [003](topic-003.md) の `dm` を使用）

---

## R ではこう書く

```r
library(dplyr)

dm |> select(subjid, arm, age)   # 名前で選ぶ
dm |> select(starts_with("s"))   # ヘルパで選ぶ
dm |> select(-weight)            # 除外
```

出力（先頭のみ抜粋）:

```text
# A tibble: 5 × 3
  subjid arm     age
  <chr>  <chr> <dbl>
1 01     A        45
...
# A tibble: 5 × 2   （subjid, sex）
# A tibble: 5 × 4   （weight を除いた4列）
```

!!! note "select ヘルパの使い分け"
    `starts_with()` / `ends_with()` / `contains()` / `matches()`（正規表現）/ `where(is.numeric)`（型で選ぶ）/ `everything()`（残り全部）。臨床データの `AE**` 系や `**DT` 系の列をまとめて掴むのに多用します。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 名前で選ぶ（二重括弧！）
    dm[["subjid", "arm", "age"]]

    # 前方一致で選ぶ
    dm.loc[:, dm.columns.str.startswith("s")]

    # 除外
    dm.drop(columns="weight")

    # 型で選ぶ（where(is.numeric) 相当）
    dm.select_dtypes("number")
    ```

    列名の一致で選ぶなら `df.filter`（※行ではなく**列**）も使えます:

    ```python
    dm.filter(like="id")           # 名前に "id" を含む列
    dm.filter(regex="^s")          # 正規表現
    ```

=== "polars"

    ```python
    import polars as pl

    dmp.select(["subjid", "arm", "age"])       # 名前で
    dmp.select(pl.col("^s.*$"))                # 正規表現（^...$ で囲む）
    dmp.drop("weight")                         # 除外
    dmp.select(pl.col(pl.NUMERIC_DTYPES))      # 型で
    ```

    polars の `select` は dplyr の `select` に最も近く、名前・正規表現・型セレクタが揃っています。

!!! tip "実務ではこれ"
    - 単純な列選択は pandas の **`df[[...]]`**（**二重括弧**。一重 `df["col"]` は Series になる）。
    - `starts_with` 系のヘルパ選択は、pandas なら `df.columns.str.*` か `df.filter(regex=...)`、polars なら `pl.col("正規表現")` が近い。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 名前で選ぶ | `select(a, b)` | `df[["a","b"]]` | `df.select(["a","b"])` |
| 前方一致 | `select(starts_with("s"))` | `df.filter(regex="^s")` | `df.select(pl.col("^s.*$"))` |
| 除外 | `select(-x)` | `df.drop(columns="x")` | `df.drop("x")` |
| 型で選ぶ | `select(where(is.numeric))` | `df.select_dtypes("number")` | `df.select(pl.col(pl.NUMERIC_DTYPES))` |
| 並べ替え | `select(b, a)` | `df[["b","a"]]` | `df.select(["b","a"])` |
| 残り全部 | `everything()` | `df.columns` を使う | `pl.all()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`filter` の意味が真逆**：dplyr の `filter()` は**行**抽出、pandas の `df.filter()` は**列**選択。行抽出は [008](topic-008.md) の `query` / boolean mask。
    - **一重 vs 二重括弧**：`df["age"]` は 1 次元の Series、`df[["age"]]` は 1 列の DataFrame。列選択のつもりなら二重。
    - **選択後のコピー警告**：pandas で選択した後に代入すると `SettingWithCopyWarning` が出ることがある。`.copy()` を付けるか `.assign()`（→ [009](topic-009.md)）で新規に作る。

## 関連項目

- [008. 行フィルタ（filter）](topic-008.md)
- [009. 列作成・変更（mutate）](topic-009.md)
- [025. リネーム（rename）](../roadmap.md)
