# 作業日報

---

## 2026-04-19

### 完了した作業
- CLAUDE.md の作成（プロジェクト指示書）
- 環境構築
  - fnm 1.39.0 インストール（Node.jsバージョン管理）
  - Node.js v24.15.0 インストール
  - pnpm 10.33.0 インストール
  - Firebase CLI 15.15.0 インストール
  - Firebase に `jam006097@gmail.com` でログイン済み

### 次回やること
- Nuxtプロジェクトの作成（`pnpm dlx nuxi@latest init app`）
- 追加パッケージのインストール（Pinia, Firebase SDK, Vitest等）
- 設定ファイルの作成（nuxt.config.ts, .env 等）
- Firebaseプロジェクトとの連携（`firebase init`）

### 備考
- dotfilesで設定管理しているため、`~/.zshrc` は `~/dotfiles/.zshrc` へのシンボリックリンク
- pnpm setup・fnm の設定は `~/dotfiles/.zshrc` に追記済み
