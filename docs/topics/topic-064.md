# 064. 生存時間の要約 — `survival::survfit` → `lifelines`

!!! abstract "この項目の R→Python 対応"
    - **R**: `survival` パッケージ。`Surv(time, event)` → `survfit()`（Kaplan–Meier）／`survdiff()`（log-rank）／`coxph()`
    - **Python（推奨）**: **`lifelines`**。`KaplanMeierFitter` / `logrank_test` / `CoxPHFitter`
    - **要注意**: event の符号（1=イベント, 0=打ち切り）を揃える。中央生存時間・時点生存率は R と一致

（time/event の10例。中央生存 10、S(7)=0.686, S(10)=0.411）

---

## R ではこう書く

```r
library(survival)
s <- data.frame(time = c(5,6,6,7,8,9,10,12,14,15),
                event = c(1,0,1,1,0,1,1,0,1,1))

f <- survfit(Surv(time, event) ~ 1, data = s)
summary(f)$table["median"]          # 中央生存時間
summary(f, times = c(7, 10))$surv   # 指定時点の生存率
# survdiff(Surv(time, event) ~ group, data = s)   # log-rank
# coxph(Surv(time, event) ~ group, data = s)      # Cox
```

出力:

```text
median: 10
[1] 0.6857143 0.4114286
```

!!! note "R の勘所"
    - `Surv(time, event)`：event は 1=イベント / 0=打ち切り（`TRUE`/`FALSE` も可）。
    - `survfit(~ group)` で群別 KM、`survdiff` で log-rank、`coxph` で比例ハザード。
    - 中央生存は `summary(f)$table["median"]`、時点生存率は `summary(f, times=)`。

---

## Python ではこう書く

=== "lifelines"

    ```python
    # pip install lifelines
    import pandas as pd
    from lifelines import KaplanMeierFitter

    surv = pd.DataFrame({"time":  [5,6,6,7,8,9,10,12,14,15],
                         "event": [1,0,1,1,0,1,1,0,1,1]})

    kmf = KaplanMeierFitter().fit(surv["time"], surv["event"])
    print(kmf.median_survival_time_)                       # 中央生存時間
    print([round(float(kmf.predict(t)), 4) for t in (7, 10)])  # 時点生存率
    ```

    出力:

    ```text
    10.0
    [0.6857, 0.4114]
    ```

    → R の `survfit` と一致。

=== "群比較・Cox"

    ```python
    from lifelines.statistics import logrank_test
    from lifelines import CoxPHFitter

    # log-rank（2 群）
    # logrank_test(t1, t2, event_observed_A=e1, event_observed_B=e2)

    # Cox 比例ハザード
    # CoxPHFitter().fit(df, duration_col="time", event_col="event")
    ```

!!! tip "実務ではこれ"
    - **KM 推定・中央生存・時点生存率** → `KaplanMeierFitter`。R の `survfit` と数値が一致。
    - **log-rank** → `lifelines.statistics.logrank_test`。**Cox 回帰** → `CoxPHFitter`（`print_summary()` が R の `summary(coxph)` 相当）。
    - **KM 曲線の作図**は `kmf.plot_survival_function()`（→ [085](../roadmap.md)）。
    - event の符号（1=イベント）と打ち切りの定義を R と厳密に合わせる。

---

## 対応早見表

| やりたいこと | R（survival） | Python（lifelines） |
|---|---|---|
| KM 推定 | `survfit(Surv(t,e) ~ 1)` | `KaplanMeierFitter().fit(t, e)` |
| 中央生存時間 | `summary(f)$table["median"]` | `kmf.median_survival_time_` |
| 時点生存率 | `summary(f, times=)` | `kmf.predict(times)` |
| 群別 KM | `survfit(~ group)` | 群ごとに `fit` |
| log-rank | `survdiff(~ group)` | `logrank_test(...)` |
| Cox 回帰 | `coxph(Surv(t,e) ~ x)` | `CoxPHFitter().fit(df, ...)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **インストール**：lifelines は別途 `pip install lifelines`（依存も多め）。
    - **event の符号**：1=イベント/0=打ち切りを揃える。逆に入れると生存曲線が反転する。
    - **中央生存が未達**：イベントが半数に満たないと中央値は `inf`（lifelines）/`NA`（R）。到達しない旨を表に明記。
    - **信頼区間の方法**：KM の CI は log-log 変換など方法差がある。R と厳密比較するなら `ci_labels`/変換法を合わせる。
    - **タイの扱い**（Cox）：Efron（既定）/Breslow で係数が微妙に違う。R `coxph` の既定も Efron。

## 関連項目

- [062. 分散分析・線形回帰](topic-062.md)
- [085. KM 曲線・フォレストプロット](../roadmap.md)
