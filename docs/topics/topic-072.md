# 072. 数値の小数点・右揃え

!!! abstract "この項目の R→Python 対応"
    - **R**: `gt::fmt_number(decimals=)` / `cols_align()`、または `formatC`/`sprintf` で文字列化
    - **Python（推奨）**: 表示だけなら pandas **`Styler.format`**、値を文字列化するなら **f-string**（→ [016](topic-016.md)）
    - **要注意**: 「表示の丸め」と「値の丸め」は別（[018](topic-018.md)）。右揃えは等幅か表エンジンに任せる

TFL の数値列は「小数第1位固定」「右揃え／小数点揃え」が定番。表示整形の道具を整理します。

---

## R ではこう書く

```r
library(gt)

tab |>
  gt() |>
  fmt_number(columns = c(A, B), decimals = 1) |>   # 小数1桁固定
  cols_align(align = "right", columns = c(A, B))

# あるいは値を文字列に固定
sprintf("%.1f", 49.5)   # "49.5"
```

!!! note "R の勘所"
    - `fmt_number(decimals=)`：表示だけ小数桁を固定（末尾ゼロも保持）。
    - `cols_align("right")`：右揃え。数値は既定で右。
    - 文字列に焼くなら `sprintf`/`formatC`（→ [016](topic-016.md)）。

---

## Python ではこう書く

=== "pandas Styler（表示）"

    ```python
    import pandas as pd
    df = pd.DataFrame({"stat":["mean","sd"], "A":[52.0, 7.94], "B":[49.5, 16.26]})

    (df.style
       .format({"A": "{:.1f}", "B": "{:.1f}"})          # 小数1桁
       .set_properties(subset=["A","B"], **{"text-align": "right"}))
    # → HTML / Excel（to_excel）に反映
    ```

=== "f-string（値を文字列化）"

    ```python
    # 値そのものを固定表記の文字列にする
    df["A"] = df["A"].map(lambda v: f"{v:.1f}")
    df["B"] = df["B"].map(lambda v: f"{v:.1f}")
    print(df.to_string(index=False))
    ```

    出力:

    ```text
    stat    A    B
    mean 52.0 49.5
      sd  7.9 16.3
    ```

=== "great_tables"

    ```python
    from great_tables import GT
    GT(df).fmt_number(columns=["A","B"], decimals=1)   # gt と同名
    ```

!!! tip "実務ではこれ"
    - **表示だけ整える**（数値のまま保持）→ `Styler.format` か great_tables `fmt_number`。ソート・検算は数値で可能。
    - **値を固定表記に焼く**（`"49.5"` の文字列にする）→ f-string で `map`。TFL の最終セルはこれが確実（末尾ゼロ・桁が動かない）。
    - **右揃え・小数点揃え**：数値は表エンジンで既定右揃え。テキストで揃えるなら等幅前提で `rjust`（→ [017](topic-017.md)）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 小数桁固定（表示） | `gt::fmt_number(decimals=)` | `Styler.format("{:.1f}")` / gt `fmt_number` |
| 小数桁固定（値） | `sprintf("%.1f", x)` | `f"{x:.1f}"` |
| 右揃え | `cols_align("right")` | `Styler.set_properties(text-align)` |
| 桁区切り | `fmt_number(sep_mark=",")` | `"{:,.1f}"` |
| パーセント | `fmt_percent()` | `f"{x:.1f}%"` |
| 末尾ゼロ保持 | `fmt_number` | `f"{x:.1f}"`（保持される） |

## つまずきポイント

!!! warning "R と Python の差"
    - **表示 ≠ 値**：`Styler.format` や `fmt_number` は**見た目だけ**。CSV に落とすと元の数値。値を固定したいなら f-string で文字列化してから出力。
    - **末尾ゼロ**：`round(49.50, 1)` は `49.5`（float は末尾ゼロを保持しない）。`"49.5"` と出すには f-string/format が必須。
    - **丸め方式**：`{:.1f}` の丸めが SAS とズレ得る（[018](topic-018.md), [065](topic-065.md)）。
    - **小数点揃え**：桁数が揃っていれば右揃えで小数点も揃う。桁が不揃いなら表エンジンの小数点揃え機能に任せる。

## 関連項目

- [016. 書式付き数値・sprintf](topic-016.md)
- [018. 数値の丸めと表示桁](topic-018.md)
- [074. pandas Styler で整形](topic-074.md)
