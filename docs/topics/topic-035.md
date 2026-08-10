# 035. 欠損の補完 — `coalesce` / `replace_na` / `fill` → `fillna` / `ffill`

!!! abstract "この項目の R→Python 対応"
    - **R**: `coalesce()`（複数列で最初の非 NA）／`tidyr::replace_na()`（定数で埋める）／`tidyr::fill()`（前後の値で埋める）
    - **Python（推奨）**: 定数は **`fillna(0)`**、前方埋めは **`ffill()`**、列優先は **`combine_first()`**
    - **要注意**: `fill`（前方埋め）は**並び順と群**に依存。LOCF は群化＆整列を必ずセットで

（LOCF: Last Observation Carried Forward などで多用）

---

## R ではこう書く

```r
library(dplyr); library(tidyr)

coalesce(c(1, NA, 3), 0)                       # 定数で
tibble(x = c(1, NA, NA, 4)) |> fill(x)         # 前方の値で埋める（LOCF）
replace_na(c(1, NA, 3), 0)                     # NA を定数に
```

出力:

```text
[1] 1 0 3
[1] 1 1 1 4
[1] 1 0 3
```

!!! note "R の勘所"
    - `coalesce(a, b, c)`：**複数ベクトルで最初の非 NA** を採用（列の優先順）。
    - `replace_na(x, 0)` / `replace_na(list(col = 0))`：定数で埋める。
    - `fill(x, .direction = "down")`：前後の観測値で埋める（LOCF は down、NOCB は up）。**群があれば `group_by` を先に**。

---

## Python ではこう書く

=== "pandas"

    ```python
    import numpy as np, pandas as pd

    pd.Series([1, np.nan, 3]).fillna(0)                 # 定数で
    pd.Series([1, np.nan, np.nan, 4]).ffill()           # 前方埋め（LOCF）
    pd.Series([1, np.nan, np.nan, 4]).bfill()           # 後方埋め（NOCB）

    # coalesce：列の優先で最初の非 NA
    df = pd.DataFrame({"a":[1, np.nan, 3], "b":[9, 8, 7]})
    df["a"].combine_first(df["b"])                      # a 優先、欠けたら b
    ```

    出力:

    ```text
    [1.0, 0.0, 3.0]
    [1.0, 1.0, 1.0, 4.0]
    [1.0, 8.0, 3.0]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series([1, None, None, 4])
    print(s.fill_null(0).to_list())                     # 定数
    print(s.fill_null(strategy="forward").to_list())    # 前方埋め
    # coalesce
    pl.DataFrame({"a":[1,None,3], "b":[9,8,7]}).select(pl.coalesce("a", "b"))
    ```

!!! tip "実務ではこれ"
    - **定数で埋める** → `fillna(値)`（列ごとに違う値なら `fillna({"col": v})`）。
    - **LOCF（前回値を持ち越す）** → **群化＆整列してから** `groupby(key)[col].ffill()`。順序と群を外すと他被験者の値が漏れる。
    - **複数列の優先合成**（`coalesce`）→ pandas `combine_first`、polars `pl.coalesce`。
    - 欠損補完は解析仕様（ITT/LOCF/MI）に直結。**「なぜ埋めるか」を明示**し、安易な `fillna(0)` は避ける。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 定数で埋める | `replace_na(x, 0)` | `s.fillna(0)` | `s.fill_null(0)` |
| 前方埋め（LOCF） | `fill(x, "down")` | `s.ffill()` | `fill_null(strategy="forward")` |
| 後方埋め（NOCB） | `fill(x, "up")` | `s.bfill()` | `fill_null(strategy="backward")` |
| 列優先で合成 | `coalesce(a, b)` | `a.combine_first(b)` | `pl.coalesce("a","b")` |
| 群ごとに前方埋め | `group_by \|> fill(x)` | `groupby(k)[x].ffill()` | `fill_null(...).over(k)` |
| 平均で埋める | `replace_na(mean(x))` | `s.fillna(s.mean())` | `fill_null(pl.mean())` |

## つまずきポイント

!!! warning "R と Python の差"
    - **並び順・群依存**：`ffill`/`fill` は行順に沿って埋める。**群化と整列を忘れると被験者をまたいで値が漏れる**。LOCF は `sort_values([subj, time])` → `groupby(subj)[col].ffill()`。
    - **`fillna(0)` の危険**：欠損を 0 として集計に混ぜると平均・割合が狂う。埋めるべきか除外すべきかは解析方針次第。
    - **`coalesce` vs `combine_first`**：`combine_first` は2つずつ。3列以上は連鎖 `a.combine_first(b).combine_first(c)` か polars の `pl.coalesce`。
    - **型**：`fillna` で整数列に float を入れると float 化。nullable `"Int64"` を検討。

## 関連項目

- [005. 欠損値の扱い](topic-005.md)
- [031. ウィンドウ関数](topic-031.md)
- [056. 欠損数の集計](../roadmap.md)
