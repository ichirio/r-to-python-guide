# 092. 日付・時刻の計算 — lubridate → pandas offsets

!!! abstract "この項目の R→Python 対応"
    - **R**: `Date` / `POSIXct` と lubridate（`ymd`, `%m+%`, `interval`, `difftime`）
    - **Python（推奨）**: pandas **`to_datetime`**, **`DateOffset`** / `Timedelta`、`.dt` アクセサ
    - **要注意**: 「月の加算」は暦依存（月末問題）。日数差は `Timedelta`、暦月は `DateOffset(months=)`

臨床の来院日・経過日数・年齢計算。パースと差分の作法を揃えます。

---

## R ではこう書く

```r
library(lubridate)
d <- ymd("2024-01-15")

d %m+% months(1)                          # 1か月後（月末調整あり）
as.integer(ymd("2024-03-01") - ymd("2024-01-15"))  # 日数差 → 46
year(d); month(d); day(d)                 # 要素の取り出し
```

出力:

```text
[1] "2024-02-15"
[1] 46
```

!!! note "R の勘所"
    - `ymd`/`mdy`/`dmy` で文字列をパース。
    - `%m+% months(1)` は**月末を考慮**（1/31 + 1 month = 2/29 等）。単純な `+ months(1)` は月末で NA を出しうる。
    - 差は `difftime`（単位指定可）。`interval` で期間。

---

## Python ではこう書く

=== "pandas"

    ```python
    import pandas as pd
    d = pd.Timestamp("2024-01-15")

    d + pd.DateOffset(months=1)                 # 1か月後（暦月）
    (pd.Timestamp("2024-03-01") - pd.Timestamp("2024-01-15")).days   # 日数差 → 46
    d.year, d.month, d.day                       # 要素

    # 列に対して
    s = pd.to_datetime(df["visitdt"])            # パース
    s.dt.year                                    # .dt アクセサ
    (s2 - s1).dt.days                            # 日数差の列
    ```

    出力:

    ```text
    2024-02-15
    46
    ```

!!! tip "実務ではこれ"
    - **パース** → `pd.to_datetime(col, format=)`（曖昧な形式は `format=` を明示、`errors="coerce"` で不正を NaT に）。
    - **日数差** → 引き算 → `.dt.days`（`Timedelta`）。**暦月・暦年の加算** → `DateOffset(months=, years=)`。
    - **要素取り出し・曜日・週** → `.dt.year` / `.dt.month` / `.dt.dayofweek` など `.dt` アクセサ。
    - 年齢は「参照日 − 生年月日」を年に換算（単純割りは閏で誤差。厳密には月日比較）。

---

## 対応早見表

| やりたいこと | R（lubridate） | Python（pandas） |
|---|---|---|
| パース | `ymd(x)` | `pd.to_datetime(x)` |
| 年/月/日 | `year()/month()/day()` | `.dt.year/.month/.day` |
| 曜日 | `wday()` | `.dt.dayofweek` |
| 日数差 | `d2 - d1`（difftime） | `(d2 - d1).dt.days` |
| 月の加算 | `%m+% months(n)` | `+ pd.DateOffset(months=n)` |
| 期間の切り出し | `floor_date()` | `.dt.to_period("M")` |
| 書式化 | `format(d, "%Y-%m-%d")` | `.dt.strftime("%Y-%m-%d")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **月末問題**：`DateOffset(months=1)` と lubridate の `%m+%` はどちらも暦月加算だが、月末の丸め挙動が細部で違うことがある（1/31 + 1 month）。境界日で検算。
    - **`Timedelta` vs `DateOffset`**：固定長（日・時間）は `Timedelta`、暦依存（月・年）は `DateOffset`。`Timedelta(days=30)` は「1か月」ではない。
    - **欠損日付**：pandas の欠損日時は `NaT`（NaN の日時版）。`errors="coerce"` で不正日付を NaT に。
    - **タイムゾーン**：`tz_localize`/`tz_convert`。臨床の日付は tz なしが多いが、日時なら注意。
    - **文字列書式**：出力用は `.dt.strftime`。ISO 以外の入力は `format=` を明示しないと誤解釈。

## 関連項目

- [028. 型変換](topic-028.md)
- [011. 文字列の抽出・長さ](topic-011.md)
- [070. リスティング](topic-070.md)
