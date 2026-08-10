# 098. データベース接続 — `DBI` / `dbplyr` → `SQLAlchemy` / polars

!!! abstract "この項目の R→Python 対応"
    - **R**: `DBI`（接続・クエリ）＋ `dbplyr`（dplyr 構文を SQL に翻訳）
    - **Python（推奨）**: **SQLAlchemy**（接続）＋ pandas **`read_sql`**／polars `read_database`
    - **要注意**: dbplyr のような「dplyr→SQL 自動翻訳」の定番は薄い。素の SQL か SQLAlchemy を書く

DB からの読み込みと書き込み。R の dbplyr の「遅延評価で SQL 生成」に慣れていると発想の違いがあります。

---

## R ではこう書く

```r
library(DBI); library(dbplyr)
con <- dbConnect(RSQLite::SQLite(), "study.db")

# 素の SQL
dbGetQuery(con, "SELECT * FROM dm WHERE arm = 'A'")

# dplyr 構文が SQL に翻訳される（遅延評価）
tbl(con, "dm") |>
  filter(arm == "A") |>
  group_by(sex) |>
  summarise(n = n()) |>
  collect()                 # ここで初めて実行

dbDisconnect(con)
```

!!! note "R の勘所"
    - `DBI` が接続・実行の共通 API、`dbplyr` が dplyr を SQL に翻訳。
    - `tbl(con, ...)` は**遅延**（`collect()` で実体化）。大きな表を DB 側で絞れる。

---

## Python ではこう書く

=== "SQLAlchemy + pandas"

    ```python
    # pip install sqlalchemy
    from sqlalchemy import create_engine
    import pandas as pd

    engine = create_engine("sqlite:///study.db")

    # SQL を書いて読む
    df = pd.read_sql("SELECT * FROM dm WHERE arm = 'A'", engine)

    # パラメータ化（SQL インジェクション対策）
    from sqlalchemy import text
    df = pd.read_sql(text("SELECT * FROM dm WHERE arm = :a"), engine,
                     params={"a": "A"})

    # 書き込み
    df.to_sql("dm_out", engine, if_exists="replace", index=False)
    ```

=== "polars"

    ```python
    import polars as pl
    df = pl.read_database("SELECT * FROM dm WHERE arm = 'A'", connection=engine)
    ```

!!! tip "実務ではこれ"
    - **接続は SQLAlchemy の `create_engine("dialect://...")`**（sqlite/postgresql/mysql/oracle…）。ドライバは別途 pip。
    - **読み込みは `pd.read_sql(sql, engine)`**。大きな表は **SQL の WHERE/GROUP BY で DB 側に絞らせる**（dbplyr の発想を SQL で手書き）。
    - **パラメータは `text()` + `params=`**。文字列連結で値を埋め込まない（インジェクション対策）。
    - **書き込みは `to_sql(if_exists=)`**（`replace`/`append`/`fail`）。
    - dbplyr 風の「Python 構文→SQL 翻訳」が欲しいなら ibis というライブラリもあるが、まずは素の SQL が確実。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 接続 | `dbConnect()` | `create_engine()` |
| SQL 実行 | `dbGetQuery()` | `pd.read_sql()` / `pl.read_database()` |
| dplyr→SQL | `tbl()` + dbplyr | 素の SQL（or ibis） |
| 遅延→実体化 | `collect()` | （read_sql は即時） |
| 書き込み | `dbWriteTable()` | `df.to_sql()` |
| パラメータ | `dbBind()` | `text()` + `params=` |
| 切断 | `dbDisconnect()` | `engine.dispose()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **dbplyr の自動翻訳がない**：pandas は基本「SQL を書いて結果を DataFrame に」。DB 側で絞る処理は SQL で手書きする。
    - **ドライバ**：`create_engine` の方言に対応するドライバ（`psycopg2`/`pymysql`/`cx_Oracle` 等）を別途 pip。接続文字列の形式も方言ごとに違う。
    - **インジェクション**：値は必ずパラメータ化（`text()` + `params`）。f-string で SQL に値を埋め込まない。
    - **大きな結果**：全件を DataFrame に載せるとメモリを食う。`chunksize=` で分割読み、または SQL で集計してから取得。
    - **型・欠損**：DB の NULL は NaN/None に。日付型・数値精度は方言依存で微差が出ることがある。

## 関連項目

- [096. CSV / 固定長 / 区切りの読み書き](topic-096.md)
- [041. 内部結合](topic-041.md)
- [010. グループ集約](topic-010.md)
