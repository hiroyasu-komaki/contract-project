# CLAUDE.md

このファイルは Claude Code がリポジトリで作業する際のガイドです。

## ディレクトリの役割

| ディレクトリ | 用途 |
|-------------|------|
| `input/` | 入力ファイルの置き場所。契約書（doc/docx）など処理対象のファイルをここに置く。 |
| `mid/` | 中間ファイル。要約など、最終出力の前段階で生成されるファイルが入る。 |
| `output/` | 最終出力。法務レビューなどの成果物がここに保存される。 |
| `src/` | Python ソースコード。Skills から呼び出される処理の実体。 |
| `direction/` | スキル仕様・参照ドキュメント。各スキルの設計方針や出力フォーマット定義。 |
| `.claude/commands/` | Skills（カスタムコマンド）の定義。Markdown ファイル 1 つが 1 つの Skill。 |

## Python 実行ルール

Python は必ず仮想環境内で実行すること。コマンドは `.venv/bin/python` を直接指定する。

```bash
# ✅ 正しい（仮想環境のPythonを直接指定）
.venv/bin/python src/main.py summarize "契約書.docx"

# ❌ 避ける（activate 忘れのリスクがある）
python src/main.py summarize "契約書.docx"
```

パッケージの追加も仮想環境内で行う：

```bash
.venv/bin/pip install <パッケージ名>
.venv/bin/pip freeze > requirements.txt
```

## コマンド実行

### legalreview
```bash
.venv/bin/python src/main.py summarize <入力ファイル名>
.venv/bin/python src/main.py review
```
出力：`mid/contract_YYYYMMDD_HHmmss.md` / `output/legalreview_YYYYMMDD_HHmmss.md`
入力ファイルは `.doc` / `.docx` に対応。

## コードの構造

```
src/
└── main.py      summarize / review の2サブコマンド。入力はdoc/docx（python-docx使用）、中間出力はmid/、最終出力はoutput/。
```

## カスタムコマンド（Skills）

`.claude/commands/` に定義済み。`/コマンド名` で呼び出せる。

| コマンド | 動作 |
|---------|------|
| `/contract-summarize` | input/ の契約書（doc/docx）をファイル選択して要約 → mid/contract_*.md を生成 → 法務レビューを生成 → output/legalreview_*.md |
| `/it-contract-classifier` | input/ の契約書（doc/docx）をファイル選択してIT契約類型マスターで分類 → output/classify_report_*.md |
