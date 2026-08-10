# 023. 重複の除去 — `distinct` → `drop_duplicates`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `distinct()`（列指定、`​.keep_all=TRUE` で他列も保持）
    - **Python（推奨）**: pandas **`drop_duplicates(subset=)`**；polars **`unique()`**
    - **要注意**: R の `distinct(a)` は**指定列だけ**に絞る。pandas の `drop_duplicates("a")` は**全列を保持**して先頭行を残す

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> distinct(arm)                 # arm の一意値だけ（1 列に）
dm |> distinct(arm, sex)            # 組の一意
dm |> distinct(arm, .keep_all = TRUE)  # 他の列も保持（各 arm の最初の行）
```

出力:

```text
# distinct(arm) → A, B（1 列）
# distinct(arm, sex) → A/M, A/F, B/M, B/F
```

!!! note "R の勘所"
    - `distinct(a)` は結果を**その列だけ**にする。他列も残したいなら `.keep_all = TRUE`。
    - 全列で重複判定するなら `distinct()`（引数なし）。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 指定列の一意値（R の distinct(arm) 相当）
    dm[["arm"]].drop_duplicates()          # → A, B（列は arm のまま）
    dm["arm"].unique()                     # ndarray で欲しいとき

    # 組の一意
    dm.drop_duplicates(["arm", "sex"])[["arm", "sex"]]

    # 他列も保持して先頭行（.keep_all=TRUE 相当）
    dm.drop_duplicates("arm")              # 全列、各 arm の最初の行
    dm.drop_duplicates("arm", keep="last") # 最後の行を残す
    ```

    出力（`drop_duplicates(["arm","sex"])`）:

    ```text
    arm sex
      A   M
      A   F
      B   M
      B   F
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.select("arm").unique()                       # 一意値
    dmp.unique(subset=["arm", "sex"])                # 組の一意
    dmp.unique(subset="arm", keep="first")           # 先頭行を残す
    ```

!!! tip "実務ではこれ"
    - 「値の一覧が欲しい」→ `dm["arm"].unique()`（順不同の ndarray）か `dm["arm"].drop_duplicates()`（出現順の Series）。
    - 「重複行を落として1行に」→ `drop_duplicates(subset=[...], keep=)`。**`keep` で先頭/末尾を選べる**のが R の `.keep_all` より柔軟。
    - 一意な**件数**だけなら `dm["arm"].nunique()`（R の `n_distinct()`）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 列の一意値 | `distinct(x)` | `df[["x"]].drop_duplicates()` | `df.select("x").unique()` |
| 一意値の配列 | `unique(df$x)` | `df["x"].unique()` | `df["x"].unique()` |
| 組の一意 | `distinct(a, b)` | `drop_duplicates(["a","b"])` | `unique(subset=["a","b"])` |
| 他列も保持 | `distinct(x, .keep_all=TRUE)` | `drop_duplicates("x")` | `unique(subset="x")` |
| 一意な件数 | `n_distinct(x)` | `df["x"].nunique()` | `df["x"].n_unique()` |
| 末尾を残す | — | `drop_duplicates("x", keep="last")` | `unique(..., keep="last")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **列が絞られるか**：`distinct(arm)` は arm 1 列だけに絞る。`drop_duplicates("arm")` は**全列**を保持して先頭行を残す。R の同じ挙動は `.keep_all=TRUE`。
    - **残す行の既定**：pandas は先頭（`keep="first"`）。R の `distinct(.keep_all=TRUE)` も最初の行。並べ替えてから重複除去すると「どの行が残るか」が変わるので順序に注意。
    - **欠損の扱い**：pandas `drop_duplicates` は NaN 同士を重複とみなす。R も NA を1カテゴリとして扱う。
    - **polars の順序**：`unique()` は既定で順序を保証しない。出現順が要るなら `maintain_order=True`。

## 関連項目

- [022. 件数と頻度](topic-022.md)
- [048. 結合キーの検証](../roadmap.md)
