# 039. 列の結合 — `unite` → `str.cat` / `agg`

!!! abstract "この項目の R→Python 対応"
    - **R**: tidyr `unite()`（複数列を1列に、区切り付き）
    - **Python（推奨）**: 2 列は **`Series.str.cat(sep=)`**、多数列は **`df[cols].agg("-".join, axis=1)`**
    - **要注意**: 数値列が混ざると Python はまず文字列化が要る（R は自動）。欠損の扱いも要指定

`grp="A", num="01"` を `id="A-01"` にまとめる、[038](topic-038.md) の逆操作。

---

## R ではこう書く

```r
library(tidyr)

tibble(grp = c("A","B"), num = c("01","02")) |>
  unite("id", grp, num, sep = "-")
```

出力:

```text
  id
  A-01
  B-02
```

!!! note "R の勘所"
    - `unite(新列, 列..., sep=)`：まとめる列を並べ、区切りを指定。
    - `remove = FALSE` で元の列も残す。
    - 数値列も自動で文字列化して結合。NA は `"NA"` になる（`na.rm=` で除去）。

---

## Python ではこう書く

=== "pandas"

    ```python
    gd = pd.DataFrame({"grp": ["A","B"], "num": ["01","02"]})

    # 2 列：str.cat
    gd["id"] = gd["grp"].str.cat(gd["num"], sep="-")

    # 多数列：agg で join（全列を文字列化してから）
    cols = ["grp", "num"]
    gd["id"] = gd[cols].astype(str).agg("-".join, axis=1)
    ```

    出力（`id`）:

    ```text
    ['A-01', 'B-02']
    ```

=== "polars"

    ```python
    import polars as pl
    df = pl.DataFrame({"grp": ["A","B"], "num": ["01","02"]})
    df.with_columns(
        pl.concat_str(["grp", "num"], separator="-").alias("id")
    )
    ```

!!! tip "実務ではこれ"
    - **2 列** → `str.cat(other, sep=)`（NA 制御は `na_rep=`）。
    - **3 列以上** → `df[cols].astype(str).agg(sep.join, axis=1)`。数値混在なら `astype(str)` を必ず挟む。
    - polars は `pl.concat_str([...], separator=)` が最も素直（→ [006](topic-006.md)）。

---

## 対応早見表

| やりたいこと | R（tidyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 2 列を結合 | `unite("id", a, b, sep="-")` | `a.str.cat(b, sep="-")` | `pl.concat_str([a,b], separator="-")` |
| 多数列を結合 | `unite("id", a, b, c)` | `df[cols].astype(str).agg("-".join, axis=1)` | `pl.concat_str(cols, ...)` |
| 元列も残す | `remove=FALSE` | 代入するだけ（元は残る） | with_columns（元は残る） |
| 欠損を除く | `na.rm=TRUE` | `str.cat(na_rep="")` 等 | 既定で null 伝播 |

## つまずきポイント

!!! warning "R と Python の差"
    - **自動文字列化しない**：R の `unite` は数値列も勝手に文字列化。pandas の `agg(join)` は**文字列でないと TypeError**。`astype(str)` を先に。
    - **欠損の見え方**：R は NA を `"NA"` に。pandas `str.cat` は欠損行を既定で欠損に（`na_rep=` で置換）。polars は null を伝播。TFL で空欄にするか `NA` にするか要件で決める。
    - **区切りの位置**：欠損を除いて結合すると区切りが余ることがある（`"A--02"` など）。除去仕様なら詰め処理を。
    - **速度**：`agg(join, axis=1)` は行方向で遅め。2 列なら `str.cat` か `+` の方が速い。

## 関連項目

- [006. 文字列結合](topic-006.md)
- [038. 列の分割（separate）](topic-038.md)
- [014. 分割と連結](topic-014.md)
