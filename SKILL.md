---
name: figma-refine
description: >-
  Context-aware Figma design refinement. Reads designs/ for project identity
  (brand, users, competitors, domain terms), processes ReviewNote components
  to apply modifications, and verifies with 3-layer screenshot validation.
  Use after requirements_designer Phase 5 to polish UI designs.
  Triggers: "figma-refine", "デザイン洗練", "レビューノート処理", "デザイン修正",
  "review note", "figma review", "デザインレビュー", "ReviewNote"
---

# figma-refine

プロジェクト文脈を理解した上で Figma デザインを洗練するスキル。
`designs/` フォルダからブランド・ユーザー・競合・ドメイン用語を読み込み、
ReviewNote コンポーネントの指示に基づいて修正を適用する。

## Prerequisites

- Figma MCP 接続済み（`whoami` で確認）
- Figma Design ファイルの URL（fileKey を抽出）
- `designs/` フォルダ（推奨。なくても汎用モードで動作）

## 4-Step Workflow

### Step 1: Context Load

<HARD-GATE>
designs/ フォルダが存在する場合、Step 3 の ReviewNote 処理を開始する前に
必ず本ステップを完了すること。designs/ を読まずに修正を適用してはならない。
</HARD-GATE>

**Read** → `references/context_loading.md`

1. `designs/ui_design_brief.md` を読む — ブランド色、フォント、Mood、競合参照
2. `designs/README.md` を読む — ターゲットユーザー、制約、成功指標
3. `designs/ubiquitous_language.md` を読む — UIラベル辞書
4. Design Personality を構築（brandColors, mood, targetUser, termMap）
5. designs/ なし → 警告を表示し汎用モードで続行

### Step 2: Bootstrap

**Read** → `references/review_note_component.md`

1. `get_metadata` で Figma ファイルの Design System ページを確認
2. ReviewNote マスターコンポーネントが存在するか検索
3. なければ `use_figma` で作成（reference のコード例に従う）
4. `get_screenshot` で作成結果を確認

既にマスターコンポーネントが存在する場合はスキップ。

### Step 3: Process

**Read** → `references/processing_algorithm.md`
**Read** → `references/supported_modifications.md`
**Read** → `references/figma_api_gotchas.md`

各 ReviewNote（未読のみ）に対して:

1. **Detect**: 未読 ReviewNote を全ページから収集
2. **Resolve**: Name Mode でターゲット解決（SC-XXX, WF-USXXX, 「名前」, パス解決）
   - Name Mode 失敗 → Position Mode（get_metadata 座標ベース）
3. **Interpret**: 修正意図を M-code にマッピング
   - Design Personality と照合（パレット外の色→警告、UL用語で正規化）
4. **Apply**: `use_figma` で修正を実行
   - 全コード: G-001〜G-008 の gotcha パターンに従う
5. **Update**: 検証後にステータスを更新（Step 4 で判定）

<HARD-GATE>
テキスト修正 (M-002) を実行する前に、必ず listAvailableFontsAsync で
対象フォントの利用可能性を確認すること。確認なしのテキスト修正は禁止。
</HARD-GATE>

### Step 4: Verify

**Read** → `references/screenshot_verification.md`

<HARD-GATE>
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

**失敗時**: 自動修復（最大2回）→ まだ失敗 → ユーザーに報告

## use_figma コード規約

全 `use_figma` 呼び出しで以下を遵守:

1. `await figma.setCurrentPageAsync(page)` でページ切替（G-003）
2. `loadFont()` with 3-step fallback（G-002）
3. fills/strokes は clone→modify→reassign（G-001）
4. appendChild → layoutSizing の順序（G-006）
5. `return JSON.stringify({createdNodeIds, mutatedNodeIds, success})`（G-008）

詳細 → `references/figma_api_gotchas.md`

## Reference Dispatch

| Step | Reference File | Read Timing |
|------|---------------|-------------|
| 1 | `references/context_loading.md` | Context Load 開始時 |
| 2 | `references/review_note_component.md` | Bootstrap 開始時 |
| 3 | `references/processing_algorithm.md` | Process 開始時 |
| 3 | `references/supported_modifications.md` | 修正適用時 |
| 3-4 | `references/figma_api_gotchas.md` | 全 use_figma 呼び出し前 |
| 4 | `references/screenshot_verification.md` | Verify 開始時 |

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
