# 契約書レビューエージェント with Claude Code

Claude Code のカスタムスキル（Skills）を活用した契約書レビュー・分類自動化ツールです。
Python スクリプトがファイル処理を担当し、Claude Code が要約・法務レビュー・IT契約類型分類・チェックリスト照合を行います。

## 前提条件

- Claude Pro / Max / Teams / Enterprise アカウント
- Claude Code（ネイティブインストーラー推奨）
- Python 3.10+

## セットアップ

```bash
# 1. Claude Code のインストール
curl -fsSL https://claude.ai/install.sh | bash

# 2. プロジェクトフォルダに移動
cd ~/Documents/contract_project

# 3. Python 仮想環境の構築
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt

# 4. Claude Code を起動（初回認証）
claude
```

## プロジェクト構成

```
contract_project/
├── .claude/commands/
│   ├── contract-summarize.md       ← Skill: 契約書レビュー
│   ├── it-contract-classifier.md  ← Skill: IT契約類型分類
│   └── it-contract-checker.md     ← Skill: チェックリスト照合
├── src/
│   └── main.py                     ← 契約書テキスト抽出・レビューテンプレート生成
├── direction/                      ← スキル仕様・参照ドキュメント
│   ├── it-contract-type-classifier.md  ← IT契約類型マスター
│   ├── classify_template.md             ← 分類レポートテンプレート
│   ├── review_template.md               ← 法務レビューテンプレート
│   └── check_report_template.md         ← チェック結果レポートテンプレート
├── input/      ← 処理対象の契約書（.doc / .docx）を置く
├── mid/        ← 中間ファイル（抽出テキスト）
├── output/     ← 分類レポート・法務レビュー
├── output2/    ← チェック結果レポート
├── .venv/
├── requirements.txt
├── CLAUDE.md
└── README.md
```

## 使い方

`input/` フォルダに契約書ファイル（.doc / .docx）を置いてから、Claude Code でスキルを実行します。

### 1. 契約書レビュー

```
/contract-summarize
```

契約書を読み込み、条文別の有利不利分析と関連法規制を含む法務レビューを生成します。

- 出力: `output/legalreview_*.md`
- 内容: 契約の種類・目的・当事者・契約期間、条文一覧と概要、条文別有利不利分析（甲・乙それぞれへの影響）、該当する可能性のある法規制

### 2. IT契約類型分類

```
/it-contract-classifier
```

IT契約類型マスターに基づき契約書を分類し、分類レポートを生成します。

- 出力: `output/classify_*.md`
- 内容: 主類型・副類型の類型コード・類型名、確信度（High / Medium / Low）と判定根拠、複合契約フラグ、類型別チェックリスト

### 3. チェックリスト照合

```
/it-contract-checker
```

分類済み契約書に対してチェックリストを照合し、条文ごとの充足度を評価したレポートを生成します。事前に `/it-contract-classifier` の実行が必要です。

- 出力: `output2/check_report_*.md`
- 内容: 条文あり（yes / no / unknown）判定、条文番号、充足度（1〜5段階）、根拠・備考、総評

## アーキテクチャ

```
Skills が手順を定義
    → Claude Code が Python を実行
        → Python が契約書テキストを抽出
            → Claude Code が要約・法務レビュー・分類・チェックを実施
```

- **Skills**（`.claude/commands/`）— Claude Code への指示書。処理手順を定義する。
- **Python**（`src/main.py`）— ファイル処理・テキスト抽出。情報収集の道具。
- **direction/**（スキル仕様）— 各スキルの判断基準・出力フォーマットを定義するドキュメント。
- **Claude Code** — Skills の手順に従い Python を実行し、結果を分析・要約・分類・評価する AI の頭脳。

## 注意事項

- Claude Code の利用には Pro 以上の有料プランが必要です（追加の API 課金は不要）
- 法務レビュー・チェック結果は参考情報です。法的判断が必要な事項については別途専門家へご相談ください
