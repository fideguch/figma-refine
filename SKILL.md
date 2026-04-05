---
name: figma-refine
description: >-
  Context-aware Figma design refinement with AIDesigner integration.
  Reads designs/ for project identity, classifies modifications into
  Level A (property) and Level B (structural), applies via Figma MCP
  with optional AIDesigner-powered improvements, and verifies with
  3-layer screenshot validation + Before/After comparison.
  Use after requirements_designer Phase 5 to polish UI designs.
  Triggers: "figma-refine", "デザイン洗練", "レビューノート処理", "デザイン修正",
  "review note", "figma review", "デザインレビュー", "ReviewNote"
---

# figma-refine

プロジェクト文脈を理解した上で Figma デザインを洗練するスキル。
`designs/` フォルダからブランド・ユーザー・競合・ドメイン用語を読み込み、
ReviewNote コンポーネントの指示を Level A/B に分類して修正を適用する。
Level B（構造改善）では AIDesigner MCP で改善版を生成し、ユーザー承認後に反映する。

## Prerequisites

- Figma MCP 接続済み（`whoami` で確認）
- Figma Design ファイルの URL（fileKey を抽出）
- `designs/` フォルダ（推奨。なくてもユーザーに質問して Design Personality を構築）
- AIDesigner MCP（Level B 修正に必要。なくても Level A のみで動作）

## 6-Step Workflow

### Step 1: Context Load

<HARD-GATE id="HG-1">
designs/ フォルダが存在する場合、Step 3 の ReviewNote 処理を開始する前に
必ず本ステップを完了すること。designs/ を読まずに修正を適用してはならない。
</HARD-GATE>

**Read** → `references/context_loading.md`

1. `designs/` を読み込み → Design Personality を構築
2. designs/ なし → ユーザーに質問（Q1-Q6）して手動構築、または汎用モード

### Step 2: Deep Scan

**Read** → `references/deep_scan.md`

1. `get_metadata` でファイル構造を把握（ページ、フレーム、DS）
2. ReviewNote 対象フレームの詳細スキャン（`get_design_context`）
3. Before スクリーンショット取得（修正前の状態を記録）
4. designs/ との照合（色・フォント・構造の乖離を記録）

### Step 3: Classify & Process

**Read** → `references/level_classification.md`
**Read** → `references/processing_algorithm.md`
**Read** → `references/supported_modifications.md`
**Read** → `references/figma_api_gotchas.md`

各 ReviewNote（未読のみ）に対して:

1. **Detect**: 未読 ReviewNote を全ページから収集
2. **Resolve**: Name Mode でターゲット解決 → 失敗時 Position Mode
3. **Classify**: Level A / Level B に分類
4. **Level A**: M-code にマッピング → `use_figma` で修正
5. **Level B**: → Step 4 の AIDesigner フローへ

<HARD-GATE id="HG-2">
テキスト修正 (M-002) を実行する前に、必ず figma.listAvailableFontsAsync() で
対象フォントの利用可能性を確認すること。確認なしのテキスト修正は禁止。
</HARD-GATE>

### Step 4: AIDesigner Apply (Level B only)

**Read** → `references/aidesigner_integration.md`

1. Design Personality を repo_context に変換
2. `refine_design` で改善版を生成
3. Before/After をユーザーに提示
4. 承認 → AIDesigner 出力の「意図」を M-code で1つずつ Figma に反映
5. 不承認 → 再改善（最大3回）or 手動修正

> AIDesigner 未接続時: Level B をスキップし Lite Mode で処理

### Step 5: Report

**Read** → `references/design_change_report.md`

Design Change Report を出力:
- 各変更の対象・修正内容・根拠・Level
- designs/ との乖離分析（色→palette照合、テキスト→UL照合、構造→FR照合）
- 乖離あり → `/requirements_designer enhance` での更新を推奨

<HARD-GATE id="HG-3">
designs/ への書き込みは禁止。レポート出力のみ。
designs/ の更新は /requirements_designer enhance またはユーザー手動に委譲する。
</HARD-GATE>

### Step 6: Verify

**Read** → `references/screenshot_verification.md`

<HARD-GATE id="HG-4">
以下の3層すべてで get_screenshot を実行し品質判定をパスするまで、
ReviewNote のステータスを「完了」に更新してはならない。
</HARD-GATE>

| Layer | get_screenshot 対象 | 確認観点 |
|-------|-------------------|---------|
| L3 Component | 修正ノード ID | 修正の正確さ、文字化け、切り詰め |
| L2 Frame | 親フレーム ID | 周辺との調和、余白バランス、色の統一感 |
| L1 Page | ページ全体 | 全体バランス、orphan ノード、他フレームとの整合 |

**品質判定 — 2軸チェック**:

設計観点 (D1-D5):
- D1: カラートークン準拠
- D2: タイポグラフィ準拠
- D3: スペーシング準拠（4px/8px grid）
- D4: ビジュアルヒエラルキー維持
- D5: アクセシビリティ（コントラスト≥4.5:1, タッチ≥44px）

ユーザー目線 (U1-U7):
- U1: 混乱リスク — 意味が一目で分かるか
- U2: レイアウトの自然さ — ターゲット層の期待に合うか
- U3: 情報優先度 — 最重要情報が最も目立つか
- U4: アフォーダンス — クリック可能に見えるか
- U5: 一体感 — 「継ぎはぎ」に見えないか
- U6: 日本語可読性 — 行間170%+, サイズ≥14px, 切り詰めなし
- U7: ブランド一貫性 — 同じプロダクトに見えるか

**Level B**: Before/After 比較スクリーンショットも提示
**失敗時**: 自動修復（最大2回）→ まだ失敗 → ユーザーに報告

## use_figma コード規約

全 `use_figma` 呼び出しで以下を遵守:

1. `await figma.setCurrentPageAsync(page)` でページ切替（G-003）
2. `loadFont(family, style)` with 3-step fallback（G-002）
3. fills/strokes は clone→modify→reassign（G-001）
4. appendChild → layoutSizing の順序（G-006）
5. `return JSON.stringify({createdNodeIds, mutatedNodeIds, success})`（G-008）

詳細 → `references/figma_api_gotchas.md`

## Reference Dispatch

| Step | Reference File | Read Timing |
|------|---------------|-------------|
| 1 | `references/context_loading.md` | Context Load 開始時 |
| 2 | `references/deep_scan.md` | Deep Scan 開始時 |
| 3 | `references/level_classification.md` | Classify 開始時 |
| 3 | `references/processing_algorithm.md` | Process 開始時（内部で検証・ステータス更新も参照） |
| 3 | `references/supported_modifications.md` | 修正適用時 |
| 3-6 | `references/figma_api_gotchas.md` | 全 use_figma 呼び出し前 |
| 4 | `references/aidesigner_integration.md` | Level B 修正時 |
| 5 | `references/design_change_report.md` | Report 生成時 |
| 6 | `references/screenshot_verification.md` | Verify 開始時 |

> Note: `processing_algorithm.md` は内部で `screenshot_verification.md` と `supported_modifications.md` を参照する。各参照はそのステップで個別に Read する。

## Supported Modifications

| Code | Category | Figma API |
|------|----------|-----------|
| M-001 | Color (fill/stroke) | fills, strokes (clone pattern) |
| M-002 | Text (content/size/weight) | characters, fontSize, fontName |
| M-003 | Size (width/height) | resize() (non-auto-layout only) |
| M-004 | Corner radius | cornerRadius |
| M-005 | Spacing (padding/gap) | paddingTop etc, itemSpacing |
| M-006 | Visibility | visible |
| M-007 | Opacity | opacity |

非対応: ノード追加/削除、画像差替、バリアント切替、エフェクト
Level B（構造改善）: AIDesigner で改善版生成 → ユーザー承認 → M-code で反映
