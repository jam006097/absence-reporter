# Absence Reporter 開発ガイド Part 2
# セットアップ手順：1から10まで（初心者向け）

---

## 前提確認

このガイドはmacOSを使用していることを前提にしています。
すべてのコマンドはターミナル（WezTermなど）で実行します。

---

## STEP 1: fnmのインストール（Node.jsバージョン管理）

### なぜfnmを先にインストールするか？

Node.jsは直接インストールするのではなく、**バージョン管理ツール経由でインストール**するのがベストプラクティスです。これにより、後からバージョンの変更や複数バージョンの切り替えが容易になります。

### 手順

```bash
# fnmをインストール（Homebrew経由）
brew install fnm

# fnmをzshに追加
echo 'eval "$(fnm env --use-on-cd)"' >> ~/.zshrc

# 設定を反映
source ~/.zshrc

# インストール確認
fnm --version
# → fnm 1.x.x と表示されればOK
```

---

## STEP 2: Node.jsのインストール

```bash
# 最新LTS（長期サポート）版をインストール
fnm install --lts

# インストールしたバージョンを使用
fnm use lts-latest

# デフォルトに設定（新しいターミナルでも自動で使われる）
fnm default lts-latest

# 確認
node --version
# → v22.x.x などと表示されればOK

npm --version
# → 10.x.x などと表示されればOK
```

---

## STEP 3: pnpmのインストール

```bash
# Node.jsに含まれるcorepackでpnpmを有効化（推奨方法）
corepack enable pnpm

# バージョン確認
pnpm --version
# → 9.x.x などと表示されればOK
```

> **補足**: `corepack` はNode.jsに同梱されているパッケージマネージャー管理ツールです。これを使うとpnpmのバージョン管理も自動化されます。

---

## STEP 4: Firebase CLIのインストール

Firebase Hosting へのデプロイや、Firestoreのセキュリティルール設定に使用します。

```bash
# Firebase CLIをグローバルインストール
pnpm add -g firebase-tools

# バージョン確認
firebase --version
# → 13.x.x などと表示されればOK
```

---

## STEP 5: Firebaseプロジェクトの作成

### 5-1. Googleアカウントでログイン

```bash
firebase login
```

ブラウザが開くので、Googleアカウントでログインします。

### 5-2. Firebase ConsoleでプロジェクトをGUIで作成

1. ブラウザで **https://console.firebase.google.com/** にアクセス
2. 「プロジェクトを追加」をクリック
3. プロジェクト名: `absence-reporter`（任意の名前でOK）
4. Googleアナリティクス: 「このプロジェクトでGoogle アナリティクスを有効にする」は **オフ** でOK
5. 「プロジェクトを作成」をクリック → 完了まで待つ

### 5-3. Firestoreデータベースの有効化

1. 左メニューの「構築」→「Firestore Database」をクリック
2. 「データベースの作成」をクリック
3. ロケーション: `asia-northeast1`（東京）を選択
4. セキュリティルール: 「テストモードで開始」を選択（後でルールを設定する）
5. 「作成」をクリック

### 5-4. 匿名認証の有効化

1. 左メニューの「構築」→「Authentication」をクリック
2. 「始める」をクリック
3. 「Sign-in method」タブ → 「匿名」をクリック
4. 「有効にする」をオンにして「保存」

### 5-5. WebアプリのAPIキーを取得

1. Firebase Console の「プロジェクトの概要」（左上の歯車アイコン）→「プロジェクトの設定」
2. 「全般」タブを下にスクロール
3. 「アプリ」セクションで「</> ウェブ」アイコンをクリック
4. アプリ名: `absence-reporter-web`
5. 「Firebase Hosting も設定する」は **チェックしない**（後でやる）
6. 「アプリを登録」をクリック
7. 以下のような設定情報が表示される：

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "absence-reporter-xxxxx.firebaseapp.com",
  projectId: "absence-reporter-xxxxx",
  storageBucket: "absence-reporter-xxxxx.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};
```

**この情報を後のSTEP 8で使います。メモしておいてください。**

---

## STEP 6: Nuxtプロジェクトの初期化

```bash
# absence-reporter ディレクトリにいることを確認
pwd
# → /Users/ik/work/absence-reporter と表示されればOK

# Nuxtプロジェクトを app/ ディレクトリに作成
pnpm dlx nuxi@latest init app
```

いくつか質問が出るので、以下のように答えます：

```
Which package manager would you like to use?
→ pnpm を選択

Initialize git repository?
→ No（既にgit管理されているため）
```

```bash
# app/ ディレクトリに移動
cd app

# 依存パッケージをインストール
pnpm install

# 開発サーバーを起動して動作確認
pnpm dev
```

ブラウザで `http://localhost:3000` にアクセスし、Nuxtのデフォルトページが表示されればOKです。

確認できたら `Ctrl + C` でサーバーを停止します。

---

## STEP 7: 追加パッケージのインストール

```bash
# app/ ディレクトリにいることを確認
cd /Users/ik/work/absence-reporter/app

# Piniaのインストール
pnpm add pinia @pinia/nuxt

# Firebaseのインストール
pnpm add firebase

# ESLint（Nuxt公式モジュール）
pnpm add -D @nuxt/eslint eslint

# Prettier
pnpm add -D prettier eslint-config-prettier

# Vitest + Vue Test Utils
pnpm add -D vitest @vue/test-utils happy-dom @vitest/coverage-v8

# Playwright
pnpm add -D @playwright/test
pnpm exec playwright install chromium  # ブラウザのインストール
```

---

## STEP 8: 設定ファイルの作成

### 8-1. .env ファイルの作成（Firebase設定）

```bash
# app/ ディレクトリで実行
touch .env
```

`.env` ファイルに以下を記述（STEP 5-5でメモしたAPIキーを使用）：

```env
NUXT_PUBLIC_FIREBASE_API_KEY=AIzaSy...
NUXT_PUBLIC_FIREBASE_AUTH_DOMAIN=absence-reporter-xxxxx.firebaseapp.com
NUXT_PUBLIC_FIREBASE_PROJECT_ID=absence-reporter-xxxxx
NUXT_PUBLIC_FIREBASE_STORAGE_BUCKET=absence-reporter-xxxxx.appspot.com
NUXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
NUXT_PUBLIC_FIREBASE_APP_ID=1:123456789:web:abcdef123456
```

> ⚠️ `.env` は `.gitignore` に記載済みなので、Gitにコミットされません（APIキーの漏洩を防ぐ）。

### 8-2. nuxt.config.ts の設定

```typescript
// app/nuxt.config.ts
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: [
    '@pinia/nuxt',
    '@nuxt/eslint',
  ],
  runtimeConfig: {
    public: {
      firebaseApiKey: process.env.NUXT_PUBLIC_FIREBASE_API_KEY,
      firebaseAuthDomain: process.env.NUXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
      firebaseProjectId: process.env.NUXT_PUBLIC_FIREBASE_PROJECT_ID,
      firebaseStorageBucket: process.env.NUXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
      firebaseMessagingSenderId: process.env.NUXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
      firebaseAppId: process.env.NUXT_PUBLIC_FIREBASE_APP_ID,
    }
  },
  typescript: {
    strict: false,
  },
})
```

### 8-3. Vitest設定ファイルの作成

```typescript
// app/vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'happy-dom',
    globals: true,
  },
})
```

### 8-4. Playwright設定ファイルの作成

```bash
# Playwrightの設定を初期化
pnpm exec playwright init
```

質問が出たら以下のように答えます：

```
Where to put your end-to-end tests?
→ e2e

Add a GitHub Actions workflow?
→ No（今は不要）
```

### 8-5. ESLint設定ファイルの作成

```javascript
// app/eslint.config.mjs
import withNuxt from './.nuxt/eslint.config.mjs'

export default withNuxt()
```

### 8-6. Prettier設定ファイルの作成

```json
// app/.prettierrc
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

### 8-7. package.json にスクリプトを追加

`app/package.json` の `scripts` セクションを編集：

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "playwright test"
  }
}
```

---

## STEP 9: ディレクトリ構成の作成

```bash
# app/ ディレクトリで実行
mkdir -p components pages stores services types utils e2e

# 型定義ファイル
touch types/absence.ts

# Firebaseサービスファイル
touch services/firebase.ts

# ページファイル
touch pages/index.vue
touch pages/entry.vue
touch pages/dashboard.vue
```

### types/absence.ts の中身

```typescript
// app/types/absence.ts
import type { Timestamp } from 'firebase/firestore'

export type Absence = {
  id?: string
  name: string
  reason: '体調不良' | '私用' | 'その他'
  startDate: string
  endDate: string
  note?: string
  createdAt: Timestamp | null
}
```

### services/firebase.ts の中身

```typescript
// app/services/firebase.ts
import { initializeApp, getApps } from 'firebase/app'
import { getFirestore } from 'firebase/firestore'
import { getAuth } from 'firebase/auth'

export const useFirebase = () => {
  const config = useRuntimeConfig()

  const firebaseConfig = {
    apiKey: config.public.firebaseApiKey,
    authDomain: config.public.firebaseAuthDomain,
    projectId: config.public.firebaseProjectId,
    storageBucket: config.public.firebaseStorageBucket,
    messagingSenderId: config.public.firebaseMessagingSenderId,
    appId: config.public.firebaseAppId,
  }

  const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
  const db = getFirestore(app)
  const auth = getAuth(app)

  return { app, db, auth }
}
```

---

## STEP 10: Firebaseプロジェクトとの連携設定

```bash
# absence-reporter ルートディレクトリで実行
cd /Users/ik/work/absence-reporter

# Firebaseプロジェクトを初期化
firebase init
```

質問に以下のように答えます：

```
Which Firebase features do you want to set up?
→ スペースキーで「Firestore」と「Hosting」を選択してEnter

Please select an option
→ Use an existing project（STEP 5で作成したプロジェクトを選択）

What file should be used for Firestore Rules?
→ firestore.rules（Enterでデフォルト）

What file should be used for Firestore indexes?
→ firestore.indexes.json（Enterでデフォルト）

What do you want to use as your public directory?
→ app/.output/public（Nuxtのビルド出力先）

Configure as a single-page app?
→ Yes

Set up automatic builds and deploys with GitHub?
→ No
```

### firestore.rules の初期設定

```
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /absences/{document} {
      // 認証済みユーザー（匿名含む）のみ読み書き可能
      allow read, write: if request.auth != null;
    }
  }
}
```

### 動作確認

```bash
cd app
pnpm dev
```

`http://localhost:3000` にアクセスして画面が表示されればセットアップ完了です！

---

## セットアップ完了後のチェックリスト

- [ ] `node --version` でv22以上が表示される
- [ ] `pnpm --version` でv9以上が表示される
- [ ] `firebase --version` でv13以上が表示される
- [ ] `pnpm dev` でNuxtの開発サーバーが起動する（http://localhost:3000）
- [ ] `.env` ファイルにFirebase設定が記入されている
- [ ] `firebase.json` と `firestore.rules` が存在する

---

*次のページ → Part 3: Claude Code の活用ガイド*
