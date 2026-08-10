# 071. 見出し・スパンヘッダ・脚注

!!! abstract "この項目の R→Python 対応"
    - **R**: `gt`（`tab_spanner`/`tab_header`/`tab_footnote`）や `flextable` で多層見出し・脚注
    - **Python（推奨）**: 構造は pandas **MultiIndex 列**、清書は **`great_tables`**（`tab_spanner` 等、gt とほぼ同名）
    - **要注意**: 「群をまとめるスパンヘッダ」は pandas では MultiIndex 列、great_tables では `tab_spanner`

TFL の「Treatment ┃ [Placebo | Drug]」のような**2 段ヘッダ**、表題、脚注の付け方。

---

## R ではこう書く

```r
library(gt)

tab |>
  gt(rowname_col = "var") |>
  tab_header(title = "Table 1. Demographics",
             subtitle = "Safety population") |>
  tab_spanner(label = "Treatment", columns = c(A, B)) |>   # 群をまとめる上段見出し
  tab_footnote("n (%) or mean (SD).")
```

!!! note "R の勘所"
    - `gt` は `tab_spanner`（スパンヘッダ）、`tab_header`（表題）、`tab_footnote`（脚注）を宣言的に付ける。
    - `flextable` なら `add_header_row` で多層ヘッダ、`footnote()` で脚注。

---

## Python ではこう書く

=== "great_tables（清書）"

    ```python
    from great_tables import GT, md

    (GT(tab, rowname_col="var")
       .tab_header(title="Table 1. Demographics", subtitle="Safety population")
       .tab_spanner(label="Treatment", columns=["A", "B"])   # gt と同名！
       .tab_source_note(source_note=md("*n (%) or mean (SD).*")))
    ```

    → HTML/画像/Word に出力（`.as_raw_html()` / `.save("t.png")` など）。メソッド名が gt とほぼ同じで移行しやすい。

=== "pandas（構造だけ）"

    ```python
    import pandas as pd
    df = pd.DataFrame({
        ("Placebo","n"):[10], ("Placebo","%"):[50.0],
        ("Drug","n"):[12], ("Drug","%"):[60.0],
    })
    df.columns = pd.MultiIndex.from_tuples(df.columns, names=["Treatment", None])
    print(df)
    ```

    出力:

    ```text
    Treatment Placebo       Drug
                    n     %    n     %
    0              10  50.0   12  60.0
    ```

!!! tip "実務ではこれ"
    - **見た目まで作るなら great_tables**：`tab_spanner` / `tab_header` / `tab_source_note` が gt と同名で、脚注・表題・スパンが宣言的。
    - **構造だけ pandas で持つ**なら MultiIndex 列。Excel 出力時はセル結合で2段ヘッダになる（openpyxl、→ [077](topic-077.md)）。
    - 脚注記号（†, ‡）との対応は、great_tables の `tab_footnote(locations=)` でセル指定できる。

---

## 対応早見表

| やりたいこと | R（gt） | Python（great_tables） |
|---|---|---|
| 表題 | `tab_header(title=)` | `tab_header(title=)` |
| スパンヘッダ | `tab_spanner(label=, columns=)` | `tab_spanner(label=, columns=)` |
| 脚注 | `tab_footnote()` | `tab_footnote()` / `tab_source_note()` |
| 行見出し列 | `gt(rowname_col=)` | `GT(rowname_col=)` |
| 列ラベル変更 | `cols_label()` | `cols_label()` |
| 2段ヘッダ（構造） | tibble | pandas MultiIndex 列 |

## つまずきポイント

!!! warning "R と Python の差"
    - **構造 vs 体裁**：pandas の MultiIndex は「構造」。見た目（罫線・脚注記号・配置）は great_tables/Styler の担当。R の gt は両方を1つで。
    - **great_tables の対応範囲**：gt の全機能を網羅はしていない（バージョンで拡張中）。込み入った rtables 的レイアウトは難しいことがある。
    - **Excel の2段ヘッダ**：MultiIndex を Excel に書くとセル結合される。読み戻すと扱いにくいので、出力専用と割り切る。
    - **脚注の位置指定**：セル単位の脚注記号は great_tables の `locations=` で。手作業だとズレやすい。

## 関連項目

- [066. 出力テーブルの考え方](topic-066.md)
- [073. great_tables で整形](topic-073.md)
- [072. 数値の小数点・右揃え](topic-072.md)
