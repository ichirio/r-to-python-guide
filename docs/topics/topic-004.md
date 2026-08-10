# 004. パッケージ管理と import — `install.packages` / `library` → pip / conda / import

!!! abstract "この項目の R→Python 対応"
    - **R**: `install.packages()` で入れて `library()` で読み込む（名前空間は全部見える）
    - **Python（推奨）**: `pip` か `conda` で入れて `import` で読み込む。**慣用の別名**（`pd` / `np` / `pl`）を使う
    - **要注意**: R の `library(dplyr)` は関数を裸で使えるが、Python は `pd.read_csv` のように**接頭辞付き**が基本

---

## R ではこう書く

```r
# インストール（1回だけ）
install.packages("dplyr")

# 読み込み（セッションごと）
library(dplyr)
packageVersion("dplyr")

filter(mtcars, cyl == 4)   # 関数を裸で呼べる
```

出力（バージョン部）:

```text
[1] '1.2.1'
```

!!! note "読み込み方の使い分け"
    | 書き方 | 意味 | いつ使う |
    |---|---|---|
    | `library(pkg)` | 名前空間を丸ごと公開 | 対話・スクリプトの通常 |
    | `require(pkg)` | 同上だがエラーで止まらず `FALSE` を返す | 条件付き読み込み |
    | `pkg::func()` | 読み込まずに1関数だけ使う | 衝突回避・パッケージ開発 |

---

## Python ではこう書く

Python は「インストール（`pip`/`conda`）」と「読み込み（`import`）」が R と同じく別ステップ。データ分析では**別名を付けた import が慣習**です。

```python
# インストール（ターミナルで1回。どちらか一方でよい）
#   pip install pandas polars numpy
#   conda install -c conda-forge pandas polars numpy

# 読み込み（スクリプト冒頭。別名が事実上の標準）
import numpy as np
import pandas as pd
import polars as pl

print("pandas", pd.__version__, "polars", pl.__version__)
```

出力:

```text
pandas 2.2.3 polars 1.25.2
```

!!! tip "実務ではこれ"
    - **別名は固定**：`import pandas as pd` / `import numpy as np` / `import polars as pl`。読み手の期待どおりにする。
    - **一部だけ import** も可：`from pathlib import Path` のように、よく使う名前は裸で持ってくる。ただし乱用すると出所が不明になるので、データ操作は `pd.` を付けたままが読みやすい。

### 環境管理は R より重要

R は基本1つのライブラリ置き場を共有しますが、Python は**プロジェクトごとに環境を分ける**のが標準作法です。

| 目的 | R | Python |
|---|---|---|
| パッケージを入れる | `install.packages()` | `pip install` / `conda install` |
| 環境を分ける | renv | **venv** / **conda 環境** / uv |
| 依存を固定 | `renv.lock` | `requirements.txt` / `environment.yml` / `pyproject.toml` |

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| インストール | `install.packages("x")` | `pip install x` / `conda install x` |
| 読み込み | `library(x)` | `import x`（`as` で別名） |
| 関数1つだけ | `x::func()` | `from x import func` |
| バージョン確認 | `packageVersion("x")` | `x.__version__` |
| 入っているか | `requireNamespace("x")` | `import importlib.util; importlib.util.find_spec("x")` |
| 依存の固定 | `renv::snapshot()` | `pip freeze > requirements.txt` |

## つまずきポイント

!!! warning "R と Python の差"
    - **接頭辞**：`library(dplyr)` の後は `filter()` を裸で呼べるが、Python は `pd.read_csv()` のように**モジュール名を付ける**のが普通。`from pandas import *` は名前衝突の元なので避ける。
    - **pip と conda を混ぜる**と依存が壊れやすい。1つの環境ではどちらかに寄せる（このユーザ環境は Miniforge/conda 主軸）。
    - **`import` は実行文**：R の `library()` と同じく、スクリプトの**冒頭で毎回**書く。入れただけでは使えない。
    - **名前の綴り違い**：インストール名と import 名が違うことがある（例: `pip install scikit-learn` → `import sklearn`）。

## 関連項目

- [003. データフレーム入門](topic-003.md)
- [096. CSV / 固定長 / 区切りの読み書き](../roadmap.md)
