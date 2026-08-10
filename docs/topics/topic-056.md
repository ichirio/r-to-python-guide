# 056. 欠損数の集計 — `sum(is.na)` → `isna().sum`

!!! abstract "この項目の R→Python 対応"
    - **R**: `sum(is.na(x))`（1 列）／`colSums(is.na(df))` / `sapply(df, \(x) sum(is.na(x)))`（全列）
    - **Python（推奨）**: pandas **`df.isna().sum()`**（列ごとの欠損数）
    - **要注意**: 「欠損数」と「非欠損数（N）」を混同しない。`isna().sum()` と `count()` は逆向き

データの品質確認と、TFL の「Missing」行の材料。

---

## R ではこう書く

```r
sum(is.na(dm3$age))                     # 1 列の欠損数
colSums(is.na(dm3))                     # 全列
sapply(dm3, function(x) sum(is.na(x)))  # 同上（型混在に強い）
mean(is.na(dm3$age))                    # 欠損割合
```

出力（age に1つ欠損を入れた場合）:

```text
subjid    arm    age    sex
     0      0      1      0
```

!!! note "R の勘所"
    - `is.na(x)` は論理ベクトル、`sum()` で個数、`mean()` で割合。
    - 全列は `colSums(is.na(df))`。`NaN` も `is.na` で TRUE。
    - 行ごとの欠損数は `rowSums(is.na(df))`。

---

## Python ではこう書く

=== "pandas"

    ```python
    dm3["age"].isna().sum()        # 1 列の欠損数
    dm3.isna().sum()               # 列ごと（Series）
    dm3.isna().mean()              # 列ごとの欠損割合
    dm3.isna().sum(axis=1)         # 行ごとの欠損数

    dm3.notna().sum()              # 非欠損数（N）＝ count() と同じ
    ```

    出力（age に1つ欠損）:

    ```text
    {'subjid': 0, 'arm': 0, 'age': 1, 'sex': 0}
    ```

=== "polars"

    ```python
    import polars as pl
    dm3p.null_count()              # 各列の null 数（1 行の DataFrame）
    dm3p.select(pl.all().is_null().sum())
    ```

!!! tip "実務ではこれ"
    - **列ごとの欠損数** → `df.isna().sum()`。割合は `.mean()`。データ受領時の品質チェックの定番。
    - **N（非欠損数）** → `df.notna().sum()` または `df.count()`（`count` は非欠損を数える）。
    - polars は `null_count()` が一発。ただし `NaN`（非数）は null と別なので、必要なら `is_nan()` も見る（[005](topic-005.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 1 列の欠損数 | `sum(is.na(x))` | `x.isna().sum()` | `x.null_count()` |
| 全列の欠損数 | `colSums(is.na(df))` | `df.isna().sum()` | `df.null_count()` |
| 欠損割合 | `mean(is.na(x))` | `df.isna().mean()` | — |
| 行ごとの欠損数 | `rowSums(is.na(df))` | `df.isna().sum(axis=1)` | — |
| 非欠損数（N） | `sum(!is.na(x))` | `df.count()` / `notna().sum()` | `df.count()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **欠損数 vs N**：`isna().sum()` は欠損数、`count()`/`notna().sum()` は非欠損数。TFL では「N（解析対象）」と「Missing」を別行で出すことが多いので取り違えない。
    - **`NaN` と `None`/`null`**：pandas は両方 `isna()` で拾う。polars は `null`（欠損）と `NaN`（非数）が別。`is_null()` と `is_nan()` を使い分ける。
    - **文字列の "NA"/空文字**：`""` や文字列 `"NA"` は欠損ではない。読込時に `na_values` で欠損化するか、明示的に判定する。
    - **`axis` の向き**：列ごとは既定（`axis=0`）、行ごとは `axis=1`。

## 関連項目

- [005. 欠損値の扱い](topic-005.md)
- [035. 欠損の補完](topic-035.md)
- [051. 記述統計の一括](topic-051.md)
