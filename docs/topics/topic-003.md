# 003. データフレーム入門 — `tibble` / `data.frame` → pandas / polars

!!! abstract "この項目の R→Python 対応"
    - **R**: `data.frame()`（base）と `tibble()`（tidyverse、表示が親切）
    - **Python（推奨）**: **pandas `DataFrame`**（定番）／**polars `DataFrame`**（式ベース・高速）
    - **要注意**: pandas には **行インデックス（index）** という R にない概念がある

以降の項目で使い回す、被験者レベルの小さなデータを作ります。

---

## R ではこう書く

```r
library(tibble)

dm <- tibble(
  subjid = c("01","02","03","04","05"),
  arm    = c("A","A","B","B","A"),
  age    = c(45, 52, 38, 61, NA),
  sex    = c("M","F","M","F","M"),
  weight = c(70.5, 60.2, 80.1, 55.9, 72.0)
)
dm
nrow(dm); ncol(dm)
names(dm)
```

出力:

```text
# A tibble: 5 × 5
  subjid arm     age sex   weight
  <chr>  <chr> <dbl> <chr>  <dbl>
1 01     A        45 M       70.5
2 02     A        52 F       60.2
3 03     B        38 M       80.1
4 04     B        61 F       55.9
5 05     A        NA M       72
[1] 5
[1] 5
[1] "subjid" "arm"    "age"    "sex"    "weight"
```

!!! note "`data.frame` と `tibble` の使い分け"
    - `tibble`：列の型を勝手に変えない（文字列を factor にしない）、表示が簡潔で型が見える。**新規は基本これ**。
    - `data.frame`：base R の素の形。古い関数の戻り値や、行名（rownames）を使う処理で登場。

---

## Python ではこう書く

=== "pandas"

    ```python
    import numpy as np
    import pandas as pd

    dm = pd.DataFrame({
        "subjid": ["01","02","03","04","05"],
        "arm":    ["A","A","B","B","A"],
        "age":    [45, 52, 38, 61, np.nan],
        "sex":    ["M","F","M","F","M"],
        "weight": [70.5, 60.2, 80.1, 55.9, 72.0],
    })
    print(dm)
    print(dm.shape)          # (行, 列)
    print(list(dm.columns))
    print(dm.dtypes)
    ```

    出力:

    ```text
      subjid arm   age sex  weight
    0     01   A  45.0   M    70.5
    1     02   A  52.0   F    60.2
    2     03   B  38.0   M    80.1
    3     04   B  61.0   F    55.9
    4     05   A   NaN   M    72.0
    (5, 5)
    ['subjid', 'arm', 'age', 'sex', 'weight']
    subjid     object
    arm        object
    age       float64
    sex        object
    weight    float64
    dtype: object
    ```

=== "polars"

    ```python
    import polars as pl

    dmp = pl.DataFrame({
        "subjid": ["01","02","03","04","05"],
        "arm":    ["A","A","B","B","A"],
        "age":    [45, 52, 38, 61, None],
        "sex":    ["M","F","M","F","M"],
        "weight": [70.5, 60.2, 80.1, 55.9, 72.0],
    })
    print(dmp)
    print(dmp.shape)
    print(dmp.columns)
    ```

    polars の表示は tibble と同様、**各列の下に型（`str` / `i64` / `f64`）が出る**ので実務で読みやすいです。欠損は `None` で書きます。

!!! tip "実務ではこれ"
    まずは情報とサンプルの多い **pandas** を土台に。速度や式ベースの書き味が欲しくなったら **polars** を足す、という順番が無難です。本ガイドは両方併記します。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 作る | `tibble(...)` | `pd.DataFrame({...})` | `pl.DataFrame({...})` |
| 行数 / 列数 | `nrow()` / `ncol()` | `df.shape[0]` / `df.shape[1]` | `df.height` / `df.width` |
| 列名 | `names(df)` | `df.columns` | `df.columns` |
| 先頭を見る | `head(df)` | `df.head()` | `df.head()` |
| 構造・型 | `str(df)` / `glimpse(df)` | `df.info()` / `df.dtypes` | `df.schema` |
| 要約統計 | `summary(df)` | `df.describe()` | `df.describe()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **行インデックス（pandas 特有）**：pandas には各行に付く `index` があり、`filter` 相当の後は index が飛び飛びになる。集計後に `.reset_index()` で振り直すのが定石。polars には index がない（R に近い）。
    - **整数列に NA を入れると float 化**：`age` に `NaN` があると pandas では `float64`（`45` が `45.0` 表示）になる。整数を保ちたいなら `"Int64"`（nullable 整数、→ [005](topic-005.md)）。
    - **列の型変換の既定**：`data.frame` は文字列を factor 化する版が歴史的にあった（R 4.0 で既定 OFF）。`tibble` と pandas/polars は勝手に factor 化しない。

## 関連項目

- [004. パッケージ管理と import](topic-004.md)
- [005. 欠損値の扱い](topic-005.md)
- [007. 列選択（select）](topic-007.md)
