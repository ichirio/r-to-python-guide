# 084. 図の保存 — `ggsave` → `savefig`

!!! abstract "この項目の R→Python 対応"
    - **R**: `ggsave("f.png", width=, height=, dpi=)`
    - **Python（推奨）**: matplotlib **`fig.savefig("f.png", dpi=, bbox_inches="tight")`**／plotnine **`ggplot.save()`**
    - **要注意**: サイズの単位（R はインチ/cm、matplotlib は figsize がインチ）。ベクター形式は `dpi` が効かない要素あり

作った図をファイルに落とす。解像度・サイズ・余白の指定が主な論点です。

---

## R ではこう書く

```r
ggsave("fig.png", plot = p, width = 8, height = 5, dpi = 300)   # インチ
ggsave("fig.pdf", plot = p, width = 20, height = 12, units = "cm")
```

---

## Python ではこう書く

=== "matplotlib"

    ```python
    import matplotlib.pyplot as plt
    fig, ax = plt.subplots(figsize=(8, 5))     # figsize はインチ
    ax.plot([1, 2, 3], [1, 4, 9])

    fig.savefig("fig.png", dpi=300, bbox_inches="tight")   # 余白を詰める
    fig.savefig("fig.pdf")                                  # ベクター
    plt.close(fig)                                          # メモリ解放
    ```

=== "plotnine"

    ```python
    from plotnine import ggplot, aes, geom_point
    p = ggplot(d, aes("dose", "resp")) + geom_point()
    p.save("fig.png", width=8, height=5, dpi=300, verbose=False)   # ggsave 相当
    ```

!!! tip "実務ではこれ"
    - **サイズは `figsize=(w, h)`（インチ）**で図を作り、`savefig(dpi=)` で解像度。R の `ggsave(width, height, dpi)` に対応。
    - **`bbox_inches="tight"`** で余白を自動で詰める（ラベル切れ防止）。
    - **ベクター（PDF/SVG）** は拡大しても綺麗。ラスタ（PNG）は `dpi` を上げる。提出図は用途で選択。
    - ループで大量保存するときは **`plt.close(fig)`** を忘れずに（メモリリーク防止）。
    - plotnine は `ggplot.save()`（引数が ggsave そっくり）。

---

## 対応早見表

| やりたいこと | R（ggsave） | Python |
|---|---|---|
| PNG 保存 | `ggsave("f.png", dpi=)` | `fig.savefig("f.png", dpi=)` / `p.save()` |
| PDF/SVG | `ggsave("f.pdf")` | `fig.savefig("f.pdf"/".svg")` |
| サイズ | `width=, height=, units=` | `figsize=(w,h)`（インチ） |
| 解像度 | `dpi=` | `dpi=` |
| 余白調整 | — | `bbox_inches="tight"` |
| 図を閉じる | 自動 | `plt.close(fig)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **サイズ指定の場所**：R は `ggsave` の引数、matplotlib は**図作成時の `figsize`**（インチ）。`savefig` では基本サイズを変えない（`dpi` で解像度のみ）。
    - **単位**：`ggsave(units="cm")` に当たるものは matplotlib にない。cm 指定なら自分で `/2.54` する。
    - **`dpi` の効き方**：ベクター（PDF/SVG）では図の寸法は `figsize`、`dpi` はラスタ要素にのみ影響。
    - **ラベル切れ**：`bbox_inches="tight"` か `tight_layout()` を付けないと軸ラベルが切れることがある。
    - **日本語文字化け**：`font.family` を日本語対応フォントに設定（matplotlib の rcParams）。

## 関連項目

- [080. 図の基本](topic-080.md)
- [078. PDF 出力](topic-078.md)
- [083. 複数図の配置](topic-083.md)
