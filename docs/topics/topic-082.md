# 082. 群別・ファセット — `facet_wrap` → subplots / plotnine

!!! abstract "この項目の R→Python 対応"
    - **R**: `ggplot2` の `facet_wrap(~g)` / `facet_grid(a ~ b)`（小分けパネル）
    - **Python（推奨）**: plotnine の **`facet_wrap("~g")`**（そのまま）／matplotlib は **`plt.subplots()`** で自前
    - **要注意**: matplotlib で facet を作るなら軸の共有（`sharex/sharey`）とループを自分で管理

群ごとに小さなパネルを並べる facet。plotnine ならそのまま、matplotlib なら subplots で組みます。

---

## R ではこう書く

```r
library(ggplot2)
ggplot(d, aes(dose, resp)) +
  geom_point() +
  facet_wrap(~ arm)              # arm ごとにパネル
# facet_grid(sex ~ arm)          # 行×列のグリッド
```

---

## Python ではこう書く

=== "plotnine（そのまま）"

    ```python
    from plotnine import ggplot, aes, geom_point, facet_wrap
    (ggplot(d, aes("dose", "resp")) + geom_point() + facet_wrap("~arm"))
    # facet_grid("sex ~ arm")
    ```

=== "matplotlib（自前 subplots）"

    ```python
    import matplotlib.pyplot as plt

    arms = d["arm"].unique()
    fig, axes = plt.subplots(1, len(arms), sharey=True, figsize=(8, 3))
    for ax, arm in zip(axes, arms):
        g = d[d["arm"] == arm]
        ax.scatter(g["dose"], g["resp"])
        ax.set_title(arm)
    fig.tight_layout()
    ```

=== "seaborn（近道）"

    ```python
    import seaborn as sns
    sns.relplot(data=d, x="dose", y="resp", col="arm", kind="scatter")  # col= が facet
    ```

!!! tip "実務ではこれ"
    - **ggplot 資産があるなら plotnine の `facet_wrap`** をそのまま。
    - **matplotlib で作るなら `subplots(nrow, ncol)`** で軸を並べ、群でループ。軸範囲を揃えるなら `sharex=/sharey=`。
    - **seaborn の `col=`/`row=`**（`relplot`/`catplot`/`displot`）は facet の近道。
    - パネル数が可変なら `subplots` の行列数を `math.ceil(n/ncol)` で計算。

---

## 対応早見表

| やりたいこと | ggplot2 | plotnine | matplotlib / seaborn |
|---|---|---|---|
| 1 変数 facet | `facet_wrap(~g)` | `facet_wrap("~g")` | `subplots` ループ / `col=g` |
| 行×列 grid | `facet_grid(a~b)` | `facet_grid("a~b")` | `subplots(nrow,ncol)` / `col=,row=` |
| 軸を共有 | 既定で共有 | 既定で共有 | `sharex=/sharey=True` |
| 自由軸 | `scales="free"` | `scales="free"` | 各 ax を個別設定 |
| パネル見出し | strip ラベル | strip ラベル | `ax.set_title()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **matplotlib は手作業**：facet が言語機能でない。行列数・ループ・軸共有・レイアウトを自分で書く。plotnine/seaborn なら宣言的。
    - **軸の共有**：ggplot の facet は既定で軸を共有。matplotlib は `sharex/sharey` を明示しないとバラバラ。
    - **空パネル**：`subplots` で作った余りの軸は空で残る。`ax.set_visible(False)` で消す。
    - **凡例の重複**：ループで各 ax に凡例を付けると重複。図全体で1つにまとめる（`fig.legend()`）。

## 関連項目

- [080. 図の基本](topic-080.md)
- [083. 複数図の配置](topic-083.md)
- [030. グループ内変換](topic-030.md)
