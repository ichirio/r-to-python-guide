# 016. 書式付き数値・sprintf — `sprintf` / `formatC` / `format` → format spec

!!! abstract "この項目の R→Python 対応"
    - **R**: `sprintf()`（C 由来の `%` 書式）／`formatC()`（幅・桁・桁区切り）／`format()`（`nsmall` など）
    - **Python（推奨）**: **f-string の書式指定** `f"{x:.1f}"`（`sprintf` と `format` を一手に）
    - **要注意**: 書式指定子はほぼ共通だが、R は `%%` でパーセント記号、Python は `%` そのまま

TFL の数値は「小数第1位まで」「幅を揃える」「桁区切り」が定番。R の3系統を f-string に集約できます。

---

## R ではこう書く

```r
sprintf("%.1f", 3.14159)     # 小数1桁
sprintf("%05.1f", 3.1)       # 幅5・ゼロ埋め
sprintf("%+.2f", 3.1)        # 符号付き
sprintf("%e", 12345)         # 指数表記
formatC(1234567, big.mark = ",", format = "d")  # 桁区切り
format(3.1, nsmall = 2)      # 末尾ゼロを保つ
```

出力:

```text
3.1
003.1
+3.10
1.234500e+04
1,234,567
3.10
```

!!! note "使い分け"
    - `sprintf()`：`%d %s %f %e %g` と幅・精度。C を知っていれば直感的。TFL の桁揃えの主役。
    - `formatC()`：`big.mark`（桁区切り）、`flag="0"`（ゼロ埋め）、`format=`（"d"/"f"/"e"/"g"）。
    - `format()`：`nsmall`（最低小数桁）、`scientific=`。**末尾ゼロを残したい**ときに。

---

## Python ではこう書く

=== "f-string（推奨）"

    ```python
    print(f"{3.14159:.1f}")     # 小数1桁
    print(f"{3.1:05.1f}")       # 幅5・ゼロ埋め
    print(f"{3.1:+.2f}")        # 符号付き
    print(f"{12345:e}")         # 指数表記
    print(f"{1234567:,d}")      # 桁区切り
    print(f"{3.1:.2f}")         # 末尾ゼロを保つ
    ```

    出力:

    ```text
    3.1
    003.1
    +3.10
    1.234500e+04
    1,234,567
    3.10
    ```

=== "pandas 列に一括"

    ```python
    import pandas as pd
    s = pd.Series([3.14159, 22.0, 1234.5])
    print(s.map(lambda v: f"{v:,.1f}").tolist())   # 各要素に書式
    # ['3.1', '22.0', '1,234.5']
    ```

    表示だけ整えたいなら `df.style.format("{:.1f}")`（→ [074](../roadmap.md)）。値そのものを文字列にするなら `map`/`apply`。

!!! tip "実務ではこれ"
    - **f-string の `:書式`** に一本化：`{x:.1f}`（精度）、`{x:8.2f}`（幅）、`{x:0>3}`（ゼロ埋め）、`{x:,}`（桁区切り）、`{x:+.2f}`（符号）。R の `sprintf`+`formatC`+`format` を一手に置き換えられる。
    - 列全体は `s.map(lambda v: f"{v:.1f}")`。表の**表示**だけなら Styler。

---

## 対応早見表

| やりたいこと | R | Python（f-string） |
|---|---|---|
| 小数 n 桁 | `sprintf("%.1f", x)` | `f"{x:.1f}"` |
| 幅・ゼロ埋め | `sprintf("%05.1f", x)` | `f"{x:05.1f}"` |
| 符号付き | `sprintf("%+.2f", x)` | `f"{x:+.2f}"` |
| 桁区切り | `formatC(x, big.mark=",")` | `f"{x:,}"` |
| 指数 | `sprintf("%e", x)` | `f"{x:e}"` |
| 末尾ゼロ保持 | `format(x, nsmall=2)` | `f"{x:.2f}"` |
| パーセント記号 | `sprintf("%.1f%%", x)` | `f"{x:.1f}%"` |

## つまずきポイント

!!! warning "R と Python の差"
    - **`%%` の要否**：R の `sprintf` はリテラルの % を `%%` と書く。Python の f-string は `%` をそのまま書く（`f"{x:.1f}%"`）。
    - **丸め方式**：`%.1f` の丸めは環境の C ライブラリ依存で、**四捨五入とは限らない**（round-half-to-even になりがち）。数値そのものを丸める話は [018](topic-018.md) を参照。
    - **整数と `:,d`**：`f"{x:,d}"` は整数専用。float に `d` を使うと `ValueError`。`f"{x:,.0f}"` にする。
    - **文字列の幅揃え**：数値以外の桁揃えは `f"{s:>8}"`（右寄せ）等。R の `formatC(..., flag="-")` に対応。

## 関連項目

- [017. ゼロ埋め・桁揃え](topic-017.md)
- [018. 数値の丸めと表示桁](topic-018.md)
- [019. パーセント表記の作成](topic-019.md)
