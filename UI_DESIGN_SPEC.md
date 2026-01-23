# UIデザイン仕様書

## 1. カラーデザインのコンセプト定義

### コンセプト名

**"Green Flow Dashboard"**

### デザイン方針

- ベース：淡くやさしいグリーン（目に優しい）
- アクセント：オレンジの流線（動き・導線・おしゃれ感）
- 白を多く使い、情報は詰めすぎない
- 「役所っぽさ」「Excel感」を消す

### 色の意味

| 色 | 意味 |
|----|------|
| 緑 | 安心感・安定・業務ツール感 |
| オレンジ | 活力・動線・視線誘導 |

---

## 2. カラーパレット

### ベースカラー（背景）

```text
ライトグリーン     #EAF5EE
ホワイト           #FFFFFF
```

### メインカラー（UI要素）

```text
メイングリーン     #6FBF8E
ダークグリーン     #3E8E6D
```

### アクセント（流線・強調）

```text
アクセントオレンジ #F5A623
ソフトオレンジ     #FFD8A8
```

### テキスト

```text
メイン文字         #2E2E2E
サブ文字           #6B6B6B
```

> **重要**: オレンジは「使いすぎない」。
> **流線・区切り・視線誘導のみに限定**するのが上品。

---

## 3. UIパーツ別 色の割当仕様

### ヘッダー

- 背景：`#EAF5EE`
- タイトル文字：`#3E8E6D`
- 下部に **オレンジの流線（SVG or CSS）**

### ボタン（カード）

- 背景：`#FFFFFF`
- 枠線：`#6FBF8E`
- ホバー時：
  - 影を強く
  - 枠線 or 左ラインを `#F5A623`

### 強調カード（wide / tall）

- 左端に **オレンジの細ライン（4px）**
- 重要情報感を出す

---

## 4. 流線デザインの実装方法

### 方法① SVG（最もきれい・おすすめ）

```html
<svg viewBox="0 0 1440 80" preserveAspectRatio="none">
  <path d="M0,40 C240,80 480,0 720,20 960,40 1200,60 1440,20 L1440,0 L0,0 Z"
        fill="#F5A623" opacity="0.15"/>
</svg>
```

- ヘッダー下に配置
- opacity を下げて **主張しすぎない**

### 方法② CSS疑似要素（簡易）

```css
.header::after {
  content: "";
  display: block;
  height: 6px;
  background: linear-gradient(
    90deg,
    transparent,
    #F5A623,
    transparent
  );
  margin-top: 8px;
}
```

---

## 5. CSSサンプル

```css
body {
  background-color: #EAF5EE;
  font-family: "Segoe UI", "Hiragino Kaku Gothic ProN", sans-serif;
  color: #2E2E2E;
}

.header {
  background: #EAF5EE;
  padding: 16px 24px;
}

.header h1 {
  color: #3E8E6D;
}

.card {
  background: #FFFFFF;
  border: 1px solid #6FBF8E;
  border-radius: 12px;
  padding: 20px;
  transition: box-shadow 0.2s ease, transform 0.2s ease;
}

.card:hover {
  box-shadow: 0 6px 18px rgba(0,0,0,0.12);
  transform: translateY(-2px);
}

.card.wide,
.card.tall {
  border-left: 4px solid #F5A623;
}
```

---

## 6. デザインレビュー基準

- 緑が「濃すぎない」こと
- オレンジは **線・アクセントのみ**
- 影・余白で立体感を出す
- 文字が背景に埋もれていない
