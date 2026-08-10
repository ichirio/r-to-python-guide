# 027. 値の対応付け・recode — `recode` / `case_match` → `map` / `replace`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `recode()`（`旧 = 新`）／`case_match()`（新しい推奨）／factor は `fct_recode()`
    - **Python（推奨）**: 辞書対応は **`Series.map({旧: 新})`**（未対応は NaN）／**`replace({旧: 新})`**（未対応は元のまま）
    - **要注意**: **`map` は対応外を欠損に、`replace` は対応外をそのまま残す**。ここを取り違えると事故る

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> mutate(sex2 = recode(sex, "M" = "Male", "F" = "Female"))

# 新しい書き方（推奨）
dm |> mutate(sex2 = case_match(sex, "M" ~ "Male", "F" ~ "Female"))
```

出力（sex → sex2）:

```text
M → Male
F → Female
```

!!! note "R の勘所"
    - `recode(x, 旧 = 新)`：**左が旧**。対応にない値は既定でそのまま、`.default=` で一括変更。
    - `case_match()`：`recode` の後継で `旧 ~ 新`。可読性が高く公式も推奨。
    - **factor の水準名**を変えるなら `forcats::fct_recode()`（順序を保ったまま）。

---

## Python ではこう書く

=== "pandas"

    ```python
    mapping = {"M": "Male", "F": "Female"}

    dm["sex"].map(mapping)          # 対応外は NaN になる
    dm["sex"].replace(mapping)      # 対応外は元の値のまま

    # 対応外を残しつつ辞書対応（map + fillna）
    dm["sex"].map(mapping).fillna(dm["sex"])
    ```

    未対応値の違い:

    ```python
    import pandas as pd
    s = pd.Series(["M", "F", "X"])
    print(s.map(mapping).tolist())      # ['Male', 'Female', nan]   ← X が NaN
    print(s.replace(mapping).tolist())  # ['Male', 'Female', 'X']   ← X は残る
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.with_columns(
        pl.col("sex").replace({"M": "Male", "F": "Female"}).alias("sex2")
    )
    # 対応外を既定値にするなら replace_strict(..., default=...)
    ```

!!! tip "実務ではこれ"
    - **決まった符号の言い換え**（`M/F` → `Male/Female`、`Y/N` → `Yes/No`）は、**未対応をどうしたいか**で選ぶ:
        - 未対応は欠損にして検出したい → `map`
        - 未対応はそのまま通したい → `replace`
    - `map` で「対応外は元の値」にしたいなら `.map(d).fillna(original)`。
    - コード→ラベルの対応表が大きいなら dict を外部（辞書/CSV）に持ち、`map` で当てる。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 辞書で置換（未対応→欠損） | — | `s.map({...})` | `replace_strict(..., default=None)` |
| 辞書で置換（未対応→残す） | `recode(x, ...)` | `s.replace({...})` | `s.replace({...})` |
| 既定値を付ける | `recode(..., .default=)` | `s.map(d).fillna(def)` | `replace_strict(..., default=def)` |
| factor 水準の改名 | `fct_recode()` | `cat.rename_categories()` | — |
| 数値→ラベル（範囲） | `case_when` / `cut` | `pd.cut`（→ [034](../roadmap.md)） | `pl.when()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **未対応値の運命**：`map` は NaN、`replace` は元のまま。R の `recode` は「元のまま」寄り＝ `replace` に近い。取り違えると欠損が湧く/残る。
    - **`replace` の癖**：pandas の `replace` は正規表現・部分一致の設定もあり、意図せぬ置換が起きうる。辞書完全一致なら安全だが、pandas のバージョンで将来の挙動変更に注意（`FutureWarning` が出る版あり）。
    - **factor/Categorical**：単なる文字列置換だと Categorical の順序・水準が崩れる。順序を保つなら `cat.rename_categories({...})`（→ [091](../roadmap.md)）。
    - **型**：数値キーの dict と文字列キーの列は一致しない。キーの型を列に合わせる。

## 関連項目

- [026. 条件で値を作る（case_when）](topic-026.md)
- [034. ビン分割（cut）](../roadmap.md)
- [091. 因子とラベル](../roadmap.md)
