# Figma Plugin API Gotchas (MCP Environment)

> use_figma コード記述時に必ず参照する。各 G-code を違反すると無言の失敗またはランタイムエラーが発生する。

## G-001: fills / strokes は読み取り専用配列

直接変更不可。clone → modify → reassign パターンを使う。

```js
// WRONG: node.fills[0].color = newColor;
// CORRECT:
const newFills = JSON.parse(JSON.stringify(node.fills));
newFills[0] = { type: 'SOLID', color: { r: 0.06, g: 0.73, b: 0.51 } };
node.fills = newFills;
```

strokes も同じパターン。`setBoundVariableForPaint()` は新しい paint を返すので再代入が必要。

## G-002: loadFontAsync はローカルフォントで失敗する

MCP 環境はサンドボックス。Figma にアップロード済みフォントのみ利用可能。

```js
async function loadFont(family, style) {
  // Step 1: Target font
  try {
    await figma.loadFontAsync({ family, style });
    return { family, style };
  } catch {}
  // Step 2: Inter fallback
  const interMap = { 'Regular': 'Regular', 'Medium': 'Medium', 'Bold': 'Bold' };
  try {
    await figma.loadFontAsync({ family: 'Inter', style: interMap[style] || 'Regular' });
    return { family: 'Inter', style: interMap[style] || 'Regular' };
  } catch {}
  // Step 3: First available sans-serif
  const fonts = await figma.listAvailableFontsAsync();
  const sans = fonts.find(f => f.fontName.style === 'Regular');
  if (sans) {
    await figma.loadFontAsync(sans.fontName);
    return sans.fontName;
  }
  throw new Error('No available fonts');
}
```

**Noto Sans JP の注意**: Semi Bold は存在しない。Regular / Medium / Bold のみ。

## G-003: currentPage の代入はエラー

```js
// WRONG: figma.currentPage = targetPage;
// CORRECT:
await figma.setCurrentPageAsync(targetPage);
```

## G-004: use_figma 呼び出しごとにコンテキストがリセットされる

変数は呼び出し間で保持されない。ノードを跨ぎ呼び出しで参照するには ID を返して次の呼び出しで再取得する。

```js
// Call 1: return JSON.stringify({ componentId: comp.id });
// Call 2: const comp = figma.getNodeById(inputComponentId);
```

## G-005: resize() は sizing mode を FIXED にリセットする

```js
node.resize(320, 48);
// resize 後に必ず再設定:
node.layoutSizingHorizontal = 'FILL';  // or 'HUG'
node.layoutSizingVertical = 'HUG';
```

Auto-layout 親には resize() を使わない。padding / itemSpacing で制御する。

## G-006: appendChild → layoutSizing の順序厳守

FILL は親が auto-layout を持ち、子が追加済みでないと設定できない。

```js
parent.appendChild(child);           // Step 1: 追加
child.layoutSizingHorizontal = 'FILL'; // Step 2: サイジング設定
```

逆順にするとエラー。

## G-007: figma.notify() は未実装

MCP 環境に UI 通知機能はない。ステータスは return 値で伝達する。

```js
// WRONG: figma.notify('Done!');
// CORRECT: return JSON.stringify({ status: 'success', message: 'Done' });
```

## G-008: 返却値契約

全 use_figma スクリプトは以下の形式で返却する:

```js
return JSON.stringify({
  createdNodeIds: [node1.id, node2.id],
  mutatedNodeIds: [existing.id],
  success: true,
  details: '...'  // optional
});
```

失敗時も JSON で返却し、スクリプト外でのエラーハンドリングを可能にする。
