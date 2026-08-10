# 052. 連続変数の要約 — `mean` / `sd` / `median` / `quantile` → `agg`

!!! abstract "この項目の R→Python 対応"
    - **R**: `mean()` / `sd()` / `median()` / `quantile()` / `IQR()`（群ごとは `group_by |> summarise`）
    - **Python（推奨）**: pandas **`agg(["mean","std","median","min","max"])`** ＋ `quantile([...])`
    - **要注意**: `sd`/`std` はどちらも **ddof=1（不偏）**。ただし **`np.std` は ddof=0** なので混ぜない

デモグラ表の連続変数行（Age: mean (SD), median [Q1, Q3], min–max）の材料。

（age = 45,52,60,48,55,38,61,50,42,44）

---

## R ではこう書く

```r
a <- dm3$age
c(mean = mean(a), sd = sd(a), median = median(a),
  q1 = quantile(a, .25), q3 = quantile(a, .75),
  min = min(a), max = max(a))
```

出力:

```text
mean=49.50 sd=7.60 median=49.0 q1=44.25 q3=54.25 min=38 max=61
```

!!! note "R の勘所"
    - `sd()` は**不偏（n−1）**。`var()` も同様。
    - `quantile()` の既定は **type 7**（線形補間）。他の定義は `type=` で。
    - `IQR()` は Q3−Q1。欠損があると `na.rm=TRUE` が要る。

---

## Python ではこう書く

=== "pandas"

    ```python
    a = dm3["age"]
    a.agg(["mean", "std", "median", "min", "max"])   # std は ddof=1（不偏）
    a.quantile([0.25, 0.75])                          # 線形補間（R type 7 と一致）
    a.quantile(0.75) - a.quantile(0.25)               # IQR
    ```

    出力（要点）:

    ```text
    mean=49.50 sd=7.60 median=49.0 q1=44.25 q3=54.25 min=38 max=61
    ```

    群ごと・複数統計をまとめて:

    ```python
    dm3.groupby("arm")["age"].agg(["mean", "std", "median",
                                   lambda s: s.quantile(.25),
                                   lambda s: s.quantile(.75)])
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p.select(
        pl.col("age").mean().alias("mean"),
        pl.col("age").std().alias("sd"),          # ddof=1
        pl.col("age").median().alias("median"),
        pl.col("age").quantile(0.25).alias("q1"),
        pl.col("age").quantile(0.75).alias("q3"),
    )
    ```

!!! tip "実務ではこれ"
    - 複数統計は **`agg([...])`** でまとめて。名前付きにするなら `agg(mean=("age","mean"), sd=("age","std"))`。
    - **`std` は必ず pandas/polars の `.std()`（ddof=1）を使う**。`np.std(a)` は ddof=0 で **R の `sd` と一致しない**。
    - 分位点は `quantile()`。pandas の既定補間は R type 7 と同じなので値が一致する。

---

## 対応早見表

| 統計量 | R | Python（pandas） | 既定 |
|---|---|---|---|
| 平均 | `mean(x)` | `x.mean()` | 欠損無視（pandas） |
| 標準偏差 | `sd(x)` | `x.std()` | **ddof=1** |
| 分散 | `var(x)` | `x.var()` | ddof=1 |
| 中央値 | `median(x)` | `x.median()` | — |
| 分位点 | `quantile(x, p)` | `x.quantile(p)` | 線形（type 7） |
| IQR | `IQR(x)` | `q75 - q25` | — |
| 範囲 | `range(x)` | `[x.min(), x.max()]` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **`np.std` の罠**：NumPy の `np.std`/`np.var` は既定 **ddof=0**（母集団）。R の `sd` と合わせるなら pandas の `.std()`（ddof=1）か `np.std(a, ddof=1)`。
    - **分位点の定義**：pandas 既定は R type 7 と一致するが、SAS の一部プロシジャは別定義。TFL 照合時は定義を確認（`interpolation=`/`method=` で変更可）。
    - **欠損の既定**：pandas は無視、R は `na.rm=TRUE` が要る（[005](topic-005.md)）。
    - **lambda 集約は遅い**：分位点を群ごとに出すなら `groupby().quantile([.25,.75])` の方が速い。

## 関連項目

- [051. 記述統計の一括](topic-051.md)
- [055. 群別 N・mean(sd) のデモグラ表](topic-055.md)
- [058. 中央値 [Q1, Q3] / min–max の整形](topic-058.md)
