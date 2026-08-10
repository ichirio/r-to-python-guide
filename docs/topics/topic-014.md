# 014. 分割と連結 — `strsplit` / `str_split` → `split` / `join`

!!! abstract "この項目の R→Python 対応"
    - **R**: `strsplit()`（**リストを返す**）／`str_split()` / `str_split_fixed()`（行列）／連結は `paste(collapse=)`
    - **Python（推奨）**: `str.split()`（リスト）／`"sep".join(...)`（連結）／列は pandas `str.split(expand=True)`
    - **要注意**: R の `strsplit` は**必ずリスト**を返す（1要素でも）。取り出しに `[[1]]` が要る

---

## R ではこう書く

```r
library(stringr)

strsplit("a,b,c", ",")               # リストで返る
paste(c("a","b","c"), collapse = "+")# 連結（collapse）
str_split_fixed("a,b,c", ",", 3)     # 固定数で列に展開（行列）
```

出力:

```text
[[1]]
[1] "a" "b" "c"

a+b+c
     [,1] [,2] [,3]
[1,] "a"  "b"  "c"
```

!!! note "使い分け"
    - `strsplit()`：base。**リスト**が返るので、1本なら `strsplit(x, ",")[[1]]` で取り出す。
    - `str_split()`：stringr。同じくリスト。`simplify = TRUE` で行列化。
    - `str_split_fixed(x, sep, n)`：**列数を固定**して行列に。データフレームの列分割の下ごしらえに便利。
    - 連結（畳む）は `paste(v, collapse=)`（→ [006](topic-006.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd

    print("a,b,c".split(","))          # リスト
    print("+".join(["a", "b", "c"]))   # 連結（collapse 相当）

    # 列を分割して複数列に展開（str_split_fixed 相当）
    s = pd.Series(["a,b,c"])
    print(s.str.split(",", expand=True).values.tolist())
    ```

    出力:

    ```text
    ['a', 'b', 'c']
    a+b+c
    [['a', 'b', 'c']]
    ```

=== "polars"

    ```python
    import polars as pl
    df = pl.DataFrame({"x": ["a,b,c", "d,e,f"]})
    # リスト列にする
    print(df.select(pl.col("x").str.split(",")))
    # 固定数で列展開
    print(df.select(pl.col("x").str.split(",").list.to_struct()).unnest("x"))
    ```

!!! tip "実務ではこれ"
    - **1 本の文字列** → `str.split(sep)`（区切りなしの空白分割は引数なしの `split()`）。
    - **列を複数列へ** → pandas `str.split(sep, expand=True)`（→ [038](../roadmap.md)）。
    - **連結（畳む）** → `"sep".join(iterable)`。`+` の繰り返しより速く読みやすい。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 分割（1本） | `strsplit(x, ",")[[1]]` | `x.split(",")` | `s.str.split(",")` |
| 空白で分割 | `strsplit(x, "\\s+")` | `x.split()` | `s.str.split(" ")` |
| 列を複数列に | `str_split_fixed(x, ",", n)` | `s.str.split(",", expand=True)` | `.str.split(",").list.to_struct()` |
| 連結（畳む） | `paste(v, collapse="+")` | `"+".join(v)` | `s.str.join("+")`（リスト列） |
| n 個で制限 | `str_split(x, ",", n=2)` | `x.split(",", 1)` | `s.str.splitn(",", 2)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`strsplit` はリスト**：R は1本でもリストで返るので `[[1]]` が要る。Python の `str.split` は**そのままリスト**。
    - **分割数の指定**：R の `n=` は「最大ピース数」。Python `str.split(sep, maxsplit)` の `maxsplit` は**分割回数**（ピース数は +1）。オフバイワンに注意。
    - **空文字の扱い**：`"a,,b".split(",")` は Python で `['a','','b']`（空要素を残す）。R の `strsplit` も同様に残す。連続空白を無視したいなら Python は引数なし `split()`。
    - **連結の型**：`join` の対象は**文字列の反復可能**。数値リストは `"".join(map(str, v))` と文字列化が必要。

## 関連項目

- [006. 文字列結合](topic-006.md)
- [038. 列の分割（separate）](../roadmap.md)
- [040. 行の展開（separate_rows）](../roadmap.md)
