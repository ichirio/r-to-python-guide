# 086. 関数定義 — `function` → `def`

!!! abstract "この項目の R→Python 対応"
    - **R**: `f <- function(x, y = 1) { ... }`（最後の式が戻り値）／無名 `\(x) x + 1`
    - **Python（推奨）**: `def f(x, y=1): ...`（`return` が必須）／無名 `lambda x: x + 1`
    - **要注意**: R は最後の式が暗黙の戻り値。Python は **`return` を書かないと `None`**

---

## R ではこう書く

```r
summarize_age <- function(x, digits = 1) {
  m  <- mean(x, na.rm = TRUE)
  sd <- sd(x, na.rm = TRUE)
  sprintf("%.*f (%.*f)", digits, m, digits + 1, sd)   # 最後の式が戻り値
}
summarize_age(c(45, 52, 60))

sq <- \(x) x^2      # 無名関数（R 4.1+）
```

!!! note "R の勘所"
    - **最後に評価した式が戻り値**（`return()` は任意）。
    - 既定引数 `y = 1`、可変長 `...`。
    - `\(x)` は無名関数の短縮（`function(x)` と同じ）。

---

## Python ではこう書く

```python
def summarize_age(x, digits=1):
    m  = x.mean()
    sd = x.std()
    return f"{m:.{digits}f} ({sd:.{digits+1}f})"   # return が必須

import pandas as pd
summarize_age(pd.Series([45, 52, 60]))

sq = lambda x: x**2          # 無名関数（単一式のみ）
```

!!! tip "実務ではこれ"
    - **`return` を明示**。書かないと `None` が返る（R の暗黙戻り値と最大の違い）。
    - 既定引数 `digits=1`、可変長 `*args` / キーワード可変長 `**kwargs`。
    - 無名関数 `lambda` は**単一式のみ**。複数行のロジックは `def` で名前を付ける。
    - 型ヒント `def f(x: pd.Series) -> str:` は任意だが可読性・補完に有効。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 関数定義 | `f <- function(x) {...}` | `def f(x): ...` |
| 戻り値 | 最後の式（`return` 任意） | `return`（必須） |
| 既定引数 | `function(x, y = 1)` | `def f(x, y=1)` |
| 可変長 | `...` | `*args`, `**kwargs` |
| 無名関数 | `\(x) x + 1` | `lambda x: x + 1` |
| 複数戻り値 | `list(a, b)` | `return a, b`（タプル） |

## つまずきポイント

!!! warning "R と Python の差"
    - **`return` 必須**：Python は明示しないと `None`。R の「最後の式が戻り値」に慣れていると忘れる。
    - **可変既定引数の罠**：`def f(x, acc=[])` のように**可変オブジェクトを既定値**にすると呼び出し間で共有される。既定は `None` にして中で生成する。
    - **スコープ**：Python の関数内から外の変数を書き換えるには `global`/`nonlocal`。R の `<<-` とは作法が違う（[093](topic-093.md)）。
    - **複数戻り値**：Python はタプルで返し `a, b = f()` で受ける。R は list/vector。

## 関連項目

- [087. 反復：apply 系と map](topic-087.md)
- [002. パイプとメソッドチェーン](topic-002.md)
- [093. NULL / 欠損の安全な処理](topic-093.md)
