# 009. 列作成・変更 — `mutate` → `assign` / `with_columns`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `mutate()`（既存列を参照して新列を作る／上書きする）
    - **Python（推奨）**: pandas は **`df.assign(...)`**（チェーンで安全）／`df["新列"] = ...`；polars は **`with_columns()`**
    - **要注意**: 条件列を作る `if_else` と `np.where` は **NA/NaN の扱いが違う**（下の警告は必読）

（データは [003](topic-003.md) の `dm` を使用）

---

## R ではこう書く

```r
library(dplyr)

dm |> mutate(
  bmi_ish = weight / 2,
  age_grp = if_else(age >= 50, ">=50", "<50")
)
```

出力:

```text
# A tibble: 5 × 7
  subjid arm     age sex   weight bmi_ish age_grp
  <chr>  <chr> <dbl> <chr>  <dbl>   <dbl> <chr>
1 01     A        45 M       70.5    35.2 <50
2 02     A        52 F       60.2    30.1 >=50
3 03     B        38 M       80.1    40.0 <50
4 04     B        61 F       55.9    28.0 >=50
5 05     A        NA M       72      36   <NA>     ← age が NA なので age_grp も NA
```

!!! note "R の mutate の勘所"
    - **同じ `mutate` 内で直前に作った列を参照できる**：`mutate(a = x*2, b = a + 1)`。
    - `if_else()`（stringr/dplyr）は型に厳格で **NA を伝播**。ゆるい base の `ifelse()` もあるが型が崩れやすい。
    - 複数分岐は `case_when()`（→ [026](../roadmap.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    import numpy as np

    out = dm.assign(
        bmi_ish=dm["weight"] / 2,
        age_grp=np.where(dm["age"] >= 50, ">=50", "<50"),
    )
    print(out)
    ```

    出力:

    ```text
      subjid arm   age sex  weight  bmi_ish age_grp
    0     01   A  45.0   M    70.5    35.25     <50
    1     02   A  52.0   F    60.2    30.10    >=50
    2     03   B  38.0   M    80.1    40.05     <50
    3     04   B  61.0   F    55.9    27.95    >=50
    4     05   A   NaN   M    72.0    36.00     <50   ← R と違い "<50" になる！
    ```

    直接代入でもよい（チェーンしないとき）:

    ```python
    dm["bmi_ish"] = dm["weight"] / 2
    ```

=== "polars"

    ```python
    import polars as pl

    dmp.with_columns(
        (pl.col("weight") / 2).alias("bmi_ish"),
        pl.when(pl.col("age") >= 50).then(pl.lit(">=50"))
          .otherwise(pl.lit("<50")).alias("age_grp"),
    )
    ```

    出力（抜粋）:

    ```text
    │ 05     ┆ 36.0    ┆ <50     │   ← polars も既定では "<50"（下の警告参照）
    ```

    `with_columns` は「複数の新列を一度に」書け、`pl.when().then().otherwise()` が `if_else` / `case_when` に対応します。

!!! tip "実務ではこれ"
    - **チェーンの途中で列を足す**なら pandas `assign`（元の df を壊さず新 df を返す）。ループで `df["x"]=...` を繰り返すより意図が明確。
    - **複数の新列＋条件分岐**が絡むなら polars の `with_columns` + `when/then/otherwise` が最も読みやすい。

---

## :material-alert: NA/NaN の分岐がずれる（最重要）

`age` が欠損の被験者 05 で、**R は `NA`、Python(np.where) は `"<50"`** と結果が食い違いました。

- R の `if_else(age >= 50, ...)`：`NA >= 50` は `NA` → **結果も `NA`**（欠損を伝播）。
- NumPy の `np.where(age >= 50, ...)`：`NaN >= 50` は `False` → **`else` 側の `"<50"`** に落ちる。

臨床の年齢群・カテゴリ化でこれをそのままにすると、**欠損者が黙って「若年群」に混入**します。欠損を欠損のまま残すには、条件を明示します。

=== "pandas（欠損を保持）"

    ```python
    import numpy as np
    # np.select で欠損を明示的に NaN にする
    cond = [dm["age"] >= 50, dm["age"] < 50]
    dm.assign(age_grp=np.select(cond, [">=50", "<50"], default=np.nan))
    # → 05 は NaN（R の if_else と一致）
    ```

=== "polars（欠損を保持）"

    ```python
    import polars as pl
    dmp.with_columns(
        pl.when(pl.col("age").is_null()).then(None)
          .when(pl.col("age") >= 50).then(pl.lit(">=50"))
          .otherwise(pl.lit("<50")).alias("age_grp")
    )
    ```

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 新列を作る | `mutate(y = x*2)` | `df.assign(y=df["x"]*2)` | `with_columns((pl.col("x")*2).alias("y"))` |
| 列を上書き | `mutate(x = x*2)` | `df.assign(x=df["x"]*2)` | `with_columns((pl.col("x")*2).alias("x"))` |
| 2値の条件列 | `if_else(c, a, b)` | `np.where(c, a, b)` | `pl.when(c).then(a).otherwise(b)` |
| 多分岐 | `case_when(...)` | `np.select([...],[...])` | `pl.when().then()...` を連ねる |
| 直前列を参照 | `mutate(a=..., b=a+1)` | `.assign(a=...).assign(b=lambda d: d["a"]+1)` | `with_columns` は同時参照不可、分割 |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損の分岐**（上記）：`if_else` は NA 伝播、`np.where` は NaN を偽扱い。**欠損者の群分けは必ず検算**。
    - **同時参照**：R の `mutate` は同じ呼び出し内で作りたての列を参照できる。pandas `assign` は `lambda d: d["新列"]` で参照、polars `with_columns` は**同じ呼び出し内で作った列を参照できない**（2回に分ける）。
    - **文字列の欠損表示**：`np.where` の結果を持つ object 列では、R の `<NA>` は pandas で `NaN` として現れる。整形時の見え方に注意。

## 関連項目

- [008. 行フィルタ（filter）](topic-008.md)
- [010. グループ集約（group_by+summarise）](topic-010.md)
- [026. 条件で値を作る（case_when）](../roadmap.md)
