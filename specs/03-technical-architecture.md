# SubCheck 技術仕様

## 🛠 技術スタック

### フロントエンド

- **フレームワーク**: Next.js 14 (App Router)
- **言語**: TypeScript
- **スタイリング**: Tailwind CSS
- **UIコンポーネント**: shadcn/ui
- **グラフライブラリ**: Recharts
- **アニメーション**: Framer Motion（オプション）
- **アイコン**: Lucide React

### データ管理

- **ローカルストレージ**: ブラウザの localStorage
- **状態管理**: React useState/useReducer
- **将来的**: Firebase/Supabase を検討

### ホスティング・デプロイ

- **メインホスティング**: Vercel（推奨）
- **代替**: Netlify
- **CDN**: Vercel Edge Network
- **ドメイン**: subcheck.app（予定）

### 開発・ビルドツール

- **パッケージマネージャ**: npm/yarn
- **リンター**: ESLint + Prettier
- **TypeScript**: strict mode
- **テスト**: Jest + React Testing Library（将来）

### 分析・監視

- **アナリティクス**: Google Analytics 4
- **エラー監視**: Sentry（将来）
- **パフォーマンス**: Vercel Analytics

## 📁 プロジェクト構造

```
subcheck/
├── app/                          # Next.js App Router
│   ├── page.tsx                  # トップページ
│   ├── select/
│   │   └── page.tsx             # サブスク選択画面
│   ├── frequency/
│   │   └── page.tsx             # 使用頻度入力画面
│   ├── result/
│   │   └── page.tsx             # 診断結果表示
│   ├── layout.tsx               # ルートレイアウト
│   ├── globals.css              # グローバルスタイル
│   └── api/                     # API Routes（将来）
│       └── share/
│           └── route.ts         # OGP画像生成
├── components/                   # React コンポーネント
│   ├── ui/                      # shadcn/ui コンポーネント
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── progress.tsx
│   │   └── ...
│   ├── SubscriptionCard.tsx     # サブスクカード
│   ├── FrequencySelector.tsx    # 使用頻度選択UI
│   ├── ResultChart.tsx          # 診断結果グラフ
│   ├── ShareButtons.tsx         # SNSシェアボタン
│   ├── ProgressBar.tsx          # 進捗表示
│   └── Layout/
│       ├── Header.tsx
│       ├── Footer.tsx
│       └── Navigation.tsx
├── lib/                         # ロジック・ユーティリティ
│   ├── subscriptions.ts         # サブスクマスターデータ
│   ├── calculator.ts            # 診断ロジック
│   ├── storage.ts               # ローカルストレージ管理
│   ├── share.ts                 # SNSシェア機能
│   └── utils.ts                 # 汎用ユーティリティ
├── types/                       # TypeScript型定義
│   ├── subscription.ts          # サブスク関連の型
│   ├── diagnosis.ts             # 診断結果の型
│   └── index.ts                 # 型のエクスポート
├── data/                        # 静的データ
│   ├── subscriptions.json       # サブスクマスターデータ
│   └── comparisons.json         # 浪費額換算データ
├── public/                      # 静的ファイル
│   ├── logos/                   # サブスクロゴ画像
│   │   ├── netflix.png
│   │   ├── spotify.png
│   │   └── ...
│   ├── icons/                   # アイコン
│   └── images/                  # その他画像
├── hooks/                       # カスタムフック
│   ├── useLocalStorage.ts       # ローカルストレージフック
│   ├── useDiagnosis.ts         # 診断ロジックフック
│   └── useShare.ts             # シェア機能フック
└── styles/                      # スタイルファイル
    ├── globals.css              # グローバルCSS
    └── components.css           # コンポーネント用CSS
```

## 🔧 データ構造

### TypeScript型定義

```typescript
// types/subscription.ts
export interface Subscription {
  id: string;
  name: string;
  category: 'video' | 'music' | 'game' | 'news' | 'other';
  monthlyPrice: number;
  logo?: string;
  description?: string;
  color?: string;
}

// types/diagnosis.ts
export type FrequencyType = 'daily' | 'weekly' | 'monthly' | 'unused';

export interface UserSubscription {
  subscriptionId: string;
  frequency: FrequencyType;
  customPrice?: number;
  customName?: string;
}

export interface DiagnosisResult {
  subscriptions: UserSubscription[];
  totalMonthly: number;
  totalYearly: number;
  unusedYearly: number;
  wasteRate: number;
  frequencyBreakdown: {
    daily: number;
    weekly: number;
    monthly: number;
    unused: number;
  };
  recommendations: RecommendationItem[];
  createdAt: Date;
  shareId?: string;
}

export interface RecommendationItem {
  subscriptionId: string;
  action: 'cancel' | 'downgrade' | 'review';
  potentialSaving: number;
  reason: string;
}

export interface ComparisonItem {
  amount: number;
  description: string;
  icon: string;
}
```

### ローカルストレージ構造

```typescript
// localStorage keys
const STORAGE_KEYS = {
  USER_SUBSCRIPTIONS: 'subcheck_user_subscriptions',
  DIAGNOSIS_HISTORY: 'subcheck_diagnosis_history',
  USER_PREFERENCES: 'subcheck_user_preferences'
} as const;

// データ構造
interface StoredData {
  userSubscriptions: UserSubscription[];
  diagnosisHistory: DiagnosisResult[];
  userPreferences: {
    language: 'ja' | 'en';
    theme: 'light' | 'dark';
    shareSettings: {
      includeAmount: boolean;
      includeDetails: boolean;
    };
  };
}
```

## 🎨 UI/UXライブラリ設定

### Tailwind CSS設定

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
        accent: {
          orange: '#f59e0b',
          red: '#ef4444',
        },
        success: '#10b981',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      animation: {
        'bounce-gentle': 'bounce 2s infinite',
        'pulse-slow': 'pulse 3s infinite',
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
}
```

### shadcn/ui コンポーネント

使用予定のコンポーネント:

- Button
- Card
- Progress
- Badge
- Dialog
- Tabs
- RadioGroup
- Checkbox
- Input
- Label

## 📊 パフォーマンス最適化

### Next.js最適化

- **Image最適化**: next/image を使用
- **フォント最適化**: next/font を使用
- **動的インポート**: React.lazy + Suspense
- **静的生成**: ISG (Incremental Static Generation)

### バンドル最適化

- **Tree Shaking**: 未使用コードの除去
- **Code Splitting**: ページ単位での分割
- **動的インポート**: 重いライブラリの遅延読み込み

### キャッシュ戦略

- **静的アセット**: 長期キャッシュ（1年）
- **HTMLページ**: 短期キャッシュ（1時間）
- **API応答**: 適切なキャッシュヘッダー

## 🔒 セキュリティ

### データ保護

- **個人情報収集なし**: MVP版では不要
- **ローカルストレージのみ**: サーバーにデータ送信なし
- **HTTPS強制**: 本番環境では必須

### Content Security Policy

```javascript
// next.config.js
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: "default-src 'self'; script-src 'self' 'unsafe-eval'; style-src 'self' 'unsafe-inline'"
          }
        ]
      }
    ]
  }
}
```

## 🚀 デプロイ設定

### Vercel設定

```json
// vercel.json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "regions": ["nrt1"],
  "functions": {
    "app/api/**/*.ts": {
      "runtime": "@vercel/node"
    }
  }
}
```

### 環境変数

```bash
# .env.local
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
NEXT_PUBLIC_SITE_URL=https://subcheck.app
NEXT_PUBLIC_API_BASE_URL=https://api.subcheck.app
```

## 📈 監視・分析

### Google Analytics 4設定

- **ページビュー**: 自動追跡
- **カスタムイベント**:
  - 診断開始
  - 診断完了
  - SNSシェア
  - サブスク選択

### Core Web Vitals

- **LCP**: < 2.5秒
- **FID**: < 100ms
- **CLS**: < 0.1

## 🔗 関連ドキュメント

- [[01-requirements]] - 要件定義
- [[02-user-interface]] - UI/UX設計
- [[04-development-plan]] - 開発計画
