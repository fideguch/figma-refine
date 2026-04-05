# figma-refine

> [English version](README.en.md)

プロジェクトの背景（ブランド、ターゲットユーザー、競合、ドメイン用語）を理解した上で、Figma デザインを洗練する Claude Code スキル。ReviewNote コンポーネントによる非同期デザインレビュー・修正ワークフローを提供する。

## 概要

- `designs/` フォルダからプロジェクトコンテキストを読み込み（Design Personality 構築）
- Figma 上の ReviewNote コンポーネントを検出し、修正意図を解釈
- プロジェクト固有の文脈に基づいて Figma MCP で修正を適用
- 3層スクリーンショット検証（Component → Frame → Page）× 2軸品質判定（設計観点 + ユーザー目線）

## 特徴

**コンテキスト駆動**: 「AIらしい汎用UI」ではなく、プロジェクトのブランドカラー、Mood、競合参照、ターゲットユーザーに基づいた独自性のあるデザイン判断を行う。

**3層 × 2軸検証**: 修正後のスクリーンショットをコンポーネント・フレーム・ページの3層で取得し、設計観点（トークン準拠、アクセシビリティ）とユーザー目線（違和感、自然さ、ブランド一貫性）の2軸で品質を判定する。

## インストール

```bash
git clone https://github.com/fideguch/figma-refine.git ~/.claude/skills/figma-refine
```

## 前提条件

- **Figma MCP**: Claude Code の Figma MCP server が接続済みであること
- **Figma file**: 修正対象の Figma Design ファイル URL
- **designs/ フォルダ** (推奨): `/requirements_designer` で生成されたプロジェクト要件

## 使い方

```
/figma-refine
```

トリガーワード: "レビューノート処理して", "デザイン修正", "デザイン洗練", "figma review"

## ワークフロー

```
Step 1: Context Load   — designs/ を読み込み、Design Personality を構築
Step 2: Bootstrap      — ReviewNote マスターコンポーネントを作成（初回のみ）
Step 3: Process        — 未読 ReviewNote を検出 → 修正適用
Step 4: Verify         — 3層 × 2軸でスクリーンショット検証
```

## ReviewNote の配置方法

ReviewNote インスタンスを修正したい要素の近くに配置し、Content テキストに修正指示を記入してください。
TargetHint フィールドにターゲットのフレーム名やノード名を記入すると、Name Mode による解決精度が上がります。
Position Mode（フォールバック）は ReviewNote の位置からターゲットを推定するため、対象要素の近くに配置することが重要です。

## ファイル構成

```
figma-refine/
├── SKILL.md                              — オーケストレーター（4ステップ + HARD-GATE）
├── references/
│   ├── context_loading.md                — designs/ 読み込みプロトコル
│   ├── review_note_component.md          — コンポーネント仕様 + Bootstrap コード
│   ├── processing_algorithm.md           — 検出 → ターゲット解決 → 修正適用
│   ├── supported_modifications.md        — M-001〜M-007 修正カタログ
│   ├── figma_api_gotchas.md              — G-001〜G-008 MCP環境の落とし穴
│   └── screenshot_verification.md        — 3層検証 + 2軸品質判定
├── README.md                             — 日本語ドキュメント（このファイル）
└── README.en.md                          — English documentation
```

## 関連スキル

| スキル | 関係 |
|--------|------|
| `/requirements_designer` | 上流: designs/ と Figma UI デザインを生成 |
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

## 制限事項

- ノードの追加・削除は非対応（構造変更は手動対応が必要）
- 画像差し替えは image hash の取得が必要なため非対応
- バリアント切り替えは component set の理解が必要なため非対応
- エフェクト（影、ぼかし）はパラメータが複雑なため非対応

## ライセンス

MIT
