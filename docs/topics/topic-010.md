# 010. グループ集約 — `group_by` + `summarise` → `groupby.agg`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `group_by()` → `summarise()`（群ごとに1行へ畳む）
    - **Python（推奨）**: pandas は **`groupby().agg(...)` + `reset_index()`**（名前付き集約）；polars は **`group_by().agg(...)`**
    - **要注意**: pandas は群キーが index になるので **`reset_index()`** で列に戻す。欠損の扱いは [005](topic-005.md) の既定差がそのまま出る

（データは [003](topic-003.md) の `dm` を使用。`arm` ごとに人数・平均年齢・平均体重を出す）

---

## R ではこう書く

```r
library(dplyr)

dm |>
  group_by(arm) |>
  summarise(
    n        = n(),
    mean_age = mean(age, na.rm = TRUE),
    mean_wt  = mean(weight)
  )
```

出力:

```text
# A tibble: 2 × 4
  arm       n mean_age mean_wt
  <chr> <int>    <dbl>   <dbl>
1 A         3     48.5    67.6
2 B         2     49.5    68
```

!!! note "R の group_by/summarise の勘所"
    - `n()` は群の行数、`sum(!is.na(x))` は非欠損数。TFL の「N」をどちらにするか要件で決める。
    - **`mean(age)` は `na.rm=TRUE` を書かないと NA**（A 群は年齢欠損が1人いる）。
    - `summarise` 後は**最後の group が1つ外れる**（`.groups=` で制御）。続けて群操作するなら注意。

---

## Python ではこう書く

=== "pandas"

    ```python
    g = (dm.groupby("arm")
           .agg(
               n=("subjid", "size"),
               mean_age=("age", "mean"),      # 既定で NaN を無視
               mean_wt=("weight", "mean"),
           )
           .reset_index())
    print(g)
    ```

    出力:

    ```text
      arm  n  mean_age    mean_wt
    0   A  3      48.5  67.566667
    1   B  2      49.5  68.000000
    ```

    `("列", "関数")` の**名前付き集約**が `summarise(name = f(col))` に一番近い書き方です。

=== "polars"

    ```python
    import polars as pl

    (dmp.group_by("arm")
        .agg(
            pl.len().alias("n"),
            pl.col("age").mean().alias("mean_age"),
            pl.col("weight").mean().alias("mean_wt"),
        )
        .sort("arm"))
    ```

    出力:

    ```text
    shape: (2, 4)
    ┌─────┬─────┬──────────┬───────────┐
    │ arm ┆ n   ┆ mean_age ┆ mean_wt   │
    ╞═════╪═════╪══════════╪═══════════╡
    │ A   ┆ 3   ┆ 48.5     ┆ 67.566667 │
    │ B   ┆ 2   ┆ 49.5     ┆ 68.0      │
    └─────┴─────┴──────────┴───────────┘
    ```

    polars の `group_by` は**順序を保証しない**ので、再現性のため最後に `sort()` を付けます。

!!! tip "実務ではこれ"
    - pandas は **名前付き集約 `agg(name=("col","func"))` + `reset_index()`** を定型にする。群キーが index に入る挙動を毎回 reset で潰しておくと後段が楽。
    - `size` は行数（`n()`）、`count` は**非欠損数**。TFL の N をどちらにするか意識して選ぶ。
    - 群ごとに複数統計をまとめて出すなら、後の項目 [052](../roadmap.md)（連続変数の要約）で `mean`/`std`/`median` を一括する型を扱う。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 群化して集約 | `group_by(g) \|> summarise(...)` | `groupby("g").agg(...)` | `group_by("g").agg(...)` |
| 行数 | `n()` | `("col","size")` | `pl.len()` |
| 非欠損数 | `sum(!is.na(x))` | `("x","count")` | `pl.col("x").count()` |
| 平均（欠損無視） | `mean(x, na.rm=TRUE)` | `("x","mean")`（既定） | `pl.col("x").mean()` |
| 群キーを列に戻す | 既定で列 | `.reset_index()` | 既定で列 |
| 群内で畳まず付与 | `group_by \|> mutate` | `groupby().transform` | `over(...)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損の既定が逆**：R は `mean(age)` が NA、pandas は自動で無視（[005](topic-005.md)）。この例では pandas の `mean_age` が **R の `na.rm=TRUE` と同じ 48.5** になる。SAS/R と照合するなら欠損の扱いを明示。
    - **group キーの位置**：pandas は群キーが **index** に入る → `reset_index()` を忘れると後段でズレる。polars/R は列のまま。
    - **順序**：polars の `group_by` は順不同。TFL は並びが命なので `sort()` を必ず付ける。pandas は既定で群キーをソートする（`sort=False` で抑制可）。
    - **群内変換（畳まない）は別物**：`group_by |> mutate` に当たるのは `summarise`/`agg` ではなく `transform`（pandas）/`over`（polars）。→ [030](../roadmap.md)。

## 関連項目

- [009. 列作成・変更（mutate）](topic-009.md)
- [022. 件数と頻度（count）](../roadmap.md)
- [052. 連続変数の要約](../roadmap.md)
- [055. 群別 N・mean(sd) のデモグラ表](../roadmap.md)
