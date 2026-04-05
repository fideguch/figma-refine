# AIDesigner MCP Integration

> Level B（構造改善）の修正に AIDesigner MCP を活用する。
> 既存デザインの HTML 表現を refine_design で改善し、
> ユーザー承認後に Figma MCP で反映する。

## Prerequisites

- AIDesigner MCP 接続済み（`whoami` + `get_credit_status` で確認）
- Deep Scan (Step 2) 完了済み — 対象フレームの構造を把握済み
- Design Personality 構築済み (Step 1)

## AIDesigner MCP 接続確認

```
1. whoami → ユーザー情報を確認
2. get_credit_status → 残りクレジットを確認
3. クレジット不足 → ユーザーに報告、Level A のみで処理
```

## repo_context の構築

Design Personality を AIDesigner が理解できる形式に変換する。

```
repo_context = {
  "brand": {
    "colors": Design Personality の brandColors,
    "typography": Design Personality の typography,
    "mood": Design Personality の mood
  },
  "target_user": Design Personality の targetUser,
  "constraints": Design Personality の constraints,
  "competitive_refs": Design Personality の competitiveRefs
}
```

> **CRITICAL**: repo_context は Design Personality の要約のみ。
> designs/ のファイル全文を渡さないこと（トークン浪費 + AIDesigner の創造性を阻害）。

## refine_design ワークフロー

### Step 1: 既存デザインの構造情報取得

Figma MCP `get_design_context` でフレームの構造を取得する（HTML ではなく JSON 構造）。

```
get_design_context(fileKey, nodeId=対象フレームID)
  → レイヤー構造、スタイル情報、テキスト内容、Auto-layout 設定等
```

取得した構造情報を自然言語で要約し、AIDesigner の `refine_design` プロンプトに組み込む:
- レイアウト構成（ヘッダー/サイドバー/コンテンツ等）
- 使用色と背景色
- フォント種類・サイズ
- 主要なテキスト内容
- Auto-layout の方向・padding・gap

> **CRITICAL**: `get_design_context` は Figma ノードの JSON 構造を返す。
> AIDesigner の `refine_design` はテキストプロンプトを受け付ける。
> JSON → 自然言語要約 → プロンプトに含める、という変換が必要。

### Step 2: refine_design 呼び出し

AIDesigner の `refine_design` に渡すプロンプト構成:

```
プロンプト要素:
  1. 現在のデザインの概要（Deep Scan の frameDetail から）
  2. 改善指示（ReviewNote の Content テキスト）
  3. デザインの方向性（Design Personality の mood + competitiveRefs）
  4. 制約（アクセシビリティ、日本語対応、ブランド一貫性）
```

**プロンプト作成ルール**:
- 短く、アート・ディレクション志向にする（PRD のように書かない）
- プロダクトタイプ、ターゲット、UXの優先事項、望む雰囲気を伝える
- セクション順序、カード枚数、ボタンラベル等の詳細は指定しない
- AIDesigner に構造・構成・ビジュアルリズムの発明余地を残す

**プロンプト例**:
```
"SaaS ダッシュボードのヘッダーセクション。
ターゲットは日本の40代CTO。信頼感と洗練さを重視。
現在のデザインをよりプレミアムにブラッシュアップ。
Linear のようなクリーンな余白、控えめだが明確なナビゲーション。
WCAG AA 準拠、日本語テキスト14px以上。"
```

### Step 3: Before/After 提示

AIDesigner から改善版 HTML が返却されたら、ユーザーに Before/After を提示する。

```
📐 Level B 改善提案

🔹 Before: [Deep Scan で取得した Before スクリーンショット]
  - 現在のデザイン構造の説明

🔹 After (AIDesigner 提案):
  - AIDesigner が生成した改善版の説明
  - 主な変更点のリスト

🔹 Design Personality との整合性:
  - brandColors: ✓ パレット内 / ⚠ 新色追加あり
  - typography: ✓ 準拠 / ⚠ 新フォントサイズあり
  - mood: ✓ 一致 / ⚠ トーン変化あり

承認しますか？
  [Y] 承認 → Figma に反映
  [N] 却下 → 修正指示を追加して再改善
  [M] 手動 → 具体的な修正指示に分解して Level A で処理
```

### Step 4: Figma 反映（承認時）

AIDesigner の出力はデザイン参照として扱い、Figma にそのまま貼り付けない。

```
AIDesigner 出力の解釈:
  1. 色の変更 → M-001 で反映（brandColors と照合）
  2. タイポグラフィ変更 → M-002 で反映（フォント確認必須）
  3. 余白変更 → M-005 で反映（4px grid 準拠）
  4. サイズ変更 → M-003 で反映
  5. 構造変更 → use_figma で手動コード実装
```

> **CRITICAL**: AIDesigner の HTML をそのまま Figma に変換しない。
> 改善の「意図」を抽出し、既存の M-code パターンで1つずつ反映する。

### Step 5: 再改善（却下時）

```
1. ユーザーから追加の修正指示を受け取る
2. 前回のプロンプト + 追加指示で refine_design を再実行
3. Before/After を再提示
4. 最大3回まで。3回却下 → 手動対応を提案
```

## AIDesigner クレジット管理

- 各 refine_design 呼び出し前に `get_credit_status` を確認
- クレジット残量をユーザーに表示
- 残り少ない場合は警告してから実行

```
💳 AIDesigner クレジット: 残り {n} 回
  この修正で 1 クレジット使用します。続行しますか？
```

## Fallback: AIDesigner 未接続時

Level B の修正が検出されたが AIDesigner が利用できない場合:

```
1. 修正指示を具体化できるか試みる
   "モダンに" → "余白を広げ、フォントをlight weightに、角丸を大きく"
   → 3つの Level A 修正に分解
2. 分解できない場合 → ユーザーに手動対応を報告
3. 処理可能な Level A のみ先に実行
```

## 出力フォーマット

Level B 処理完了後の報告:

```
📐 Level B 修正完了

  ReviewNote #3: "ヘッダーをモダンに"
  分類: Level B（構造改善）
  AIDesigner: refine_design 1回実行（承認: 1回目）

  適用した変更:
  1. M-001: 背景色 #FFFFFF → #F8FAFC (Slate-50)
  2. M-005: paddingY 16px → 24px
  3. M-002: フォントウェイト Regular → Medium
  4. use_figma: ナビゲーション配置を Flex → Space-between に変更

  検証: L3 ✓ | L2 ✓ | L1 ✓
  ステータス: 未読 → 完了 ✅
```
