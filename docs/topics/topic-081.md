# 081. ggplot 文法の対応 — `aes` / `geom` → plotnine

!!! abstract "この項目の R→Python 対応"
    - **R**: `ggplot2` の `aes()` + `geom_*()` + `scale_*()` + `facet_*()` + `theme()`
    - **Python（推奨）**: **plotnine** が**同名 API**。列名は文字列で渡す点だけ違う
    - **要注意**: `+` の重ね方・レイヤ構造は同じ。非引用の列名（ggplot2）が plotnine では文字列

ggplot2 の各要素が plotnine にどう対応するかの逐語対訳。

---

## R ではこう書く

```r
library(ggplot2)
ggplot(d, aes(x = dose, y = resp, color = arm)) +
  geom_point(size = 2) +
  geom_smooth(method = "lm", se = FALSE) +
  scale_y_continuous(limits = c(0, 10)) +
  facet_wrap(~ arm) +
  labs(x = "Dose", y = "Response") +
  theme_bw()
```

---

## Python ではこう書く

=== "plotnine"

    ```python
    from plotnine import (ggplot, aes, geom_point, geom_smooth,
                          scale_y_continuous, facet_wrap, labs, theme_bw)

    (ggplot(d, aes("dose", "resp", color="arm"))
       + geom_point(size=2)
       + geom_smooth(method="lm", se=False)
       + scale_y_continuous(limits=(0, 10))
       + facet_wrap("~arm")
       + labs(x="Dose", y="Response")
       + theme_bw())
    ```

    ほぼ**そのまま**。違いは (1) 列名を**文字列**で渡す、(2) `facet_wrap("~arm")` の式も文字列、(3) 一部引数が Python 記法（`True/False`、タプル）。

!!! tip "実務ではこれ"
    - **逐語で移せる**：`geom_*`/`scale_*`/`facet_*`/`theme_*` は同名。まず plotnine でそのまま書き、動かない所だけ直す。
    - **列名は文字列**（`aes("x")`）。ggplot2 の非引用（`aes(x)`）と違う唯一の癖。
    - **`+` は Python の演算子オーバーロード**：長い連結は全体を丸括弧で囲む（[002](topic-002.md) と同じ）。
    - どうしても plotnine で表現しにくい所は、`draw()` で matplotlib の Figure を取り出して微調整。

---

## 対応早見表

| ggplot2 | plotnine | 備考 |
|---|---|---|
| `aes(x, y, color=)` | `aes("x", "y", color="...")` | 列名は文字列 |
| `geom_point/line/col/boxplot` | 同名 | — |
| `geom_smooth(method=)` | `geom_smooth(method=)` | `se=False` |
| `scale_*_continuous/manual` | 同名 | 引数はタプル/リスト |
| `facet_wrap(~g)` / `facet_grid` | `facet_wrap("~g")` | 式も文字列 |
| `coord_flip()` | `coord_flip()` | — |
| `theme_bw()` / `theme()` | 同名 | — |
| `labs()` / `ggtitle()` | `labs()` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **列名は文字列**：`aes(dose)` ではなく `aes("dose")`。非引用評価は Python にないため。
    - **論理値・タプル**：`se = FALSE` → `se=False`、`limits = c(0,10)` → `limits=(0, 10)`。
    - **`+` の改行**：Python は行継続に丸括弧が要る。全体を `()` で囲む。
    - **機能差**：plotnine は ggplot2 の大半を実装するが、最新拡張や一部 `geom`/`stat` は未対応のことがある。無い場合は matplotlib/seaborn で代替。
    - **日本語フォント**：文字化けは matplotlib の `font.family` 設定で対応（plotnine の下地は matplotlib）。

## 関連項目

- [080. 図の基本](topic-080.md)
- [082. 群別・ファセット](topic-082.md)
- [084. 図の保存](topic-084.md)
