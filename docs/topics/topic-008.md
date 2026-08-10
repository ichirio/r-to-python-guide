# 008. 行フィルタ — `filter` → `query` / boolean mask

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `filter()`（複数条件はカンマ＝AND）
    - **Python（推奨）**: pandas は **`df.query("...")`**（読みやすい）または **boolean mask**；polars は **`filter()`**
    - **要注意**: mask で複数条件は `&` / `|` と**各条件を丸括弧で囲む**。`and` / `or` は使えない

（データは [003](topic-003.md) の `dm` を使用）

---

## R ではこう書く

```r
library(dplyr)

dm |> filter(arm == "A")            # 単一条件
dm |> filter(age >= 50)             # 比較
dm |> filter(arm == "A", age >= 50) # カンマは AND
dm |> filter(is.na(age))            # 欠損の行
```

出力:

```text
# arm == "A" → 3 行（01, 02, 05）
# age >= 50  → 2 行（02, 04）
# arm=="A" & age>=50 → 1 行（02）
# is.na(age) → 1 行（05）
```

!!! note "R の filter の勘所"
    - カンマ区切りの条件は **AND**（`&` と同じ）。OR は `|` を明示。
    - `filter(age >= 50)` は **NA の行を落とす**（`NA >= 50` は `NA`＝偽扱いで除外）。ここは Python でも同様。
    - 値の集合は `%in%`：`filter(arm %in% c("A","B"))`。

---

## Python ではこう書く

=== "pandas"

    ```python
    # query：文字列で書けて読みやすい（列名を裸で使える）
    dm.query("arm == 'A'")
    dm.query("arm == 'A' and age >= 50")   # query 内は and/or でOK
    dm.query("age >= 50")

    # boolean mask：括弧と & | が必須
    dm[dm["arm"] == "A"]
    dm[(dm["arm"] == "A") & (dm["age"] >= 50)]
    dm[dm["age"].isna()]                    # 欠損の行
    dm[dm["arm"].isin(["A", "B"])]          # %in% 相当
    ```

    出力（`arm=="A" & age>=50`）:

    ```text
      subjid arm   age sex  weight
    1     02   A  52.0   F    60.2
    ```

=== "polars"

    ```python
    import polars as pl

    dmp.filter(pl.col("arm") == "A")
    dmp.filter((pl.col("arm") == "A") & (pl.col("age") >= 50))
    dmp.filter(pl.col("age").is_null())
    dmp.filter(pl.col("arm").is_in(["A", "B"]))
    ```

    メソッド名が `filter` で dplyr と同じ。複数条件は pandas mask と同様 `&` / `|` ＋ 括弧。

!!! tip "実務ではこれ"
    - **読みやすさ優先なら `df.query("...")`**。SQL の WHERE に近く、`and`/`or`/`in`/`@変数` が自然に書ける。
      ```python
      lo = 50
      dm.query("age >= @lo and sex == 'M'")   # @ でPython変数を参照
      ```
    - **動的に条件を組む**ときや**速度**重視なら boolean mask / polars。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas query） | Python（polars） |
|---|---|---|---|
| 単一条件 | `filter(x == 1)` | `query("x == 1")` | `filter(pl.col("x")==1)` |
| AND | `filter(a, b)` | `query("a and b")` | `filter(a & b)` |
| OR | <code>filter(a &#124; b)</code> | `query("a or b")` | <code>filter(a &#124; b)</code> |
| 集合 | `filter(x %in% v)` | `query("x in @v")` | `filter(pl.col("x").is_in(v))` |
| 欠損の行 | `filter(is.na(x))` | `df[df["x"].isna()]` | `filter(pl.col("x").is_null())` |
| 否定 | `filter(!cond)` | `query("not cond")` | `filter(~cond)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **mask の `and`/`or` は使えない**：`df[(a) and (b)]` は `ValueError`。要素ごとの `&` / `|` を使い、**各条件を丸括弧で囲む**（`&` の優先順位が高いため）。`query` 内なら `and`/`or` でよい。
    - **`%in%` → `isin` / `in`**：R の `%in%` は pandas mask では `Series.isin(...)`、`query` では `x in @list`。
    - **欠損の除外は自動**：`age >= 50` は R も Python も NA/NaN 行を落とす。「欠損も残したい」場合は `| df["age"].isna()` を明示。
    - **index が飛ぶ**：pandas は抽出後に行 index が元番号のまま飛び飛びに残る。連番にしたいなら `.reset_index(drop=True)`。

## 関連項目

- [007. 列選択（select）](topic-007.md)
- [009. 列作成・変更（mutate）](topic-009.md)
- [026. 条件で値を作る（case_when）](../roadmap.md)
