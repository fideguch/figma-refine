# Level Classification

> ReviewNote の修正指示を Level A（プロパティ修正）と Level B（構造改善）に分類する。
> Step 3 で各 ReviewNote に対して実行する。

## 分類フロー

```
ReviewNote の Content テキストを解析
  ├── Level A 判定 → 既存 M-001〜M-007 で処理
  └── Level B 判定 → AIDesigner 改善フロー（aidesigner_integration.md）
```

## Level A: プロパティ修正

単一プロパティの変更で完結する修正。既存の supported_modifications.md (M-001〜M-007) で対応可能。

### 判定条件（いずれか1つ以上に該当）

| # | 条件 | 例 |
|---|------|-----|
| A1 | HEX値が明示されている | "色を #1A1A2E に" |
| A2 | 具体的な数値 + 単位がある | "フォントサイズを 16px に", "余白を 24px に" |
| A3 | 単一プロパティのキーワード | "Bold に", "非表示にして", "角丸を 12px に" |
| A4 | M-code が明示されている | "M-001 で #FF0000 に" |
| A5 | トリガー語が1種類のみ | supported_modifications.md のトリガー語が1カテゴリのみマッチ |

### Level A の処理

```
1. processing_algorithm.md の Step 3: Interpret に従い M-code にマッピング
2. Design Personality と照合（パレット外警告、UL正規化等）
3. supported_modifications.md のコードパターンで use_figma 実行
4. screenshot_verification.md で 3層検証
```

## Level B: 構造改善

レイアウト変更やデザインの方向性変更など、単一プロパティ修正では対応できない修正。
AIDesigner MCP を使用して改善版デザインを生成する。

### 判定条件（いずれか1つ以上に該当）

| # | 条件 | 例 |
|---|------|-----|
| B1 | 抽象的なデザイン指示 | "モダンに", "プレミアムに", "もっとスッキリ" |
| B2 | レイアウト変更の示唆 | "カラム数を変えて", "配置を入れ替えて" |
| B3 | コンポーネント再構成 | "カードデザインにして", "リスト形式に変更" |
| B4 | 複数プロパティの同時変更 | "全体的にトーンを変えて"（色+フォント+余白） |
| B5 | デザインシステム変更 | "Material Design 風に", "iOS ネイティブ風に" |
| B6 | M-code マッピング不可 | supported_modifications.md のトリガー語に該当なし |

### Level B の処理

```
1. aidesigner_integration.md に従い AIDesigner refine_design を実行
2. Before/After をユーザーに提示
3. 承認 → Figma MCP use_figma で反映
4. 不承認 → refine_design で再改善 or 手動修正
```

## 分類の優先順位

1. **明示的 Level A**: HEX値・px値・M-code → 即座に Level A
2. **明示的 Level B**: "レイアウト変更", "構造を変えて" → 即座に Level B
3. **あいまい**: 上記に該当しない → 以下のヒューリスティクスで判定

### あいまい判定のヒューリスティクス

```
指示テキストを解析:
  1. トリガー語マッチ数をカウント
     - 1カテゴリのみ → Level A
     - 2カテゴリ以上 → Level B 候補
  2. 抽象度を判定
     - 具体値あり（#HEX, Npx, Bold等）→ Level A
     - 形容詞のみ（"洗練された", "見やすく"）→ Level B
  3. 影響範囲を推定
     - 単一ノード → Level A
     - フレーム全体 → Level B
```

## AIDesigner MCP 未接続時

AIDesigner MCP が利用できない場合、Level B の処理:

```
⚠ AIDesigner MCP が接続されていません。
  Level B の修正には AIDesigner が必要です。
  以下の選択肢があります:
  1. /mcp で AIDesigner を接続して再試行
  2. 具体的な修正指示に分解して Level A として処理
  3. 手動で対応
```

Level A のみで動作する「Lite Mode」として処理を継続する。
