# 041. 内部結合 — `inner_join` → `merge`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `inner_join(x, y, by=)`（両方にキーがある行だけ残す）
    - **Python（推奨）**: pandas **`merge(y, on=, how="inner")`**；polars **`join(y, on=, how="inner")`**
    - **要注意**: pandas の `merge` の既定は `how="inner"`。キー名が違うときは `left_on`/`right_on`

（例: `dm2`=subjid 01,02,03 / `lb2`=subjid 02,03,04）

---

## R ではこう書く

```r
library(dplyr)
dm2 <- tibble(subjid = c("01","02","03"), arm = c("A","A","B"))
lb2 <- tibble(subjid = c("02","03","04"), val = c(5,6,7))

dm2 |> inner_join(lb2, by = "subjid")   # 両方にある 02, 03 だけ
```

出力:

```text
  subjid arm     val
  02     A         5
  03     B         6
```

!!! note "R の勘所"
    - `by = "subjid"`：共通キー。複数キーは `by = c("subjid","visit")`。
    - キー名が左右で違うなら `by = c("id" = "subject")`（左=右）。
    - dplyr 1.1+ は `join_by(subjid)` 構文も。不等号結合（`join_by(a >= b)`）もここで。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm2.merge(lb2, on="subjid", how="inner")   # how 既定も inner

    # キー名が違う場合
    # dm2.merge(lb2, left_on="id", right_on="subject")
    ```

    出力:

    ```text
    subjid arm  val
        02   A    5
        03   B    6
    ```

=== "polars"

    ```python
    import polars as pl
    dm2p.join(lb2p, on="subjid", how="inner")
    ```

!!! tip "実務ではこれ"
    - **`merge(y, on=key, how="inner")`** を基本形に。`how` を毎回明示すると意図が読める（既定 inner に頼らない）。
    - **キー名が左右で違う** → `left_on=` / `right_on=`。余分な重複キー列は後で `drop`。
    - 結合前に**キーの一意性を確認**（→ [048](topic-048.md)）。想定外の重複は行数爆発の元。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 内部結合 | `inner_join(x, y, by=)` | `x.merge(y, on=, how="inner")` | `x.join(y, on=, how="inner")` |
| 複数キー | `by = c("a","b")` | `on=["a","b"]` | `on=["a","b"]` |
| キー名が違う | `by = c("id"="subject")` | `left_on=, right_on=` | `left_on=, right_on=` |
| インデックス結合 | — | `x.join(y)`（index 同士） | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **`merge` の既定 how**：pandas は `how="inner"`。R の `*_join` は関数名で結合種別を決めるので、pandas では `how=` を明示する癖をつける。
    - **重複キーで行が増える**：一方に同じキーが複数あると内部結合でも行が掛け算になる。事前に `drop_duplicates` か `validate=`（→ [048](topic-048.md)）。
    - **キー列の型不一致**：`"01"`（文字）と `1`（数値）は結合できない。事前に型を揃える（→ [028](topic-028.md)）。
    - **列名衝突**：キー以外に同名列があると `_x`/`_y` が付く。`suffixes=` で制御。

## 関連項目

- [042. 左・外部結合](topic-042.md)
- [048. 結合キーの検証](topic-048.md)
- [047. ルックアップで値付与](topic-047.md)
