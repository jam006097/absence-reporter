# Absence Reporter 開発ガイド Part 1
# 技術スタック完全解説：歴史・背景・使い方・メリデメ

---

## はじめに

このガイドは、Absence Reporterプロジェクトの開発を始めるにあたって、使用する技術を一から理解するための資料です。各技術の「なぜ存在するのか」「何ができるのか」「どんな注意点があるか」を丁寧に解説します。

---

## 1. Node.js

### 歴史・背景

- **2009年**: Ryan Dahl氏がChromeのV8エンジンをサーバーサイドで動かすために開発。
- それまでJavaScriptはブラウザの中だけで動く言語だった。Node.jsの登場でサーバー・ツール・CLIなど、あらゆる場所でJavaScriptが使えるようになった。
- **現在**: バージョン管理は「LTS（長期サポート）版」を使うのが定番。2025年時点ではNode.js 22がLTS。

### シェア・立ち位置

- フロントエンド開発では「JavaScriptのツールを動かすための土台」として必須。NuxtもVitestもpnpmも、全てNode.jsの上で動く。
- GitHubのリポジトリ数・Stack Overflowの質問数でも常にトップクラス。

### 基本的な使い方

```bash
# バージョン確認
node --version

# JavaScriptファイルを実行
node hello.js

# Node.jsはインストールするだけでOK。直接触る機会は少ない。
```

### メリット

- JavaScriptひとつで「ブラウザ」も「サーバー」も書ける（言語統一）
- npm/pnpmなどのパッケージ管理が使えるようになる
- 巨大なエコシステム（世界最大規模のパッケージ数）

### デメリット・注意点

- バージョンが頻繁に上がるため、**バージョン管理ツール（nvm / fnm）** を使うのがベストプラクティス
- 古いバージョンを使うと他のツールと動かないことがある

### このプロジェクトでの役割

Nuxt・pnpm・Vitestなどすべてのツールの「土台」。直接コードを書くことはほぼない。

---

## 2. fnm（Fast Node Manager）

### 歴史・背景

- Node.jsは頻繁にバージョンが変わる。プロジェクトごとに異なるバージョンが必要になることも多い。
- **nvm**（Node Version Manager）が長年デファクトスタンダードだったが、起動が遅い問題があった。
- **fnm**（2019年〜）はRustで書かれており、nvmの10〜40倍高速。現在最も推奨されるNode.jsバージョン管理ツール。

### 基本的な使い方

```bash
# Node.jsの最新LTS版をインストール
fnm install --lts

# プロジェクト用のバージョンを固定（.node-versionファイル生成）
fnm use --version-file-strategy=local lts-latest

# 現在使用中のバージョン確認
fnm current
```

### メリット

- 高速（Rustで実装）
- `.node-version` ファイルで「このプロジェクトはこのバージョン」を自動切り替え
- インストールが簡単

### デメリット

- nvmより後発なので情報が少ない場面がある（ただし基本操作は十分）

---

## 3. pnpm

### 歴史・背景

- JavaScriptのパッケージ管理ツールとして **npm**（2010年〜）が登場。`node_modules` フォルダに依存パッケージをすべてコピーする仕組みだった。
- **yarn**（2016年〜, Facebook製）がnpmの遅さ・信頼性問題を解決するために登場。
- **pnpm**（2016年〜）は「コピーではなくシンボリックリンク」方式でディスク使用量を劇的に削減。現在最も効率的なパッケージマネージャーとして急速に普及中。

### シェア・立ち位置

- Nuxt公式・Vite公式がpnpmを推奨するようになり、フロントエンド界隈での採用が急増。
- 2024年時点でnpmの代替として多くの企業・OSSプロジェクトが採用。

### 基本的な使い方

```bash
# プロジェクトの初期化
pnpm init

# パッケージのインストール（npmのinstallに相当）
pnpm install

# 特定パッケージを追加
pnpm add firebase
pnpm add -D vitest  # -D は開発依存（devDependencies）

# パッケージを削除
pnpm remove パッケージ名

# スクリプトを実行
pnpm dev
pnpm build
pnpm test
```

### メリット

- **ディスク節約**: 同じパッケージは1回だけダウンロード。複数プロジェクトで共有。
- **高速**: npmより2〜3倍速いケースが多い
- **厳格な依存管理**: 意図しないパッケージアクセスを防ぐ（フラットなnode_modulesを作らない）

### デメリット

- 一部の古いツールとシンボリックリンクの相性問題がまれにある
- npmに慣れた人は最初少し戸惑う（コマンドはほぼ同じだが）

---

## 4. TypeScript

### 歴史・背景

- **2012年**: Microsoft社がJavaScriptに「型」を追加した言語として開発・公開。
- JavaScriptは動的型付け言語で、変数の型が実行時まで分からないため、大規模開発でバグが増えやすかった。
- TypeScriptは「型チェック」を追加することで、エディタでのエラー検出・補完を大幅に強化した。
- **現在**: フロントエンド・バックエンド問わず、新規プロジェクトの大多数がTypeScriptを採用。

### シェア・立ち位置

- Stack Overflow調査（2024年）で「最も愛されている言語」上位常連。
- NuxtもVueもFirebase SDKも全てTypeScriptで書かれており、TypeScriptとの親和性が高い。

### 基本的な使い方

```typescript
// 変数に型を付ける
const name: string = "山田太郎";
const age: number = 30;

// 関数の引数と戻り値に型を付ける
function greet(name: string): string {
  return `こんにちは、${name}さん`;
}

// オブジェクトの型定義（type / interface）
type Absence = {
  name: string;
  reason: '体調不良' | '私用' | 'その他';  // ユニオン型
  startDate: string;
  note?: string;  // ?は省略可能
};
```

### メリット

- エディタ（VS Code等）で補完・エラー検知が強力になる
- バグを実行前に発見できる
- コードの意図が伝わりやすくなる（ドキュメント代わり）

### デメリット

- 最初は型を書く手間がかかる（慣れれば気にならない）
- ビルド（tsc）が必要（NuxtやViteが自動でやってくれるので実際はほぼ気にしない）

---

## 5. Vue.js / Nuxt.js

### Vue.js の歴史・背景

- **2014年**: Google社員だったEvan You氏が個人で開発・公開。AngularJSの良いとこ取りを目指した。
- **Vue 3（2020年〜）**: Composition APIを導入し、コードの再利用性・TypeScript対応が大幅向上。
- **シェア**: React・Angularと並ぶ3大フロントエンドフレームワーク。特にアジア圏（日本・中国）での採用率が高い。

### Nuxt.js の歴史・背景

- **2016年〜**: Vueの上に「ファイルベースのルーティング」「SSR（サーバーサイドレンダリング）」「自動インポート」などを追加したフレームワーク。
- **Nuxt 3（2022年〜）**: Vite・TypeScript・Vue 3をフル活用した現行バージョン。
- ReactにおけるNext.jsと同じポジション。

### Vue 3 Composition API の基本

```vue
<script setup lang="ts">
// <script setup> がVue3の標準スタイル。return不要で簡潔。
import { ref, computed } from 'vue'

const count = ref(0)  // リアクティブな変数
const double = computed(() => count.value * 2)  // 算出プロパティ

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">{{ count }} (double: {{ double }})</button>
</template>
```

### Nuxtの自動インポートとファイルルーティング

```
pages/
  index.vue      → / （トップページ）
  dashboard.vue  → /dashboard
  entry.vue      → /entry
```

- `pages/`に置くだけでルーティングが自動設定される（手動設定不要）
- `composables/`, `components/` 等は自動インポートされる

### メリット

| Vue.js | Nuxt.js |
|---|---|
| 学習コストが低い（HTMLに近い） | ファイルルーティングで設定不要 |
| 日本語ドキュメントが充実 | 自動インポートで`import`文が減る |
| 段階的に導入可能 | Firebase Hostingと相性が良い（静的ビルド） |

### デメリット

- Reactより求人・コミュニティがやや小さい（国内は差が少ない）
- Nuxtの規約を覚える必要がある

---

## 6. Pinia

### 歴史・背景

- **2019年〜**: Vue CoreチームメンバーのEduardo San Martin Morote氏が開発。
- VuexというVue公式の状態管理ライブラリが複雑すぎたため、よりシンプルな代替として登場。
- **2022年**: Vue公式の状態管理ライブラリとして正式採用（Vuexの後継）。

### 状態管理とは？

複数のコンポーネントで共有したいデータ（例: ログイン中のユーザー情報、フォームの入力値）を一箇所で管理する仕組み。

### 基本的な使い方

```typescript
// stores/absence.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import type { Absence } from '@/types/absence'

export const useAbsenceStore = defineStore('absence', () => {
  const absences = ref<Absence[]>([])

  function addAbsence(absence: Absence) {
    absences.value.push(absence)
  }

  return { absences, addAbsence }
})
```

```vue
<!-- コンポーネントで使う -->
<script setup lang="ts">
import { useAbsenceStore } from '@/stores/absence'
const store = useAbsenceStore()
</script>
```

### メリット

- シンプルで学習コストが低い
- TypeScriptとの親和性が高い
- Vue DevToolsでデバッグしやすい

### デメリット

- 大規模アプリだと設計の自由度が高すぎて迷うことも（小規模なら問題なし）

---

## 7. Firebase

### 歴史・背景

- **2011年**: Envolve社がリアルタイムチャット機能として開発。
- **2014年**: Googleが買収。Googleのインフラ上で提供するBaaS（Backend as a Service）として進化。
- **現在**: 世界中のスタートアップ・個人開発者が採用。特に「サーバーを立てずにアプリを作りたい」ニーズに最適。

### このプロジェクトで使うFirebaseサービス

#### Cloud Firestore（データベース）

```typescript
// ドキュメントの追加
import { addDoc, collection, serverTimestamp } from 'firebase/firestore'
import { db } from '@/services/firebase'

await addDoc(collection(db, 'absences'), {
  name: '山田太郎',
  reason: '体調不良',
  createdAt: serverTimestamp()  // サーバー側の正確な時刻
})
```

```typescript
// リアルタイム取得（onSnapshot）
import { onSnapshot, query, collection, orderBy } from 'firebase/firestore'

onSnapshot(query(collection(db, 'absences'), orderBy('createdAt', 'desc')), (snapshot) => {
  const absences = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))
  // absencesが自動で最新状態に更新される
})
```

#### Firebase Hosting（ホスティング）

- NuxtでビルドしたHTMLファイルをGoogleのCDNで配信。
- `firebase deploy` コマンド一発で公開できる。

#### Firebase匿名認証

- ユーザーにアカウント登録させずに「匿名ユーザー」として認証。
- セキュリティルールで「認証済みユーザーのみ書き込み可」と設定できる。

### メリット

- **サーバー不要**: バックエンドを自分で構築・管理しなくていい
- **リアルタイム同期**: onSnapshotで全クライアントが自動更新
- **無料枠が充実**: Sparkプラン（無料）でこのプロジェクトは十分動く
- **スケール**: アクセスが増えても自動でスケール

### デメリット

- ベンダーロックイン（Googleのサービスに依存する）
- 複雑なクエリが苦手（JOINなどRDB的な操作は難しい）
- 大規模になると費用が高くなる可能性

---

## 8. Vitest

### 歴史・背景

- **2021年〜**: Viteの作者（Anthony Fu氏）が開発。ViteプロジェクトでのJestの遅さ・設定の複雑さを解決するために登場。
- **Jest**（2014年〜, Facebook製）が長らくJavaScriptテストのスタンダードだったが、Vite環境では設定が複雑だった。
- **Vitest**はVite設定をそのまま使えるため、ゼロコンフィグに近い。

### 基本的な使い方

```typescript
// utils/date.test.ts
import { describe, it, expect } from 'vitest'
import { isToday } from '@/utils/date'

describe('isToday', () => {
  it('今日の日付はtrueを返す', () => {
    const today = new Date().toISOString().split('T')[0]
    expect(isToday(today)).toBe(true)
  })

  it('昨日の日付はfalseを返す', () => {
    expect(isToday('2020-01-01')).toBe(false)
  })
})
```

```bash
pnpm test          # テスト実行
pnpm test --watch  # ウォッチモード（ファイル変更で自動再実行）
```

### メリット

- Viteと設定を共有できるためゼロコンフィグに近い
- 高速
- JestとほぼAPIが同じため移行しやすい

### デメリット

- Jestより歴史が浅く情報が少ない場合がある（現在はほぼ解消）

---

## 9. Playwright

### 歴史・背景

- **2020年〜**: Microsoft開発。Puppeteer（Google製）の主要開発者が移籍して作成。
- **E2Eテスト（End-to-End）**: 実際のブラウザを動かして「ユーザーが操作する一連の流れ」をテストする。
- **Cypress**が長らくE2Eテストの主流だったが、Playwrightの登場で急速にシェアを奪っている（2024年時点でほぼ同率）。

### 基本的な使い方

```typescript
// e2e/entry.spec.ts
import { test, expect } from '@playwright/test'

test('欠勤登録フォームが送信できる', async ({ page }) => {
  await page.goto('http://localhost:3000/entry')

  await page.fill('[name="name"]', '山田太郎')
  await page.selectOption('[name="reason"]', '体調不良')
  await page.click('button[type="submit"]')

  // 成功メッセージが表示されることを確認
  await expect(page.locator('.success-message')).toBeVisible()
})
```

### メリット

- Chrome・Firefox・Safariの複数ブラウザでテスト可能
- 自動待機機能（要素が現れるまで自動で待つ）
- スクリーンショット・動画録画でデバッグが容易
- Microsoft製でVS Codeとの連携が強力

### デメリット

- テスト実行が重い（実際のブラウザを起動するため）
- E2Eテストは書くコストが高い（ユニットテストより）

---

## 10. ESLint + Prettier

### 歴史・背景

**ESLint（2013年〜）**
- JavaScriptのコードの「問題のある書き方」を検出するLinter（静的解析ツール）。
- 例: `==` の代わりに `===` を使う、未使用変数を検出するなど。

**Prettier（2017年〜）**
- コードの「見た目（インデント、改行、クォート等）」を自動整形するFormatter。
- 人によって違うスタイルを強制的に統一する。

### 基本的な使い方

```bash
# Lintチェック
pnpm lint

# 自動修正
pnpm lint --fix

# フォーマット
pnpm format
```

### メリット

- チームでコードスタイルを統一できる
- バグになりやすい書き方を事前に検出できる
- VS Codeと連携して「保存時に自動フォーマット」が可能

### デメリット

- 設定が複雑になりやすい（NuxtのESLint moduleを使うと最小限の設定で済む）

---

*次のページ → Part 2: セットアップ手順 1〜10*
