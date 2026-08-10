# 011. 文字列の抽出・長さ — `substr` / `str_sub` / `nchar` → slice / `len` / `.str`

!!! abstract "この項目の R→Python 対応"
    - **R**: `nchar()`（長さ）、`substr()` / `str_sub()`（部分抽出、**1 始まり・両端含む**）
    - **Python（推奨）**: `len()` と **スライス `x[a:b]`**（**0 始まり・右端含まず**）／列は pandas `.str[...]`
    - **要注意**: 位置指定が R と 2 か所ずれる（0 始まり・半開区間）

---

## R ではこう書く

```r
library(stringr)
x <- "SUBJ-001-AE"

nchar(x)            # 長さ
substr(x, 1, 4)     # 1〜4文字目（両端含む）
str_sub(x, 1, 4)    # 同上（stringr）
str_sub(x, -2, -1)  # 末尾2文字（負の位置が使える）
substr(x, 6, 8)     # 6〜8文字目
```

出力:

```text
11
SUBJ
SUBJ
AE
001
```

!!! note "使い分け"
    - `substr()` は base、`str_sub()` は stringr。**`str_sub()` は負のインデックス**（末尾から）が使えて便利。
    - どちらも**開始・終了とも「含む」1 始まり**。`str_sub(x, 6, 8)` は 6,7,8 文字目の3文字。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd
    x = "SUBJ-001-AE"

    print(len(x))       # 長さ
    print(x[:4])        # 先頭4文字（index 0..3）
    print(x[-2:])       # 末尾2文字
    print(x[5:8])       # 6〜8文字目 = index 5,6,7

    s = pd.Series(["SUBJ-001-AE", "SUBJ-002-CM"])
    print(s.str[:4].tolist())    # 列に対しても同じスライス
    print(s.str.len().tolist())
    ```

    出力:

    ```text
    11
    SUBJ
    AE
    001
    ['SUBJ', 'SUBJ']
    [11, 11]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series(["SUBJ-001-AE", "SUBJ-002-CM"])
    print(s.str.len_chars().to_list())        # 文字数
    print(s.str.slice(0, 4).to_list())        # offset, length
    print(s.str.slice(5, 3).to_list())        # index5 から3文字 = "001"
    ```

    polars の `str.slice(offset, length)` は**「開始位置と長さ」**指定（R の `substr` の第2引数が長さではない点に注意）。

!!! tip "実務ではこれ"
    - スカラーは素の**スライス `x[a:b]`**。1 個の文字は `x[i]`。
    - 列は pandas **`s.str[a:b]`** / **`s.str.slice()`**。固定長フィールド（SUBJID の桁など）の切り出しに多用。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 長さ | `nchar(x)` | `len(x)` / `s.str.len()` | `s.str.len_chars()` |
| 部分抽出 | `substr(x,1,4)` | `x[0:4]` / `s.str[0:4]` | `s.str.slice(0,4)` |
| 末尾 n 文字 | `str_sub(x,-2,-1)` | `x[-2:]` | `s.str.slice(-2)` |
| 1 文字 | `substr(x,i,i)` | `x[i-1]` | `s.str.slice(i-1,1)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **位置が 2 か所ずれる**：R は「1 始まり・両端含む」、Python スライスは「0 始まり・右端含まず」。`substr(x,6,8)`（3文字）＝ `x[5:8]`。
    - **`substr` の3引数目**：R は「終了位置」、polars `slice` は「長さ」。移植時に取り違えやすい。
    - **文字数 vs バイト数**：日本語など多バイト文字で `nchar(type="bytes")` 相当が要るときは、Python は `len(x.encode("utf-8"))`。通常の `len` は**文字数**（R の既定と同じ）。

## 関連項目

- [006. 文字列結合](topic-006.md)
- [014. 分割と連結](topic-014.md)
- [038. 列の分割（separate）](../roadmap.md)
