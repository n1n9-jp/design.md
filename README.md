# DESIGN.md

[English](README_en.md)

コーディングエージェントにビジュアルアイデンティティを伝えるためのフォーマット仕様です。DESIGN.md は、デザインシステムを永続的かつ構造化された形でエージェントに理解させます。

## フォーマット

DESIGN.md ファイルは、機械可読なデザイントークン（YAML フロントマター）と、人間が読めるデザインの意図（Markdown の説明文）を組み合わせたものです。トークンは正確な値をエージェントに伝え、説明文はその値が存在する理由と適用方法を伝えます。

```md
---
name: Heritage
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 3rem
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
  label-caps:
    fontFamily: Space Grotesk
    fontSize: 0.75rem
rounded:
  sm: 4px
  md: 8px
spacing:
  sm: 8px
  md: 16px
---

## Overview

Architectural Minimalism meets Journalistic Gravitas. The UI evokes a
premium matte finish — a high-end broadsheet or contemporary gallery.

## Colors

The palette is rooted in high-contrast neutrals and a single accent color.

- **Primary (#1A1C1E):** Deep ink for headlines and core text.
- **Secondary (#6C7278):** Sophisticated slate for borders, captions, metadata.
- **Tertiary (#B8422E):** "Boston Clay" — the sole driver for interaction.
- **Neutral (#F7F5F2):** Warm limestone foundation, softer than pure white.
```

このファイルを読み込んだエージェントは、Public Sans の濃いインク色の見出し、温かみのあるライムストーン色の背景、Boston Clay 色の CTA ボタンを備えた UI を生成します。

## はじめに

DESIGN.md を仕様に照らして検証し、壊れたトークン参照の検出、WCAG コントラスト比の確認、構造上の問題の提示を行います。すべての結果は、エージェントが処理できる構造化 JSON として出力されます。

```bash
npx @google/design.md lint DESIGN.md
```

```json
{
  "findings": [
    {
      "severity": "warning",
      "path": "components.button-primary",
      "message": "textColor (#ffffff) on backgroundColor (#1A1C1E) has contrast ratio 15.42:1 — passes WCAG AA."
    }
  ],
  "summary": { "errors": 0, "warnings": 1, "info": 1 }
}
```

デザインシステムの 2 つのバージョンを比較し、トークン単位および説明文のリグレッションを検出します。

```bash
npx @google/design.md diff DESIGN.md DESIGN-v2.md
```

```json
{
  "tokens": {
    "colors": { "added": ["accent"], "removed": [], "modified": ["tertiary"] },
    "typography": { "added": [], "removed": [], "modified": [] }
  },
  "regression": false
}
```

## 仕様

DESIGN.md の完全な仕様は [`docs/spec.md`](docs/spec.md) にあります。以下はその要約です。

### ファイル構造

DESIGN.md ファイルは 2 つのレイヤーで構成されます。

1. **YAML フロントマター** — ファイル先頭の `---` で囲まれた、機械可読なデザイントークン。
2. **Markdown 本文** — `##` セクションで整理された、人間が読めるデザインの意図。

トークンは規範となる値です。説明文は、その値を適用する方法についての文脈を提供します。

### トークンスキーマ

```yaml
version: <string>          # 任意、現在値: "alpha"
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
    <token-name>: <string | token reference>
```

### トークン型

| 型 | 形式 | 例 |
|:---|:-----|:---|
| Color | 任意の CSS カラー（hex、`rgb()`、`oklch()`、名前付きカラーなど） | `"#1A1C1E"`、`"oklch(62% 0.18 250)"` |
| Dimension | 数値 + 単位（`px`、`em`、`rem`） | `48px`、`-0.02em` |
| Token Reference | `{path.to.token}` | `{colors.primary}` |
| Typography | `fontFamily`、`fontSize`、`fontWeight`、`lineHeight`、`letterSpacing`、`fontFeature`、`fontVariation` を持つオブジェクト | 上記の例を参照 |

### セクションの順序

セクションには `##` 見出しを使用します。各セクションは省略できますが、記載する場合は次の順序にする必要があります。

| # | セクション | 別名 |
|:--|:-----------|:-----|
| 1 | Overview | Brand & Style |
| 2 | Colors | |
| 3 | Typography | |
| 4 | Layout | Layout & Spacing |
| 5 | Elevation & Depth | Elevation |
| 6 | Shapes | |
| 7 | Components | |
| 8 | Do's and Don'ts | |

### コンポーネントトークン

コンポーネントは、名前をサブトークンのプロパティ群に対応づけます。

```yaml
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.on-tertiary}"
    rounded: "{rounded.sm}"
    padding: 12px
  button-primary-hover:
    backgroundColor: "{colors.tertiary-container}"
```

有効なコンポーネントプロパティは `backgroundColor`、`textColor`、`typography`、`rounded`、`padding`、`size`、`height`、`width` です。

バリアント（hover、active、pressed）は、関連するキー名を持つ個別のコンポーネントエントリとして表現します。

### 未知のコンテンツに対するコンシューマーの動作

| 状況 | 動作 |
|:-----|:-----|
| 未知のセクション見出し | 保持し、エラーにしない |
| 未知のカラートークン名 | 値が有効なら受け入れる |
| 未知のタイポグラフィトークン名 | 有効なタイポグラフィとして受け入れる |
| 未知のコンポーネントプロパティ | 警告付きで受け入れる |
| 重複したセクション見出し | エラーとしてファイルを拒否する |

## CLI リファレンス

### インストール

```bash
npm install @google/design.md
```

シェルが `@` を特別に扱う **Windows** 環境（PowerShell、一部のターミナル）では、パッケージ名を引用符で囲んでください。

```bash
npm install "@google/design.md"
```

直接実行することもできます（常に公開 npm レジストリから解決されます）。

```bash
npx @google/design.md lint DESIGN.md
```

**Windows/PowerShell** では、`design.md` という bin 名の `.md` 接尾辞がコマンド解決時に Windows の Markdown ファイル関連付けと衝突するため、この形式では何も出力されないか、Markdown エディターで `DESIGN.md` が開く場合があります。代わりに、ドットを含まない `designmd` エイリアスを使用してください。`-p` で `npx` にパッケージを指定してから、`designmd` を呼び出します。

```bash
npx -p @google/design.md designmd lint DESIGN.md
```

`designmd` shim は同じエントリーポイントに解決され、すべてのプラットフォームで同じように動作します。

#### `npm error ENOVERSIONS`（「No versions available for @google/design.md」）

CLI は npm に [`@google/design.md`](https://www.npmjs.com/package/@google/design.md) として公開されています。`ENOVERSIONS` はほとんどの場合、npm が公開レジストリを参照していないことを意味します（`.npmrc` の独自 `registry=`、このパッケージをまだ同期していない社内ミラー、`@google` スコープに対する `@google:registry` の設定ミスなど）。

現在有効なレジストリを確認します。

```bash
npm config get registry
```

通常のインターネット経由のインストールでは、`https://registry.npmjs.org/` になっている必要があります。設定を修正した後、古い 404 がキャッシュされている場合は `npm cache clean --force` を実行してから再試行してください。

すべてのコマンドは、ファイルパスまたは標準入力を表す `-` を受け付けます。デフォルトの出力形式は JSON です。

> **Windows のヒント**: `npx` を介さず、`package.json` スクリプトから CLI を直接呼び出す場合は、`design.md` ではなく `designmd` エイリアスを使用してください。元の bin 名の `.md` 接尾辞が Windows のコマンド解決時に Markdown のファイル関連付けと衝突します。`designmd` shim は同じエントリーポイントに解決され、すべてのプラットフォームで同じように動作します。
>
> ```jsonc
> // package.json
> {
>   "scripts": {
>     "design:lint": "designmd lint DESIGN.md"
>   }
> }
> ```

### `lint`

DESIGN.md ファイルの構造が正しいか検証します。

```bash
npx @google/design.md lint DESIGN.md
npx @google/design.md lint --format json DESIGN.md
cat DESIGN.md | npx @google/design.md lint -
```

| オプション | 型 | デフォルト | 説明 |
|:-----------|:---|:-----------|:-----|
| `file` | 位置引数 | 必須 | DESIGN.md へのパス（標準入力の場合は `-`） |
| `--format` | `json` | `json` | 出力形式 |

エラーが見つかった場合の終了コードは `1`、それ以外は `0` です。

### `diff`

2 つの DESIGN.md ファイルを比較し、トークン単位の変更を報告します。

```bash
npx @google/design.md diff DESIGN.md DESIGN-v2.md
```

| オプション | 型 | デフォルト | 説明 |
|:-----------|:---|:-----------|:-----|
| `before` | 位置引数 | 必須 | 変更前の DESIGN.md へのパス |
| `after` | 位置引数 | 必須 | 変更後の DESIGN.md へのパス |
| `--format` | `json` | `json` | 出力形式 |

リグレッション（変更後のファイルでエラーまたは警告が増加）が検出された場合の終了コードは `1` です。

### `export`

DESIGN.md のトークンを他の形式にエクスポートします。

```bash
npx @google/design.md export --format json-tailwind DESIGN.md > tailwind.theme.json
npx @google/design.md export --format css-tailwind DESIGN.md > theme.css
npx @google/design.md export --format dtcg DESIGN.md > tokens.json
```

| オプション | 型 | デフォルト | 説明 |
|:-----------|:---|:-----------|:-----|
| `file` | 位置引数 | 必須 | DESIGN.md へのパス（標準入力の場合は `-`） |
| `--format` | `json-tailwind` \| `css-tailwind` \| `tailwind` \| `dtcg` | 必須 | 出力形式 |

| 形式 | 出力 | 説明 |
|:-----|:-----|:-----|
| `json-tailwind` | JSON | Tailwind v3 の `theme.extend` 設定オブジェクト |
| `css-tailwind` | CSS | CSS カスタムプロパティを含む Tailwind v4 の `@theme { ... }` ブロック |
| `tailwind` | JSON | `json-tailwind` のエイリアス |
| `dtcg` | JSON | W3C Design Tokens Format Module |

エクスポートに成功した場合の終了コードは `0`（ソースに lint の指摘があっても変わりません。指摘を判定に使用するには `lint` を実行してください）、無効な `--format` またはエミッターエラーの場合は `1`、入力ファイルを読み込めない場合は `2` です。

### `spec`

DESIGN.md のフォーマット仕様を出力します（エージェントのプロンプトに仕様のコンテキストを注入する場合に便利です）。

```bash
npx @google/design.md spec
npx @google/design.md spec --rules
npx @google/design.md spec --rules-only --format json
```

| オプション | 型 | デフォルト | 説明 |
|:-----------|:---|:-----------|:-----|
| `--rules` | boolean | `false` | 有効な lint ルールの表を末尾に追加 |
| `--rules-only` | boolean | `false` | lint ルールの表のみを出力 |
| `--format` | `markdown` \| `json` | `markdown` | 出力形式 |

## Lint ルール

リンターは、解析された DESIGN.md に対して 9 つのルールを実行します。各ルールは固定の重要度で指摘を生成します。

| ルール | 重要度 | 確認内容 |
|:-------|:-------|:---------|
| `broken-ref` | error | 定義済みトークンに解決できないトークン参照（`{colors.primary}`） |
| `missing-primary` | warning | 色が定義されているが `primary` カラーが存在しない状態（エージェントが自動生成します） |
| `contrast-ratio` | warning | コンポーネントの `backgroundColor` と `textColor` の組み合わせが WCAG AA の最低基準（4.5:1）を下回っている状態 |
| `orphaned-tokens` | warning | 定義されているが、どのコンポーネントからも参照されていないカラートークン |
| `token-summary` | info | 各セクションで定義されているトークン数の概要 |
| `missing-sections` | info | 他のトークンが存在する場合に任意セクション（spacing、rounded）が欠けている状態 |
| `missing-typography` | warning | 色が定義されているがタイポグラフィトークンが存在しない状態（エージェントはデフォルトフォントを使用します） |
| `section-order` | warning | セクションが仕様で定められた順序になっていない状態 |
| `unknown-key` | warning | トップレベルの YAML キーが既知のスキーマキーのタイプミスに見える状態（例: `colours:` → `colors:`）。独自の拡張キーは対象外 |

### プログラマティック API

リンターはライブラリとしても利用できます。

```typescript
import { lint } from '@google/design.md/linter';

const report = lint(markdownString);

console.log(report.findings);       // Finding[]
console.log(report.summary);        // { errors, warnings, info }
console.log(report.designSystem);   // Parsed DesignSystemState
```

## デザイントークンの相互運用性

DESIGN.md のトークンは [W3C Design Token Format](https://www.designtokens.org/) から着想を得ています。`export` コマンドでトークンを他の形式に変換できます。

- **Tailwind v3 設定（JSON）** — `npx @google/design.md export --format json-tailwind DESIGN.md` — `tailwind.config.js` 用の `theme.extend` JSON オブジェクトを出力します。`--format tailwind` は後方互換性のためのエイリアスです。
- **Tailwind v4 テーマ（CSS）** — `npx @google/design.md export --format css-tailwind DESIGN.md` — Tailwind v4 の CSS 変数トークン名前空間（`--color-*`、`--font-*`、`--text-*`、`--leading-*`、`--tracking-*`、`--font-weight-*`、`--radius-*`、`--spacing-*`）を使用する CSS の `@theme { ... }` ブロックを出力します。
- **DTCG tokens.json**（[W3C Design Tokens Format Module](https://tr.designtokens.org/format/)）— `npx @google/design.md export --format dtcg DESIGN.md`

## ステータス

DESIGN.md フォーマットのバージョンは `alpha` です。仕様、トークンスキーマ、CLI は現在活発に開発されています。フォーマットの成熟に伴い、変更される可能性があります。

## 免責事項

このプロジェクトは [Google Open Source Software Vulnerability Rewards Program](https://bughunters.google.com/open-source-security) の対象外です。
