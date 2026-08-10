# 070. リスティング（listing）

!!! abstract "この項目の R→Python 対応"
    - **R**: 集計せず**元データを整形して並べる**（`arrange` → 体裁 → `r2rtf`/`flextable`）
    - **Python（推奨）**: pandas で列選択・並べ替え・整形 → great_tables / Word / Excel へ
    - **要注意**: リスティングは**集計しない**。全レコードを可読に並べるのが目的。ソートキーと繰り返し値の抑制が肝

被験者データ行を1件ずつ載せる一覧表（AE listing、逸脱 listing など）。表（集計）とは作り方が違います。

---

## R ではこう書く

```r
library(dplyr)

ae |>
  arrange(subjid, aestdt) |>              # 並べ替え
  select(subjid, arm, pt, aestdt, sev) |> # 出す列
  mutate(sev = recode(sev, "1"="Mild","2"="Moderate","3"="Severe"))
  # → r2rtf / flextable で清書・改ページ
```

!!! note "R の勘所"
    - リスティングは**集計しない**。`arrange` で並べ、`select` で列を選び、コードをラベルへ。
    - 同じ被験者が続くとき、2 行目以降の subjid を空白にする（繰り返し抑制）ことが多い。
    - ページ跨ぎの見出し繰り返しは `r2rtf`/`rtables` が面倒を見る。

---

## Python ではこう書く

=== "pandas"

    ```python
    lst = (ae.sort_values(["subjid","aestdt"])
              [["subjid","arm","pt","aestdt","sev"]]
              .assign(sev=lambda d: d["sev"].map({"1":"Mild","2":"Moderate","3":"Severe"})))

    # 繰り返し値の抑制（同じ subjid の2行目以降を空に）
    lst["subjid"] = lst["subjid"].mask(lst["subjid"].duplicated(), "")
    print(lst.to_string(index=False))
    ```

!!! tip "実務ではこれ"
    - **`sort_values` → 列選択 → コードをラベルへ `map`**。集計は挟まない。
    - **繰り返し抑制**：`col.mask(col.duplicated(), "")`（連続重複だけ消すなら `groupby` 内で）。
    - 日付は表示用に文字列化（`dt.strftime`、→ [092](../roadmap.md)）。
    - 清書・改ページは出力層で：great_tables（HTML/画像）、`python-docx`（Word）、`openpyxl`（Excel）。大きい listing は Excel が扱いやすい。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 並べ替え | `arrange(...)` | `sort_values([...])` |
| 出す列 | `select(...)` | `df[[...]]` |
| コード→ラベル | `recode` / `left_join` | `map(dict)` |
| 繰り返し抑制 | 自前 / rtables | `col.mask(col.duplicated(), "")` |
| 日付整形 | `format(date)` | `dt.strftime("%Y-%m-%d")` |
| 出力 | `r2rtf` / `flextable` | great_tables / docx / openpyxl |

## つまずきポイント

!!! warning "R と Python の差"
    - **集計しない**：listing は全件表示。うっかり `groupby` すると別物になる。
    - **繰り返し抑制の範囲**：`duplicated()` は「直前と同じ」ではなく「既出」を消す。連続重複だけ消したいなら `df.groupby("subjid").cumcount()>0` を使う。
    - **行数が多い**：数千行の listing は great_tables/Word だと重い。Excel（openpyxl/xlsxwriter）や CSV が現実的。
    - **改ページ・ヘッダ繰り返し**：Python 側は自前。提出品質の RTF が要るなら R 併用も検討（[076](topic-076.md)）。

## 関連項目

- [021. 並べ替え](topic-021.md)
- [047. ルックアップで値付与](topic-047.md)
- [077. Excel 出力](topic-077.md)
