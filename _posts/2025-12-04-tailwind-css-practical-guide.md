---
title: "Tailwind CSS 실전 가이드 - 유틸리티 퍼스트 CSS"
description: "Tailwind CSS의 핵심 개념부터 실전 활용까지 완벽 가이드. 유틸리티 클래스, 반응형 디자인, 커스터마이징, CVA와 tailwind-merge를 활용한 컴포넌트 패턴, 성능 최적화 전략을 다룹니다."
date: 2025-12-04 14:00:00 +0900
categories: [Frontend, CSS]
tags: [tailwind, css, utility-first, responsive, design-system, react, typescript, cva, nextjs]
---

## 들어가며

Tailwind CSS는 2017년 등장 이후 프론트엔드 개발 생태계에서 가장 빠르게 성장한 CSS 프레임워크입니다. State of CSS 2024 조사에서 사용률 1위를 기록했으며, Vercel, GitHub, Shopify, Netflix 등 주요 기업들이 채택했습니다.

이 글에서는 Tailwind CSS의 철학과 핵심 개념부터 실전 활용까지 체계적으로 다룹니다:

- **유틸리티 퍼스트 CSS**의 개념과 장단점
- **설치부터 프로덕션 배포**까지의 전체 워크플로우
- **반응형 디자인**과 **상태 변형** 완벽 가이드
- **성능 최적화**와 **베스트 프랙티스**

> Tailwind CSS를 다른 CSS 방법론과 비교하려면 [CSS 스타일링 전략 비교](/posts/css-styling-strategies-comparison/) 포스트를 참고하세요.

## 유틸리티 퍼스트 CSS란?

### 전통적인 CSS 방식

전통적인 CSS 개발에서는 시맨틱한 클래스명을 만들고, 별도의 CSS 파일에 스타일을 정의합니다.

```css
/* Button.css */
.btn-primary {
  background-color: #3b82f6;
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 0.375rem;
  font-weight: 500;
  transition: background-color 0.2s;
}

.btn-primary:hover {
  background-color: #2563eb;
}
```

```tsx
import './Button.css';

export function Button({ children }) {
  return (
    <button className="btn-primary">
      {children}
    </button>
  );
}
```

**문제점:**
- 새로운 컴포넌트마다 클래스명을 고민해야 함
- CSS 파일이 계속 커짐 (삭제하기 어려움)
- 파일 간 이동이 잦음 (CSS <-> JSX)
- 비슷한 스타일의 중복 정의

### 유틸리티 퍼스트 방식

유틸리티 퍼스트는 하나의 CSS 속성에 대응하는 작은 클래스들을 조합하여 스타일을 구성합니다.

```tsx
export function Button({ children }) {
  return (
    <button className="bg-blue-500 hover:bg-blue-600 text-white px-6 py-3 rounded-md font-medium transition-colors">
      {children}
    </button>
  );
}
```

**장점:**
- 별도의 CSS 파일 불필요
- 클래스명 고민 없음
- 컴포넌트만 보면 스타일 파악 가능
- 사용하지 않는 스타일 자동 제거
- 일관된 디자인 시스템 강제

### Tailwind의 핵심 철학

Tailwind CSS는 "제약된 자유(Constrained Freedom)"를 추구합니다.

```tsx
// 모든 값을 자유롭게 사용하는 대신
<div style={% raw %}{{ marginTop: '17px', padding: '13px' }}{% endraw %}>

// 미리 정의된 스케일을 사용
<div className="mt-4 p-3">  {/* mt-4 = 1rem, p-3 = 0.75rem */}
```

**디자인 토큰 예시:**

| 클래스 | 값 | 픽셀 환산 |
|--------|-----|---------|
| `p-1` | `0.25rem` | 4px |
| `p-2` | `0.5rem` | 8px |
| `p-3` | `0.75rem` | 12px |
| `p-4` | `1rem` | 16px |
| `p-6` | `1.5rem` | 24px |
| `p-8` | `2rem` | 32px |

이러한 제약은 디자인 일관성을 보장하고, 팀 전체가 같은 언어로 스타일을 표현할 수 있게 합니다.

## 설치 및 설정

### Vite + React 프로젝트

```bash
# 새 프로젝트 생성
npm create vite@latest my-app -- --template react-ts
cd my-app

# Tailwind CSS 설치
npm install -D tailwindcss postcss autoprefixer

# 설정 파일 생성
npx tailwindcss init -p
```

`tailwind.config.js` 설정:

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

`src/index.css`에 Tailwind 지시어 추가:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Next.js 프로젝트

Next.js 15 App Router에 대한 자세한 내용은 [Next.js 15 App Router 완벽 가이드](/posts/nextjs-15-app-router-guide/)를 참고하세요.

```bash
# Tailwind가 포함된 새 프로젝트
npx create-next-app@latest my-app --tailwind --typescript

# 기존 프로젝트에 추가
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

`tailwind.config.ts` (Next.js용):

```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};

export default config;
```

### PostCSS 설정

`postcss.config.js`:

```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### VS Code 개발 환경

필수 확장 프로그램:

```bash
# Tailwind CSS IntelliSense
# - 클래스 자동완성
# - 호버 시 CSS 미리보기
# - Lint 경고
```

`.vscode/settings.json`:

```jsonc
{
  // Tailwind CSS 자동완성 활성화
  "tailwindCSS.includeLanguages": {
    "javascript": "javascript",
    "typescript": "typescript",
    "javascriptreact": "javascriptreact",
    "typescriptreact": "typescriptreact"
  },
  // 클래스 정렬 시 Tailwind 순서 적용
  "editor.quickSuggestions": {
    "strings": "on"
  }
}
```

## 핵심 유틸리티 클래스

### 레이아웃 - Flexbox

```tsx
// 기본 Flex 컨테이너
<div className="flex">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

// 방향과 정렬
<div className="flex flex-col items-center justify-between gap-4">
  <div>위</div>
  <div>아래</div>
</div>

// Flex 아이템 제어
<div className="flex">
  <div className="flex-none w-20">고정 너비</div>
  <div className="flex-1">나머지 공간</div>
  <div className="flex-shrink-0">축소 안됨</div>
</div>
```

**주요 Flex 클래스:**

| 클래스 | CSS 속성 |
|--------|---------|
| `flex` | `display: flex` |
| `flex-row` | `flex-direction: row` |
| `flex-col` | `flex-direction: column` |
| `items-center` | `align-items: center` |
| `justify-between` | `justify-content: space-between` |
| `gap-4` | `gap: 1rem` |
| `flex-1` | `flex: 1 1 0%` |
| `flex-none` | `flex: none` |

### 레이아웃 - Grid

```tsx
// 기본 Grid
<div className="grid grid-cols-3 gap-4">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>

// 복잡한 레이아웃
<div className="grid grid-cols-12 gap-6">
  <aside className="col-span-3">사이드바</aside>
  <main className="col-span-6">메인 콘텐츠</main>
  <aside className="col-span-3">위젯</aside>
</div>

// Auto-fill 반응형 Grid
<div className="grid grid-cols-[repeat(auto-fill,minmax(250px,1fr))] gap-4">
  {/* 아이템이 자동으로 줄바꿈 */}
</div>
```

### 스페이싱 (Padding, Margin)

```tsx
// Padding
<div className="p-4">        {/* 모든 방향 1rem */}
<div className="px-4 py-2">  {/* 좌우 1rem, 상하 0.5rem */}
<div className="pt-4 pb-2">  {/* 상단 1rem, 하단 0.5rem */}

// Margin
<div className="m-4">        {/* 모든 방향 1rem */}
<div className="mx-auto">    {/* 가로 중앙 정렬 */}
<div className="mt-8 mb-4">  {/* 상단 2rem, 하단 1rem */}

// 음수 Margin
<div className="-mt-4">      {/* margin-top: -1rem */}

// Space Between (자식 요소 간격)
<div className="flex flex-col space-y-4">
  <div>첫 번째</div>
  <div>두 번째</div>  {/* 위에 1rem 간격 */}
  <div>세 번째</div>  {/* 위에 1rem 간격 */}
</div>
```

### 타이포그래피

```tsx
// 폰트 크기
<p className="text-sm">작은 텍스트 (0.875rem)</p>
<p className="text-base">기본 텍스트 (1rem)</p>
<p className="text-lg">큰 텍스트 (1.125rem)</p>
<p className="text-xl">더 큰 텍스트 (1.25rem)</p>
<p className="text-2xl">제목 크기 (1.5rem)</p>

// 폰트 굵기
<span className="font-light">Light</span>
<span className="font-normal">Normal</span>
<span className="font-medium">Medium</span>
<span className="font-semibold">Semibold</span>
<span className="font-bold">Bold</span>

// 줄 간격
<p className="leading-none">줄간격 1</p>
<p className="leading-tight">줄간격 1.25</p>
<p className="leading-normal">줄간격 1.5</p>
<p className="leading-relaxed">줄간격 1.625</p>
<p className="leading-loose">줄간격 2</p>

// 텍스트 정렬
<p className="text-left">왼쪽 정렬</p>
<p className="text-center">중앙 정렬</p>
<p className="text-right">오른쪽 정렬</p>

// 텍스트 꾸미기
<p className="underline">밑줄</p>
<p className="line-through">취소선</p>
<p className="truncate">긴 텍스트 말줄임...</p>
```

### 색상과 배경

```tsx
// 텍스트 색상
<p className="text-gray-900">진한 회색</p>
<p className="text-blue-600">파란색</p>
<p className="text-red-500">빨간색</p>

// 배경 색상
<div className="bg-white">흰색 배경</div>
<div className="bg-gray-100">연한 회색 배경</div>
<div className="bg-blue-500">파란 배경</div>

// 투명도
<div className="bg-black/50">반투명 검정 (50%)</div>
<div className="bg-blue-500/75">반투명 파랑 (75%)</div>

// 그라데이션
<div className="bg-gradient-to-r from-blue-500 to-purple-500">
  왼쪽에서 오른쪽 그라데이션
</div>

<div className="bg-gradient-to-br from-pink-500 via-red-500 to-yellow-500">
  대각선 3색 그라데이션
</div>
```

**Tailwind 색상 팔레트:**

```
50  - 가장 밝은 색
100 - 매우 밝음
200 - 밝음
300 - 약간 밝음
400 - 중간 밝음
500 - 기본 (메인 색상)
600 - 약간 어두움
700 - 어두움
800 - 매우 어두움
900 - 가장 어두운 색
950 - 거의 검정
```

### 테두리와 그림자

```tsx
// 테두리
<div className="border">기본 테두리</div>
<div className="border-2">두꺼운 테두리</div>
<div className="border-t">상단만 테두리</div>
<div className="border-gray-300">회색 테두리</div>
<div className="border-dashed">점선 테두리</div>

// 둥근 모서리
<div className="rounded">약간 둥글게 (0.25rem)</div>
<div className="rounded-md">중간 둥글게 (0.375rem)</div>
<div className="rounded-lg">많이 둥글게 (0.5rem)</div>
<div className="rounded-full">완전히 둥글게</div>
<div className="rounded-t-lg">상단만 둥글게</div>

// 그림자
<div className="shadow-sm">작은 그림자</div>
<div className="shadow">기본 그림자</div>
<div className="shadow-md">중간 그림자</div>
<div className="shadow-lg">큰 그림자</div>
<div className="shadow-xl">매우 큰 그림자</div>
<div className="shadow-2xl">가장 큰 그림자</div>
```

### 크기 조절 (Width, Height)

```tsx
// 고정 너비
<div className="w-20">5rem</div>
<div className="w-64">16rem</div>
<div className="w-full">100%</div>
<div className="w-screen">100vw</div>
<div className="w-1/2">50%</div>
<div className="w-1/3">33.333%</div>

// 최소/최대 너비
<div className="min-w-0">min-width: 0</div>
<div className="max-w-md">max-width: 28rem</div>
<div className="max-w-screen-xl">max-width: 1280px</div>

// 높이
<div className="h-20">5rem</div>
<div className="h-screen">100vh</div>
<div className="min-h-screen">min-height: 100vh</div>

// 종횡비 (Aspect Ratio)
<div className="aspect-video">16:9 비율</div>
<div className="aspect-square">1:1 비율</div>
```

## 반응형 디자인

### 브레이크포인트 시스템

Tailwind는 **모바일 퍼스트** 접근법을 사용합니다:

| 접두사 | 최소 너비 | CSS |
|--------|----------|-----|
| (없음) | 0px | 기본 (모바일) |
| `sm:` | 640px | `@media (min-width: 640px)` |
| `md:` | 768px | `@media (min-width: 768px)` |
| `lg:` | 1024px | `@media (min-width: 1024px)` |
| `xl:` | 1280px | `@media (min-width: 1280px)` |
| `2xl:` | 1536px | `@media (min-width: 1536px)` |

### 반응형 유틸리티 사용법

```tsx
// 기본: 모바일부터 시작하여 큰 화면에서 변경
<div className="text-sm md:text-base lg:text-lg">
  {/* 모바일: small, 태블릿: base, 데스크탑: large */}
</div>

// 반응형 레이아웃
<div className="flex flex-col md:flex-row gap-4">
  <div className="w-full md:w-1/3">사이드바</div>
  <div className="w-full md:w-2/3">메인 콘텐츠</div>
</div>

// 반응형 Grid
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
  {/* 화면 크기에 따라 열 수 변경 */}
</div>

// 반응형 숨김/표시
<nav className="hidden md:flex">데스크탑 네비게이션</nav>
<button className="md:hidden">모바일 메뉴 버튼</button>
```

### 실전 반응형 레이아웃 예제

```tsx
// 반응형 Hero 섹션
function HeroSection() {
  return (
    <section className="px-4 py-12 md:px-8 md:py-20 lg:py-32">
      <div className="max-w-6xl mx-auto">
        <div className="flex flex-col lg:flex-row items-center gap-8 lg:gap-16">
          {/* 텍스트 영역 */}
          <div className="flex-1 text-center lg:text-left">
            <h1 className="text-3xl md:text-4xl lg:text-5xl xl:text-6xl font-bold mb-4 md:mb-6">
              멋진 제목이 들어갑니다
            </h1>
            <p className="text-gray-600 text-base md:text-lg lg:text-xl mb-6 md:mb-8">
              설명 텍스트가 여기에 들어갑니다.
              반응형으로 크기가 조절됩니다.
            </p>
            <div className="flex flex-col sm:flex-row gap-4 justify-center lg:justify-start">
              <button className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
                시작하기
              </button>
              <button className="px-6 py-3 border border-gray-300 rounded-lg hover:bg-gray-50">
                더 알아보기
              </button>
            </div>
          </div>

          {/* 이미지 영역 */}
          <div className="flex-1 w-full max-w-md lg:max-w-none">
            <img
              src="/hero-image.png"
              alt="Hero"
              className="w-full h-auto rounded-2xl shadow-xl"
            />
          </div>
        </div>
      </div>
    </section>
  );
}
```

### 컨테이너 쿼리 (Tailwind v3.2+)

```tsx
// 부모 요소 크기에 따른 스타일링
<div className="@container">
  <div className="@lg:flex @lg:gap-4">
    {/* 컨테이너가 1024px 이상일 때 flex */}
  </div>
</div>
```

`tailwind.config.js`:

```javascript
export default {
  plugins: [
    require('@tailwindcss/container-queries'),
  ],
}
```

## 상태 변형 (State Variants)

### 기본 상태

```tsx
// hover, focus, active
<button className="bg-blue-500 hover:bg-blue-600 focus:ring-2 focus:ring-blue-300 active:bg-blue-700">
  버튼
</button>

// focus-visible (키보드 포커스만)
<button className="focus-visible:ring-2 focus-visible:ring-offset-2">
  접근성 고려 버튼
</button>

// disabled
<button className="bg-blue-500 disabled:bg-gray-300 disabled:cursor-not-allowed" disabled>
  비활성화 버튼
</button>
```

### 자식 요소 상태 (group)

부모의 상태에 따라 자식 요소 스타일을 변경할 때 사용합니다:

```tsx
<div className="group p-4 bg-white hover:bg-gray-50 rounded-lg transition-colors">
  <h3 className="font-semibold group-hover:text-blue-600">제목</h3>
  <p className="text-gray-500 group-hover:text-gray-700">설명</p>
  <span className="opacity-0 group-hover:opacity-100 transition-opacity">
    더보기 →
  </span>
</div>
```

```tsx
// 중첩된 group
<div className="group/card p-4 bg-white">
  <div className="group/title flex items-center">
    <h3 className="group-hover/title:text-blue-600">제목</h3>
    <span className="group-hover/card:opacity-100 opacity-0">카드 호버</span>
  </div>
</div>
```

### 형제 요소 상태 (peer)

형제 요소의 상태에 따라 스타일을 변경합니다:

```tsx
// 체크박스에 따른 라벨 스타일
<label className="flex items-center gap-2">
  <input type="checkbox" className="peer" />
  <span className="peer-checked:text-blue-600 peer-checked:font-semibold">
    동의합니다
  </span>
</label>

// 입력 유효성에 따른 메시지
<div>
  <input
    type="email"
    className="peer border rounded px-3 py-2 invalid:border-red-500"
    required
  />
  <p className="hidden peer-invalid:block text-red-500 text-sm mt-1">
    유효한 이메일을 입력하세요
  </p>
</div>
```

### 리스트 상태

```tsx
<ul>
  {items.map((item, index) => (
    <li
      key={item.id}
      className="py-2 first:pt-0 last:pb-0 odd:bg-gray-50 even:bg-white"
    >
      {item.name}
    </li>
  ))}
</ul>
```

### 다크 모드

```tsx
// 시스템 설정 기반 (기본)
<div className="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">
  다크 모드 지원
</div>

// 수동 토글
<button
  className="bg-gray-200 dark:bg-gray-700"
  onClick={() => document.documentElement.classList.toggle('dark')}
>
  테마 전환
</button>
```

`tailwind.config.js`:

```javascript
export default {
  darkMode: 'class', // 'media' (시스템) 또는 'class' (수동)
}
```

## 커스터마이징

디자인 토큰과 테마 시스템에 대한 자세한 내용은 [디자인 시스템 구축 가이드](/posts/design-system-guide/)를 참고하세요.

### 테마 확장

```javascript
// tailwind.config.js
export default {
  theme: {
    extend: {
      // 색상 추가
      colors: {
        brand: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
        primary: 'rgb(var(--color-primary) / <alpha-value>)',
      },

      // 폰트 추가
      fontFamily: {
        sans: ['Pretendard', 'sans-serif'],
        display: ['Montserrat', 'sans-serif'],
      },

      // 스페이싱 추가
      spacing: {
        '18': '4.5rem',
        '128': '32rem',
      },

      // 애니메이션 추가
      animation: {
        'fade-in': 'fadeIn 0.5s ease-in-out',
        'slide-up': 'slideUp 0.3s ease-out',
      },
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        slideUp: {
          '0%': { transform: 'translateY(10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      },
    },
  },
}
```

### CSS 변수와 함께 사용

```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --color-primary: 59 130 246;
    --color-secondary: 100 116 139;
    --radius: 0.5rem;
  }

  .dark {
    --color-primary: 96 165 250;
    --color-secondary: 148 163 184;
  }
}
```

```javascript
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        primary: 'rgb(var(--color-primary) / <alpha-value>)',
        secondary: 'rgb(var(--color-secondary) / <alpha-value>)',
      },
      borderRadius: {
        DEFAULT: 'var(--radius)',
      },
    },
  },
}
```

```tsx
// 사용
<button className="bg-primary text-white hover:bg-primary/80">
  CSS 변수 기반 버튼
</button>
```

### 커스텀 유틸리티 추가

```javascript
// tailwind.config.js
const plugin = require('tailwindcss/plugin');

export default {
  plugins: [
    plugin(function({ addUtilities, addComponents, theme }) {
      // 유틸리티 추가
      addUtilities({
        '.text-balance': {
          'text-wrap': 'balance',
        },
        '.scrollbar-hide': {
          '-ms-overflow-style': 'none',
          'scrollbar-width': 'none',
          '&::-webkit-scrollbar': {
            display: 'none',
          },
        },
      });

      // 컴포넌트 스타일 추가
      addComponents({
        '.btn': {
          padding: theme('spacing.2') + ' ' + theme('spacing.4'),
          borderRadius: theme('borderRadius.md'),
          fontWeight: theme('fontWeight.medium'),
        },
      });
    }),
  ],
}
```

### 공식 플러그인

```bash
npm install -D @tailwindcss/typography @tailwindcss/forms @tailwindcss/aspect-ratio @tailwindcss/container-queries
```

```javascript
// tailwind.config.js
export default {
  plugins: [
    require('@tailwindcss/typography'),  // prose 클래스
    require('@tailwindcss/forms'),        // 폼 기본 스타일
    require('@tailwindcss/aspect-ratio'), // 종횡비
    require('@tailwindcss/container-queries'), // 컨테이너 쿼리
  ],
}
```

```tsx
// Typography 플러그인 사용
<article className="prose prose-lg dark:prose-invert max-w-none">
  <h1>마크다운 콘텐츠</h1>
  <p>자동으로 예쁜 타이포그래피가 적용됩니다.</p>
</article>

// Forms 플러그인 사용
<input
  type="text"
  className="rounded-md border-gray-300 focus:border-blue-500 focus:ring-blue-500"
/>
```

## 컴포넌트 패턴

### @apply로 클래스 추출

반복되는 클래스 조합을 CSS로 추출할 수 있습니다:

```css
/* globals.css */
@layer components {
  .btn {
    @apply px-4 py-2 rounded-md font-medium transition-colors;
  }

  .btn-primary {
    @apply btn bg-blue-500 text-white hover:bg-blue-600;
  }

  .btn-secondary {
    @apply btn bg-gray-200 text-gray-900 hover:bg-gray-300;
  }

  .input {
    @apply w-full px-3 py-2 border border-gray-300 rounded-md
           focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent;
  }
}
```

```tsx
<button className="btn-primary">Primary</button>
<button className="btn-secondary">Secondary</button>
<input className="input" placeholder="이메일" />
```

**주의:** `@apply`는 남용하면 Tailwind의 장점을 잃습니다. 정말 반복되는 패턴에만 사용하세요.

### CVA (Class Variance Authority)

타입 안전한 variant 기반 컴포넌트를 만들 때 사용합니다:

```bash
npm install class-variance-authority
```

```tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // 기본 클래스
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:ring-blue-600',
        destructive: 'bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-600',
        outline: 'border border-gray-300 bg-transparent hover:bg-gray-100',
        ghost: 'hover:bg-gray-100',
        link: 'text-blue-600 underline-offset-4 hover:underline',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4',
        lg: 'h-12 px-6 text-lg',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button
      className={buttonVariants({ variant, size, className })}
      {...props}
    />
  );
}
```

```tsx
// 사용
<Button>기본 버튼</Button>
<Button variant="destructive" size="lg">삭제</Button>
<Button variant="outline">취소</Button>
<Button variant="ghost" size="icon"><IconMenu /></Button>
```

### tailwind-merge

클래스 충돌을 지능적으로 해결합니다:

```bash
npm install tailwind-merge
```

```tsx
import { twMerge } from 'tailwind-merge';

// 문제: 나중 클래스가 앞 클래스를 덮어쓰지 못할 수 있음
const className = 'px-4 py-2 px-6'; // px-4가 적용될 수도 있음

// 해결: twMerge 사용
const mergedClassName = twMerge('px-4 py-2', 'px-6'); // 'py-2 px-6'
```

### clsx + twMerge 조합 (cn 유틸리티)

```bash
npm install clsx tailwind-merge
```

```tsx
// lib/utils.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```tsx
// 컴포넌트에서 사용
import { cn } from '@/lib/utils';

interface CardProps {
  className?: string;
  variant?: 'default' | 'bordered';
}

export function Card({ className, variant = 'default' }: CardProps) {
  return (
    <div
      className={cn(
        'rounded-lg p-6',
        variant === 'default' && 'bg-white shadow-md',
        variant === 'bordered' && 'border border-gray-200',
        className // 외부에서 전달된 클래스로 덮어쓰기 가능
      )}
    />
  );
}
```

## 실전 예제

### 카드 컴포넌트

```tsx
interface CardProps {
  title: string;
  description: string;
  imageUrl?: string;
  tags?: string[];
  href?: string;
}

export function Card({ title, description, imageUrl, tags, href }: CardProps) {
  const Wrapper = href ? 'a' : 'div';

  return (
    <Wrapper
      href={href}
      className={cn(
        'group block overflow-hidden rounded-xl bg-white shadow-sm',
        'border border-gray-200 transition-all duration-300',
        href && 'hover:shadow-lg hover:border-gray-300 hover:-translate-y-1'
      )}
    >
      {/* 이미지 */}
      {imageUrl && (
        <div className="aspect-video overflow-hidden">
          <img
            src={imageUrl}
            alt={title}
            className="h-full w-full object-cover transition-transform duration-300 group-hover:scale-105"
          />
        </div>
      )}

      {/* 콘텐츠 */}
      <div className="p-5">
        {/* 태그 */}
        {tags && tags.length > 0 && (
          <div className="mb-3 flex flex-wrap gap-2">
            {tags.map((tag) => (
              <span
                key={tag}
                className="inline-flex items-center rounded-full bg-blue-50 px-2.5 py-0.5 text-xs font-medium text-blue-700"
              >
                {tag}
              </span>
            ))}
          </div>
        )}

        {/* 제목 */}
        <h3 className="mb-2 text-lg font-semibold text-gray-900 group-hover:text-blue-600 transition-colors">
          {title}
        </h3>

        {/* 설명 */}
        <p className="text-sm text-gray-500 line-clamp-2">
          {description}
        </p>
      </div>
    </Wrapper>
  );
}
```

### 네비게이션 바

```tsx
import { useState } from 'react';

const navItems = [
  { label: '홈', href: '/' },
  { label: '제품', href: '/products' },
  { label: '가격', href: '/pricing' },
  { label: '블로그', href: '/blog' },
];

export function Navbar() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <nav className="sticky top-0 z-50 bg-white/80 backdrop-blur-md border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex items-center justify-between h-16">
          {/* 로고 */}
          <a href="/" className="flex items-center gap-2">
            <div className="w-8 h-8 bg-blue-600 rounded-lg" />
            <span className="font-bold text-xl">Logo</span>
          </a>

          {/* 데스크탑 메뉴 */}
          <div className="hidden md:flex items-center gap-8">
            {navItems.map((item) => (
              <a
                key={item.href}
                href={item.href}
                className="text-gray-600 hover:text-gray-900 font-medium transition-colors"
              >
                {item.label}
              </a>
            ))}
          </div>

          {/* CTA 버튼 */}
          <div className="hidden md:flex items-center gap-4">
            <a href="/login" className="text-gray-600 hover:text-gray-900 font-medium">
              로그인
            </a>
            <a
              href="/signup"
              className="bg-blue-600 text-white px-4 py-2 rounded-lg font-medium hover:bg-blue-700 transition-colors"
            >
              시작하기
            </a>
          </div>

          {/* 모바일 메뉴 버튼 */}
          <button
            onClick={() => setIsOpen(!isOpen)}
            className="md:hidden p-2 rounded-lg hover:bg-gray-100"
            aria-label="메뉴 열기"
          >
            <svg
              className="w-6 h-6"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              {isOpen ? (
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
              ) : (
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
              )}
            </svg>
          </button>
        </div>
      </div>

      {/* 모바일 메뉴 */}
      <div
        className={cn(
          'md:hidden border-t border-gray-200 bg-white',
          isOpen ? 'block' : 'hidden'
        )}
      >
        <div className="px-4 py-4 space-y-3">
          {navItems.map((item) => (
            <a
              key={item.href}
              href={item.href}
              className="block py-2 text-gray-600 hover:text-gray-900 font-medium"
            >
              {item.label}
            </a>
          ))}
          <hr className="my-4" />
          <a href="/login" className="block py-2 text-gray-600 hover:text-gray-900 font-medium">
            로그인
          </a>
          <a
            href="/signup"
            className="block w-full text-center bg-blue-600 text-white py-2 rounded-lg font-medium"
          >
            시작하기
          </a>
        </div>
      </div>
    </nav>
  );
}
```

### 폼 요소 스타일링

```tsx
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

export function Input({ label, error, className, ...props }: InputProps) {
  return (
    <div className="space-y-1">
      <label className="block text-sm font-medium text-gray-700">
        {label}
      </label>
      <input
        className={cn(
          'w-full px-3 py-2 border rounded-lg transition-colors',
          'focus:outline-none focus:ring-2 focus:ring-offset-0',
          error
            ? 'border-red-500 focus:ring-red-500'
            : 'border-gray-300 focus:ring-blue-500 focus:border-blue-500',
          'disabled:bg-gray-50 disabled:text-gray-500 disabled:cursor-not-allowed',
          className
        )}
        {...props}
      />
      {error && (
        <p className="text-sm text-red-500">{error}</p>
      )}
    </div>
  );
}

interface SelectProps extends React.SelectHTMLAttributes<HTMLSelectElement> {
  label: string;
  options: { value: string; label: string }[];
}

export function Select({ label, options, className, ...props }: SelectProps) {
  return (
    <div className="space-y-1">
      <label className="block text-sm font-medium text-gray-700">
        {label}
      </label>
      <select
        className={cn(
          'w-full px-3 py-2 border border-gray-300 rounded-lg',
          'focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500',
          'bg-white',
          className
        )}
        {...props}
      >
        {options.map((option) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
    </div>
  );
}
```

### 모달 컴포넌트

```tsx
import { useEffect } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

export function Modal({ isOpen, onClose, title, children }: ModalProps) {
  // ESC 키로 닫기
  useEffect(() => {
    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEsc);
      document.body.style.overflow = 'hidden';
    }

    return () => {
      document.removeEventListener('keydown', handleEsc);
      document.body.style.overflow = 'unset';
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      {/* 오버레이 */}
      <div
        className="absolute inset-0 bg-black/50 backdrop-blur-sm animate-fade-in"
        onClick={onClose}
      />

      {/* 모달 */}
      <div className="relative z-10 w-full max-w-lg mx-4 bg-white rounded-xl shadow-2xl animate-slide-up">
        {/* 헤더 */}
        <div className="flex items-center justify-between px-6 py-4 border-b border-gray-200">
          <h2 className="text-lg font-semibold text-gray-900">{title}</h2>
          <button
            onClick={onClose}
            className="p-2 rounded-lg hover:bg-gray-100 transition-colors"
            aria-label="닫기"
          >
            <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        {/* 콘텐츠 */}
        <div className="px-6 py-4">
          {children}
        </div>
      </div>
    </div>
  );
}
```

## 성능 최적화

Core Web Vitals와 성능 최적화에 대한 종합적인 가이드는 [웹 성능 최적화 완벽 가이드](/posts/web-performance-optimization-guide/)를 참고하세요.

### Content 설정과 PurgeCSS

Tailwind는 `content` 설정에 지정된 파일에서 사용된 클래스만 빌드에 포함합니다:

```javascript
// tailwind.config.js
export default {
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx}',
    // 주의: node_modules의 UI 라이브러리도 포함해야 함
    './node_modules/@my-ui/**/*.{js,ts,jsx,tsx}',
  ],
}
```

**주의사항:**

```tsx
// 동적 클래스 생성 시 문제
const color = 'blue';
<div className={`bg-${color}-500`} /> // 작동 안함!

// 해결: 전체 클래스명 사용
const colorClass = {
  blue: 'bg-blue-500',
  red: 'bg-red-500',
  green: 'bg-green-500',
}[color];
<div className={colorClass} /> // 작동함
```

### 번들 크기 분석

```bash
# 빌드 후 CSS 크기 확인
npx tailwindcss -o output.css --minify
ls -lh output.css

# 일반적인 크기:
# 개발 빌드: 3-4 MB (전체 유틸리티)
# 프로덕션 빌드: 10-30 KB (사용된 클래스만)
```

### JIT 모드 이해

Tailwind v3부터 JIT(Just-In-Time) 모드가 기본입니다:

**장점:**
- 개발 중에도 사용된 클래스만 생성
- 임의 값(Arbitrary values) 지원
- 빌드 시간 단축

```tsx
// JIT에서 가능한 임의 값
<div className="w-[137px]">정확히 137px</div>
<div className="bg-[#1da1f2]">트위터 파랑</div>
<div className="grid-cols-[1fr_auto_2fr]">커스텀 그리드</div>
<div className="text-[clamp(1rem,5vw,3rem)]">반응형 폰트</div>
```

### 개발 vs 프로덕션 빌드

```bash
# 개발
npm run dev
# - 전체 유틸리티 로드
# - Hot Module Replacement
# - 소스맵 포함

# 프로덕션
npm run build
# - Tree-shaking (사용된 클래스만)
# - 클래스명 최적화
# - CSS 압축
```

### Lighthouse 최적화 팁

```tsx
// 1. 초기 로드 CSS 최소화
// Critical CSS만 인라인, 나머지는 비동기 로드

// 2. 레이아웃 시프트 방지
<img
  src="/image.jpg"
  alt=""
  className="aspect-video w-full"  // 미리 공간 확보
/>

// 3. 폰트 최적화
<link
  rel="preload"
  href="/fonts/pretendard.woff2"
  as="font"
  type="font/woff2"
  crossOrigin="anonymous"
/>
```

## 베스트 프랙티스

### 클래스 정렬 규칙 (Prettier 플러그인)

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

`.prettierrc`:

```jsonc
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindFunctions": ["clsx", "cn", "cva"]
}
```

정렬 전:
```tsx
<div className="text-center bg-white p-4 flex rounded-lg shadow-md items-center">
```

정렬 후:
```tsx
<div className="flex items-center rounded-lg bg-white p-4 text-center shadow-md">
```

### 일관된 스타일 유지

**1. 디자인 토큰 활용:**

```javascript
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        // 시맨틱 컬러 정의
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
      },
    },
  },
}
```

**2. 컴포넌트 추상화:**

```tsx
// 반복되는 패턴은 컴포넌트로
function SectionTitle({ children }) {
  return (
    <h2 className="text-2xl font-bold text-gray-900 mb-6">
      {children}
    </h2>
  );
}

function SectionDescription({ children }) {
  return (
    <p className="text-gray-600 leading-relaxed">
      {children}
    </p>
  );
}
```

### 팀 협업 가이드라인

**1. ESLint 규칙:**

```bash
npm install -D eslint-plugin-tailwindcss
```

```javascript
// .eslintrc.js
module.exports = {
  plugins: ['tailwindcss'],
  rules: {
    'tailwindcss/classnames-order': 'warn',
    'tailwindcss/no-custom-classname': 'warn',
    'tailwindcss/no-contradicting-classname': 'error',
  },
}
```

**2. 코드 리뷰 체크리스트:**

- 반응형 디자인 고려 여부
- 다크 모드 지원 여부
- 접근성 (포커스 스타일, 색상 대비)
- 불필요한 커스텀 값 사용
- 중복 클래스 여부

### 흔한 실수와 해결 방법

**1. 동적 클래스 문제:**

```tsx
// 문제
const size = 'lg';
<div className={`text-${size}`} /> // PurgeCSS가 찾지 못함

// 해결
const sizeClasses = {
  sm: 'text-sm',
  md: 'text-base',
  lg: 'text-lg',
};
<div className={sizeClasses[size]} />
```

**2. Important 남용:**

```tsx
// 피해야 함
<div className="!p-4" />

// 대신 specificity 이해하고 구조 개선
```

**3. 과도한 커스텀 값:**

```tsx
// 피해야 함 - 디자인 시스템 깨짐
<div className="mt-[17px] p-[13px] text-[15px]" />

// 권장 - 가장 가까운 스케일 사용
<div className="mt-4 p-3 text-sm" />
```

**4. @apply 남용:**

```css
/* 피해야 함 - Tailwind 장점 상실 */
.card {
  @apply flex flex-col items-center justify-center p-4 bg-white rounded-lg shadow-md;
}

/* 권장 - 진짜 반복될 때만 */
.btn {
  @apply px-4 py-2 rounded-md font-medium;
}
```

## 마치며

Tailwind CSS는 2025년 현재 가장 인기 있는 CSS 솔루션입니다. 성공적인 도입을 위한 핵심 포인트를 정리하면:

**1. 유틸리티 퍼스트 사고방식 전환**
- CSS 파일 없이 컴포넌트에서 바로 스타일링
- 디자인 토큰 기반의 일관된 스타일
- 사용하지 않는 CSS 자동 제거

**2. 생산성과 성능의 균형**
- 빠른 프로토타이핑과 개발
- 제로 런타임 오버헤드
- 작은 프로덕션 번들

**3. 팀 협업 최적화**
- Prettier 플러그인으로 클래스 정렬
- ESLint로 일관성 유지
- 컴포넌트 추상화로 재사용성 확보

**4. 실전 도입 전략**
- 새 프로젝트는 처음부터 Tailwind로
- 기존 프로젝트는 점진적 마이그레이션
- CVA + twMerge로 타입 안전한 컴포넌트

Tailwind는 도구일 뿐입니다. 팀의 상황과 프로젝트 요구사항에 맞게 활용하되, 최종 사용자에게 가치를 전달하는 것이 가장 중요합니다.

## 참고 자료

- [Tailwind CSS 공식 문서](https://tailwindcss.com/docs)
- [Tailwind CSS v4 발표](https://tailwindcss.com/blog/tailwindcss-v4)
- [Tailwind UI](https://tailwindui.com/) - 공식 컴포넌트 라이브러리
- [Headless UI](https://headlessui.com/) - 접근성 고려된 UI 컴포넌트
- [Class Variance Authority](https://cva.style/) - 타입 안전한 variant
- [tailwind-merge](https://github.com/dcastil/tailwind-merge) - 클래스 충돌 해결
- [Prettier Plugin Tailwindcss](https://github.com/tailwindlabs/prettier-plugin-tailwindcss)
- [State of CSS 2024](https://stateofcss.com/)
