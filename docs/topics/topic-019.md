# 019. パーセント表記の作成 — `n (xx.x%)` の組み立て

!!! abstract "この項目の R→Python 対応"
    - **R**: `sprintf("%d (%.1f%%)", n, 100*n/N)` や `glue()` で「件数（割合）」を作る
    - **Python（推奨）**: **f-string** `f"{n} ({100*n/N:.1f}%)"`。列一括は pandas の `map`/ベクトル演算
    - **要注意**: 分母 0 の扱い、丸め方式（[018](topic-018.md)）、"0 (0.0%)" を出すか空欄にするか

デモグラ表・AE 表の中身は結局この「`n (%)`」の量産です。まず1セルの作り方を固め、列に展開します。

---

## R ではこう書く

```r
n <- 3; N <- 7
sprintf("%d (%.1f%%)", n, 100 * n / N)   # "3 (42.9%)"

# glue でも
library(glue)
glue("{n} ({sprintf('%.1f', 100*n/N)}%)")

# ゼロ分母を守る
pct <- if (N > 0) 100 * n / N else NA_real_
```

出力:

```text
3 (42.9%)
3 (42.9%)
```

!!! note "R の定番"
    - `sprintf("%d (%.1f%%)", n, pct)` が最短。`%%` でリテラルの % を出す。
    - 群別の `n (%)` は `group_by |> summarise(n = ..., pct = ...) |> mutate(cell = sprintf(...))` の流れ（→ [057](../roadmap.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    n, N = 3, 7
    print(f"{n} ({100*n/N:.1f}%)")     # "3 (42.9%)"

    # 列に一括：件数列 n と分母 N から "n (xx.x%)" を作る
    import pandas as pd
    df = pd.DataFrame({"cat": ["AE", "CM"], "n": [3, 0]})
    N = 7
    df["cell"] = df.apply(lambda r: f"{r['n']} ({100*r['n']/N:.1f}%)", axis=1)
    print(df["cell"].tolist())
    ```

    出力:

    ```text
    3 (42.9%)
    ['3 (42.9%)', '0 (0.0%)']
    ```

=== "polars"

    ```python
    import polars as pl
    df = pl.DataFrame({"cat": ["AE", "CM"], "n": [3, 0]})
    N = 7
    out = df.with_columns(
        (pl.col("n").cast(pl.Utf8) + " (" +
         (100 * pl.col("n") / N).round(1).cast(pl.Utf8) + "%)").alias("cell")
    )
    print(out["cell"].to_list())
    ```

    ※ polars の `round` は銀行丸め。表示桁を厳密に固定するなら Python 側で `f"{v:.1f}"` を `map_elements` するのが確実。

!!! tip "実務ではこれ"
    - 1 セルは **f-string** `f"{n} ({pct:.1f}%)"` に統一。`pct` は先に計算しておく。
    - **列一括**は「n と割合を数値で用意 → 最後に文字列化」。数値のまま持っておくと並べ替え・検算ができる。
    - **分母 0**：`100*n/N` は N=0 で `ZeroDivisionError`（スカラー）や `inf`（ベクトル）。事前に `N==0` を分岐し、空欄や `"-"` にする。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 1 セル | `sprintf("%d (%.1f%%)", n, pct)` | `f"{n} ({pct:.1f}%)"` |
| 割合（%） | `100 * n / N` | `100 * n / N` |
| 列一括 | `mutate(cell = sprintf(...))` | `df.apply(..., axis=1)` / ベクトル演算 |
| 分母 0 を守る | `ifelse(N>0, ..., NA)` | `np.where(N>0, ..., np.nan)` |
| 0 件を "0 (0.0%)" | そのまま | そのまま（空欄にするなら分岐） |

## つまずきポイント

!!! warning "R と Python の差"
    - **丸め方式**：`%.1f` も `round` も銀行丸めで、**SAS 出力と 0.05 系でズレ得る**（[018](topic-018.md)）。仕様に合わせて `ROUND_HALF_UP` を使うか確認。
    - **分母 0**：スカラーの `n/0` は Python で例外、ベクトルの `n/0` は `inf`/`nan`。R は `NaN`/`Inf`。必ず前段でガード。
    - **`0 (0.0%)` の扱い**：出さない（空欄）か出すかは表の規約次第。`np.where(n==0, "", cell)` で切り替え。
    - **パーセント記号のエスケープ**：R `sprintf` は `%%`、Python f-string は `%` そのまま。

## 関連項目

- [016. 書式付き数値・sprintf](topic-016.md)
- [018. 数値の丸めと表示桁](topic-018.md)
- [057. n (%) 整形のパターン](../roadmap.md)
- [067. デモグラフィック表の組み立て](../roadmap.md)
