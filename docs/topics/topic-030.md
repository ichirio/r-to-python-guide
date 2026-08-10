# 030. グループ内変換 — `group_by` + `mutate` → `groupby.transform` / `over`

!!! abstract "この項目の R→Python 対応"
    - **R**: `group_by()` の後に `mutate()`（**行を畳まず**、群ごとの集計値を各行に付与）
    - **Python（推奨）**: pandas **`groupby(key)[col].transform("mean")`**；polars **`expr.over(key)`**
    - **要注意**: 畳む `summarise`/`agg` とは別物。**行数を保つ**のが `transform`/`over`

「各被験者の値 − 群平均」「群内順位」「群内累積」など、行を残したまま群統計を使う操作。

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> group_by(arm) |>
  mutate(
    mean_wt = mean(weight),          # 群平均を各行に
    dev     = weight - mean(weight)  # 群平均からの偏差
  ) |> ungroup()
```

出力:

```text
  subjid arm   weight mean_wt    dev
  01     A       70.5    67.6   2.93
  02     A       60.2    67.6  -7.37
  03     B       80.1    68    12.1
  04     B       55.9    68   -12.1
  05     A       72      67.6   4.43
```

!!! note "R の勘所"
    - `group_by |> mutate` は**行を畳まない**。`summarise` は畳む。ここが最大の分かれ道。
    - 使い終えたら `ungroup()`。付けっぱなしだと後段の集計が群単位になって事故る。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm["mean_wt"] = dm.groupby("arm")["weight"].transform("mean")
    dm["dev"] = dm["weight"] - dm["mean_wt"]
    ```

    出力:

    ```text
    subjid arm  weight  mean_wt    dev
        01   A    70.5    67.57   2.93
        02   A    60.2    67.57  -7.37
        03   B    80.1    68.00  12.10
        04   B    55.9    68.00 -12.10
        05   A    72.0    67.57   4.43
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.with_columns(
        pl.col("weight").mean().over("arm").alias("mean_wt"),
        (pl.col("weight") - pl.col("weight").mean().over("arm")).alias("dev"),
    )
    ```

!!! tip "実務ではこれ"
    - **行を残して群統計を付ける** → pandas `transform`、polars `over`。「群平均との差」「群内%」「初回値との差（baseline）」の定番。
    - `transform` は**元の行数・行順を保ったまま**ブロードキャストして返す（`agg` は畳む）。
    - 群内の順位・累積は `transform("rank")` / `groupby().cumsum()`（→ [031](../roadmap.md), [032](../roadmap.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 群平均を各行に | <code>group_by &#124;&gt; mutate(m=mean(x))</code> | `groupby(k)[x].transform("mean")` | `pl.col(x).mean().over(k)` |
| 群内偏差 | `x - mean(x)` | `x - transform("mean")` | `x - x.mean().over(k)` |
| 群内件数 | `add_count()` / `n()` | `transform("size")` | `pl.len().over(k)` |
| 群内順位 | `min_rank(x)` | `groupby(k)[x].rank()` | `x.rank().over(k)` |
| 群内累積 | `cumsum(x)` | `groupby(k)[x].cumsum()` | `x.cum_sum().over(k)` |
| baseline との差 | `x - first(x)` | `x - transform("first")` | `x - x.first().over(k)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`transform` と `agg` の混同**：`agg`/`summarise` は畳む（行数減）、`transform`/`mutate` は保つ。付けたい列なのに集約してしまうミスが多い。
    - **`ungroup()` 忘れ**（R）：group を残したまま後続処理をすると想定外に群単位になる。pandas/polars は式ごとに `over`/`transform` を指定するので付けっぱなしの事故は起きにくい。
    - **`transform` に渡す関数**：文字列（`"mean"`）が速い。ラムダも使えるが群ごとに Python 呼び出しになり遅い。
    - **欠損**：`transform("mean")` は群内で NaN を無視（pandas 既定）。R は `na.rm` を明示。

## 関連項目

- [010. グループ集約（summarise）](topic-010.md)
- [031. ウィンドウ関数：lag / lead / cumsum](../roadmap.md)
- [032. ランク付け](../roadmap.md)
