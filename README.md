# figma-refine

> [English version](README.en.md)

プロジェクトの背景（ブランド、ターゲットユーザー、競合、ドメイン用語）を理解した上で、Figma デザインを洗練する Claude Code スキル。ReviewNote コンポーネントによる非同期デザインレビュー・修正ワークフローを提供する。

## v2.0 の新機能

- **Level A/B 分類**: プロパティ修正（Level A）と構造改善（Level B）を自動分類
- **AIDesigner MCP 統合**: Level B の修正で AIDesigner の `refine_design` を活用し、デザイン改善版を生成
- **Deep Scan**: 修正前に Figma ファイル全体の構造を把握し、Before スクリーンショットを取得
- **Design Change Report**: 修正内容と designs/ との乖離を分析し、更新推奨を出力
- **Before/After 比較**: Level B 修正の前後をスクリーンショットで比較
- **designs/ なし対応**: ユーザーに質問して Design Personality を手動構築

## 概要

- `designs/` フォルダからプロジェクトコンテキストを読み込み（Design Personality 構築）
- Figma 上の ReviewNote コンポーネントを検出し、修正意図を Level A/B に分類
- Level A: プロジェクト固有の文脈に基づいて Figma MCP で修正を適用
- Level B: AIDesigner で改善版デザインを生成 → ユーザー承認後に反映
- Design Change Report で designs/ との乖離を分析
- 3層スクリーンショット検証（Component → Frame → Page）× 2軸品質判定（設計観点 + ユーザー目線）

## インストール

```bash
git clone https://github.com/fideguch/figma-refine.git ~/.claude/skills/figma-refine
```

## 前提条件

- **Figma MCP**: Claude Code の Figma MCP server が接続済みであること
- **Figma file**: 修正対象の Figma Design ファイル URL
- **designs/ フォルダ** (推奨): `/requirements_designer` で生成されたプロジェクト要件
- **AIDesigner MCP** (Level B に必要): `/mcp` でブラウザサインイン

## 使い方

```
/figma-refine
```

トリガーワード: "レビューノート処理して", "デザイン修正", "デザイン洗練", "figma review"

## ワークフロー

```
Step 1: Context Load   — designs/ を読み込み、Design Personality を構築
                         designs/ なし → ユーザーに質問して手動構築
Step 2: Deep Scan      — Figma ファイル構造を把握、Before SS 取得
Step 3: Classify       — ReviewNote を Level A/B に分類 → Level A は修正適用
Step 4: AIDesigner     — Level B の修正を AIDesigner で改善 → ユーザー承認
Step 5: Report         — Design Change Report（乖離分析 + 更新推奨）
Step 6: Verify         — 3層 × 2軸でスクリーンショット検証 + Before/After
```

## Level A/B 分類

| Level | 対象 | 処理 |
|-------|------|------|
| **A** (プロパティ修正) | 色変更、テキスト修正、余白調整等 | M-001〜M-007 で直接修正 |
| **B** (構造改善) | レイアウト変更、抽象指示（"モダンに"等） | AIDesigner refine_design + ユーザー承認 |

## ReviewNote の配置方法

ReviewNote インスタンスを修正したい要素の近くに配置し、Content テキストに修正指示を記入してください。
TargetHint フィールドにターゲットのフレーム名やノード名を記入すると、Name Mode による解決精度が上がります。

## 権限モデル

| 操作 | 権限 |
|------|------|
| designs/ 読み取り | 自由 |
| designs/ 書き込み | **禁止** |
| designs/ 更新提案 | Design Change Report でユーザーに報告 |
| designs/ 実更新 | `/requirements_designer enhance` or ユーザー手動 |

## ファイル構成

```
figma-refine/
├── SKILL.md                              — オーケストレーター（6ステップ + 4 HARD-GATE）
├── references/
│   ├── context_loading.md                — designs/ 読み込み + designs/ なし時の質問フロー
│   ├── deep_scan.md                      — ファイル構造把握 + Before SS 取得
│   ├── level_classification.md           — Level A/B 分類ロジック
│   ├── review_note_component.md          — コンポーネント仕様 + Bootstrap コード
│   ├── processing_algorithm.md           — 検出 → ターゲット解決 → 修正適用
│   ├── supported_modifications.md        — M-001〜M-007 修正カタログ
│   ├── aidesigner_integration.md         — AIDesigner MCP 連携 + Before/After
│   ├── design_change_report.md           — 乖離分析 + レポート出力
│   ├── figma_api_gotchas.md              — G-001〜G-008 MCP環境の落とし穴
│   └── screenshot_verification.md        — 3層検証 + 2軸品質判定 + Before/After
├── README.md                             — 日本語ドキュメント（このファイル）
└── README.en.md                          — English documentation
```

## 関連スキル

| スキル | 関係 |
|--------|------|
| `/requirements_designer` | 上流: designs/ と Figma UI デザインを生成 |
| `/aidesigner-frontend` | 統合: Level B 修正で AIDesigner MCP を活用 |
| `/speckit-bridge` | 下流: designs/ をエンジニア向け仕様書に変換 |
| `/ui-ux-pro-max` | 補完: UI/UX デザイン品質評価 |
| `/frontend-design` | 補完: プロダクション品質のフロントエンド実装 |

## 対応修正タイプ

| コード | カテゴリ | 例 |
|--------|---------|-----|
| M-001 | 色 | "背景色を #1A1A2E に", "ボーダーを赤に" |
| M-002 | テキスト | "フォントサイズを 16px に", "テキストを Bold に" |
| M-003 | サイズ | "幅を 320px に" |
| M-004 | 角丸 | "角丸を 12px に" |
| M-005 | 余白 | "余白を 24px に", "gap を広げて" |
| M-006 | 表示/非表示 | "非表示にして" |
| M-007 | 不透明度 | "透明度を 50% に" |
| Level B | 構造改善 | "モダンに", "レイアウト変更", "プレミアムに" |

## 制限事項

- ノードの追加・削除は非対応（構造変更は手動対応が必要）
- 画像差し替えは image hash の取得が必要なため非対応
- バリアント切り替えは component set の理解が必要なため非対応
- エフェクト（影、ぼかし）はパラメータが複雑なため非対応
- designs/ への書き込みは禁止（レポートで乖離を報告のみ）

## ライセンス

MIT
