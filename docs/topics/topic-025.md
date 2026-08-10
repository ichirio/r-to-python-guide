# 025. リネーム — `rename` → `rename`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `rename(new = old)`（新 = 旧の向き）／`rename_with()`（関数で一括）
    - **Python（推奨）**: pandas **`rename(columns={"old": "new"})`**（旧: 新の向き）；polars **`rename({"old": "new"})`**
    - **要注意**: **向きが逆**。R は `新 = 旧`、pandas/polars は `{旧: 新}`

（データは [003](topic-003.md) の `dm`）

---

## R ではこう書く

```r
library(dplyr)

dm |> rename(subject = subjid, treatment = arm)   # 新 = 旧
dm |> rename_with(toupper)                         # 全列名を大文字に
dm |> rename_with(~ gsub("_", ".", .x))            # 関数で一括変換
```

出力（列名）:

```text
subject  treatment  age  sex  weight
```

!!! note "R の勘所"
    - `rename(new = old)`：**左が新**、右が旧。tidy-select と同じ向き。
    - `rename_with(fn)`：全列（または一部）に関数を適用。命名規則の一括変更に。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm.rename(columns={"subjid": "subject", "arm": "treatment"})   # {旧: 新}

    # 関数で一括
    dm.rename(columns=str.upper)                 # 全列大文字
    dm.rename(columns=lambda c: c.replace("_", "."))

    # まるごと差し替え（順序に依存）
    dm.columns = ["subject", "treatment", "age", "sex", "weight"]
    ```

    出力（列名）:

    ```text
    ['subject', 'treatment', 'age', 'sex', 'weight']
    ```

=== "polars"

    ```python
    import polars as pl
    dmp.rename({"subjid": "subject", "arm": "treatment"})   # {旧: 新}
    ```

!!! tip "実務ではこれ"
    - 個別は pandas **`rename(columns={旧:新})`**（存在しないキーは黙って無視される＝安全）。
    - **命名規則の一括変換**（小文字化、`.`→`_` など）は `rename(columns=関数)`。ADaM 変数名の正規化などに。
    - 列を**全部置き換える**ときだけ `df.columns = [...]`。順序ミスで取り違えやすいので、対応が明示できる dict 版を優先。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 個別リネーム | `rename(new = old)` | `rename(columns={"old":"new"})` | `rename({"old":"new"})` |
| 関数で一括 | `rename_with(fn)` | `rename(columns=fn)` | `rename({c: fn(c) for c in df.columns})` |
| 全列大文字 | `rename_with(toupper)` | `rename(columns=str.upper)` | 同上 |
| 全部差し替え | `names(df) <- v` | `df.columns = v` | `df.columns = v` |

## つまずきポイント

!!! warning "R と Python の差"
    - **向きが逆**：R は `新 = 旧`、pandas/polars は `{旧: 新}`。移植時の最頻ミス。
    - **存在しない列**：pandas `rename` は該当なしのキーを**黙って無視**（`errors="raise"` で厳格化）。polars は存在しない旧名でエラー。
    - **`df.columns = [...]` は位置対応**：dict と違い順序ズレが起きても気付きにくい。列数が合わないとエラー。
    - **行インデックス名**：pandas は `rename(index=...)` で行ラベルも変えられる（R に対応概念が薄い）。

## 関連項目

- [007. 列選択（select）](topic-007.md)
- [009. 列作成・変更（mutate）](topic-009.md)
