# 085. KM 曲線・フォレストプロット

!!! abstract "この項目の R→Python 対応"
    - **R**: KM は `survminer::ggsurvplot()`、フォレストは `forestplot` / `ggforestplot` / `meta`
    - **Python（推奨）**: KM は **lifelines**（`plot_survival_function` / `add_at_risk_counts`）、フォレストは **matplotlib で自作**（`errorbar` + 対数軸）
    - **要注意**: リスク集合（at risk）表付き KM は lifelines のヘルパで。フォレストは定番が薄く自作が基本

臨床で頻出の2図。KM は lifelines が強く、フォレストは matplotlib での自作が実務的です。

---

## R ではこう書く

```r
library(survminer)
fit <- survfit(Surv(time, event) ~ arm, data = s)
ggsurvplot(fit, risk.table = TRUE, conf.int = TRUE)   # KM＋リスク集合表

# フォレスト
library(forestplot)
forestplot(labeltext, mean = est, lower = lo, upper = hi)
```

---

## Python ではこう書く

=== "KM 曲線（lifelines）"

    ```python
    import matplotlib.pyplot as plt
    from lifelines import KaplanMeierFitter
    from lifelines.plotting import add_at_risk_counts

    fig, ax = plt.subplots()
    fitters = []
    for arm, g in surv.groupby("arm"):
        kmf = KaplanMeierFitter(label=arm).fit(g["time"], g["event"])
        kmf.plot_survival_function(ax=ax, ci_show=True)
        fitters.append(kmf)

    add_at_risk_counts(*fitters, ax=ax)     # リスク集合表（risk.table 相当）
    ax.set(xlabel="Time", ylabel="Survival probability")
    ```

=== "フォレストプロット（matplotlib 自作）"

    ```python
    import numpy as np, matplotlib.pyplot as plt

    labels = ["Overall", "Subgroup A", "Subgroup B"]
    est = np.array([0.85, 0.78, 0.92]); lo = np.array([0.70,0.60,0.75]); hi = np.array([1.03,1.02,1.13])
    y = np.arange(len(labels))[::-1]

    fig, ax = plt.subplots()
    ax.errorbar(est, y, xerr=[est - lo, hi - est], fmt="o", capsize=3)  # 点推定＋CI
    ax.axvline(1.0, ls="--", color="gray")     # 無効ライン（HR/OR=1）
    ax.set_yticks(y); ax.set_yticklabels(labels)
    ax.set_xscale("log")                         # 比の指標は対数軸
    ax.set_xlabel("Hazard ratio (95% CI)")
    ```

!!! tip "実務ではこれ"
    - **KM 曲線** → lifelines の `plot_survival_function` を群でループ。**`add_at_risk_counts`** で R の `risk.table` 相当のリスク集合表が付く（[064](topic-064.md)）。
    - **フォレスト** → 定番ライブラリが薄いので **matplotlib の `errorbar` で自作**。HR/OR は**対数軸**（`set_xscale("log")`）、無効線は `axvline(1)`。
    - ラベル・数値（推定値と CI の表記）は左右にテキストで添える。テンプレ関数を1つ作ると使い回せる。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| KM 曲線 | `ggsurvplot(fit)` | lifelines `plot_survival_function` |
| リスク集合表 | `risk.table=TRUE` | `add_at_risk_counts()` |
| 信頼区間帯 | `conf.int=TRUE` | `ci_show=True` |
| log-rank p 値 | `pval=TRUE` | `logrank_test()` を別途表示 |
| フォレスト | `forestplot` / `ggforestplot` | matplotlib `errorbar` 自作 |
| 無効ライン | 縦線を追加 | `ax.axvline(1)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **フォレストの定番不在**：R の `forestplot` 級はない。matplotlib で自作が基本。テンプレ化して社内共有すると効率的。
    - **対数軸**：HR/OR/RR は比なので**対数軸**（`set_xscale("log")`）。線形軸だと CI の見え方が歪む。
    - **at risk 表の整列**：`add_at_risk_counts` は時間軸と揃う。目盛り（`xticks`）を明示して R と揃える。
    - **群の色・順序**：群の順序・色は Categorical と palette を固定して R の見た目に寄せる。
    - **CI の方法**：KM の CI 変換法（log-log など）で R と細部が変わる（[064](topic-064.md)）。

## 関連項目

- [064. 生存時間の要約](topic-064.md)
- [080. 図の基本](topic-080.md)
- [083. 複数図の配置](topic-083.md)
