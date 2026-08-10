# 077. Excel 出力 — `openxlsx` / `writexl` → `openpyxl` / `xlsxwriter`

!!! abstract "この項目の R→Python 対応"
    - **R**: `writexl`（手軽）／`openxlsx`（書式・数式・複数シート）
    - **Python（推奨）**: 手軽なら **`df.to_excel()`**、書式込みは **`xlsxwriter`** or **`openpyxl`** エンジン
    - **要注意**: 「表示形式（number_format）」と「値」は別。文字列化すると Excel 上で計算不可になる

Excel は listing・大きい表・社内共有に強い出力先。pandas から直接書けます。

---

## R ではこう書く

```r
library(writexl)
write_xlsx(list(T1 = tab, AE = ae_tab), "output.xlsx")   # 複数シート

library(openxlsx)                                         # 書式込み
wb <- createWorkbook(); addWorksheet(wb, "T1")
writeData(wb, "T1", tab, startRow = 2)
writeData(wb, "T1", "Table 1. Demographics", startCol = 1, startRow = 1)
saveWorkbook(wb, "output.xlsx", overwrite = TRUE)
```

---

## Python ではこう書く

=== "pandas + xlsxwriter（書式込み）"

    ```python
    import pandas as pd
    df = pd.DataFrame({"var":["Age","Sex M"], "A":[52.3, 66.7], "B":[49.5, 50.0]})

    with pd.ExcelWriter("output.xlsx", engine="xlsxwriter") as xw:
        df.to_excel(xw, sheet_name="T1", index=False, startrow=1)
        wb, ws = xw.book, xw.sheets["T1"]
        ws.write(0, 0, "Table 1. Demographics")          # タイトル行
        fmt = wb.add_format({"num_format": "0.0", "align": "right"})
        ws.set_column("B:C", 12, fmt)                     # 表示形式（値は数値のまま）
    ```

    → `output.xlsx` が生成される（数値は数値のまま、表示だけ小数1桁）。

=== "複数シート"

    ```python
    with pd.ExcelWriter("output.xlsx") as xw:      # 既定エンジンは openpyxl
        tab.to_excel(xw, sheet_name="T1")
        ae_tab.to_excel(xw, sheet_name="AE")
    ```

!!! tip "実務ではこれ"
    - **手早く出す** → `df.to_excel("x.xlsx", index=False)`。複数シートは `ExcelWriter`。
    - **書式（表示形式・列幅・色）** → `xlsxwriter`（新規作成に強い）か `openpyxl`（既存ブック編集に強い）。
    - **数値は数値のまま**、`num_format`（`"0.0"` 等）で表示だけ整える。f-string で文字列化すると Excel で再計算できなくなる。
    - listing など大きい表は Excel が扱いやすい（[070](topic-070.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 手軽に出力 | `write_xlsx()` | `df.to_excel()` |
| 複数シート | `write_xlsx(list(...))` | `ExcelWriter` に複数 `to_excel` |
| 書式・列幅 | `openxlsx` | `xlsxwriter` / `openpyxl` |
| 表示形式 | `openxlsx::createStyle(numFmt=)` | `add_format({"num_format":})` |
| 既存ブック編集 | `loadWorkbook()` | `openpyxl.load_workbook()` |
| 数式を書く | `writeFormula()` | `ws.write_formula()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **エンジン2種**：`xlsxwriter`（新規作成・書式に強いが既存編集不可）と `openpyxl`（既存編集可）。用途で選ぶ。`to_excel(engine=)` で指定。
    - **表示形式 vs 文字列**：`num_format` は値を保ったまま見た目を整える。文字列化（f-string）すると計算不可。TFL の最終提出なら文字列固定、再利用データなら数値＋書式。
    - **startrow/startcol**：タイトル行を上に空けるなら `startrow=1`。R の `startRow` は 1 始まり、pandas は 0 始まり。
    - **インデックス**：`to_excel` は既定で行 index を書き出す。不要なら `index=False`。

## 関連項目

- [070. リスティング](topic-070.md)
- [074. pandas Styler で整形](topic-074.md)
- [096. CSV / 固定長 / 区切りの読み書き](../roadmap.md)
