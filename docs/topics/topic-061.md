# 061. カイ二乗・Fisher — `chisq.test` / `fisher.test` → `scipy.stats`

!!! abstract "この項目の R→Python 対応"
    - **R**: `chisq.test()`（2×2 は既定で **Yates 連続補正**）／`fisher.test()`（正確確率）
    - **Python（推奨）**: **`scipy.stats.chi2_contingency(table)`**（既定 `correction=True`）／**`fisher_exact(table)`**
    - **要注意**: Fisher の**オッズ比**は R（条件付き MLE）と scipy（標本オッズ比）で**値が違う**。p 値は一致

カテゴリ×群の関連。クロス表（→ [054](topic-054.md)）を検定に渡します。

（sex × arm の 2×2: F(2,3), M(3,2)）

---

## R ではこう書く

```r
tab <- table(dm3$sex, dm3$arm)

chisq.test(tab)     # 2×2 は既定で Yates 連続補正
fisher.test(tab)    # 正確確率検定
```

出力:

```text
chisq (Yates): X2=0.0000 df=1 p=1.0000
fisher: p=1.0000 OR=0.4831
```

!!! note "R の勘所"
    - `chisq.test` は **2×2 で既定 `correct = TRUE`**（Yates）。補正を外すなら `correct = FALSE`。
    - 期待度数が小さい（<5）と警告 → `fisher.test` を使う。
    - `fisher.test` のオッズ比は**条件付き最尤推定**（非心超幾何）。

---

## Python ではこう書く

=== "scipy"

    ```python
    import pandas as pd
    from scipy import stats

    tab = pd.crosstab(dm3["sex"], dm3["arm"])

    chi2, p, dof, exp = stats.chi2_contingency(tab)   # 既定 correction=True
    print(f"chi2 (Yates): X2={chi2:.4f} df={dof} p={p:.4f}")

    odds, pf = stats.fisher_exact(tab)
    print(f"fisher: p={pf:.4f} OR={odds:.4f}")
    ```

    出力:

    ```text
    chi2 (Yates): X2=0.0000 df=1 p=1.0000
    fisher: p=1.0000 OR=0.4444     ← OR が R と違う（下記）
    ```

!!! tip "実務ではこれ"
    - **χ² 検定** → `chi2_contingency(tab)`。R と同じく**2×2 で既定 Yates 補正あり**。補正を外すなら `correction=False`。
    - **Fisher** → `fisher_exact(tab)`。p 値は R と一致。
    - 期待度数の確認（`exp`）で χ² の適用可否を判断。小さければ Fisher。

---

## 対応早見表

| やりたいこと | R | Python（scipy） |
|---|---|---|
| χ²（Yates 補正） | `chisq.test(tab)` | `chi2_contingency(tab)` |
| χ²（補正なし） | `chisq.test(tab, correct=FALSE)` | `chi2_contingency(tab, correction=False)` |
| Fisher 正確 | `fisher.test(tab)` | `fisher_exact(tab)` |
| 期待度数 | `chisq.test(tab)$expected` | `chi2_contingency(tab)[3]` |
| 大きい表の Fisher | `fisher.test(tab)` | `fisher_exact` は 2×2 のみ（大表は要 R/別実装） |

## つまずきポイント

!!! danger "Fisher のオッズ比は R と scipy で違う"
    - R `fisher.test` の OR は**条件付き最尤推定**（例 0.4831）。scipy `fisher_exact` の OR は**標本オッズ比 ad/bc**（例 0.4444）。**p 値は一致するが OR は一致しない**。OR を報告するなら定義を明記し、必要なら R 側/条件付き推定に揃える。

!!! warning "その他"
    - **2×2 の Yates 補正**：R も scipy も既定で ON。補正の有無で χ² 値が変わる。SAP に合わせて `correction=`。
    - **`fisher_exact` は 2×2 限定**：scipy は r×c の Fisher を標準サポートしない。大きい表は R の `fisher.test` かモンテカルロ版を使う。
    - **入力**：`chi2_contingency` には**度数の表**を渡す（割合ではない）。`crosstab` の生カウントをそのまま。
    - **ゼロセル**：0 を含むと χ² が不安定。Fisher に切り替える。

## 関連項目

- [054. クロス集計表](topic-054.md)
- [059. 二値の比率と信頼区間](topic-059.md)
- [060. t検定・Wilcoxon](topic-060.md)
