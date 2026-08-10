# 073. great_tables で整形 — gt 相当

!!! abstract "この項目の R→Python 対応"
    - **R**: `gt`（宣言的に表題・スパンヘッダ・脚注・数値書式）
    - **Python（推奨）**: **`great_tables`**（gt の Python 移植。メソッド名がほぼ同じ）
    - **要注意**: HTML/画像/Word には強いが、提出用 RTF は非対応。gt の全機能網羅ではない

R で `gt` を使っているなら、great_tables は最も学習コストの低い移行先です。

---

## R ではこう書く

```r
library(gt)

tab |>
  gt(rowname_col = "var") |>
  tab_header(title = "Table 1. Demographics") |>
  tab_spanner(label = "Treatment", columns = c(A, B)) |>
  fmt_number(columns = c(A, B), decimals = 1) |>
  tab_source_note("n (%) or mean (SD).")
```

---

## Python ではこう書く

=== "great_tables"

    ```python
    # pip install great_tables
    from great_tables import GT, md

    gt = (GT(tab, rowname_col="var")
            .tab_header(title="Table 1. Demographics", subtitle="Safety population")
            .tab_spanner(label="Treatment", columns=["A", "B"])
            .tab_source_note(source_note=md("*n (%) or mean (SD).*")))

    gt.as_raw_html()          # HTML 文字列
    # gt.save("table1.png")   # 画像として保存
    # gt.as_word()            # Word 用 XML（python-docx と組み合わせ）
    ```

    メソッド名（`tab_header` / `tab_spanner` / `fmt_number` / `tab_source_note` / `cols_label` / `cols_align`）が **gt とほぼ同名**。パイプ（`|>`）がメソッドチェーン（`.`）になるだけ。

!!! tip "実務ではこれ"
    - **R で gt を使っていたなら great_tables 一択**。書き味がそのまま移る。
    - **出力先**：`as_raw_html()`（レビュー・社内共有）、`save("x.png")`（画像貼り付け）、`as_word()`（Word）。
    - 提出用 **RTF は非対応**。RTF が要件なら R 併用か別手段（→ [076](topic-076.md)）。
    - 数値は great_tables 側の `fmt_number` で整形するか、事前に文字列化して渡す（[072](topic-072.md)）。

---

## 対応早見表

| gt（R） | great_tables（Python） |
|---|---|
| `gt(rowname_col=)` | `GT(rowname_col=)` |
| `tab_header()` | `tab_header()` |
| `tab_spanner()` | `tab_spanner()` |
| `fmt_number()` / `fmt_percent()` | `fmt_number()` / `fmt_percent()` |
| `cols_label()` / `cols_align()` | `cols_label()` / `cols_align()` |
| `tab_footnote()` | `tab_footnote()` / `tab_source_note()` |
| `gtsave("x.png"/".html")` | `save("x.png")` / `as_raw_html()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **入力は「整形済み DF」**：great_tables は集計しない。値・`n (%)`・丸めは pandas で先に作る（[067](topic-067.md)）。
    - **機能網羅ではない**：gt の一部機能は未実装/実装中。込み入ったレイアウトは事前に試す。
    - **画像保存の依存**：`save("x.png")` はブラウザエンジン（Chrome 等）が要ることがある。環境により追加設定が必要。
    - **RTF 非対応**：規制提出の RTF はカバーしない。

## 関連項目

- [071. 見出し・スパンヘッダ・脚注](topic-071.md)
- [074. pandas Styler で整形](topic-074.md)
- [075. Word 出力](topic-075.md)
