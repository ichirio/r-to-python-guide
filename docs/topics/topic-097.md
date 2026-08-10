# 097. SAS / SPSS / Stata の読み書き — `haven` → `pyreadstat` / pandas

!!! abstract "この項目の R→Python 対応"
    - **R**: `haven::read_sas()` / `read_xpt()` / `read_sav()` / `read_dta()`、`write_xpt()`
    - **Python（推奨）**: **`pyreadstat`**（`read_sas7bdat` / `read_xport` / `read_sav` / `read_dta`、ラベル・メタ付き）／pandas `read_sas`
    - **要注意**: `pyreadstat` は **sas7bdat の書き込み非対応**（読み込みのみ）。**提出で使う XPT（v5）は読み書き可**

臨床データの SAS 連携。読み込みは pyreadstat が haven に最も近く、**変数ラベルや値ラベルもメタとして取れます**。

---

## R ではこう書く

```r
library(haven)
dm  <- read_sas("dm.sas7bdat")     # SAS データセット
dm2 <- read_xpt("dm.xpt")          # SAS transport（提出形式）
sav <- read_sav("data.sav")        # SPSS
dta <- read_dta("data.dta")        # Stata

write_xpt(dm, "dm.xpt", version = 5)   # XPT 書き出し
```

!!! note "R の勘所"
    - `read_sas` は sas7bdat、`read_xpt` は XPT（提出用の SAS transport）。
    - haven は変数ラベル（`label` 属性）・値ラベル（labelled）を保持。
    - 書き込みは XPT が `write_xpt`。sas7bdat の書き込みは基本しない。

---

## Python ではこう書く

=== "pyreadstat（推奨・メタ付き）"

    ```python
    # pip install pyreadstat
    import pyreadstat

    df, meta = pyreadstat.read_sas7bdat("dm.sas7bdat")   # 読み込み
    df, meta = pyreadstat.read_xport("dm.xpt")           # XPT
    df, meta = pyreadstat.read_sav("data.sav")           # SPSS
    df, meta = pyreadstat.read_dta("data.dta")           # Stata

    meta.column_labels      # 変数ラベル
    meta.value_labels       # 値ラベル（コード→意味）

    # XPT の書き出し（提出形式 v5）は可能
    pyreadstat.write_xport(df, "dm.xpt", table_name="DM")
    ```

    XPT round trip（検証済み）:

    ```text
    xport roundtrip: [{'SUBJID': '01', 'VAL': 1.5}, {'SUBJID': '02', 'VAL': 2.5}]
    ```

=== "pandas（読み込みのみ・軽量）"

    ```python
    import pandas as pd
    df = pd.read_sas("dm.sas7bdat")     # メタは限定的
    df = pd.read_sas("dm.xpt", format="xport")
    df = pd.read_spss("data.sav")       # 要 pyreadstat
    df = pd.read_stata("data.dta")
    ```

!!! tip "実務ではこれ"
    - **メタ（変数ラベル・値ラベル）が要る**なら **pyreadstat**（`meta.column_labels` / `value_labels`）。haven に最も近い。
    - **XPT（提出の SAS transport v5）は pyreadstat で読み書き可**。sas7bdat は**読み込みのみ**（書き込み不可）。
    - 手早く読むだけなら pandas `read_sas` / `read_stata`。ただしラベルの扱いは限定的。
    - 値ラベルを列に反映するには `apply_value_formats=True`（pyreadstat）や自前の `map`（[047](topic-047.md)）。

---

## 対応早見表

| やりたいこと | R（haven） | Python |
|---|---|---|
| sas7bdat 読み | `read_sas()` | `pyreadstat.read_sas7bdat()` / `pd.read_sas()` |
| XPT 読み | `read_xpt()` | `pyreadstat.read_xport()` |
| XPT 書き | `write_xpt()` | `pyreadstat.write_xport()` |
| SPSS 読み | `read_sav()` | `pyreadstat.read_sav()` / `pd.read_spss()` |
| Stata 読み | `read_dta()` | `pyreadstat.read_dta()` / `pd.read_stata()` |
| 変数ラベル | `attr(x, "label")` | `meta.column_labels` |
| 値ラベル | labelled | `meta.value_labels` |

## つまずきポイント

!!! warning "R と Python の差"
    - **sas7bdat は書けない**：pyreadstat は sas7bdat の**読み込み専用**。書き出しは XPT（`write_xport`）を使う。sas7bdat 生成が要るなら SAS 本体か R 側で。
    - **ラベルの保持**：pandas の `read_sas` はラベルを落としがち。ラベルが重要なら pyreadstat の `meta` を使う。
    - **文字コード**：日本語ラベルの SAS は encoding 指定が要ることがある（`encoding=` 引数）。
    - **XPT のバージョン・命名規則**：v5 は変数名 8 文字などの制約。提出要件に合わせる。
    - **欠損の特殊値**：SAS の特殊欠損（`.A`〜`.Z`）は単一の欠損に丸められる。区別が要るなら注意。

## 関連項目

- [096. CSV / 固定長 / 区切りの読み書き](topic-096.md)
- [047. ルックアップで値付与](topic-047.md)
- [091. 因子とラベル](topic-091.md)
