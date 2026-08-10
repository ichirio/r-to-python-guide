# 006. 文字列結合 — `paste` / `paste0` / `glue` → f-string ほか

!!! abstract "この項目の R→Python 対応"
    - **R**: `paste()`（区切り付き）／`paste0()`（区切りなし）／`str_c()`（stringr、NA 厳格）／`glue()`（変数埋め込み）／`sprintf()`（書式付き）
    - **Python（推奨）**: **f-string**（1件の組み立て）／**`Series.str.cat` や `+`**（列同士）／polars **`pl.concat_str`**
    - **要注意**: R は結合が**ベクトル化**され自動で各行に効く。Python は「スカラーか列か」で道具を替える

文字列結合は R でも複数のやり方があり、使い分けがあります。まず R 側を整理してから Python の最善手へ。

---

## R ではこう書く

```r
library(glue)

paste("A", "B", "C")                  # 既定は空白区切り
paste0("SUBJ", "001")                 # 区切りなし（sep="")
paste0("SUBJ", 1:3)                   # ベクトル化：各要素に効く
paste(c("A","B","C"), collapse = "-") # 1本にまとめる

n <- 5; pct <- 12.3456
glue("{n} ({format(round(pct,1), nsmall=1)}%)")  # 変数埋め込み
sprintf("%d (%.1f%%)", n, pct)                   # 書式付き（C 由来）
```

出力:

```text
[1] "A B C"
[1] "SUBJ001"
[1] "SUBJ1" "SUBJ2" "SUBJ3"
[1] "A-B-C"
5 (12.3%)
[1] "5 (12.3%)"
```

!!! note "R 側の使い分け"
    | 関数 | 区切り | 得意なこと | ひとこと |
    |---|---|---|---|
    | `paste()` | 既定 `" "` | 複数ベクトルを横に結合 | `sep=` と `collapse=` を混同しない |
    | `paste0()` | なし | ID 生成など区切り不要な結合 | `paste(sep="")` の短縮形 |
    | `str_c()` | なし | tidyverse で一貫 | **NA が混ざると結果も NA**（厳格） |
    | `glue()` | — | `"{var}"` で読みやすい埋め込み | 可読性が最高。ログ・メッセージ向き |
    | `sprintf()` | — | 桁・幅・ゼロ埋めの**書式** | `%d %s %.1f` を使うレポート向き |

    - **`sep=` と `collapse=` の違い**：`sep` は「複数ベクトルを横に繋ぐ区切り」、`collapse` は「1本のベクトルを縦に畳む区切り」。
    - **NA の扱い**：`paste()` は NA を文字列 `"NA"` にする。`str_c()` は NA を伝播する。TFL で欠損を空欄にしたいかで選ぶ。

---

## Python ではこう書く

「1件を組み立てる」のか「列（Series）同士を結合する」のかで最善手が変わります。

=== "pandas"

    ```python
    import pandas as pd

    # 1件の組み立て → f-string（最も読みやすい）
    print("SUBJ" + "001")
    n, pct = 5, 12.3456
    print(f"{n} ({pct:.1f}%)")        # 書式付きもここで完結

    # 列同士 → + でベクトル化（R の paste0 相当）
    print("SUBJ" + pd.Series(["1", "2", "3"]))

    # 1本に畳む（collapse 相当） → str.join
    print("-".join(["A", "B", "C"]))
    ```

    出力:

    ```text
    SUBJ001
    5 (12.3%)
    0    SUBJ1
    1    SUBJ2
    2    SUBJ3
    dtype: object
    A-B-C
    ```

    複数列を区切り付きで結合するなら `str.cat`：

    ```python
    dm["label"] = dm["arm"].str.cat(dm["subjid"], sep="-")   # 例: "A-01"
    ```

=== "polars"

    ```python
    import polars as pl

    # 列を結合（paste0 相当）
    print(dmp.select(
        pl.concat_str([pl.lit("SUBJ"), pl.col("subjid")]).alias("id")
    ))

    # 区切り付きは separator=
    # pl.concat_str([pl.col("arm"), pl.col("subjid")], separator="-")
    ```

    出力:

    ```text
    shape: (5, 1)
    ┌────────┐
    │ id     │
    │ ---    │
    │ str    │
    ╞════════╡
    │ SUBJ01 │
    │ SUBJ02 │
    │ SUBJ03 │
    │ SUBJ04 │
    │ SUBJ05 │
    └────────┘
    ```

!!! tip "実務ではこれ"
    - **メッセージ・1件の整形** → **f-string**。`f"{pct:.1f}%"` で書式も同時に決まり、`sprintf` と `glue` の役割を1つでこなす。
    - **列同士の結合** → pandas は **`Series.str.cat(..., sep=)`**（複数列や NA 制御が効く）、polars は **`pl.concat_str(..., separator=)`**。
    - **たくさん繋ぐ / 区切りで畳む** → `"sep".join(iterable)`。ループで `+=` するより速く読みやすい。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 区切りなし結合 | `paste0(a, b)` | `a + b` / `f"{a}{b}"` | `pl.concat_str([a, b])` |
| 区切りあり結合 | `paste(a, b, sep="-")` | `s1.str.cat(s2, sep="-")` | `pl.concat_str([a, b], separator="-")` |
| 変数埋め込み | `glue("{x}")` | `f"{x}"` | `f"{x}"`（Python 文字列側） |
| 書式付き | `sprintf("%.1f", x)` | `f"{x:.1f}"` | `f"{x:.1f}"` |
| 1本に畳む | `paste(v, collapse="-")` | `"-".join(v)` | `s.str.join("-")`（リスト列） |
| NA を伝播させたい | `str_c()` | `str.cat(..., na_rep=None)` | 既定で null 伝播 |

## つまずきポイント

!!! warning "R と Python の差"
    - **ベクトル化の自動さ**：R は `paste0("SUBJ", 1:3)` が黙って各要素に効く。Python の f-string は**スカラー専用**で、列に効かせるには `Series` の `+` / `str.cat` を使う。
    - **数値の暗黙変換**：R の `paste0("SUBJ", 1)` は数値を勝手に文字列化する。Python の `"SUBJ" + 1` は **TypeError**。`str(1)` か f-string を使う。
    - **NA の見え方**：`paste()` は NA を `"NA"` という文字にする。pandas の `str.cat` は既定で欠損行の結果を欠損にする（`na_rep=` で `"NA"` などに変えられる）。TFL では「空欄にするか `NA` と出すか」を必ず意識する。
    - **`sep` と `collapse`**：R 特有の2種類の区切り。Python では「横に繋ぐ＝`str.cat`/`+`」「縦に畳む＝`join`」と道具自体が分かれるので取り違えにくい。

## 関連項目

- [016. 書式付き数値・sprintf](../roadmap.md)
- [019. パーセント表記の作成](../roadmap.md)
- [039. 列の結合（unite）](../roadmap.md)
