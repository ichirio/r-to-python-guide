# ロードマップ（全100項目）

R（Tidyverse・TFL）を Python でどう書くかを、R を起点にまとめる全100項目の一覧です。
`[x]` は執筆済み。**番号は安定 ID**（振り直さない）で、表示順は必要に応じて `nav` 側だけ変えます。

!!! info "進捗"
    第1部〜第5部（001–065）執筆済み。次は第6部 TFL 作成。以降は順次追加します。
    「この項目を先に」というリクエストがあれば優先して書けます。

---

## 第1部　R→Python の勘所（まず10項目）

R から Python に移るときに最初に効く、発想と中核 verb。

- [x] 001. R と Python の考え方の違い（ベクトル化・0/1始まり・代入・真偽値・欠損）
- [x] 002. パイプとメソッドチェーン（`%>%` / `|>` → method chain / `pipe`）
- [x] 003. データフレーム入門（`data.frame` / `tibble` → pandas / polars）
- [x] 004. パッケージ管理と import（`install.packages` / `library` → pip / conda / import）
- [x] 005. 欠損値の扱い（`NA` → `NaN` / `None` / `pd.NA`）
- [x] 006. 文字列結合（`paste` / `paste0` / `glue` → f-string ほか）
- [x] 007. 列選択（`select` → `[]` / `filter` / polars `select`）
- [x] 008. 行フィルタ（`filter` → `query` / boolean mask / polars `filter`）
- [x] 009. 列作成・変更（`mutate` → `assign` / `[]` / `with_columns`）
- [x] 010. グループ集約（`group_by` + `summarise` → `groupby.agg` / polars `group_by`）

## 第2部　文字列と書式

臨床の表づくりは文字列整形が半分。stringr / glue の使い分けから。

- [x] 011. 文字列の抽出・長さ（`substr` / `str_sub` / `nchar` → slice / `len` / `.str`）
- [x] 012. 検索と置換（`gsub` / `sub` / `str_replace` → `re.sub` / `str.replace`）
- [x] 013. パターン検出・抽出（`grepl` / `str_detect` / `str_extract` → `str.contains` / `str.extract`）
- [x] 014. 分割と連結（`strsplit` / `str_split`, collapse → `split` / `join`）
- [x] 015. 大文字小文字・前後空白（`toupper` / `str_to_*` / `str_trim` → `upper` / `strip`）
- [x] 016. 書式付き数値・`sprintf`（`sprintf` / `formatC` / `format` → format spec）
- [x] 017. ゼロ埋め・桁揃え（`formatC` / `str_pad` → `zfill` / `rjust`）
- [x] 018. 数値の丸めと表示桁（`round` / `signif` / `formatC` → `round` / `format`、丸め方式）
- [x] 019. パーセント表記の作成（`n (xx.x%)` の組み立て）
- [x] 020. 正規表現の違い（R の regex 系 vs Python `re` の作法）

## 第3部　dplyr データ加工の中核

filter / mutate / summarise の先にある、実務で毎日使う verb 群。

- [x] 021. 並べ替え（`arrange` → `sort_values`）
- [x] 022. 件数と頻度（`count` / `n()` / `tally` → `value_counts` / `size`）
- [x] 023. 重複の除去（`distinct` → `drop_duplicates`）
- [x] 024. 上位N・スライス（`slice_max` / `top_n` / `slice` → `nlargest` / `head`）
- [x] 025. リネーム（`rename` → `rename`）
- [x] 026. 条件で値を作る（`case_when` / `if_else` → `np.select` / `np.where`）
- [x] 027. 値の対応付け・recode（`recode` / `case_match` → `map` / `replace`）
- [x] 028. 型変換（`as.numeric` / `as.character` → `astype` / `to_numeric`）
- [x] 029. `across` で複数列に適用（`across` → agg dict / `apply`）
- [x] 030. グループ内変換（`group_by` + `mutate` → `groupby.transform`）
- [x] 031. ウィンドウ関数：lag / lead / cumsum（→ `shift` / `cumsum`）
- [x] 032. ランク付け（`min_rank` / `dense_rank` / `row_number` → `rank`）
- [x] 033. 行ごと処理・rowwise（`rowwise` → `apply(axis=1)` / ベクトル化）
- [x] 034. ビン分割（`cut` → `pd.cut` / `qcut`）
- [x] 035. 欠損の補完（`coalesce` / `replace_na` / `fill` → `fillna` / `ffill`）

## 第4部　tidyr 整形・結合

ロング／ワイドの往復と join。TFL レイアウトづくりの核心。

- [x] 036. 縦持ち化（`pivot_longer` → `melt`）
- [x] 037. 横持ち化（`pivot_wider` → `pivot` / `pivot_table`）
- [x] 038. 列の分割（`separate` / `separate_wider_*` → `str.split(expand=True)`）
- [x] 039. 列の結合（`unite` → `cat` / `agg`）
- [x] 040. 行の展開（`separate_rows` / `unnest` → `explode`）
- [x] 041. 内部結合（`inner_join` → `merge`）
- [x] 042. 左・外部結合（`left_join` / `full_join` → `merge(how=)`）
- [x] 043. アンチ・セミ結合（`anti_join` / `semi_join` → indicator / `isin`）
- [x] 044. 縦結合・横結合（`bind_rows` / `bind_cols` → `concat`）
- [x] 045. クロス結合・総当たり（`cross_join` / `expand_grid` → `merge(how="cross")`）
- [x] 046. 欠けた組合せを補完（`complete` / `expand` → `reindex`）
- [x] 047. ルックアップで値付与（`left_join` lookup → `map`）
- [x] 048. 結合キーの検証（多対多・重複チェック）
- [x] 049. 集計してから結合（join + summarise）
- [x] 050. ロング／ワイドを行き来する実務パターン（TFL整形）

## 第5部　集計・統計・要約（TFL向け）

デモグラ表・AE 表の中身。数値の作り方と丸め。

- [x] 051. 記述統計の一括（`summary` → `describe`）
- [x] 052. 連続変数の要約（`mean` / `sd` / `median` / `quantile` → `agg`）
- [x] 053. カテゴリの頻度と割合（`table` / `prop.table` → `value_counts` / `crosstab`）
- [x] 054. クロス集計表（`xtabs` / `table` → `crosstab` / `pivot_table`）
- [x] 055. 群別 N・mean(sd) のデモグラ表
- [x] 056. 欠損数の集計（`sum(is.na)` → `isna().sum`）
- [x] 057. `n (%)` 整形のパターン
- [x] 058. 中央値 [Q1, Q3] / min–max の整形
- [x] 059. 二値の比率と信頼区間
- [x] 060. t検定・Wilcoxon（`t.test` / `wilcox.test` → `scipy.stats`）
- [x] 061. カイ二乗・Fisher（`chisq.test` / `fisher.test` → `scipy.stats`）
- [x] 062. 分散分析・線形回帰（`lm` / `aov` → `statsmodels`）
- [x] 063. 相関（`cor` / `cor.test` → `corr` / `scipy`）
- [x] 064. 生存時間の要約（`survfit` → `lifelines`）
- [x] 065. 集計値の丸め規則と SAS 一致（`round` 方式）

## 第6部　TFL 作成（表・図・出力）

Tables / Figures / Listings を Python で。出力フォーマット別。

- [ ] 066. 出力テーブルの考え方（rtables / gt vs Python の選択肢）
- [ ] 067. デモグラフィック表の組み立て
- [ ] 068. 有害事象（AE）集計表
- [ ] 069. シフトテーブル
- [ ] 070. リスティング（listing）
- [ ] 071. 見出し・スパンヘッダ・脚注
- [ ] 072. 数値の小数点・右揃え
- [ ] 073. `great_tables` で整形（gt 相当）
- [ ] 074. pandas `Styler` で整形
- [ ] 075. Word 出力（`officer` → `python-docx`）
- [ ] 076. RTF 出力（`r2rtf` / rtfreporter → 選択肢）
- [ ] 077. Excel 出力（`openxlsx` / `writexl` → `openpyxl` / `xlsxwriter`）
- [ ] 078. PDF 出力（選択肢）
- [ ] 079. 大きな表のページ分割
- [ ] 080. 図の基本（ggplot2 → matplotlib / plotnine）
- [ ] 081. ggplot 文法の対応（`aes` / `geom` → plotnine）
- [ ] 082. 群別・ファセット（`facet_wrap` → subplots / plotnine）
- [ ] 083. 複数図の配置（patchwork → subplots / gridspec）
- [ ] 084. 図の保存（`ggsave` → `savefig`）
- [ ] 085. KM 曲線・フォレストプロット

## 第7部　プログラミング・制御・関数

再利用可能なコードにするための足回り。

- [ ] 086. 関数定義（`function` → `def`）
- [ ] 087. 反復：apply 系と map（`sapply` / `purrr::map` → comprehension / `apply`）
- [ ] 088. 条件分岐（`if` / `ifelse` → `if` / `np.where`）
- [ ] 089. エラー処理（`tryCatch` → `try` / `except`）
- [ ] 090. ベクトル化の考え方（`for` を避ける）
- [ ] 091. 因子とラベル（factor levels / labels → `Categorical(ordered=True)`）
- [ ] 092. 日付・時刻の計算（lubridate → pandas offsets）
- [ ] 093. NULL / 欠損の安全な処理（`%||%` / `coalesce` → `or` / `fillna`）
- [ ] 094. パイプで関数をつなぐ設計（`|>` / `%>%` → `pipe` / method chain）
- [ ] 095. デバッグと検算のコツ

## 第8部　入出力・再現性・相互運用

データの出し入れと、R⇔Python の橋渡し。

- [ ] 096. CSV / 固定長 / 区切りの読み書き（`readr` → pandas / polars）
- [ ] 097. SAS / SPSS / Stata の読み書き（`haven` → `pyreadstat` / pandas）
- [ ] 098. データベース接続（`DBI` / `dbplyr` → `SQLAlchemy` / polars）
- [ ] 099. 乱数と再現性（`set.seed` → `np.random.default_rng`）
- [ ] 100. R⇔Python 相互運用（`reticulate` / `rpy2`）とパイプライン設計
