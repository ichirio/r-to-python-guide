# 026. 条件で値を作る — `case_when` / `if_else` → `np.select` / `np.where`

!!! abstract "この項目の R→Python 対応"
    - **R**: 2 値は `if_else()`、多分岐は `case_when()`（`条件 ~ 値`、`TRUE ~ 既定`）
    - **Python（推奨）**: 2 値は **`np.where()`**、多分岐は **`np.select()`**；polars は **`when().then().otherwise()`**
    - **要注意**: 欠損の行がどの分岐に落ちるかは要検算（[009](topic-009.md) と同根）

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> mutate(grp = case_when(
  age < 40 ~ "young",
  age < 60 ~ "mid",
  TRUE     ~ "old"     # どれにも当たらない（NA 含む）→ old
))
```

出力:

```text
  subjid   age grp
1 01        45 mid
2 02        52 mid
3 03        38 young
4 04        61 old
5 05        NA old    ← NA は TRUE 節に落ちる
```

!!! note "R の勘所"
    - `case_when` は**上から順に**最初に真になった枝を採用。`TRUE ~ x` が既定値（`else`）。
    - 枝の値は**型を揃える**（全部文字列など）。混在するとエラー。
    - 2 値だけなら `if_else(cond, a, b)`（型に厳格、NA 伝播）。

---

## Python ではこう書く

=== "pandas"

    ```python
    import numpy as np

    # 多分岐：np.select（条件リストと値リスト）
    cond = [dm["age"] < 40, dm["age"] < 60]
    dm["grp"] = np.select(cond, ["young", "mid"], default="old")

    # 2 値：np.where
    dm["age50"] = np.where(dm["age"] >= 50, ">=50", "<50")
    ```

    出力（`grp`）:

    ```text
    subjid  age   grp
        01 45.0   mid
        02 52.0   mid
        03 38.0 young
        04 61.0   old
        05  NaN   old    ← NaN<40, NaN<60 が False → default に落ちる
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.with_columns(
        pl.when(pl.col("age") < 40).then(pl.lit("young"))
          .when(pl.col("age") < 60).then(pl.lit("mid"))
          .otherwise(pl.lit("old")).alias("grp")
    )
    ```

    `when().then()` を鎖にして、最後の `otherwise()` が `TRUE ~` に対応。pandas 2.2+ なら `Series.case_when()` も使えます。

!!! tip "実務ではこれ"
    - **多分岐** → `np.select([条件...], [値...], default=...)`。`case_when` に最も素直に対応。
    - **2 値** → `np.where(cond, a, b)`。
    - polars を使うなら `when/then/otherwise` が読みやすく型も安全。

---

## 欠損の落とし先に注意

この例では **R も Python も NA/NaN が既定値 "old" に落ち**、結果は一致しました。ただし理由が違います。

- R `case_when`：NA は `age < 40` も `age < 60` も真にならず、`TRUE ~ "old"` に到達。
- `np.select`：`NaN < 40` は `False` なので `default="old"`。

**欠損を独立の群にしたい**なら、明示の枝を先頭に置きます。

=== "R"

    ```r
    case_when(
      is.na(age) ~ NA_character_,   # まず欠損を捕まえる
      age < 40 ~ "young",
      age < 60 ~ "mid",
      TRUE     ~ "old"
    )
    ```

=== "pandas"

    ```python
    cond = [dm["age"].isna(), dm["age"] < 40, dm["age"] < 60]
    np.select(cond, [np.nan, "young", "mid"], default="old")
    ```

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 2 値分岐 | `if_else(c, a, b)` | `np.where(c, a, b)` | `when(c).then(a).otherwise(b)` |
| 多分岐 | `case_when(...)` | `np.select([...],[...],default=)` | `when().then()...otherwise()` |
| 既定値（else） | `TRUE ~ x` | `default=x` | `.otherwise(x)` |
| 欠損を別扱い | `is.na(x) ~ ...` を先頭 | `df["x"].isna()` を先頭 | `when(col.is_null())` を先頭 |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損の落とし先**：`np.select` は `NaN < k` を False とみなすので、明示しないと欠損が `default` に混ざる。R の `case_when` も `TRUE ~` に落ちるので同様。**群分けは必ず検算**。
    - **型の混在**：`np.select` の値に数値と文字列を混ぜると object 型で扱いづらくなる。値の型は揃える。
    - **評価順**：どちらも**上から最初に真**。範囲条件は狭い順（`<40` → `<60`）に並べる。
    - **`np.where` の結果は ndarray**：index を持たないので、`df["x"] = np.where(...)` のように代入して DataFrame に載せる。

## 関連項目

- [009. 列作成・変更（mutate）](topic-009.md)
- [027. 値の対応付け・recode](topic-027.md)
- [034. ビン分割（cut）](../roadmap.md)
