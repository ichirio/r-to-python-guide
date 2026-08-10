# 049. 集計してから結合 — join + summarise

!!! abstract "この項目の R→Python 対応"
    - **R**: `inner_join` などで結合 → `group_by |> summarise` で集計（あるいは集計してから結合）
    - **Python（推奨）**: `merge` → `groupby().agg()`；順序は「先に集計して軽くしてから結合」も定石
    - **要注意**: 結合と集計の**順序**で結果と速度が変わる。粒度（1 行が何を表すか）を意識する

「群ごとの平均検査値」のように、**複数テーブルをまたいだ集計**。結合してから畳むか、畳んでから結合するか。

（例: `dm2`(subjid, arm) と `lb2`(subjid, val) を結合し arm ごとに val 平均）

---

## R ではこう書く

```r
library(dplyr)

dm2 |>
  inner_join(lb2, by = "subjid") |>
  group_by(arm) |>
  summarise(mean_val = mean(val))
```

出力:

```text
  arm   mean_val
  A            5
  B            6
```

!!! note "R の勘所"
    - まず結合して「1 行＝1 被験者の測定」にしてから `group_by(arm) |> summarise`。
    - 逆に**先に被験者内で集計**（`group_by(subjid) |> summarise`）してから群情報を結合、という順もある。データ量が大きいほど「先に畳む」が速い。

---

## Python ではこう書く

=== "pandas"

    ```python
    (dm2.merge(lb2, on="subjid")
        .groupby("arm")["val"].mean()
        .reset_index(name="mean_val"))
    ```

    出力:

    ```text
    arm  mean_val
      A       5.0
      B       6.0
    ```

    「先に集計して軽くしてから結合」する型:

    ```python
    per_subj = lb2.groupby("subjid")["val"].mean().reset_index()
    dm2.merge(per_subj, on="subjid").groupby("arm")["val"].mean()
    ```

=== "polars"

    ```python
    import polars as pl
    (dm2p.join(lb2p, on="subjid")
         .group_by("arm")
         .agg(pl.col("val").mean().alias("mean_val"))
         .sort("arm"))
    ```

!!! tip "実務ではこれ"
    - **粒度をそろえてから集計**：結合すると1行の意味（被験者×測定）が変わる。何を平均しているかを常に確認。
    - **大きいデータは先に畳む**：`lb` を被験者内で集計してから `dm` に貼ると、結合対象が小さくなり速い・重複事故も減る。
    - 結合後の集計は [010](topic-010.md) と同じ `groupby().agg()`。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 結合→群集計 | <code>join &#124;&gt; group_by &#124;&gt; summarise</code> | `merge(...).groupby(...).agg(...)` |
| 先に群内集計→結合 | <code>group_by(id) &#124;&gt; summarise &#124;&gt; left_join</code> | `groupby(id).agg().merge(...)` |
| 集計値を元表に戻す | <code>group_by &#124;&gt; mutate</code> | `groupby().transform()`（→ [030](topic-030.md)） |
| 件数付きで結合 | <code>add_count() &#124;&gt; join</code> | `groupby().size()` を merge |

## つまずきポイント

!!! warning "R と Python の差"
    - **粒度の取り違え**：結合で行が増える（多対多）と、平均・件数が二重計上される。[048](topic-048.md) の検証とセットで。
    - **順序による速度差**：大きな明細を結合してから畳むと重い。先に畳めるなら畳む。結果は同じでも計算量が違う。
    - **欠損の伝播**：左結合で欠損になった値を平均に含めるか（pandas は既定で無視、R は `na.rm` 明示）。[005](topic-005.md) の既定差に注意。
    - **reset_index 忘れ**：pandas は集計後に群キーが index。次の結合で `on=` が効かず詰まる。

## 関連項目

- [010. グループ集約](topic-010.md)
- [030. グループ内変換](topic-030.md)
- [048. 結合キーの検証](topic-048.md)
