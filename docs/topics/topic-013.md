# 013. パターン検出・抽出 — `grepl` / `str_detect` / `str_extract` → `str.contains` / `str.extract`

!!! abstract "この項目の R→Python 対応"
    - **R**: `grepl()` / `str_detect()`（含むか＝TRUE/FALSE）、`grep()`（位置）、`str_extract()`（取り出す）
    - **Python（推奨）**: 列は pandas **`str.contains()` / `str.extract()`**；単発は **`re.search()` / `re.findall()`**
    - **要注意**: `grep()` は**位置（index）**、`grepl()` は**論理値**。Python 側の対応関数が別

---

## R ではこう書く

```r
library(stringr)
v <- c("SUBJ-AE", "SUBJ-CM")

grepl("AE", v)            # 含むか（論理値）
str_detect(v, "AE")       # 同上（stringr）
grep("AE", v)             # 含む要素の位置
str_extract("visit-12", "[0-9]+")  # 最初のマッチを取り出す
```

出力:

```text
[1]  TRUE FALSE
[1]  TRUE FALSE
[1] 1
12
```

!!! note "使い分け"
    - `grepl()` / `str_detect()`：**論理ベクトル**。`filter(str_detect(term, "^AE"))` のように抽出条件へ。
    - `grep()`：**位置**（`value=TRUE` で値そのもの）。
    - `str_extract()` / `str_extract_all()`：**マッチ文字列**を取り出す。捕捉群は `str_match()`。

---

## Python ではこう書く

=== "pandas"

    ```python
    import re
    import pandas as pd
    s = pd.Series(["SUBJ-001-AE", "SUBJ-002-CM"])

    print(s.str.contains("AE").tolist())          # 含むか（str_detect）
    print(s[s.str.contains("AE")].tolist())       # 位置ではなく行抽出が自然
    print(s.str.extract(r"([0-9]+)")[0].tolist()) # 最初の数字列を取り出す

    print(re.search(r"[0-9]+", "visit-12").group())  # 単発の抽出
    ```

    出力:

    ```text
    [True, False]
    ['SUBJ-001-AE']
    ['001', '002']
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series(["SUBJ-001-AE", "SUBJ-002-CM"])
    print(s.str.contains("AE").to_list())              # 含むか
    print(s.str.extract(r"([0-9]+)", 1).to_list())     # 群1を取り出す
    print(s.str.extract_all(r"[0-9]+").to_list())      # すべて
    ```

!!! tip "実務ではこれ"
    - 「含むか」で**行を絞る**なら pandas `df[df["term"].str.contains("^AE")]`（→ [008](topic-008.md)）。R の `filter(grepl(...))` と同じ発想。
    - 数値・コードの**取り出し**は `str.extract(r"(...)")`（捕捉群が必須）。`str.contains` は `regex=True` が既定なので、リテラルなら `regex=False`。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 含むか（論理） | `grepl(p, x)` / `str_detect` | `s.str.contains(p)` | `s.str.contains(p)` |
| 含む行を抽出 | `filter(grepl(p, col))` | `df[df[c].str.contains(p)]` | `df.filter(pl.col(c).str.contains(p))` |
| 位置 | `grep(p, x)` | `s[s.str.contains(p)].index` | — |
| 取り出す（最初） | `str_extract(x, p)` | `s.str.extract(r"(p)")` | `s.str.extract(p,1)` |
| 取り出す（全部） | `str_extract_all` | `s.str.findall(p)` | `s.str.extract_all(p)` |
| 先頭一致 | `startsWith` / `^` | `s.str.startswith` | `s.str.starts_with` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`grep` は位置、`grepl` は論理**。Python では「論理→`contains`」「位置→`s[...].index`」と対応先が分かれる。
    - **捕捉群が必要**：pandas `str.extract` は**丸括弧の群**がないと `NaN`。R の `str_extract` は群なしでもマッチ全体を返すので癖が違う。
    - **`contains` の既定は正規表現**：`str.contains(".")` は「任意1文字」でほぼ全 True。リテラルは `regex=False`。
    - **欠損**：欠損行は `contains` が `NaN` を返し、そのままだと boolean 抽出で `NA` 扱い → `na=False` を付けると安全。

## 関連項目

- [012. 検索と置換](topic-012.md)
- [008. 行フィルタ（filter）](topic-008.md)
- [020. 正規表現の違い](topic-020.md)
