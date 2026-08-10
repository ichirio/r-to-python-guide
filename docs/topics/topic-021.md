# 021. 並べ替え — `arrange` → `sort_values`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `arrange()`（`desc()` で降順、複数キー可、**NA は最後**）
    - **Python（推奨）**: pandas **`sort_values()`**（`ascending=` で向き、`na_position=`）；polars **`sort()`**
    - **要注意**: 欠損の位置が既定でずれる。**pandas/R は NA を最後、polars は null を先頭**

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> arrange(age)                 # 昇順（NA は最後）
dm |> arrange(desc(weight))        # 降順
dm |> arrange(arm, desc(weight))   # 複数キー
```

出力（`arrange(age)` の subjid 順）:

```text
03, 01, 02, 04, 05   （05 は age=NA で最後）
```

!!! note "R の勘所"
    - 降順は列を `desc()` で包む。複数キーはカンマ区切りで優先順。
    - `arrange()` は既定で **NA を末尾**に置く。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm.sort_values("age")                                   # 昇順（NaN は最後）
    dm.sort_values("weight", ascending=False)               # 降順
    dm.sort_values(["arm", "weight"], ascending=[True, False])  # 複数キー
    ```

    出力（`sort_values("age")` の subjid 順）:

    ```text
    03, 01, 02, 04, 05   （05 は NaN で最後）
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.sort("age")                                  # 既定で null が先頭！
    dmp.sort("age", nulls_last=True)                 # R/pandas に合わせる
    dmp.sort(["arm", "weight"], descending=[False, True])
    ```

    出力（`dmp.sort("age")` の subjid 順）:

    ```text
    05, 03, 01, 02, 04   （05 は null で先頭）
    ```

!!! tip "実務ではこれ"
    - pandas は **`sort_values(by, ascending=)`**。複数キーは `ascending=[...]` を列ごとに。
    - **polars は欠損が先頭に来る**ので、R と同じ並びにするなら `nulls_last=True` を必ず付ける。
    - 並べ替え後は pandas の index が飛ぶ。連番にするなら `.reset_index(drop=True)`。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 昇順 | `arrange(x)` | `sort_values("x")` | `sort("x")` |
| 降順 | `arrange(desc(x))` | `sort_values("x", ascending=False)` | `sort("x", descending=True)` |
| 複数キー | `arrange(a, desc(b))` | `sort_values(["a","b"], ascending=[True,False])` | `sort(["a","b"], descending=[False,True])` |
| 欠損を最後 | 既定 | 既定（`na_position="last"`） | `nulls_last=True` |
| 行位置で | — | `df.iloc[order]` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損の位置**：pandas/R は NA を末尾、**polars は既定で null を先頭**。TFL の並びが要件なら `nulls_last=`/`na_position=` を明示。
    - **安定ソート**：pandas の既定 `quicksort` は同値の順序を保証しない。元の順序を保ちたいなら `kind="stable"`（`mergesort`）。R の `arrange` は安定。
    - **index が残る**：pandas は並べ替えても行 index は元番号のまま付いてくる。位置で後段処理するなら `reset_index(drop=True)`。
    - **文字列の順**：ロケールにより R と Python で並びが微妙に違うことがある（大文字小文字・記号）。

## 関連項目

- [024. 上位N・スライス](topic-024.md)
- [032. ランク付け](../roadmap.md)
