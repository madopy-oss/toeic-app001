# 完了報告書: TOEIC SRS PWA — ステップ1 Phase 4（shadcn/ui 導入 / ネットワーク非依存版）

**報告日**: 2026-06-14
**担当**: Claude Code（自動開発セッション）
**ブランチ**: `claude/zen-mayer-lpffs4`（リポジトリ `madopy-oss/toeic-app001`）
**ステータス**: ✅ 完了（完了基準を全項目クリア / ローカルコミットまで・push 未実施）

---

## 1. 経緯（重要）

Phase 4 は CLI 仕様の食い違いとネットワーク制約で計3回差し戻した経緯がある。最終的に、`ui.shadcn.com`（shadcn のレジストリ/プリセット配信ホスト）が本実行環境のネットワークポリシーで遮断（HTTP 403）されたままだったため、**shadcn の生成物（button/card/input/cn/標準 neutral テーマ CSS/components.json）を別環境取得の本物のソースとして直接同梱する「ネットワーク非依存版」手順**で完了した。依存パッケージのみ npm registry（許可済み）から取得。

---

## 2. 実施内容

| ステップ         | 内容                                                                                         | 結果                                     |
| ---------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------- |
| クリーン確認     | `git status`                                                                                 | 差分なし（`67707c7` 基点）               |
| 依存導入         | `pnpm add radix-ui class-variance-authority clsx tailwind-merge tw-animate-css lucide-react` | 6個追加・npm のみ・完走                  |
| 孤立アセット削除 | `git rm src/assets/hero.png public/icons.svg src/assets/vite.svg`                            | 参照ゼロ確認後に削除（favicon.svg 残存） |
| ファイル配置     | button/card/input.tsx・lib/utils.ts・index.css（置換）・components.json・App.tsx（置換）     | 7ファイル配置                            |
| 検証             | `tsc --noEmit` / `pnpm build` / `pnpm dev`                                                   | すべて成功                               |
| コミット         | `chore: setup shadcn/ui with button/card/input`                                              | `c78580b`                                |

---

## 3. 完了基準チェックリスト

| 項目                                     | 結果                                                                                                                                |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 依存6個が package.json に追加            | ✅ radix-ui 1.5.0 / class-variance-authority 0.7.1 / clsx 2.1.1 / tailwind-merge 3.6.0 / tw-animate-css 1.4.0 / lucide-react 1.18.0 |
| 孤立アセット3つ削除（favicon.svg 残存）  | ✅                                                                                                                                  |
| 7ファイル配置                            | ✅                                                                                                                                  |
| 保護対象3点が無傷                        | ✅ vite alias / main.tsx `@/App` / tsconfig paths                                                                                   |
| `tsc --noEmit` / `pnpm build` エラーなし | ✅ 両 exit 0                                                                                                                        |
| `.dark` でテーマ反転                     | ✅ dev 変換後 CSS で検証                                                                                                            |
| コミット                                 | ✅ `c78580b`                                                                                                                        |

---

## 4. 保護対象3点の無傷確認（生出力）

```
vite.config.ts:11    '@': path.resolve(import.meta.dirname, './src'),
src/main.tsx:4        import App from '@/App'
tsconfig.app.json:20  "@/*": ["./src/*"]
src/index.css:1       @import "tailwindcss";   ← 新版冒頭で保持
```

※ index.css は手順どおり新版で置換。冒頭に `@import "tailwindcss";` を含むため Tailwind は保持。

---

## 5. components.json（実測）

`baseColor: "neutral"` / `cssVariables: true` / `style: "new-york"` / `rsc: false` / `iconLibrary: "lucide"`。alias は `@/components`・`@/lib/utils`・`@/components/ui`・`@/lib`・`@/hooks` で Phase 3 の `@`→`src` と整合。primitives は Radix（button.tsx が統合 `radix-ui` パッケージの `Slot` を import）。

---

## 6. src/index.css 主要構造

```
1: @import "tailwindcss";
2: @import "tw-animate-css";
4: @custom-variant dark (&:is(.dark *));
6: @theme inline { --color-background: var(--background); ... }
   :root  { --background: oklch(1 0 0); ... }      ← light
   .dark  { --background: oklch(0.145 0 0); ... }   ← dark
   @layer base { body { @apply bg-background text-foreground; } }
```

---

## 7. ls -R src public

```
public: favicon.svg
src: App.tsx  index.css  main.tsx  components/  lib/
src/components/ui: button.tsx  card.tsx  input.tsx
src/lib: utils.ts
```

（`src/assets/` は空になり消滅）

---

## 8. 検証結果

- `tsc --noEmit`: exit 0
- `pnpm build`: exit 0（124 modules transformed、CSS 6.55→22.18 kB ＝ テーマ変数+コンポーネント取り込みの証跡）
- ダーク反転（ヘッドレス方式）: dev 変換後 CSS を HTTP 取得して検証。`.dark` セレクタ出力あり、`--background` が light=`oklch(1 0 0)`（白）↔ dark=`oklch(0.145 0 0)`（ほぼ黒）で反転。`bg-background` ユーティリティ生成済み、root HTTP 200。`index.html` に `class="dark"` は付けていない（デフォルト light）。

---

## 9. 詰まった点・想定外 / 申し送り

1. `ui.shadcn.com` 遮断は本フェーズでは未解決のまま。ネットワーク非依存版で回避して完了したが、今後 shadcn コンポーネントを追加する際は、同ホストの allowlist 解決か、同様の別環境取得→取り込みが必要。
2. `.agents/`（shadcn skill）の git 追跡可否（rev3 §7 申し送り）: `.gitignore` に追加して非追跡で確定済み（コミット `67707c7`）。アプリのソースではないため。
3. ダークモードは土台（CSS 変数 + `.dark`）のみ。切り替え UI / 永続化 / システム追従は設定画面フェーズで実装予定。
4. `favicon.svg` は仮アイコンとして残置。正規アイコン差し替えは後続ステップ。
5. base color = neutral。要件の青み（`theme_color #0f172a`）は PWA manifest 側で別途担保する想定。

---

## 10. 停止状態

完了基準を全項目満たし、ローカルコミットまでで停止。Phase 5 以降には自動進行していない。
