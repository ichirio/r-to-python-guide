# 047. ルックアップで値付与 — `left_join` lookup → `map`

!!! abstract "この項目の R→Python 対応"
    - **R**: コード表を `left_join()` で貼る（`recode` より保守しやすい）
    - **Python（推奨）**: 1 対 1 の対応なら **`Series.map(dict)`**、多列を貼るなら **`merge`**
    - **要注意**: `map` は未対応キーを NaN に。対応表の網羅性を確認する

コード → ラベル（`M`→`Male`、施設 ID→施設名）のような**参照表による値の付与**。

（例: `dat`=code M,F,M / `codes`=M→Male, F→Female）

---

## R ではこう書く

```r
library(dplyr)
codes <- tibble(code = c("M","F"), label = c("Male","Female"))
dat   <- tibble(id = c("01","02","03"), code = c("M","F","M"))

dat |> left_join(codes, by = "code")
```

出力:

```text
  id    code  label
  01    M     Male
  02    F     Female
  03    M     Male
```

!!! note "R の勘所"
    - 対応表を**データとして持ち**、`left_join` で貼るのが保守しやすい（`recode` のハードコードより）。
    - 複数の付随列（ラベル＋順序＋群）も一度に付く。
    - 未対応コードは NA。突合チェックには `anti_join`（→ [043](topic-043.md)）。

---

## Python ではこう書く

=== "pandas"

    ```python
    # 1 対 1 の対応 → map（辞書）
    codes = {"M": "Male", "F": "Female"}
    dat = pd.DataFrame({"id":["01","02","03"], "code":["M","F","M"]})
    dat["label"] = dat["code"].map(codes)

    # 多列を貼る → merge（対応表が DataFrame）
    codes_df = pd.DataFrame({"code":["M","F"], "label":["Male","Female"]})
    dat.merge(codes_df, on="code", how="left")
    ```

    出力（`map`）:

    ```text
    id code   label
    01    M    Male
    02    F  Female
    03    M    Male
    ```

=== "polars"

    ```python
    import polars as pl
    codes = {"M": "Male", "F": "Female"}
    datp.with_columns(pl.col("code").replace(codes).alias("label"))
    # もしくは対応表 DataFrame を join
    ```

!!! tip "実務ではこれ"
    - **単一列の言い換え** → `map(dict)`（→ [027](topic-027.md)）。辞書は外部 CSV/辞書から読むと保守が楽。
    - **ラベル＋順序＋群など複数列を貼る** → 対応表を DataFrame にして `merge(how="left")`。
    - **未対応コードの検出**：`dat[dat["code"].map(codes).isna()]` で漏れを洗う。無言の NaN 化を放置しない。

---

## 対応早見表

| やりたいこと | R | Python（pandas） | Python（polars） |
|---|---|---|---|
| 1 対 1 の言い換え | `left_join(codes)` / `recode` | `s.map(dict)` | `col.replace(dict)` |
| 多列を貼る | `left_join(codes_df)` | `merge(codes_df, how="left")` | `join(codes_df, how="left")` |
| 未対応を検出 | `anti_join` | `s.map(d).isna()` | `is_null()` |
| 既定値を付ける | `coalesce(label, "他")` | `s.map(d).fillna("他")` | `replace(..., default=)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **未対応の扱い**：`map` は未対応キーを NaN に（`replace` は残す、→ [027](topic-027.md)）。対応表が全コードを網羅しているか確認。
    - **キー型の一致**：辞書キーが `"1"`（文字）で列が `1`（数値）だと当たらない。型を揃える。
    - **重複する対応表**：`merge` で使う対応表に同じ `code` が複数あると行が増える。対応表は `drop_duplicates` してから（→ [048](topic-048.md)）。
    - **順序・因子**：ラベルに表示順があるなら Categorical にして水準順を持たせる（→ [091](../roadmap.md)）。

## 関連項目

- [027. 値の対応付け・recode](topic-027.md)
- [041. 内部結合](topic-041.md)
- [042. 左・外部結合](topic-042.md)
