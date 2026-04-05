# Supported Modifications

> ReviewNote の修正指示を Figma API 操作にマッピングする。各 M-code に対応する Gotcha (G-xxx) も記載。

## M-001: Color (fill / stroke)

**トリガー語**: 色, カラー, 背景色, ボーダー色, fill, stroke, #HEX値

```js
// G-001: clone→modify→reassign 必須
function setFill(node, hex) {
  const rgb = hexToRgb(hex);
  const newFills = JSON.parse(JSON.stringify(node.fills));
  newFills[0] = { type: 'SOLID', color: rgb };
  node.fills = newFills;
}

function setStroke(node, hex) {
  const rgb = hexToRgb(hex);
  const newStrokes = JSON.parse(JSON.stringify(node.strokes));
  newStrokes[0] = { type: 'SOLID', color: rgb };
  node.strokes = newStrokes;
}

function hexToRgb(hex) {
  const h = hex.replace('#', '');
  return {
    r: parseInt(h.substring(0, 2), 16) / 255,
    g: parseInt(h.substring(2, 4), 16) / 255,
    b: parseInt(h.substring(4, 6), 16) / 255
  };
}
```

**Context 照合**: ui_design_brief.md の Colors Table と一致するか確認。パレット外の色を指定された場合はユーザーに警告。

## M-002: Text (content / size / weight)

**トリガー語**: テキスト, 文字, フォント, サイズ, ウェイト, Bold, 太字, 文言変更

**HARD-GATE**: HG-2 — 変更前に listAvailableFontsAsync でフォント確認。

```js
// Content change
await figma.loadFontAsync(node.fontName);  // G-002
node.characters = newText;

// Font size change
await figma.loadFontAsync(node.fontName);
node.fontSize = 16;

// Font weight change (G-002 fallback)
const newFont = await loadFont('Noto Sans JP', 'Bold');
node.fontName = newFont;
```

**Context 照合**: ubiquitous_language.md の UI Label カラムと一致するか確認。ドメイン用語が正しいラベルになっているか検証。

## M-003: Size (width / height)

**トリガー語**: 幅, 高さ, サイズ, width, height, 大きく, 小さく

```js
// Non-auto-layout nodes only (auto-layout は padding/constraints で調整)
node.resize(newWidth, newHeight);
// G-005: resize 後に sizing mode を再設定
// Non-auto-layout → FIXED, auto-layout 子 → FILL or HUG (G-005 参照)
node.layoutSizingHorizontal = 'FIXED';
node.layoutSizingVertical = 'FIXED';
```

**制約**: Auto-layout 親ノードには resize() を使わない。padding / constraints で調整する。

## M-004: Corner Radius

**トリガー語**: 角丸, radius, 丸み, rounded

```js
node.cornerRadius = 12;
// 個別角の場合:
node.topLeftRadius = 12;
node.topRightRadius = 12;
node.bottomLeftRadius = 0;
node.bottomRightRadius = 0;
```

## M-005: Spacing (padding / gap)

**トリガー語**: 余白, パディング, 間隔, スペース, gap, padding, margin

```js
// Padding (auto-layout frame only)
node.paddingTop = 16;
node.paddingBottom = 16;
node.paddingLeft = 24;
node.paddingRight = 24;

// Item spacing (gap between children)
node.itemSpacing = 12;
```

**Context 照合**: ui_design_brief.md の spacing tokens (4px grid) と一致するか確認。任意の値は警告。

## M-006: Visibility

**トリガー語**: 非表示, 表示, 隠す, 見せる, visible, hidden

```js
node.visible = false;  // hide
node.visible = true;   // show
```

## M-007: Opacity

**トリガー語**: 透明度, 不透明度, opacity, 透明, 半透明

```js
node.opacity = 0.5;  // 0.0 (transparent) to 1.0 (opaque)
```

---

## NOT Supported (明示的除外)

以下の操作は ReviewNote では対応しない。ユーザーにその旨を報告し、手動対応を促す。

| 操作 | 理由 |
|------|------|
| ノード追加 / 削除 | 構造変更は設計意図の確認が必要 |
| 画像差し替え | image hash の取得が必要 |
| バリアント切り替え | component set の構造理解が必要 |
| エフェクト (shadow, blur) | パラメータが複雑、視覚確認が必須 |
| レイアウトモード変更 | auto-layout ↔ absolute の切替はリスクが高い |

非対応の修正指示を検出した場合:
1. ReviewNote ステータスを「既読」に更新
2. TargetHint に「⚠ 非対応: [操作名]。手動で対応してください」を追記

## Intent Parsing Priority

修正指示テキストから M-code へのマッピング順序:

1. 明示的な M-code 参照 → 直接マッピング
2. HEX値 `#` 検出 → M-001
3. 数値 + `px` 検出 → M-003 or M-005 (padding/gap キーワードで分岐)
4. フォント関連キーワード → M-002
5. 上記以外のキーワードマッチ → トリガー語テーブルで照合
6. マッチなし → ユーザーに確認
