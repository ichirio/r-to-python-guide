# 083. 複数図の配置 — patchwork → subplots / gridspec

!!! abstract "この項目の R→Python 対応"
    - **R**: `patchwork`（`p1 + p2`, `p1 / p2`）／`cowplot` / `gridExtra`
    - **Python（推奨）**: matplotlib **`plt.subplots()`** / **`GridSpec`**（自由配置）／`subplot_mosaic`（レイアウトを文字で）
    - **要注意**: patchwork の演算子的な合成に当たる仕組みはないが、`subplot_mosaic` が近い書き味

facet（同じ図の小分け、[082](topic-082.md)）ではなく、**別々の図を1枚に配置**する話。

---

## R ではこう書く

```r
library(patchwork)
(p1 + p2) / p3          # 上段に p1,p2 横並び、下段に p3
p1 | p2                 # 横並び
p1 / p2                 # 縦積み
```

---

## Python ではこう書く

=== "subplots（格子）"

    ```python
    import matplotlib.pyplot as plt
    fig, axes = plt.subplots(2, 2, figsize=(8, 6))
    axes[0, 0].plot(...)         # 左上
    axes[0, 1].bar(...)          # 右上
    # ...
    fig.tight_layout()
    ```

=== "subplot_mosaic（レイアウトを文字で・patchwork 風）"

    ```python
    fig, ax = plt.subplot_mosaic(
        """
        AB
        CC
        """)                     # A,B 上段横並び、C 下段いっぱい
    ax["A"].plot(...); ax["B"].bar(...); ax["C"].scatter(...)
    ```

=== "GridSpec（比率を細かく）"

    ```python
    from matplotlib.gridspec import GridSpec
    fig = plt.figure(figsize=(8, 6))
    gs = GridSpec(2, 2, height_ratios=[2, 1])
    ax1 = fig.add_subplot(gs[0, :])   # 上段いっぱい
    ax2 = fig.add_subplot(gs[1, 0]); ax3 = fig.add_subplot(gs[1, 1])
    ```

!!! tip "実務ではこれ"
    - **格子で足りる** → `plt.subplots(nrow, ncol)`。
    - **patchwork のような直感的レイアウト** → **`subplot_mosaic`**（文字列でレイアウトを描く）。名前でパネルを参照できて読みやすい。
    - **比率や結合が複雑** → `GridSpec`（行高・列幅の比、セル結合）。
    - plotnine の図を混ぜるなら `p.draw()` で Figure/Axes を取り出して matplotlib 側に組み込む。

---

## 対応早見表

| やりたいこと | patchwork（R） | matplotlib |
|---|---|---|
| 横並び | `p1 \| p2` | `subplots(1, 2)` / mosaic `"AB"` |
| 縦積み | `p1 / p2` | `subplots(2, 1)` / mosaic `"A\nB"` |
| 混在レイアウト | `(p1 + p2) / p3` | `subplot_mosaic("AB\nCC")` |
| セル結合 | — | `GridSpec` のスライス |
| 余白・タイトル | `plot_annotation()` | `fig.suptitle()` / `tight_layout()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **演算子合成がない**：patchwork の `+`/`/` のような図同士の演算はない。`subplot_mosaic` が最も近い代替。
    - **軸オブジェクトを渡す設計**：matplotlib は「どの ax に描くか」を明示（`ax.plot(...)`）。関数化するときは `ax` を引数に取る作りにすると組み立てやすい。
    - **レイアウト崩れ**：重なりは `tight_layout()` / `constrained_layout=True` で調整。
    - **plotnine との混在**：plotnine の図はそのままでは並べにくい。`draw()` で Figure を取り出すか、matplotlib に寄せる。

## 関連項目

- [082. 群別・ファセット](topic-082.md)
- [084. 図の保存](topic-084.md)
- [085. KM 曲線・フォレストプロット](topic-085.md)
