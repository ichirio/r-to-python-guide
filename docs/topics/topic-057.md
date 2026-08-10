# 057. n (%) 整形のパターン

!!! abstract "この項目の R→Python 対応"
    - **R**: `count()` → 群内割合を `mutate` → `sprintf` で `n (xx.x%)`
    - **Python（推奨）**: `groupby().size()` → `transform("sum")` で分母 → f-string で整形
    - **要注意**: **分母を何にするか**（群 N か全体 N か）で割合が変わる。群内%は `transform`／`groupby` の分母管理

カテゴリ変数のデモグラ行・AE 表セルの中身。[019](topic-019.md)（1 セルの作り方）を群集計に広げます。

（sex を arm ごとに `n (群内%)` で出す）

---

## R ではこう書く

```r
library(dplyr)

dm3 |>
  count(arm, sex) |>                       # 群×カテゴリの件数
  group_by(arm) |>
  mutate(pct  = 100 * n / sum(n),          # 群内割合（分母＝群 N）
         cell = sprintf("%d (%.1f%%)", n, pct)) |>
  ungroup()
```

出力（要点）:

```text
  arm   sex       n   pct
  A     F         2    40
  A     M         3    60
  B     F         3    60
  B     M         2    40
```

!!! note "R の勘所"
    - `count(arm, sex)` で件数 → `group_by(arm) |> mutate(sum(n))` で**群 N を分母**に。
    - 分母を全体にするなら `group_by` を外す。**分母の取り方が割合の意味を決める**。
    - 整形は `sprintf`（→ [016](topic-016.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    c = dm3.groupby(["arm", "sex"]).size().reset_index(name="n")
    c["pct"]  = 100 * c["n"] / c.groupby("arm")["n"].transform("sum")  # 群内%
    c["cell"] = c.apply(lambda r: f"{r['n']} ({r['pct']:.1f}%)", axis=1)
    print(c)
    ```

    出力:

    ```text
    arm sex  n   pct
      A   F  2  40.0
      A   M  3  60.0
      B   F  3  60.0
      B   M  2  40.0
    ```

=== "polars"

    ```python
    import polars as pl
    (dm3p.group_by(["arm","sex"]).len()
         .with_columns((100 * pl.col("len") / pl.col("len").sum().over("arm")).alias("pct")))
    ```

!!! tip "実務ではこれ"
    - **群内%の分母** → `groupby(group)["n"].transform("sum")`（[030](topic-030.md) の transform）。全体%なら `c["n"].sum()`。
    - **数値で n と pct を持ってから文字列化**。検算・並べ替え・0 埋めがしやすい。
    - **0 件カテゴリ**を `0 (0.0%)` で出すには、集計前に枠を用意（`complete`/Categorical、→ [046](topic-046.md)）。
    - 分母は「割付 N」か「非欠損 N」か——**仕様で固定**する。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 群×カテゴリ件数 | `count(g, cat)` | `groupby([g,cat]).size()` |
| 群内割合 | `n / sum(n)`（group内） | `n / groupby(g)[n].transform("sum")` |
| 全体割合 | `n / sum(n)`（全体） | `n / n.sum()` |
| `n (%)` 整形 | `sprintf("%d (%.1f%%)")` | `f"{n} ({p:.1f}%)"` |
| 0 件も出す | `complete()` | 枠結合 / Categorical |

## つまずきポイント

!!! warning "R と Python の差"
    - **分母の取り違え**：群内%か全体%か列%かで数値が変わる。`transform("sum")` の group を要件どおりに。
    - **丸め方式**：`%.1f` の丸めが SAS と 0.05 でズレ得る（[018](topic-018.md)）。
    - **分母 0**：割付 0 の群で `n/0` は inf/NaN。事前にガード（[019](topic-019.md)）。
    - **0 件の欠落**：集計だけだと出現した組しか出ない。0 件を出すには枠づくりが必須。

## 関連項目

- [019. パーセント表記の作成](topic-019.md)
- [030. グループ内変換](topic-030.md)
- [046. 欠けた組合せを補完](topic-046.md)
- [068. 有害事象（AE）集計表](../roadmap.md)
