# Copilot Instructions - React Router v7 + React 19 + TailwindCSS v4 + TypeScript v5 + Vite v7

## 全般的な指針

- **パッケージマネージャー**: 常にpnpmを使用する
- **言語**: TypeScript 5を使用し、厳密な型指定を行う
- **コメント**: すべてのコードに適切なコメントを追加する
- **命名規則**: 
  - 変数名：camelCase
  - クラス名：PascalCase
  - コンポーネント名：PascalCase
  - ファイル名：kebab-case（コンポーネントはPascalCase）

## React 19 ベストプラクティス

### コンポーネント作成
```tsx
// 関数コンポーネントを使用し、React.FCは避ける
interface UserProfileProps {
  userId: string;
  className?: string;
}

export function UserProfile({ userId, className }: UserProfileProps) {
  // useTransitionを活用してユーザー体験を向上
  const [isPending, startTransition] = useTransition();
  
  return (
    <div className={className}>
      {/* コンポーネントの内容 */}
    </div>
  );
}
```

### Hooks使用ガイドライン
- `useState`よりも`useReducer`を複雑な状態管理で使用
- `useMemo`と`useCallback`は適切に使用（過度な最適化は避ける）
- カスタムHooksで再利用可能なロジックを抽出
- `useTransition`を使用してパフォーマンス向上

### エラーハンドリング
```tsx
// Error Boundaryを適切に配置
function ErrorFallback({ error }: { error: Error }) {
  return (
    <div role="alert" className="error-container">
      <h2>何かが間違っています</h2>
      <pre>{error.message}</pre>
    </div>
  );
}
```

## React Router v7 (App Router) ベストプラクティス

### ルート定義
```tsx
// routes.ts - 型安全なルート定義
import {
  type RouteConfig,
  route,
  index,
  layout,
  prefix,
} from "@react-router/dev/routes";

export default [
  // インデックスルート（ホームページ）
  index("./routes/home.tsx"),
  
  // 単一ルート
  route("about", "./routes/about.tsx"),
  
  // パラメータ付きルート
  route("photos/:id", "./routes/photo-detail.tsx"),
  
  // ネストされたルート
  route("gallery", "./routes/gallery.tsx", [
    index("./routes/gallery.home.tsx"),
    route("albums/:albumId", "./routes/gallery.album.tsx"),
  ]),
  
  // プレフィックス付きルート
  ...prefix("admin", [
    index("./routes/admin.home.tsx"),
    route("users", "./routes/admin.users.tsx"),
    route("settings", "./routes/admin.settings.tsx"),
  ]),
  
  // レイアウトルート（URLパスを追加せずにネスト）
  layout("./layouts/auth-layout.tsx", [
    route("login", "./routes/auth.login.tsx"),
    route("register", "./routes/auth.register.tsx"),
  ]),
] satisfies RouteConfig;
```

### ページコンポーネント
```tsx
// routes/photo-detail.tsx
import type { Route } from "./+types/photo-detail";

// メタデータの定義
export function meta({ params }: Route.MetaArgs) {
  return [
    { title: `フォト ${params.id}` },
    { name: "description", content: "写真の詳細ページ" },
  ];
}

// データローダー
export async function loader({ params }: Route.LoaderArgs) {
  const photo = await getPhoto(params.id);
  if (!photo) {
    throw new Response("Not Found", { status: 404 });
  }
  return { photo };
}

// ページコンポーネント（デフォルトエクスポート）
export default function PhotoDetail({ loaderData }: Route.ComponentProps) {
  const { photo } = loaderData;
  
  return (
    <main className="container mx-auto px-4 py-8">
      <img 
        src={photo.url} 
        alt={photo.title}
        className="w-full max-w-4xl mx-auto rounded-lg shadow-lg"
      />
    </main>
  );
}
```

### ネストされたルートとOutlet
```tsx
// routes/gallery.tsx（親ルート）
import { Outlet } from "react-router";
import type { Route } from "./+types/gallery";

export async function loader() {
  const categories = await getPhotoCategories();
  return { categories };
}

export default function Gallery({ loaderData }: Route.ComponentProps) {
  const { categories } = loaderData;
  
  return (
    <div className="min-h-screen">
      <header className="bg-white shadow">
        <h1 className="text-2xl font-bold">フォトギャラリー</h1>
        <nav>
          {categories.map(category => (
            <Link 
              key={category.id} 
              to={`/gallery/albums/${category.id}`}
              className="text-blue-600 hover:text-blue-800"
            >
              {category.name}
            </Link>
          ))}
        </nav>
      </header>
      {/* 子ルートがここにレンダリングされる */}
      <Outlet />
    </div>
  );
}
```

### ナビゲーション
```tsx
// Link、NavLinkを適切に使用
import { Link, NavLink } from "react-router";

function Navigation() {
  return (
    <nav className="flex space-x-4">
      <NavLink 
        to="/" 
        className={({ isActive }) => 
          `px-3 py-2 rounded-md ${isActive ? 'bg-blue-500 text-white' : 'text-gray-700'}`
        }
      >
        ホーム
      </NavLink>
      <Link to="/gallery" className="px-3 py-2 text-gray-700 hover:text-blue-500">
        ギャラリー
      </Link>
    </nav>
  );
}
```

## TailwindCSS v4 ベストプラクティス

### クラス命名とコンポーネント設計
```tsx
// コンポーネント固有のスタイリング
function PhotoCard({ photo }: { photo: Photo }) {
  return (
    <div className="group relative overflow-hidden rounded-lg bg-white shadow-md transition-all hover:shadow-xl">
      <img 
        src={photo.thumbnail} 
        alt={photo.title}
        className="aspect-square w-full object-cover transition-transform group-hover:scale-105"
      />
      <div className="absolute inset-0 bg-gradient-to-t from-black/50 to-transparent opacity-0 transition-opacity group-hover:opacity-100">
        <div className="absolute bottom-4 left-4 text-white">
          <h3 className="text-lg font-semibold">{photo.title}</h3>
          <p className="text-sm opacity-90">{photo.date}</p>
        </div>
      </div>
    </div>
  );
}
```

### レスポンシブデザイン
```tsx
// モバイルファーストアプローチ
function Gallery({ photos }: { photos: Photo[] }) {
  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5">
      {photos.map(photo => (
        <PhotoCard key={photo.id} photo={photo} />
      ))}
    </div>
  );
}
```

### アクセシビリティ
```tsx
// focus-visible、aria属性、semantic HTMLを使用
function ImageModal({ isOpen, onClose, photo }: ImageModalProps) {
  return (
    <dialog 
      open={isOpen}
      className="fixed inset-0 z-50 flex items-center justify-center bg-black/80 backdrop-blur-sm"
      aria-labelledby="modal-title"
      aria-describedby="modal-description"
    >
      <div className="relative max-h-[90vh] max-w-[90vw] rounded-lg bg-white p-4">
        <button
          onClick={onClose}
          className="absolute right-2 top-2 rounded-full bg-gray-100 p-2 text-gray-600 hover:bg-gray-200 focus-visible:ring-2 focus-visible:ring-blue-500"
          aria-label="モーダルを閉じる"
        >
          ✕
        </button>
        <img 
          src={photo.url} 
          alt={photo.title}
          className="h-auto w-full object-contain"
        />
      </div>
    </dialog>
  );
}
```

## TypeScript 5 ベストプラクティス

### 型定義
```tsx
// 明確で再利用可能な型定義
interface Photo {
  readonly id: string;
  title: string;
  url: string;
  thumbnail: string;
  date: Date;
  tags: readonly string[];
  metadata: PhotoMetadata;
}

interface PhotoMetadata {
  camera?: string;
  lens?: string;
  settings?: CameraSettings;
  location?: GeolocationCoordinates;
}

// ユニオン型とリテラル型の活用
type ViewMode = 'grid' | 'list' | 'masonry';
type SortOrder = 'date-asc' | 'date-desc' | 'title-asc' | 'title-desc';
```

### 型ガード
```tsx
// 型ガードの適切な使用
function isPhoto(item: unknown): item is Photo {
  return (
    typeof item === 'object' &&
    item !== null &&
    'id' in item &&
    'title' in item &&
    'url' in item
  );
}

// 条件付き型の活用
type PhotoWithOptionalMetadata<T extends boolean> = T extends true 
  ? Photo & { metadata: PhotoMetadata }
  : Omit<Photo, 'metadata'>;
```

### ジェネリクス
```tsx
// 再利用可能なジェネリック関数
function createApiHook<TData, TParams = void>(
  endpoint: string
) {
  return function useApiData(params?: TParams) {
    const [data, setData] = useState<TData | null>(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<Error | null>(null);
    
    // API呼び出しロジック
    
    return { data, loading, error };
  };
}

// 使用例
const usePhotos = createApiHook<Photo[], { albumId?: string }>('/api/photos');
```

## Vite v7 ベストプラクティス

### 設定ファイル
```typescript
// vite.config.ts - パフォーマンス最適化
import { defineConfig } from 'vite';
import { reactRouter } from '@react-router/dev/vite';
import tailwindcss from '@tailwindcss/vite';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [
    reactRouter(),
    tailwindcss(),
    tsconfigPaths(), // TypeScriptパスマッピング
  ],
  // 開発サーバー設定
  server: {
    port: 3000,
    host: true, // ネットワーク経由でのアクセスを許可
  },
  // ビルド最適化
  build: {
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          // ベンダーチャンクの分割
          vendor: ['react', 'react-dom'],
          router: ['react-router'],
        },
      },
    },
  },
  // 環境変数プレフィックス
  envPrefix: 'VITE_',
});
```

### パフォーマンス最適化
```tsx
// 動的インポートによるコード分割
const PhotoEditor = lazy(() => import('./PhotoEditor'));
const ImageProcessingWorker = lazy(() => import('./ImageProcessingWorker'));

// 遅延ローディング
function Gallery() {
  return (
    <Suspense fallback={<div className="animate-pulse">読み込み中...</div>}>
      <PhotoGrid />
    </Suspense>
  );
}
```

## プロジェクト構造

```
app/
├── components/           # 再利用可能なコンポーネント
│   ├── ui/              # 基本UIコンポーネント
│   ├── layout/          # レイアウトコンポーネント
│   └── feature/         # 機能固有のコンポーネント
├── hooks/               # カスタムHooks
├── lib/                 # ユーティリティ関数
├── types/               # 型定義
├── routes/              # ページコンポーネント
└── styles/              # グローバルスタイル
```

## 開発ワークフロー

### 1. 新機能開発時
1. 型定義から開始
2. コンポーネント設計（小さく、再利用可能に）
3. ルート定義（必要に応じて）
4. スタイリング（TailwindCSSクラス）
5. テスト記述

### 2. パフォーマンス考慮事項
- React.memoは必要な場合のみ使用
- useCallbackとuseMemoの適切な使用
- 画像の遅延読み込み実装
- バンドルサイズの監視

### 3. アクセシビリティ
- セマンティックHTML使用
- ARIA属性の適切な設定
- キーボードナビゲーション対応
- カラーコントラスト確保

これらのガイドラインに従ってコードを生成し、保守性とパフォーマンスを重視した開発を行ってください。
