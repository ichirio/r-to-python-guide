# 065. 集計値の丸め規則と SAS 一致 — `round` 方式

!!! abstract "この項目の R→Python 対応"
    - **R**: `round()` は**銀行丸め（round half to even）**。SAS の `ROUND` は四捨五入（round half up）
    - **Python（推奨）**: `round`/`np.round` も銀行丸め。SAS 一致には **`decimal.ROUND_HALF_UP`** のヘルパを共通化
    - **要注意**: R と Python は一致するが、**SAS 生成の検証データとは 0.5 系でズレる**。集計パイプラインで丸め規則を固定する

[018](topic-018.md) の丸めを、**集計・TFL の運用**の観点でまとめます。デモグラ表・AE 表の数値を SAS と突き合わせるときの実務指針。

---

## 問題：3 者の既定がそろわない

| 環境 | 既定の丸め | `round(2.5)` |
|---|---|---|
| R `round()` | 銀行丸め（half to even） | 2 |
| Python `round` / `np.round` | 銀行丸め（half to even） | 2 |
| SAS `ROUND()` | 四捨五入（half up） | 3 |

**R と Python は一致**するが、**SAS とは 0.5 ちょうどでズレる**。臨床では検証データが SAS 生成のことが多く、割合（`xx.5%`）や平均でズレが表面化します。

---

## 解決：丸めヘルパを1つに固定する

=== "Python（SAS 流 四捨五入）"

    ```python
    from decimal import Decimal, ROUND_HALF_UP

    def round_half_up(v, nd=0):
        q = Decimal(1).scaleb(-nd)
        return float(Decimal(str(v)).quantize(q, rounding=ROUND_HALF_UP))

    # 集計 → 丸め を1か所に通す
    df["pct_disp"] = df["pct"].map(lambda v: round_half_up(v, 1))
    ```

    `Decimal(str(v))` と**文字列経由**にするのが要（浮動小数の誤差を持ち込まない）。

=== "R（SAS 流 四捨五入）"

    ```r
    round_half_up <- function(x, digits = 0) {
      z <- x * 10^digits
      floor(z + 0.5 + 1e-8) / 10^digits   # 素朴版（要件に応じ丁寧に）
    }
    # janitor::round_half_up() を使う手もある
    ```

!!! tip "実務ではこれ"
    1. **丸め規則を SAP/仕様で1つに決める**（多くは四捨五入 half up）。
    2. **共通ヘルパを1つ**作り、集計の最終段だけで丸める（途中は丸めない）。
    3. **「値の丸め」と「表示の丸め」を分ける**：集計値は数値で丸め、表示は f-string（→ [016](topic-016.md)）。二重丸めを避ける。
    4. **検証**：SAS 出力と数セル（`xx.5` になるもの）を突き合わせ、方式差が出ないか確認。

---

## つまずきポイント

!!! danger "既定に任せない"
    - R も Python も既定は銀行丸め。**「四捨五入のつもり」で `round` を使うと SAS 一致を外す**。方式を明示したヘルパを通す。

!!! warning "その他"
    - **浮動小数の綾**：`round(2.675, 2)` が `2.67` になる（2 進表現）。桁が効く集計は `Decimal` を検討。R も Python も同挙動。
    - **途中丸めの累積**：中間で丸めると誤差が積もる。丸めは**最終段のみ**。
    - **負値**：half up の「away from zero か toward positive か」を仕様で確認（`ROUND_HALF_UP` は絶対値基準で 0 から離す）。
    - **割合の丸めと合計**：各セルを丸めると行合計が 100.0% にならないことがある（丸め残差）。TFL 規約に従い、最大セルで調整するなどのルールを決める。

## 関連項目

- [018. 数値の丸めと表示桁](topic-018.md)
- [057. n (%) 整形のパターン](topic-057.md)
- [072. 数値の小数点・右揃え](../roadmap.md)
