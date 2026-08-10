# 089. エラー処理 — `tryCatch` → `try` / `except`

!!! abstract "この項目の R→Python 対応"
    - **R**: `tryCatch(expr, error=, warning=, finally=)`／`try()`／`withCallingHandlers`
    - **Python（推奨）**: `try / except / else / finally`（例外の型で捕捉）
    - **要注意**: R は warning も条件として捕まえられる。Python の warning は例外ではない（`warnings` モジュール別扱い）

---

## R ではこう書く

```r
r <- tryCatch(
  log(-1),                              # 警告が出る
  warning = function(w) NA_real_,       # 警告を捕捉
  error   = function(e) NULL,           # エラーを捕捉
  finally = cat("done\n")
)
print(r)   # NA
```

出力:

```text
[1] NA
```

!!! note "R の勘所"
    - `tryCatch` は **error / warning / message** を条件として捕捉できる。
    - `finally` は必ず実行。
    - 単に失敗を無視するなら `try(expr, silent = TRUE)`。

---

## Python ではこう書く

=== "try / except"

    ```python
    try:
        r = risky()
    except ValueError as e:          # 型を指定して捕捉
        r = None
    except (KeyError, IndexError):   # 複数型
        r = None
    else:
        pass                         # 例外が出なかったとき
    finally:
        print("done")               # 必ず実行
    ```

=== "warning を捕捉する"

    ```python
    import warnings, numpy as np
    with warnings.catch_warnings():
        warnings.simplefilter("error")     # 警告を例外に昇格
        try:
            r = float(np.log(-1))
        except Exception:
            r = np.nan
    print(r)                                # nan
    ```

    出力:

    ```text
    nan
    ```

!!! tip "実務ではこれ"
    - **例外の型を指定**して捕まえる（`except Exception:` の握りつぶしは最小限に）。R の `error=` に対応。
    - **warning は例外ではない**：Python の警告は `warnings` モジュール。例外として扱いたいなら `warnings.simplefilter("error")` で昇格。
    - `finally` は R と同じく必ず実行。後片付け（ファイルクローズ等）に。
    - 失敗を値で返したいなら `except` で既定値を返す（`tryCatch(..., error=function(e) NA)` 相当）。

---

## 対応早見表

| やりたいこと | R | Python |
|---|---|---|
| エラー捕捉 | `tryCatch(expr, error=)` | `try: ... except SomeError:` |
| 警告捕捉 | `tryCatch(expr, warning=)` | `warnings.catch_warnings` + `simplefilter("error")` |
| 必ず実行 | `finally=` | `finally:` |
| 失敗を無視 | `try(expr, silent=TRUE)` | `try: ... except Exception: pass` |
| 例外を投げる | `stop("msg")` | `raise ValueError("msg")` |
| 警告を出す | `warning("msg")` | `warnings.warn("msg")` |

## つまずきポイント

!!! warning "R と Python の差"
    - **warning の扱いが別世界**：Python では警告は例外でない。`tryCatch(warning=)` のノリで捕まえるには `warnings` モジュールで昇格が要る。
    - **握りつぶし注意**：`except Exception: pass` は全例外を飲み込み、バグを隠す。**型を絞る**か、少なくともログを残す。
    - **例外の型階層**：`ValueError`/`KeyError` などを指定。`except:`（型なし）は `KeyboardInterrupt` まで飲むので避ける。
    - **`else` 節**：`try` が成功したときだけ走る節（R にない）。副作用を try 本体から分離できる。

## 関連項目

- [086. 関数定義](topic-086.md)
- [095. デバッグと検算のコツ](topic-095.md)
- [028. 型変換](topic-028.md)
