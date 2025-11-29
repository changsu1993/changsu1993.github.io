---
title: "Vite 완벽 가이드 - 차세대 번들러로 개발 경험 혁신하기"
description: "Vite의 핵심 원리부터 실무 활용까지. Native ES Modules와 esbuild로 빠른 개발 경험을 제공하는 Vite의 모든 것을 알아봅니다."
date: 2025-11-29 10:00:00 +0900
categories: [Frontend, Build Tools]
tags: [vite, bundler, webpack, esbuild, frontend, hmr, rollup]
---

## Vite란?

Vite(비트, 프랑스어로 "빠른")는 Vue.js의 창시자 Evan You가 만든 차세대 프론트엔드 빌드 툴입니다. 2020년 처음 공개된 이후, 빠른 속도와 뛰어난 개발 경험으로 많은 개발자들의 선택을 받고 있습니다.

### Vite의 탄생 배경

기존 번들러(Webpack, Parcel 등)는 다음과 같은 문제점을 가지고 있었습니다:

1. **느린 개발 서버 시작 시간**: 전체 애플리케이션을 번들링한 후에야 개발 서버가 시작됩니다.
2. **느린 HMR**: 파일을 수정할 때마다 전체 모듈 그래프를 재구성해야 합니다.
3. **복잡한 설정**: Webpack 설정 파일은 수백 줄에 달하는 경우가 많습니다.
4. **확장성 문제**: 프로젝트가 커질수록 개발 서버와 빌드 시간이 선형적으로 증가합니다.

Vite는 이러한 문제를 해결하기 위해 브라우저의 Native ES Modules를 활용하고, esbuild와 Rollup을 조합하여 획기적인 성능 향상을 이루어냈습니다.

## Vite vs Webpack 비교

### 개발 서버 시작 속도

**Webpack**의 경우:
```
Entry → Bundling → Dev Server Ready
(20~30초, 대형 프로젝트의 경우 1분 이상)
```

**Vite**의 경우:
```
Dependencies Pre-bundling → Dev Server Ready
(1~3초, 프로젝트 크기와 무관)
```

### 실제 벤치마크 결과

중형 React 프로젝트(~500개 컴포넌트) 기준:

| 항목 | Webpack 5 | Vite 5 | 성능 향상 |
|------|-----------|--------|-----------|
| 초기 시작 | 28초 | 2.1초 | **13배** |
| HMR | 1.2초 | 0.05초 | **24배** |
| 프로덕션 빌드 | 45초 | 22초 | **2배** |

### 주요 차이점

| 특징 | Webpack | Vite |
|------|---------|------|
| 개발 서버 | 번들 기반 | Native ESM 기반 |
| HMR | 전체 재번들링 | 정확한 모듈만 교체 |
| 설정 복잡도 | 높음 | 낮음 |
| 플러그인 생태계 | 방대함 | 빠르게 성장 중 |
| 프로덕션 빌드 | Webpack | Rollup |

## Vite의 핵심 원리

### 1. Native ES Modules 활용

Vite는 개발 환경에서 번들링을 하지 않고, 브라우저의 Native ES Modules를 직접 활용합니다.

```tsx
// src/App.tsx
import { useState } from 'react';
import Header from './components/Header';
import Footer from './components/Footer';

function App() {
  const [count, setCount] = useState(0);

  return (
    <div className="app">
      <Header />
      <main>
        <button onClick={() => setCount(count + 1)}>
          Count: {count}
        </button>
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

브라우저는 위 코드를 다음과 같이 로드합니다:

```
GET /src/App.tsx
GET /src/components/Header.tsx
GET /src/components/Footer.tsx
GET /node_modules/react/index.js
```

각 파일이 개별 HTTP 요청으로 로드되며, 변경된 파일만 다시 요청하므로 HMR이 매우 빠릅니다.

### 2. esbuild를 이용한 사전 번들링

node_modules의 의존성은 변경이 드물기 때문에 Vite는 이를 esbuild로 사전 번들링합니다.

```typescript
// Vite가 자동으로 수행하는 작업
node_modules/lodash-es/*.js → .vite/deps/lodash-es.js (단일 파일)
```

**esbuild의 장점**:
- Go 언어로 작성되어 JavaScript 기반 번들러보다 10~100배 빠름
- 병렬 처리를 통한 최적화
- 메모리 효율적

### 3. Dependency Pre-Bundling

Vite는 다음 두 가지 이유로 의존성을 사전 번들링합니다:

**1) CommonJS → ESM 변환**
```javascript
// node_modules/some-package/index.js (CommonJS)
module.exports = { foo: 'bar' };

// Vite가 변환
export default { foo: 'bar' };
export const foo = 'bar';
```

**2) 성능 최적화**
```javascript
// lodash-es는 600개 이상의 모듈로 분리되어 있음
import { debounce } from 'lodash-es';

// Vite가 단일 파일로 번들링
// .vite/deps/lodash-es.js
```

### 4. Rollup 기반 프로덕션 빌드

개발 환경과 달리, 프로덕션 빌드는 Rollup을 사용합니다:

- **Tree-shaking**: 사용하지 않는 코드 제거
- **Code Splitting**: 효율적인 청크 분리
- **압축**: Terser를 통한 최적화
- **레거시 브라우저 지원**: 필요시 폴리필 자동 추가

## Vite 프로젝트 시작하기

### React + TypeScript 프로젝트 생성

```bash
# npm
npm create vite@latest my-react-app -- --template react-ts

# yarn
yarn create vite my-react-app --template react-ts

# pnpm
pnpm create vite my-react-app --template react-ts
```

### 프로젝트 구조

```
my-react-app/
├── node_modules/
├── public/              # 정적 파일 (복사만 됨)
│   └── vite.svg
├── src/
│   ├── assets/          # 빌드 시 처리되는 자산
│   ├── components/
│   ├── App.tsx
│   ├── main.tsx         # 진입점
│   └── vite-env.d.ts    # Vite 타입 정의
├── index.html           # 진입 HTML (중요!)
├── package.json
├── tsconfig.json
├── tsconfig.node.json   # Vite 설정용 tsconfig
└── vite.config.ts       # Vite 설정
```

**중요**: Vite는 `index.html`을 프로젝트의 진입점으로 사용합니다. Webpack과 달리 `public` 폴더가 아닌 **루트 디렉토리**에 위치합니다.

```html
<!-- index.html -->
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 다른 프레임워크 지원

```bash
# Vue
npm create vite@latest my-vue-app -- --template vue-ts

# Svelte
npm create vite@latest my-svelte-app -- --template svelte-ts

# Solid
npm create vite@latest my-solid-app -- --template solid-ts

# Preact
npm create vite@latest my-preact-app -- --template preact-ts

# Vanilla TypeScript
npm create vite@latest my-vanilla-app -- --template vanilla-ts
```

## Vite 설정 (vite.config.ts)

### 기본 설정

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true, // 브라우저 자동 열기
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### 경로 별칭 (Path Alias)

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@assets': path.resolve(__dirname, './src/assets'),
    },
  },
});
```

TypeScript 설정도 함께 업데이트:

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@utils/*": ["./src/utils/*"],
      "@assets/*": ["./src/assets/*"]
    }
  }
}
```

사용 예시:

```tsx
// Before
import Button from '../../components/common/Button';
import { useAuth } from '../../hooks/useAuth';

// After
import Button from '@components/common/Button';
import { useAuth } from '@hooks/useAuth';
```

### 환경 변수 설정

Vite는 `VITE_` 접두사가 붙은 환경 변수만 클라이언트에 노출합니다.

```bash
# .env
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My Awesome App
DB_PASSWORD=secret123  # VITE_ 접두사가 없어서 노출되지 않음
```

```typescript
// src/config.ts
export const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  appTitle: import.meta.env.VITE_APP_TITLE,
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
};
```

환경별 파일:
```
.env                # 모든 환경
.env.local          # 로컬 환경 (git ignore)
.env.development    # 개발 환경
.env.production     # 프로덕션 환경
```

TypeScript 타입 정의:

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 프록시 설정

개발 중 CORS 문제를 해결하기 위한 프록시 설정:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
      '/ws': {
        target: 'ws://localhost:8080',
        ws: true,
      },
    },
  },
});
```

사용 예시:

```typescript
// 개발 환경에서 /api/users → http://localhost:8080/users로 프록시됨
const response = await fetch('/api/users');
const users = await response.json();
```

### 빌드 최적화 옵션

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    // 출력 디렉토리
    outDir: 'dist',

    // 정적 자산을 base64로 인라인할 크기 (바이트)
    assetsInlineLimit: 4096,

    // CSS 코드 스플리팅
    cssCodeSplit: true,

    // 소스맵 생성
    sourcemap: false,

    // Rollup 옵션
    rollupOptions: {
      output: {
        // 청크 분리 전략
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
        // 청크 파일명 패턴
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: '[ext]/[name]-[hash].[ext]',
      },
    },

    // Terser 압축 옵션
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },

    // 청크 크기 경고 임계값 (KB)
    chunkSizeWarningLimit: 500,
  },
});
```

## 플러그인 생태계

### 공식 플러그인

**@vitejs/plugin-react**
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // Fast Refresh 옵션
      fastRefresh: true,

      // Babel 옵션
      babel: {
        plugins: ['babel-plugin-styled-components'],
      },
    }),
  ],
});
```

**@vitejs/plugin-legacy** (레거시 브라우저 지원)
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import legacy from '@vitejs/plugin-legacy';

export default defineConfig({
  plugins: [
    react(),
    legacy({
      targets: ['defaults', 'not IE 11'],
    }),
  ],
});
```

### 유용한 커뮤니티 플러그인

**vite-plugin-svgr** (SVG를 React 컴포넌트로)
```bash
npm install vite-plugin-svgr -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import svgr from 'vite-plugin-svgr';

export default defineConfig({
  plugins: [react(), svgr()],
});
```

```tsx
// 사용 예시
import { ReactComponent as Logo } from './logo.svg';

function App() {
  return <Logo width={100} height={100} />;
}
```

**vite-tsconfig-paths** (tsconfig paths 자동 해석)
```bash
npm install vite-tsconfig-paths -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
});
```

**vite-plugin-compression** (Gzip/Brotli 압축)
```bash
npm install vite-plugin-compression -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import compression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    compression({
      algorithm: 'brotliCompress',
      ext: '.br',
    }),
  ],
});
```

### 커스텀 플러그인 만들기

Vite 플러그인은 Rollup 플러그인과 호환됩니다.

```typescript
// plugins/html-transform.ts
import type { Plugin } from 'vite';

export function htmlTransformPlugin(): Plugin {
  return {
    name: 'html-transform',
    transformIndexHtml(html) {
      return html.replace(
        /<title>(.*?)<\/title>/,
        `<title>$1 - ${new Date().getFullYear()}</title>`
      );
    },
  };
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { htmlTransformPlugin } from './plugins/html-transform';

export default defineConfig({
  plugins: [react(), htmlTransformPlugin()],
});
```

## CRA에서 Vite로 마이그레이션

### 단계별 마이그레이션 가이드

**1단계: 의존성 변경**

```bash
# CRA 관련 패키지 제거
npm uninstall react-scripts

# Vite 및 관련 패키지 설치
npm install vite @vitejs/plugin-react -D
npm install @types/node -D  # path 모듈 사용 위해
```

**2단계: package.json 스크립트 수정**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  }
}
```

**3단계: index.html 이동 및 수정**

```bash
# public/index.html → index.html (루트로 이동)
mv public/index.html .
```

```html
<!-- index.html -->
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <!-- %PUBLIC_URL% 제거 -->
    <!-- <script> 태그를 type="module"로 변경 -->
    <script type="module" src="/src/index.tsx"></script>
  </body>
</html>
```

**4단계: vite.config.ts 생성**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
  },
});
```

**5단계: 환경 변수 변환**

```bash
# .env
# Before (CRA)
REACT_APP_API_URL=https://api.example.com

# After (Vite)
VITE_API_URL=https://api.example.com
```

```typescript
// 코드 변경
// Before
const apiUrl = process.env.REACT_APP_API_URL;

// After
const apiUrl = import.meta.env.VITE_API_URL;
```

**6단계: tsconfig.json 업데이트**

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```jsonc
// tsconfig.node.json (새로 생성)
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

**7단계: vite-env.d.ts 추가**

```typescript
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  // 다른 환경 변수들...
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 주의사항 및 호환성 이슈

**1) require() 사용 불가**
```typescript
// ❌ 작동하지 않음
const logo = require('./logo.png');

// ✅ ES Module import 사용
import logo from './logo.png';
```

**2) 동적 import**
```typescript
// ❌ 변수를 사용한 동적 import는 제한적
const moduleName = 'Component';
import(`./components/${moduleName}.tsx`);

// ✅ import.meta.glob 사용
const modules = import.meta.glob('./components/*.tsx');
const Component = await modules[`./components/${name}.tsx`]();
```

**3) global 객체**
```typescript
// ❌ Node.js의 global
global.myVar = 'value';

// ✅ window 사용 (브라우저 환경)
window.myVar = 'value';
```

**4) process.env**
```typescript
// ❌ Node.js process
if (process.env.NODE_ENV === 'production') { }

// ✅ import.meta.env
if (import.meta.env.PROD) { }
```

## 프로덕션 빌드 최적화

### 코드 스플리팅

**동적 import 사용**
```tsx
import { lazy, Suspense } from 'react';

// 라우트 기반 코드 스플리팅
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```

**컴포넌트 단위 스플리팅**
```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./components/HeavyChart'));

function Analytics() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>

      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

### 청크 최적화

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          // node_modules 청크 분리
          if (id.includes('node_modules')) {
            // React 관련 라이브러리
            if (id.includes('react') || id.includes('react-dom')) {
              return 'react-vendor';
            }

            // 라우터 라이브러리
            if (id.includes('react-router')) {
              return 'router';
            }

            // UI 라이브러리
            if (id.includes('@mui') || id.includes('antd')) {
              return 'ui-vendor';
            }

            // 유틸리티 라이브러리
            if (id.includes('lodash') || id.includes('date-fns')) {
              return 'utils';
            }

            // 나머지 node_modules
            return 'vendor';
          }
        },
      },
    },
  },
});
```

### 압축 설정

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import compression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    // Gzip 압축
    compression({
      algorithm: 'gzip',
      ext: '.gz',
    }),
    // Brotli 압축 (더 높은 압축률)
    compression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 10240, // 10KB 이상만 압축
    }),
  ],
  build: {
    // 압축 최적화
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
        pure_funcs: ['console.log', 'console.info'],
      },
    },
  },
});
```

### 정적 자산 처리

**이미지 최적화**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
  plugins: [react(), imagetools()],
});
```

```tsx
// 사용 예시
import heroImage from './hero.jpg?w=800&format=webp';
import heroImageSmall from './hero.jpg?w=400&format=webp';

function Hero() {
  return (
    <picture>
      <source srcSet={heroImageSmall} media="(max-width: 600px)" />
      <img src={heroImage} alt="Hero" />
    </picture>
  );
}
```

**SVG 최적화**
```bash
npm install vite-plugin-svgr -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import svgr from 'vite-plugin-svgr';

export default defineConfig({
  plugins: [
    react(),
    svgr({
      svgrOptions: {
        icon: true, // viewBox 제거하고 크기 조정 가능하게
      },
    }),
  ],
});
```

## 실무 팁

### 모노레포에서 Vite 사용

```typescript
// packages/app/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@shared': path.resolve(__dirname, '../shared/src'),
      '@ui': path.resolve(__dirname, '../ui/src'),
    },
  },
  optimizeDeps: {
    // 워크스페이스 패키지를 명시적으로 포함
    include: ['@my-workspace/shared', '@my-workspace/ui'],
  },
});
```

### SSR (Server-Side Rendering) 지원

**서버 엔트리 포인트**
```typescript
// src/entry-server.tsx
import { renderToString } from 'react-dom/server';
import App from './App';

export function render() {
  const html = renderToString(<App />);
  return { html };
}
```

**Vite SSR 설정**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    ssr: true,
  },
  ssr: {
    noExternal: ['react', 'react-dom'],
  },
});
```

### 라이브러리 모드

라이브러리를 개발할 때 Vite를 사용할 수 있습니다.

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.ts'),
      name: 'MyLibrary',
      fileName: (format) => `my-library.${format}.js`,
      formats: ['es', 'umd'],
    },
    rollupOptions: {
      // 라이브러리에 번들하지 않을 외부 의존성
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
});
```

### 트러블슈팅

**1) "Failed to resolve import" 오류**
```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: ['problematic-package'],
  },
});
```

**2) "Cannot find module" TypeScript 오류**
```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true
  }
}
```

**3) 느린 개발 서버**
```typescript
// vite.config.ts
export default defineConfig({
  server: {
    fs: {
      // 허용할 디렉토리 명시
      allow: ['..'],
    },
  },
  optimizeDeps: {
    // 자주 변경되지 않는 큰 의존성 명시
    include: ['react', 'react-dom', 'lodash-es'],
  },
});
```

**4) 메모리 부족 오류**
```jsonc
// package.json
{
  "scripts": {
    "build": "NODE_OPTIONS='--max-old-space-size=4096' vite build"
  }
}
```

**5) import.meta.glob 제대로 작동하지 않음**
```typescript
// ❌ 동적 경로는 지원하지 않음
const modules = import.meta.glob(`./locales/${language}/*.json`);

// ✅ 정적 glob 패턴 사용
const modules = import.meta.glob('./locales/**/*.json');
const filtered = Object.keys(modules)
  .filter(key => key.includes(language))
  .reduce((acc, key) => {
    acc[key] = modules[key];
    return acc;
  }, {});
```

## 성능 모니터링

개발 중 성능을 모니터링하려면:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // 청크 정보 출력
        manualChunks(id) {
          if (id.includes('node_modules')) {
            const module = id.split('node_modules/')[1].split('/')[0];
            console.log(`Module: ${module}, Size: ${id.length}`);
            return `vendor-${module}`;
          }
        },
      },
    },
  },
});
```

빌드 후 번들 분석:

```bash
# rollup-plugin-visualizer 설치
npm install rollup-plugin-visualizer -D
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

## 결론

Vite는 단순히 빠른 번들러를 넘어, 모던 프론트엔드 개발의 새로운 표준이 되어가고 있습니다. Native ES Modules와 esbuild를 활용한 혁신적인 접근 방식은 개발 경험을 획기적으로 개선했습니다.

**Vite의 핵심 장점**:
- **압도적인 속도**: 개발 서버 시작과 HMR이 매우 빠름
- **간단한 설정**: 복잡한 Webpack 설정 없이도 대부분의 경우 기본 설정만으로 충분
- **모던 기술 스택**: ESM, esbuild, Rollup 등 최신 기술 활용
- **뛰어난 개발 경험**: 빠른 피드백으로 생산성 향상

**Vite를 사용하면 좋은 경우**:
- 새로운 프로젝트 시작
- 빠른 개발 속도가 중요한 프로젝트
- 모던 브라우저를 타겟으로 하는 애플리케이션
- React, Vue, Svelte 등 주요 프레임워크 사용

**주의가 필요한 경우**:
- 매우 복잡한 레거시 빌드 설정이 필요한 경우
- 특정 Webpack 플러그인에 강하게 의존하는 경우
- IE11 등 구형 브라우저를 반드시 지원해야 하는 경우

Vite는 계속해서 발전하고 있으며, 커뮤니티의 지원도 빠르게 성장하고 있습니다. 특히 Vite 5부터는 안정성과 성능이 크게 향상되어 프로덕션 환경에서도 안심하고 사용할 수 있습니다. 아직 Webpack을 사용하고 있다면, 이번 기회에 Vite로 전환을 고려해보는 것을 추천합니다.
