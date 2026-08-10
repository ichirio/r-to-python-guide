# 063. 相関 — `cor` / `cor.test` → `corr` / `scipy`

!!! abstract "この項目の R→Python 対応"
    - **R**: `cor(x, y)`（係数）／`cor.test(x, y)`（p 値・CI）／`method =`（pearson/spearman/kendall）
    - **Python（推奨）**: pandas **`df.corr()`** / `Series.corr()`（行列・係数）／**`scipy.stats.pearsonr`**（p 値付き）
    - **要注意**: `cor()` は欠損があると既定で NA。pandas は既定でペアワイズに欠損を無視

（age と weight の相関。r ≈ 0.984）

---

## R ではこう書く

```r
cor(dm3$age, dm3$weight)                       # ピアソン係数
cor.test(dm3$age, dm3$weight)                  # p 値・95%CI 付き
cor(dm3$age, dm3$weight, method = "spearman")  # スピアマン
cor(dm3[c("age","weight")])                    # 相関行列
```

出力:

```text
pearson r = 0.9836
cor.test: r=0.9836 p=0.0000
spearman = 0.9879
```

!!! note "R の勘所"
    - `cor()` は係数だけ、`cor.test()` は検定と CI も。
    - `method = "pearson"`（既定）/`"spearman"`/`"kendall"`。
    - 欠損があると `cor()` は NA。`use = "pairwise.complete.obs"` で無視。

---

## Python ではこう書く

=== "pandas + scipy"

    ```python
    from scipy import stats

    dm3["age"].corr(dm3["weight"])                    # ピアソン係数
    dm3["age"].corr(dm3["weight"], method="spearman") # スピアマン
    dm3[["age","weight"]].corr()                      # 相関行列

    # p 値・CI が要るなら scipy
    r, p = stats.pearsonr(dm3["age"], dm3["weight"])
    print(f"r={r:.4f} p={p:.4f}")
    ```

    出力:

    ```text
    pearson r = 0.9836
    pearsonr: r=0.9836 p=0.0000
    spearman = 0.9879
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p.select(pl.corr("age", "weight"))                       # ピアソン
    dm3p.select(pl.corr("age", "weight", method="spearman"))
    ```

!!! tip "実務ではこれ"
    - **係数・相関行列** → pandas `.corr()`（`method=` で種類）。ヒートマップ化しやすい。
    - **p 値・CI が要る** → `scipy.stats.pearsonr` / `spearmanr` / `kendalltau`。`pearsonr` は新しめの scipy で CI も返す（`.confidence_interval()`）。
    - 欠損は事前に `dropna()` して対象を明確に。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| ピアソン係数 | `cor(x, y)` | `x.corr(y)` / `pearsonr(x,y)[0]` |
| スピアマン | `cor(x, y, method="spearman")` | `x.corr(y, method="spearman")` |
| ケンドール | `cor(x, y, method="kendall")` | `x.corr(y, method="kendall")` |
| p 値・CI | `cor.test(x, y)` | `scipy.stats.pearsonr(x, y)` |
| 相関行列 | `cor(df)` | `df.corr()` |
| 欠損を無視 | `use="pairwise.complete.obs"` | 既定でペアワイズ無視 |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損の既定**：`cor()` は欠損で NA（`use=` 指定が要る）。pandas `.corr()` は既定でペアワイズに欠損を無視。scipy `pearsonr` は NaN があると NaN を返すので `dropna` してから。
    - **係数だけ or 検定付き**：pandas `.corr()` は p 値を返さない。有意性は scipy を併用。
    - **相関行列の対象**：`df.corr()` は数値列のみ。文字列列は自動除外。
    - **ケンドールの実装差**：tie の扱い（tau-b など）で微差が出ることがある。定義を確認。

## 関連項目

- [062. 分散分析・線形回帰](topic-062.md)
- [052. 連続変数の要約](topic-052.md)
