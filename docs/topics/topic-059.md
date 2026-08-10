# 059. 二値の比率と信頼区間 — `binom.test` / `prop.test` → `scipy.stats`

!!! abstract "この項目の R→Python 対応"
    - **R**: `binom.test()`（正確・Clopper–Pearson）／`prop.test()`（Wilson 系、既定は連続補正あり）
    - **Python（推奨）**: **`scipy.stats.binomtest(k, n).proportion_ci(method=)`**（`"wilson"` / `"exact"`）
    - **要注意**: CI の**方法**（Wald / Wilson / Clopper–Pearson）と**連続補正**の有無で値が変わる。方法を明示

奏効率などの「割合とその 95%CI」。方法を揃えないと R と一致しません。

（例: 3/10 = 0.30 の Wilson CI）

---

## R ではこう書く

```r
# 正確法（Clopper–Pearson）
binom.test(3, 10)$conf.int

# Wilson 系（連続補正なしで解析的な Wilson に近い）
prop.test(3, 10, correct = FALSE)$conf.int
```

出力（`prop.test(3, 10, correct=FALSE)`）:

```text
p=0.300  CI=[0.108, 0.603]
```

!!! note "R の勘所"
    - `binom.test`：**正確法（Clopper–Pearson）**。小標本で保守的。
    - `prop.test`：既定は**連続補正あり**の Wilson。補正を外すなら `correct = FALSE`。
    - どの方法かで CI 幅が変わる。論文・SAP に合わせる。

---

## Python ではこう書く

=== "scipy"

    ```python
    from scipy import stats

    bt = stats.binomtest(3, 10)          # p 値・点推定
    print(bt.proportion_ci(method="wilson"))   # Wilson CI（correct なし）
    print(bt.proportion_ci(method="exact"))    # Clopper–Pearson（正確法）
    ```

    出力（Wilson）:

    ```text
    p=0.300  CI=[0.108, 0.603]
    ```

    → R の `prop.test(3, 10, correct=FALSE)` と一致。

=== "2 群の比率差"

    ```python
    from scipy.stats import norm
    # 2×2 の比率差・カイ二乗は chi2_contingency（→ [061]）
    # 差の CI や検定は statsmodels が便利:
    #   from statsmodels.stats.proportion import proportions_ztest, confint_proportions_2indep
    ```

!!! tip "実務ではこれ"
    - **1 群の割合＋CI** → `binomtest(k, n).proportion_ci(method=)`。
        - `"wilson"` = R `prop.test(correct=FALSE)`
        - `"exact"` = R `binom.test`（Clopper–Pearson）
    - **SAP で方法が指定**されているはず。Wald（`k/n ± 1.96·SE`）は 0/1 付近で破綻するので既定にしない。
    - **2 群の比率差・オッズ比**は `statsmodels.stats.proportion`（要インストール）や `scipy` の 2×2 検定（→ [061](../roadmap.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 正確法 CI | `binom.test(k, n)` | `binomtest(k,n).proportion_ci("exact")` |
| Wilson CI | `prop.test(k, n, correct=FALSE)` | `binomtest(k,n).proportion_ci("wilson")` |
| 連続補正あり Wilson | `prop.test(k, n)` | statsmodels `proportion_confint(method="wilson")` 等 |
| 2 群の比率差検定 | `prop.test(c(x1,x2), c(n1,n2))` | statsmodels `proportions_ztest` |
| Wald CI | 手計算 | statsmodels `proportion_confint(method="normal")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **方法を明示**：CI は Wald/Wilson/Clopper–Pearson で値が違う。**R の関数と scipy の `method=` の対応**（上表）を合わせないと一致しない。
    - **連続補正**：R `prop.test` は既定で連続補正あり。scipy の `"wilson"` は補正なし。合わせるなら R 側を `correct=FALSE`、または statsmodels の補正あり Wilson を使う。
    - **点推定**：`binomtest` の点推定は `k/n`。CI は非対称（Wilson/正確法）なので「点±幅」で書けない。
    - **2 群比較**は scipy だけだと手薄。`statsmodels.stats.proportion` を入れると R の `prop.test` 2 群版に近いことができる。

## 関連項目

- [060. t検定・Wilcoxon](topic-060.md)
- [061. カイ二乗・Fisher](../roadmap.md)
- [057. n (%) 整形のパターン](topic-057.md)
