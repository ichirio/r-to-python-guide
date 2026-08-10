# 002. パイプとメソッドチェーン — `%>%` / `|>` → method chain

!!! abstract "この項目の R→Python 対応"
    - **R**: magrittr の `%>%`（旧来の主流）と、base R の `|>`（R 4.1+）
    - **Python（推奨）**: **メソッドチェーン**（`df.query(...).assign(...).groupby(...)`）。任意関数は `.pipe()`
    - **要注意**: R のパイプは「第1引数に流す」。Python は「メソッドを `.` でつなぐ」— つなぐ対象が違う

---

## R ではこう書く

R には2種類のパイプがあり、実務では今どちらも見かけます。

```r
library(dplyr)

# ベクトルにも使える
c(4, 1, 3, 2) |> sort() |> rev()

# dplyr の定番の流れ
dm |>
  filter(arm == "A") |>
  summarise(n = n(), mean_wt = mean(weight))
```

出力:

```text
[1] 4 3 2 1
# A tibble: 1 × 2
      n mean_wt
  <int>   <dbl>
1     3    67.6
```

!!! note "`%>%` と `|>` の使い分け"
    | | magrittr `%>%` | base <code>&#124;&gt;</code>（R 4.1+） |
    |---|---|---|
    | 追加パッケージ | 要（tidyverse） | 不要（base） |
    | プレースホルダ | `.`（どこにでも置ける） | `_`（名前付き引数のみ、R 4.2+） |
    | 速度・依存 | わずかに重い | 軽い |
    新規コードは **`|>`** で十分。既存資産や `.` を多用する処理では `%>%` が残ります。

---

## Python ではこう書く

Python は「関数をパイプでつなぐ」のではなく、**オブジェクトのメソッドを `.` でつなぐ**のが基本形です。dplyr の1ステップ＝1メソッドと考えると対応が取れます。

=== "pandas"

    ```python
    import pandas as pd

    # Series のチェーン
    print(pd.Series([4, 1, 3, 2]).sort_values(ascending=False).tolist())

    # dplyr の流れに対応するメソッドチェーン
    print(
        dm.query("arm == 'A'")
          .groupby("arm")
          .agg(n=("subjid", "size"), mean_wt=("weight", "mean"))
          .reset_index()
    )
    ```

    出力:

    ```text
    [4, 3, 2, 1]
      arm  n    mean_wt
    0   A  3  67.566667
    ```

=== "polars"

    ```python
    import polars as pl

    print(
        dmp.filter(pl.col("arm") == "A")
           .group_by("arm")
           .agg(pl.len().alias("n"), pl.col("weight").mean().alias("mean_wt"))
    )
    ```

    polars のメソッド名（`filter` / `group_by` / `agg`）は dplyr とほぼ同名で、最も移行しやすい書き味です。

!!! tip "実務ではこれ"
    - **括弧で囲んで改行**：`(df.method1().method2())` と全体を `()` で包むと、行末バックスラッシュなしで縦に並べられ、dplyr のパイプと同じ見た目になります。
    - **任意の関数を挟みたい**ときは `.pipe(func)`。これが `%>%` の一般形にいちばん近い道具です。
      ```python
      df.pipe(my_func, arg=1).query("x > 0")
      ```

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| つなぐ | <code>x &#124;&gt; f() &#124;&gt; g()</code> | `x.f().g()` | `x.f().g()` |
| 第1引数以外に流す | <code>x &#124;&gt; f(y, z = _)</code> | `x.pipe(lambda d: f(y, d))` | `x.pipe(...)` |
| 任意関数を挟む | 関数を直接 | `.pipe(func)` | `.pipe(func)` |
| 複数行で書く | 改行して <code>&#124;&gt;</code> | 全体を `()` で囲む | 全体を `()` で囲む |

## つまずきポイント

!!! warning "R と Python の差"
    - R のパイプは値を**関数の第1引数**へ流す。Python のチェーンは**メソッド**をつなぐので、そのメソッドが存在しない操作（自作関数など）は `.pipe()` を使う。
    - pandas のメソッドチェーンは**途中での再代入がしにくい**。列を足しながら進むなら `.assign()`（→ [009](topic-009.md)）を挟む。
    - 改行して書くときは、R は行末に `|>` を置けるが、Python は**全体を丸括弧で囲む**必要がある（囲まないと構文エラー）。

## 関連項目

- [009. 列作成・変更（mutate）](topic-009.md)
- [010. グループ集約（group_by+summarise）](topic-010.md)
- [094. パイプで関数をつなぐ設計](../roadmap.md)
