# 066. 出力テーブルの考え方 — rtables / gt vs Python の選択肢

!!! abstract "この項目の R→Python 対応"
    - **R**: 集計は dplyr、清書は `rtables` / `gt` / `flextable`、出力は `r2rtf` / `officer`
    - **Python（推奨）**: 集計は pandas/polars、清書は **`great_tables`** / **pandas `Styler`**、出力は **`python-docx`** / **`openpyxl`**
    - **要注意**: R の `rtables` に相当する「臨床特化の表エンジン」は Python に定番がない。**「集計」と「清書」を分けて**組む

第6部の地図。TFL（Tables / Figures / Listings）を Python でどう作るか、道具の対応を最初に俯瞰します。

---

## 役割ごとの対応

| 役割 | R | Python |
|---|---|---|
| 集計（数値を作る） | dplyr / tidyr | **pandas** / **polars** |
| 表の清書（体裁） | `gt` / `flextable` / `rtables` | **`great_tables`** / pandas **`Styler`** |
| Word 出力 | `officer` / `flextable` | **`python-docx`** |
| RTF 出力 | `r2rtf` / `rtables` | 定番なし（[076](topic-076.md) で選択肢） |
| Excel 出力 | `openxlsx` / `writexl` | **`openpyxl`** / **`xlsxwriter`** |
| PDF 出力 | `gt` → LaTeX / `flextable` | matplotlib / reportlab / HTML→PDF |
| 図 | `ggplot2` | **matplotlib** / **plotnine** |

!!! note "R の清書エンジンの位置づけ"
    - `gt`：汎用の美しい表（HTML/Word/PDF）。**great_tables はこの Python 移植**で最も近い。
    - `flextable`：Word/PowerPoint 向けの表。`python-docx` ＋自前整形が対応。
    - `rtables` / `tern`：**臨床特化**（多層ヘッダ・行構造・ページ分割）。Python に直接の等価物はなく、**pandas で構造を作り分ける**ことになる。

---

## Python 側の基本戦略

臨床特化エンジンがない分、**層を分けて**組み立てるのが安定します。

1. **集計層（数値）**：pandas/polars で「行＝解析項目、列＝群、セル＝`n (%)` や `mean (SD)`」の**文字列済みデータフレーム**を作る（→ [067](topic-067.md)〜[069](topic-069.md)）。
2. **体裁層（見た目）**：`great_tables`（HTML/画像/Word）か `Styler`（HTML/Excel）で見出し・スパンヘッダ・脚注・桁揃え（→ [071](topic-071.md), [072](topic-072.md)）。
3. **出力層（ファイル）**：Word は `python-docx`、Excel は `openpyxl`/`xlsxwriter`、RTF は選択肢を検討（→ [075](topic-075.md)〜[077](topic-077.md)）。

!!! tip "実務ではこれ"
    - **「値の入ったDF」を先に完成**させる（丸め・`n (%)` 済み）。体裁と出力はその後。この分離が R の rtables 不在を補う。
    - 提出用 RTF が要件なら、**R（`r2rtf`/rtfreporter）を出力層だけ併用**するのも現実解（Python で集計 → R で清書）。無理に Python 単独にしない。
    - 社内配布・レビューなら great_tables の画像/HTML や Excel が手早い。

---

## つまずきポイント

!!! warning "R と Python の差"
    - **臨床特化エンジンの不在**：`rtables` の多層行・自動ページ分割に当たるものがない。構造は pandas の MultiIndex（→ [071](topic-071.md)）や自前の並べ替えで作る。
    - **「集計と体裁の混在」を避ける**：R の gt はパイプで集計〜体裁を一気に書けるが、Python は分けた方が保守しやすい。数値の検算と見た目を切り離す。
    - **提出フォーマット（RTF）**：規制提出の RTF は Python 側が手薄。要件次第で R 併用を前提に設計する（[076](topic-076.md)）。
    - **フォント・改ページ**：Word/RTF の細かな体裁は各ライブラリの流儀に依存。早めに1本、実物で通しておく。

## 関連項目

- [067. デモグラフィック表の組み立て](topic-067.md)
- [071. 見出し・スパンヘッダ・脚注](topic-071.md)
- [076. RTF 出力](topic-076.md)
- [050. ロング／ワイド実務パターン](topic-050.md)
