# 032. ランク付け — `min_rank` / `dense_rank` / `row_number` → `rank`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `min_rank()` / `dense_rank()` / `row_number()` / `percent_rank()`（同順位の扱いが違う）
    - **Python（推奨）**: pandas **`Series.rank(method=)`**（`"min"` / `"dense"` / `"first"`）
    - **要注意**: 同順位（tie）の扱いが method で変わる。R の各関数と `method` の対応を押さえる

（例データ: `x = 10, 20, 20, 5`）

---

## R ではこう書く

```r
library(dplyr)
d <- tibble(id = c("a","b","c","d"), x = c(10,20,20,5))

d |> mutate(
  r_min   = min_rank(x),     # 同順位は同じ、次は飛ぶ（1,3,3,4 の型）
  r_dense = dense_rank(x),   # 同順位は同じ、次は飛ばない
  r_row   = row_number(x),   # 同順位でも一意（出現順で決着）
  r_desc  = min_rank(desc(x))# 降順
)
```

出力:

```text
  id      x r_min r_dense r_row r_desc
  a      10     2       2     2      3
  b      20     3       3     3      1
  c      20     3       3     4      1
  d       5     1       1     1      4
```

!!! note "R の勘所"
    - `min_rank`：同順位は最小順位を共有、次順位は**飛ぶ**（競技順位）。
    - `dense_rank`：同順位を共有し、次順位は**飛ばない**（水準番号的）。
    - `row_number`：同順位でも**一意**（先に出た方が小）。
    - 降順は `desc(x)` で包む。

---

## Python ではこう書く

=== "pandas"

    ```python
    d["r_min"]   = d["x"].rank(method="min").astype(int)
    d["r_dense"] = d["x"].rank(method="dense").astype(int)
    d["r_row"]   = d["x"].rank(method="first").astype(int)   # row_number 相当
    d["r_desc"]  = d["x"].rank(method="min", ascending=False).astype(int)
    ```

    出力:

    ```text
    id  x  r_min  r_dense  r_row  r_desc
     a 10      2        2      2       3
     b 20      3        3      3       1
     c 20      3        3      4       1
     d  5      1        1      1       4
    ```

=== "polars"

    ```python
    import polars as pl
    dp.with_columns(
        pl.col("x").rank("min").alias("r_min"),
        pl.col("x").rank("dense").alias("r_dense"),
        pl.col("x").rank("ordinal").alias("r_row"),          # row_number 相当
        pl.col("x").rank("min", descending=True).alias("r_desc"),
    )
    ```

!!! tip "実務ではこれ"
    - R 関数 → pandas `method` の対応を丸暗記:
        - `min_rank` → `method="min"`
        - `dense_rank` → `method="dense"`
        - `row_number` → `method="first"`（polars は `"ordinal"`）
    - **群内順位**は `groupby(key)[col].rank(method=)`（→ [030](topic-030.md)）。
    - `rank` の戻り値は **float**（NaN を扱えるように）。整数が要るなら `.astype("Int64")`。

---

## 対応早見表

| R | pandas `method` | polars | 同順位の次 |
|---|---|---|---|
| `min_rank(x)` | `"min"` | `"min"` | 飛ぶ |
| `dense_rank(x)` | `"dense"` | `"dense"` | 飛ばない |
| `row_number(x)` | `"first"` | `"ordinal"` | 一意 |
| `percent_rank(x)` | `pct=True`（min系） | — | — |
| 平均順位 | `"average"`（既定） | `"average"` | 平均 |

## つまずきポイント

!!! warning "R と Python の差"
    - **pandas の既定は `"average"`**：`rank()` を method なしで呼ぶと同順位が平均（2.5 など小数）になる。R の `min_rank` の感覚で呼ぶと違う結果。必ず `method=` を指定。
    - **戻り値が float**：NaN 対応のため。整数表示にするなら `astype`。
    - **欠損の位置**：`na_option="keep"`（既定、NaN のまま）/`"top"`/`"bottom"`。R の rank 系は既定で NA を最後扱い。
    - **降順の書き方**：R は `desc(x)`、pandas は `ascending=False`、polars は `descending=True`。

## 関連項目

- [031. ウィンドウ関数](topic-031.md)
- [024. 上位N・スライス](topic-024.md)
- [030. グループ内変換](topic-030.md)
