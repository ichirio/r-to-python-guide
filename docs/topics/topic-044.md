# 044. 縦結合・横結合 — `bind_rows` / `bind_cols` → `concat`

!!! abstract "この項目の R→Python 対応"
    - **R**: dplyr `bind_rows()`（縦に積む、列名で揃える）／`bind_cols()`（横に並べる）
    - **Python（推奨）**: pandas **`concat([...], axis=0)`**（縦）／**`axis=1`**（横）；polars `concat` / `hstack`
    - **要注意**: `bind_rows` は**列名で自動整列**。pandas `concat` も列名で揃うが、index を `ignore_index=True` で振り直すのが定石

キーで結び付けるのではなく、**同じ構造のデータを積む/並べる**操作。

---

## R ではこう書く

```r
library(dplyr)
a <- tibble(subjid = c("01","02"), x = c(1,2))
b <- tibble(subjid = c("03","04"), x = c(3,4))

bind_rows(a, b)                       # 縦に積む
bind_cols(a, tibble(y = c(9,8)))      # 横に並べる
bind_rows(list(t1 = a, t2 = b), .id = "src")  # 由来ラベル付き
```

出力（`bind_rows`）:

```text
  subjid     x
  01         1
  02         2
  03         3
  04         4
```

!!! note "R の勘所"
    - `bind_rows`：**列名で対応**。片方にない列は NA 埋め。型が違うとエラー/昇格。
    - `bind_cols`：位置で横並び。**行数が一致**していること。
    - `.id=` で「どのデータ由来か」の列を付けられる。

---

## Python ではこう書く

=== "pandas"

    ```python
    a = pd.DataFrame({"subjid":["01","02"], "x":[1,2]})
    b = pd.DataFrame({"subjid":["03","04"], "x":[3,4]})

    pd.concat([a, b], ignore_index=True)          # 縦（列名で整列）
    pd.concat([a, pd.DataFrame({"y":[9,8]})], axis=1)  # 横

    # 由来ラベル付き（.id 相当）
    pd.concat([a, b], keys=["t1","t2"], names=["src"]).reset_index(level=0)
    ```

    出力（縦）:

    ```text
    subjid  x
        01  1
        02  2
        03  3
        04  4
    ```

=== "polars"

    ```python
    import polars as pl
    pl.concat([ap, bp], how="vertical")     # 縦
    ap.hstack(pl.DataFrame({"y":[9,8]}))    # 横
    ```

!!! tip "実務ではこれ"
    - **縦積み** → `pd.concat([...], ignore_index=True)`。`ignore_index` で index の重複を防ぐ。
    - **横並び** → `pd.concat([...], axis=1)`。ただし**index が揃っていないと NaN が湧く**ので、位置で並べるなら両者を `reset_index(drop=True)` してから。
    - 由来ラベルは `keys=` か、事前に各データへ `assign(src=...)`。

---

## 対応早見表

| やりたいこと | R（dplyr） | Python（pandas） | Python（polars） |
|---|---|---|---|
| 縦に積む | `bind_rows(a, b)` | `pd.concat([a,b], ignore_index=True)` | `pl.concat([a,b])` |
| 横に並べる | `bind_cols(a, b)` | `pd.concat([a,b], axis=1)` | `a.hstack(b)` |
| 由来ラベル | `.id="src"` | `keys=[...], names=["src"]` | 事前に `with_columns` |
| 列が違っても縦積み | 自動 NA 埋め | 自動 NaN 埋め | `how="diagonal"` |

## つまずきポイント

!!! warning "R と Python の差"
    - **横並びの index 罠**：pandas `concat(axis=1)` は**index を突き合わせて**並べる。行順で並べたいだけなら両方 `reset_index(drop=True)` を。R の `bind_cols` は純粋に位置で並べる。
    - **列の不一致**：`bind_rows` も `concat` も列名が違えば NaN 埋め。polars の素の `concat` は**列が完全一致**を要求（違うなら `how="diagonal"`）。
    - **型の昇格**：整数＋欠損で float 化、文字＋数値で object 化。積んだ後の dtype を確認。
    - **index の重複**：縦積みで `ignore_index=True` を忘れると index が重複し、後段の `loc` で事故る。

## 関連項目

- [041. 内部結合](topic-041.md)
- [050. ロング／ワイドを行き来する実務パターン](topic-050.md)
