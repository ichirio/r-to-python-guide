# 095. デバッグと検算のコツ

!!! abstract "この項目の R→Python 対応"
    - **R**: `str()` / `glimpse()` / `summary()`、`browser()`、`stopifnot()`
    - **Python（推奨）**: `df.info()` / `df.head()` / `df.dtypes`、`breakpoint()`、`assert`
    - **要注意**: パイプライン検算は「行数・キー一意性・欠損数・合計」を各段で確認。移植時は R と数値照合

R から移植したコードが「同じ数字を出すか」を確かめる作法。TFL は数値一致が命です。

---

## データの中身を見る

| R | Python |
|---|---|
| `str(df)` / `glimpse(df)` | `df.info()` |
| `head(df)` / `tail(df)` | `df.head()` / `df.tail()` |
| `summary(df)` | `df.describe(include="all")` |
| `dim(df)` | `df.shape` |
| `sapply(df, class)` | `df.dtypes` |
| `table(df$x, useNA="ifany")` | `df["x"].value_counts(dropna=False)` |

---

## パイプラインの検算ポイント

各段で「壊れていないか」を機械的に確認します。

```python
# 行数（結合で増減していないか → [048]）
assert len(out) == len(raw), f"row count changed: {len(raw)} -> {len(out)}"

# キーの一意性
assert not out["subjid"].duplicated().any(), "subjid duplicated"

# 欠損数（想定外に湧いていないか → [056]）
print(out.isna().sum())

# 合計・件数の突き合わせ（集計の検算）
assert out["n"].sum() == raw.shape[0]
```

R でも同様に `stopifnot(nrow(out) == nrow(raw))`。

---

## R との数値照合（移植時）

```python
# R 側の出力を CSV に出し、Python 側と突き合わせる
r_out = pd.read_csv("r_result.csv")
py_out = build_table(raw)

# 数値列の一致（浮動小数は許容誤差つき）
pd.testing.assert_frame_equal(
    py_out.reset_index(drop=True),
    r_out.reset_index(drop=True),
    check_dtype=False, atol=1e-6)
```

!!! tip "実務ではこれ"
    - **各段で `shape` / 一意性 / 欠損 / 合計を確認**。特に結合（[048](topic-048.md)）と集計の前後。
    - **R と Python を数セル突き合わせる**：丸め方式（[018](topic-018.md), [065](topic-065.md)）、欠損の既定（[005](topic-005.md)）、Welch/等分散（[060](topic-060.md)）など、既定差が出やすい所を重点的に。
    - **対話デバッグ**：`breakpoint()`（Python 3.7+）でその場に入る。R の `browser()` に対応。Jupyter なら `%debug`。
    - **`assert` で前提を固定**：想定（行数・一意性）をコードに書くと、壊れた瞬間に気づける。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| 構造確認 | `str()` / `glimpse()` | `df.info()` / `df.dtypes` |
| 先頭表示 | `head()` | `df.head()` |
| 前提を検証 | `stopifnot()` | `assert` |
| ブレークポイント | `browser()` | `breakpoint()` |
| 事後デバッグ | `traceback()` | `%debug`（Jupyter） |
| DF 一致検証 | `all.equal()` | `pd.testing.assert_frame_equal()` |

## つまずきポイント

!!! warning "R と Python の差"
    - **既定差が数値差になる**：丸め・欠損・検定の既定が R と違うと、移植で微妙にズレる。**照合は既定差の出る所を狙う**。
    - **浮動小数の一致**：完全一致（`==`）でなく許容誤差（`atol`/`rtol`）で比較。`assert_frame_equal` の `check_exact=False`。
    - **index のズレ**：pandas は行 index が残るので、比較前に `reset_index(drop=True)`。
    - **`assert` は本番で無効化され得る**：`python -O` は assert を飛ばす。データ検証を本番でも効かせたいなら明示的に例外を投げる。

## 関連項目

- [048. 結合キーの検証](topic-048.md)
- [056. 欠損数の集計](topic-056.md)
- [065. 集計値の丸め規則と SAS 一致](topic-065.md)
