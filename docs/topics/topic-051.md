# 051. 記述統計の一括 — `summary` → `describe`

!!! abstract "この項目の R→Python 対応"
    - **R**: `summary()`（Min/1Q/Median/Mean/3Q/Max）／`psych::describe()`（歪度など）
    - **Python（推奨）**: pandas **`describe()`**（count/mean/std/min/四分位/max）
    - **要注意**: 並ぶ統計量が微妙に違う。`summary` は sd を出さず、`describe` は count/std を出す

まず全体像を掴む一発。列ごと・群ごとの詳細は [052](topic-052.md) 以降で。

（データは [050](topic-050.md) の `dm3`、age = 45,52,60,48,55,38,61,50,42,44）

---

## R ではこう書く

```r
summary(dm3$age)          # 数値ベクトルの5数要約＋平均
summary(dm3)              # データフレーム全体（列ごと）
# psych::describe(dm3)    # n, mean, sd, median, min, max, skew, kurtosis...
```

出力（`summary(dm3$age)`）:

```text
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  38.00   44.25   49.00   49.50   54.25   61.00
```

!!! note "R の勘所"
    - `summary()` は **sd を出さない**（5 数要約＋平均）。ばらつきは別途 `sd()`。
    - 因子列は水準ごとの度数を出す。数値と因子で表示が変わる。
    - より詳細（sd・歪度・尖度）は `psych::describe()` や `skimr::skim()`。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm3["age"].describe()     # count/mean/std/min/25%/50%/75%/max
    dm3.describe()            # 数値列すべて
    dm3.describe(include="all")            # 文字列・カテゴリも（unique/top/freq）
    dm3.describe(percentiles=[.1, .5, .9]) # 分位点を指定
    ```

    出力（`dm3["age"].describe()`）:

    ```text
    count    10.000000
    mean     49.500000
    std       7.604823
    min      38.000000
    25%      44.250000
    50%      49.000000
    75%      54.250000
    max      61.000000
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p.describe()          # count/null_count/mean/std/min/25%/50%/75%/max
    ```

!!! tip "実務ではこれ"
    - ざっと見るなら `describe()`。**`std` が入る**ので R の `summary` より情報が多い。
    - 文字列・カテゴリ列も見たいなら `include="all"`。
    - 群ごとの要約は `groupby("arm").describe()` か、必要な統計だけ `agg`（→ [052](topic-052.md), [055](topic-055.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 数値の要約 | `summary(x)` | `x.describe()` |
| DF 全体 | `summary(df)` | `df.describe()` |
| 文字列も含める | — | `df.describe(include="all")` |
| 分位点指定 | `quantile(x, p)` | `describe(percentiles=[...])` |
| sd・歪度も | `psych::describe()` | `df.agg(["mean","std","skew","kurt"])` |
| 群ごと | `by(x, g, summary)` | `df.groupby(g).describe()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **並ぶ統計量が違う**：`summary` は Min/1Q/Median/Mean/3Q/Max（sd なし）。`describe` は count/mean/std/min/25/50/75/max（**std あり、平均の位置も違う**）。
    - **四分位の定義**：`describe` の 25%/75% は線形補間（R の `quantile` の既定 type 7 と**一致**）。他ソフト（SAS の一部定義）とは違い得る（→ [052](topic-052.md)）。
    - **欠損**：`describe` の `count` は非欠損数。欠損があると mean 等は自動で無視（[005](topic-005.md)）。
    - **文字列列**：`describe()` は既定で数値列だけ。カテゴリの要約は `include=`。

## 関連項目

- [052. 連続変数の要約](topic-052.md)
- [055. 群別 N・mean(sd) のデモグラ表](topic-055.md)
- [005. 欠損値の扱い](topic-005.md)
