# Design Change Report

> 修正内容を構造化されたレポートとして出力し、
> designs/ との乖離を分析する。Step 5 で実行。

## レポートの目的

1. 修正内容の記録 — 何を、なぜ、どう変えたか
2. designs/ との乖離分析 — 修正により designs/ の記述と合わなくなった箇所の特定
3. 更新推奨 — `/requirements_designer enhance` での designs/ 更新を提案

## レポート出力先

レポートはユーザーの会話に直接出力する（ファイル書き出しはしない）。
ユーザーが保存を求めた場合のみ、プロジェクトルートに出力する。

## レポートフォーマット

```markdown
# Design Change Report

**ファイル**: {Figma file name} ({fileKey})
**日時**: {YYYY-MM-DD HH:MM}
**処理した ReviewNote**: {n} 件（Level A: {a} 件, Level B: {b} 件）

## 変更サマリ

| # | ReviewNote | Level | 対象 | 修正内容 | M-code |
|---|-----------|-------|------|---------|--------|
| 1 | "HeaderをBoldに" | A | WF-US001 > Header > Title | fontWeight: Regular → Bold | M-002 |
| 2 | "モダンに" | B | WF-US001 > Header | 背景色+余白+フォント変更 | M-001,002,005 |
| 3 | "非表示にして" | A | WF-US002 > Sidebar > Badge | visible: true → false | M-006 |

## designs/ 乖離分析

### 色の乖離

| 変更 | Figma 修正後 | designs/ palette | 状態 |
|------|-------------|-----------------|------|
| Header 背景 | #F8FAFC | brandColors に該当なし | ⚠ 乖離 |
| CTA ボタン | #2563EB | primary: #2563EB | ✓ 一致 |

### テキストの乖離

| 変更 | Figma 修正後 | designs/ UL | 状態 |
|------|-------------|------------|------|
| "ダッシュボード" | 変更なし | UL-001: ダッシュボード | ✓ 一致 |

### 構造の乖離

| 変更 | Figma 修正後 | designs/ FR/SC | 状態 |
|------|-------------|---------------|------|
| Header レイアウト | Flex space-between | FR-001: 横並び | ✓ 概ね一致 |
| Sidebar 非表示 | visible: false | SC-002: Sidebar あり | ⚠ 乖離 |

## 推奨アクション
```

## 乖離分析のロジック

### 色の照合

```
1. 修正で使用した全 HEX 値を抽出
2. Design Personality の brandColors と照合
3. マッチ: "✓ 一致" + パレット内の色名
4. 不一致: "⚠ 乖離" + 最も近いパレット色との距離
```

### テキストの照合

```
1. 修正で変更した全テキストを抽出
2. Design Personality の termMap (UL辞書) と照合
3. ドメイン用語が正しく使われているか確認
4. 不一致: "⚠ 乖離" + 正しいラベルの提案
```

### 構造の照合

```
1. 修正で変更したフレーム構造を抽出
2. functional_requirements.md の Screen Inventory と照合
3. 画面の構成要素が FR の記述と合致するか確認
4. 要素の追加/削除/非表示 → "⚠ 乖離"
```

## 推奨アクションの生成

乖離が検出された場合、以下の推奨を出力:

```markdown
## 推奨アクション

### designs/ の更新が必要な箇所

1. **ui_design_brief.md** — Colors Table に #F8FAFC (Slate-50) を追加
   → `/requirements_designer enhance` で更新推奨

2. **functional_requirements.md** — SC-002 の Sidebar 記述を更新
   → Sidebar 非表示オプションを AC に追加

### 更新不要

- テキスト変更: UL辞書と一致、乖離なし
- CTA 色変更: パレット内の primary 色を使用

### 更新方法

designs/ の更新は figma-refine のスコープ外です。
以下のいずれかで対応してください:
  1. `/requirements_designer enhance` — AI支援で designs/ を更新
  2. 手動編集 — 該当ファイルを直接修正
```

> **CRITICAL**: figma-refine は designs/ を書き換えない。
> レポートで乖離を報告し、更新はユーザーまたは /requirements_designer に委譲する。

## 乖離なしの場合

```markdown
## designs/ 乖離分析

✅ 全変更が designs/ の定義と一致しています。designs/ の更新は不要です。
```

## Before/After セクション（Level B のみ）

Level B の修正が含まれる場合、Before/After の比較を追加:

```markdown
## Before/After 比較

### ReviewNote #2: "モダンに" (Level B)

**Before**: [Deep Scan で取得したスクリーンショットの説明]
**After**: [修正後のスクリーンショットの説明]

**主な変更点**:
1. 背景色: 白 → Slate-50 (softer tone)
2. 余白: 16px → 24px (breathing room)
3. フォント: Regular → Medium (stronger hierarchy)
```
