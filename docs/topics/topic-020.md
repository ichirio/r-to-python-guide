# 020. 正規表現の違い — R の regex 系 vs Python `re` の作法

!!! abstract "この項目の R→Python 対応"
    - **R**: base（`grepl`/`gsub`、既定 POSIX 拡張、`perl=TRUE` で PCRE）／stringr（ICU 正規表現）
    - **Python（推奨）**: 標準 **`re`** モジュール。**raw 文字列 `r"\d+"`** を使うのが鉄則
    - **要注意**: R は普通の文字列で `\\d`（**バックスラッシュ2つ**）、Python は raw 文字列で `\d`（1つ）

---

## R ではこう書く

```r
library(stringr)

grepl("\\d+", "abc123")               # TRUE（\\ でエスケープ）
str_extract("abc123", "\\d+")         # "123"
gsub("(?<=a)b", "X", "ab cb", perl = TRUE)  # 後読みは perl=TRUE
str_extract_all("a1b22c333", "\\d+")  # すべて
```

出力:

```text
TRUE
123
aX cb
[[1]]
[1] "1"   "22"  "333"
```

!!! note "R の正規表現の系統"
    | | エンジン | 特徴 |
    |---|---|---|
    | base 既定 | POSIX 拡張（TRE） | `[[:digit:]]` 系。後読みなし |
    | base `perl=TRUE` | PCRE | 後読み・非貪欲など全部入り |
    | stringr | ICU | Unicode に強い。`str_*` で一貫 |
    普通の文字列リテラルなのでメタ文字は `\\d` `\\.` と**バックスラッシュを重ねる**。

---

## Python ではこう書く

```python
import re

print(bool(re.search(r"\d+", "abc123")))     # マッチの有無
print(re.search(r"\d+", "abc123").group())   # 最初のマッチ
print(re.sub(r"(?<=a)b", "X", "ab cb"))      # 後読みは標準で使える
print(re.findall(r"\d+", "a1b22c333"))       # すべて
```

出力:

```text
True
123
aX cb
['1', '22', '333']
```

主な関数の対応:

| R | Python `re` | pandas 列 |
|---|---|---|
| `grepl` / `str_detect` | `re.search`（有無）| `s.str.contains` |
| `grep` | `[i for i,...]` | `s[s.str.contains(...)].index` |
| `sub` / `gsub` | `re.sub`（`count=` で回数）| `s.str.replace(..., regex=True)` |
| `str_extract` | `re.search(...).group()` | `s.str.extract(r"(...)")` |
| `str_extract_all` | `re.findall` | `s.str.findall` |
| `regmatches` + `regexec` | `re.match(...).groups()` | `s.str.extract` |

!!! tip "実務ではこれ"
    - **必ず raw 文字列 `r"..."`**：`"\d"` は Python で警告/将来エラー。`r"\d"` と書く。
    - **後読み・非貪欲**は Python `re` に**最初から入っている**（R base の `perl=TRUE` 不要）。
    - **コンパイルして再利用**：`pat = re.compile(r"^AE")` を作っておくと、ループや `Series.apply` で速い。
    - **フラグ**：大文字小文字無視は `re.IGNORECASE`（R の `ignore.case=TRUE` / `regex(ignore_case=TRUE)`）。複数行は `re.MULTILINE`。

---

## 記法の対応早見

| 意味 | R（普通の文字列） | Python（raw 文字列） |
|---|---|---|
| 数字 | `"\\d"` / `"[[:digit:]]"` | `r"\d"` |
| 単語境界 | `"\\b"`（perl） | `r"\b"` |
| 空白 | `"\\s"` | `r"\s"` |
| 行頭 / 行末 | `"^"` / `"$"` | `r"^"` / `r"$"` |
| 後方参照（置換） | `"\\1"` | `r"\1"` / `r"\g<1>"` |
| 大小無視 | `ignore.case=TRUE` | `flags=re.IGNORECASE` |

## つまずきポイント

!!! warning "R と Python の差"
    - **バックスラッシュの数**：R の普通の文字列は `\\d`（2つ）、Python は raw 文字列で `\d`（1つ）。raw を忘れると `\d` が無効エスケープ扱い。
    - **POSIX クラス**：R でよく使う `[[:alpha:]]` `[[:digit:]]` は Python `re` では**未対応**。`[A-Za-z]` `\d` `[^\W\d_]`（Unicode 文字）に置き換える。
    - **後読みの可否**：R base 既定では後読み不可（`perl=TRUE` が必要）。Python `re` は標準で可。ただし**可変長後読み**は Python `re` も不可（`regex` パッケージなら可）。
    - **Unicode 既定**：Python `re` は既定で Unicode。`\d` が全角数字にマッチし得るので、ASCII 限定なら `re.ASCII` フラグ。stringr(ICU) との細部の差に注意。
    - **`str.contains` の既定は正規表現**：リテラル一致なら `regex=False` を明示（→ [013](topic-013.md)）。

## 関連項目

- [012. 検索と置換](topic-012.md)
- [013. パターン検出・抽出](topic-013.md)
