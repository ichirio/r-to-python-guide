# 042. 左・外部結合 — `left_join` / `full_join` → `merge(how=)`

!!! abstract "この項目の R→Python 対応"
    - **R**: `left_join()`（左を全部残す）／`right_join()`／`full_join()`（両方を残す）
    - **Python（推奨）**: pandas **`merge(how="left" / "right" / "outer")`**；polars 同名
    - **要注意**: 相手にない行は欠損になり、整数列が float 化する（pandas）

解析対象（左＝母集団）に、後付けの測定値を貼るのが `left_join`。TFL の基本操作。

（例: `dm2`=01,02,03 / `lb2`=02,03,04）

---

## R ではこう書く

```r
library(dplyr)

dm2 |> left_join(lb2, by = "subjid")   # 左(dm2)を全部残す
dm2 |> full_join(lb2, by = "subjid")   # 両方を残す
```

出力:

```text
# left_join → 01(NA), 02(5), 03(6)
# full_join → 01(NA), 02(5), 03(6), 04(7)
```

!!! note "R の勘所"
    - `left_join`：左の行を全部保持、右にない所は NA。**解析集団を軸に情報を足す**定番。
    - `full_join`：どちらか一方にしかない行も全部残す。
    - `right_join`：右を全部（`left_join` の左右逆）。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm2.merge(lb2, on="subjid", how="left")    # 左を全部
    dm2.merge(lb2, on="subjid", how="outer")   # 両方（full_join）
    dm2.merge(lb2, on="subjid", how="right")   # 右を全部
    ```

    出力（`how="left"`）:

    ```text
    subjid arm  val
        01   A  NaN     ← lb2 に 01 なし
        02   A  5.0
        03   B  6.0
    ```

=== "polars"

    ```python
    import polars as pl
    dm2p.join(lb2p, on="subjid", how="left")
    dm2p.join(lb2p, on="subjid", how="full")     # polars は "full"
    ```

!!! tip "実務ではこれ"
    - **母集団を軸に情報を貼る** → `merge(y, on=key, how="left")`。左の行数が保たれる（重複キーがなければ）。
    - 結合後は**行数が変わっていないか**必ず確認（`len` 比較）。増えていたら右キーの重複を疑う（→ [048](topic-048.md)）。
    - polars の全結合は `how="full"`（pandas は `"outer"`）。名前が違う。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 左を全部 | `left_join(x, y)` | `x.merge(y, how="left")` | `x.join(y, how="left")` |
| 右を全部 | `right_join(x, y)` | `x.merge(y, how="right")` | `x.join(y, how="right")` |
| 両方 | `full_join(x, y)` | `x.merge(y, how="outer")` | `x.join(y, how="full")` |
| 元がどちら由来か | — | `indicator=True`（`_merge` 列） | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **`outer` vs `full`**：pandas は `how="outer"`、polars は `how="full"`。R は `full_join`。
    - **整数の float 化**：左結合で欠損が入ると、右由来の整数列が `float64`（`5` → `5.0`）に。保ちたいなら `astype("Int64")`。
    - **行数の増減**：右キーが重複すると左結合でも行が増える。R も同じ。**結合前後で `nrow` を照合**する習慣を。
    - **キー列の残り方**：`outer` では左右どちらか欠けたキーが混ざる。polars の full は両方のキー列を残すことがある（`coalesce=True` で1本化）。

## 関連項目

- [041. 内部結合](topic-041.md)
- [043. アンチ・セミ結合](topic-043.md)
- [048. 結合キーの検証](topic-048.md)
