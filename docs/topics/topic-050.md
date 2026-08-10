# 050. ロング／ワイドを行き来する実務パターン（TFL整形）

!!! abstract "この項目の R→Python 対応"
    - **R**: 「縦持ちで集計 → `pivot_wider` で群を列に」が TFL の定石
    - **Python（推奨）**: 「`groupby` で集計 → `pivot` で群を列に → 0 埋め・整形」
    - **要注意**: 集計は**ロング**、表示は**ワイド**。この往復を型として持つと TFL が安定する

第4部の総まとめ。縦持ち集計 → 横持ち表という、デモグラ表・件数表の共通の骨格を1つの型にします。

（例: 被験者データを「性別（行）× 群（列）の人数表」にする）

---

## R ではこう書く

```r
library(dplyr); library(tidyr)
raw <- tibble(subjid = c("01","02","03","04","05"),
              arm = c("A","A","B","B","A"),
              sex = c("M","F","M","F","M"))

raw |>
  count(sex, arm) |>                                   # ロングで集計
  pivot_wider(names_from = arm, values_from = n,       # 群を列へ
              values_fill = 0)                          # 0 件を 0 に
```

出力:

```text
  sex       A     B
  F         1     1
  M         2     1
```

!!! note "R の勘所"
    - **集計はロング**（`count`/`summarise`）→ **表示はワイド**（`pivot_wider`）が基本の流れ。
    - `values_fill = 0` で空セルを 0 に（→ [046](topic-046.md) の complete と同趣旨）。
    - 群の順序・欠測群は factor 水準で固定すると崩れない。

---

## Python ではこう書く

=== "pandas"

    ```python
    raw = pd.DataFrame({"subjid":["01","02","03","04","05"],
                        "arm":["A","A","B","B","A"],
                        "sex":["M","F","M","F","M"]})

    tab = (raw.groupby(["sex","arm"]).size().reset_index(name="n")  # ロング集計
              .pivot(index="sex", columns="arm", values="n")        # 群を列へ
              .fillna(0).astype(int).reset_index())                 # 0 埋め・整形
    tab.columns.name = None
    print(tab)
    ```

    出力:

    ```text
    sex  A  B
      F  1  1
      M  2  1
    ```

=== "polars"

    ```python
    import polars as pl
    rawp = pl.DataFrame({"subjid":["01","02","03","04","05"],
                         "arm":["A","A","B","B","A"],
                         "sex":["M","F","M","F","M"]})
    (rawp.group_by(["sex","arm"]).len()
         .pivot(on="arm", index="sex", values="len")
         .fill_null(0)
         .sort("sex"))
    ```

!!! tip "実務ではこれ（TFL の型）"
    1. **ロングで集計**：`groupby([行キー, 列キー]).agg(...)`（件数なら `size`、統計量なら `mean` 等）。
    2. **ワイドに展開**：`pivot(index=行キー, columns=列キー, values=...)`。
    3. **整形**：`fillna(0)` で 0 件、`astype(int)`、`reset_index()`、`columns.name=None`。列順・行順を規定水準で固定。
    4. **"n (%)" 化**：件数と割合を数値で持ってから f-string で文字列化（→ [019](topic-019.md), [057](../roadmap.md)）。

    この4段を**関数化**しておくと、デモグラ表・AE 表・シフト表が同じ骨格で作れます。

---

## 対応早見表

| 段階 | R | Python（pandas） |
|---|---|---|
| ロング集計 | `count(a, b)` / `summarise` | `groupby([a,b]).agg()` |
| ワイド展開 | `pivot_wider(names_from=b)` | `pivot(index=a, columns=b)` |
| 0 埋め | `values_fill=0` | `fillna(0)` |
| 列名整形 | 自動 | `columns.name=None` / 平坦化 |
| 行・列順の固定 | factor 水準 | Categorical / 明示の並べ替え |

## つまずきポイント

!!! warning "R と Python の差"
    - **列の後始末**：pandas `pivot` は `columns.name` と MultiIndex 列が残りがち。`reset_index()`＋`columns.name=None`（多層なら平坦化）で仕上げる。
    - **0 件が消える**：集計だけだと出現した組しか出ない。0 件を出すには `pivot` の `fillna(0)` か、事前に `complete`/枠結合（→ [046](topic-046.md)）。
    - **順序**：`pivot` はキーをソートする。TFL の群順（例: Placebo → 低用量 → 高用量）は Categorical 水準で固定。
    - **型の float 化**：欠損 0 埋め前は float。`fillna(0).astype(int)` の順で整数に戻す。

## 関連項目

- [036. 縦持ち化（pivot_longer）](topic-036.md)
- [037. 横持ち化（pivot_wider）](topic-037.md)
- [055. 群別 N・mean(sd) のデモグラ表](../roadmap.md)
- [067. デモグラフィック表の組み立て](../roadmap.md)
