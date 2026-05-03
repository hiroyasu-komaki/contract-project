# 契約書レビューエージェント with Claude Code

Claude Code のカスタムスキル（Skills）を活用した契約書レビュー・分類自動化ツールです。
Python スクリプトがファイル処理を担当し、Claude Code が要約・法務レビュー・IT契約類型分類を行います。

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
│   ├── contract-summarize.md      ← Skills（契約書レビュー）
│   └── it-contract-classifier.md ← Skills（IT契約類型分類）
├── src/
│   └── main.py                    ← 契約書テキスト抽出・レビューテンプレート生成
├── direction/                     ← スキル仕様・参照ドキュメント
│   ├── it-contract-type-classifier.md  ← IT契約類型マスター
│   ├── classify_template.md            ← 分類レポートテンプレート
│   └── review_template.md              ← 法務レビューテンプレート
├── input/                     ← 入力ファイルの置き場所（.doc / .docx）
├── mid/                       ← 中間ファイル（抽出テキスト）
├── output/                    ← 最終出力（法務レビュー・分類レポート）
├── .venv/
├── requirements.txt
├── CLAUDE.md
└── README.md
```

## 使い方

`input/` フォルダに契約書ファイル（.doc / .docx）を置いてから、Claude Code でコマンドを実行します。

### 契約書レビュー

```
/contract-summarize
    ↓ 契約書を読み込み要約 → mid/contract_*.md
    ↓ 条文別有利不利分析 + 関連法規制の列挙 → output/legalreview_*.md
```

**出力内容：**
- 契約の種類・目的、当事者、契約期間
- 条文一覧と概要
- 条文別 有利不利分析（甲・乙それぞれへの影響）
- 該当する可能性のある法規制・業界規制

### IT契約類型分類

```
/it-contract-classifier
    ↓ 契約書を読み込み要約 → mid/contract_*.md
    ↓ IT契約類型マスターに基づき分類 → output/classify_report_*.md
```

**出力内容：**
- 主類型（Primary）と副類型（Secondary）の類型コード・類型名
- 確信度（High / Medium / Low）と判定根拠（契約書内の具体的な記述を引用）
- 複合契約フラグ
- 注意事項・推奨アクション

## アーキテクチャ

```
Skills が手順を定義
    → Claude Code が Python を実行
        → Python が契約書テキストを抽出
            → Claude Code が要約・法務レビューを生成
```

- **Skills**（`.claude/commands/`）— Claude Code への指示書。やり方を定義する。
- **Python**（`src/main.py`）— ファイル処理・テンプレート生成。情報収集の道具。
- **direction/**（スキル仕様）— 各スキルの判断基準・出力フォーマットを定義するドキュメント。
- **Claude Code** — Skills の手順に従い Python を実行し、結果を分析・要約・分類する AI の頭脳。

## 注意事項

- Claude Code の利用には Pro 以上の有料プランが必要です（追加の API 課金は不要）
- 法務レビューは参考情報です。法的判断が必要な事項については別途専門家へご相談ください
