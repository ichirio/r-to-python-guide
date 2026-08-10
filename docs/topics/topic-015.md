# 015. 大文字小文字・前後空白 — `toupper` / `str_trim` → `upper` / `strip`

!!! abstract "この項目の R→Python 対応"
    - **R**: `toupper()` / `tolower()`、`str_to_title()`、`trimws()` / `str_trim()`、`str_squish()`
    - **Python（推奨）**: `str.upper()` / `str.lower()` / `str.title()`、`str.strip()`、内部空白の圧縮は `" ".join(x.split())`
    - **要注意**: 大文字小文字化は**ロケール依存**。トルコ語の I 問題などは Python も R も注意

---

## R ではこう書く

```r
library(stringr)

toupper("abc"); tolower("ABC")
str_to_title("hello world")   # 各単語の頭を大文字
trimws("  x  ")               # 前後の空白除去
str_squish("a   b   c")       # 前後除去＋内部の連続空白を1つに
```

出力:

```text
ABC
abc
Hello World
x
a b c
```

!!! note "使い分け"
    - `trimws()`（base）/ `str_trim()`（stringr）：**前後**の空白。`which=` で左右指定。
    - `str_squish()`：前後を除き、**内部の連続空白も 1 個に**畳む。CSV 由来のガタガタ空白の正規化に便利。
    - タイトルケースは `str_to_title()`（stringr）。base には `tools::toTitleCase()`。

---

## Python ではこう書く

=== "pandas"

    ```python
    print("abc".upper(), "ABC".lower())
    print("hello world".title())          # 各単語の頭を大文字
    print("[" + "  x  ".strip() + "]")     # 前後の空白除去
    print("[" + " ".join("a   b   c".split()) + "]")  # squish 相当

    # 列に対して
    import pandas as pd
    s = pd.Series(["  Ethanol  ", "  water "])
    print(s.str.strip().str.upper().tolist())
    ```

    出力:

    ```text
    ABC abc
    Hello World
    [x]
    [a b c]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series(["  Ethanol  ", "  water "])
    print(s.str.strip_chars().str.to_uppercase().to_list())
    print(s.str.to_titlecase().to_list())
    ```

!!! tip "実務ではこれ"
    - 前後空白は `str.strip()`（`lstrip`/`rstrip` で片側）。列は `s.str.strip()`。
    - **内部の連続空白を1個に** → `" ".join(x.split())`。R の `str_squish` に一撃で対応。
    - キー照合の前処理は `s.str.strip().str.upper()` をまとめて。**大小・前後空白を正規化してから join** すると事故らない。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 大文字 | `toupper(x)` | `x.upper()` / `s.str.upper()` | `s.str.to_uppercase()` |
| 小文字 | `tolower(x)` | `x.lower()` | `s.str.to_lowercase()` |
| タイトル | `str_to_title(x)` | `x.title()` | `s.str.to_titlecase()` |
| 前後空白除去 | `trimws(x)` / `str_trim` | `x.strip()` | `s.str.strip_chars()` |
| 左/右のみ | `trimws(x, "left")` | `x.lstrip()` / `x.rstrip()` | `strip_chars_start/_end` |
| 内部空白圧縮 | `str_squish(x)` | `" ".join(x.split())` | `s.str.replace_all(r"\s+"," ")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`title()` の癖**：Python の `str.title()` はアポストロフィ等で妙な区切り方をする（`"it's"` → `"It'S"`）。人名・薬剤名は注意。R の `str_to_title` も同種の癖あり。
    - **strip の引数は「文字集合」**：`x.strip("0")` は「両端から文字 0 を剥がす」。R の `trimws` は空白専用なので、引数の意味が違う。
    - **ロケール依存**：大文字小文字化は言語設定に依存し得る（トルコ語 I）。厳密な照合は `str.casefold()` を検討。
    - **全角空白**：`strip()` の既定は Unicode 空白を落とすが、環境により全角空白（U+3000）の扱いが揺れることがある。明示的に対象へ含める。

## 関連項目

- [011. 文字列の抽出・長さ](topic-011.md)
- [020. 正規表現の違い](topic-020.md)
