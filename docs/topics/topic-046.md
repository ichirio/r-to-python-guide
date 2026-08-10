# 046. 欠けた組合せを補完 — `complete` / `expand` → `reindex`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `complete()`（現れていない組合せの行を作り、値を埋める）
    - **Python（推奨）**: **`set_index(keys).reindex(MultiIndex.from_product(...), fill_value=)`**
    - **要注意**: 「0 件を 0 と出す」ための操作。集計してから枠を埋める順番を守る

集計結果に**存在しなかったカテゴリ**（0 件の群など）を明示的に 0 行として出す。TFL で「空欄でなく 0」を出す定番。

---

## R ではこう書く

```r
library(tidyr)
part <- tibble(subjid = c("01","01","02"), visit = c(1,2,1), val = c(10,12,20))

part |> complete(subjid, visit, fill = list(val = 0))
```

出力:

```text
  subjid visit   val
  01         1    10
  01         2    12
  02         1    20
  02         2     0    ← 元になかった (02, visit2) が 0 で追加
```

!!! note "R の勘所"
    - `complete(a, b)`：a×b の**現れた値**の全組合せを作り、無かった行を追加。
    - `fill = list(col = 0)` で新規行の値を指定（既定は NA）。
    - 特定の水準まで含めたいなら `complete(a, b = 1:4)` のように明示。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd
    part = pd.DataFrame({"subjid":["01","01","02"], "visit":[1,2,1], "val":[10,12,20]})

    full = pd.MultiIndex.from_product(
        [part["subjid"].unique(), part["visit"].unique()],
        names=["subjid", "visit"])

    comp = (part.set_index(["subjid","visit"])
                .reindex(full, fill_value=0)
                .reset_index())
    print(comp)
    ```

    出力:

    ```text
    subjid  visit  val
        01      1   10
        01      2   12
        02      1   20
        02      2    0
    ```

=== "polars"

    ```python
    import polars as pl
    partp = pl.DataFrame({"subjid":["01","01","02"], "visit":[1,2,1], "val":[10,12,20]})
    # 全枠を作って左結合＋fill
    frame = (partp.select("subjid").unique()
                  .join(partp.select("visit").unique(), how="cross"))
    frame.join(partp, on=["subjid","visit"], how="left").with_columns(
        pl.col("val").fill_null(0))
    ```

!!! tip "実務ではこれ"
    - **枠を作って左結合**（→ [045](topic-045.md)）＋ `fillna(0)` が最も分かりやすい。`reindex` は同じことを index 操作でやる版。
    - **カテゴリの母集団を固定**したいときは `unique()` ではなく**規定の水準リスト**を使う（データに無い群も出す）。Categorical の水準（→ [091](../roadmap.md)）を使うと自動化できる。
    - 0 件を出すか空欄かは表の規約次第。`fill_value` を変えるだけで切替。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） |
|---|---|---|
| 現れた値の全組合せを補完 | `complete(a, b)` | `reindex(MultiIndex.from_product([...]))` |
| 新規行の値 | `fill = list(x = 0)` | `fill_value=0` |
| 水準を明示 | `complete(a, b = 1:4)` | 明示の list を product に渡す |
| 枠＋左結合方式 | `expand()` → `left_join` | `product` 枠 → `merge(how="left")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **「現れた値」か「規定の水準」か**：`unique()` はデータに出た値だけ。0 件の群（誰も割付かなかった用量など）を出すには**規定水準のリスト**を使う。Categorical の全水準を使うのが堅い。
    - **集計との順序**：普通は「集計 → complete で 0 埋め」。逆にすると 0 行を平均に含めてしまう。
    - **fill 対象の列**：`reindex` の `fill_value` は全列一律。列ごとに違う値なら reindex 後に `fillna({...})`。
    - **多キーの直積**：キーが多いと組合せが急増。必要な軸だけで complete する。

## 関連項目

- [045. クロス結合・総当たり](topic-045.md)
- [069. シフトテーブル](../roadmap.md)
- [091. 因子とラベル](../roadmap.md)
