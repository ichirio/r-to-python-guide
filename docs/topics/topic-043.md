# 043. アンチ・セミ結合 — `anti_join` / `semi_join` → indicator / `isin`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `anti_join()`（右に**ない**行）／`semi_join()`（右に**ある**行、列は増やさない）
    - **Python（推奨）**: `isin()` で絞る（**列を増やさない**フィルタ）／`merge(indicator=True)` で由来判定
    - **要注意**: `semi_join` は「絞るだけで結合しない」。pandas の `merge` は列が増えるので `isin` が対応

「lb に測定がない被験者」「登録済みだけ残す」など、**キーの有無で行を絞る**操作。

（例: `dm2`=01,02,03 / `lb2`=02,03,04）

---

## R ではこう書く

```r
library(dplyr)

dm2 |> anti_join(lb2, by = "subjid")   # lb2 に「ない」→ 01
dm2 |> semi_join(lb2, by = "subjid")   # lb2 に「ある」→ 02, 03
```

出力:

```text
# anti_join → 01
# semi_join → 02, 03
```

!!! note "R の勘所"
    - `semi_join`：右に一致がある左の行を残す。**右の列は付かない**（フィルタ専用）。
    - `anti_join`：右に一致が**ない**左の行。欠測チェック・突合の定番。
    - どちらも右キーの重複で行が増えない（フィルタなので）。

---

## Python ではこう書く

=== "pandas"

    ```python
    keys = lb2["subjid"]

    # semi_join：右にある行だけ（列は増やさない）
    dm2[dm2["subjid"].isin(keys)]

    # anti_join：右にない行
    dm2[~dm2["subjid"].isin(keys)]

    # indicator を使う手（複数キーや厳密な由来判定に）
    m = dm2.merge(lb2, on="subjid", how="left", indicator=True)
    anti = m[m["_merge"] == "left_only"][dm2.columns]
    ```

    出力:

    ```text
    # isin → semi: 02, 03 ／ anti: 01
    ```

=== "polars"

    ```python
    import polars as pl
    dm2p.join(lb2p, on="subjid", how="semi")   # polars は専用の how がある！
    dm2p.join(lb2p, on="subjid", how="anti")
    ```

!!! tip "実務ではこれ"
    - **単一キー** → `isin()` が最短・最速。`~` で否定＝anti。
    - **複数キー**や**重複を厳密に**扱うなら `merge(how="left", indicator=True)` の `_merge` 列で `left_only`（anti）/`both`（semi）を選ぶ。
    - **polars は `how="semi"` / `"anti"` を直接サポート**。dplyr にいちばん近い。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 右にある行だけ | `semi_join(x, y)` | `x[x[k].isin(y[k])]` | `x.join(y, how="semi")` |
| 右にない行 | `anti_join(x, y)` | `x[~x[k].isin(y[k])]` | `x.join(y, how="anti")` |
| 複数キーで | `by=c("a","b")` | `merge(indicator=True)` で判定 | `on=["a","b"]` |
| 由来を列に | — | `indicator=True`（`_merge`） | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **`isin` は単一列向け**：複数キーの anti/semi は `isin` だけでは書けない。タプル化するか `merge(indicator=True)` を使う。
    - **列が増えない**のが semi の要件：`merge` で書くと右の列が付く／行が重複しうるので、フィルタなら `isin` が正解。
    - **欠損キー**：`NaN` は `isin` で一致しない（`NaN != NaN`）。キーに欠損があると anti 側に回る点に注意。
    - **`indicator` の後始末**：`_merge` 列と右由来の列が残るので、`[dm2.columns]` で元の列だけに戻す。

## 関連項目

- [042. 左・外部結合](topic-042.md)
- [048. 結合キーの検証](topic-048.md)
- [008. 行フィルタ（filter）](topic-008.md)
