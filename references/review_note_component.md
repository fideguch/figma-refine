# ReviewNote Component Specification

> ReviewNote コンポーネントの視覚仕様と Bootstrap 手順。

## Visual Specification

```
ReviewNote (COMPONENT) — Auto-Layout: VERTICAL, HUG/HUG
│ background: #FFFBEB, stroke: #F59E0B 2px INSIDE, cornerRadius: 8
│ padding: 12 all sides, itemSpacing: 8
│
├── Header (FRAME) — Auto-Layout: HORIZONTAL, HUG/HUG
│   │ itemSpacing: 8, counterAxisAlignItems: CENTER
│   │
│   ├── StatusBadge (FRAME) — Auto-Layout: HORIZONTAL, HUG/HUG
│   │   │ cornerRadius: 12, padding: 2 top/bottom, 8 left/right
│   │   └── StatusText (TEXT) — "未読"
│   │       font: Inter Medium 11px, textAutoResize: WIDTH_AND_HEIGHT
│   │
│   └── NoteLabel (TEXT) — "Review Note"
│       font: Inter Medium 11px, color: #92400E
│       textAutoResize: WIDTH_AND_HEIGHT
│
└── Body (FRAME) — Auto-Layout: VERTICAL, HUG/HUG
    │ itemSpacing: 4
    │
    ├── Content (TEXT) — ユーザーが記入する修正指示テキスト
    │   font: Noto Sans JP Regular 14px, color: #1F2937
    │   lineHeight: { value: 170, unit: 'PERCENT' }
    │   textAutoResize: HEIGHT (幅は親の FILL に従う)
    │
    └── TargetHint (TEXT) — "対象: [自動検出]" or ユーザー指定
        font: Noto Sans JP Regular 12px, color: #6B7280
        textAutoResize: WIDTH_AND_HEIGHT
```

## Status Badge Variants

| Status | Label | Badge BG | Text Color | Contrast | Use Case |
|--------|-------|----------|------------|----------|----------|
| Unread | 未読 | #FFF7ED | #B45309 | 5.2:1 AA | 新規配置、未処理 |
| Read | 既読 | #EFF6FF | #1D4ED8 | 7.8:1 AAA | 処理中 or 修正適用済み・検証待ち |
| Done | 完了 | #F0FDF4 | #047857 | 5.8:1 AA | 3層検証パス済み |

## Bootstrap Procedure (use_figma)

Design System ページに ReviewNote マスターコンポーネントを作成する。

```js
// 1. ページ切替
const pages = figma.root.children;
let dsPage = pages.find(p => p.name === 'Design System');
if (!dsPage) {
  dsPage = figma.createPage();
  dsPage.name = 'Design System';
}
await figma.setCurrentPageAsync(dsPage);

// 2. フォント読み込み (G-002 fallback)
let bodyFont = { family: 'Noto Sans JP', style: 'Regular' };
let labelFont = { family: 'Inter', style: 'Medium' };
try { await figma.loadFontAsync(bodyFont); } catch {
  bodyFont = { family: 'Inter', style: 'Regular' };
  await figma.loadFontAsync(bodyFont);
}
await figma.loadFontAsync(labelFont);

// 3. マスターコンポーネント作成
const comp = figma.createComponent();
comp.name = 'ReviewNote';
comp.layoutMode = 'VERTICAL';
comp.primaryAxisSizingMode = 'AUTO';
comp.counterAxisSizingMode = 'AUTO';
comp.paddingTop = comp.paddingBottom = comp.paddingLeft = comp.paddingRight = 12;
comp.itemSpacing = 8;
comp.fills = [{ type: 'SOLID', color: { r: 1, g: 0.98, b: 0.92 } }];
comp.strokes = [{ type: 'SOLID', color: { r: 0.96, g: 0.62, b: 0.04 } }];
comp.strokeWeight = 2;
comp.strokeAlign = 'INSIDE';
comp.cornerRadius = 8;

// 4. Header 行
const header = figma.createFrame();
header.name = 'Header';
header.layoutMode = 'HORIZONTAL';
header.primaryAxisSizingMode = 'AUTO';
header.counterAxisSizingMode = 'AUTO';
header.counterAxisAlignItems = 'CENTER';
header.itemSpacing = 8;
header.fills = [];
comp.appendChild(header);

// 4a. StatusBadge
const badge = figma.createFrame();
badge.name = 'StatusBadge';
badge.layoutMode = 'HORIZONTAL';
badge.primaryAxisSizingMode = 'AUTO';
badge.counterAxisSizingMode = 'AUTO';
badge.paddingTop = badge.paddingBottom = 2;
badge.paddingLeft = badge.paddingRight = 8;
badge.cornerRadius = 12;
badge.fills = [{ type: 'SOLID', color: { r: 1, g: 0.97, b: 0.93 } }];
header.appendChild(badge);

const statusText = figma.createText();
statusText.name = 'StatusText';
await figma.loadFontAsync(labelFont);
statusText.fontName = labelFont;
statusText.fontSize = 11;
statusText.characters = '未読';
statusText.fills = [{ type: 'SOLID', color: { r: 0.71, g: 0.33, b: 0.04 } }]; // #B45309 (WCAG AA 5.2:1)
badge.appendChild(statusText);

// 4b. NoteLabel
const noteLabel = figma.createText();
noteLabel.name = 'NoteLabel';
noteLabel.fontName = labelFont;
noteLabel.fontSize = 11;
noteLabel.characters = 'Review Note';
noteLabel.fills = [{ type: 'SOLID', color: { r: 0.57, g: 0.25, b: 0.05 } }];
header.appendChild(noteLabel);

// 5. Body
const body = figma.createFrame();
body.name = 'Body';
body.layoutMode = 'VERTICAL';
body.primaryAxisSizingMode = 'AUTO';
body.counterAxisSizingMode = 'AUTO';
body.itemSpacing = 4;
body.fills = [];
comp.appendChild(body);

const content = figma.createText();
content.name = 'Content';
content.fontName = bodyFont;
content.fontSize = 14;
content.characters = '修正指示をここに記入';
content.fills = [{ type: 'SOLID', color: { r: 0.12, g: 0.16, b: 0.22 } }];
content.lineHeight = { value: 170, unit: 'PERCENT' };
content.textAutoResize = 'HEIGHT';
body.appendChild(content);
content.layoutSizingHorizontal = 'FILL';

const hint = figma.createText();
hint.name = 'TargetHint';
hint.fontName = bodyFont;
hint.fontSize = 12;
hint.characters = '対象: [自動検出]';
hint.fills = [{ type: 'SOLID', color: { r: 0.42, g: 0.45, b: 0.50 } }];
body.appendChild(hint);

// 6. Return
return JSON.stringify({
  createdNodeIds: [comp.id],
  componentKey: comp.key,
  success: true
});
```

## Instance Creation

```js
// masterComponent は getNodeById or findOne で取得済み
const instance = masterComponent.createInstance();
instance.x = targetNode.x + targetNode.width + 20;
instance.y = targetNode.y - 20;
```

ステータス変更は StatusBadge の fills と StatusText の characters を更新する。
fills 更新時は G-001 clone パターンを使用する。
