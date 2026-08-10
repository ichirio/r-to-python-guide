# 024. 上位N・スライス — `slice_max` / `top_n` / `slice` → `nlargest` / `head`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `slice_max()` / `slice_min()`（値で上位）／`slice()`（位置で）／旧 `top_n()`
    - **Python（推奨）**: 値で上位は **`nlargest()` / `nsmallest()`**、位置は **`head()` / `iloc[]`**
    - **要注意**: `slice(1:2)` は 1 始まり両端含む、`iloc[0:2]` は 0 始まり右端含まず

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> slice_max(weight, n = 2)   # weight 上位2件
dm |> slice_min(age, n = 1)      # age 最小1件
dm |> slice(1:2)                 # 位置で先頭2行
dm |> group_by(arm) |> slice_max(weight, n = 1)  # 群ごとの最大
```

出力:

```text
# slice_max(weight, n=2) → 03(80.1), 05(72.0)
# slice_min(age, n=1)    → 03(38)
```

!!! note "R の勘所"
    - `slice_max/min(col, n=)` は**値**で上位/下位。同値の扱いは `with_ties=`。
    - `slice(1:2)` は**位置**。`slice_head(n=)` / `slice_tail(n=)` もある。
    - **群ごとの1位**は `group_by |> slice_max(n=1)` が定番（各群の代表行）。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm.nlargest(2, "weight")     # weight 上位2件
    dm.nsmallest(1, "age")       # age 最小1件
    dm.head(2)                   # 位置で先頭2行
    dm.iloc[0:2]                 # 同上（0始まり・右端含まず）

    # 群ごとの最大（各 arm の weight 最大行）
    dm.loc[dm.groupby("arm")["weight"].idxmax()]
    ```

    出力（`nlargest(2, "weight")`）:

    ```text
    subjid  weight
        03    80.1
        05    72.0
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.top_k(2, by="weight")            # 上位2件
    dmp.bottom_k(1, by="age")            # 下位1件
    dmp.head(2)                          # 位置で先頭
    # 群ごとの最大
    dmp.sort("weight", descending=True).group_by("arm").first()
    ```

!!! tip "実務ではこれ"
    - **値で上位/下位** → `nlargest(n, col)` / `nsmallest(n, col)`。並べ替え＋head より速く意図が明確。
    - **群ごとの代表行**（各被験者の最終観測など）→ `df.loc[df.groupby(key)[col].idxmax()]` か「ソート＋`groupby().head(1)`」。
    - **位置で切る** → `head`/`tail`/`iloc`。ラベルでなく行番号で取りたいなら `iloc`。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 値で上位 N | `slice_max(x, n=2)` | `nlargest(2, "x")` | `top_k(2, by="x")` |
| 値で下位 N | `slice_min(x, n=1)` | `nsmallest(1, "x")` | `bottom_k(1, by="x")` |
| 先頭 N 行 | `slice_head(n=2)` | `head(2)` / `iloc[:2]` | `head(2)` |
| 末尾 N 行 | `slice_tail(n=2)` | `tail(2)` / `iloc[-2:]` | `tail(2)` |
| 群ごとの最大行 | `group_by \|> slice_max(n=1)` | `loc[groupby(k)[c].idxmax()]` | `sort().group_by(k).first()` |
| 位置指定 | `slice(1:2)` | `iloc[0:2]` | `slice(0, 2)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **位置の基準**：`slice(1:2)` は 1 始まり・両端含む＝先頭2行。pandas は `iloc[0:2]`（0 始まり・右端含まず）で同じ2行。
    - **同値（tie）**：`slice_max(with_ties=TRUE)` は同値を全部残すので行数が n を超えうる。`nlargest` は既定 `keep="first"` で n 行に切る。合わせるなら `keep="all"`。
    - **群ごとの最大**：`idxmax` は最初の最大値の**index ラベル**を返す。事前に `reset_index` していないと想定外の行を拾うことがある。
    - **欠損**：`nlargest` は NaN を除外。`slice_max` も NA を上位に含めない。

## 関連項目

- [021. 並べ替え](topic-021.md)
- [030. グループ内変換](topic-030.md)
