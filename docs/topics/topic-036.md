# 036. 縦持ち化 — `pivot_longer` → `melt`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `pivot_longer()`（横広 → 縦長。旧 `gather()`）
    - **Python（推奨）**: pandas **`melt()`**；polars **`unpivot()`**
    - **要注意**: 「残す列（id）」と「溶かす列（value）」の指定を取り違えない。names/values の列名指定が対応

wk1, wk2… のような**測定時点が列**のデータを、1 行 1 観測の縦長へ。解析・作図の下ごしらえ。

---

## R ではこう書く

```r
library(tidyr)
wide <- tibble(subjid = c("01","02"), wk1 = c(10,20), wk2 = c(12,19))

wide |> pivot_longer(cols = c(wk1, wk2),
                     names_to = "week", values_to = "val")
```

出力:

```text
  subjid week    val
  01     wk1      10
  01     wk2      12
  02     wk1      20
  02     wk2      19
```

!!! note "R の勘所"
    - `cols=`：溶かす列（tidy-select、`starts_with("wk")` 等）。
    - `names_to` / `values_to`：新しい「キー列名」「値列名」。
    - 列名に情報が埋まっているなら `names_sep=` / `names_pattern=` で分解しながら展開できる。

---

## Python ではこう書く

=== "pandas"

    ```python
    wide = pd.DataFrame({"subjid":["01","02"], "wk1":[10,20], "wk2":[12,19]})

    long = wide.melt(id_vars="subjid",
                     value_vars=["wk1", "wk2"],
                     var_name="week", value_name="val")
    ```

    出力（`sort_values(["subjid","week"])`）:

    ```text
    subjid week  val
        01  wk1   10
        01  wk2   12
        02  wk1   20
        02  wk2   19
    ```

=== "polars"

    ```python
    import polars as pl
    widep = pl.DataFrame({"subjid":["01","02"], "wk1":[10,20], "wk2":[12,19]})
    widep.unpivot(index="subjid", on=["wk1","wk2"],
                  variable_name="week", value_name="val")
    ```

!!! tip "実務ではこれ"
    - **`melt(id_vars=, value_vars=, var_name=, value_name=)`** を丸ごと `pivot_longer` の対応として覚える。
        - `id_vars` = 残す列（`pivot_longer` で溶かさない列）
        - `value_vars` = 溶かす列（`cols=`）
    - `value_vars` を省くと `id_vars` 以外を全部溶かす。
    - 溶かした後は並びが崩れるので、必要なら `sort_values`。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 縦持ち化 | `pivot_longer(cols=)` | `melt(value_vars=)` | `unpivot(on=)` |
| 残す列 | 指定しない列 | `id_vars=` | `index=` |
| キー列名 | `names_to=` | `var_name=` | `variable_name=` |
| 値列名 | `values_to=` | `value_name=` | `value_name=` |
| 列名を分解 | `names_sep=` / `names_pattern=` | melt 後に `str.split` | 同左 |

## つまずきポイント

!!! warning "R と Python の差"
    - **id と value の指定**：`melt` は `id_vars`（残す）を主に指定する発想。`pivot_longer` は `cols`（溶かす）を指定する発想。逆側を書くので注意。
    - **列名の型**：溶かした後の `week` 列は文字列（`"wk1"`）。数値の週番号が要るなら `str.extract` や `str.replace` で数値化。
    - **欠損の増殖**：横広に空セルがあると縦長で NaN 行が増える。`dropna=` 相当は melt 後に `.dropna(subset=["val"])`。
    - **複数の値ブロック**：`pivot_longer(names_pattern=...)` で `val_wk1, se_wk1` のような多値を一度に展開できるが、pandas は `pd.wide_to_long` か melt 複数回で対応。

## 関連項目

- [037. 横持ち化（pivot_wider）](topic-037.md)
- [050. ロング／ワイドを行き来する実務パターン](../roadmap.md)
