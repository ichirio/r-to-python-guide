# 076. RTF 出力 — `r2rtf` / rtfreporter → 選択肢

!!! abstract "この項目の R→Python 対応"
    - **R**: `r2rtf`（clinical RTF の定番）／`rtables` の RTF 出力／rtfreporter
    - **Python（推奨）**: **定番なし**。現実解は「Python で集計 → **R（r2rtf）で清書**」の分業、または docx/HTML で代替
    - **要注意**: 規制提出の RTF は Python 単独では手薄。**RTF が要件なら R 併用を設計に織り込む**

TFL の提出フォーマットとして根強い RTF。ここは正直に「Python だけでは弱い」と認識するのが実務的です。

---

## R ではこう書く

```r
library(r2rtf)

tab |>
  rtf_title("Table 1. Demographics") |>
  rtf_colheader("var | A | B") |>
  rtf_body() |>
  rtf_encode() |>
  write_rtf("table1.rtf")
```

!!! note "R の勘所"
    - `r2rtf`：ページ設定・スパンヘッダ・脚注・改ページなど**臨床 RTF の作法**が揃う。提出実績が豊富。
    - `rtables` + `rtf` 出力、rtfreporter も選択肢。
    - RTF は「見た目を固定した提出物」。細部（フォント・余白・改ページ）が要件化されている。

---

## Python での選択肢（いずれも一長一短）

Python に `r2rtf` 級の定番はありません。要件に応じて選びます。

| 方針 | 方法 | 向き |
|---|---|---|
| **R 併用（推奨）** | Python で集計 DF を作り、`reticulate`/CSV 経由で **R の `r2rtf`** に渡して清書 | 提出品質の RTF が必須なとき |
| docx で代替 | `python-docx`（[075](topic-075.md)）で Word 出力 → 必要なら Word で RTF 保存 | 社内・非提出 |
| HTML/PDF で代替 | great_tables → HTML/画像、または HTML→PDF | レビュー・閲覧用 |
| 手書き RTF | RTF は制御語のテキスト形式。単純な表なら自力生成も可能だが**保守が重い** | 非推奨（限定的） |

=== "R 併用の考え方"

    ```python
    # Python：集計して「値の入った DF」を完成させる（丸め・n(%)済み）
    demo.to_csv("demo_table.csv", index=True)
    ```

    ```r
    # R：清書と RTF 化だけ担当
    library(r2rtf); library(readr)
    read_csv("demo_table.csv") |> rtf_body() |> rtf_encode() |> write_rtf("t1.rtf")
    ```

!!! tip "実務ではこれ"
    - **提出用 RTF が要件なら、集計は Python・清書は R（r2rtf）** の分業が現実的。数値検証は Python、体裁は実績ある R に任せる。
    - 社内レビューやドラフトなら docx/HTML で十分なことも多い。**「提出物か閲覧物か」で道具を分ける**。
    - RTF を Python で自作するのは、単純表以外は割に合わない。

---

## つまずきポイント

!!! warning "R と Python の差"
    - **Python 側の空白**：r2rtf に相当する成熟ライブラリがない。無理に Python 単独で提出 RTF を作らない。
    - **改ページ・ヘッダ繰り返し**：臨床 RTF の要件（各ページに見出し再掲など）は自作だと極めて煩雑。R に任せる。
    - **文字コード・特殊文字**：RTF は制御語エスケープが必要。日本語・記号でハマりやすい。
    - **役割分担の設計**：「Python=集計・検証、R=清書・提出」を最初に決めておくと後段が楽。

## 関連項目

- [066. 出力テーブルの考え方](topic-066.md)
- [075. Word 出力](topic-075.md)
- [100. R⇔Python 相互運用とパイプライン設計](../roadmap.md)
