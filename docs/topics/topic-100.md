# 100. R⇔Python 相互運用とパイプライン設計 — reticulate / rpy2

!!! abstract "この項目の R→Python 対応"
    - **R → Python 呼び出し**: `reticulate`（R から Python を使う）
    - **Python → R 呼び出し**: `rpy2`（Python から R を使う）
    - **要注意**: 全部を一方の言語に寄せなくてよい。**「集計は Python、提出清書は R」**のような役割分担が実務的

最終回。R と Python を**対立でなく分業**で組む設計指針です。移行は「全置換」でなく「適材適所」。

---

## どちらから呼ぶか

=== "R から Python（reticulate）"

    ```r
    library(reticulate)
    pd <- import("pandas")
    np <- import("numpy")

    df <- pd$read_csv("data.csv")
    # R のデータフレーム ⇄ pandas を自動変換
    py_df <- r_to_py(r_dataframe)
    r_df  <- py_to_r(py_object)
    ```

    R Markdown/Quarto では ```python チャンクをそのまま実行でき、`py$obj` / `r$obj` で相互参照。

=== "Python から R（rpy2）"

    ```python
    # pip install rpy2
    import rpy2.robjects as ro
    from rpy2.robjects import pandas2ri
    pandas2ri.activate()

    ro.r("library(r2rtf)")
    ro.globalenv["df"] = my_pandas_df          # pandas → R data.frame
    ro.r('df |> rtf_body() |> rtf_encode() |> write_rtf("t.rtf")')
    ```

---

## パイプライン設計の指針

TFL パイプライン全体（SAP → shells → ARD → 表）を R/Python どちらで、が問いです。**層ごとに強い方**を選ぶのが現実解。

| 層 | 強い方 | 理由 |
|---|---|---|
| データ加工・集計 | Python（pandas/polars）or R（dplyr） | どちらも十分。既存資産・チームスキルで選ぶ |
| 統計モデル | 場合による | 特殊な臨床モデルは R が厚い（生存・混合効果の一部） |
| 図 | どちらも可 | plotnine で ggplot 資産を活かせる |
| **提出 RTF** | **R（r2rtf/rtables）** | Python 側が手薄（[076](topic-076.md)） |
| 再現・パイプライン | targets（R）/ snakemake・papermill（Py） | ワークフロー管理 |

!!! tip "実務ではこれ"
    - **全部を片方に寄せない**：「Python で集計・検証 → R で提出清書（RTF）」のハイブリッドが、移行期の最も安全な着地点。
    - **データの受け渡し**は CSV/parquet/XPT（[097](topic-097.md)）でファイル経由にすると疎結合で堅い。相互運用ライブラリ（reticulate/rpy2）は密結合で便利だが依存が増える。
    - **数値の検証**は言語をまたいで突き合わせる（[095](topic-095.md)）。丸め・欠損・検定の既定差（[005](topic-005.md), [018](topic-018.md), [060](topic-060.md)）を重点確認。
    - **チームの現実**：レビュアーが読める言語、既存マクロ資産、提出要件で決める。技術的な優劣だけで選ばない。

---

## 対応早見表

| やりたいこと | 方法 |
|---|---|
| R から Python | `reticulate::import()` / py チャンク |
| Python から R | `rpy2.robjects` |
| DF 変換（R→Py） | `r_to_py()` / `pandas2ri` |
| DF 変換（Py→R） | `py_to_r()` / `pandas2ri` |
| ファイル経由連携 | CSV / parquet / XPT |
| ワークフロー | targets（R）/ snakemake・papermill（Py）/ Quarto |

## つまずきポイント

!!! warning "R と Python の差・相互運用の注意"
    - **依存の複雑化**：reticulate/rpy2 は「R と Python の両環境」を1プロセスに同居させる。バージョン整合・環境構築が難しくなる。ファイル経由の疎結合が安全なことも多い。
    - **型変換のズレ**：因子↔Categorical、日付、欠損（NA/NaN/None）は変換で意味が変わりうる（[005](topic-005.md), [091](topic-091.md)）。境界で検算。
    - **提出要件が最終審**：RTF・XPT・定義書などの規制要件は R 側ツールが実績豊富。無理に Python 単独にしない。
    - **移行は段階的に**：1 プログラムずつ Python 化し、R の出力と数値照合しながら進める（ビッグバン移行を避ける）。

---

!!! success "全100項目 完"
    R（Tidyverse・TFL）を起点に、pandas/polars での最善手を対比してきました。
    移行の要点は3つ——**ベクトル化の発想**、**既定差（0/1始まり・欠損・丸め）の把握**、**適材適所の分業**。
    このガイドが、R の資産を活かしながら Python を実務に組み込む一助になれば幸いです。

## 関連項目

- [076. RTF 出力](topic-076.md)
- [095. デバッグと検算のコツ](topic-095.md)
- [097. SAS / SPSS / Stata の読み書き](topic-097.md)
- [ロードマップ（全100項目）](../roadmap.md)
