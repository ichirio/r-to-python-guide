# 087. 反復：apply 系と map — `sapply` / `purrr::map` → comprehension / `apply`

!!! abstract "この項目の R→Python 対応"
    - **R**: `sapply` / `lapply` / `vapply` / `mapply`、purrr の `map` / `map_dbl` / `map2` / `pmap`
    - **Python（推奨）**: **リスト内包表記**（第一選択）／`map()`／pandas の `Series.apply` / `DataFrame.apply`
    - **要注意**: DataFrame の行/列に対する処理はまず**ベクトル化**を検討（[033](topic-033.md), [090](topic-090.md)）。`apply` は最後の手段

---

## R ではこう書く

```r
sapply(1:3, function(x) x^2)          # ベクトルを返す → 1 4 9
lapply(1:3, function(x) x^2)          # リストを返す

library(purrr)
map_dbl(1:3, ~ .x^2)                  # 型を指定して返す
map2_dbl(1:3, 4:6, ~ .x + .y)         # 2 引数
pmap_dbl(list(a, b, c), ~ ..1 + ..2)  # 複数引数
```

出力（`sapply`）:

```text
[1] 1 4 9
```

!!! note "R の勘所"
    - `sapply` は簡約（ベクトル/行列）、`lapply` はリスト、`vapply` は型を固定（安全）。
    - purrr の `map_*` は**戻り型を接尾辞で指定**（`_dbl`/`_chr`/`_dfr`）。
    - `map_dfr` は結果を行方向に結合してデータフレームに。

---

## Python ではこう書く

=== "内包表記・map"

    ```python
    [x**2 for x in range(1, 4)]          # → [1, 4, 9]（第一選択）
    list(map(lambda x: x**2, range(1, 4)))
    [a + b for a, b in zip([1,2,3], [4,5,6])]   # map2 相当
    ```

=== "pandas apply"

    ```python
    import pandas as pd
    pd.Series([1,2,3]).apply(lambda x: x**2).tolist()   # → [1, 4, 9]

    # 各行を辞書化して関数へ（map_dfr 的にデータを組む）
    rows = [make_row(r) for _, r in df.iterrows()]
    pd.DataFrame(rows)
    ```

    出力:

    ```text
    [1, 4, 9]
    ```

!!! tip "実務ではこれ"
    - **まずリスト内包表記**（`[f(x) for x in xs]`）。R の `sapply`/`map` の大半はこれで足りる。
    - **pandas 列に関数** → `Series.apply`。ただし**ベクトル演算で書けるなら書く**（速い、[090](topic-090.md)）。
    - **行ごと** → `df.apply(f, axis=1)`（遅い、[033](topic-033.md)）。可能ならベクトル化。
    - `map_dfr` 的に「各要素→行→結合」は、辞書のリストを作って `pd.DataFrame(...)` が読みやすい。

---

## 対応早見表

| R | Python |
|---|---|
| `sapply(xs, f)` | `[f(x) for x in xs]` |
| `lapply(xs, f)` | `[f(x) for x in xs]`（リスト） |
| `map_dbl(xs, f)` | `[f(x) for x in xs]` / `np.array([...])` |
| `map2(xs, ys, f)` | `[f(x,y) for x,y in zip(xs,ys)]` |
| `pmap(list, f)` | `[f(*t) for t in zip(...)]` |
| `map_dfr(xs, f)` | `pd.DataFrame([f(x) for x in xs])` |
| `sapply(df, f)`（列ごと） | `df.apply(f)`（既定 axis=0） |

## つまずきポイント

!!! warning "R と Python の差"
    - **ベクトル化優先**：`apply`/内包表記は「関数を1つずつ呼ぶ」ので、pandas/NumPy のベクトル演算より遅い。数値処理はベクトル化（[090](topic-090.md)）。
    - **`apply` の axis**：pandas は `axis=0`（列ごと、既定）/`axis=1`（行ごと）。R の `apply(m, 1, f)`（行）/`apply(m, 2, f)`（列）と番号の意味が違う。
    - **戻り型**：purrr は接尾辞で型を保証。Python の内包表記は list、数値配列が要るなら `np.array(...)`。
    - **`map` は遅延**：Python の `map()` はイテレータ。`list()` しないと中身が出ない。

## 関連項目

- [033. 行ごと処理・rowwise](topic-033.md)
- [090. ベクトル化の考え方](topic-090.md)
- [086. 関数定義](topic-086.md)
