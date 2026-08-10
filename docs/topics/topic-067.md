# 067. デモグラフィック表の組み立て

!!! abstract "この項目の R→Python 対応"
    - **R**: `tableone` / `gtsummary::tbl_summary()`（一発）や、dplyr で連続＋カテゴリを積む
    - **Python（推奨）**: pandas で**連続変数行（mean (SD)）とカテゴリ行（n (%)）を別々に作り縦結合**
    - **要注意**: 連続とカテゴリで集計が違う。**別々に作って `concat`**、群を列に `pivot`

Table 1（背景因子表）。[050](topic-050.md)・[055](topic-055.md)・[057](topic-057.md) の型を1枚に組み上げます。

（arm A: 01,02,03 / arm B: 04,05）

---

## R ではこう書く

```r
# 一発で作るなら gtsummary
library(gtsummary)
dm |> tbl_summary(by = arm, include = c(age, sex))

# 手組みなら：連続行と カテゴリ行を作って bind_rows
```

!!! note "R の勘所"
    - `gtsummary::tbl_summary(by=arm)` は連続＝median[IQR]（既定）、カテゴリ＝n(%) を自動で組む。`tableone` も定番。
    - 手組みなら、連続変数を `summarise(mean, sd)`、カテゴリを `count |> mutate(pct)` にして `bind_rows`。

---

## Python ではこう書く

Python に `gtsummary` の定番等価物はないので、**手組み**が基本です（それが構造を制御しやすい）。

=== "pandas"

    ```python
    N = dm.groupby("arm")["subjid"].nunique()      # 群 N（分母）

    # 連続変数行：Age mean (SD)
    cont = (dm.groupby("arm")["age"].agg(["mean","std"])
              .apply(lambda r: f"{r['mean']:.1f} ({r['std']:.2f})", axis=1)
              .rename("value").reset_index())
    cont.insert(0, "var", "Age mean(SD)")

    # カテゴリ変数行：Sex n (%)
    cat = dm.groupby(["arm","sex"]).size().reset_index(name="n")
    cat["pct"]   = 100 * cat["n"] / cat["arm"].map(N)
    cat["value"] = cat.apply(lambda r: f"{r['n']} ({r['pct']:.1f}%)", axis=1)
    cat["var"]   = "Sex: " + cat["sex"]
    cat = cat[["var","arm","value"]]

    # 縦に積んで群を列へ
    demo = (pd.concat([cont, cat], ignore_index=True)
              .pivot(index="var", columns="arm", values="value").fillna("0"))
    demo.columns.name = None
    print(demo)
    ```

    出力:

    ```text
                            A             B
    var
    Age mean(SD)  52.3 (7.51)  49.5 (16.26)
    Sex: F          1 (33.3%)     1 (50.0%)
    Sex: M          2 (66.7%)     1 (50.0%)
    ```

!!! tip "実務ではこれ"
    - **連続（mean(SD)/median[Q1,Q3]）とカテゴリ（n(%)）を別々に作り `concat` → `pivot`**。これがデモグラ表の骨格。
    - 変数の**表示順**は最後に並べ替え（`reindex`）で固定。群順は Categorical。
    - 「Total」列が要るなら、arm を含めない集計を同じ手順で作って横に結合。
    - 清書（見出し・脚注・桁揃え）は great_tables / Styler（→ [071](topic-071.md), [073](topic-073.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 一発生成 | `tbl_summary(by=)` / `tableone` | 手組み（定番なし） |
| 連続行 | `summarise(mean, sd)` | `groupby.agg(["mean","std"])` |
| カテゴリ行 | `count \|> mutate(pct)` | `groupby.size()` + 分母 |
| 縦に積む | `bind_rows` | `pd.concat` |
| 群を列へ | `pivot_wider` | `pivot` |
| Total 列 | `add_overall()` | 群なし集計を結合 |

## つまずきポイント

!!! warning "R と Python の差"
    - **一発関数がない**：`gtsummary`/`tableone` の等価物は薄い。手組みは手間だが、丸め・分母・表記を完全制御できる利点がある。
    - **分母の定義**：`n (%)` の分母は割付 N か非欠損 N か。`map(N)` の N を要件どおりに。
    - **連続とカテゴリの混在**：値がすべて文字列になる（`"52.3 (7.51)"`）。ソートや検算は文字列化前の数値で。
    - **欠損カテゴリ**：現れない水準は 0 埋め（Categorical＋`complete`、→ [046](topic-046.md)）。

## 関連項目

- [055. 群別 N・mean(sd) のデモグラ表](topic-055.md)
- [057. n (%) 整形のパターン](topic-057.md)
- [073. great_tables で整形](topic-073.md)
