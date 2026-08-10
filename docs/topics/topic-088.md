# 088. 条件分岐 — `if` / `ifelse` → `if` / `np.where`

!!! abstract "この項目の R→Python 対応"
    - **R**: スカラー分岐 `if (cond) {...} else {...}`／ベクトル分岐 `ifelse(cond, a, b)`
    - **Python（推奨）**: スカラー `if/elif/else`（式版 `a if cond else b`）／ベクトル `np.where` / `np.select`
    - **要注意**: **スカラーの `if` とベクトルの `ifelse`/`np.where` は別物**。列に `if` を使うとエラー

制御フローの `if` と、列に効く条件分岐（[026](topic-026.md)）は道具が違います。ここを混同しない。

---

## R ではこう書く

```r
# スカラー分岐（制御フロー）
grade <- if (score >= 60) "pass" else "fail"

# ベクトル分岐（各要素）
ifelse(scores >= 60, "pass", "fail")
```

!!! note "R の勘所"
    - `if` は**長さ1の条件**用。ベクトルを渡すと警告/エラー（R 4.2+ はエラー）。
    - `ifelse(cond, a, b)` は**ベクトル化**。各要素に効く（[026](topic-026.md)）。
    - 多分岐は `if/else if/else` か `switch()`、ベクトルなら `case_when()`。

---

## Python ではこう書く

=== "スカラー（制御フロー）"

    ```python
    # 文の if
    if score >= 60:
        grade = "pass"
    else:
        grade = "fail"

    # 式の if（三項演算子）
    grade = "pass" if score >= 60 else "fail"
    ```

=== "ベクトル（列）"

    ```python
    import numpy as np
    np.where(scores >= 60, "pass", "fail")          # ifelse 相当
    # 多分岐は np.select（case_when 相当、→ [026]）
    ```

!!! tip "実務ではこれ"
    - **1 個の値の分岐** → `if/elif/else`、または式 `a if cond else b`。
    - **列（Series/array）の分岐** → `np.where(cond, a, b)`、多分岐は `np.select`。**列に文の `if` は使えない**（真偽が曖昧でエラー）。
    - `switch()` に当たるのは辞書ディスパッチ（`{"a": fa, "b": fb}[key]()`）や `match`（Python 3.10+）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| スカラー分岐（文） | `if (c) {...} else {...}` | `if c: ... else: ...` |
| スカラー分岐（式） | `if (c) a else b` | `a if c else b` |
| ベクトル分岐 | `ifelse(c, a, b)` | `np.where(c, a, b)` |
| 多分岐（スカラー） | `switch()` | `match`（3.10+）/ dict |
| 多分岐（ベクトル） | `case_when()` | `np.select([...],[...])` |

## つまずきポイント

!!! warning "R と Python の差"
    - **スカラー if にベクトルは不可**：`if series >= 60:` は「真偽が曖昧」で `ValueError`。列は `np.where`。R も `if` にベクトルはエラー。
    - **`ifelse` の型・欠損**：R の `ifelse` は NA を伝播。`np.where` は NaN を偽扱い（[009](topic-009.md), [026](topic-026.md)）。欠損の分岐は要検算。
    - **三項演算子の順序**：Python は `a if cond else b`（値・条件・値の順）。R の `if (cond) a else b` と語順が違う。
    - **短絡評価**：`and`/`or` は短絡。`x is not None and x > 0` の順序で None 安全に（[093](topic-093.md)）。

## 関連項目

- [026. 条件で値を作る（case_when）](topic-026.md)
- [009. 列作成・変更（mutate）](topic-009.md)
- [093. NULL / 欠損の安全な処理](topic-093.md)
