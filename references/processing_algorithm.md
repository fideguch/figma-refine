# ReviewNote Processing Algorithm

> 検出 → ターゲット解決 → 修正解釈 → 適用 → ステータス更新。
> 各ステップで context_loading.md の Design Personality を参照する。

## Overview

```
Step 1: Detect    — 未読 ReviewNote を全ページから収集
Step 2: Resolve   — Name Mode → Position Mode でターゲットノード特定
Step 3: Interpret — 修正意図を M-code にマッピング、コンテキスト照合
Step 4: Apply     — use_figma で修正を実行
Step 5: Verify    — 3層スクリーンショット検証 (screenshot_verification.md)
Step 6: Update    — ReviewNote ステータスを更新
```

## Step 1: Detect ReviewNotes

```
1. get_metadata(fileKey) でページ一覧を取得
2. 各ページの子ノードを走査
3. ReviewNote コンポーネントインスタンスを検出
   - 判定基準: name === "ReviewNote" or parent component name === "ReviewNote"
4. StatusBadge の StatusText.characters === "未読" のものだけ収集
5. 収集結果: [{noteId, content, targetHint, position, pageId}]
```

未読ノートが0件の場合:
```
✅ 未読の ReviewNote はありません。すべて処理済みです。
```

## Step 2: Target Resolution

### Name Mode（優先 — 推奨）

ReviewNote の Content テキストと TargetHint を正規表現でパースする。

| Priority | Pattern | Example | Resolution |
|----------|---------|---------|------------|
| 1 | `SC-(\d{3})` | "SC-001のHeaderを赤に" | Screen Inventory → WF-/MK- フレーム検索 |
| 2 | `(WF\|MK)-US\d+` | "WF-US001の余白を広げて" | フレーム名で直接マッチ |
| 3 | `「(.+?)」` | "「ダッシュボード」のSidebar" | 括弧内をノード名として findOne 検索 |
| 4 | TargetHint field | "対象: WF-US003 Header" | TargetHint テキストをそのまま使用 |

### Path Traversal（子ノード解決）

「の」パーティクルで分割してノードツリーを走査する:

```
"SC-001のHeaderのロゴをBoldに"
  → segments: ["SC-001", "Header", "ロゴ"]
  → action: "Boldに"

1. SC-001 → functional_requirements.md の Screen Inventory → US-001
2. ページ内で "WF-US001" or "MK-US001" プレフィックスのフレームを検索
3. フレーム → children → "Header" を名前マッチ
4. Header → children → "ロゴ" を名前マッチ（部分一致可）
5. 最終ノード = ターゲット、"Boldに" = 修正指示
```

requirements_designer が生成する標準子ノード名:
`Layout, Header, Sidebar, Content, Footer, Card N, Field — Label, Row N`

### Position Mode（フォールバック）

Name Mode でマッチしなかった場合のみ使用。

```
1. ReviewNote の absoluteBoundingBox を取得
2. 中心点を計算: (x + width/2, y + height/2)
3. 同一ページの WF-/MK- プレフィックスフレームのみ検索対象
   （DS- ページは除外）
4. 各候補ノードの中心点との Euclidean 距離を計算
5. フィルタ: distance < 2000px
6. ソート: 距離昇順
```

**あいまい判定**:
```
if (distances[1] - distances[0]) / distances[0] < 0.20:
  → あいまい: ユーザーに確認
    "⚠ 2つのフレームが近い距離にあります:
     1. WF-US001 (距離: 150px)
     2. WF-US002 (距離: 170px)
     どちらが対象ですか?"
else:
  → distances[0] のノードを採用
```

距離 2000px 超: "⚠ 近くにターゲットが見つかりません。TargetHint を指定してください。"

## Step 3: Interpret Modification

1. **Intent extraction**: Content テキストから修正意図を抽出
2. **M-code mapping**: supported_modifications.md の Intent Parsing Priority に従う
3. **Context validation**: Design Personality と照合

```
Context照合チェックリスト:
[ ] 色指定 → brandColors パレットに含まれるか
[ ] テキスト → termMap (UL辞書) で正規化
[ ] 余白 → spacing tokens (4px grid) に合致するか
[ ] 抽象指示 → mood / competitiveRefs から具体値を導出
[ ] Trust Design → P1-P7 パターンを損なわないか
```

**非対応の修正を検出した場合**:
- ステータスを「既読」に更新
- TargetHint に `⚠ 非対応: [操作名]` を追記
- ユーザーに手動対応を報告

## Step 4: Apply Modification

```
1. HARD-GATE HG-3: テキスト修正の場合、figma.listAvailableFontsAsync() を実行
2. use_figma コードブロック構成:
   a. Preamble: await figma.setCurrentPageAsync(targetPage)  // G-003
   b. Font loading: loadFont() with fallback chain            // G-002
   c. Node retrieval: figma.getNodeById(targetNodeId)          // G-004
   d. Modification: supported_modifications.md のコード例      // G-001, G-005, G-006
   e. Return: JSON.stringify({mutatedNodeIds, success})        // G-008
3. レスポンスの success フィールドを確認
4. success: false → エラー報告、ステータス「既読」で中断
```

## Step 5: Verify

→ screenshot_verification.md に従って 3層検証を実行

## Step 6: Update Status

検証結果に基づいてステータスを更新:

| 検証結果 | 新ステータス | StatusBadge色 |
|---------|------------|--------------|
| 3層全パス | 完了 | #047857 (green, WCAG AA) |
| 修正適用済み・検証中 | 既読 | #1D4ED8 (blue, WCAG AAA) |
| 非対応 or エラー | 既読 + 注記 | #1D4ED8 (blue, WCAG AAA) |

ステータス更新の use_figma コード:
```js
// G-001: fills は clone→modify→reassign
const badge = figma.getNodeById(badgeId);
const newFills = JSON.parse(JSON.stringify(badge.fills));
newFills[0] = { type: 'SOLID', color: { r: 0.02, g: 0.47, b: 0.34 } }; // #047857 完了=green (WCAG AA 5.8:1)
badge.fills = newFills;

// StatusText 更新
const statusText = figma.getNodeById(statusTextId);
await figma.loadFontAsync(statusText.fontName);
statusText.characters = '完了';
statusText.fills = [{ type: 'SOLID', color: { r: 0.02, g: 0.47, b: 0.34 } }]; // #047857
```

## Batch Processing

複数の ReviewNote がある場合の処理順序:

1. **ページ別にグループ化**: Wireframes ページ → Mockups ページ
2. **フレーム別にグループ化**: 同一フレーム内のノートをまとめる
3. **フレーム内の処理順**: 上→下、左→右（y座標 → x座標ソート）
4. **フレーム内の全ノート処理後**: L2 (Frame) + L1 (Page) 検証
5. **次のフレームへ移動**

同一フレームに複数ノートがある場合、L3 検証はノートごと、L2/L1 はフレーム完了後にまとめて実行する。
