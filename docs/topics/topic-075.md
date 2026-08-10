# 075. Word 出力 — `officer` / `flextable` → `python-docx`

!!! abstract "この項目の R→Python 対応"
    - **R**: `officer` + `flextable`（Word 文書に表を差し込む）
    - **Python（推奨）**: **`python-docx`**（`import docx`）で見出し・段落・表を組む
    - **要注意**: 表はセルにテキストを流し込む方式。スタイル名は Word テンプレート依存

Word 形式の TFL / 報告書。python-docx で見出し・表・段落を積みます。

---

## R ではこう書く

```r
library(officer); library(flextable)

doc <- read_docx()
ft <- flextable(tab)                      # データフレームを表に
doc <- doc |>
  body_add_par("Table 1. Demographics", style = "heading 1") |>
  body_add_flextable(ft)
print(doc, target = "table1.docx")
```

---

## Python ではこう書く

=== "python-docx"

    ```python
    # pip install python-docx  →  import は docx
    from docx import Document

    df = pd.DataFrame({"var":["Age","Sex M"], "A":["52.3","2 (66.7%)"],
                       "B":["49.5","1 (50.0%)"]})

    doc = Document()
    doc.add_heading("Table 1. Demographics", level=1)

    t = doc.add_table(rows=1, cols=len(df.columns))
    t.style = "Light Grid Accent 1"
    for j, c in enumerate(df.columns):           # ヘッダ行
        t.rows[0].cells[j].text = str(c)
    for _, r in df.iterrows():                    # データ行
        cells = t.add_row().cells
        for j, c in enumerate(df.columns):
            cells[j].text = str(r[c])

    doc.add_paragraph("n (%) or mean (SD).")      # 脚注
    doc.save("table1.docx")
    ```

    → 実行すると `table1.docx`（数万バイト）が生成されます。

!!! tip "実務ではこれ"
    - **pip 名は `python-docx`、import は `docx`**（紛らわしい）。
    - 表はセルへ**文字列を流し込む**方式。値は事前に整形（`n (%)` 済み、[067](topic-067.md)）。
    - スタイル（`t.style`）は**テンプレートに存在する名前**を使う。自社テンプレートの `.docx` を `Document("template.docx")` で開いて追記すると体裁が揃う。
    - great_tables の `as_word()` と併用して凝った表を貼る手もある（[073](topic-073.md)）。

---

## 対応早見表

| やりたいこと | R（officer/flextable） | Python（python-docx） |
|---|---|---|
| 文書を作る | `read_docx()` | `Document()` |
| テンプレから | `read_docx("tpl.docx")` | `Document("tpl.docx")` |
| 見出し | `body_add_par(style=)` | `add_heading(level=)` |
| 表を追加 | `body_add_flextable()` | `add_table()` + セル流し込み |
| 段落・脚注 | `body_add_par()` | `add_paragraph()` |
| 保存 | `print(doc, target=)` | `doc.save()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **import 名**：`pip install python-docx` だが `import docx`。`pip install docx`（別物・古い）を入れない。
    - **表の作り方**：flextable は DF を直接表に。python-docx は**行・セルを自分でループ**して流し込む。ヘルパ関数を1つ書くと楽。
    - **スタイル名**：`t.style` に無いスタイル名を指定するとエラー。テンプレートにある名前を使う。
    - **細かな体裁**（列幅・罫線・フォント）は python-docx の低レベル API を触る必要があり、flextable より手数が多い。
    - **RTF ではない**：Word（.docx）と RTF は別形式。提出が RTF 指定なら [076](topic-076.md)。

## 関連項目

- [066. 出力テーブルの考え方](topic-066.md)
- [076. RTF 出力](topic-076.md)
- [073. great_tables で整形](topic-073.md)
