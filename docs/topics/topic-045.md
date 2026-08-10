# 045. クロス結合・総当たり — `cross_join` / `expand_grid` → `merge(how="cross")`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `cross_join()`（全行 × 全行）／tidyr `expand_grid()`（値の全組合せ）
    - **Python（推奨）**: pandas **`merge(how="cross")`**；値の組合せは **`itertools.product`** か `pd.MultiIndex.from_product`
    - **要注意**: 行数が掛け算で爆発する。テンプレート枠づくり以外では乱用しない

「全群 × 全来院」のような**空の枠**を作り、そこに集計を貼るとシフト表・欠測ゼロ埋めが楽（→ [046](topic-046.md)）。

---

## R ではこう書く

```r
library(tidyr)

expand_grid(arm = c("A","B"), visit = c(1,2))   # 値の全組合せ

# データフレーム同士の総当たり
# dm2 |> cross_join(visits)
```

出力:

```text
  arm   visit
  A         1
  A         2
  B         1
  B         2
```

!!! note "R の勘所"
    - `expand_grid(a, b)`：**値ベクトルの全組合せ**（`data.frame` の `expand.grid` と似るが型を保つ・順序が違う）。
    - `cross_join(x, y)`：**行同士の総当たり**（キーなし結合）。
    - `expand(df, a, b)`：df に**現れた値**の全組合せ（→ [046](topic-046.md) の `complete` の下地）。

---

## Python ではこう書く

=== "pandas"

    ```python
    import itertools, pandas as pd

    # 値の全組合せ
    pd.DataFrame(itertools.product(["A","B"], [1,2]), columns=["arm","visit"])

    # データフレーム同士の総当たり
    arms = pd.DataFrame({"arm": ["A","B"]})
    visits = pd.DataFrame({"visit": [1,2]})
    arms.merge(visits, how="cross")
    ```

    出力（`merge(how="cross")`）:

    ```text
    arm  visit
      A      1
      A      2
      B      1
      B      2
    ```

=== "polars"

    ```python
    import polars as pl
    pl.DataFrame({"arm":["A","B"]}).join(
        pl.DataFrame({"visit":[1,2]}), how="cross")
    ```

!!! tip "実務ではこれ"
    - **枠（テンプレート）づくり** → `merge(how="cross")` か `itertools.product`。全群×全カテゴリの空表を用意し、集計を左結合で貼ると 0 件も表に出せる。
    - **既存データに現れた組合せだけ**補完したいなら `complete`（→ [046](topic-046.md)）。
    - クロス結合は行数が `n × m`。大きいデータでうっかり使うとメモリを食う。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 値の全組合せ | `expand_grid(a, b)` | `itertools.product` → DataFrame | `pl.DataFrame(...).join(..., how="cross")` |
| 行の総当たり | `cross_join(x, y)` | `x.merge(y, how="cross")` | `x.join(y, how="cross")` |
| 現れた値の組合せ | `expand(df, a, b)` | `MultiIndex.from_product` | — |
| 欠測の枠埋め | `complete()` | `reindex`（→ [046](topic-046.md)） | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **行数爆発**：クロス結合は `n×m` 行。フィルタ前の大きな表で使うと危険。まず必要な列に絞る。
    - **並び順**：`expand_grid`（tidyr）と `expand.grid`（base R）は**列の変化の速さが逆**。pandas `itertools.product` は最後の引数が速く回る（`expand_grid` 寄り）。TFL の並びに合わせて確認。
    - **キー無し結合の明示**：pandas は `how="cross"` を指定（`on=` は付けない）。うっかり `on=` を付けると別物に。
    - **型の保持**：`expand_grid` は型を保つ。pandas も product 経由なら保つが、`MultiIndex.from_product` 経由は object になりがち。

## 関連項目

- [046. 欠けた組合せを補完（complete）](topic-046.md)
- [069. シフトテーブル](../roadmap.md)
