# 012. 検索と置換 — `gsub` / `sub` / `str_replace` → `str.replace` / `re.sub`

!!! abstract "この項目の R→Python 対応"
    - **R**: `sub()`（最初の1つ）／`gsub()`（全部）／`str_replace()` / `str_replace_all()`。**既定で正規表現**
    - **Python（推奨）**: 文字どおりの置換は **`str.replace()`**（リテラル）、正規表現は **`re.sub()`**；列は pandas `.str.replace(..., regex=)`
    - **要注意**: R の `gsub` は**パターンが正規表現**、Python の `str.replace` は**リテラル**。前提が逆

---

## R ではこう書く

```r
library(stringr)

gsub("-", "_", "A-B-C")            # すべて置換
sub("-", "_", "A-B-C")             # 最初だけ
str_replace_all("A-B-C", "-", "_") # stringr：全部
gsub("[0-9]", "#", "a1b2")         # 既定で正規表現
```

出力:

```text
A_B_C
A_B-C
A_B_C
a#b#
```

!!! note "使い分け"
    | 関数 | 範囲 | 正規表現 | リテラル指定 |
    |---|---|---|---|
    | `sub()` | 最初の1つ | 既定で有効 | `fixed = TRUE` |
    | `gsub()` | 全部 | 既定で有効 | `fixed = TRUE` |
    | `str_replace()` | 最初の1つ | 既定で有効 | `fixed("...")` で囲む |
    | `str_replace_all()` | 全部 | 既定で有効 | 同上 |

    `.` や `(` を**そのまま**置換したいのに正規表現扱いされて事故る、が R でありがち。`fixed=TRUE` を忘れない。

---

## Python ではこう書く

=== "pandas"

    ```python
    import re
    import pandas as pd

    # リテラル置換（正規表現ではない）
    print("A-B-C".replace("-", "_"))       # 全部
    print("A-B-C".replace("-", "_", 1))    # 最初の1つだけ（回数指定）

    # 正規表現置換
    print(re.sub(r"[0-9]", "#", "a1b2"))

    # 列に対して（regex= を明示）
    s = pd.Series(["SUBJ-001-AE", "SUBJ-002-CM"])
    print(s.str.replace("-", "_", regex=False).tolist())
    ```

    出力:

    ```text
    A_B_C
    A_B-C
    a#b#
    ['SUBJ_001_AE', 'SUBJ_002_CM']
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series(["A-B-C"])
    print(s.str.replace("-", "_").to_list())        # 最初の1つ（正規表現）
    print(s.str.replace_all("-", "_").to_list())    # 全部
    print(s.str.replace_all("-", "_", literal=True).to_list())  # リテラル
    ```

    polars は `replace`（最初）/ `replace_all`（全部）と R の `sub`/`gsub` に名前が対応。既定は正規表現、`literal=True` でリテラル。

!!! tip "実務ではこれ"
    - **記号をそのまま置換**（`.` `(` `+` など）→ `str.replace`（リテラル）か `regex=False`。R の `fixed=TRUE` に当たる。
    - **パターンで置換** → `re.sub(r"...")`（raw 文字列を使う。→ [020](topic-020.md)）。
    - pandas の `str.replace` は **`regex=` の既定が `False`**（pandas 2.x）。R の gsub の癖でパターンを渡すと効かないので、正規表現なら `regex=True` を明示。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 最初の1つ | `sub(p, r, x)` | `x.replace(p, r, 1)` | `s.str.replace(p, r)` |
| 全部 | `gsub(p, r, x)` | `x.replace(p, r)` | `s.str.replace_all(p, r)` |
| 正規表現で | 既定 | `re.sub(p, r, x)` / `regex=True` | 既定 |
| リテラルで | `fixed=TRUE` | `regex=False`（既定） | `literal=True` |

## つまずきポイント

!!! warning "R と Python の差"
    - **既定の前提が逆**：R の `gsub` はパターンが正規表現。Python の `str.replace` はリテラル。移植時は「どちらのつもりか」を毎回確認。
    - **後方参照の記法**：R は置換文字列で `\\1`、Python `re.sub` は `\1`（raw なら `r"\1"`）または `\g<1>`。
    - **回数指定の向き**：R は `sub`（1回）/`gsub`（全部）と関数が分かれる。Python `str.replace(p, r, count)` は**回数**を数値で渡す（`count` 省略で全部）。
    - **NA/欠損**：R の `str_replace` は NA を NA のまま返す。pandas も欠損行は欠損のまま。

## 関連項目

- [013. パターン検出・抽出](topic-013.md)
- [020. 正規表現の違い](topic-020.md)
