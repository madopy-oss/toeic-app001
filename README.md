# TOEIC SRS

TOEIC 語彙学習のための個人用 PWA。SM-2 アルゴリズムによる SRS（間隔反復学習）と Anki 流の4段階評価で単語を覚える。モバイル特化（Android Chrome 主軸、iOS Safari ベストエフォート）。

## 技術スタック

- **React 19** + **TypeScript** + **Vite 8**
- **Tailwind CSS 4** + **shadcn/ui**（new-york / base color: neutral）
- **Dexie.js**（IndexedDB）※ ステップ2以降で導入
- **vite-plugin-pwa** ※ 後続ステップで導入
- デプロイ: **Cloudflare Pages**（ルートデプロイ）

## 必要環境

- Node.js 22.x
- pnpm

## セットアップ

```
pnpm install
pnpm dev          # 開発サーバ（http://localhost:5173）
```

## スクリプト

| コマンド | 内容 |
|---|---|
| `pnpm dev` | 開発サーバ |
| `pnpm build` | 本番ビルド |
| `pnpm preview` | ビルド成果物のプレビュー |
| `pnpm lint` | ESLint |
| `pnpm format` | Prettier で整形 |
| `pnpm format:check` | 整形チェック |

## ディレクトリ構成

```
src/
  components/ui/      shadcn/ui の生成物（手動編集しない）
  components/common/  自作の共通 UI（Header, BottomNav 等）
  features/           画面・機能単位（実装時にサブディレクトリを作成）
  db/                 Dexie スキーマ + repository 層
  lib/                純粋ロジック（SM-2 SRS 計算等、テスト対象）
  hooks/              共通カスタムフック
  types/              共通型定義
  routes/             ルーター定義
```

## 開発メモ

- **shadcn/ui の追加**: 実行環境で `ui.shadcn.com` が遮断されているため、コンポーネント追加はネットワーク制約のない環境で取得してリポジトリに取り込む。
- **ダークモード**: CSS 変数 + `.dark` の土台のみ実装済み。切り替え（auto/light/dark・システム追従・永続化）は設定画面フェーズで実装予定。
- **カラー**: base color は neutral。PWA の `theme_color` は `#0f172a`。
- `src/components/ui/` は ESLint の `react-refresh/only-export-components` を warn に緩和（shadcn 標準構造のため）。

## ステータス

ステップ1（環境セットアップ）完了。ステップ2以降で db（Dexie）→ lib（SRS ロジック）→ features（画面）→ routes（ルーティング）→ PWA 構成 の順に実装予定。
