# ccken リポジトリ ガイド

三菱CC研究会 第3部会（エンタープライズにおけるAI駆動開発の研究）の作業リポジトリ。

## ディレクトリの役割

| ディレクトリ | 用途 |
|---|---|
| `CC研_運営/` | 研究会の運営・進め方・To-Be/ギャップ資料（公式寄りの土台文書） |
| `personal_doc/` | 個人の整理メモ・登壇資料サマリー。`分科会インセプションデッキ/` に4分科会（①AI-Ready基盤 ②AI駆動開発プロセス ③品質保証 ④人と組織）の編成資料とデッキ |
| `参考資料/` | 登壇スライド等のPDF原本 |
| `参考資料/MD/` | 参考資料PDFの構造化Markdown要約（PDFと同名） |

## 作業規約

- **PDF→Markdown変換**: `参考資料/` のPDFをMD化する際は `pdf-to-markdown` skill の規約に従う（出力は `参考資料/MD/` に同名 `.md`、既存は再作成しない、構造化要約スタイル）。
- **コミット/プッシュ**: ユーザーは作業内容の事前確認を求めない。まとまった単位で**自律的に commit & push してよい**。コミットは新規作成（amendしない）、メッセージ末尾に `Co-Authored-By: Claude ...` トレーラを付ける。リモートは `origin`（GitHub: Ponthai-project/ccken）、デフォルトブランチ `main`。

## 環境メモ（Windows）

- 日本語ファイル名は NFC/NFD 差でシェルの単純比較（`for f in *.pdf` 等）が誤検知する。PDF↔MDの照合は Python の `unicodedata.normalize('NFC', ...)` で行う。
- Python スクリプトで日本語を print すると Windows コンソール(cp932)で `UnicodeEncodeError`。`PYTHONUTF8=1` ＋ `io.TextIOWrapper(...encoding='utf-8')` を使う。Python に渡すパスは msys形式(`/c/...`)でなく Windowsパス(`C:\...`)。
- Git の `LF will be replaced by CRLF` 警告は無害。
