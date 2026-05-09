# Microsoft 企業環境への移植検討メモ

作成日: 2026-05-09

## 背景

Microsoft Copilot Pro ライセンスを持つ企業環境への移植を検討する際の選択肢と比較。  
契約書の機密性・処理ロジックの複雑さを考慮し、以下の3つの選択肢を検討する。

---

## 現行アーキテクチャと移植先の比較

| レイヤー | 現状 | A: Azure OpenAI | B: Copilot Studio | C: M365 Copilot |
|:---:|:---:|:---:|:---:|:---:|
| AI基盤 | Anthropic Claude<br>（Claude Code / API） | Azure OpenAI Service<br>（GPT-4o） | Copilot Studio（GPT-4o）<br>+ AI Builder | Microsoft 365 Copilot<br>（Word / Teams） |
| 文書処理 | Python<br>（python-docx / antiword） | Python<br>（python-docx / antiword）<br>※変更なし | AI Builder ドキュメント処理<br>+ SharePoint コネクタ | Word が自動処理<br>（docx ネイティブ対応） |
| ロジック | `.claude/commands/` の Skill<br>（Markdown定義） | Python スクリプト<br>（プロンプトをコードに内包） | Copilot Studio トピック／アクション<br>（ローコード定義） | 人手によるプロンプト入力<br>（自動化なし） |
| UI | CLI<br>（ターミナル操作） | CLI<br>（ターミナル操作）<br>※変更なし | Teams チャット<br>/ SharePoint ポータル | Word / Teams の<br>Copilot チャット欄 |
| 出力 | Markdownファイル<br>（output/, output2/） | Markdownファイル<br>（output/, output2/）<br>※変更なし | SharePoint ドキュメント<br>/ Teams メッセージ | Word ドキュメント<br>/ Teams チャット返答 |

---

## 各選択肢の詳細

### 選択肢 A: Azure OpenAI Service へのバックエンド差し替え（推奨）

**概要**: 現行の Python コードと分類ロジックをほぼそのまま活かし、AI 呼び出し先を Claude → Azure OpenAI (GPT-4o) に切り替える。

```
[.docx 入力] → [Python: python-docx で抽出] → [Azure OpenAI API で分類/チェック] → [Markdown 出力]
```

**工数感**: 中（1〜2週間）

---

### 選択肢 B: Copilot Studio でカスタム Copilot を構築

**概要**: Microsoft の Copilot Studio（ローコード）でエージェントを作り、SharePoint 上の契約書を処理させる。

```
[SharePoint: .docx 保存] → [Copilot Studio エージェント] → [AI Builder + GPT-4] → [SharePoint/Teams に出力]
```

**工数感**: 大（1〜2ヶ月、かつスキルセットが M365 管理者寄りになる）

---

### 選択肢 C: M365 Copilot（Word / Teams）を補助ツールとして活用（最小移植）

**概要**: 契約書を SharePoint に置き、Word Copilot や Teams Copilot を使って手動レビューを補助する。自動化はしない。

**工数感**: 小（ほぼゼロ）、ただし機能的な価値も小

---

## Pros / Cons 比較

| 観点 | A: Azure OpenAI | B: Copilot Studio | C: M365 Copilot |
|:---:|:---|:---|:---|
| **Pros** | ・現行ロジック（チェックリスト・分類マスター）をそのまま流用できる<br>・Azure テナント内でデータが完結し、情報漏洩リスクが低い<br>・Azure OpenAI の従量課金は安価（GPT-4o）<br>・段階的な移行が可能（AI 呼び出しだけ先行差し替え） | ・M365 Copilot ライセンスがあれば追加コストが比較的少ない<br>・Teams・Word との統合が容易（社内で使い慣れた UI）<br>・ガバナンス・監査ログが M365 管理センターで一元管理できる | ・追加開発ゼロ<br>・Copilot Pro / M365 Copilot ライセンスをそのまま活用できる<br>・導入障壁が最も低い |
| **Cons** | ・Claude と GPT-4o でプロンプトの挙動が異なるため調整が必要<br>・Skill 機構（Markdown定義）が使えなくなり Python コードへの書き直しが必要<br>・Azure リソースのプロビジョニング作業が発生<br>・Copilot Pro ライセンスに Azure OpenAI は含まれない（別途コスト） | ・チェックリスト照合や複雑な分類ロジックの実装がローコードでは困難<br>・AI Builder のドキュメント処理能力は Claude より劣る<br>・分類マスター等を「知識ソース」として取り込む工夫が必要<br>・Markdown 出力フォーマットを SharePoint 向けに再設計が必要 | ・自動分類・チェックレポート生成の価値がほぼ失われる<br>・1件ごとに人手でプロンプトを入力する必要がある<br>・出力フォーマットが毎回異なり品質が安定しない<br>・チェックリスト照合の自動化が不可能 |

---

## 総合評価

| 観点 | A: Azure OpenAI | B: Copilot Studio | C: M365 Copilot |
|:---:|:---:|:---:|:---:|
| 現行機能の維持 | 高 | 中 | 低 |
| 開発コスト | 中（1〜2週間） | 大（1〜2ヶ月） | 小（ほぼゼロ） |
| ライセンス活用 | 低〜中 | 高 | 高 |
| データセキュリティ | 高 | 高 | 高 |
| 運用・保守容易性 | 中 | 高 | 高 |

---

## 推奨

**選択肢 A（Azure OpenAI）が最も現実的。**

このプロジェクトの核心価値は「分類マスター・チェックリストとの照合ロジック」にあり、Copilot Studio のローコードでは忠実な再現が難しい。

Copilot Pro ライセンスの活用を最大化するなら、**A + C の組み合わせ**（バックエンドは Azure OpenAI で自動処理、結果確認・共有は Teams/SharePoint）が現実的な落としどころ。

---

## Appendix: 現行アーキテクチャ シーケンス図

### A-1. `/contract-summarize`（法務レビュー生成）

```mermaid
sequenceDiagram
    actor User
    participant Skill as Claude Code<br>(Skill)
    participant FS as ファイルシステム
    participant Python as Python<br>(src/main.py)
    participant Claude as Claude AI<br>(推論エンジン)

    User->>Skill: /contract-summarize 実行

    Note over Skill,FS: Step 0: ファイル選択
    Skill->>FS: find input/ *.doc/*.docx
    FS-->>Skill: ファイル一覧
    alt ファイルが複数
        Skill->>User: 番号選択を促す
        User->>Skill: 番号入力
    end

    Note over Skill,Python: Step 1: テキスト抽出
    Skill->>Python: summarize "選択ファイル名"
    Python->>FS: input/*.docx を読み込み
    FS-->>Python: バイナリデータ
    Python->>Python: python-docx でテキスト抽出
    Python->>FS: mid/contract_*.md を書き込み
    FS-->>Skill: 完了通知
    Skill->>FS: mid/contract_*.md を読み込み
    FS-->>Skill: 契約書テキスト

    Note over Skill,Claude: Step 1-2: 要約生成（Claude による追記）
    Skill->>Claude: 契約書テキストを渡し要約を依頼
    Claude->>Claude: 種類・目的・当事者・条文一覧・特記事項を分析
    Claude-->>Skill: 要約テキスト
    Skill->>FS: mid/contract_*.md に要約を追記

    Note over Skill,Python: Step 2: 法務レビューテンプレート生成
    Skill->>Python: review
    Python->>FS: mid/contract_*.md を読み込み
    FS-->>Python: 契約書テキスト + 要約
    Python->>FS: direction/review_template.md を読み込み
    FS-->>Python: テンプレート
    Python->>FS: output/legalreview_*.md を書き込み
    FS-->>Skill: 完了通知
    Skill->>FS: output/legalreview_*.md を読み込み
    FS-->>Skill: レビューテンプレート

    Note over Skill,Claude: Step 2-2: 法務レビュー記入（Claude による追記）
    Skill->>Claude: 条文別 有利不利分析・関連法規制の記入を依頼
    Claude->>Claude: 各条文を甲乙視点で分析<br>関連法規制（個人情報保護法 等）を列挙
    Claude-->>Skill: レビュー内容
    Skill->>FS: output/legalreview_*.md にレビューを追記
    Skill->>User: 完了通知（出力ファイルパス）
```

---

### A-2. `/it-contract-classifier`（IT契約類型分類）

```mermaid
sequenceDiagram
    actor User
    participant Skill as Claude Code<br>(Skill)
    participant FS as ファイルシステム
    participant Python as Python<br>(src/main.py)
    participant Claude as Claude AI<br>(推論エンジン)

    User->>Skill: /it-contract-classifier 実行

    Note over Skill,FS: Step 0: ファイル選択
    Skill->>FS: find input/ *.doc/*.docx
    FS-->>Skill: ファイル一覧
    alt ファイルが複数
        Skill->>User: 番号選択を促す
        User->>Skill: 番号入力
    end

    Note over Skill,Python: Step 1: テキスト抽出
    Skill->>Python: summarize "選択ファイル名"
    Python->>FS: input/*.docx を読み込み
    FS-->>Python: バイナリデータ
    Python->>Python: python-docx でテキスト抽出
    Python->>FS: mid/contract_*.md を書き込み
    FS-->>Skill: 完了通知
    Skill->>FS: mid/contract_*.md を読み込み
    FS-->>Skill: 契約書テキスト

    Note over Skill,Claude: Step 2: IT契約類型分類
    Skill->>FS: direction/it-contract-type-classifier.md を読み込み
    FS-->>Skill: 分類マスター（類型定義・判定キーワード）
    Skill->>Claude: 契約書テキスト + 分類マスターを渡し分類を依頼
    Claude->>Claude: ①基本情報抽出<br>②主たる目的特定<br>③判定キーワード照合<br>④複合契約判定<br>⑤主類型・副類型決定<br>⑥確信度評価（High/Medium/Low）
    Claude->>Claude: セルフチェック（根拠・複合・区別点・確信度を確認）
    Claude-->>Skill: 分類結果（主類型・副類型・確信度・根拠）

    Note over Skill,FS: Step 3: レポート出力
    Skill->>FS: direction/classify_template.md を読み込み
    FS-->>Skill: 分類レポートテンプレート
    Skill->>FS: checklists/{主類型コード}_it-contract-checklist.md を読み込み
    FS-->>Skill: 主類型チェックリスト
    alt 複合契約の場合
        Skill->>FS: checklists/{副類型コード}_it-contract-checklist.md を読み込み
        FS-->>Skill: 副類型チェックリスト
    end
    Skill->>FS: output/classify_*.md を書き込み
    Skill->>User: 完了通知（出力ファイルパス・分類結果サマリー）
```

---

### A-3. `/it-contract-checker`（チェックリスト照合）

```mermaid
sequenceDiagram
    actor User
    participant Skill as Claude Code<br>(Skill)
    participant FS as ファイルシステム
    participant Claude as Claude AI<br>(推論エンジン)

    User->>Skill: /it-contract-checker 実行
    Note right of User: /it-contract-classifier 実行済みが前提

    Note over Skill,FS: Step 0: ファイル選択
    Skill->>FS: find input/ *.doc/*.docx
    FS-->>Skill: ファイル一覧
    alt ファイルが複数
        Skill->>User: 番号選択を促す
        User->>Skill: 番号入力
    end

    Note over Skill,FS: Step 1: 対応ファイルの自動検索
    Skill->>FS: grep -rl "選択ファイル名" mid/
    FS-->>Skill: 対応する contract_*.md
    Skill->>FS: grep -rl "選択ファイル名" output/
    FS-->>Skill: 対応する classify_*.md
    Note over Skill: 複数ヒット時はタイムスタンプ最新を使用
    Skill->>FS: mid/contract_*.md を読み込み
    FS-->>Skill: 契約書テキスト
    Skill->>FS: output/classify_*.md を読み込み
    FS-->>Skill: 分類レポート（類型・チェックリスト全文）

    Note over Skill,Claude: Step 2: チェック実施
    Skill->>Claude: 契約書テキスト + チェックリストを渡し照合を依頼
    loop 全チェック項目
        Claude->>Claude: 【第1段階】条文あり判定<br>yes / no / unknown
        alt 条文あり = yes
            Claude->>Claude: 【第2段階】充足度評価<br>1〜5 の5段階
        end
    end
    Claude->>Claude: 総評作成<br>（充足度傾向・優先課題・リスク感）
    Claude-->>Skill: チェック結果（全項目 + 総評）

    Note over Skill,FS: Step 3: レポート出力
    Skill->>FS: direction/check_report_template.md を読み込み
    FS-->>Skill: チェックレポートテンプレート
    Skill->>FS: output2/check_report_*.md を書き込み<br>（classify_*.md 全文 + チェック結果セクション）
    Skill->>User: 完了通知<br>（出力パス・条文あり/なし/不明の件数・総評）
```

---

