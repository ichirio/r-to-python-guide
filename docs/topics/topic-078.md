# 078. PDF 出力（選択肢）

!!! abstract "この項目の R→Python 対応"
    - **R**: `gt` → LaTeX/PDF、`flextable` → PDF、R Markdown / Quarto で文書化
    - **Python（推奨）**: 図中心なら **matplotlib の PdfPages**、表中心なら **HTML→PDF**（WeasyPrint 等）、文書なら **Quarto**
    - **要注意**: 「表の PDF」は Python でやや手薄。図の PDF は matplotlib が最短

PDF は用途で道具が分かれます。図の束か、表の清書か、レポート全体か。

---

## R ではこう書く

```r
# 表 → PDF（gt 経由）
gt(tab) |> gtsave("table1.pdf")

# 図を1つの PDF に束ねる
pdf("figures.pdf"); print(p1); print(p2); dev.off()

# レポート全体：R Markdown / Quarto
```

---

## Python ではこう書く

=== "図の PDF（matplotlib）"

    ```python
    from matplotlib.backends.backend_pdf import PdfPages
    import matplotlib.pyplot as plt

    with PdfPages("figures.pdf") as pdf:
        for data in datasets:
            fig, ax = plt.subplots()
            ax.plot(data["x"], data["y"])
            pdf.savefig(fig)          # 1 図＝1 ページ
            plt.close(fig)
    ```

=== "表・レポートの PDF"

    ```python
    # 表中心：great_tables → HTML → PDF（WeasyPrint など）
    #   html = GT(tab).as_raw_html()
    #   from weasyprint import HTML; HTML(string=html).write_pdf("t.pdf")

    # レポート全体：Quarto（.qmd）で Python を実行して PDF/Word/HTML に
    ```

!!! tip "実務ではこれ"
    - **図を束ねた PDF** → `matplotlib.backends.backend_pdf.PdfPages`（1 図 1 ページ）。R の `pdf()`/`dev.off()` に対応。
    - **表の PDF** → great_tables/HTML を経由して HTML→PDF 変換（WeasyPrint 等）。直接の定番は薄い。
    - **レポート全体（文＋表＋図）** → **Quarto**（R でも Python でも同じツール）。R Markdown からの移行先として自然。
    - 提出用の厳密な PDF/RTF レイアウトは R 併用が無難（[076](topic-076.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 図を PDF に束ねる | `pdf(); print(p); dev.off()` | `PdfPages` + `savefig` |
| 単一図の PDF | `ggsave("f.pdf")` | `fig.savefig("f.pdf")` |
| 表の PDF | `gt \|> gtsave(".pdf")` | HTML→PDF（WeasyPrint 等） |
| レポート全体 | R Markdown / Quarto | Quarto / nbconvert |

## つまずきポイント

!!! warning "R と Python の差"
    - **表 PDF が手薄**：Python で「表を綺麗に PDF」は定番が弱い。HTML→PDF 変換ツール（外部依存）を挟むことが多い。
    - **外部依存**：WeasyPrint 等は追加インストールやシステムライブラリが要ることがある。環境構築を早めに確認。
    - **フォント埋め込み**：PDF のフォント（日本語含む）埋め込みは matplotlib/変換ツールの設定次第。文字化けに注意。
    - **ベクター vs ラスタ**：`savefig("f.pdf")` はベクター。`dpi` はラスタ要素にのみ効く。図の品質はベクターで保つ。

## 関連項目

- [080. 図の基本（ggplot2 → matplotlib/plotnine）](topic-080.md)
- [084. 図の保存（ggsave → savefig）](topic-084.md)
- [076. RTF 出力](topic-076.md)
