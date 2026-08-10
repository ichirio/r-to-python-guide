# 048. 結合キーの検証 — 多対多・重複チェック

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr 1.1+ の `relationship = "one-to-one"` 等／結合前に `count()` で重複確認
    - **Python（推奨）**: pandas **`merge(validate="one_to_one" / "one_to_many" / ...)`**
    - **要注意**: 重複キーは**行数を掛け算で爆発**させる。結合の事故で最多。検証を習慣に

「片方のキーが重複していて、左結合したら行が増えた」は解析事故の常連。**結合の前後で行数を確認**し、想定と違えば検証で止める。

（例: `dm2`=01,02,03 に、`subjid=02` が2行ある `dup` を左結合）

---

## R ではこう書く

```r
library(dplyr)
dup <- tibble(subjid = c("02","02"), val = c(5,50))   # 02 が重複

# 何も指定しないと 02 が2行に増える（警告は出る）
dm2 |> left_join(dup, by = "subjid")

# 想定を宣言して守る（dplyr 1.1+）
dm2 |> left_join(dup, by = "subjid", relationship = "one-to-one")
# → Error: 重複により one-to-one を満たさない
```

出力（無検証の左結合）:

```text
  subjid arm     val
  01     A        NA
  02     A         5
  02     A        50   ← 02 が2行に増殖
  03     B        NA
```

!!! note "R の勘所"
    - dplyr 1.1 以降、`by` に一致が複数あると**警告**が出る。`relationship=` で `"one-to-one"`/`"many-to-one"` 等を宣言できる。
    - 事前チェックは `dup |> count(subjid) |> filter(n > 1)`。
    - `multiple = "first"` などで重複時の挙動も指定可。

---

## Python ではこう書く

=== "pandas"

    ```python
    dup = pd.DataFrame({"subjid":["02","02"], "val":[5,50]})

    # 無検証：02 が2行に増える
    dm2.merge(dup, on="subjid", how="left")

    # 想定を宣言して守る
    try:
        dm2.merge(dup, on="subjid", how="left", validate="one_to_one")
    except Exception as e:
        print(type(e).__name__)   # MergeError
    ```

    出力:

    ```text
    # 無検証 → 02 が2行に増殖（01, 02, 02, 03）
    MergeError
    ```

    事前チェック:

    ```python
    dup["subjid"].duplicated().any()          # True なら重複あり
    dup.groupby("subjid").size().gt(1)        # どのキーが重複か
    ```

=== "polars"

    ```python
    import polars as pl
    # 重複確認
    dupp.filter(pl.col("subjid").is_duplicated())
    # join は validate 引数（polars 1.x）
    dm2p.join(dupp, on="subjid", how="left", validate="1:1")
    ```

!!! tip "実務ではこれ"
    - **結合には `validate=` を付ける**のを既定に。`"one_to_one"` / `"one_to_many"` / `"many_to_one"` / `"many_to_many"`。想定と違えば例外で気づける。
    - **結合前後で `len(df)` を照合**。左結合で行数が変わったら右キーの重複を疑う。
    - **右を先に一意化**：ルックアップ表なら `drop_duplicates(subset=key)` してから結合。

---

## 対応早見表

| やりたいこと | R（dplyr 1.1+） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 1対1を保証 | `relationship="one-to-one"` | `validate="one_to_one"` | `validate="1:1"` |
| 多対一を保証 | `relationship="many-to-one"` | `validate="many_to_one"` | `validate="m:1"` |
| 重複キー確認 | <code>count(k) &#124;&gt; filter(n&gt;1)</code> | `df[k].duplicated()` | `col.is_duplicated()` |
| 重複時の挙動 | `multiple=` | 事前に一意化 | 事前に一意化 |

## つまずきポイント

!!! warning "R と Python の差"
    - **無言の行増殖**：pandas `merge` は既定で検証しない。重複キーで黙って行が増える。**`validate=` を付ける**まで気づけないことがある。
    - **`validate` の名前**：pandas はアンダースコア（`"one_to_many"`）、polars はコロン（`"1:m"`）。
    - **多対多**：本当に必要な場合以外は避ける。件数が `n×m` に膨らむ。
    - **キーの型・空白**：`"01 "`（末尾空白）と `"01"` は別キー。結合前に `str.strip()`（→ [015](topic-015.md)）で正規化。

## 関連項目

- [041. 内部結合](topic-041.md)
- [042. 左・外部結合](topic-042.md)
- [023. 重複の除去](topic-023.md)
