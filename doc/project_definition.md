# Absence Reporter - プロジェクト定義書

## 1. プロジェクト概要
- **アプリ名**: Absence Reporter（欠勤連絡アプリ）
- **目的**: 小規模組織（約40名）における欠勤情報のリアルタイム共有。
- **コアバリュー**: 軽量、低コスト（Firebase無料枠）、スマホ対応、即時同期。
- **制約事項**: 原則として「当日」のデータのみを扱う（表示・管理のシンプル化）。

## 2. ターゲットユーザー
1. **一般従業員**: 自身の欠勤登録（名前、理由、期間、備考）を行う。
2. **管理者**: 今日の欠勤状況をリアルタイム・ダッシュボードで確認する。

## 3. 技術スタック（TypeScript必須）
- **フロントエンド**: Nuxt.js (Vue 3, Composition API)
- **言語**: TypeScript (厳格すぎない設定を推奨)
- **バックエンド / インフラ**: Firebase
    - **Cloud Firestore**: リアルタイムDB (NoSQL)
    - **Firebase Hosting**: 静的サイトホスティング
- **状態管理**: Pinia
- **スタイリング**: Vanilla CSS (軽量性とパフォーマンスを優先)
- **テスト**: Vitest (ロジック), Playwright (UI/E2E - 必要に応じて)
- **パッケージ管理**: pnpm

## 4. 機能要件
### 4.1 欠勤登録 (Absence Entry)
- **入力項目**:
    - `name`: 氏名 (string)
    - `reason`: 理由 ('体調不良' | '私用' | 'その他')
    - `startDate`: 開始日 (ISO 8601 string)
    - `endDate`: 終了日 (ISO 8601 string)
    - `note`: 備考 (string, 任意)
- **動作**: 登録後、即座にFirestoreに反映。

### 4.2 ダッシュボード (List View)
- **表示**: Firestoreの `onSnapshot` によるリアルタイム更新。
- **フィルタ**: 当日データのみ表示（`startDate <= 今日 <= endDate` で判定。複数日欠勤も正しく表示される）。
- **ソート**: 新着順。

### 4.3 認証
- **方式**: 共有パスワード方式（簡易認証）。
- **ロジック**: フロントエンドでのゲートキーピング ＋ 可能であればFirebase 匿名認証を利用したセキュリティルールの制御。

## 5. データ設計 (TypeScript)
```typescript
export type Absence = {
  id?: string;
  name: string;
  reason: '体調不良' | '私用' | 'その他';
  startDate: string;
  endDate: string;
  note?: string;
  createdAt: any; // Firestore ServerTimestamp
}
```

## 6. 実装ポリシー (AIへの指示事項)
- **哲学**: MVP優先。過剰設計禁止。シンプルな抽象化。
- **コンポーネント**: `<script setup>` と Composition API を使用。
- **Linter/Formatter**: ESLint (Nuxt/TS推奨設定) + Prettier。
- **ディレクトリ構成**:
    - `/app`: Nuxtルート
    - `/components`: UIコンポーネント
    - `/pages`: ページビュー
    - `/stores`: Piniaストア
    - `/services`: Firestore/外部APIロジック
    - `/types`: 型定義
    - `/utils`: ヘルパー関数
- **Git戦略**: `develop` ブランチでフィーチャー単位の開発を行い、`main` にマージ。

## 7. 運用・パフォーマンス
- **データクリーンアップ**: 古いデータはUIで無視する（当日フィルタ）。
- **セキュリティ**: Firestoreセキュリティルールで不正な書き込みを防止。
- **パフォーマンス**: リアルタイムリスナーを最小限にし、当日のコレクションのみを購読。
