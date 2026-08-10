# 017. ゼロ埋め・桁揃え — `formatC` / `str_pad` → `zfill` / `rjust`

!!! abstract "この項目の R→Python 対応"
    - **R**: `formatC(x, width=, flag="0")` / `sprintf("%03d", x)` / `str_pad(x, width, pad="0")`
    - **Python（推奨）**: 数値のゼロ埋めは **`f"{x:03d}"`**、文字列は **`str.zfill()` / `str.rjust()` / `str.ljust()`**
    - **要注意**: `zfill` は符号を考慮するが、右寄せ `rjust("0")` は符号も 0 で潰す

被験者番号や訪問番号を `"001"` に揃える、値を右寄せする、といった TFL 定番の整形。

---

## R ではこう書く

```r
library(stringr)

formatC(7, width = 3, flag = "0")           # "007"
sprintf("%03d", 7)                          # "007"
str_pad("7", 3, pad = "0")                  # "007"
str_pad("x", 5, side = "right")             # "x    "（右側に空白）
formatC(3.1, width = 8, format = "f", digits = 2)  # "    3.10"（右寄せ）
```

出力:

```text
007
007
007
x
    3.10
```

（`str_pad("x", 5, side="right")` は `"x    "` = 右に空白4つ）

!!! note "使い分け"
    - **数値のゼロ埋め**：`sprintf("%03d", n)` か `formatC(n, width=3, flag="0")`。
    - **文字列のパディング**：`str_pad(x, width, side=, pad=)`。`side` は `"left"`（右寄せ）/`"right"`/`"both"`。
    - **右寄せで幅揃え**（表の数値列）：`formatC(x, width=, format="f", digits=)`。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 数値のゼロ埋め
    print(f"{7:03d}")          # "007"
    print(str(7).zfill(3))     # "007"（文字列から）

    # 文字列のパディング
    print("[" + "x".ljust(5) + "]")     # 左寄せ（右に空白）
    print("[" + "x".rjust(5) + "]")     # 右寄せ（左に空白）
    print("[" + f"{3.1:8.2f}" + "]")    # 幅8で右寄せ

    # 列に対して
    import pandas as pd
    s = pd.Series([1, 22, 333])
    print(s.astype(str).str.zfill(3).tolist())   # ['001','022','333']
    ```

    出力:

    ```text
    007
    007
    [x    ]
    [    x]
    [    3.10]
    ```

=== "polars"

    ```python
    import polars as pl
    s = pl.Series([1, 22, 333])
    print(s.cast(pl.Utf8).str.zfill(3).to_list())   # ['001','022','333']
    ```

!!! tip "実務ではこれ"
    - **ID のゼロ埋め**：数値なら `f"{n:03d}"`、列なら `s.astype(str).str.zfill(3)`。
    - **数値列の右寄せ**（表体裁）は `f"{x:>8.2f}"` か Styler に任せる（→ [072](../roadmap.md)）。
    - `zfill` は**文字列メソッド**なので、数値列はいったん `astype(str)` する。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 数値ゼロ埋め | `sprintf("%03d", n)` | `f"{n:03d}"` | `str.zfill(3)`（cast 後） |
| 文字列ゼロ埋め | `str_pad(x,3,pad="0")` | `x.zfill(3)` / `s.str.zfill(3)` | `s.str.zfill(3)` |
| 右寄せ（空白） | `str_pad(x,8)` | `x.rjust(8)` / `f"{x:>8}"` | `s.str.pad_start(8)` |
| 左寄せ（空白） | `str_pad(x,8,"right")` | `x.ljust(8)` / `f"{x:<8}"` | `s.str.pad_end(8)` |
| 中央寄せ | `str_pad(x,8,"both")` | `x.center(8)` / `f"{x:^8}"` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **`side` の向きが紛らわしい**：R の `str_pad(side="left")` は**左に詰めて右寄せ**。Python の `rjust` が右寄せ（左を埋める）。言葉と結果が逆に感じるので確認。
    - **符号の扱い**：`"-7".zfill(4)` は `"-007"`（符号を保持）。一方 `"-7".rjust(4, "0")` は `"00-7"`。負値 ID は `zfill` を使う。
    - **`zfill` は文字列専用**：数値に直接は使えない。`f"{n:03d}"` か `astype(str)`。
    - **全角・多バイト幅**：日本語混じりの「見た目の桁揃え」は文字数ベースの pad ではズレる。等幅前提の表なら英数字に限る。

## 関連項目

- [016. 書式付き数値・sprintf](topic-016.md)
- [072. 数値の小数点・右揃え](../roadmap.md)
