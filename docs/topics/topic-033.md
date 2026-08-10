# 033. 行ごと処理・rowwise — `rowwise` → `apply(axis=1)` / ベクトル化

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `rowwise()`（1 行ずつ処理）／`c_across()`／`rowMeans()`・`rowSums()`
    - **Python（推奨）**: まず**ベクトル化**（`df[cols].mean(axis=1)` 等）。どうしても行単位なら `apply(axis=1)`
    - **要注意**: `rowwise` も `apply(axis=1)` も**遅い**。多くは列演算に書き換えられる

（例: 3 列 a, b, cc の行平均）

---

## R ではこう書く

```r
library(dplyr)
w <- tibble(a = c(1,2), b = c(3,4), cc = c(5,6))

# rowwise：1 行ずつ
w |> rowwise() |> mutate(m = mean(c(a, b, cc))) |> ungroup()

# 専用関数（速い）
w |> mutate(m = rowMeans(across(c(a, b, cc))))
```

出力:

```text
      a     b    cc     m
      1     3     5     3
      2     4     6     4
```

!!! note "R の勘所"
    - `rowwise()` は「各行を1グループ」にする仕組み。`c_across()` で列をまとめて渡す。
    - **行方向の集計は `rowMeans`/`rowSums` が断然速い**。rowwise は最後の手段。

---

## Python ではこう書く

=== "pandas"

    ```python
    w = pd.DataFrame({"a":[1,2], "b":[3,4], "cc":[5,6]})

    # ベクトル化（推奨・速い）：axis=1 で行方向
    w["m"] = w[["a","b","cc"]].mean(axis=1)

    # 行単位の任意処理（rowwise 相当・遅い）
    w["m2"] = w.apply(lambda r: (r["a"] + r["b"] + r["cc"]) / 3, axis=1)
    ```

    出力（`m`）:

    ```text
    [3.0, 4.0]
    ```

=== "polars"

    ```python
    import polars as pl
    wp = pl.DataFrame({"a":[1,2], "b":[3,4], "cc":[5,6]})
    # 行方向の平均は式で（速い）
    wp.with_columns(pl.mean_horizontal("a", "b", "cc").alias("m"))
    ```

!!! tip "実務ではこれ"
    - **まず列演算に落とせないか考える**：行平均は `df[cols].mean(axis=1)`、行合計は `.sum(axis=1)`、条件は `np.where`/`np.select`（→ [026](topic-026.md)）。
    - **本当に行単位のロジック**（外部関数呼び出し等）だけ `apply(axis=1)`。ただし数万行で遅くなる。
    - polars は `*_horizontal`（`sum_horizontal`/`mean_horizontal`/`min_horizontal`）で行方向をベクトル化。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 行平均 | `rowMeans(across(cols))` | `df[cols].mean(axis=1)` | `pl.mean_horizontal(cols)` |
| 行合計 | `rowSums(across(cols))` | `df[cols].sum(axis=1)` | `pl.sum_horizontal(cols)` |
| 行最大 | `pmax(...)` | `df[cols].max(axis=1)` | `pl.max_horizontal(cols)` |
| 任意の行処理 | <code>rowwise() &#124;&gt; mutate()</code> | `df.apply(f, axis=1)` | `map_rows` / struct |
| 行ごと欠損数 | — | `df.isna().sum(axis=1)` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **速度**：`rowwise` も `apply(axis=1)` も1行ずつ Python/R を呼ぶので遅い。列演算（ベクトル化）に置き換えると桁で速くなる。
    - **`axis` の向き**：pandas は `axis=1` が「行方向（列をまたぐ）」。`axis=0`（既定）は列方向。R の `rowMeans`/`colMeans` の対応を取り違えない。
    - **欠損の扱い**：`df[cols].mean(axis=1)` は既定で NaN を無視して残りで平均。全列 NaN の行は NaN。R の `rowMeans(..., na.rm=TRUE)` に相当。
    - **`apply(axis=1)` の戻り型**：行ごとに Series を返すと DataFrame になる。スカラーを返すよう設計する。

## 関連項目

- [029. across で複数列に適用](topic-029.md)
- [026. 条件で値を作る](topic-026.md)
- [090. ベクトル化の考え方](../roadmap.md)
