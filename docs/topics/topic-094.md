# 094. パイプで関数をつなぐ設計 — `|>` / `%>%` → `pipe` / method chain

!!! abstract "この項目の R→Python 対応"
    - **R**: `|>` / `%>%` で「データ → 関数 → 関数」を一直線に書く
    - **Python（推奨）**: **メソッドチェーン**（`.query().assign()...`）＋任意関数は **`.pipe()`**
    - **要注意**: 自作の変換関数を挟むなら `.pipe(func)`。チェーンは全体を丸括弧で囲む（[002](topic-002.md)）

[002](topic-002.md) の続編。「読める処理パイプライン」を Python で組む設計指針です。

---

## R ではこう書く

```r
library(dplyr)

result <- raw |>
  filter(!is.na(age)) |>
  mutate(age_grp = if_else(age >= 50, ">=50", "<50")) |>
  my_custom_step() |>              # 自作関数もパイプに乗る
  group_by(arm, age_grp) |>
  summarise(n = n())
```

!!! note "R の勘所"
    - パイプは値を**次の関数の第1引数**へ流す。dplyr でも自作関数でも一直線。
    - 途中に自作関数を挟んでも自然（`|> my_step()`）。
    - 中間変数を作らずに読める。

---

## Python ではこう書く

=== "メソッドチェーン + pipe"

    ```python
    result = (
        raw
        .dropna(subset=["age"])
        .assign(age_grp=lambda d: np.where(d["age"] >= 50, ">=50", "<50"))
        .pipe(my_custom_step)                     # 自作関数は pipe で
        .groupby(["arm", "age_grp"]).size()
        .reset_index(name="n")
    )
    ```

    - **組み込みメソッド**はそのまま `.` でつなぐ。
    - **自作の変換関数**（DataFrame を受け取り DataFrame を返す）は `.pipe(func)`。R の `|> func()` に対応。
    - `assign` の中で直前の列を参照するには `lambda d: ...`。

=== "polars"

    ```python
    import polars as pl
    (raw
        .filter(pl.col("age").is_not_null())
        .with_columns(pl.when(pl.col("age") >= 50).then(pl.lit(">=50"))
                        .otherwise(pl.lit("<50")).alias("age_grp"))
        .group_by(["arm","age_grp"]).len())
    ```

!!! tip "実務ではこれ"
    - **自作関数は「DataFrame in → DataFrame out」**に設計すると `.pipe()` で自然につながる:
      ```python
      def flag_outliers(df, col, k=3):
          z = (df[col] - df[col].mean()) / df[col].std()
          return df.assign(is_outlier=z.abs() > k)
      df.pipe(flag_outliers, "age")
      ```
    - **チェーンは全体を丸括弧**で囲んで縦に並べる（[002](topic-002.md)）。1 行 1 ステップで読める。
    - 長すぎるチェーンは意味の切れ目で中間変数に。可読性と再利用性のバランス。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| つなぐ | <code>x &#124;&gt; f() &#124;&gt; g()</code> | `x.f().g()` |
| 自作関数を挟む | <code>x &#124;&gt; my_fn()</code> | `x.pipe(my_fn)` |
| 引数付き自作関数 | <code>x &#124;&gt; my_fn(k = 3)</code> | `x.pipe(my_fn, k=3)` |
| 途中で列参照 | `mutate(b = a + 1)` | `.assign(b=lambda d: d["a"] + 1)` |
| 複数行 | 改行して <code>&#124;&gt;</code> | 全体を `()` で囲む |

## つまずきポイント

!!! warning "R と Python の差"
    - **メソッドがない操作**：pandas に該当メソッドがない処理は `.pipe(func)` で。R のパイプは「関数」を流すので自作も同列だが、Python のチェーンは「メソッド」なので pipe が要る。
    - **丸括弧で囲む**：改行チェーンは全体を `()` で。囲まないと構文エラー（R は行末 `|>` でOK）。
    - **`assign` の遅延参照**：作りたての列を参照するなら `lambda d:`。R の `mutate` の即時参照とは書き方が違う（[009](topic-009.md)）。
    - **副作用のある関数**：`pipe` に渡す関数は原則「入力を変えず新しい DF を返す」。元を書き換えると追跡困難。

## 関連項目

- [002. パイプとメソッドチェーン](topic-002.md)
- [009. 列作成・変更（mutate）](topic-009.md)
- [086. 関数定義](topic-086.md)
