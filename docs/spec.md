<!-- docs/spec_en.md の日本語訳 | version: alpha -->
<!-- 英語版は spec.mdx + spec-config.ts から生成されます。英語版の変更時はこの翻訳も更新してください。 -->

[English](spec_en.md)

# DESIGN.md フォーマット

DESIGN.md は、デザインシステムを自己完結したプレーンテキストで表現するフォーマットです。ブランドとプロダクトのビジュアルアイデンティティを定義し、複数のデザインセッションや異なる AI エージェント、ツールの間でも一貫したスタイルを適用できるようにします。人間が読めるオープンフォーマットの文書として、人間と AI の両方が理解し、改善できる継続的な信頼できる情報源となります。

DESIGN.md ファイルは、任意の YAML フロントマターと Markdown 本文の 2 つの部分で構成されます。YAML フロントマターには、機械可読なデザイントークンを記述します。Markdown 本文の各セクションには、人間が読めるデザインの意図とガイダンスを記述します。説明文では、体系的なトークン名（例: `primary`）に対応する説明的な色名（例:「Midnight Forest Green」）を使用できます。トークンは規範となる値であり、説明文はその値を適用するための文脈を提供します。

# デザイントークン

DESIGN.md には、構造化された形式でデザイントークンを埋め込むことができます。デザイントークンの記述には、[Design Token JSON 仕様](https://www.designtokens.org/tr/2025.10/format/#abstract)から着想を得た仕組みを使用します。具体的には、型付きトークングループ（colors、typography、spacing）の概念と、値を相互参照するための `{path.to.token}` 構文を採用しています。

これらのトークンは、`tokens.json`、Figma Variables、Tailwind テーマ設定との間で容易に相互変換できます。

デザイントークンは、ファイル先頭の YAML フロントマターとして埋め込みます。フロントマターブロックは、`---` だけを含む行で始まり、`---` だけを含む行で終わる必要があります。この区切り文字の間にある YAML は、以下で定義するスキーマに従って解析されます。

例:

```yaml
---
version: alpha
name: Daylight Prestige
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 48px
    fontWeight: 600
    lineHeight: 1.1
    letterSpacing: -0.02em
---
```

## スキーマ

フロントマターで定義するデザイントークンのスキーマは次のとおりです。

```yaml
version: <string>          # 任意、現在のバージョン: "alpha"
name: <string>
description: <string>      # 任意
colors:
  <token-name>: <Color>
typography:
  <token-name>: <Typography>
rounded:
  <scale-level>: <Dimension>
spacing:
  <scale-level>: <Dimension | number>
components:
  <component-name>:
    <token-name>: <string|token reference>
```

`<scale-level>` プレースホルダーは、サイズまたはスペーシングのスケールにおける名前付きレベルを表します。一般的なレベル名には `xs`、`sm`、`md`、`lg`、`xl`、`full` があります。説明的な任意の文字列キーを使用できます。

**Color**: 有効な CSS カラー文字列を色の値として使用できます。対応する形式は次のとおりです。

- Hex: `#RGB`、`#RGBA`、`#RRGGBB`、`#RRGGBBAA`
- 名前付きカラー: `red`、`cornflowerblue`、`transparent`
- 関数形式: `rgb()`、`rgba()`、`hsl()`、`hsla()`、`hwb()`
- 広色域: `oklch()`、`oklab()`、`lch()`、`lab()`
- 混色: `color-mix(in srgb, ...)`

WCAG コントラストの検証では、すべての色の値を内部的に sRGB へ変換します。表示とエクスポートでは元の形式を保持します。

単純で幅広いツールに対応できるため、引き続き Hex 表記（`#RRGGBB`）を推奨します。

- `fontFamily`（string）
- `fontSize`（Dimension）
- `fontWeight`（number）— 数値のフォントウェイト（例: `400`、`700`）。YAML では裸の数値または引用符で囲んだ文字列のどちらでも記述でき、両者は同等です。
- `lineHeight`（Dimension | number）— Dimension（例: `24px`、`1.5rem`）または単位なしの数値（例: `1.6`）を受け付けます。単位なしの数値は要素の `fontSize` に対する倍率を表し、CSS で推奨される方法です。
- `letterSpacing`（Dimension）
- `fontFeature`（string）— [`font-feature-settings`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-feature-settings) を設定します。
- `fontVariation`（string）— [`font-variation-settings`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-variation-settings) を設定します。

**Dimension**: 単位の接尾辞を持つ文字列です。有効な単位は px、em、rem です。

**トークン参照**: トークン参照は波括弧で囲み、YAML ツリー内の別の値を指すオブジェクトパスを含める必要があります。ほとんどのトークングループでは、参照先はグループ（例: `colors`）ではなくプリミティブ値（例: `colors.primary-60`）である必要があります。`components` セクション内では、複合値（例: `{typography.label-md}`）への参照も許可されます。

# セクション

すべての `DESIGN.md` は同じ構造に従います。プロジェクトに関係のないセクションは省略できますが、記載するセクションは以下の順序で並べます。すべてのセクションには `<h2>`（`##`）見出しを使用します。文書タイトル用に任意の `<h1>` 見出しを記載できますが、セクションとしては解析されません。

### セクションの順序

1. **Overview**（別名:「Brand & Style」）
2. **Colors**
3. **Typography**
4. **Layout**（別名:「Layout & Spacing」）
5. **Elevation & Depth**（別名:「Elevation」）
6. **Shapes**
7. **Components**
8. **Do's and Don'ts**

### 説明文とトークン

## Overview

「Brand & Style」とも呼ばれます。

このセクションでは、プロダクトのルック＆フィールを包括的に説明します。ブランドの個性、対象ユーザー、UI が喚起すべき感情を定義します。たとえば、遊び心があるかプロフェッショナルか、情報密度が高いか余白が広いか、といった性質です。特定のルールやトークンが明示されていない場合に、エージェントが高いレベルのスタイル判断を行うための基礎的な文脈となります。

## Colors

このセクションでは、デザインシステムのカラーパレットを定義します。

少なくとも `primary` カラーパレットを定義する必要があり、必要に応じて追加のカラーパレットを定義できます。

複数のカラーパレットがある場合、デザインシステムは各パレットにセマンティックな役割を割り当てられます。一般的な命名規則では、`primary`、`secondary`、`tertiary`、`neutral` の順にパレットへ名前を付けます。

例:

```markdown
## Colors

The palette is rooted in high-contrast neutrals and a single, evocative accent color.

- **Primary (#1A1C1E):** A deep ink used for headlines and core text to provide
  maximum readability and a sense of permanence.
- **Secondary (#6C7278):** A sophisticated slate used primarily for utilitarian
  elements like borders, captions, and metadata.
- **Tertiary (#B8422E):** A vibrant earthy red as the sole driver for
  interaction, used exclusively for primary actions and critical highlights.
- **Neutral (#F7F5F2):** A warm limestone that serves as the foundation for all
  pages, providing a softer, more organic feel than pure white.
```

### デザイントークン

`colors` セクションでは、すべてのカラーデザイントークンを定義します。カラートークンは、Markdown の説明文で定義した主要なカラーパレットから導出します。カラーパレットからカラートークンへの正確な対応付けには、一貫性のある任意の命名規則を使用できます。

これは、カラートークン名を値へ対応付ける map\<string, Color> です。

```yaml
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
```

## Typography

このセクションでは、タイポグラフィのレベルを定義します。

多くのデザインシステムには 9〜15 のタイポグラフィレベルがあります。デザインシステムは、各タイポグラフィレベルの役割を規定できます。

一般的な命名規則では、`headline`、`display`、`body`、`label`、`caption` などのセマンティックなカテゴリを使用します。各カテゴリは、`small`、`medium`、`large` などの異なるサイズにさらに分けることができます。

例:

```markdown
## Typography

The typography strategy leverages two distinct weights of **Public Sans** for
the narrative and **Space Grotesk** for technical data.

- **Headlines:** Set in Public Sans Semi-Bold to establish an institutional
  and trustworthy voice.
- **Body:** Public Sans Regular at 16px ensures contemporary professionalism
  and long-form readability.
- **Labels:** Space Grotesk is used for all technical data, timestamps, and
  metadata. Its geometric construction evokes the precision of a digital
  stopwatch. Labels are strictly uppercase with generous letter spacing.
```

### デザイントークン

`typography` セクションでは、タイポグラフィデザイントークンの正確なフォントプロパティを定義します。

これは map\<string, Typography> です。

```yaml
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 48px
    fontWeight: 600
    lineHeight: 1.1
    letterSpacing: -0.02em
  body-md:
    fontFamily: Public Sans
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.6
  label-caps:
    fontFamily: Space Grotesk
    fontSize: 12px
    fontWeight: 500
    lineHeight: 1
    letterSpacing: 0.1em
```

## Layout

「Layout & Spacing」とも呼ばれます。

このセクションでは、レイアウトとスペーシングの方針を説明します。

多くのデザインシステムはグリッドベースのレイアウトに従います。Liquid Glass のように、マージン、セーフエリア、動的なパディングを使用するものもあります。

例:

```markdown
## Layout

The layout follows a **Fluid Grid** model for mobile devices and a
**Fixed-Max-Width Grid** for desktop (max 1200px).

A strict 8px spacing scale (with a 4px half-step for micro-adjustments) is used to maintain a consistent rhythm. Components are grouped using "containment" principles, where related items are housed in cards with generous internal padding (24px) to emphasize the soft, approachable nature of the brand.
```

### デザイントークン

spacing セクションでは、スペーシングのデザイントークンを定義します。レイアウトモデルの実装に役立つスペーシング単位を含めることができます。たとえば、固定グリッドレイアウトでは、列のスパン、ガター、マージン用のスペーシング単位を定義できます。

これは、スペーシングスケールの識別子を寸法値または単位なしの数値（例: 列数や比率）へ対応付ける map\<string, Dimension | number> です。

```yaml
spacing:
  base: 16px
  xs: 4px
  sm: 8px
  md: 16px
  lg: 32px
  xl: 64px
  gutter: 24px
  margin: 32px
```

## Elevation & Depth

「Elevation」とも呼ばれます。

このセクションでは、デザインスタイルに基づいて視覚的な階層を伝える方法を説明します。エレベーションを使用する場合は、必要なスタイル（広がり、ぼかし、色）を定義します。フラットデザインの場合は、視覚的な階層を伝える代替手段（例: ボーダー、色のコントラスト）を説明します。

例:

```markdown
## Elevation & Depth

Depth is achieved through **Tonal Layers** rather than heavy shadows. The
background uses a soft off-white or very light green, while primary content sits on pure white cards.
```

## Shapes

このセクションでは、視覚要素の形状を説明します。

例:

```markdown
## Shapes

The shape language is defined by **Architectural Sharpness**. All interactive
elements, containers, and inputs utilize a minimal **4px corner radius**. This
provides just enough softness to feel modern while maintaining a rigid,
engineered aesthetic.
```

### デザイントークン

`rounded` セクションでは、ボタン、カード、その他の矩形に使用する角丸のデザイントークンを定義します。

これは map\<string, Dimension> です。

```yaml
rounded:
  sm: 4px
  md: 8px
  lg: 12px
  full: 9999px
```

## Components

このセクションでは、デザインシステム内のコンポーネントアトムに対するスタイルガイダンスを提供します。一般的なコンポーネントの種類は次のとおりです。デザインシステムでは、その領域に関連する追加のコンポーネントを定義することを推奨します。

* **Buttons**: primary、secondary、tertiary のバリアント（サイズ、パディング、状態を含む）。
* **Chips**: 選択チップ、フィルターチップ、アクションチップ。
* **Lists**: リスト項目、区切り線、先頭／末尾の要素のスタイル。
* **Tooltips**: 位置、色、タイミング。
* **Checkboxes**: checked、unchecked、indeterminate の状態。
* **Radio buttons**: selected、unselected の状態。
* **Input fields**: テキスト入力、テキストエリア、ラベル、ヘルパーテキスト、エラー状態。

> **注:** コンポーネント仕様は現在も活発に進化しています。現行の構造は、仕様が成熟するまでの間、領域固有のコンポーネント定義に意図的な柔軟性を持たせています。

### デザイントークン

components セクションでは、一般的なコンポーネントの一貫したスタイルを保証するために使用するデザイントークンの集合を定義します。これは、コンポーネント識別子をサブトークン名と値のグループへ対応付ける map\<string, map\<string, string>> です。デザイントークンの値には、リテラル値または先に定義したデザイントークンへの参照を使用できます。

**バリアント**: コンポーネントには、active、hover、pressed などの異なる UI 状態に対応するバリアントを設定できます。これらのバリアントコンポーネントは、たとえば「button-primary」「button-primary-hover」「button-primary-active」のように、異なる関連キーの下に定義できます。エージェントはすべてのバリアントを考慮し、適切なスタイルを判断します。

```yaml
components:
  button-primary:
    backgroundColor: "{colors.primary-60}"
    textColor: "{colors.primary-20}"
    rounded: "{rounded.md}"
    padding: 12px
  button-primary-hover:
    backgroundColor: "{colors.primary-70}"
```

### コンポーネントプロパティトークン

各コンポーネントは、それ自体がデザイントークンとなる一連のプロパティを持ちます。

- backgroundColor: \<Color\>
- textColor: \<Color\>
- typography: \<Typography\>
- rounded: \<Dimension\>
- padding: \<Dimension\>
- size: \<Dimension\>
- height: \<Dimension\>
- width: \<Dimension\>

## Do's and Don'ts

このセクションでは、実践的なガイドラインとよくある落とし穴を示します。これらはデザイン作成時のガードレールとして機能します。

```markdown
## Do's and Don'ts

- Do use the primary color only for the single most important action per screen
- Don't mix rounded and sharp corners in the same view
- Do maintain WCAG AA contrast ratios (4.5:1 for normal text)
- Don't use more than two font weights on a single screen
```

# 推奨トークン名（非規範）

以下の名前は、デザインシステム全般で一般的に使用されています。必須ではありませんが、一貫性を保つためのガイダンスとして示します。

**Colors:** `primary`、`secondary`、`tertiary`、`neutral`、`surface`、`on-surface`、`error`

**Typography:** `headline-display`、`headline-lg`、`headline-md`、`body-lg`、`body-md`、`body-sm`、`label-lg`、`label-md`、`label-sm`

**Rounded:** `none`、`sm`、`md`、`lg`、`xl`、`full`

# 未知のコンテンツに対するコンシューマーの動作

DESIGN.md のコンシューマーがこの仕様で定義されていないコンテンツに遭遇した場合は、次のように動作します。

| 状況 | 動作 | 例 |
|---|---|---|
| 未知のセクション見出し | 保持し、エラーにしない | `## Iconography` |
| 未知のカラートークン名 | 値が有効なら受け入れる | `surface-container-high: '#ede7dd'` |
| 未知のタイポグラフィトークン名 | 有効なタイポグラフィとして受け入れる | `telemetry-data` |
| 未知の spacing の値 | 有効な Dimension でなければ文字列として格納して受け入れる | `grid-columns: '5'` |
| 未知のコンポーネントプロパティ | 警告付きで受け入れる | `borderColor` |
| 重複したセクション見出し | エラーとしてファイルを拒否する | `## Colors` 見出しが 2 つある |
