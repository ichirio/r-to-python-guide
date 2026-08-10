# 079. 大きな表のページ分割

!!! abstract "この項目の R→Python 対応"
    - **R**: `rtables` / `r2rtf` が**自動でページ分割**（ヘッダ再掲・改ページ）
    - **Python（推奨）**: 自動分割の定番はなし。**行をチャンクに分けて**各ページに見出しを付けて出力
    - **要注意**: Python では改ページを**自前で管理**。行数で分割し、ヘッダを各チャンクに再掲する

長い表を複数ページに割り、各ページに列見出しを繰り返す処理。R の提出ツールは自動ですが、Python は手作業寄りです。

---

## R ではこう書く

```r
library(r2rtf)
# r2rtf / rtables はページ設定に従い自動でページ割り・ヘッダ再掲・改ページ
tab |> rtf_body(page_by = "soc") |> rtf_encode() |> write_rtf("t.rtf")
```

!!! note "R の勘所"
    - `rtables`/`r2rtf` は**行数・用紙サイズから自動でページ分割**し、各ページに列見出しを再掲する。
    - `page_by=` で「この変数が変わったら改ページ」も指定できる。

---

## Python ではこう書く

Python には自動分割の定番がないので、**チャンク分割を自前**で書きます。

=== "行数で分割（汎用）"

    ```python
    def paginate(df, rows_per_page=40):
        for i in range(0, len(df), rows_per_page):
            yield df.iloc[i:i + rows_per_page]

    for page_no, chunk in enumerate(paginate(long_table, 40), start=1):
        # 各ページに見出し・ページ番号を付けて出力（Word/Excel/HTML）
        ...  # add_heading(f"... (page {page_no})") + 表を追加
    ```

=== "キーで改ページ（page_by 相当）"

    ```python
    for soc, chunk in long_table.groupby("soc", sort=False):
        # soc ごとに1ページ（またはページグループ）
        ...  # 見出し = soc、表 = chunk
    ```

!!! tip "実務ではこれ"
    - **行数で機械的に分割**（`iloc[i:i+n]`）するか、**キーで改ページ**（`groupby`）。用途で選ぶ。
    - Word なら各チャンクの前に `add_heading` と列見出し行を再掲、末尾に `add_page_break()`。Excel なら `set_h_pagebreaks`（xlsxwriter）。
    - **提出品質の自動ページ割りが必須**なら、素直に **R（rtables/r2rtf）に清書を任せる**（[076](topic-076.md)）。Python の自前実装は保守が重い。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 自動ページ分割 | `rtables` / `r2rtf` | なし（自前チャンク分割） |
| 行数で分割 | 自動 | `df.iloc[i:i+n]` をループ |
| キーで改ページ | `page_by=` | `groupby(key)` |
| ヘッダ再掲 | 自動 | 各チャンクで見出しを再出力 |
| Word の改ページ | — | `doc.add_page_break()` |
| Excel の改ページ | — | `ws.set_h_pagebreaks()`（xlsxwriter） |

## つまずきポイント

!!! warning "R と Python の差"
    - **自動分割がない**：R の提出ツールが当然にやることを、Python では手で組む。行数・用紙・フォントを考慮した厳密なページ割りは特に大変。
    - **ヘッダ再掲**：各ページに列見出しを繰り返すのを忘れない。関数化して毎ページ呼ぶ。
    - **改ページ位置**：意味の切れ目（SOC など）で割りたいなら `groupby`。単純な行数分割は途中で群が切れる。
    - **提出要件**：ページ番号・脚注の再掲など規定が多い。要件が厳しいなら R 併用が現実的。

## 関連項目

- [076. RTF 出力](topic-076.md)
- [070. リスティング](topic-070.md)
- [066. 出力テーブルの考え方](topic-066.md)
