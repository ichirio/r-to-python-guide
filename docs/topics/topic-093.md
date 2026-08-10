# 093. NULL / 欠損の安全な処理 — `%||%` / `coalesce` → `or` / `fillna`

!!! abstract "この項目の R→Python 対応"
    - **R**: `NULL`（存在しない）と `NA`（欠損）は別。`%||%`（rlang）で既定値、`coalesce()` でベクトル
    - **Python（推奨）**: `None` の既定は `x if x is not None else default`、列の欠損は `fillna` / `combine_first`
    - **要注意**: `x or default` は `x` が `""`/`0`/`False` でも既定に落ちる（falsy 全般）。**None だけ**を判定するなら `is None`

「値が無いときの既定値」。スカラーの None とベクトルの欠損で道具が違います。

---

## R ではこう書く

```r
`%||%` <- function(a, b) if (is.null(a)) b else a   # rlang にもある
NULL %||% "default"     # "default"
"x"  %||% "default"     # "x"

coalesce(c(1, NA, 3), 0)  # ベクトルの欠損を埋める → 1 0 3
```

出力:

```text
[1] "default"
[1] "x"
```

!!! note "R の勘所"
    - `%||%`：**NULL のときだけ**右を返す（`NA` には効かない）。引数の既定値によく使う。
    - `coalesce(a, b, ...)`：**ベクトルの NA** を後続の値で埋める（[035](topic-035.md)）。
    - `NULL`（長さ0・存在しない）と `NA`（欠損）を区別。

---

## Python ではこう書く

=== "スカラーの None"

    ```python
    # None のときだけ既定に（推奨：is None で厳密判定）
    y = x if x is not None else "default"

    # 短絡評価（簡潔だが falsy 全般に効く）
    y = x or "default"           # x が "", 0, False でも "default" に！
    ```

=== "列の欠損"

    ```python
    import pandas as pd
    pd.Series([1, None, 3]).fillna(0)             # → 1, 0, 3
    df["a"].combine_first(df["b"])                # coalesce（列優先）
    ```

!!! tip "実務ではこれ"
    - **引数の既定（None のみ）** → `x if x is not None else default`。`or` は手軽だが、`0`/`""`/`False` を有効値として扱いたい場面で誤爆する。
    - **可変既定引数の罠**を避けるため、関数の既定は `None` にして中で生成（[086](topic-086.md)）:
      ```python
      def f(cols=None):
          cols = cols if cols is not None else ["a", "b"]
      ```
    - **列の欠損** → `fillna`（定数）/`combine_first`（列優先、`coalesce` 相当、[035](topic-035.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| None なら既定 | `x %\|\|% default` | `x if x is not None else default` |
| 列の欠損を埋める | `coalesce(x, 0)` | `s.fillna(0)` |
| 列優先で合成 | `coalesce(a, b)` | `a.combine_first(b)` |
| None か判定 | `is.null(x)` | `x is None` |
| 欠損か判定 | `is.na(x)` | `pd.isna(x)` |
| 空か判定 | `length(x) == 0` | `not x` / `len(x) == 0` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`or` は falsy 全般に効く**：`0 or 5` は `5`、`"" or "x"` は `"x"`。「None のときだけ」なら `is None` を使う。R の `%||%` は NULL 限定なので挙動が違う。
    - **`None` と `NaN` は別**：`None is None` は True だが `np.nan is None` は False。列の欠損判定は `pd.isna()`（両方拾う）。
    - **`is` と `==`**：None 判定は `is None`（同一性）。`== None` は非推奨（オブジェクトによって挙動が変わる）。
    - **NULL/NA の区別**（R）：`%||%` は NULL、`coalesce` は NA。Python は `None`/`NaN` の判定関数を使い分ける（[005](topic-005.md)）。

## 関連項目

- [005. 欠損値の扱い](topic-005.md)
- [035. 欠損の補完](topic-035.md)
- [086. 関数定義](topic-086.md)
