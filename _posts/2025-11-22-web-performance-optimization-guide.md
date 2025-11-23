---
title: "웹 성능 최적화 완벽 가이드 - Core Web Vitals부터 번들 최적화까지"
date: 2025-11-22 14:00:00 +0900
categories: [Frontend, Performance]
tags: [web-performance, core-web-vitals, lcp, inp, cls, vite, webpack, next-image, lazy-loading, 웹성능, 성능최적화]
description: "Core Web Vitals(LCP, INP, CLS) 측정과 개선, Vite/Webpack 번들 최적화, next/image와 lazy loading, 실제 성능 개선 사례까지. 프론트엔드 성능 최적화의 모든 것"
---

## 웹 성능이 중요한 이유

웹 성능은 단순히 기술적인 지표가 아닙니다. 사용자 경험, 비즈니스 성과, 그리고 검색 엔진 순위에 직접적인 영향을 미치는 핵심 요소입니다.

### 사용자 경험과 성능의 관계

사용자는 빠른 웹사이트를 기대합니다. 연구에 따르면:

- **페이지 로드 시간이 1초에서 3초로 증가**하면 이탈 확률이 **32% 증가**
- **로드 시간이 1초에서 5초로 증가**하면 이탈 확률이 **90% 증가**
- **모바일 사용자의 53%**는 3초 이상 로딩되는 사이트를 이탈

**사용자 기대치:**

| 로드 시간 | 사용자 반응 |
|-----------|-------------|
| 0-1초 | 즉각적 (Instant) |
| 1-2초 | 수용 가능 (Acceptable) |
| 2-3초 | 느려지고 있음 (Getting slower) |
| 3초+ | 이탈 가능성 높음 (Likely to leave) |

### 비즈니스 지표에 미치는 영향

웹 성능은 이탈률, 전환율, SEO 순위에 직접적인 영향을 미칩니다.

| 지표 | 성능 영향 |
|------|----------|
| **이탈률** | 페이지 로드 시간 1초 증가 시 이탈률 7% 증가 |
| **전환율** | 0.1초 개선 시 전환율 최대 8% 증가 (Walmart) |
| **SEO 순위** | Core Web Vitals가 Google 검색 랭킹 요소로 포함 |
| **사용자 만족도** | 빠른 사이트는 재방문율 25% 높음 |

### 실제 기업 사례와 데이터

#### Amazon
- **페이지 로드 시간 100ms 증가 시 매출 1% 감소**
- 이는 연간 약 16억 달러의 손실로 추산

#### Google
- **검색 결과 페이지가 0.5초 지연**되면 트래픽 **20% 감소**
- Speed Update(2018): 페이지 속도를 모바일 검색 랭킹 요소로 공식 채택

#### Pinterest
- **페이지 로드 시간 40% 단축 후**:
  - SEO 트래픽 **15% 증가**
  - 회원 가입률 **15% 증가**
  - 대기 시간 체감 **40% 감소**

#### Walmart
- **페이지 로드 시간 1초 개선 시 전환율 2% 증가**
- 100ms 개선당 점진적 수익 최대 1% 증가

```typescript
// 성능이 비즈니스에 미치는 영향 계산 예시
interface PerformanceImpact {
  currentLoadTime: number;       // 현재 로드 시간 (초)
  improvedLoadTime: number;      // 개선된 로드 시간 (초)
  monthlyVisitors: number;       // 월간 방문자 수
  conversionRate: number;        // 현재 전환율 (%)
  averageOrderValue: number;     // 평균 주문 금액
}

function calculateROI(impact: PerformanceImpact): number {
  const timeImprovement = impact.currentLoadTime - impact.improvedLoadTime;
  // 0.1초 개선당 전환율 0.8% 증가 (보수적 추정)
  const conversionIncrease = (timeImprovement / 0.1) * 0.008;
  const newConversionRate = impact.conversionRate * (1 + conversionIncrease);

  const currentRevenue = impact.monthlyVisitors *
    (impact.conversionRate / 100) * impact.averageOrderValue;
  const newRevenue = impact.monthlyVisitors *
    (newConversionRate / 100) * impact.averageOrderValue;

  return newRevenue - currentRevenue; // 월간 추가 수익
}

// 예시: 3초 → 1.5초 개선
const result = calculateROI({
  currentLoadTime: 3,
  improvedLoadTime: 1.5,
  monthlyVisitors: 100000,
  conversionRate: 2,
  averageOrderValue: 50
});

console.log(`월간 추가 예상 수익: $${result.toFixed(2)}`);
// 출력: 월간 추가 예상 수익: $12000.00
```

---

## Core Web Vitals 완벽 이해

Core Web Vitals는 Google이 정의한 웹 페이지의 사용자 경험을 측정하는 핵심 지표입니다. 2024년 3월부터 **INP(Interaction to Next Paint)**가 FID를 대체하여 새로운 Core Web Vitals 지표가 되었습니다.

### 세 가지 핵심 메트릭 개요

**Core Web Vitals (2024~)**

| 메트릭 | 전체 명칭 | 측정 영역 |
|--------|-----------|-----------|
| **LCP** | Largest Contentful Paint | 로딩 성능 |
| **INP** | Interaction to Next Paint | 상호작용 반응성 |
| **CLS** | Cumulative Layout Shift | 시각적 안정성 |

### LCP (Largest Contentful Paint)

**정의**: 뷰포트 내에서 가장 큰 콘텐츠 요소가 렌더링되는 시점

LCP가 측정하는 요소:
- `<img>` 요소
- `<svg>` 내부의 `<image>` 요소
- `<video>` 요소의 포스터 이미지
- CSS `background-image`로 로드된 이미지
- 텍스트 노드를 포함하는 블록 레벨 요소

#### 측정 기준값

| 등급 | 시간 | 사용자 경험 |
|------|------|------------|
| **Good** | 2.5초 이하 | 빠름 |
| **Needs Improvement** | 2.5초 ~ 4.0초 | 개선 필요 |
| **Poor** | 4.0초 초과 | 느림 |

#### LCP 개선 방법

```typescript
// 1. 리소스 우선순위 힌트 사용
// HTML에서 LCP 이미지에 fetchpriority 추가
const LCPImage = () => (
  <img
    src="/hero-image.webp"
    alt="Hero"
    fetchPriority="high"  // 우선순위 높음
    loading="eager"       // 즉시 로드
  />
);

// 2. Preload로 중요 리소스 미리 로드
// <link rel="preload" as="image" href="/hero-image.webp" fetchpriority="high">
```

```css
/* 3. 중요 CSS 인라인화 */
/* Critical CSS - LCP 요소 스타일을 <head>에 인라인 */
.hero-section {
  width: 100%;
  height: 60vh;
  background-color: #f0f0f0; /* placeholder 색상 */
}

.hero-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

```typescript
// 4. 서버 응답 시간 최적화 (TTFB 개선)
// Next.js에서 정적 생성 활용
export async function getStaticProps() {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 3600 // 1시간마다 재생성
  };
}
```

### INP (Interaction to Next Paint)

**정의**: 페이지 수명 동안 발생하는 모든 클릭, 탭, 키보드 상호작용의 지연 시간 중 가장 긴 상호작용을 측정 (2024년 3월부터 FID 대체)

INP는 FID와 달리 **첫 번째 상호작용만이 아닌 모든 상호작용**을 측정합니다.

#### INP 측정 과정

```
사용자 상호작용 → 입력 지연 → 처리 시간 → 렌더링 지연 → 다음 페인트
                  ←─────────────── INP 측정 범위 ───────────────→
```

#### 측정 기준값

| 등급 | 시간 | 사용자 경험 |
|------|------|------------|
| **Good** | 200ms 이하 | 반응이 빠름 |
| **Needs Improvement** | 200ms ~ 500ms | 약간의 지연 감지 |
| **Poor** | 500ms 초과 | 확실히 느림 |

#### INP 개선 방법

```typescript
// 1. 긴 작업 분할 (Long Task Breaking)
// Bad: 메인 스레드 블로킹
function processLargeArray(items: number[]) {
  items.forEach(item => {
    heavyComputation(item); // 전체가 동기적으로 실행
  });
}

// Good: 청크로 분할하여 처리
async function processLargeArrayOptimized(items: number[]) {
  const CHUNK_SIZE = 100;

  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);

    // 각 청크 처리 후 브라우저에 제어권 양보
    await new Promise(resolve => {
      requestIdleCallback(() => {
        chunk.forEach(item => heavyComputation(item));
        resolve(undefined);
      });
    });
  }
}
```

```typescript
// 2. 이벤트 핸들러 최적화
// Bad: 동기적 무거운 작업
const handleClick = () => {
  const result = expensiveCalculation(); // 메인 스레드 블로킹
  setData(result);
};

// Good: 비동기 처리와 로딩 상태 분리
const handleClickOptimized = async () => {
  setIsLoading(true);

  // 다음 프레임에서 처리하여 UI 먼저 업데이트
  await new Promise(resolve => requestAnimationFrame(resolve));

  const result = await new Promise(resolve => {
    requestIdleCallback(() => {
      resolve(expensiveCalculation());
    });
  });

  setData(result);
  setIsLoading(false);
};
```

```typescript
// 3. Web Worker로 무거운 연산 오프로드
// worker.ts
self.onmessage = (e: MessageEvent) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};

// main.ts
const worker = new Worker(new URL('./worker.ts', import.meta.url));

function processWithWorker(data: any): Promise<any> {
  return new Promise(resolve => {
    worker.onmessage = (e) => resolve(e.data);
    worker.postMessage(data);
  });
}

// 컴포넌트에서 사용
const handleClick = async () => {
  setIsLoading(true);
  const result = await processWithWorker(inputData);
  setData(result);
  setIsLoading(false);
};
```

### CLS (Cumulative Layout Shift)

**정의**: 페이지 로드 중 발생하는 예기치 않은 레이아웃 이동의 누적 점수

CLS는 뷰포트 내에서 보이는 요소가 시작 위치에서 예상치 못하게 이동할 때 측정됩니다.

#### 측정 기준값

| 등급 | 점수 | 사용자 경험 |
|------|------|------------|
| **Good** | 0.1 이하 | 안정적 |
| **Needs Improvement** | 0.1 ~ 0.25 | 약간의 이동 |
| **Poor** | 0.25 초과 | 불안정 |

#### CLS 계산 공식

```
Layout Shift Score = Impact Fraction × Distance Fraction

Impact Fraction: 불안정한 요소가 차지하는 뷰포트 비율
Distance Fraction: 요소가 이동한 거리 / 뷰포트 크기
```

#### CLS 개선 방법

```css
/* 1. 이미지/비디오에 크기 지정 */
/* Bad */
img {
  width: 100%;
}

/* Good - 종횡비 유지 */
img {
  width: 100%;
  aspect-ratio: 16 / 9;
}

/* 또는 HTML에서 직접 지정 */
/* <img src="..." width="800" height="450" alt="..."> */
```

```typescript
// 2. 동적 콘텐츠를 위한 자리 표시자
// Bad: 광고나 임베드가 갑자기 나타남
const AdBanner = () => {
  const [adLoaded, setAdLoaded] = useState(false);

  return adLoaded ? <AdContent /> : null; // 갑자기 공간 차지
};

// Good: 미리 공간 확보
const AdBannerOptimized = () => {
  const [adLoaded, setAdLoaded] = useState(false);

  return (
    <div
      style={{
        minHeight: '250px', // 광고 높이 예약
        backgroundColor: '#f0f0f0'
      }}
    >
      {adLoaded ? <AdContent /> : <AdPlaceholder />}
    </div>
  );
};
```

```css
/* 3. 폰트 로딩 최적화 */
/* font-display: swap 사용으로 FOIT 방지 */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap; /* 폴백 폰트 먼저 표시 */
}

/* 폰트 크기 조정으로 CLS 최소화 */
body {
  font-family: 'CustomFont', -apple-system, BlinkMacSystemFont, sans-serif;
  /* size-adjust로 폴백 폰트 크기 맞춤 */
}
```

```typescript
// 4. 애니메이션은 transform 사용
// Bad: top/left 변경은 레이아웃 이동 발생
const badAnimation = {
  initial: { top: 0 },
  animate: { top: 100 } // CLS 유발
};

// Good: transform은 레이아웃 이동 없음
const goodAnimation = {
  initial: { transform: 'translateY(0)' },
  animate: { transform: 'translateY(100px)' } // CLS 없음
};
```

### 측정 도구

#### Chrome DevTools Performance 패널

```typescript
// DevTools에서 Core Web Vitals 확인
// 1. F12 → Performance 탭
// 2. 체크박스 'Web Vitals' 활성화
// 3. 녹화 시작 후 페이지 새로고침
// 4. 타임라인에서 LCP, CLS 마커 확인
```

#### Lighthouse

```bash
# CLI에서 Lighthouse 실행
npx lighthouse https://example.com \
  --output=json \
  --output-path=./lighthouse-report.json \
  --chrome-flags="--headless"

# 특정 카테고리만 테스트
npx lighthouse https://example.com \
  --only-categories=performance \
  --output=html \
  --view
```

#### Web Vitals 라이브러리

```typescript
import { onLCP, onINP, onCLS } from 'web-vitals';

// Core Web Vitals 측정
function sendToAnalytics(metric: { name: string; value: number; id: string }) {
  // 분석 서비스로 전송
  console.log(`${metric.name}: ${metric.value}`);

  // Google Analytics 4로 전송 예시
  gtag('event', metric.name, {
    event_category: 'Web Vitals',
    value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
    event_label: metric.id,
    non_interaction: true,
  });
}

// 각 메트릭 측정 시작
onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);

// Next.js에서 사용 예시 (_app.tsx)
export function reportWebVitals(metric: NextWebVitalsMetric) {
  switch (metric.name) {
    case 'LCP':
    case 'INP':
    case 'CLS':
      sendToAnalytics(metric);
      break;
  }
}
```

#### PageSpeed Insights API

```typescript
// PageSpeed Insights API 호출
async function getPageSpeedData(url: string): Promise<void> {
  const API_KEY = 'YOUR_API_KEY';
  const apiUrl = `https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=${encodeURIComponent(url)}&key=${API_KEY}&strategy=mobile`;

  const response = await fetch(apiUrl);
  const data = await response.json();

  // Core Web Vitals 추출
  const metrics = data.lighthouseResult.audits;

  console.log({
    LCP: metrics['largest-contentful-paint'].displayValue,
    CLS: metrics['cumulative-layout-shift'].displayValue,
    // INP는 필드 데이터에서 확인
    fieldData: data.loadingExperience?.metrics
  });
}
```

---

## 번들 사이즈 최적화

현대 프론트엔드 애플리케이션에서 번들 사이즈는 초기 로딩 성능에 가장 큰 영향을 미치는 요소 중 하나입니다.

### Vite 최적화

Vite는 Rollup 기반의 번들러로, 기본적으로 우수한 최적화를 제공하지만 추가 설정으로 더욱 개선할 수 있습니다.

#### 기본 빌드 최적화 설정

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],

  build: {
    // 타겟 브라우저 설정 (최신 브라우저 대상)
    target: 'es2020',

    // 최소화 설정
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,      // console.log 제거
        drop_debugger: true,     // debugger 제거
        pure_funcs: ['console.info', 'console.debug'],
      },
    },

    // 소스맵 설정 (프로덕션에서는 'hidden' 권장)
    sourcemap: 'hidden',

    // 청크 사이즈 경고 임계값 (KB)
    chunkSizeWarningLimit: 500,

    // CSS 코드 스플리팅
    cssCodeSplit: true,

    // Rollup 옵션
    rollupOptions: {
      output: {
        // 청크 파일명 패턴
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',

        // 수동 청크 분리
        manualChunks: {
          // 벤더 청크 분리
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router-dom'],
          'vendor-state': ['zustand', '@tanstack/react-query'],
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        },
      },
    },
  },
});
```

#### 고급 manualChunks 전략

```typescript
// vite.config.ts - 동적 청크 분리 전략
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // node_modules 기반 청크 분리
          if (id.includes('node_modules')) {
            // React 관련
            if (id.includes('react') || id.includes('react-dom') || id.includes('scheduler')) {
              return 'vendor-react';
            }

            // 라우터
            if (id.includes('react-router') || id.includes('@remix-run')) {
              return 'vendor-router';
            }

            // 상태 관리
            if (id.includes('zustand') || id.includes('immer') || id.includes('@tanstack')) {
              return 'vendor-state';
            }

            // UI 라이브러리
            if (id.includes('@radix-ui') || id.includes('@headlessui')) {
              return 'vendor-ui';
            }

            // 유틸리티
            if (id.includes('lodash') || id.includes('date-fns') || id.includes('clsx')) {
              return 'vendor-utils';
            }

            // 기타 벤더
            return 'vendor';
          }

          // 페이지별 청크 분리
          if (id.includes('/pages/')) {
            const match = id.match(/\/pages\/([^/]+)/);
            if (match) {
              return `page-${match[1]}`;
            }
          }
        },
      },
    },
  },
});
```

#### Dynamic Import 활용

```typescript
// 라우트 기반 코드 스플리팅
// router.tsx
import { lazy, Suspense } from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import LoadingSpinner from './components/LoadingSpinner';

// 페이지 컴포넌트 지연 로딩
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));

// 번들 분석을 위한 청크 이름 지정
const Analytics = lazy(() =>
  import(/* webpackChunkName: "analytics" */ './pages/Analytics')
);

const router = createBrowserRouter([
  {
    path: '/',
    element: (
      <Suspense fallback={<LoadingSpinner />}>
        <Home />
      </Suspense>
    ),
  },
  {
    path: '/dashboard',
    element: (
      <Suspense fallback={<LoadingSpinner />}>
        <Dashboard />
      </Suspense>
    ),
  },
  // ... 기타 라우트
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

```typescript
// 컴포넌트 레벨 코드 스플리팅
// 무거운 컴포넌트 지연 로딩
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./components/HeavyChart'));
const MarkdownEditor = lazy(() => import('./components/MarkdownEditor'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showEditor, setShowEditor] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>
        차트 보기
      </button>

      {showChart && (
        <Suspense fallback={<div>차트 로딩 중...</div>}>
          <HeavyChart />
        </Suspense>
      )}

      <button onClick={() => setShowEditor(true)}>
        에디터 열기
      </button>

      {showEditor && (
        <Suspense fallback={<div>에디터 로딩 중...</div>}>
          <MarkdownEditor />
        </Suspense>
      )}
    </div>
  );
}
```

### Webpack 최적화

Webpack은 더 세밀한 최적화 제어가 가능한 번들러입니다.

#### SplitChunksPlugin 설정

```javascript
// webpack.config.js
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  mode: 'production',

  entry: {
    main: './src/index.tsx',
  },

  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    clean: true,
  },

  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true,
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true,
            pure_funcs: ['console.log', 'console.info'],
            passes: 2, // 압축 패스 횟수
          },
          mangle: {
            safari10: true, // Safari 10 호환성
          },
          output: {
            comments: false, // 주석 제거
          },
        },
        extractComments: false,
      }),
      new CssMinimizerPlugin({
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true },
            },
          ],
        },
      }),
    ],

    splitChunks: {
      chunks: 'all',
      minSize: 20000,        // 최소 청크 크기 (20KB)
      minRemainingSize: 0,
      minChunks: 1,          // 최소 공유 청크 수
      maxAsyncRequests: 30,  // 비동기 청크 최대 요청 수
      maxInitialRequests: 30, // 초기 청크 최대 요청 수
      enforceSizeThreshold: 50000, // 강제 분할 임계값 (50KB)

      cacheGroups: {
        // React 코어
        reactVendor: {
          test: /[\\/]node_modules[\\/](react|react-dom|scheduler)[\\/]/,
          name: 'vendor-react',
          chunks: 'all',
          priority: 40,
        },

        // 라우팅
        routerVendor: {
          test: /[\\/]node_modules[\\/](react-router|react-router-dom|@remix-run)[\\/]/,
          name: 'vendor-router',
          chunks: 'all',
          priority: 30,
        },

        // 상태 관리
        stateVendor: {
          test: /[\\/]node_modules[\\/](zustand|@tanstack|immer)[\\/]/,
          name: 'vendor-state',
          chunks: 'all',
          priority: 30,
        },

        // 기타 벤더
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10,
        },

        // 공통 모듈
        common: {
          minChunks: 2,
          name: 'common',
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },

    // 런타임 청크 분리
    runtimeChunk: 'single',

    // 모듈 ID 해시 (캐시 최적화)
    moduleIds: 'deterministic',
  },

  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
      chunkFilename: '[id].[contenthash].css',
    }),

    // 번들 분석 (개발 시에만)
    process.env.ANALYZE && new BundleAnalyzerPlugin(),
  ].filter(Boolean),
};
```

### 공통 최적화 기법

#### Tree Shaking 극대화

```typescript
// package.json - sideEffects 설정
{
  "name": "my-app",
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.ts"
  ]
}

// Bad: 전체 라이브러리 임포트
import _ from 'lodash';
const result = _.debounce(fn, 300);

// Good: 필요한 함수만 임포트
import debounce from 'lodash/debounce';
const result = debounce(fn, 300);

// Better: lodash-es 사용 (ES 모듈)
import { debounce } from 'lodash-es';
const result = debounce(fn, 300);
```

#### 무거운 라이브러리 대체

```typescript
// moment.js (326KB) → date-fns (13KB per function)
// Before
import moment from 'moment';
const formatted = moment().format('YYYY-MM-DD');

// After
import { format } from 'date-fns';
const formatted = format(new Date(), 'yyyy-MM-dd');

// lodash (71KB) → native 또는 개별 함수
// Before
import _ from 'lodash';
const unique = _.uniq(array);

// After - native
const unique = [...new Set(array)];

// axios (13KB) → fetch API 또는 ky (3.5KB)
// Before
import axios from 'axios';
const response = await axios.get('/api/data');

// After - native fetch
const response = await fetch('/api/data').then(r => r.json());

// After - ky (간단한 래퍼)
import ky from 'ky';
const data = await ky.get('/api/data').json();
```

#### 번들 분석

```bash
# Vite - rollup-plugin-visualizer
npm install -D rollup-plugin-visualizer

# vite.config.ts에 추가
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      open: true,
      filename: 'stats.html',
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

```bash
# Webpack - bundle analyzer
npm install -D webpack-bundle-analyzer

# 분석 실행
ANALYZE=true npm run build

# 또는 CLI로 직접 실행
npx webpack-bundle-analyzer dist/stats.json
```

---

## 이미지 최적화

이미지는 평균적으로 웹 페이지 전체 용량의 50% 이상을 차지합니다. 이미지 최적화는 성능 개선에 가장 큰 효과를 볼 수 있는 영역입니다.

### Next.js Image 최적화

Next.js의 `next/image` 컴포넌트는 자동 최적화 기능을 제공합니다.

#### 기본 사용법

```tsx
import Image from 'next/image';

// 정적 이미지 (자동 크기 추론)
import heroImage from '../public/hero.jpg';

export default function Hero() {
  return (
    <div className="relative h-screen">
      <Image
        src={heroImage}
        alt="Hero background"
        fill                      // 부모 요소 채우기
        priority                  // LCP 이미지에 사용
        placeholder="blur"        // 로딩 중 blur 효과
        quality={85}              // 품질 (1-100)
        sizes="100vw"             // 반응형 크기 힌트
        style={{ objectFit: 'cover' }}
      />
    </div>
  );
}

// 외부 이미지
export function ProductCard({ product }) {
  return (
    <div className="relative w-full aspect-square">
      <Image
        src={product.imageUrl}
        alt={product.name}
        fill
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
        loading="lazy"           // 뷰포트 밖 이미지는 지연 로딩
        placeholder="blur"
        blurDataURL={product.blurHash} // 커스텀 blur 데이터
      />
    </div>
  );
}
```

#### next.config.js 이미지 설정

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    // 외부 도메인 허용
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
        pathname: '/uploads/**',
      },
      {
        protocol: 'https',
        hostname: '**.cloudinary.com',
      },
    ],

    // 이미지 포맷 (자동 WebP/AVIF 변환)
    formats: ['image/avif', 'image/webp'],

    // 디바이스 크기 설정
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],

    // 이미지 크기 설정
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],

    // 최소화 활성화
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1년

    // 동시 이미지 최적화 제한
    dangerouslyAllowSVG: false,
  },
};

module.exports = nextConfig;
```

#### 반응형 이미지 패턴

```tsx
// 반응형 갤러리 컴포넌트
import Image from 'next/image';

interface GalleryImage {
  src: string;
  alt: string;
  width: number;
  height: number;
}

export function ResponsiveGallery({ images }: { images: GalleryImage[] }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {images.map((image, index) => (
        <div
          key={index}
          className="relative aspect-[4/3] overflow-hidden rounded-lg"
        >
          <Image
            src={image.src}
            alt={image.alt}
            fill
            sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
            className="object-cover transition-transform hover:scale-105"
            loading={index < 6 ? 'eager' : 'lazy'} // 처음 6개만 eager
            priority={index === 0} // 첫 번째 이미지만 priority
          />
        </div>
      ))}
    </div>
  );
}
```

### 범용 이미지 최적화

Next.js 없이도 적용할 수 있는 이미지 최적화 기법입니다.

#### 현대적 이미지 포맷 활용

```html
<!-- picture 요소로 포맷 대체 -->
<picture>
  <!-- AVIF (최신 브라우저) -->
  <source
    type="image/avif"
    srcset="
      /images/hero-400.avif 400w,
      /images/hero-800.avif 800w,
      /images/hero-1200.avif 1200w
    "
    sizes="(max-width: 768px) 100vw, 50vw"
  >

  <!-- WebP (대부분 브라우저) -->
  <source
    type="image/webp"
    srcset="
      /images/hero-400.webp 400w,
      /images/hero-800.webp 800w,
      /images/hero-1200.webp 1200w
    "
    sizes="(max-width: 768px) 100vw, 50vw"
  >

  <!-- JPEG (폴백) -->
  <img
    src="/images/hero-800.jpg"
    srcset="
      /images/hero-400.jpg 400w,
      /images/hero-800.jpg 800w,
      /images/hero-1200.jpg 1200w
    "
    sizes="(max-width: 768px) 100vw, 50vw"
    alt="Hero image"
    loading="lazy"
    decoding="async"
    width="800"
    height="600"
  >
</picture>
```

#### Native Lazy Loading

```html
<!-- HTML native lazy loading -->
<img
  src="image.jpg"
  alt="Description"
  loading="lazy"           <!-- lazy, eager -->
  decoding="async"         <!-- async, sync, auto -->
  fetchpriority="low"      <!-- high, low, auto -->
  width="800"
  height="600"
>
```

#### Intersection Observer 기반 Lazy Loading

```typescript
// React 컴포넌트로 구현
import { useRef, useState, useEffect } from 'react';

interface LazyImageProps {
  src: string;
  alt: string;
  placeholder?: string;
  className?: string;
}

export function LazyImage({ src, alt, placeholder, className }: LazyImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);

  useEffect(() => {
    if (!imgRef.current) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      {
        rootMargin: '50px 0px', // 50px 전에 미리 로드
        threshold: 0.01,
      }
    );

    observer.observe(imgRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    <div className={`relative overflow-hidden ${className}`}>
      {/* Placeholder */}
      {!isLoaded && placeholder && (
        <img
          src={placeholder}
          alt=""
          className="absolute inset-0 w-full h-full object-cover blur-lg scale-110"
          aria-hidden="true"
        />
      )}

      {/* 실제 이미지 */}
      <img
        ref={imgRef}
        src={isInView ? src : undefined}
        data-src={src}
        alt={alt}
        className={`w-full h-full object-cover transition-opacity duration-300 ${
          isLoaded ? 'opacity-100' : 'opacity-0'
        }`}
        onLoad={() => setIsLoaded(true)}
      />
    </div>
  );
}
```

#### 이미지 CDN 활용

```typescript
// Cloudinary URL 생성 유틸리티
interface CloudinaryOptions {
  width?: number;
  height?: number;
  quality?: number;
  format?: 'auto' | 'webp' | 'avif' | 'jpg' | 'png';
  crop?: 'fill' | 'fit' | 'scale' | 'thumb';
}

function getCloudinaryUrl(
  publicId: string,
  options: CloudinaryOptions = {}
): string {
  const {
    width,
    height,
    quality = 'auto',
    format = 'auto',
    crop = 'fill',
  } = options;

  const transforms: string[] = [];

  if (width) transforms.push(`w_${width}`);
  if (height) transforms.push(`h_${height}`);
  transforms.push(`q_${quality}`);
  transforms.push(`f_${format}`);
  transforms.push(`c_${crop}`);

  const transformString = transforms.join(',');

  return `https://res.cloudinary.com/YOUR_CLOUD_NAME/image/upload/${transformString}/${publicId}`;
}

// 사용 예시
const imageUrl = getCloudinaryUrl('hero-image', {
  width: 800,
  height: 600,
  quality: 80,
  format: 'auto',
});

// 반응형 srcSet 생성
function getResponsiveSrcSet(publicId: string, widths: number[]): string {
  return widths
    .map(w => `${getCloudinaryUrl(publicId, { width: w })} ${w}w`)
    .join(', ');
}

const srcSet = getResponsiveSrcSet('hero-image', [400, 800, 1200, 1600]);
```

#### SVG 최적화

```bash
# SVGO로 SVG 최적화
npm install -g svgo

# 단일 파일 최적화
svgo input.svg -o output.svg

# 폴더 전체 최적화
svgo -f ./src/icons -o ./dist/icons

# 설정 파일 사용
svgo --config svgo.config.js input.svg
```

```javascript
// svgo.config.js
module.exports = {
  multipass: true,
  plugins: [
    'preset-default',
    'removeDoctype',
    'removeXMLProcInst',
    'removeComments',
    'removeMetadata',
    'removeEditorsNSData',
    'cleanupAttrs',
    'mergeStyles',
    'inlineStyles',
    'minifyStyles',
    'removeUselessDefs',
    'cleanupNumericValues',
    'convertColors',
    'removeUnknownsAndDefaults',
    'removeNonInheritableGroupAttrs',
    'removeUselessStrokeAndFill',
    'removeViewBox', // viewBox 유지하려면 제거
    'cleanupEnableBackground',
    'removeHiddenElems',
    'removeEmptyText',
    'convertShapeToPath',
    'convertEllipseToCircle',
    'moveElemsAttrsToGroup',
    'moveGroupAttrsToElems',
    'collapseGroups',
    'convertPathData',
    'convertTransform',
    'removeEmptyAttrs',
    'removeEmptyContainers',
    'mergePaths',
    'removeUnusedNS',
    'sortDefsChildren',
    'removeTitle',
    'removeDesc',
  ],
};
```

---

## JavaScript 최적화

JavaScript는 파싱, 컴파일, 실행 과정에서 메인 스레드를 차지하여 렌더링을 블로킹할 수 있습니다.

### 코드 스플리팅과 지연 로딩

```typescript
// Route 기반 코드 스플리팅
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// 각 페이지를 별도 청크로 분리
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Checkout = lazy(() => import('./pages/Checkout'));
const Admin = lazy(() => import('./pages/Admin'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="/checkout" element={<Checkout />} />
        <Route path="/admin/*" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}
```

### Third-party 스크립트 최적화

```typescript
// Third-party 스크립트 지연 로딩
// utils/loadScript.ts
export function loadScript(src: string, id: string): Promise<void> {
  return new Promise((resolve, reject) => {
    // 이미 로드된 경우 건너뛰기
    if (document.getElementById(id)) {
      resolve();
      return;
    }

    const script = document.createElement('script');
    script.src = src;
    script.id = id;
    script.async = true;

    script.onload = () => resolve();
    script.onerror = () => reject(new Error(`Failed to load ${src}`));

    document.body.appendChild(script);
  });
}

// 사용자 상호작용 후 로드
export function loadOnInteraction(
  src: string,
  id: string,
  events = ['click', 'scroll', 'mousemove', 'touchstart']
): void {
  let loaded = false;

  const load = () => {
    if (loaded) return;
    loaded = true;

    events.forEach(event => {
      window.removeEventListener(event, load);
    });

    loadScript(src, id);
  };

  events.forEach(event => {
    window.addEventListener(event, load, { once: true, passive: true });
  });
}

// 페이지 로드 완료 후 지연 로드
export function loadOnIdle(src: string, id: string): void {
  if ('requestIdleCallback' in window) {
    requestIdleCallback(() => loadScript(src, id));
  } else {
    setTimeout(() => loadScript(src, id), 200);
  }
}
```

```tsx
// Next.js Script 컴포넌트 활용
import Script from 'next/script';

export default function Layout({ children }) {
  return (
    <>
      {children}

      {/* 중요한 스크립트 - 하이드레이션 전 */}
      <Script
        src="https://example.com/critical.js"
        strategy="beforeInteractive"
      />

      {/* 일반 스크립트 - 하이드레이션 후 */}
      <Script
        src="https://www.google-analytics.com/analytics.js"
        strategy="afterInteractive"
      />

      {/* 지연 로드 - 브라우저 유휴 시 */}
      <Script
        src="https://example.com/chat-widget.js"
        strategy="lazyOnload"
        onLoad={() => console.log('Chat widget loaded')}
      />
    </>
  );
}
```

### Web Workers 활용

```typescript
// heavy-computation.worker.ts
self.onmessage = (e: MessageEvent<{ data: number[] }>) => {
  const { data } = e.data;

  // 무거운 연산 수행
  const result = data
    .map(n => {
      // 복잡한 계산 시뮬레이션
      let sum = 0;
      for (let i = 0; i < 1000000; i++) {
        sum += Math.sqrt(n * i);
      }
      return sum;
    });

  self.postMessage({ result });
};

// hooks/useWorker.ts
import { useCallback, useRef, useEffect } from 'react';

export function useWorker<T, R>(workerFactory: () => Worker) {
  const workerRef = useRef<Worker | null>(null);

  useEffect(() => {
    workerRef.current = workerFactory();

    return () => {
      workerRef.current?.terminate();
    };
  }, [workerFactory]);

  const runWorker = useCallback((data: T): Promise<R> => {
    return new Promise((resolve, reject) => {
      if (!workerRef.current) {
        reject(new Error('Worker not initialized'));
        return;
      }

      workerRef.current.onmessage = (e: MessageEvent<R>) => {
        resolve(e.data);
      };

      workerRef.current.onerror = (error) => {
        reject(error);
      };

      workerRef.current.postMessage(data);
    });
  }, []);

  return runWorker;
}

// 컴포넌트에서 사용
function DataProcessor() {
  const [result, setResult] = useState<number[] | null>(null);
  const [isProcessing, setIsProcessing] = useState(false);

  const runComputation = useWorker<{ data: number[] }, { result: number[] }>(
    () => new Worker(new URL('./heavy-computation.worker.ts', import.meta.url))
  );

  const handleProcess = async () => {
    setIsProcessing(true);
    try {
      const { result } = await runComputation({ data: [1, 2, 3, 4, 5] });
      setResult(result);
    } finally {
      setIsProcessing(false);
    }
  };

  return (
    <div>
      <button onClick={handleProcess} disabled={isProcessing}>
        {isProcessing ? '처리 중...' : '데이터 처리'}
      </button>
      {result && <div>결과: {result.join(', ')}</div>}
    </div>
  );
}
```

### requestIdleCallback과 requestAnimationFrame

```typescript
// 작업 스케줄링 유틸리티
class TaskScheduler {
  private taskQueue: (() => void)[] = [];
  private isProcessing = false;

  // 유휴 시간에 작업 실행
  scheduleIdleTask(task: () => void): void {
    this.taskQueue.push(task);
    this.processQueue();
  }

  private processQueue(): void {
    if (this.isProcessing || this.taskQueue.length === 0) return;

    this.isProcessing = true;

    const processNextTask = (deadline: IdleDeadline) => {
      // 남은 시간이 있고 작업이 남아있는 동안
      while (deadline.timeRemaining() > 0 && this.taskQueue.length > 0) {
        const task = this.taskQueue.shift();
        task?.();
      }

      if (this.taskQueue.length > 0) {
        requestIdleCallback(processNextTask);
      } else {
        this.isProcessing = false;
      }
    };

    if ('requestIdleCallback' in window) {
      requestIdleCallback(processNextTask);
    } else {
      // 폴백: setTimeout
      setTimeout(() => {
        const task = this.taskQueue.shift();
        task?.();
        this.isProcessing = false;
        if (this.taskQueue.length > 0) this.processQueue();
      }, 0);
    }
  }

  // 다음 프레임에 작업 실행 (애니메이션/UI 업데이트용)
  scheduleAnimationTask(task: () => void): number {
    return requestAnimationFrame(task);
  }

  // 우선순위 높은 작업 (사용자 인터랙션 응답)
  scheduleUrgentTask(task: () => void): void {
    queueMicrotask(task);
  }
}

const scheduler = new TaskScheduler();

// 사용 예시
function handleUserAction() {
  // 즉시 UI 피드백
  scheduler.scheduleUrgentTask(() => {
    updateButtonState();
  });

  // 애니메이션 업데이트
  scheduler.scheduleAnimationTask(() => {
    animateProgress();
  });

  // 중요하지 않은 분석 작업
  scheduler.scheduleIdleTask(() => {
    sendAnalytics();
  });
}
```

---

## CSS 최적화

CSS는 렌더링 차단 리소스입니다. 최적화를 통해 First Paint 시간을 크게 단축할 수 있습니다.

### Critical CSS 추출

```typescript
// Vite에서 Critical CSS 추출
// vite.config.ts
import { defineConfig } from 'vite';
import critical from 'vite-plugin-critical';

export default defineConfig({
  plugins: [
    critical({
      criticalUrl: './',
      criticalBase: './dist/',
      criticalPages: [
        { uri: '', template: 'index' },
        { uri: 'about', template: 'about' },
      ],
      criticalConfig: {
        inline: true,
        dimensions: [
          { width: 375, height: 667 },  // Mobile
          { width: 1440, height: 900 }, // Desktop
        ],
      },
    }),
  ],
});
```

```html
<!-- Critical CSS 인라인 패턴 -->
<head>
  <!-- 중요 CSS 인라인 -->
  <style>
    /* Above-the-fold 콘텐츠에 필요한 스타일만 */
    .header { /* ... */ }
    .hero { /* ... */ }
    .nav { /* ... */ }
  </style>

  <!-- 나머지 CSS 지연 로드 -->
  <link
    rel="preload"
    href="/styles/main.css"
    as="style"
    onload="this.onload=null;this.rel='stylesheet'"
  >
  <noscript>
    <link rel="stylesheet" href="/styles/main.css">
  </noscript>
</head>
```

### CSS-in-JS 성능 고려사항

```typescript
// Emotion - 런타임 vs 컴파일 타임
// Bad: 런타임 스타일 계산
import styled from '@emotion/styled';

const DynamicButton = styled.button`
  background: ${props => props.variant === 'primary' ? 'blue' : 'gray'};
  padding: ${props => props.size === 'large' ? '16px' : '8px'};
`;

// Good: 정적 스타일 + CSS 변수
const StaticButton = styled.button`
  background: var(--button-bg);
  padding: var(--button-padding);
`;

// Better: 컴파일 타임 CSS-in-JS 사용 (vanilla-extract, Linaria)
// styles.css.ts (vanilla-extract)
import { style, styleVariants } from '@vanilla-extract/css';

export const button = style({
  borderRadius: '4px',
  cursor: 'pointer',
});

export const buttonVariant = styleVariants({
  primary: [button, { background: 'blue', color: 'white' }],
  secondary: [button, { background: 'gray', color: 'black' }],
});
```

### Unused CSS 제거

```javascript
// postcss.config.js with PurgeCSS
module.exports = {
  plugins: [
    require('autoprefixer'),
    process.env.NODE_ENV === 'production' && require('@fullhuman/postcss-purgecss')({
      content: [
        './src/**/*.{js,jsx,ts,tsx}',
        './public/index.html',
      ],
      defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
      safelist: {
        standard: [/^modal/, /^tooltip/, /^animate/], // 동적 클래스 보존
        deep: [/^data-/],
        greedy: [/^react-/],
      },
    }),
  ].filter(Boolean),
};
```

### CSS Containment

```css
/* contain 속성으로 렌더링 최적화 */
.card {
  /* 레이아웃, 스타일, 페인트가 요소 내부로 제한됨 */
  contain: layout style paint;
}

.sidebar {
  /* 요소의 크기가 자식에 의존하지 않음 */
  contain: size layout style paint;
}

/* content-visibility로 오프스크린 콘텐츠 렌더링 건너뛰기 */
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 200px; /* 예상 크기 힌트 */
}

/* 대규모 목록 최적화 */
.virtual-list-container {
  contain: strict; /* 모든 containment 적용 */
}

.virtual-list-item {
  content-visibility: auto;
  contain-intrinsic-block-size: 80px;
}
```

### will-change 올바른 사용

```css
/* Bad: 항상 will-change 적용 */
.card {
  will-change: transform; /* 메모리 낭비 */
}

/* Good: 호버 시에만 will-change 적용 */
.card {
  transition: transform 0.3s ease;
}

.card:hover {
  will-change: transform;
}

/* Better: JavaScript로 동적 적용 */
```

```typescript
// will-change를 동적으로 관리
function setupWillChange(element: HTMLElement) {
  element.addEventListener('mouseenter', () => {
    element.style.willChange = 'transform';
  });

  element.addEventListener('mouseleave', () => {
    // 트랜지션 완료 후 제거
    element.addEventListener('transitionend', () => {
      element.style.willChange = 'auto';
    }, { once: true });
  });
}
```

---

## 네트워크 최적화

네트워크 지연은 웹 성능의 주요 병목입니다. HTTP 프로토콜 활용과 캐싱 전략으로 개선할 수 있습니다.

### HTTP/2, HTTP/3 활용

```nginx
# Nginx HTTP/2 설정
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    # HTTP/2 Server Push (선택적)
    location / {
        http2_push /styles/critical.css;
        http2_push /scripts/main.js;
        try_files $uri $uri/ /index.html;
    }
}
```

### 리소스 힌트

```html
<head>
  <!-- DNS 미리 조회 -->
  <link rel="dns-prefetch" href="//api.example.com">
  <link rel="dns-prefetch" href="//cdn.example.com">

  <!-- 연결 미리 설정 (DNS + TCP + TLS) -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

  <!-- 리소스 미리 로드 (현재 페이지) -->
  <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/images/hero.webp" as="image" fetchpriority="high">
  <link rel="preload" href="/styles/critical.css" as="style">

  <!-- 리소스 미리 가져오기 (다음 페이지) -->
  <link rel="prefetch" href="/pages/about.html">
  <link rel="prefetch" href="/scripts/dashboard.js">

  <!-- 페이지 미리 렌더링 -->
  <link rel="prerender" href="/checkout">
</head>
```

```typescript
// 동적 리소스 힌트 추가
// 사용자 행동 기반 Prefetch
function prefetchOnHover(linkElement: HTMLAnchorElement) {
  let prefetchLink: HTMLLinkElement | null = null;

  linkElement.addEventListener('mouseenter', () => {
    const href = linkElement.getAttribute('href');
    if (!href) return;

    prefetchLink = document.createElement('link');
    prefetchLink.rel = 'prefetch';
    prefetchLink.href = href;
    document.head.appendChild(prefetchLink);
  });

  linkElement.addEventListener('mouseleave', () => {
    if (prefetchLink) {
      document.head.removeChild(prefetchLink);
      prefetchLink = null;
    }
  });
}

// 뷰포트 내 링크 자동 Prefetch
function setupViewportPrefetch() {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const link = entry.target as HTMLAnchorElement;
        const href = link.getAttribute('href');

        if (href && !href.startsWith('#')) {
          const prefetch = document.createElement('link');
          prefetch.rel = 'prefetch';
          prefetch.href = href;
          document.head.appendChild(prefetch);

          observer.unobserve(link);
        }
      }
    });
  });

  document.querySelectorAll('a[href]').forEach(link => {
    observer.observe(link);
  });
}
```

### Service Worker와 캐싱 전략

```typescript
// service-worker.ts
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import {
  CacheFirst,
  NetworkFirst,
  StaleWhileRevalidate
} from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';
import { CacheableResponsePlugin } from 'workbox-cacheable-response';

// 빌드 시 생성된 자산 프리캐시
precacheAndRoute(self.__WB_MANIFEST);

// 이미지: Cache First (장기 캐시)
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images-cache',
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30일
      }),
    ],
  })
);

// API 요청: Network First (항상 최신 데이터)
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    networkTimeoutSeconds: 10,
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 5 * 60, // 5분
      }),
    ],
  })
);

// 정적 자산: Stale While Revalidate (빠른 응답 + 업데이트)
registerRoute(
  ({ request }) =>
    request.destination === 'script' ||
    request.destination === 'style',
  new StaleWhileRevalidate({
    cacheName: 'static-cache',
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
    ],
  })
);

// 오프라인 폴백
const offlineFallbackPage = '/offline.html';

self.addEventListener('install', async (event) => {
  event.waitUntil(
    caches.open('offline-cache').then(cache => {
      return cache.add(offlineFallbackPage);
    })
  );
});

self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request).catch(() => {
        return caches.match(offlineFallbackPage);
      })
    );
  }
});
```

### Compression 설정

```nginx
# Nginx Gzip/Brotli 설정
http {
    # Gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/json
        application/xml
        application/rss+xml
        image/svg+xml;

    # Brotli (ngx_brotli 모듈 필요)
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/json
        application/xml
        image/svg+xml;
}
```

```typescript
// Express.js 압축 미들웨어
import compression from 'compression';
import express from 'express';

const app = express();

app.use(compression({
  level: 6,
  threshold: 1024, // 1KB 이상만 압축
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
}));
```

---

## 렌더링 최적화

프론트엔드 렌더링 방식 선택과 React 컴포넌트 최적화는 성능에 큰 영향을 미칩니다.

### SSR vs SSG vs CSR vs ISR 선택 기준

```
┌─────────────────────────────────────────────────────────────────┐
│                     렌더링 전략 선택 가이드                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SSG (Static Site Generation)                                   │
│  └─ 콘텐츠가 거의 변하지 않는 경우                                │
│  └─ 블로그, 문서, 마케팅 페이지                                   │
│  └─ 최고의 성능 (CDN에서 직접 제공)                               │
│                                                                 │
│  ISR (Incremental Static Regeneration)                          │
│  └─ 정적이지만 주기적 업데이트 필요                               │
│  └─ 제품 카탈로그, 뉴스 사이트                                    │
│  └─ SSG 성능 + 데이터 최신성                                     │
│                                                                 │
│  SSR (Server-Side Rendering)                                    │
│  └─ 개인화된 콘텐츠, 실시간 데이터                                │
│  └─ 대시보드, 사용자별 피드                                       │
│  └─ SEO 필요 + 동적 데이터                                       │
│                                                                 │
│  CSR (Client-Side Rendering)                                    │
│  └─ 인증 필요한 앱, 높은 인터랙티비티                             │
│  └─ 관리자 패널, SaaS 앱                                         │
│  └─ SEO 불필요 + 빠른 네비게이션                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// Next.js 렌더링 전략 예시

// SSG - 빌드 시 생성
export async function generateStaticParams() {
  const posts = await getBlogPosts();
  return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return <Article post={post} />;
}

// ISR - 주기적 재생성
export const revalidate = 3600; // 1시간마다 재검증

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  return <ProductDetail product={product} />;
}

// SSR - 매 요청마다 렌더링
export const dynamic = 'force-dynamic';

export default async function DashboardPage() {
  const userData = await getCurrentUserData();
  return <Dashboard data={userData} />;
}
```

### React 렌더링 최적화

```typescript
// 1. React.memo로 불필요한 리렌더링 방지
interface UserCardProps {
  user: {
    id: string;
    name: string;
    avatar: string;
  };
  onClick: (id: string) => void;
}

// Bad: 부모 리렌더링 시 항상 리렌더링
function UserCard({ user, onClick }: UserCardProps) {
  return (
    <div onClick={() => onClick(user.id)}>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  );
}

// Good: props가 변경될 때만 리렌더링
const MemoizedUserCard = memo(function UserCard({ user, onClick }: UserCardProps) {
  return (
    <div onClick={() => onClick(user.id)}>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
    </div>
  );
});

// 2. useMemo로 계산 결과 메모이제이션
function ExpensiveList({ items, filter }: { items: Item[]; filter: string }) {
  // filter나 items가 변경될 때만 재계산
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    ).sort((a, b) => b.score - a.score);
  }, [items, filter]);

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// 3. useCallback으로 함수 참조 안정화
function ParentComponent() {
  const [count, setCount] = useState(0);

  // count가 변경되어도 함수 참조 유지
  const handleClick = useCallback((id: string) => {
    console.log('Clicked:', id);
    // count를 사용하지 않으므로 의존성 없음
  }, []);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      {/* handleClick 참조가 안정적이므로 불필요한 리렌더링 방지 */}
      <MemoizedUserCard user={user} onClick={handleClick} />
    </div>
  );
}
```

### 리플로우/리페인트 최소화

```typescript
// Bad: 여러 번의 리플로우 발생
function badBatchDOMUpdates() {
  const element = document.getElementById('box')!;

  element.style.width = '100px';  // 리플로우
  console.log(element.offsetHeight); // 강제 동기 레이아웃
  element.style.height = '100px'; // 리플로우
  console.log(element.offsetWidth);  // 강제 동기 레이아웃
}

// Good: 읽기/쓰기 분리로 배치 처리
function goodBatchDOMUpdates() {
  const element = document.getElementById('box')!;

  // 읽기 먼저
  const height = element.offsetHeight;
  const width = element.offsetWidth;

  // 쓰기는 나중에 (자동 배치)
  element.style.width = '100px';
  element.style.height = '100px';
}

// Better: requestAnimationFrame으로 다음 프레임에 배치
function betterBatchDOMUpdates() {
  requestAnimationFrame(() => {
    const element = document.getElementById('box')!;
    element.style.width = '100px';
    element.style.height = '100px';
    element.style.transform = 'translateX(50px)';
  });
}

// Best: CSS 클래스 토글로 한 번에 변경
function bestBatchDOMUpdates() {
  const element = document.getElementById('box')!;
  element.classList.add('expanded'); // 한 번의 리플로우
}
```

```css
/* 애니메이션 성능 최적화 */
/* Bad: top/left 사용 (리플로우 발생) */
.bad-animation {
  position: absolute;
  animation: slide-bad 1s ease;
}

@keyframes slide-bad {
  from { left: 0; }
  to { left: 100px; }
}

/* Good: transform 사용 (GPU 가속, 리플로우 없음) */
.good-animation {
  animation: slide-good 1s ease;
  will-change: transform; /* 애니메이션 직전에만 */
}

@keyframes slide-good {
  from { transform: translateX(0); }
  to { transform: translateX(100px); }
}

/* 레이어 분리가 필요한 경우 */
.gpu-accelerated {
  transform: translateZ(0); /* 새 레이어 생성 */
  /* 또는 */
  will-change: transform;
}
```

---

## 실제 성능 개선 사례

가상의 이커머스 플랫폼 "ShopFast"의 성능 개선 프로젝트를 통해 각 최적화 기법의 실제 효과를 살펴봅니다.

### 프로젝트 개요

- **플랫폼**: Next.js 14 기반 이커머스
- **일일 활성 사용자**: 50,000명
- **페이지 수**: 메인, 카테고리, 상품 상세, 장바구니, 결제

### Before: 최적화 전 상태

```
┌────────────────────────────────────────────────────────────────┐
│ 최적화 전 Core Web Vitals (Mobile)                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ LCP: 4.8초      ████████████████████░░░░░░░░░░░░░░░░░░  Poor   │
│ INP: 380ms      ██████████████████░░░░░░░░░░░░░░░░░░░░  Poor   │
│ CLS: 0.32       ██████████████░░░░░░░░░░░░░░░░░░░░░░░░  Poor   │
│                                                                │
│ Bundle Size: 1.8MB (gzip: 520KB)                               │
│ First Load: 6.2초                                              │
│ Lighthouse Score: 38                                           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 최적화 과정

#### 1단계: 번들 분석 및 최적화

```typescript
// 문제 발견: moment.js와 lodash 전체 번들 포함
// Bundle Analyzer 결과
// - moment.js: 287KB (with locales)
// - lodash: 71KB
// - 사용하지 않는 MUI 컴포넌트: 150KB

// 해결 1: moment.js → date-fns
// Before
import moment from 'moment';
const formatted = moment(date).format('YYYY-MM-DD');

// After
import { format } from 'date-fns';
const formatted = format(date, 'yyyy-MM-dd');
// 절감: 287KB → 12KB

// 해결 2: lodash 개별 임포트
// Before
import _ from 'lodash';
const debounced = _.debounce(fn, 300);

// After
import debounce from 'lodash/debounce';
const debounced = debounce(fn, 300);
// 절감: 71KB → 3KB

// 해결 3: 청크 분리
// next.config.js
module.exports = {
  experimental: {
    optimizePackageImports: ['@mui/material', '@mui/icons-material'],
  },
};
```

**결과**: 번들 사이즈 1.8MB → 890KB (50% 감소)

#### 2단계: 이미지 최적화

```typescript
// 문제: 최적화되지 않은 이미지
// - 상품 이미지: 원본 JPEG 2-5MB
// - 배너: PNG 3MB
// - Lazy loading 미적용

// 해결: next/image 적용
// Before
<img src={product.image} alt={product.name} />

// After
<Image
  src={product.image}
  alt={product.name}
  width={400}
  height={400}
  sizes="(max-width: 768px) 100vw, 400px"
  placeholder="blur"
  blurDataURL={product.blurHash}
  loading={index < 4 ? 'eager' : 'lazy'}
/>
```

```javascript
// next.config.js 이미지 설정
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200],
    minimumCacheTTL: 60 * 60 * 24 * 30,
  },
};
```

**결과**: 이미지 전송량 85% 감소, LCP 3.2초 개선

#### 3단계: CLS 개선

```typescript
// 문제: 이미지와 광고 배너로 인한 레이아웃 이동
// CLS 발생 원인 분석:
// 1. 이미지 크기 미지정
// 2. 동적 광고 배너
// 3. 웹폰트 로딩 시 FOUT

// 해결 1: 이미지 aspect-ratio
<div className="relative aspect-[4/3]">
  <Image fill sizes="100vw" ... />
</div>

// 해결 2: 광고 스켈레톤
function AdBanner() {
  const [isLoaded, setIsLoaded] = useState(false);

  return (
    <div
      className="min-h-[250px] bg-gray-100"
      style={{ aspectRatio: '728/90' }}
    >
      {isLoaded ? <Ad /> : <AdSkeleton />}
    </div>
  );
}

// 해결 3: 폰트 최적화
// next.config.js
const nextConfig = {
  optimizeFonts: true,
};

// layout.tsx
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  preload: true,
});
```

**결과**: CLS 0.32 → 0.05 (84% 개선)

#### 4단계: INP 개선

```typescript
// 문제: 필터 변경 시 느린 응답 (380ms)
// 원인: 동기적 필터링 + 전체 리스트 리렌더링

// Before
function ProductList({ products, filters }) {
  // 매번 동기적 필터링
  const filtered = products.filter(p => matchesFilters(p, filters));

  return filtered.map(p => <ProductCard key={p.id} product={p} />);
}

// After
function ProductList({ products, filters }) {
  const [isPending, startTransition] = useTransition();
  const [displayProducts, setDisplayProducts] = useState(products);

  useEffect(() => {
    startTransition(() => {
      const filtered = products.filter(p => matchesFilters(p, filters));
      setDisplayProducts(filtered);
    });
  }, [products, filters]);

  return (
    <>
      {isPending && <LoadingOverlay />}
      <VirtualizedList
        items={displayProducts}
        height={600}
        itemHeight={200}
        renderItem={(product) => (
          <MemoizedProductCard key={product.id} product={product} />
        )}
      />
    </>
  );
}

// 가상화된 리스트 컴포넌트
function VirtualizedList({ items, height, itemHeight, renderItem }) {
  const [startIndex, setStartIndex] = useState(0);
  const visibleCount = Math.ceil(height / itemHeight) + 2;

  const handleScroll = useCallback((e: React.UIEvent) => {
    const scrollTop = e.currentTarget.scrollTop;
    setStartIndex(Math.floor(scrollTop / itemHeight));
  }, [itemHeight]);

  const visibleItems = items.slice(startIndex, startIndex + visibleCount);

  return (
    <div
      style={{ height, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems.map((item, index) => (
          <div
            key={item.id}
            style={{
              position: 'absolute',
              top: (startIndex + index) * itemHeight,
              height: itemHeight,
              width: '100%',
            }}
          >
            {renderItem(item)}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**결과**: INP 380ms → 120ms (68% 개선)

### After: 최적화 후 결과

```
┌────────────────────────────────────────────────────────────────┐
│ 최적화 후 Core Web Vitals (Mobile)                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ LCP: 1.6초      ██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Good   │
│ INP: 120ms      ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Good   │
│ CLS: 0.05       ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Good   │
│                                                                │
│ Bundle Size: 890KB (gzip: 245KB)                               │
│ First Load: 2.1초                                              │
│ Lighthouse Score: 92                                           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 비즈니스 성과

| 지표 | Before | After | 개선율 |
|------|--------|-------|--------|
| 이탈률 | 54% | 38% | 30% 감소 |
| 전환율 | 1.8% | 2.9% | 61% 증가 |
| 평균 세션 시간 | 2분 15초 | 4분 30초 | 100% 증가 |
| 모바일 매출 | - | - | 42% 증가 |

---

## 성능 모니터링 및 지속적 개선

성능 최적화는 일회성 작업이 아닙니다. 지속적인 모니터링과 개선이 필요합니다.

### RUM (Real User Monitoring) 도구

```typescript
// Web Vitals 수집 및 전송
import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals';

interface PerformanceMetric {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
  navigationType: string;
  url: string;
  timestamp: number;
}

function sendToAnalytics(metric: PerformanceMetric) {
  // Beacon API로 안정적 전송
  const body = JSON.stringify(metric);

  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics/performance', body);
  } else {
    fetch('/api/analytics/performance', {
      method: 'POST',
      body,
      keepalive: true,
    });
  }
}

function collectWebVitals() {
  const processMetric = (metric: any) => {
    sendToAnalytics({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      navigationType: metric.navigationType,
      url: window.location.href,
      timestamp: Date.now(),
    });
  };

  onLCP(processMetric);
  onINP(processMetric);
  onCLS(processMetric);
  onFCP(processMetric);
  onTTFB(processMetric);
}

// 앱 초기화 시 호출
if (typeof window !== 'undefined') {
  collectWebVitals();
}
```

```typescript
// 커스텀 성능 대시보드 데이터 구조
interface PerformanceDashboard {
  // 집계된 메트릭
  aggregates: {
    lcp: { p50: number; p75: number; p95: number };
    inp: { p50: number; p75: number; p95: number };
    cls: { p50: number; p75: number; p95: number };
  };

  // 페이지별 분석
  byPage: Record<string, {
    views: number;
    metrics: typeof aggregates;
  }>;

  // 디바이스별 분석
  byDevice: {
    mobile: typeof aggregates;
    tablet: typeof aggregates;
    desktop: typeof aggregates;
  };

  // 시간대별 추이
  timeline: Array<{
    timestamp: string;
    metrics: typeof aggregates;
  }>;
}
```

### 성능 예산 설정

```typescript
// performance-budget.config.ts
export const performanceBudget = {
  // 번들 크기 제한
  bundles: {
    'main.js': { maxSize: 200 * 1024 },      // 200KB
    'vendor.js': { maxSize: 300 * 1024 },    // 300KB
    'total': { maxSize: 500 * 1024 },        // 500KB (gzip)
  },

  // Core Web Vitals 목표
  metrics: {
    LCP: { target: 2500, warning: 3000 },    // ms
    INP: { target: 200, warning: 300 },      // ms
    CLS: { target: 0.1, warning: 0.15 },     // score
    FCP: { target: 1800, warning: 2500 },    // ms
    TTFB: { target: 800, warning: 1000 },    // ms
  },

  // 리소스 제한
  resources: {
    images: { maxSize: 200 * 1024 },         // 200KB per image
    fonts: { maxCount: 4, maxSize: 100 * 1024 },
    thirdParty: { maxCount: 5 },
  },
};
```

### CI/CD에 성능 테스트 통합

```yaml
# .github/workflows/performance.yml
name: Performance CI

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Start server
        run: npm run start &
        env:
          PORT: 3000

      - name: Wait for server
        run: npx wait-on http://localhost:3000

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: |
            http://localhost:3000
            http://localhost:3000/products
            http://localhost:3000/products/1
          budgetPath: ./lighthouse-budget.json
          uploadArtifacts: true
          temporaryPublicStorage: true

  bundle-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Check bundle size
        uses: preactjs/compressed-size-action@v2
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          pattern: ".next/static/**/*.js"
          exclude: "{**/*.map,**/node_modules/**}"
```

```json
// lighthouse-budget.json
[
  {
    "path": "/*",
    "timings": [
      { "metric": "interactive", "budget": 3500 },
      { "metric": "first-contentful-paint", "budget": 1800 },
      { "metric": "largest-contentful-paint", "budget": 2500 }
    ],
    "resourceSizes": [
      { "resourceType": "script", "budget": 300 },
      { "resourceType": "stylesheet", "budget": 100 },
      { "resourceType": "image", "budget": 500 },
      { "resourceType": "total", "budget": 1000 }
    ],
    "resourceCounts": [
      { "resourceType": "third-party", "budget": 10 }
    ]
  }
]
```

### 지속적인 모니터링 전략

```typescript
// 성능 알림 시스템
interface PerformanceAlert {
  metric: string;
  threshold: number;
  currentValue: number;
  severity: 'warning' | 'critical';
  page: string;
  timestamp: Date;
}

class PerformanceMonitor {
  private alerts: PerformanceAlert[] = [];

  checkThresholds(metrics: Record<string, number>, page: string): void {
    const thresholds = {
      LCP: { warning: 2500, critical: 4000 },
      INP: { warning: 200, critical: 500 },
      CLS: { warning: 0.1, critical: 0.25 },
    };

    for (const [metric, value] of Object.entries(metrics)) {
      const threshold = thresholds[metric as keyof typeof thresholds];
      if (!threshold) continue;

      if (value > threshold.critical) {
        this.createAlert(metric, threshold.critical, value, 'critical', page);
      } else if (value > threshold.warning) {
        this.createAlert(metric, threshold.warning, value, 'warning', page);
      }
    }
  }

  private createAlert(
    metric: string,
    threshold: number,
    currentValue: number,
    severity: 'warning' | 'critical',
    page: string
  ): void {
    const alert: PerformanceAlert = {
      metric,
      threshold,
      currentValue,
      severity,
      page,
      timestamp: new Date(),
    };

    this.alerts.push(alert);
    this.notifyTeam(alert);
  }

  private notifyTeam(alert: PerformanceAlert): void {
    // Slack, Email, PagerDuty 등으로 알림 전송
    console.log(`[${alert.severity.toUpperCase()}] ${alert.metric} on ${alert.page}: ${alert.currentValue} (threshold: ${alert.threshold})`);
  }
}
```

---

## FAQ

### Q1. Core Web Vitals 중 어떤 것을 가장 먼저 최적화해야 하나요?

**A**: 일반적으로 **LCP**를 먼저 최적화하는 것을 권장합니다.

LCP는 사용자가 가장 직접적으로 체감하는 지표이며, 개선 방법이 비교적 명확합니다. 이미지 최적화, 서버 응답 시간 개선, Critical CSS 적용 등으로 빠르게 개선할 수 있습니다.

다만, 현재 사이트 상태에 따라 달라질 수 있으므로 PageSpeed Insights에서 가장 낮은 점수를 받은 지표부터 시작하세요.

```typescript
// 우선순위 결정을 위한 점검 순서
const optimizationPriority = {
  1: 'LCP > 4초면 LCP 먼저',
  2: 'CLS > 0.25면 CLS 먼저 (UX 영향 큼)',
  3: 'INP > 500ms면 INP 먼저 (인터랙션 문제)',
  4: '모두 양호하면 가장 낮은 점수부터',
};
```

### Q2. React.lazy와 dynamic import의 차이점은 무엇인가요?

**A**: 본질적으로 같은 기능을 수행하지만, 사용 맥락이 다릅니다.

```typescript
// React.lazy - React 컴포넌트 전용
const LazyComponent = React.lazy(() => import('./Component'));

// 반드시 Suspense와 함께 사용
<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>

// Dynamic import - 범용 (모듈, 함수 등)
const loadModule = async () => {
  const module = await import('./utils');
  return module.someFunction();
};

// Next.js dynamic - React.lazy + SSR 지원
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('./Component'), {
  loading: () => <Loading />,
  ssr: false, // 클라이언트에서만 로드
});
```

### Q3. 성능 최적화가 SEO에 미치는 영향은 어느 정도인가요?

**A**: 2021년부터 Google은 Core Web Vitals를 검색 랭킹 요소로 공식 채택했습니다.

다만, 콘텐츠 품질, 백링크, 키워드 관련성 등 다른 요소들이 여전히 더 중요합니다. Core Web Vitals는 "동점일 때의 타이브레이커" 역할을 합니다.

```
SEO 랭킹 요소 중요도 (대략적 추정):
┌────────────────────────────────────────┐
│ 콘텐츠 품질/관련성     █████████  45%   │
│ 백링크                ███████    35%   │
│ 기술적 SEO            ████       15%   │
│ Core Web Vitals       ██         5%    │
└────────────────────────────────────────┘
```

### Q4. 번들 사이즈를 줄이면 성능이 무조건 좋아지나요?

**A**: 대부분의 경우 그렇지만, 항상 그런 것은 아닙니다.

```typescript
// 케이스 1: 효과적인 번들 크기 감소
// moment.js → date-fns = LCP, TTI 개선

// 케이스 2: 오히려 역효과
// 과도한 코드 스플리팅 → 청크 수 증가 → HTTP 요청 증가
// HTTP/1.1에서는 성능 저하 가능

// 최적의 청크 크기
const optimalChunkSize = {
  min: 20_000,  // 20KB 미만은 병합
  max: 250_000, // 250KB 이상은 분할
  initial: 3,   // 초기 청크 수 제한
};

// HTTP/2에서는 더 작은 청크도 괜찮음
// HTTP/1.1에서는 청크 수를 6개 이하로 유지
```

### Q5. Server Components를 사용하면 무조건 성능이 좋아지나요?

**A**: 적절한 상황에서 사용해야 합니다.

```typescript
// Good: Server Component가 효과적인 경우
// - 데이터베이스 직접 조회
// - 큰 종속성을 서버에서만 사용
// - 정적 콘텐츠

// app/products/page.tsx (Server Component)
async function ProductsPage() {
  const products = await db.products.findMany(); // 서버에서 직접 쿼리
  return <ProductList products={products} />;
}

// Bad: Client Component가 더 나은 경우
// - 자주 업데이트되는 인터랙티브 UI
// - 사용자 입력에 반응하는 컴포넌트
// - 브라우저 API 필요

// components/SearchInput.tsx
'use client';
function SearchInput() {
  const [query, setQuery] = useState('');
  // 실시간 입력 처리에는 Client Component가 적합
}
```

### Q6. Lighthouse 점수 100점을 목표로 해야 하나요?

**A**: 100점이 목표가 아니라, 실제 사용자 경험 개선이 목표여야 합니다.

Lighthouse는 **실험실 데이터(Lab Data)**를 기반으로 합니다. 실제 사용자 환경과 다를 수 있으므로, **필드 데이터(Field Data)**인 CrUX(Chrome User Experience Report)를 함께 확인해야 합니다.

```typescript
// 목표 설정 가이드
const performanceGoals = {
  // 최소 목표 (Good 범위)
  minimum: {
    LCP: 2500,
    INP: 200,
    CLS: 0.1,
    lighthouseScore: 75,
  },

  // 권장 목표
  recommended: {
    LCP: 1800,
    INP: 100,
    CLS: 0.05,
    lighthouseScore: 90,
  },

  // 100점 목표보다 중요한 것
  priorities: [
    'Core Web Vitals Good 달성',
    '실제 사용자 메트릭(RUM) 개선',
    '비즈니스 지표 개선 (전환율, 이탈률)',
  ],
};
```

### Q7. 성능 최적화 작업의 ROI를 어떻게 측정하나요?

**A**: 성능 개선 전후의 비즈니스 지표 변화를 측정합니다.

```typescript
// ROI 측정 프레임워크
interface PerformanceROI {
  // 성능 지표
  performanceMetrics: {
    before: { lcp: number; inp: number; cls: number };
    after: { lcp: number; inp: number; cls: number };
    improvement: { lcp: string; inp: string; cls: string };
  };

  // 비즈니스 지표
  businessMetrics: {
    conversionRate: { before: number; after: number; change: string };
    bounceRate: { before: number; after: number; change: string };
    sessionDuration: { before: number; after: number; change: string };
    revenuePerSession: { before: number; after: number; change: string };
  };

  // ROI 계산
  roi: {
    developmentCost: number;      // 개발 비용
    monthlyRevenueLift: number;   // 월간 추가 수익
    paybackPeriod: string;        // 회수 기간
  };
}

// A/B 테스트로 검증
// - 50% 사용자: 최적화 버전
// - 50% 사용자: 기존 버전
// - 2-4주간 지표 비교
```

---

웹 성능 최적화는 끊임없이 발전하는 분야입니다. 새로운 기술과 도구가 계속 등장하므로, 공식 문서와 커뮤니티를 통해 최신 트렌드를 파악하는 것이 중요합니다. 무엇보다 실제 사용자 데이터를 기반으로 지속적으로 모니터링하고 개선하는 것이 가장 효과적인 성능 최적화 전략입니다.

---

## 참고 자료

### 공식 문서

- [Web.dev - Web Vitals](https://web.dev/articles/vitals)
- [MDN - Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [Chrome DevTools Performance](https://developer.chrome.com/docs/devtools/performance)
- [Next.js Optimizing](https://nextjs.org/docs/app/building-your-application/optimizing)
- [Vite Build Optimizations](https://vitejs.dev/guide/build.html)
- [Webpack Optimization](https://webpack.js.org/configuration/optimization/)

### 성능 측정 도구

- [PageSpeed Insights](https://pagespeed.web.dev/)
- [WebPageTest](https://www.webpagetest.org/)
- [Lighthouse](https://developer.chrome.com/docs/lighthouse/overview)
- [Web Vitals Extension](https://chrome.google.com/webstore/detail/web-vitals/ahfhijdlegdabablpippeagghigmibma)

### 추가 학습 자료

- [Google Web Fundamentals](https://developers.google.com/web/fundamentals/performance)
- [Performance Calendar](https://calendar.perfplanet.com/)
- [HTTP Archive State of the Web](https://httparchive.org/reports/state-of-the-web)
