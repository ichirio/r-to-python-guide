# 018. 数値の丸めと表示桁 — `round` / `signif` → `round` / Decimal

!!! abstract "この項目の R→Python 対応"
    - **R**: `round()`（**round half to even＝銀行丸め**）／`signif()`（有効数字）／`formatC` / `format`（表示）
    - **Python（推奨）**: 組み込み `round()` / `np.round()` も**銀行丸め**。SAS 流の四捨五入が要るなら **`decimal.ROUND_HALF_UP`**
    - **要注意**: **R と Python は互いに一致（両方とも銀行丸め）だが、SAS の `ROUND`（四捨五入）とは 0.5 で食い違う**

臨床の集計で最も揉めるポイント。「丸めた値」と「表示桁」は別問題なので分けて考えます。

---

## R ではこう書く

```r
round(0.5); round(1.5); round(2.5); round(3.5)  # 0 2 2 4（偶数側へ）
round(2.675, 2)                                 # 2.67（浮動小数の綾）
signif(123456, 3)                               # 123000（有効数字3桁）
sprintf("%.0f", 2.5)                            # "2"（printf も偶数丸め）
```

出力:

```text
0 2 2 4
2.67
123000
2
```

!!! note "R の丸めの正体"
    `round()` は IEC 60559（IEEE 754）の **round half to even（銀行丸め）**。`0.5→0`, `2.5→2`。
    「必ず四捨五入（0.5 は切り上げ）」ではありません。SAS 出身者がここで驚きます。

---

## Python ではこう書く

=== "既定（銀行丸め・R と一致）"

    ```python
    import numpy as np
    print(round(0.5), round(1.5), round(2.5), round(3.5))  # 0 2 2 4
    print(round(2.675, 2))                                 # 2.67
    print(np.round([0.5, 1.5, 2.5, 3.5]))                  # [0. 2. 2. 4.]
    ```

    出力:

    ```text
    0 2 2 4
    2.67
    [0. 2. 2. 4.]
    ```

=== "SAS 流の四捨五入（round half up）"

    ```python
    from decimal import Decimal, ROUND_HALF_UP

    def round_half_up(v, nd=0):
        q = Decimal(1).scaleb(-nd)                      # 10^-nd
        return float(Decimal(str(v)).quantize(q, rounding=ROUND_HALF_UP))

    print(round_half_up(2.5), round_half_up(0.5), round_half_up(2.675, 2))
    ```

    出力:

    ```text
    3.0 1.0 2.68
    ```

    `Decimal(str(v))` と**文字列経由**にするのがコツ。`Decimal(2.675)` だと浮動小数の誤差込みになり `2.67` に転びます。

!!! tip "実務ではこれ"
    - **社内標準が SAS 突き合わせ**なら、集計の丸めは `ROUND_HALF_UP` のヘルパを1つ用意して全員で使う。R の `round`／Python の `round`／`np.round` の既定に任せると、0.5 系で SAS と 1 桁ずれる。
    - **有効数字**は Python 標準に直接はない。`float(f"{x:.3g}")` か `np.format_float_positional`、または `Decimal` で。
    - **「丸めた値」と「表示桁」を混ぜない**：計算は数値で丸め、見た目は f-string / Styler（→ [016](topic-016.md), [072](../roadmap.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 小数 n 桁（銀行丸め） | `round(x, n)` | `round(x, n)` / `np.round(x, n)` |
| 四捨五入（SAS流） | （自作 or `janitor::round_half_up`） | `Decimal(str(x)).quantize(..., ROUND_HALF_UP)` |
| 有効数字 n 桁 | `signif(x, n)` | `float(f"{x:.{n}g}")` |
| 切り上げ / 切り捨て | `ceiling(x)` / `floor(x)` | `math.ceil(x)` / `math.floor(x)` |
| 0 方向へ切り捨て | `trunc(x)` | `math.trunc(x)` / `int(x)` |
| 表示だけ丸める | `format(x, nsmall=n)` | `f"{x:.{n}f}"` |

## つまずきポイント

!!! danger "R も Python も既定は銀行丸め。SAS とはズレる"
    - `round(2.5)` は R も Python も **2**（偶数側）。SAS の `round(2.5)` は **3**（四捨五入）。**検証データが SAS 生成なら、0.5 の症例数割合などで不一致が出る**。丸め規則を仕様で固定し、必要なら `ROUND_HALF_UP` を使う。

!!! warning "その他"
    - **浮動小数の綾**：`round(2.675, 2)` が `2.68` でなく `2.67` になるのは、`2.675` が2進で正確に表せないため。R も Python も同じ挙動。桁が効く集計は `Decimal` を検討。
    - **`np.round` と `round`**：どちらも銀行丸めだが、`np.round` は配列向け・`round` はスカラー向け。混在させても結果は一致。
    - **表示の丸め ≠ 値の丸め**：`f"{x:.1f}"` は表示のためだけ。以後の計算には元の値が残る。

## 関連項目

- [016. 書式付き数値・sprintf](topic-016.md)
- [065. 集計値の丸め規則と SAS 一致](../roadmap.md)
- [072. 数値の小数点・右揃え](../roadmap.md)
