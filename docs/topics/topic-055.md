# 055. 群別 N・mean(sd) のデモグラ表

!!! abstract "この項目の R→Python 対応"
    - **R**: `group_by(arm) |> summarise(n, mean, sd, ...)`（連続変数を群ごとに要約）
    - **Python（推奨）**: pandas **`groupby("arm")["age"].agg([...])`** → f-string で `mean (sd)` に整形
    - **要注意**: `sd`/`std` は ddof=1 で一致（[052](topic-052.md)）。整形の丸めは [018](topic-018.md) の方式差に注意

デモグラフィック表の連続変数行（群ごとの N と `mean (SD)`）を作る。

（arm A: age 45,52,60,48,55 / arm B: 38,61,50,42,44）

---

## R ではこう書く

```r
library(dplyr)

dm3 |> group_by(arm) |>
  summarise(n = n(),
            mean_age = mean(age),
            sd_age   = sd(age)) |>
  mutate(cell = sprintf("%.1f (%.2f)", mean_age, sd_age))
```

出力（要約部）:

```text
  arm       n mean_age sd_age
  A         5       52   5.87
  B         5       47   8.94
```

!!! note "R の勘所"
    - `n()` は行数、非欠損数は `sum(!is.na(age))`。TFL の N の定義を確認。
    - `mean`/`sd` は `na.rm=TRUE` を要件どおりに。
    - 表示は `sprintf` で `mean (sd)` に整形（→ [016](topic-016.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    g = (dm3.groupby("arm")
             .agg(n=("subjid", "size"),
                  mean_age=("age", "mean"),
                  sd_age=("age", "std"))       # ddof=1
             .reset_index())
    g["cell"] = g.apply(lambda r: f"{r['mean_age']:.1f} ({r['sd_age']:.2f})", axis=1)
    print(g)
    ```

    出力（要約部）:

    ```text
    arm  n  mean_age   sd_age
      A  5      52.0 5.873670
      B  5      47.0 8.944272
    ```

=== "polars"

    ```python
    import polars as pl
    (dm3p.group_by("arm")
         .agg(pl.len().alias("n"),
              pl.col("age").mean().alias("mean_age"),
              pl.col("age").std().alias("sd_age"))
         .sort("arm"))
    ```

!!! tip "実務ではこれ"
    - **数値で集計 → 最後に文字列化**（`f"{m:.1f} ({s:.2f})"`）。数値のまま持てば検算・並べ替えができる。
    - **複数の連続変数**（age, weight, BMI…）を一度に出すなら、変数を縦持ち（melt）してから群×変数で集計 → 横持ち（→ [050](topic-050.md)）。
    - N を「非欠損数」にするなら `("age","count")`（`size` は欠損込み）。

---

## デモグラ表の骨格（連続＋カテゴリ）

連続変数は `mean (SD)` / `median [Q1,Q3]`、カテゴリ変数は `n (%)`。**別々に作って縦に積む**のが実務の定石です。

```python
# 連続変数行
cont = (dm3.groupby("arm")["age"].agg(["mean","std"])
           .apply(lambda r: f"{r['mean']:.1f} ({r['std']:.2f})", axis=1))

# カテゴリ変数行（sex の n(%)）は [057] のパターンで作り、pd.concat で縦結合
```

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 群別 N | `summarise(n = n())` | `agg(n=("id","size"))` |
| 群別 mean(sd) | `mean(x), sd(x)` | `agg(["mean","std"])` |
| `mean (sd)` 整形 | `sprintf("%.1f (%.2f)")` | `f"{m:.1f} ({s:.2f})"` |
| 非欠損 N | `sum(!is.na(x))` | `("x","count")` |
| 複数変数を一度に | melt → summarise | melt → groupby → pivot |

## つまずきポイント

!!! warning "R と Python の差"
    - **N の定義**：`n()`/`size` は行数、`count` は非欠損数。欠損のある変数で「N=集計対象数」を出すなら `count`。
    - **sd の自由度**：pandas `.std()` は ddof=1 で R と一致。`np.std` は ddof=0 なので使わない（[052](topic-052.md)）。
    - **丸めの方式**：`mean (sd)` の丸めが SAS と 0.5 でズレることがある（[018](topic-018.md)）。仕様に合わせる。
    - **群順**：Placebo→実薬の順など、群の表示順は Categorical 水準で固定（`groupby` の既定はソート）。

## 関連項目

- [050. ロング／ワイド実務パターン](topic-050.md)
- [052. 連続変数の要約](topic-052.md)
- [057. n (%) 整形のパターン](topic-057.md)
- [067. デモグラフィック表の組み立て](../roadmap.md)
