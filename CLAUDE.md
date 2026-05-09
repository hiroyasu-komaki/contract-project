# CLAUDE.md

このファイルは Claude Code がリポジトリで作業する際のガイドです。

## ディレクトリの役割

| ディレクトリ | 用途 |
|-------------|------|
| `input/` | 入力ファイルの置き場所。契約書（doc/docx）など処理対象のファイルをここに置く。 |
| `mid/` | 中間ファイル。契約書の抽出テキスト（contract_*.md）が入る。 |
| `output/` | 分類レポート（classify_*.md）・法務レビュー（legalreview_*.md）が保存される。 |
| `output2/` | チェック結果レポート（check_report_*.md）が保存される。 |
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

## Python コマンド（src/main.py）

```bash
.venv/bin/python src/main.py summarize <入力ファイル名>  # 契約書テキスト抽出 → mid/contract_*.md
.venv/bin/python src/main.py review                       # 法務レビュー生成 → output/legalreview_*.md
```

入力ファイルは `.doc` / `.docx` に対応。

## カスタムコマンド（Skills）

`.claude/commands/` に定義済み。`/コマンド名` で呼び出せる。

| コマンド | 主な入力 | 主な出力 | 動作概要 |
|---------|---------|---------|---------|
| `/contract-summarize` | input/*.doc/docx | mid/contract_*.md<br>output/legalreview_*.md | 契約書を要約し条文別有利不利分析・関連法規制を含む法務レビューを生成 |
| `/it-contract-classifier` | input/*.doc/docx | mid/contract_*.md<br>output/classify_*.md | IT契約類型マスターに基づき契約書を分類し分類レポートを生成 |
| `/it-contract-checker` | input/*.doc/docx<br>output/classify_*.md | output2/check_report_*.md | 分類済み契約書にチェックリストを照合しチェック結果レポートを生成。`/it-contract-classifier` の実行が前提。 |
