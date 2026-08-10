# 090. ベクトル化の考え方 — `for` を避ける

!!! abstract "この項目の R→Python 対応"
    - **R**: ループより**ベクトル演算**・`apply` 系。`for` で1要素ずつは遅い
    - **Python（推奨）**: **NumPy / pandas のベクトル演算**。`for`/`apply(axis=1)`/`iterrows` は最後の手段
    - **要注意**: pandas の `iterrows` / `apply(axis=1)` は特に遅い。列演算・`np.where`・`groupby` に置き換える

R でも Python でも「1 行ずつ回さない」が高速化の第一歩。発想は共通です。

---

## 遅い書き方 → 速い書き方

=== "pandas（遅い）"

    ```python
    # 1 行ずつ回す（遅い・非推奨）
    out = []
    for _, r in df.iterrows():
        out.append(r["a"] * 2 + r["b"])
    df["y"] = out
    ```

=== "pandas（速い・ベクトル化）"

    ```python
    # 列演算でまとめて（速い）
    df["y"] = df["a"] * 2 + df["b"]

    # 条件つきは np.where / np.select
    import numpy as np
    df["grp"] = np.where(df["age"] >= 50, "old", "young")
    ```

R でも同じ発想:

```r
# 遅い
for (i in seq_len(nrow(df))) df$y[i] <- df$a[i]*2 + df$b[i]
# 速い（ベクトル化）
df$y <- df$a*2 + df$b
```

!!! tip "実務ではこれ"
    - **列同士の算術・比較・文字列操作はベクトル化**：`df["a"] + df["b"]`、`df["x"].str.upper()`、`np.where(...)`。
    - **群単位の集計・変換は `groupby`**（`agg`/`transform`）。ループで群を回さない（[010](topic-010.md), [030](topic-030.md)）。
    - **どうしても行単位**なら `apply(axis=1)`、それでも遅ければ NumPy 配列に落として計算。
    - **早すぎる最適化は不要**だが、数万行以上で `iterrows` を使ったら黄信号。

---

## 何をベクトル化に置き換えるか

| ループでやりがち | ベクトル化 |
|---|---|
| 各行の算術 | 列演算 `df["a"] + df["b"]` |
| 各行の条件分岐 | `np.where` / `np.select`（[026](topic-026.md)） |
| 群ごとの集計 | `groupby().agg()`（[010](topic-010.md)） |
| 群ごとの前後差・累積 | `groupby().shift()/cumsum()`（[031](topic-031.md)） |
| 行方向の平均・合計 | `df[cols].mean(axis=1)`（[033](topic-033.md)） |
| 対応表の適用 | `map(dict)`（[047](topic-047.md)） |

## つまずきポイント

!!! warning "R と Python の差"
    - **`iterrows` は特に遅い**：各行を Series 化するので重い。`itertuples`（名前付きタプル）はやや速いが、それでも列演算に勝てない。
    - **`apply(axis=1)` も遅い**：見た目はベクトル化風だが内部は行ループ。列演算に落とせないか考える。
    - **文字列・日付もベクトル化**：`.str` アクセサ（[011](topic-011.md)〜）、`.dt` アクセサ（[092](topic-092.md)）を使う。ループで1件ずつ処理しない。
    - **ベクトル化できない処理**（外部 API 呼び出し等）は素直にループ/`apply`。無理にベクトル化しない。

## 関連項目

- [033. 行ごと処理・rowwise](topic-033.md)
- [087. 反復：apply 系と map](topic-087.md)
- [001. R と Python の考え方の違い](topic-001.md)
