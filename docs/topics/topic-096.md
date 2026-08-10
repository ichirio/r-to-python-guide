# 096. CSV / 固定長 / 区切りの読み書き — `readr` → pandas / polars

!!! abstract "この項目の R→Python 対応"
    - **R**: `readr::read_csv()` / `read_delim()` / `read_fwf()`、`write_csv()`
    - **Python（推奨）**: pandas **`read_csv()`** / `to_csv()`、固定長は `read_fwf()`；polars `read_csv`
    - **要注意**: 型推論・欠損文字列・文字コードの既定が R と違う。**キー列の型と `na_values` を明示**

データ入出力の基本。読み込み時の型・欠損・エンコーディングでハマりやすい所を押さえます。

---

## R ではこう書く

```r
library(readr)
df <- read_csv("data.csv",
               col_types = cols(subjid = col_character()),   # 型を固定
               na = c("", "NA", "."))
write_csv(df, "out.csv")

read_fwf("fixed.txt", fwf_widths(c(3, 8, 2)))   # 固定長
```

!!! note "R の勘所"
    - `read_csv` は型を推論。`col_types` で固定（ID の先頭ゼロ落ち防止に `col_character()`）。
    - `na =` で欠損とみなす文字列。
    - 固定長は `read_fwf`（幅 or 位置）。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd
    df = pd.read_csv("data.csv",
                     dtype={"subjid": str},               # 先頭ゼロを保つ
                     na_values=["", "NA", "."],
                     encoding="utf-8")
    df.to_csv("out.csv", index=False)                     # index=False 必須級

    # 固定長
    pd.read_fwf("fixed.txt", widths=[3, 8, 2])
    ```

    出力（round trip 確認）:

    ```text
    [{'subjid': '01', 'val': 1.5}, {'subjid': '02', 'val': 2.5}]
    ```

=== "polars"

    ```python
    import polars as pl
    pl.read_csv("data.csv", dtypes={"subjid": pl.Utf8},
                null_values=["", "NA", "."])
    ```

!!! tip "実務ではこれ"
    - **ID 列は `dtype={"subjid": str}`**。数値推論で `01` → `1` と先頭ゼロが落ちるのを防ぐ（R の `col_character()`）。
    - **`na_values`** で欠損文字列を明示（`"NA"`, `""`, `"."` など。SAS 由来の `.` に注意）。
    - **`to_csv(index=False)`**：付けないと行 index が余分な列として出る。
    - **エンコーディング**：日本語 CSV は `encoding="utf-8"` or `"cp932"`（Excel 由来は cp932/shift_jis のことが多い）。

---

## 対応早見表

| やりたいこと | R（readr） | Python（pandas） |
|---|---|---|
| CSV 読み | `read_csv()` | `pd.read_csv()` |
| 型を固定 | `col_types = cols(...)` | `dtype={...}` |
| 欠損文字列 | `na =` | `na_values=` |
| 区切り指定 | `read_delim(delim=)` | `read_csv(sep=)` |
| 固定長 | `read_fwf()` | `pd.read_fwf(widths=)` |
| 書き出し | `write_csv()` | `to_csv(index=False)` |
| 文字コード | `locale(encoding=)` | `encoding=` |

## つまずきポイント

!!! warning "R と Python の差"
    - **先頭ゼロ落ち**：ID を数値推論すると `007`→`7`。`dtype=str` で防ぐ。R の `col_character()` と同じ発想。
    - **`index=False` 忘れ**：pandas `to_csv` は既定で index を書く。R の `write_csv` は行名を書かないので、揃えるなら `index=False`。
    - **欠損文字列の既定**：pandas は `"NA"`, `"N/A"`, 空文字などを既定で欠損化する（`keep_default_na`）。SAS の `.` は既定では欠損にならないので `na_values` に追加。
    - **エンコーディング**：既定 UTF-8。Windows/Excel 由来は cp932 のことが多く、文字化けの主因。
    - **区切り自動判定**：`sep=None, engine="python"` で推定できるが、明示が安全。

## 関連項目

- [028. 型変換](topic-028.md)
- [097. SAS / SPSS / Stata の読み書き](topic-097.md)
- [077. Excel 出力](topic-077.md)
