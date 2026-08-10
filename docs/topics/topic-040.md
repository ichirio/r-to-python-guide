# 040. 行の展開 — `separate_rows` / `unnest` → `explode`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `separate_rows()`（区切り文字列を複数行に）／`unnest()`（リスト列を展開）
    - **Python（推奨）**: **`str.split` → `explode()`**（1 セル複数値を複数行へ）
    - **要注意**: 展開後は index が重複する（pandas）。必要なら `reset_index(drop=True)`

`"HA,NAU"` のように1セルに複数値が詰まった項目を、1 行 1 値に開く（AE の複数コードなど）。

---

## R ではこう書く

```r
library(tidyr)

tibble(id = c("01","02"), aes = c("HA,NAU","")) |>
  separate_rows(aes, sep = ",")
```

出力:

```text
  id    aes
  01    "HA"
  01    "NAU"
  02    ""      ← 空文字は1行残る
```

!!! note "R の勘所"
    - `separate_rows(col, sep=)`：区切りで割って**縦に増やす**。1 セル→複数行。
    - `unnest(col)`：**リスト列**（各セルがベクトル）を展開。`unnest_longer`/`unnest_wider` もある。
    - 他の列は各行にコピーされる（id が複製）。

---

## Python ではこう書く

=== "pandas"

    ```python
    ae = pd.DataFrame({"id": ["01","02"], "aes": ["HA,NAU", ""]})

    out = ae.assign(aes=ae["aes"].str.split(",")).explode("aes")
    print(out.to_string(index=False))
    ```

    出力:

    ```text
    id aes
    01  HA
    01 NAU
    02
    ```

=== "polars"

    ```python
    import polars as pl
    ae = pl.DataFrame({"id": ["01","02"], "aes": ["HA,NAU", ""]})
    ae.with_columns(pl.col("aes").str.split(",")).explode("aes")
    ```

!!! tip "実務ではこれ"
    - **区切り文字列を縦に開く** → `str.split(sep)` で**リスト列**にしてから `explode(col)`。
    - **すでにリスト列**（`[a, b]` を持つ）→ そのまま `explode(col)`。
    - 展開後は行が増え index が重複するので、後段のために `reset_index(drop=True)`。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 区切り→複数行 | `separate_rows(col, sep=)` | `assign(...str.split()).explode(col)` | `str.split().explode()` |
| リスト列→複数行 | `unnest(col)` | `df.explode(col)` | `df.explode(col)` |
| リスト列→複数列 | `unnest_wider()` | `pd.DataFrame(col.tolist())` | `struct.unnest` |
| 展開後の連番 | — | `reset_index(drop=True)` | — |

## つまずきポイント

!!! warning "R と Python の差"
    - **index の重複**：pandas `explode` は元の行 index を複製する。位置ベースの後処理をするなら `reset_index(drop=True)`。
    - **空文字/欠損**：`"".split(",")` は `[""]` で 1 行残る（R の `separate_rows` も空1行）。欠損 `NaN` を split すると `explode` で NaN 1行。空行を落とすなら後で `query("aes != ''")` など。
    - **前後の空白**：`"HA, NAU"` を割ると `" NAU"` に空白が残る。`str.strip()`（→ [015](topic-015.md)）を挟む。
    - **複数列の同時展開**：pandas `explode(["a","b"])` は**長さが一致**していないとエラー。R の `unnest` は複数列を同時に開ける。

## 関連項目

- [014. 分割と連結](topic-014.md)
- [038. 列の分割（separate）](topic-038.md)
- [068. 有害事象（AE）集計表](../roadmap.md)
