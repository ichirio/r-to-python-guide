# R→Python 対応ガイド（統計・TFL実務）

R（特に **Tidyverse** でのデータ加工、**TFL**〈Tables / Figures / Listings〉作成）を、
**Python（pandas / polars）でどう書くか** を、**R を起点に**対比する全100項目のリファレンスです。

- 対象読者: R を使う統計プログラマ（臨床開発・データ解析の実務者）
- 進め方: 1項目1テーマ。「**R ではこう書く**（メジャーな方法を使い分けとともに）→ **Python ではこれが良い** → 早見表 → つまずきポイント」の固定フォーマット
- 公開: GitHub Pages（`docs/` を MkDocs Material でサイト化）

## サイトをローカルで見る

```bash
pip install -r requirements.txt
mkdocs serve
# ブラウザで http://127.0.0.1:8000 を開く
```

## 構成

```
docs/
  index.md            … ガイドの入口・読み方
  roadmap.md          … 全100項目の一覧（カテゴリ別）
  topics/
    topic-001.md      … 001 RとPythonの考え方の違い
    topic-002.md      … 002 パイプとメソッドチェーン
    ...
CONTRIBUTING.md       … 項目の追加・執筆ルール
TEMPLATE.md           … 新規項目のひな形（コピーして使う）
```

## 項目を増やすとき（メンテ方針）

1. `TEMPLATE.md` をコピーして `docs/topics/topic-NNN.md` を作る（**番号は末尾に追加。既存番号は振り直さない**）。
2. `mkdocs.yml` の `nav:` に 1 行足す（表示順は nav で制御）。
3. `docs/roadmap.md` の該当行のチェックを更新する。
4. コード出力は **必ず実行して検証**（R は `Rscript`、Python は `python`）。詳細は `CONTRIBUTING.md`。

番号は「安定 ID」です。カテゴリの並びを変えたくなっても、ファイル名の番号は動かさず `nav` の順序だけ変えます。
