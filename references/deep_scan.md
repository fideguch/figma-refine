# Deep Scan Protocol

> 既存 Figma デザインの構造を把握し、修正判断の精度を上げる。
> Step 2 で必ず実行し、Step 3 の Classify に結果を渡す。

## 実行タイミング

ReviewNote 処理の前に、対象ファイル全体の構造を把握する。
Context Load (Step 1) で Design Personality を構築済みであること。

## スキャン手順

### 1. ファイル構造の把握

```
get_metadata(fileKey) → ページ一覧
各ページ:
  - ページ名
  - トップレベルフレーム一覧（名前、サイズ、位置）
  - DS-* ページ → Design System として記録
  - WF-* フレーム → Wireframe として記録
  - MK-* フレーム → Mockup として記録
```

**記録する情報**:
```
fileStructure:
  pages:        [{name, id, frameCount}]
  designSystem: [{componentName, variants}]  // DS- ページから
  wireframes:   [{frameName, id, size, childCount}]
  mockups:      [{frameName, id, size, childCount}]
```

### 2. デザインシステム解析

DS- ページが存在する場合:

```
get_design_context(fileKey, nodeId=DS-ページID)
```

| 抽出項目 | 用途 |
|---------|------|
| カラーパレット | Design Personality の brandColors と照合 |
| タイポグラフィスケール | Typography tokens の実測値 |
| コンポーネント一覧 | 修正時に既存コンポーネントを活用 |
| スペーシングトークン | 余白修正の基準値 |

### 3. 対象フレームの詳細スキャン

ReviewNote が存在するフレームに対して:

```
get_design_context(fileKey, nodeId=フレームID)
```

**記録する情報**:
```
frameDetail:
  layout:       "auto-layout" | "absolute" | "mixed"
  depth:        最大ネスト深度
  textNodes:    [{id, characters, fontName, fontSize}]
  colorUsage:   [{hex, count, nodeTypes}]  // 使用色の集計
  components:   [{name, instanceCount}]     // 使用コンポーネント
```

### 4. Before スクリーンショット取得

修正前の状態を記録する（Step 6: Verify の Before/After 比較用）。

```
各 ReviewNote の対象フレーム:
  get_screenshot(nodeId=フレームID, fileKey) → Before 画像として保持
```

> **MUST**: Before スクリーンショットは ReviewNote 処理開始前に取得すること。
> 処理後に取得しても Before にならない。

## スキャン結果の活用

| スキャン結果 | 活用先 |
|-------------|--------|
| fileStructure | ReviewNote のターゲット解決精度向上 |
| designSystem | Level A 修正時のトークン参照 |
| frameDetail | Level A/B 分類の判断材料 |
| Before SS | Step 6 の Before/After 比較 |

## designs/ との照合

Deep Scan 結果と Design Personality を突合する:

| 観点 | 照合内容 |
|------|---------|
| 色 | Figma 実測色 vs brandColors → 乖離を記録 |
| フォント | Figma 実測フォント vs typography → 不一致を記録 |
| 構造 | SC-XXX マッピング vs 実フレーム → 欠落を記録 |

乖離があっても修正しない（Deep Scan は観察のみ）。
乖離情報は Step 5: Report の Design Change Report に含める。
