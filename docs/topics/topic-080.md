# 080. 図の基本 — ggplot2 → matplotlib / plotnine

!!! abstract "この項目の R→Python 対応"
    - **R**: `ggplot2`（文法ベースの作図）
    - **Python（推奨）**: **plotnine**（ggplot2 の移植・ほぼ同じ文法）／**matplotlib**（土台・自由度最大）／seaborn（統計図の近道）
    - **要注意**: ggplot2 の発想をそのまま使うなら plotnine、Python 標準に寄せるなら matplotlib

作図の選択肢の地図。ggplot2 ユーザには plotnine が最短、細かい制御や既存資産には matplotlib。

---

## R ではこう書く

```r
library(ggplot2)
ggplot(d, aes(x = dose, y = resp, color = arm)) +
  geom_point() +
  geom_line() +
  labs(title = "Response by dose")
```

---

## Python ではこう書く

=== "plotnine（ggplot 文法そのまま）"

    ```python
    # pip install plotnine
    from plotnine import ggplot, aes, geom_point, geom_line, labs

    (ggplot(d, aes("dose", "resp", color="arm"))
       + geom_point()
       + geom_line()
       + labs(title="Response by dose"))
    ```

    `+` で重ねる文法・`aes`・`geom_*` が ggplot2 と同じ。R の作図資産をほぼ書き換えなしで移せます。

=== "matplotlib（土台）"

    ```python
    import matplotlib.pyplot as plt

    fig, ax = plt.subplots()
    for arm, g in d.groupby("arm"):
        ax.plot(g["dose"], g["resp"], marker="o", label=arm)
    ax.set(title="Response by dose", xlabel="dose", ylabel="resp")
    ax.legend()
    ```

=== "seaborn（統計図の近道）"

    ```python
    import seaborn as sns
    sns.lineplot(data=d, x="dose", y="resp", hue="arm", marker="o")
    ```

!!! tip "実務ではこれ"
    - **ggplot2 を使ってきたなら plotnine**。学習コストが最小で、`aes`/`geom`/`facet`/`theme` がそのまま。
    - **細かい制御・既存の matplotlib 資産・他ライブラリ連携**なら matplotlib。lifelines の KM 曲線なども matplotlib の Axes に乗る。
    - **統計的な定番図**（分布・回帰・カテゴリ別）は seaborn が短く書ける。
    - 3 つは排他でなく、plotnine/seaborn の下地は matplotlib。細部は matplotlib で微調整できる。

---

## 対応早見表

| ggplot2（R） | plotnine（Python） | matplotlib |
|---|---|---|
| `ggplot(d, aes())` | `ggplot(d, aes())` | `fig, ax = plt.subplots()` |
| `geom_point()` | `geom_point()` | `ax.scatter()` |
| `geom_line()` | `geom_line()` | `ax.plot()` |
| `geom_col()`/`geom_bar()` | 同名 | `ax.bar()` |
| `facet_wrap()` | `facet_wrap()` | subplots（→ [082](topic-082.md)） |
| `labs()`/`theme()` | `labs()`/`theme()` | `ax.set()` / rcParams |
| `ggsave()` | `ggplot.save()` | `fig.savefig()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **状態を持つ matplotlib**：`plt.` は「現在の図」に対して働く（暗黙の状態）。**明示的に `fig, ax` を作る**（オブジェクト指向 API）と混乱しない。
    - **描画のタイミング**：スクリプトでは `plt.show()`（表示）や `savefig`（保存）が要る。Jupyter は自動表示。
    - **plotnine の依存**：内部は matplotlib。日本語フォントなどは matplotlib 側の設定を触る。
    - **列名の渡し方**：plotnine は `aes("x")` と**文字列**で列名を渡す（ggplot2 は非引用）。

## 関連項目

- [081. ggplot 文法の対応](topic-081.md)
- [083. 複数図の配置](topic-083.md)
- [084. 図の保存](topic-084.md)
