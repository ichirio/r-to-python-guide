# 091. 因子とラベル — factor levels / labels → `Categorical(ordered=True)`

!!! abstract "この項目の R→Python 対応"
    - **R**: `factor(x, levels=, labels=, ordered=)`／forcats（`fct_relevel` 等）
    - **Python（推奨）**: **`pd.Categorical(x, categories=, ordered=True)`** / `astype("category")`
    - **要注意**: 水準順が TFL の並び・0 件表示・順序比較を左右する。**水準を明示的に固定**する

群の並び・グレードの順序・出現しない水準の 0 表示——すべて「水準（categories）」の管理です。

（例: Low < Mid < High の順序付き因子）

---

## R ではこう書く

```r
f <- factor(c("Low","High","Mid"),
            levels = c("Low","Mid","High"), ordered = TRUE)
as.integer(f)     # 水準番号 → 1 3 2
f < "High"        # 順序比較 → TRUE FALSE TRUE
levels(f)         # "Low" "Mid" "High"
```

出力:

```text
[1] 1 3 2
[1]  TRUE FALSE  TRUE
```

!!! note "R の勘所"
    - `levels=` で**順序を固定**（アルファベット順に流されない）。
    - `ordered = TRUE` で大小比較が可能。
    - 出現しない水準も `levels` にあれば `table()` で 0 として残る（[053](topic-053.md)）。
    - `labels=` で表示ラベルを付け替え。並べ替えは forcats。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd
    c = pd.Categorical(["Low","High","Mid"],
                       categories=["Low","Mid","High"], ordered=True)

    (c.codes + 1).tolist()      # 水準番号（codes は 0 始まり）→ [1, 3, 2]
    (c < "High").tolist()       # 順序比較 → [True, False, True]
    c.categories.tolist()       # ['Low', 'Mid', 'High']

    # DataFrame の列に
    df["grade"] = pd.Categorical(df["grade"],
                    categories=["Low","Mid","High"], ordered=True)
    ```

    出力:

    ```text
    [1, 3, 2]
    [True, False, True]
    ```

!!! tip "実務ではこれ"
    - **群・グレードは Categorical で水準を固定**。`groupby`/`sort`/`crosstab` の並びが水準順になる。
    - **順序比較・大小**が要るなら `ordered=True`。
    - **出現しない水準を 0 で出す**（シフト表・度数表）には、水準を全宣言してから集計（[046](topic-046.md), [069](topic-069.md)）。
    - `codes` は **0 始まり**。R の `as.integer`（1 始まり）と揃えるなら `+1`。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 因子を作る | `factor(x, levels=)` | `pd.Categorical(x, categories=)` |
| 順序付き | `ordered = TRUE` | `ordered=True` |
| 水準一覧 | `levels(f)` | `c.categories` |
| 水準番号 | `as.integer(f)`（1始まり） | `c.codes`（0始まり） |
| 並べ替え | `fct_relevel()` | `cat.reorder_categories()` |
| ラベル変更 | `labels=` / `fct_recode()` | `cat.rename_categories()` |
| 未使用水準を落とす | `droplevels()` | `cat.remove_unused_categories()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **codes は 0 始まり**：pandas の `cat.codes` は 0 始まり、R の `as.integer` は 1 始まり。突き合わせるなら `+1`。
    - **順序を固定しないと並びが崩れる**：`categories` を指定しないと出現順やアルファベット順になり、TFL の群順（Placebo→実薬）が狂う。
    - **文字列操作で水準が消える**：Categorical 列に `str.replace` などを掛けると object 型に戻り順序が失われる。`rename_categories` を使う。
    - **欠損水準**：`value_counts` で 0 を出すには水準宣言が要る。`observed=` 引数（groupby）も影響。

## 関連項目

- [028. 型変換](topic-028.md)
- [069. シフトテーブル](topic-069.md)
- [053. カテゴリの頻度と割合](topic-053.md)
