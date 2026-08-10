# 058. 中央値 [Q1, Q3] / min–max の整形

!!! abstract "この項目の R→Python 対応"
    - **R**: `median()` / `quantile(.25/.75)` / `min` / `max` → `sprintf` で `median (Q1, Q3)` や `min, max`
    - **Python（推奨）**: `median()` / `quantile([.25,.75])` → **f-string** で整形
    - **要注意**: 分位点の定義（type 7）は R と pandas で一致。丸め方式は [018](topic-018.md) に注意

デモグラ表の「中央値 [Q1, Q3]」「最小–最大」行。連続変数を非パラメトリックに要約するときの定番表記。

（age = 45,52,60,48,55,38,61,50,42,44 → median 49.0, Q1 44.25, Q3 54.25）

---

## R ではこう書く

```r
a <- dm3$age
sprintf("%.1f (%.1f, %.1f)", median(a), quantile(a, .25), quantile(a, .75))
sprintf("%d, %d", min(a), max(a))
```

出力:

```text
49.0 (44.2, 54.2)
38, 61
```

!!! note "R の勘所"
    - `quantile(a, .25)` の既定は type 7（線形補間）。
    - 表記の括弧・区切り（`[Q1, Q3]` か `(Q1–Q3)` か）は各社の TFL 規約に合わせる。
    - 群ごとは `group_by |> summarise` で分位点を出してから整形。

---

## Python ではこう書く

=== "pandas"

    ```python
    a = dm3["age"]
    q1, q3 = a.quantile(.25), a.quantile(.75)
    print(f"{a.median():.1f} ({q1:.1f}, {q3:.1f})")
    print(f"{a.min()}, {a.max()}")

    # 群ごと
    def med_iqr(s):
        return f"{s.median():.1f} ({s.quantile(.25):.1f}, {s.quantile(.75):.1f})"
    dm3.groupby("arm")["age"].apply(med_iqr)
    ```

    出力:

    ```text
    49.0 (44.2, 54.2)
    38, 61
    ```

=== "polars"

    ```python
    import polars as pl
    (dm3p.group_by("arm").agg(
        pl.col("age").median().alias("med"),
        pl.col("age").quantile(.25).alias("q1"),
        pl.col("age").quantile(.75).alias("q3"),
    ).sort("arm"))
    # 文字列化は Python 側で f-string
    ```

!!! tip "実務ではこれ"
    - **分位点を数値で出す → f-string で整形**。整形関数（`med_iqr`）を1つ定義して群ごとに `apply`。
    - pandas の四分位は R type 7 と一致するので、R 出力とそのまま突き合わせられる。
    - **丸め**は `f"{x:.1f}"`（表示のみ）。集計値そのものの丸め規則は仕様に合わせる（[018](topic-018.md)）。

---

## 対応早見表

| やりたいこと | R | Python（pandas） |
|---|---|---|
| 中央値 | `median(x)` | `x.median()` |
| Q1 / Q3 | `quantile(x, .25/.75)` | `x.quantile(.25/.75)` |
| `med (Q1, Q3)` | `sprintf("%.1f (%.1f, %.1f)")` | `f"{m:.1f} ({q1:.1f}, {q3:.1f})"` |
| min, max | `min(x)`, `max(x)` | `x.min()`, `x.max()` |
| 群ごと整形 | `summarise` → `sprintf` | `groupby().apply(fmt)` |

## つまずきポイント

!!! warning "R と Python の差"
    - **分位点の定義**：pandas 既定は R type 7 と一致（値がそろう）。SAS の一部プロシジャは別定義なので、SAS 突合時は要確認（`interpolation=` で変更）。
    - **丸め**：`%.1f`/`f"{:.1f}"` の丸めが SAS の四捨五入とズレ得る（[018](topic-018.md)）。
    - **欠損**：`median`/`quantile` は既定で欠損を無視（pandas）／`na.rm` 明示（R）。
    - **表記ゆれ**：`[Q1, Q3]` `(Q1–Q3)` `Q1-Q3` は規約次第。ハイフンとダッシュ（–）も区別されることがある。

## 関連項目

- [052. 連続変数の要約](topic-052.md)
- [055. 群別 N・mean(sd) のデモグラ表](topic-055.md)
- [016. 書式付き数値・sprintf](topic-016.md)
