# Context Loading Protocol

> designs/ フォルダからプロジェクトの「デザイン人格」を構築する。
> すべての修正判断はこのコンテキストに基づいて行う。

## HARD-GATE: HG-1

designs/ フォルダが存在する場合、ReviewNote の処理 (Step 3) を開始する前に必ず本手順を完了すること。

## 読み込み順序（優先度順）

### 1. designs/ui_design_brief.md — デザイン憲法

最重要ファイル。すべての視覚判断の基準。

| 抽出項目 | セクション | 用途 |
|---------|-----------|------|
| Brand Colors | Section 2: Colors Table | M-001 修正時にパレット照合。パレット外の色は警告 |
| Typography | Section 2: Typography Table | M-002 修正時にフォント/サイズ/ウェイト照合 |
| Mood keywords | Section 3: Design Style | 抽象的指示（"プレミアムに"）の解釈基準 |
| Competitive refs | Section 3: Reference URLs | "Stripe的な余白" "Linear的なシンプルさ" の判断 |
| Trust Design | Section 3.5: Trust Patterns | P1-P7 必須パターンが修正で損なわれないか確認 |
| Accessibility | Section 5 | コントラスト比、タッチターゲットの検証基準 |
| Breakpoints | Section 1 | レスポンシブ対応の判断 |
| Icon set | Section 6 | アイコン指示時のライブラリ確認 |

### 2. designs/README.md — プロジェクトの顔

| 抽出項目 | セクション | 用途 |
|---------|-----------|------|
| Target users | Section 3: Users & Actors | ユーザー目線の違和感チェックの基準 |
| Goals & metrics | Section 2: Background & Goals | "この修正はゴールに貢献するか" の判断 |
| Constraints | Section 5: Scope & Constraints | 技術・法的制約の確認 |
| Quality expectations | Section 6 | パフォーマンス・アクセシビリティ基準 |

### 3. designs/ubiquitous_language.md — UIラベル辞書

| 抽出項目 | カラム | 用途 |
|---------|--------|------|
| UI Label | ユーザー向けラベル列 | テキスト修正時に正しいドメイン用語を使用 |
| Anti-patterns | 「避けるべき表現」テーブル | 禁止用語の検出 |
| Term→Label map | UL-XXX 全行 | 内部用語→表示名の変換辞書を構築 |

### 4. designs/user_stories.md — ユーザーの目的

| 抽出項目 | セクション | 用途 |
|---------|-----------|------|
| US-XXX → acceptance criteria | Story Details | 修正が受入条件を損なわないか確認 |
| Story map | Epic × Priority | 画面の重要度判断 |

### 5. designs/functional_requirements.md — 画面マップ

| 抽出項目 | セクション | 用途 |
|---------|-----------|------|
| Screen Inventory | SC-XXX テーブル | SC-ID → 画面名 → 目的のマッピング |
| FR-XXX flows | Main Flow | 修正対象の画面が持つユーザーフローの理解 |

## Design Personality — 内部参照モデル

読み込み後、以下の情報を内部的に保持し、全ステップで参照する:

```
brandColors:      {primary, secondary, accent, success, warning, error, info, neutrals}
typography:       {display, h1-h3, body, caption} × {font, size, weight}
mood:             "信頼感・洗練・静けさ" (例)
competitiveRefs:  ["Stripe", "Linear"] (例)
targetUser:       "40代CTO、エンタープライズSaaS意思決定者" (例)
constraints:      ["WCAG 2.1 AA", "日本語必須", "60代シニアも利用"]
trustPatterns:    [P1-P7 必須パターン一覧]
termMap:          Map<internalTerm, uiLabel>  (UL辞書)
screenPurposes:   Map<SC-XXX, {name, purpose, priority}>
```

## コンテキスト駆動の修正判断（例）

| ReviewNote 指示 | コンテキストなし | コンテキストあり |
|----------------|----------------|----------------|
| "プレミアム感を出して" | 暗い色+大きいフォント | Mood"信頼感"→Primary背景 + 競合Linear風の余白 |
| "テキストを変更" | そのまま反映 | UL辞書で正しいラベルに正規化 |
| "色を #FF0000 に" | そのまま適用 | パレット外→警告、Error色 #EF4444 を提案 |
| "余白を広げて" | 16px→24px | spacing tokens に基づき spacing/6→spacing/8 |
| "ボタンを目立たせて" | サイズ拡大 | Trust P5 の適切なフリクション + アクセシビリティ確認 |

## Fallback: designs/ が存在しない場合

```
⚠ designs/ フォルダが見つかりません。
  プロジェクト固有のコンテキストなしで処理します。
  修正はユニバーサルなデザイン原則に基づきます。
  より精度の高い修正には /requirements_designer で designs/ を生成してください。
```

汎用モードでは:
- 色: Material Design / Tailwind パレットで検証
- フォント: システムデフォルト
- 余白: 4px/8px グリッドのみ検証
- ユーザー目線: 一般的なユーザビリティ原則
