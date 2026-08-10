# 060. t検定・Wilcoxon — `t.test` / `wilcox.test` → `scipy.stats`

!!! abstract "この項目の R→Python 対応"
    - **R**: `t.test()`（既定は **Welch**、等分散を仮定しない）／`wilcox.test()`（Mann–Whitney / 符号順位）
    - **Python（推奨）**: **`scipy.stats.ttest_ind(a, b, equal_var=False)`** ／ **`mannwhitneyu(a, b)`**
    - **要注意**: **scipy の `ttest_ind` 既定は等分散（Student）**。R の既定 Welch に合わせるには **`equal_var=False`** が必須

2 群の連続変数比較。既定の仮定がずれるので、そこだけ注意すれば結果は一致します。

（arm A: 45,52,60,48,55 / arm B: 38,61,50,42,44）

---

## R ではこう書く

```r
A <- dm3$age[dm3$arm == "A"]
B <- dm3$age[dm3$arm == "B"]

t.test(A, B)                 # 既定は Welch（等分散を仮定しない）
t.test(A, B, var.equal = TRUE)  # Student（等分散仮定）
wilcox.test(A, B)            # Mann–Whitney U（対応なし）
# wilcox.test(A, B, paired = TRUE)  # 符号順位（対応あり）
```

出力:

```text
Welch t: t=1.045 df=6.91 p=0.331
Wilcoxon: W=18.0 p=0.310
```

!!! note "R の勘所"
    - `t.test` の**既定は Welch**（`var.equal = FALSE`）。df が非整数になる。
    - `wilcox.test` は対応なしで Mann–Whitney、`paired=TRUE` で符号順位。
    - 対応あり t 検定は `t.test(x, y, paired = TRUE)`。

---

## Python ではこう書く

=== "scipy"

    ```python
    from scipy import stats

    A = dm3.loc[dm3["arm"]=="A", "age"]
    B = dm3.loc[dm3["arm"]=="B", "age"]

    t = stats.ttest_ind(A, B, equal_var=False)   # Welch（R の既定に合わせる）
    print(f"Welch t: t={t.statistic:.3f} df={t.df:.2f} p={t.pvalue:.3f}")

    u = stats.mannwhitneyu(A, B, alternative="two-sided")
    print(f"MWU: U={u.statistic:.1f} p={u.pvalue:.3f}")

    # 対応あり
    # stats.ttest_rel(x, y)          # paired t
    # stats.wilcoxon(x, y)           # 符号順位
    ```

    出力:

    ```text
    Welch t: t=1.045 df=6.91 p=0.331
    MWU: U=18.0 p=0.310
    ```

    → R の `t.test`（Welch）・`wilcox.test` と一致（W = U = 18.0）。

!!! tip "実務ではこれ"
    - **t 検定は `equal_var=False` を明示**して R の Welch と揃える。等分散版が要るなら `equal_var=True`。
    - **Mann–Whitney の統計量**：R の `W` と scipy の `U` は同じ（第1標本基準）。p 値は小標本で正確法、大標本で正規近似＋連続補正（`method=`, `use_continuity=`）。
    - 対応あり → `ttest_rel` / `wilcoxon`。
    - 検定は探索・補助。主要評価は SAP のモデル（→ [062](../roadmap.md)）に従う。

---

## 対応早見表

| 検定 | R | Python（scipy） |
|---|---|---|
| Welch t（既定） | `t.test(a, b)` | `ttest_ind(a, b, equal_var=False)` |
| Student t | `t.test(a, b, var.equal=TRUE)` | `ttest_ind(a, b)`（既定） |
| 対応あり t | `t.test(x, y, paired=TRUE)` | `ttest_rel(x, y)` |
| Mann–Whitney | `wilcox.test(a, b)` | `mannwhitneyu(a, b)` |
| 符号順位 | `wilcox.test(x, y, paired=TRUE)` | `wilcoxon(x, y)` |
| 片側 | `alternative="greater"` | `alternative="greater"` |

## つまずきポイント

!!! warning "R と Python の差"
    - **既定の仮定が逆**：R `t.test` は Welch、scipy `ttest_ind` は等分散（Student）。**`equal_var=False` を付けないと df・p 値が R と食い違う**。
    - **MWU の p 値**：既定の正確/近似の切替や連続補正が R と微妙に違うことがある。小標本では `method="exact"` を明示して揃える。
    - **片側の向き**：`alternative` の意味（どちらが大きい）を R と scipy で確認。
    - **入力の欠損**：scipy は NaN があると NaN を返すことがある。事前に `dropna()`。R は `na.action` 依存。

## 関連項目

- [059. 二値の比率と信頼区間](topic-059.md)
- [061. カイ二乗・Fisher](../roadmap.md)
- [062. 分散分析・線形回帰](../roadmap.md)
