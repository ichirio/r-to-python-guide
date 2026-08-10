# 099. 乱数と再現性 — `set.seed` → `np.random.default_rng`

!!! abstract "この項目の R→Python 対応"
    - **R**: `set.seed(42)` → `runif` / `rnorm` / `sample`
    - **Python（推奨）**: **`rng = np.random.default_rng(42)`** → `rng.random` / `rng.normal` / `rng.choice`
    - **要注意**: **同じシードでも R と Python は同じ乱数列にならない**（RNG が別）。再現性は各言語内で担保する

ブートストラップ・並べ替え・シミュレーションの再現性。**言語をまたぐ一致は期待できない**点が最重要です。

---

## R ではこう書く

```r
set.seed(42)
round(runif(3), 4)          # 一様乱数
rnorm(3)                    # 正規乱数
sample(1:10, 3)             # 非復元抽出
```

出力:

```text
[1] 0.9148 0.9371 0.2861
```

!!! note "R の勘所"
    - `set.seed()` はグローバルな乱数状態を固定。以後の乱数生成が再現する。
    - `sample()` は既定で非復元。`replace = TRUE` で復元。
    - スクリプトの冒頭で1回 `set.seed` すれば全体が再現。

---

## Python ではこう書く

=== "NumPy（新 API・推奨）"

    ```python
    import numpy as np
    rng = np.random.default_rng(42)     # Generator を作る（状態を持つ）

    np.round(rng.random(3), 4)          # 一様乱数
    rng.normal(size=3)                  # 正規乱数
    rng.choice(np.arange(1, 11), 3, replace=False)   # 非復元抽出
    ```

    出力:

    ```text
    [0.774, 0.4389, 0.8586]     ← R の set.seed(42) とは一致しない
    ```

=== "旧 API（既存コード）"

    ```python
    np.random.seed(42)          # グローバル状態（旧スタイル）
    np.random.rand(3)
    ```

!!! tip "実務ではこれ"
    - **新 API `default_rng(seed)` を使う**：`rng` オブジェクトに状態が閉じるので、関数間で明示的に渡せて再現性が管理しやすい。グローバル汚染がない。
    - **`rng` を引数で渡す**設計に:
      ```python
      def bootstrap(data, n=1000, rng=None):
          rng = rng or np.random.default_rng()
          ...
      ```
    - pandas の `sample` にも `random_state=rng`（または整数シード）を渡せる。
    - **言語間一致は諦める**：R と同じ数列は出ない。検証は「同一言語内で再現するか」で行う（[095](topic-095.md)）。

---

## 対応早見表

| やりたいこと | R | Python（NumPy 新 API） |
|---|---|---|
| シード固定 | `set.seed(n)` | `rng = np.random.default_rng(n)` |
| 一様乱数 | `runif(k)` | `rng.random(k)` |
| 正規乱数 | `rnorm(k)` | `rng.normal(size=k)` |
| 非復元抽出 | `sample(x, k)` | `rng.choice(x, k, replace=False)` |
| 復元抽出 | `sample(x, k, replace=TRUE)` | `rng.choice(x, k, replace=True)` |
| シャッフル | `sample(x)` | `rng.permutation(x)` |
| DF サンプリング | `slice_sample()` | `df.sample(random_state=rng)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **言語間で数列が違う**：同じシードでも R と NumPy は別アルゴリズム。**移植で乱数が絡む結果は数値一致しない**（統計的に同等かで評価する）。
    - **新旧 API の混在**：`np.random.seed`（グローバル）と `default_rng`（ローカル）を混ぜない。新 API に統一。
    - **並列・再現性**：マルチプロセスでは各ワーカーに独立なシードを（`rng.spawn` / `SeedSequence`）。同一シードの使い回しは相関を生む。
    - **`sample` の既定**：R `sample` は非復元が既定。NumPy `choice` は**復元が既定**（`replace=True`）。非復元は明示。
    - **バージョン差**：乱数列はライブラリのバージョンでも変わりうる。厳密再現には環境も固定。

## 関連項目

- [095. デバッグと検算のコツ](topic-095.md)
- [026. 条件で値を作る（case_when）](topic-026.md)
- [004. パッケージ管理と import](topic-004.md)
