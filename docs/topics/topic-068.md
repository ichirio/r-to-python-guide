# 068. 有害事象（AE）集計表

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr で「**被験者単位**の重複除去 → SOC/PT ごとに `n_distinct(subjid)` → n(%)」
    - **Python（推奨）**: pandas で **`drop_duplicates` → `groupby.nunique` → 分母で n(%)**
    - **要注意**: AE は「発現**件数**」でなく「発現**被験者数**」が基本。**被験者単位で重複を潰してから数える**

有害事象表の肝は「同じ人が同じ事象を複数回起こしても 1 と数える」こと。ここを外すと %が 100 を超えます。

（arm A: 01,02,03 (N=3) / arm B: 04,05 (N=2)）

---

## R ではこう書く

```r
library(dplyr)

N <- dm |> distinct(arm, subjid) |> count(arm)   # 群 N

ae |>
  distinct(arm, pt, subjid) |>          # 被験者単位に重複除去
  count(arm, pt) |>                      # 発現被験者数
  left_join(N, by = "arm", suffix = c("", "_N")) |>
  mutate(cell = sprintf("%d (%.1f%%)", n, 100 * n / n_N))
```

!!! note "R の勘所"
    - **`distinct(subjid, pt)`** で「1 被験者×1 事象＝1」に。これを忘れると件数集計になり%が壊れる。
    - 分母は群の**割付被験者数**（安全性解析対象集団）。
    - SOC→PT の階層は、SOC 行と PT 行を別に作って積む（rtables なら自動）。

---

## Python ではこう書く

=== "pandas"

    ```python
    N = dm.groupby("arm")["subjid"].nunique()          # 群 N（分母）

    u   = ae.drop_duplicates(["arm","pt","subjid"])    # 被験者単位に
    tab = u.groupby(["pt","arm"])["subjid"].nunique().reset_index(name="n")
    tab["pct"]  = 100 * tab["n"] / tab["arm"].map(N)
    tab["cell"] = tab.apply(lambda r: f"{r['n']} ({r['pct']:.1f}%)", axis=1)

    out = tab.pivot(index="pt", columns="arm", values="cell").fillna("0")
    out.columns.name = None
    print(out)
    ```

    出力:

    ```text
                      A          B
    pt
    Headache  2 (66.7%)          0
    Nausea    1 (33.3%)  1 (50.0%)
    ```

=== "polars"

    ```python
    import polars as pl
    (aep.unique(["arm","pt","subjid"])
        .group_by(["pt","arm"]).agg(pl.col("subjid").n_unique().alias("n")))
    # 分母結合と n(%) 整形は Python 側で
    ```

!!! tip "実務ではこれ"
    - **被験者単位に `drop_duplicates([arm, pt, subjid])` → `nunique(subjid)`**。「件数」でなく「発現被験者数」。
    - **SOC / PT の 2 階層**：SOC 集計（PT を無視して被験者単位）と PT 集計を別に作り、行を組む。「Any AE」行は `pt` を無視した被験者単位集計。
    - **0 件**を `0 (0.0%)` で出すには全 PT × 全群の枠を用意（→ [046](topic-046.md)）。
    - 重症度・関連性別の内訳は、`grade`/`related` を集計キーに足す。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 被験者単位に | `distinct(arm, pt, subjid)` | `drop_duplicates([...])` |
| 発現被験者数 | `count(arm, pt)` / `n_distinct` | `groupby([...]).nunique()` |
| 群 N（分母） | <code>distinct(arm, subjid) &#124;&gt; count</code> | `groupby("arm")["subjid"].nunique()` |
| n (%) | `sprintf` | `f"{n} ({pct:.1f}%)"` |
| Any AE 行 | `distinct(arm, subjid)` | `drop_duplicates([arm, subjid])` |
| SOC→PT 階層 | rtables / 行を積む | 別集計を concat |

## つまずきポイント

!!! warning "R と Python の差"
    - **件数 vs 被験者数**：`groupby().size()`（件数）と `nunique(subjid)`（被験者数）は別物。AE 表は原則**被験者数**。取り違えると%が 100 超に。
    - **分母**：安全性解析対象集団の N。曝露群ごとの割付数を使う。イベントのある人だけを分母にしない。
    - **階層の合計**：SOC 行は「その SOC のいずれかの PT を発現した被験者数」で、PT の単純合計ではない（1 人が複数 PT を持つと二重計上になる）。SOC も被験者単位で数え直す。
    - **0 件表示**：出現しない PT を出すには枠づくりが必要。

## 関連項目

- [057. n (%) 整形のパターン](topic-057.md)
- [040. 行の展開（separate_rows）](topic-040.md)
- [046. 欠けた組合せを補完](topic-046.md)
