# R→Python 対応ガイド（統計・TFL実務）

R（特に **Tidyverse** でのデータ加工、**TFL**〈Tables / Figures / Listings〉作成）を、
**Python でどう書くか**を、**R を起点に**まとめた対応リファレンスです。

対象は、R を使う統計プログラマ。「R でおなじみのやり方」を出発点に、
pandas / polars での最善手へ橋渡しします。

---

## このガイドの読み方

各項目は同じ構成です。**知りたい R の操作から引く**のが基本の使い方です。

1. **R ではこう書く** — メジャーな方法を複数、**使い分け**とともに提示
2. **Python ではこう書く** — `pandas` / `polars` をタブで併記し、**推奨を明言**
3. **対応早見表** — R / pandas / polars の 3 列対応表
4. **つまずきポイント** — 0/1 始まり、`NA` の伝播、丸め方式など、R と Python の差でハマる所

!!! example "たとえば「文字列結合」なら"
    R には `paste()` `paste0()` `str_c()` `glue()` と複数の道があり、それぞれ長所が違います。
    このガイドはまず **R 側の使い分け**を整理し、そのうえで
    **Python なら f-string / `Series.str.cat` / polars `pl.concat_str`** のどれが良いかを示します。
    → [006. 文字列結合](topics/topic-006.md)

---

## 前提とする道具

| 役割 | R | Python |
|---|---|---|
| データ加工（主） | dplyr / tidyr（Tidyverse） | **pandas** / **polars** |
| 文字列 | stringr / glue | 標準 `str` / `re` / pandas `.str` |
| 表（TFL） | rtables / gt / flextable / r2rtf | great_tables / pandas Styler / python-docx / openpyxl |
| 図 | ggplot2 | matplotlib / plotnine / seaborn |
| 統計 | stats / survival | scipy.stats / statsmodels / lifelines |

!!! note "pandas か polars か"
    - **pandas**: 情報量が最も多く、既存資産・周辺エコシステムが厚い。まず読めること優先ならこれ。
    - **polars**: 式（expression）ベースで dplyr に発想が近く、速い。新規の重い処理はこちらが快適。
    - 本ガイドは**両方併記**し、項目ごとに実務での推奨を書きます。

---

## まず読む

- [ロードマップ（全100項目）](roadmap.md) — 全体像とカテゴリ一覧
- [001. R と Python の考え方の違い](topics/topic-001.md) — ここから読むと以降が楽

---

## ローカルで見る・増やす

```bash
pip install -r requirements.txt
mkdocs serve
```

項目の追加ルールは、リポジトリの `CONTRIBUTING.md` と `TEMPLATE.md` を参照してください。
コード出力は R / Python で実行して検証したものを載せています。
