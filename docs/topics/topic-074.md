# 074. pandas Styler で整形

!!! abstract "この項目の R→Python 対応"
    - **R**: `gt` / `formattable` / `kableExtra`（数値書式・条件付き色・Excel/HTML 出力）
    - **Python（推奨）**: pandas **`DataFrame.style`（Styler）**（`format` / `background_gradient` / `to_excel` / `to_html`）
    - **要注意**: Styler は**表示・出力用**。値そのものは変えない。Excel 出力に強い

pandas 標準の表整形。great_tables ほど TFL 特化ではないが、**Excel 出力や条件付き書式**に手軽です。

---

## R ではこう書く

```r
library(gt)      # or formattable / kableExtra
tab |>
  gt() |>
  fmt_number(decimals = 1) |>
  data_color(columns = pct, palette = c("white","red"))   # 条件付き色
```

---

## Python ではこう書く

=== "pandas Styler"

    ```python
    df = pd.DataFrame({"stat":["mean","sd"], "A":[52.0, 7.94], "B":[49.5, 16.26]})

    styled = (df.style
                .format({"A": "{:.1f}", "B": "{:.1f}"})        # 数値書式
                .set_caption("Table 1")                        # 表題
                .set_properties(subset=["A","B"], **{"text-align": "right"})
                .background_gradient(subset=["A","B"], cmap="Blues"))  # 条件付き色

    styled.to_html("t.html")     # HTML
    styled.to_excel("t.xlsx", index=False)   # Excel（書式付き）
    ```

!!! tip "実務ではこれ"
    - **Excel を書式付きで出す**なら Styler の `to_excel` が手軽（数値書式・色が反映される）。
    - **条件付き書式**（閾値超えを色付け）は `background_gradient` / `apply`（セルごとの CSS を返す関数）。
    - HTML レビューは `to_html`。ただし**多層ヘッダや脚注は great_tables の方が得意**。用途で使い分け。
    - Styler は**表示・出力専用**。CSV に落とすと元の数値（書式は消える）。

---

## 対応早見表

| やりたいこと | R | Python（Styler） |
|---|---|---|
| 数値書式 | `fmt_number()` | `.format("{:.1f}")` |
| 表題 | `tab_header()` | `.set_caption()` |
| 右揃え | `cols_align()` | `.set_properties(text-align)` |
| 条件付き色 | `data_color()` | `.background_gradient()` / `.applymap` |
| HTML 出力 | `gtsave(".html")` | `.to_html()` |
| Excel 出力 | `openxlsx` | `.to_excel()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **表示専用**：Styler は見た目だけ。値を固定したいなら f-string で文字列化してから（[072](topic-072.md)）。
    - **多層ヘッダ・脚注**：Styler は苦手。TFL の凝った体裁は great_tables（[073](topic-073.md)）へ。
    - **`applymap`/`map` の非推奨**：pandas のバージョンで Styler の要素別関数名が変わる（`applymap`→`map`）。使用版のドキュメントを確認。
    - **Excel の数値型**：`format` で文字列化するとセルが文字列になり、Excel 上で計算できない。数値のまま「表示形式」を付けたいなら xlsxwriter の number_format（→ [077](topic-077.md)）。

## 関連項目

- [072. 数値の小数点・右揃え](topic-072.md)
- [073. great_tables で整形](topic-073.md)
- [077. Excel 出力](topic-077.md)
