# 029. across で複数列に適用 — `across` → agg dict / `apply`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `across(列, 関数)`（`summarise`/`mutate` 内で複数列に同じ処理）
    - **Python（推奨）**: 集約は **`agg({列: 関数})`** か **`groupby[列].agg(関数)`**；変換は `df[列].apply(...)`
    - **要注意**: R の `across` は「1 つの動詞の中で複数列」。Python は agg 辞書やリストで表現する

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

# 複数列を同じ関数で集約
dm |> group_by(arm) |>
  summarise(across(c(age, weight), \(x) mean(x, na.rm = TRUE)))

# 型で選んで一括変換
dm |> mutate(across(where(is.numeric), \(x) round(x, 1)))

# 複数関数（名前付き）
dm |> group_by(arm) |>
  summarise(across(weight, list(mean = mean, sd = sd)))
```

出力（`across(c(age, weight), mean)`）:

```text
  arm     age weight
  A      48.5   67.6
  B      49.5   68
```

!!! note "R の勘所"
    - `across(cols, fn)`：`cols` は tidy-select（`c()`, `starts_with()`, `where()`）。
    - **複数関数**は `list(name = fn)`。列名は `列_関数` になる。
    - 旧 `summarise_at` / `mutate_if` は `across` に統合された。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 複数列を同じ関数で（agg 辞書 or リスト）
    dm.groupby("arm")[["age", "weight"]].mean().reset_index()
    dm.groupby("arm").agg({"age": "mean", "weight": "mean"}).reset_index()

    # 型で選んで一括変換（where(is.numeric) 相当）
    num = dm.select_dtypes("number").columns
    dm[num] = dm[num].round(1)

    # 複数関数
    dm.groupby("arm")["weight"].agg(["mean", "std"]).reset_index()
    ```

    出力（`groupby("arm")[["age","weight"]].mean()`）:

    ```text
    arm  age    weight
      A 48.5 67.566667
      B 49.5 68.000000
    ```

=== "polars"

    ```python
    import polars as pl
    # 複数列を一度に（式を列リストに展開）
    dmp.group_by("arm").agg(pl.col("age", "weight").mean()).sort("arm")

    # 複数関数は式を並べる
    dmp.group_by("arm").agg(
        pl.col("weight").mean().alias("weight_mean"),
        pl.col("weight").std().alias("weight_sd"),
    ).sort("arm")
    ```

!!! tip "実務ではこれ"
    - **同じ関数を複数列に** → `groupby(key)[cols].agg(fn)` か `agg({col: fn})`。
    - **列ごとに違う関数** → `agg({"age": "mean", "weight": ["mean", "std"]})`。
    - **型で選んで一括** → `select_dtypes("number")` で列を取り、まとめて処理。
    - polars は `pl.col("a", "b").mean()` で**式が複数列に自動展開**され、`across` の感覚に最も近い。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 複数列を同関数で集約 | `summarise(across(c(a,b), f))` | `agg({"a":f,"b":f})` | `agg(pl.col("a","b").f())` |
| 型で選んで一括 | `across(where(is.numeric), f)` | `df.select_dtypes("number")` | `pl.col(pl.NUMERIC_DTYPES)` |
| 複数関数 | `across(x, list(m=mean,s=sd))` | `agg({"x":["mean","std"]})` | 式を並べる |
| 列ごとに別関数 | `summarise(a=mean(a), b=max(b))` | `agg({"a":"mean","b":"max"})` | 式を並べる |
| 変換（mutate 版） | `mutate(across(cols, f))` | `df[cols] = df[cols].apply(f)` | `with_columns(pl.col(cols).f())` |

## つまずきポイント

!!! warning "R と Python の差"
    - **列名の付き方**：R の `across` 複数関数は `weight_mean` の形。pandas の `agg([...])` は **MultiIndex 列**になり、`.columns = [...]` で平坦化が要る。
    - **`na.rm` の既定**：R は明示しないと NA 伝播。pandas の `mean`/`std` は既定で欠損を無視（[005](topic-005.md)）。
    - **`std` の自由度**：R の `sd()` と pandas の `std()` はどちらも既定 **ddof=1（不偏）**。ただし NumPy の `np.std` は **ddof=0** なので混ぜない。
    - **`select_dtypes` の範囲**：`"number"` は int/float 両方。bool を除きたい等は明示。

## 関連項目

- [010. グループ集約](topic-010.md)
- [052. 連続変数の要約](../roadmap.md)
