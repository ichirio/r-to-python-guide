# 022. 件数と頻度 — `count` / `n()` / `tally` → `value_counts` / `size`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `count()`（群ごとの行数を一発）／`n()`（`summarise` 内）／`tally()`
    - **Python（推奨）**: 1 列は **`value_counts()`**、複数キーは **`groupby(...).size()`** か **`df.value_counts([...])`**
    - **要注意**: `value_counts()` は既定で**件数の多い順**に並ぶ（R の `count` はキー順）

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> count(arm)              # arm ごとの件数
dm |> count(arm, sex)         # 2 キーの組
dm |> count(arm, sort = TRUE) # 件数降順
```

出力:

```text
# arm: A=3, B=2
# arm×sex: A/F=1, A/M=2, B/F=1, B/M=1
```

!!! note "R の勘所"
    - `count(x)` は `group_by(x) |> summarise(n = n())` の短縮。`name=` で列名変更。
    - `add_count()` は元の行を保ったまま件数列を付ける（→ [030](topic-030.md) 的な使い方）。
    - 重み付き集計は `count(x, wt = w)`。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 1 列の頻度（件数降順で返る）
    dm["arm"].value_counts()                 # A:3, B:2
    dm["arm"].value_counts(normalize=True)   # 割合

    # 複数キー
    dm.value_counts(["arm", "sex"]).reset_index(name="n")
    dm.groupby(["arm", "sex"]).size().reset_index(name="n")
    ```

    出力（`groupby(["arm","sex"]).size()`）:

    ```text
    arm sex  n
      A   F  1
      A   M  2
      B   F  1
      B   M  1
    ```

=== "polars"

    ```python
    import polars as pl
    dmp["arm"].value_counts()                       # 列の頻度
    dmp.group_by(["arm", "sex"]).len().sort(["arm", "sex"])  # 複数キー件数
    ```

!!! tip "実務ではこれ"
    - **1 変数の度数**は `value_counts()`（割合は `normalize=True`）。ただし**件数降順**なので、キー順に見せたいなら `.sort_index()`。
    - **クロス（2 変数以上）**は `groupby([...]).size().reset_index(name="n")` が `count(a, b)` に一番忠実。表向けは `crosstab`（→ [053](../roadmap.md)）。
    - 欠損も数えたいなら `value_counts(dropna=False)`（既定は欠損を除外）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 1 列の度数 | `count(x)` | `df["x"].value_counts()` | `df["x"].value_counts()` |
| 割合 | `count(x) \|> mutate(p=n/sum(n))` | `value_counts(normalize=True)` | — |
| 複数キー | `count(a, b)` | `groupby(["a","b"]).size()` | `group_by(["a","b"]).len()` |
| 件数降順 | `count(x, sort=TRUE)` | `value_counts()`（既定） | `value_counts(sort=True)` |
| 欠損も数える | 既定で数える | `value_counts(dropna=False)` | 既定で数える |
| 行を保ち件数付与 | `add_count(x)` | `groupby("x")["x"].transform("size")` | `pl.len().over("x")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **並び順**：`value_counts()` は**件数降順**、`count()` は**キー昇順**。TFL はキー順が普通なので `sort_index()` を挟む。
    - **欠損の既定**：pandas `value_counts` は既定で NaN を除外（`dropna=True`）。R の `count` は NA も 1 カテゴリとして数える。合わせるなら `dropna=False`。
    - **戻り値の形**：`value_counts` は Series（index が値）。表にするなら `.reset_index(name="n")`。
    - **`size` vs `count`**：`size()` は行数（NA 込み）、`count()` は非欠損数。度数集計は普通 `size`。

## 関連項目

- [010. グループ集約](topic-010.md)
- [053. カテゴリの頻度と割合](../roadmap.md)
- [057. n (%) 整形のパターン](../roadmap.md)
