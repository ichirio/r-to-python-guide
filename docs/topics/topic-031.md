# 031. ウィンドウ関数：lag / lead / cumsum → `shift` / `cumsum`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `lag()` / `lead()`（前後の値）、`cumsum()` / `cummax()` など（累積）
    - **Python（推奨）**: pandas **`shift(1)` / `shift(-1)`**、**`cumsum()`**（群ごとは `groupby(key)[col]` 経由）；polars は `shift` / `cum_sum` に `over()`
    - **要注意**: 群ごとに効かせるには**必ず `group_by`/`groupby`** を挟む。忘れると被験者をまたいで値が漏れる

「前回来院との差」「累積投与量」など、臨床の縦持ちデータで多用。

---

## R ではこう書く

```r
library(dplyr)
lb <- tibble(subjid = c("01","01","01","02","02"),
             visit = c(1,2,3,1,2), val = c(10,12,15,20,19))

lb |> group_by(subjid) |>
  mutate(prev = lag(val),           # 1つ前
         chg  = val - lag(val),     # 前回差
         cum  = cumsum(val)) |>     # 累積
  ungroup()
```

出力:

```text
  subjid visit   val  prev   chg   cum
  01         1    10    NA    NA    10
  01         2    12    10     2    22
  01         3    15    12     3    37
  02         1    20    NA    NA    20    ← 被験者が変わると lag は NA から
  02         2    19    20    -1    39
```

!!! note "R の勘所"
    - `lag(x, n=1, default=NA)`：n 個前。`lead()` は後ろ。
    - **必ず `group_by(subjid)`** してから。しないと被験者 02 の1行目が 01 の最後を拾う。
    - 並び順が命：`arrange(subjid, visit)` を先に。

---

## Python ではこう書く

=== "pandas"

    ```python
    lb["prev"] = lb.groupby("subjid")["val"].shift(1)          # 1つ前
    lb["chg"]  = lb["val"] - lb.groupby("subjid")["val"].shift(1)
    lb["cum"]  = lb.groupby("subjid")["val"].cumsum()          # 累積
    ```

    出力:

    ```text
    subjid  visit  val  prev  chg  cum
        01      1   10   NaN  NaN   10
        01      2   12  10.0  2.0   22
        01      3   15  12.0  3.0   37
        02      1   20   NaN  NaN   20
        02      2   19  20.0 -1.0   39
    ```

=== "polars"

    ```python
    import polars as pl
    lbp.with_columns(
        pl.col("val").shift(1).over("subjid").alias("prev"),
        (pl.col("val") - pl.col("val").shift(1).over("subjid")).alias("chg"),
        pl.col("val").cum_sum().over("subjid").alias("cum"),
    )
    ```

!!! tip "実務ではこれ"
    - **前後参照** → `groupby(key)[col].shift(±n)`。`lead` は `shift(-1)`。
    - **累積** → `groupby(key)[col].cumsum()`（`cummax`/`cummin`/`cumprod` も同様）。
    - **並べ替えを先に**：`sort_values([key, time])` してから。ウィンドウは行順に依存する。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 1つ前 | `lag(x)` | `groupby(k)[x].shift(1)` | `x.shift(1).over(k)` |
| 1つ後 | `lead(x)` | `groupby(k)[x].shift(-1)` | `x.shift(-1).over(k)` |
| 前回差 | `x - lag(x)` | `x - groupby(k)[x].shift(1)` | `x - x.shift(1).over(k)` |
| 累積和 | `cumsum(x)` | `groupby(k)[x].cumsum()` | `x.cum_sum().over(k)` |
| 累積最大 | `cummax(x)` | `groupby(k)[x].cummax()` | `x.cum_max().over(k)` |
| 変化率 | — | `groupby(k)[x].pct_change()` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **群化を忘れる**：pandas で `lb["val"].shift(1)` と書くと**群をまたいで**値が漏れる。必ず `groupby(key)[col].shift()`。
    - **並び順依存**：`shift`/`cumsum` は現在の行順で計算する。事前に `sort_values` しないと結果が狂う。R の `lag` も同じ。
    - **default 値**：R の `lag(x, default=0)` に当たるのは `shift(1).fillna(0)`。
    - **欠損の混入で float 化**：`shift` は先頭に NaN を入れるため整数列が float になる。必要なら `astype("Int64")`。

## 関連項目

- [030. グループ内変換](topic-030.md)
- [032. ランク付け](topic-032.md)
- [021. 並べ替え](topic-021.md)
