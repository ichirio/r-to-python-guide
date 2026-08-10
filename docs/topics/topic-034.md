# 034. ビン分割 — `cut` → `pd.cut` / `qcut`

!!! abstract "この項目の R→Python 対応"
    - **R**: `cut(x, breaks, labels, right=)`（境界で区切る）／`Hmisc::cut2` や `ntile()`（等分位）
    - **Python（推奨）**: 境界指定は **`pd.cut()`**、等分位は **`pd.qcut()`**
    - **要注意**: 区間の**開閉**（`right=`）の既定が R と pandas で逆。境界値の所属がずれる

年齢群・検査値カテゴリなど、連続値をカテゴリに畳む定番。

（例: `age = 45, 52, 38, 61, NA` を `<40 / 40-59 / 60+` に）

---

## R ではこう書く

```r
age <- c(45, 52, 38, 61, NA)

cut(age,
    breaks = c(0, 40, 60, Inf),
    labels = c("<40", "40-59", "60+"),
    right = FALSE)   # [0,40) [40,60) [60,Inf)
```

出力:

```text
[1] 40-59 40-59 <40   60+   <NA>
Levels: <40 40-59 60+
```

!!! note "R の勘所"
    - `right = TRUE`（既定）は **(lo, hi]（右閉）**、`right = FALSE` は **[lo, hi)（左閉）**。
    - `Inf`/`-Inf` で開区間の端を作る。ラベルは `labels=`。
    - **等分位**（各群が同数）は `dplyr::ntile(x, 4)` や `Hmisc::cut2(x, g=4)`。

---

## Python ではこう書く

=== "pandas"

    ```python
    import numpy as np, pandas as pd
    age = pd.Series([45, 52, 38, 61, np.nan])

    # 境界指定（右開＝ R の right=FALSE に合わせる）
    pd.cut(age, bins=[0, 40, 60, np.inf],
           labels=["<40", "40-59", "60+"], right=False)

    # 等分位（各群が同数に）
    pd.qcut(pd.Series([1,2,3,4,5,6,7,8]), q=4, labels=["Q1","Q2","Q3","Q4"])
    ```

    出力（`pd.cut`）:

    ```text
    ['40-59', '40-59', '<40', '60+', NaN]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series([45, 52, 38, 61, None])
    print(s.cut(breaks=[40, 60], labels=["<40", "40-59", "60+"]).to_list())
    # 等分位は qcut
    ```

    polars の `cut` は**内側の境界だけ**（`[40, 60]`）を渡し、両端は自動。

!!! tip "実務ではこれ"
    - **決まった境界**（年齢層など）→ `pd.cut(bins=[...], right=False, labels=[...])`。R と揃えるなら `right=False`。
    - **各群を同数に**（四分位群など）→ `pd.qcut(x, q=4)`。
    - 結果は Categorical（順序付き）。並び順やラベルはそのまま TFL に使える（→ [091](../roadmap.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 境界で分割 | `cut(x, breaks, labels)` | `pd.cut(x, bins, labels)` |
| 左閉区間 [lo,hi) | `cut(..., right=FALSE)` | `pd.cut(..., right=False)` |
| 右閉区間 (lo,hi] | `cut(...)`（既定） | `pd.cut(...)`（既定） |
| 等分位で分割 | `ntile(x, 4)` / `cut2(x, g=4)` | `pd.qcut(x, 4)` |
| 開区間の端 | `Inf` / `-Inf` | `np.inf` / `-np.inf` |
| 群の数だけ返す | — | `pd.cut(..., labels=False)`（整数コード） |

## つまずきポイント

!!! warning "R と Python の差"
    - **区間の開閉の既定**：R `cut` は既定 `right=TRUE`（右閉）、pandas `pd.cut` も既定 `right=True`。**ただし境界値がちょうどの症例**（例: age=40）は開閉で群が変わる。仕様に合わせて `right=` を明示し、境界値のデータで必ず検算。
    - **境界の渡し方**：pandas は**全境界**（`[0,40,60,inf]`）、polars は**内側だけ**（`[40,60]`）。混同しやすい。
    - **範囲外は NaN**：`bins` の外側の値は `pd.cut` で NaN。R も `cut` で範囲外は NA。端は `-inf/inf` で受ける。
    - **欠損**：NA/NaN はどちらも欠損カテゴリ（`<NA>`/NaN）のまま。

## 関連項目

- [026. 条件で値を作る（case_when）](topic-026.md)
- [091. 因子とラベル](../roadmap.md)
