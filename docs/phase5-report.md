# 完了報告書: TOEIC SRS PWA — ステップ1 Phase 5（ESLint + Prettier / rev2・A案）

**報告日**: 2026-06-14
**担当**: Claude Code（自動開発セッション）
**ブランチ**: `claude/zen-mayer-lpffs4`（リポジトリ `madopy-oss/toeic-app001`）
**ステータス**: ✅ 完了（完了基準を全項目クリア / ローカルコミットまで・push 未実施）

---

## 1. 経緯

初版 Phase 5 は最終検証の `pnpm lint` で `react-refresh/only-export-components` の error 1件（shadcn の `button.tsx` が `Button` と `buttonVariants` を併せて export する標準構造との衝突）が残り、上流へ差し戻した。上流が **A案（`src/components/ui/**`のみ当該ルールを warn に下げる）** ＋`.vscode/settings.json` 追跡例外を反映した rev2 を発行。本報告は rev2 に基づく完了報告。

---

## 2. 実施内容

| ステップ      | 内容                                                                                                       | 結果              |
| ------------- | ---------------------------------------------------------------------------------------------------------- | ----------------- |
| 依存導入      | `pnpm add -D prettier eslint-config-prettier prettier-plugin-tailwindcss eslint-plugin-simple-import-sort` | 4個追加・npm のみ |
| Prettier 設定 | `.prettierrc`（`tailwindStylesheet` 含む）/ `.prettierignore` 作成                                         | 完了              |
| ESLint 拡張   | import sort / unused 緩和 / **A案 ui/** warn\*\* / `eslint-config-prettier` 最後                           | 既存維持で追記    |
| scripts       | `package.json` に `format` / `format:check`                                                                | 完了              |
| VSCode        | `.vscode/settings.json` / `extensions.json` + `.gitignore` 例外                                            | 完了              |
| 検証          | `pnpm format` / `pnpm lint` / `tsc --noEmit` / `pnpm build`                                                | 全成功            |
| コミット      | `chore: configure eslint and prettier`                                                                     | `b700a46`         |

---

## 3. 完了基準チェックリスト

| 項目                                                                                           | 結果                                                                                                                            |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 依存4個が devDependencies に                                                                   | ✅ prettier 3.8.4 / eslint-config-prettier 10.1.8 / prettier-plugin-tailwindcss 0.8.0 / eslint-plugin-simple-import-sort 13.0.0 |
| `.prettierrc`（`tailwindStylesheet: ./src/index.css`）                                         | ✅                                                                                                                              |
| `.prettierignore`                                                                              | ✅                                                                                                                              |
| `eslint.config.js` 拡張（import sort・unused 緩和・ui/\*\* react-refresh warn・prettier 最後） | ✅ 既存構成維持                                                                                                                 |
| `package.json` に `format` / `format:check`                                                    | ✅                                                                                                                              |
| `.vscode/settings.json` / `extensions.json` + `.gitignore` 例外                                | ✅ `!.vscode/settings.json` 追加                                                                                                |
| `pnpm format` でクラス順ソート                                                                 | ✅ 実証（§5）                                                                                                                   |
| `pnpm lint` が error 0 で完走                                                                  | ✅ 0 errors / 6 warnings / exit 0                                                                                               |
| `tsc --noEmit` / `pnpm build` エラーなし                                                       | ✅ 両 exit 0                                                                                                                    |
| `.vscode/settings.json` を含むコミット                                                         | ✅ `b700a46`                                                                                                                    |

---

## 4. A案の効果（前回 error の解消）

```
src/components/ui/button.tsx
  62:18  warning  Fast refresh only works when a file only exports components ...
                  react-refresh/only-export-components
```

前回 **error** だったこの行が、`files: ['src/components/ui/**']` 限定ブロックで **warning** に変わり、`pnpm lint` は **0 errors**（exit 0）になった。自作コード側は error のまま維持。`allowConstantExport` は cva の戻り値（関数）に効かないため不使用。

---

## 5. eslint.config.js 追記部分の要点

```js
import simpleImportSort from 'eslint-plugin-simple-import-sort'
import eslintConfigPrettier from 'eslint-config-prettier'

// 既存ブロック内に:
plugins: { 'simple-import-sort': simpleImportSort },
rules: {
  'simple-import-sort/imports': 'warn',
  'simple-import-sort/exports': 'warn',
  '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_', varsIgnorePattern: '^_' }],
},

// A案・shadcn ui だけ緩和:
{ files: ['src/components/ui/**'], rules: { 'react-refresh/only-export-components': 'warn' } },

// 配列末尾:
eslintConfigPrettier,
```

`.gitignore` 追加行:

```
!.vscode/settings.json
```

---

## 6. クラス順ソート実証（before/after・プロジェクト内一時ファイル）

- BEFORE: `p-4 flex bg-background items-center min-h-screen justify-center`
- AFTER : `flex min-h-screen items-center justify-center bg-background p-4`
- → `tailwindStylesheet` が効き、theme 色 `bg-background` も正しく整列。`src/App.tsx` は既にこの順序のため format で unchanged。

---

## 7. 検証結果

- `pnpm lint`: 0 errors / 6 warnings（内訳＝import/export ソート5件＋ui の react-refresh 1件、いずれも文書どおり warn 設定で許容）、exit 0
- `pnpm exec tsc --noEmit`: exit 0
- `pnpm build`: exit 0（124 modules、CSS 22.24 kB）

---

## 8. 詰まった点・想定外

- なし。A案で前回のブロッカーは解消。
- 補足: `prettier --write .` が `docs/phase*-report.md` と `tsconfig.json` 等も整形対象に含めたため差分に出ている（全ファイル一貫整形の想定どおり）。

---

## 9. 次フェーズへの申し送り

- 以降の新規コードは lint/format 対象。Phase 6 のファイルも整形される。
- 自作コンポーネントでは `react-refresh/only-export-components` が error のまま（HMR チェック有効）。コンポーネントと定数/関数は別ファイルに分けるのが望ましい。
- テスト基盤（Vitest）は SRS ロジック着手フェーズで導入予定。

---

## 10. 停止状態

完了基準を全項目満たし、ローカルコミットまでで停止。Phase 6 以降には自動進行していない。
