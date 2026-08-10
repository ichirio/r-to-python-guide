# 037. 横持ち化 — `pivot_wider` → `pivot` / `pivot_table`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `pivot_wider()`（縦長 → 横広。旧 `spread()`）
    - **Python（推奨）**: 一意キーなら **`pivot()`**、集約が要るなら **`pivot_table()`**；polars **`pivot()`**
    - **要注意**: キーが重複すると `pivot` はエラー。**集計しながら広げる**なら `pivot_table`（R は `values_fn=`）

TFL の「行＝カテゴリ、列＝群」のレイアウトはこれ。縦持ちの集計結果を表の形へ。

---

## R ではこう書く

```r
library(tidyr)
long <- tibble(subjid = c("01","01","02","02"),
               week = c("wk1","wk2","wk1","wk2"),
               val = c(10,12,20,19))

long |> pivot_wider(names_from = week, values_from = val)
```

出力:

```text
  subjid   wk1   wk2
  01        10    12
  02        20    19
```

!!! note "R の勘所"
    - `names_from`：列名になる変数。`values_from`：セルの値。
    - キーが重複する（同じ subjid×week が複数）なら **`values_fn = mean`** 等で集約。
    - 埋まらないセルは `values_fill =` で既定値。

---

## Python ではこう書く

=== "pandas"

    ```python
    long = pd.DataFrame({"subjid":["01","01","02","02"],
                         "week":["wk1","wk2","wk1","wk2"], "val":[10,12,20,19]})

    # キーが一意なら pivot
    wide = long.pivot(index="subjid", columns="week", values="val").reset_index()
    wide.columns.name = None                       # 列の名前ラベルを消す

    # 集約が要るなら pivot_table
    long.pivot_table(index="subjid", columns="week", values="val",
                     aggfunc="mean", fill_value=0).reset_index()
    ```

    出力（`pivot`）:

    ```text
    subjid  wk1  wk2
        01   10   12
        02   20   19
    ```

=== "polars"

    ```python
    import polars as pl
    longp = pl.DataFrame({"subjid":["01","01","02","02"],
                          "week":["wk1","wk2","wk1","wk2"], "val":[10,12,20,19]})
    longp.pivot(on="week", index="subjid", values="val", aggregate_function="mean")
    ```

!!! tip "実務ではこれ"
    - **キーが一意**（1 subjid×week に1値）→ `pivot(index=, columns=, values=)`。
    - **同キーが複数あり集計が必要**（例: 群×カテゴリの件数）→ `pivot_table(aggfunc=)`。デモグラ/AE 表の骨格。
    - `pivot` 後は列が MultiIndex/`columns.name` を持つので、`reset_index()` と `columns.name = None` で整える。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 横持ち（一意） | `pivot_wider()` | `pivot(index, columns, values)` | `pivot(on, index, values)` |
| 集約しながら | `pivot_wider(values_fn=)` | `pivot_table(aggfunc=)` | `pivot(aggregate_function=)` |
| 埋め値 | `values_fill=` | `fill_value=` | — |
| 列名の接頭辞 | `names_prefix=` | 後で `add_prefix` | — |
| 複数値列 | 複数 `values_from` | `values=[...]`（MultiIndex 列） | 複数 values |

## つまずきポイント

!!! warning "R と Python の差"
    - **重複キーでエラー**：pandas `pivot` は index×columns が重複すると `ValueError`。集計するなら `pivot_table`。R も `pivot_wider` が重複時に警告＋リストセルになるので、`values_fn=` を付ける。
    - **列名の後始末**：pandas `pivot` の結果は `columns.name`（例 "week"）が残り、列が階層化することも。`reset_index()` と `columns.name = None`、多層なら列名を平坦化。
    - **欠損セル**：組が揃わないと NaN。`fill_value=` / `values_fill=` で 0 埋め等。件数表では 0 埋めが普通。
    - **集計関数の既定**：`pivot_table` の既定 `aggfunc="mean"`。件数表なら `aggfunc="size"`/`"count"` を明示。

## 関連項目

- [036. 縦持ち化（pivot_longer）](topic-036.md)
- [054. クロス集計表](../roadmap.md)
- [050. ロング／ワイドを行き来する実務パターン](../roadmap.md)
