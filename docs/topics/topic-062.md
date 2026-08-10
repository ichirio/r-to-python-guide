# 062. 分散分析・線形回帰 — `lm` / `aov` → `statsmodels`

!!! abstract "この項目の R→Python 対応"
    - **R**: `lm(y ~ x)`（回帰）／`aov()`（分散分析）／`summary()` で係数表
    - **Python（推奨）**: **`statsmodels.formula.api.ols("y ~ x", data).fit()`**（R と同じ formula 構文）
    - **要注意**: statsmodels は別途インストールが要る。因子の**基準水準**の取り方は R と揃う（アルファベット順）

R の `y ~ x` 記法がほぼそのまま使える statsmodels が、移行の最短路です。

（`age ~ arm`。arm A が基準、係数 = B − A）

---

## R ではこう書く

```r
fit <- lm(age ~ arm, data = dm3)
coef(fit)
summary(fit)$coefficients

# 分散分析
anova(fit)          # または aov(age ~ arm, data = dm3)
```

出力:

```text
(Intercept)        armB
         52          -5
            Estimate Std. Error t value Pr(>|t|)
(Intercept)       52     3.3838 15.3674   0.0000
armB              -5     4.7854 -1.0448   0.3266
```

!!! note "R の勘所"
    - `lm(y ~ x1 + x2 + x1:x2)`：`+` で主効果、`:` で交互作用、`*` で両方。
    - 因子は**アルファベット順で最初が基準**（ここは arm A）。`relevel()` で変更。
    - `summary()` に係数・SE・t・p、`anova()` に分散分析表。

---

## Python ではこう書く

=== "statsmodels"

    ```python
    # pip install statsmodels
    import statsmodels.formula.api as smf

    fit = smf.ols("age ~ arm", data=dm3).fit()
    print(fit.params)                       # Intercept=52, arm[T.B]=-5
    print(fit.summary2().tables[1])         # 係数・SE・t・p
    # print(fit.summary())                  # R 風の全体サマリ

    # 分散分析表
    import statsmodels.api as sm
    print(sm.stats.anova_lm(fit))
    ```

    出力:

    ```text
    Intercept    52.0
    arm[T.B]     -5.0
               Coef.  Std.Err.        t   P>|t|
    Intercept   52.0    3.3838  15.3674  0.0000
    arm[T.B]    -5.0    4.7854  -1.0448  0.3266
    ```

    → R の `lm` と係数・SE・t・p がすべて一致。

!!! tip "実務ではこれ"
    - **formula 版 `smf.ols("y ~ x", data)`** を使えば R の `lm` とほぼ同じ書き味・同じ基準水準。
    - 交互作用 `x1*x2`、多項式 `I(x**2)`、カテゴリ明示 `C(arm)` も R 同様。
    - ロジスティック回帰は `smf.logit(...)`、混合効果は `smf.mixedlm(...)`。GLM は `smf.glm(..., family=)`。
    - 予測は `fit.predict(newdata)`、信頼区間は `fit.conf_int()`。

---

## 対応早見表

| やりたいこと | R | Python（statsmodels） |
|---|---|---|
| 線形回帰 | `lm(y ~ x, data)` | `smf.ols("y ~ x", data).fit()` |
| 係数表 | `summary(fit)$coefficients` | `fit.summary2().tables[1]` |
| 分散分析表 | `anova(fit)` / `aov()` | `sm.stats.anova_lm(fit)` |
| ロジスティック | `glm(y ~ x, family=binomial)` | `smf.logit("y ~ x", data).fit()` |
| 交互作用 | `y ~ a*b` | `"y ~ a*b"` |
| 予測 | `predict(fit, newdata)` | `fit.predict(newdata)` |
| 信頼区間 | `confint(fit)` | `fit.conf_int()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **インストール**：statsmodels は標準ライブラリではない（`pip install statsmodels`）。scipy だけでは回帰の係数表は出ない。
    - **`ols` vs `OLS`**：`smf.ols`（小文字、formula）は R 風。`sm.OLS`（大文字、行列）は自分で design matrix と切片列を作る必要がある。formula 版が楽。
    - **基準水準**：statsmodels も既定はアルファベット順で最初が基準（`arm[T.B]` = B の効果）。順序を変えるなら `C(arm, Treatment("B"))`。
    - **Type I/II/III 平方和**：`anova_lm` の既定は Type I（逐次）。R の `anova` も Type I。SAS の Type III と揃えるなら `typ=3` と適切なコントラスト。
    - **欠損**：statsmodels は既定で欠損行を除外（`missing="drop"`）。R も `na.action` 既定で除外。

## 関連項目

- [060. t検定・Wilcoxon](topic-060.md)
- [063. 相関](topic-063.md)
- [004. パッケージ管理と import](topic-004.md)
