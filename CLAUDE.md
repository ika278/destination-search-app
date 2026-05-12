# 目的地検索アプリ — 開発ガイド

## 技術スタック
- **フロントエンド**: React 18 + TypeScript (strict) + Vite
- **バックエンド**: Supabase (PostgreSQL + Auth + RLS)
- **ユニット/統合テスト**: Vitest + Testing Library
- **E2Eテスト**: Playwright

## 開発手法: TDD + マルチエージェント

### ワークフロー
```
要件定義 → /test-agent → /dev-agent → /review-agent → /pr
```

| フェーズ | コマンド | 状態 | やること |
|----------|----------|------|---------|
| RED | `/test-agent` | テスト失敗 | 要件からテストを先行作成 |
| GREEN | `/dev-agent` | テスト通過 | 最小限の実装でテストを通す |
| REFACTOR | `/review-agent` | レビュー | 品質・セキュリティ確認 |
| PR | `/pr` | マージ準備 | PRを作成 |

### ルール
- **実装より先にテストを書く**（テストなしの実装は禁止）
- テストはインターフェースをテストする（実装の詳細はテストしない）
- `any` 型禁止
- Supabase の操作には必ず RLS を設計する

## ディレクトリ構成
```
src/
  components/         # Reactコンポーネント
    __tests__/        # コンポーネントのテスト
  hooks/              # カスタムフック
    __tests__/
  lib/
    supabase.ts       # Supabaseクライアント
  types/              # 型定義
  utils/              # ユーティリティ関数
    __tests__/
  test/
    setup.ts          # テストセットアップ
e2e/                  # Playwright E2Eテスト
```

## よく使うコマンド
```bash
npm run dev           # 開発サーバー起動
npm test              # ユニットテスト（watch mode）
npm run test:coverage # カバレッジ付きテスト
npm run test:e2e      # E2Eテスト
npx tsc --noEmit      # 型チェック
```

## 環境変数
`.env.example` をコピーして `.env` を作成し、Supabaseの値を設定する。
```
cp .env.example .env
```
