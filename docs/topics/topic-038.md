# 038. 列の分割 — `separate` / `separate_wider_*` → `str.split(expand=True)`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `separate()`（旧）／`separate_wider_delim()` / `separate_wider_position()`（新）
    - **Python（推奨）**: pandas **`str.split(sep, expand=True)`**（区切り）／固定幅は `str.slice` を並べる
    - **要注意**: 分割数が行で違うと列数が揺れる。`n=`/`maxsplit`/`expand` の扱いを合わせる

`"A-01"` を `grp="A", num="01"` に割るような、複合コードの分解。

---

## R ではこう書く

```r
library(tidyr)

tibble(x = c("A-01","B-02")) |>
  separate(x, into = c("grp","num"), sep = "-")

# 新 API（推奨）
tibble(x = c("A-01","B-02")) |>
  separate_wider_delim(x, delim = "-", names = c("grp","num"))
```

出力:

```text
  grp   num
  A     01
  B     02
```

!!! note "R の勘所"
    - `separate()` は成熟扱い（superseded）。新規は `separate_wider_delim()`（区切り）/ `separate_wider_position()`（固定幅）/ `separate_wider_regex()`（正規表現）。
    - `sep=` は既定で正規表現。ピース数が合わない行は警告＋ NA/切り捨て。

---

## Python ではこう書く

=== "pandas"

    ```python
    x = pd.DataFrame({"x": ["A-01", "B-02"]})

    parts = x["x"].str.split("-", expand=True)
    parts.columns = ["grp", "num"]
    print(parts)

    # 固定幅で切る場合
    x["grp"] = x["x"].str[0]
    x["num"] = x["x"].str[2:4]

    # 正規表現で名前付き抽出（separate_wider_regex 相当）
    x["x"].str.extract(r"(?P<grp>[A-Z]+)-(?P<num>\d+)")
    ```

    出力:

    ```text
      grp num
    0   A  01
    1   B  02
    ```

=== "polars"

    ```python
    import polars as pl
    df = pl.DataFrame({"x": ["A-01", "B-02"]})
    df.with_columns(
        pl.col("x").str.split_exact("-", 1).struct.rename_fields(["grp","num"])
    ).unnest("x")
    ```

!!! tip "実務ではこれ"
    - **区切りで割る** → `str.split(sep, expand=True)`。列名は後で付ける。
    - **固定幅**（SUBJID の桁など）→ `str.slice`/`str[a:b]` を列ごとに。
    - **構造が決まっている複合コード** → `str.extract(r"(?P<a>...)(?P<b>...)")` が最も堅い（列名も同時に付く）。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） |
|---|---|---|
| 区切りで分割 | `separate_wider_delim(delim=)` | `str.split(sep, expand=True)` |
| 固定幅で分割 | `separate_wider_position()` | `str.slice` / `str[a:b]` |
| 正規表現で分割 | `separate_wider_regex()` | `str.extract(r"(?P<a>...)")` |
| 分割数の上限 | `too_many=` | `str.split(sep, n=k, expand=True)` |
| 余りを最後にまとめる | — | `str.split(sep, n=1)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **ピース数の不揃い**：行によって区切り数が違うと、pandas は `expand=True` で列数を最大に合わせ、足りない所は `None`。R の新 API は `too_few=`/`too_many=` で挙動を選ぶ。想定列数を `n=` で固定すると安全。
    - **`sep` の正規表現性**：R `separate` の `sep` は正規表現。pandas `str.split` の `pat` も既定で正規表現扱い（`regex=` は将来の指定に注意）。`.` などリテラルは要エスケープ。
    - **型**：分割結果は全部文字列。数値が要るなら後段で `to_numeric`（→ [028](topic-028.md)）。
    - **欠損**：元が NaN の行は分割結果も NaN。

## 関連項目

- [011. 文字列の抽出・長さ](topic-011.md)
- [014. 分割と連結](topic-014.md)
- [039. 列の結合（unite）](topic-039.md)
