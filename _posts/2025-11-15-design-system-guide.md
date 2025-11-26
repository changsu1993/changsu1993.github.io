---
title: "디자인 시스템 구축 완벽 가이드 - Storybook + Tailwind로 확장 가능한 컴포넌트 라이브러리 만들기"
date: 2025-11-15 15:45:00 +0900
categories: [Frontend, Design System]
tags: [design-system, storybook, tailwind, react]
author: changsu
toc: true
comments: true
description: Storybook 8과 Tailwind CSS 3를 활용한 확장 가능한 디자인 시스템 구축 가이드. 디자인 토큰, 컴포넌트 라이브러리, 접근성, 문서화 전략까지 실전 예제로 배우는 완벽한 디자인 시스템 아키텍처입니다.
---

## 개요

디자인 시스템은 제품의 UI/UX 일관성을 보장하고 개발 생산성을 극대화하는 핵심 인프라입니다. 이 가이드에서는 Storybook 8과 Tailwind CSS 3를 활용하여 확장 가능하고 유지보수가 쉬운 디자인 시스템을 구축하는 방법을 다룹니다.

**이 글에서 배울 수 있는 것:**
- 디자인 시스템의 개념과 필요성
- Storybook 8 최신 기능 활용법
- Tailwind CSS와 CVA를 활용한 변형 관리
- 디자인 토큰 체계 설계
- 접근성을 고려한 컴포넌트 구축
- 효과적인 문서화 전략
- 버전 관리 및 배포 자동화

**사전 요구사항:**
- React 18+ 기본 지식
- TypeScript 기본 문법
- CSS 기초 (Flexbox, Grid)
- npm/yarn 패키지 관리 경험

**예상 소요 시간:** 약 45분

---

## 디자인 시스템이란?

### 정의

디자인 시스템은 **재사용 가능한 컴포넌트와 명확한 표준을 통해 대규모로 디자인을 관리하는 체계**입니다.

```text
디자인 시스템 = 디자인 토큰 + 컴포넌트 라이브러리 + 사용 가이드 + 패턴 라이브러리
```

### 구성 요소

**1. 디자인 토큰 (Design Tokens)**
```typescript
// 색상, 타이포그래피, 스페이싱 등의 디자인 결정을 코드로 표현
const tokens = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
  },
};
```

**2. 컴포넌트 라이브러리**
```jsx
// 재사용 가능한 UI 컴포넌트
<Button variant="primary" size="md">
  Click me
</Button>
```

**3. 문서화**
```markdown
# Button 컴포넌트
## 사용법
## Props
## 접근성
## 예제
```

**4. 디자인 패턴**
```jsx
// 복합 패턴 (여러 컴포넌트 조합)
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>
```

### 필요성

**없을 때의 문제점:**

```jsx
// ❌ 팀마다 다른 스타일
// 팀 A
<button style={{ backgroundColor: '#007bff', padding: '10px 20px' }}>
  클릭
</button>

// 팀 B
<button style={{ backgroundColor: '#0069d9', padding: '12px 24px' }}>
  클릭
</button>

// 결과: 일관성 없는 UI, 유지보수 어려움
```

**있을 때의 이점:**

```jsx
// ✅ 모든 팀이 같은 컴포넌트 사용
<Button variant="primary" size="md">
  클릭
</Button>

// 결과: 일관성 보장, 빠른 개발, 쉬운 유지보수
```

**구체적 이점:**

1. **일관성 보장**: 모든 제품에서 동일한 UI/UX
2. **개발 속도 향상**: 재사용 가능한 컴포넌트로 빠른 개발
3. **유지보수 용이**: 한 곳에서 변경하면 모든 곳에 반영
4. **협업 효율성**: 디자이너-개발자 간 공통 언어
5. **접근성 보장**: 한 번 구현하면 모든 곳에서 접근 가능
6. **확장성**: 새로운 제품/기능 추가가 쉬움

### 유명 디자인 시스템

- **Material Design** (Google): Android, Web
- **Fluent Design** (Microsoft): Windows, Office
- **Human Interface Guidelines** (Apple): iOS, macOS
- **Carbon Design System** (IBM): 엔터프라이즈
- **Ant Design** (Alibaba): 관리자 대시보드
- **Chakra UI**: React 생태계
- **shadcn/ui**: Copy-paste 스타일 디자인 시스템

---

## 프로젝트 셋업

### 초기 설정

```bash
# React + TypeScript 프로젝트 생성
npx create-react-app my-design-system --template typescript

# 또는 Vite 사용 (더 빠름)
npm create vite@latest my-design-system -- --template react-ts

cd my-design-system
```

### 필수 패키지 설치

```bash
# Tailwind CSS 설치
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Storybook 설치 (최신 v8)
npx storybook@latest init

# CVA (Class Variance Authority) - Tailwind 변형 관리
npm install class-variance-authority clsx tailwind-merge

# Radix UI - 접근성 기반 Headless 컴포넌트
npm install @radix-ui/react-slot
npm install @radix-ui/react-dialog
npm install @radix-ui/react-dropdown-menu
npm install @radix-ui/react-select

# 유틸리티
npm install lucide-react # 아이콘
```

### Tailwind CSS 설정

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
    './.storybook/**/*.{js,jsx,ts,tsx}', // Storybook도 포함
  ],
  theme: {
    extend: {
      colors: {
        // 디자인 토큰으로 정의할 색상
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6', // 기본 primary
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
          950: '#172554',
        },
        secondary: {
          50: '#f9fafb',
          100: '#f3f4f6',
          200: '#e5e7eb',
          300: '#d1d5db',
          400: '#9ca3af',
          500: '#6b7280',
          600: '#4b5563',
          700: '#374151',
          800: '#1f2937',
          900: '#111827',
          950: '#030712',
        },
      },
      spacing: {
        // 커스텀 스페이싱
        '18': '4.5rem',
        '88': '22rem',
      },
      fontSize: {
        // 타이포그래피
        'xs': ['0.75rem', { lineHeight: '1rem' }],
        'sm': ['0.875rem', { lineHeight: '1.25rem' }],
        'base': ['1rem', { lineHeight: '1.5rem' }],
        'lg': ['1.125rem', { lineHeight: '1.75rem' }],
        'xl': ['1.25rem', { lineHeight: '1.75rem' }],
        '2xl': ['1.5rem', { lineHeight: '2rem' }],
        '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
      },
      borderRadius: {
        // 둥근 모서리
        'sm': '0.25rem',
        'DEFAULT': '0.5rem',
        'md': '0.5rem',
        'lg': '0.75rem',
        'xl': '1rem',
      },
    },
  },
  plugins: [],
};
```

```text
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 전역 스타일 */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
  }

  * {
    @apply border-border;
  }

  body {
    @apply bg-background text-foreground;
  }
}
```

### Storybook 설정

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y', // 접근성 테스트
  ],
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

```typescript
// .storybook/preview.ts
import type { Preview } from '@storybook/react';
import '../src/index.css'; // Tailwind CSS 임포트

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
    // 다크모드 지원
    backgrounds: {
      default: 'light',
      values: [
        {
          name: 'light',
          value: '#ffffff',
        },
        {
          name: 'dark',
          value: '#1a202c',
        },
      ],
    },
  },
};

export default preview;
```

### 유틸리티 함수

```typescript
// src/lib/utils.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

/**
 * Tailwind 클래스를 병합하는 유틸리티
 * - clsx로 조건부 클래스 처리
 * - twMerge로 Tailwind 클래스 충돌 해결
 */
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**사용 예시:**
```typescript
// 충돌하는 클래스 자동 해결
cn('px-2 py-1', 'px-4') // → 'py-1 px-4' (나중 것이 우선)
cn('text-red-500', condition && 'text-blue-500') // 조건부 스타일
```

---

## 디자인 토큰 시스템

### 디자인 토큰이란?

디자인 토큰은 **디자인 결정을 코드로 표현한 변수**입니다. 색상, 타이포그래피, 스페이싱 등을 중앙에서 관리합니다.

### 색상 토큰

```typescript
// src/tokens/colors.ts
export const colors = {
  // Primary - 주요 액션, 링크
  primary: {
    50: '#eff6ff',
    100: '#dbeafe',
    200: '#bfdbfe',
    300: '#93c5fd',
    400: '#60a5fa',
    500: '#3b82f6', // 기본값
    600: '#2563eb',
    700: '#1d4ed8',
    800: '#1e40af',
    900: '#1e3a8a',
  },

  // Secondary - 보조 액션
  secondary: {
    50: '#f9fafb',
    100: '#f3f4f6',
    200: '#e5e7eb',
    300: '#d1d5db',
    400: '#9ca3af',
    500: '#6b7280',
    600: '#4b5563',
    700: '#374151',
    800: '#1f2937',
    900: '#111827',
  },

  // Success - 성공 메시지
  success: {
    50: '#f0fdf4',
    500: '#22c55e',
    700: '#15803d',
  },

  // Warning - 경고 메시지
  warning: {
    50: '#fffbeb',
    500: '#f59e0b',
    700: '#b45309',
  },

  // Error - 에러 메시지
  error: {
    50: '#fef2f2',
    500: '#ef4444',
    700: '#b91c1c',
  },

  // Neutral - 텍스트, 배경
  neutral: {
    50: '#fafafa',
    100: '#f5f5f5',
    200: '#e5e5e5',
    300: '#d4d4d4',
    400: '#a3a3a3',
    500: '#737373',
    600: '#525252',
    700: '#404040',
    800: '#262626',
    900: '#171717',
    950: '#0a0a0a',
  },
} as const;

export type ColorScale = keyof typeof colors;
export type ColorShade = keyof typeof colors.primary;
```

### 타이포그래피 토큰

```typescript
// src/tokens/typography.ts
export const typography = {
  fontFamily: {
    sans: ['Inter', 'system-ui', 'sans-serif'],
    mono: ['Fira Code', 'monospace'],
  },

  fontSize: {
    xs: ['0.75rem', { lineHeight: '1rem' }],     // 12px
    sm: ['0.875rem', { lineHeight: '1.25rem' }], // 14px
    base: ['1rem', { lineHeight: '1.5rem' }],    // 16px
    lg: ['1.125rem', { lineHeight: '1.75rem' }], // 18px
    xl: ['1.25rem', { lineHeight: '1.75rem' }],  // 20px
    '2xl': ['1.5rem', { lineHeight: '2rem' }],   // 24px
    '3xl': ['1.875rem', { lineHeight: '2.25rem' }], // 30px
    '4xl': ['2.25rem', { lineHeight: '2.5rem' }],   // 36px
  },

  fontWeight: {
    light: '300',
    normal: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
  },

  lineHeight: {
    none: '1',
    tight: '1.25',
    snug: '1.375',
    normal: '1.5',
    relaxed: '1.625',
    loose: '2',
  },

  letterSpacing: {
    tighter: '-0.05em',
    tight: '-0.025em',
    normal: '0em',
    wide: '0.025em',
    wider: '0.05em',
    widest: '0.1em',
  },
} as const;
```

### 스페이싱 토큰

```typescript
// src/tokens/spacing.ts
export const spacing = {
  0: '0px',
  1: '0.25rem',  // 4px
  2: '0.5rem',   // 8px
  3: '0.75rem',  // 12px
  4: '1rem',     // 16px
  5: '1.25rem',  // 20px
  6: '1.5rem',   // 24px
  8: '2rem',     // 32px
  10: '2.5rem',  // 40px
  12: '3rem',    // 48px
  16: '4rem',    // 64px
  20: '5rem',    // 80px
  24: '6rem',    // 96px
} as const;

// 의미론적 스페이싱
export const semanticSpacing = {
  componentGap: spacing[4],      // 컴포넌트 간 간격
  sectionGap: spacing[8],        // 섹션 간 간격
  contentPadding: spacing[6],    // 컨텐츠 패딩
  cardPadding: spacing[4],       // 카드 패딩
} as const;
```

### 반응형 브레이크포인트

```typescript
// src/tokens/breakpoints.ts
export const breakpoints = {
  sm: '640px',   // 모바일
  md: '768px',   // 태블릿
  lg: '1024px',  // 데스크톱
  xl: '1280px',  // 대형 데스크톱
  '2xl': '1536px', // 초대형 화면
} as const;

export type Breakpoint = keyof typeof breakpoints;
```

### 그림자 토큰

```typescript
// src/tokens/shadows.ts
export const shadows = {
  sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
  DEFAULT: '0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)',
  md: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
  lg: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
  xl: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
  '2xl': '0 25px 50px -12px rgb(0 0 0 / 0.25)',
  inner: 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
  none: 'none',
} as const;
```

### 토큰 사용 예시

```tsx
// ✅ 토큰 사용
import { colors } from '@/tokens/colors';
import { spacing } from '@/tokens/spacing';

const Button = styled.button`
  background-color: ${colors.primary[500]};
  padding: ${spacing[4]};
  border-radius: ${spacing[2]};
`;

// ✅ Tailwind로 토큰 사용
<button className="bg-primary-500 p-4 rounded-md">
  Click me
</button>

// ❌ 하드코딩 (토큰 미사용)
<button style={{ backgroundColor: '#3b82f6', padding: '16px' }}>
  Click me
</button>
```

---

## 컴포넌트 라이브러리 구축

### Button 컴포넌트

```tsx
// src/components/Button/Button.tsx
import * as React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';
import { Slot } from '@radix-ui/react-slot';

/**
 * CVA로 버튼 변형 정의
 */
const buttonVariants = cva(
  // 기본 클래스
  'inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary:
          'bg-primary-600 text-white hover:bg-primary-700 active:bg-primary-800',
        secondary:
          'bg-secondary-200 text-secondary-900 hover:bg-secondary-300 active:bg-secondary-400',
        outline:
          'border border-primary-600 text-primary-600 hover:bg-primary-50 active:bg-primary-100',
        ghost:
          'text-primary-600 hover:bg-primary-50 active:bg-primary-100',
        destructive:
          'bg-error-600 text-white hover:bg-error-700 active:bg-error-800',
      },
      size: {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4 text-sm',
        lg: 'h-12 px-6 text-base',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  /**
   * Slot을 사용하여 다른 컴포넌트로 렌더링
   */
  asChild?: boolean;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';

    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);

Button.displayName = 'Button';
```

### Button Storybook

```tsx
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { Mail, ArrowRight } from 'lucide-react';

const meta = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost', 'destructive'],
      description: '버튼의 시각적 스타일',
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg', 'icon'],
      description: '버튼의 크기',
    },
    disabled: {
      control: 'boolean',
      description: '비활성화 상태',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// 기본 버튼
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

// 보조 버튼
export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

// 아웃라인 버튼
export const Outline: Story = {
  args: {
    variant: 'outline',
    children: 'Outline Button',
  },
};

// 고스트 버튼
export const Ghost: Story = {
  args: {
    variant: 'ghost',
    children: 'Ghost Button',
  },
};

// 위험 버튼
export const Destructive: Story = {
  args: {
    variant: 'destructive',
    children: 'Delete',
  },
};

// 크기 변형
export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="md">Medium</Button>
      <Button size="lg">Large</Button>
    </div>
  ),
};

// 아이콘 포함
export const WithIcon: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button>
        <Mail className="h-4 w-4" />
        Send Email
      </Button>
      <Button>
        Continue
        <ArrowRight className="h-4 w-4" />
      </Button>
    </div>
  ),
};

// 아이콘 전용
export const IconOnly: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="icon" variant="primary">
        <Mail className="h-5 w-5" />
      </Button>
      <Button size="icon" variant="secondary">
        <Mail className="h-5 w-5" />
      </Button>
      <Button size="icon" variant="outline">
        <Mail className="h-5 w-5" />
      </Button>
    </div>
  ),
};

// 비활성화
export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

// 로딩 상태
export const Loading: Story = {
  render: () => (
    <Button disabled>
      <svg
        className="animate-spin -ml-1 mr-2 h-4 w-4"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          className="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          strokeWidth="4"
        />
        <path
          className="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        />
      </svg>
      Loading...
    </Button>
  ),
};

// 전체 너비
export const FullWidth: Story = {
  args: {
    className: 'w-full',
    children: 'Full Width Button',
  },
};

// Link로 렌더링
export const AsLink: Story = {
  render: () => (
    <Button asChild>
      <a href="https://example.com" target="_blank" rel="noopener noreferrer">
        Visit Website
      </a>
    </Button>
  ),
};
```

### Input 컴포넌트

```tsx
// src/components/Input/Input.tsx
import * as React from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const inputVariants = cva(
  'flex w-full rounded-md border bg-transparent px-3 py-2 text-sm transition-colors file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-neutral-500 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50',
  {
    variants: {
      variant: {
        default:
          'border-neutral-300 focus-visible:ring-primary-500',
        error:
          'border-error-500 focus-visible:ring-error-500',
        success:
          'border-success-500 focus-visible:ring-success-500',
      },
    },
    defaultVariants: {
      variant: 'default',
    },
  }
);

export interface InputProps
  extends React.InputHTMLAttributes<HTMLInputElement>,
    VariantProps<typeof inputVariants> {
  /**
   * 입력 필드 라벨
   */
  label?: string;
  /**
   * 에러 메시지
   */
  error?: string;
  /**
   * 도움말 텍스트
   */
  helperText?: string;
}

export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ className, variant, label, error, helperText, id, ...props }, ref) => {
    const inputId = id || React.useId();
    const errorId = `${inputId}-error`;
    const helperId = `${inputId}-helper`;

    // 에러가 있으면 variant를 error로 설정
    const finalVariant = error ? 'error' : variant;

    return (
      <div className="w-full space-y-2">
        {label && (
          <label
            htmlFor={inputId}
            className="text-sm font-medium text-neutral-900"
          >
            {label}
            {props.required && (
              <span className="ml-1 text-error-500">*</span>
            )}
          </label>
        )}

        <input
          id={inputId}
          className={cn(inputVariants({ variant: finalVariant, className }))}
          ref={ref}
          aria-invalid={error ? 'true' : 'false'}
          aria-describedby={
            error ? errorId : helperText ? helperId : undefined
          }
          {...props}
        />

        {error && (
          <p id={errorId} className="text-sm text-error-600" role="alert">
            {error}
          </p>
        )}

        {helperText && !error && (
          <p id={helperId} className="text-sm text-neutral-500">
            {helperText}
          </p>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

### Card 컴포넌트

```tsx
// src/components/Card/Card.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';

export interface CardProps extends React.HTMLAttributes<HTMLDivElement> {}

export const Card = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(
          'rounded-lg border border-neutral-200 bg-white shadow-sm',
          className
        )}
        {...props}
      />
    );
  }
);
Card.displayName = 'Card';

export const CardHeader = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn('flex flex-col space-y-1.5 p-6', className)}
        {...props}
      />
    );
  }
);
CardHeader.displayName = 'CardHeader';

export const CardTitle = React.forwardRef<
  HTMLParagraphElement,
  React.HTMLAttributes<HTMLHeadingElement>
>(({ className, ...props }, ref) => {
  return (
    <h3
      ref={ref}
      className={cn(
        'text-2xl font-semibold leading-none tracking-tight',
        className
      )}
      {...props}
    />
  );
});
CardTitle.displayName = 'CardTitle';

export const CardDescription = React.forwardRef<
  HTMLParagraphElement,
  React.HTMLAttributes<HTMLParagraphElement>
>(({ className, ...props }, ref) => {
  return (
    <p
      ref={ref}
      className={cn('text-sm text-neutral-500', className)}
      {...props}
    />
  );
});
CardDescription.displayName = 'CardDescription';

export const CardContent = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, ...props }, ref) => {
    return (
      <div ref={ref} className={cn('p-6 pt-0', className)} {...props} />
    );
  }
);
CardContent.displayName = 'CardContent';

export const CardFooter = React.forwardRef<HTMLDivElement, CardProps>(
  ({ className, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn('flex items-center p-6 pt-0', className)}
        {...props}
      />
    );
  }
);
CardFooter.displayName = 'CardFooter';
```

```tsx
// src/components/Card/Card.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from './Card';
import { Button } from '../Button/Button';

const meta = {
  title: 'Components/Card',
  component: Card,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
} satisfies Meta<typeof Card>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  render: () => (
    <Card className="w-[380px]">
      <CardHeader>
        <CardTitle>Card Title</CardTitle>
        <CardDescription>Card description goes here</CardDescription>
      </CardHeader>
      <CardContent>
        <p>Card content with some meaningful information.</p>
      </CardContent>
      <CardFooter>
        <Button className="w-full">Action</Button>
      </CardFooter>
    </Card>
  ),
};

export const WithForm: Story = {
  render: () => (
    <Card className="w-[380px]">
      <CardHeader>
        <CardTitle>Create Account</CardTitle>
        <CardDescription>
          Enter your information to create an account
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="space-y-2">
          <label className="text-sm font-medium">Email</label>
          <input
            type="email"
            placeholder="name@example.com"
            className="w-full rounded-md border px-3 py-2"
          />
        </div>
        <div className="space-y-2">
          <label className="text-sm font-medium">Password</label>
          <input
            type="password"
            className="w-full rounded-md border px-3 py-2"
          />
        </div>
      </CardContent>
      <CardFooter>
        <Button className="w-full">Create Account</Button>
      </CardFooter>
    </Card>
  ),
};
```

---

## Storybook 문서화

### MDX로 문서 작성

```text
{/* src/components/Button/Button.mdx */}
import { Canvas, Meta, Story, Controls } from '@storybook/blocks';
import * as ButtonStories from './Button.stories';

<Meta of={ButtonStories} />

# Button

재사용 가능한 버튼 컴포넌트입니다.

## 사용법
// Button 컴포넌트 import 및 사용 예제 코드

## 변형 (Variants)
<Canvas of={ButtonStories.Primary} />
<Canvas of={ButtonStories.Secondary} />

## 크기 (Sizes)
<Canvas of={ButtonStories.Sizes} />

## 접근성
- 키보드 탐색 지원 (Tab, Enter, Space)
- 포커스 인디케이터 제공

## Props
<Controls of={ButtonStories.Primary} />
```

### Link로 사용

```tsx
<Button asChild>
  <a href="/about">Learn More</a>
</Button>
```

### 커스텀 스타일

```tsx
<Button className="rounded-full">Rounded Button</Button>
```

### 인터랙티브 Docs

```tsx
// src/Introduction.stories.tsx
import type { Meta } from '@storybook/react';

const meta = {
  title: 'Introduction',
  parameters: {
    layout: 'fullscreen',
  },
} satisfies Meta;

export default meta;

export const Welcome = () => (
  <div className="p-10">
    <h1 className="text-4xl font-bold mb-4">Welcome to Design System</h1>

    <p className="text-lg mb-8">
      확장 가능하고 접근 가능한 React 컴포넌트 라이브러리입니다.
    </p>

    <h2 className="text-2xl font-semibold mb-4">시작하기</h2>

    <div className="space-y-4">
      <section>
        <h3 className="text-xl font-medium mb-2">설치</h3>
        <pre className="bg-neutral-100 p-4 rounded-md">
          npm install @your-org/design-system
        </pre>
      </section>

      <section>
        <h3 className="text-xl font-medium mb-2">사용법</h3>
        <pre className="bg-neutral-100 p-4 rounded-md">
{`import { Button, Input, Card } from '@your-org/design-system';

function App() {
  return (
    <Card>
      <Input label="Name" />
      <Button>Submit</Button>
    </Card>
  );
}`}
        </pre>
      </section>

      <section>
        <h3 className="text-xl font-medium mb-2">컴포넌트</h3>
        <ul className="list-disc list-inside space-y-1">
          <li>Button - 다양한 변형의 버튼</li>
          <li>Input - 폼 입력 필드</li>
          <li>Card - 컨텐츠 컨테이너</li>
          <li>그 외 더 많은 컴포넌트...</li>
        </ul>
      </section>
    </div>
  </div>
);
```

### 테마 전환 애드온

```tsx
// .storybook/preview.tsx
import { useEffect } from 'react';

export const globalTypes = {
  theme: {
    name: 'Theme',
    description: 'Global theme for components',
    defaultValue: 'light',
    toolbar: {
      icon: 'circlehollow',
      items: ['light', 'dark'],
      dynamicTitle: true,
    },
  },
};

const withTheme = (Story, context) => {
  const { theme } = context.globals;

  useEffect(() => {
    const root = document.documentElement;

    if (theme === 'dark') {
      root.classList.add('dark');
    } else {
      root.classList.remove('dark');
    }
  }, [theme]);

  return <Story />;
};

export const decorators = [withTheme];
```

---

## 접근성 고려사항

### ARIA 속성

```tsx
// src/components/Button/Button.tsx
export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        disabled={disabled}
        // 비활성화 상태를 스크린 리더에게 전달
        aria-disabled={disabled}
        // 로딩 상태 표시
        aria-busy={props['aria-busy']}
        {...props}
      >
        {children}
      </button>
    );
  }
);
```

### 키보드 탐색

```tsx
// src/components/Dialog/Dialog.tsx
import * as DialogPrimitive from '@radix-ui/react-dialog';

export const Dialog = ({ children, ...props }: DialogProps) => {
  return (
    <DialogPrimitive.Root {...props}>
      {children}
    </DialogPrimitive.Root>
  );
};

// Radix UI는 자동으로 처리:
// - Esc 키로 닫기
// - Tab 키로 포커스 트랩
// - Enter/Space로 트리거
```

### 포커스 관리

```tsx
// src/components/Input/Input.tsx
export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ error, ...props }, ref) => {
    return (
      <input
        ref={ref}
        // 에러 상태를 스크린 리더에게 전달
        aria-invalid={error ? 'true' : 'false'}
        // 에러 메시지 연결
        aria-describedby={error ? `${props.id}-error` : undefined}
        // 포커스 인디케이터
        className="focus-visible:ring-2 focus-visible:ring-primary-500"
        {...props}
      />
    );
  }
);
```

### 색상 대비

```typescript
// src/tokens/colors.ts
// WCAG AA 기준 (4.5:1 이상) 충족하는 색상
export const accessibleColors = {
  // 흰 배경에서 읽기 좋은 텍스트
  textOnLight: {
    primary: '#1e3a8a',   // 대비 10.8:1
    secondary: '#374151', // 대비 9.2:1
  },

  // 어두운 배경에서 읽기 좋은 텍스트
  textOnDark: {
    primary: '#dbeafe',   // 대비 11.2:1
    secondary: '#e5e7eb', // 대비 12.1:1
  },
};
```

### Storybook 접근성 테스트

```tsx
// Button.stories.tsx
export const AccessibilityTest: Story = {
  parameters: {
    a11y: {
      // Axe 접근성 테스트 설정
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: true,
          },
          {
            id: 'button-name',
            enabled: true,
          },
        ],
      },
    },
  },
  render: () => (
    <div className="space-y-4">
      <Button>Accessible Button</Button>
      <Button disabled>Disabled Button</Button>
      <Button aria-label="Close">✕</Button>
    </div>
  ),
};
```

---

## 버전 관리 및 배포

### Semantic Versioning

```jsonc
// package.json
{
  "name": "@your-org/design-system",
  "version": "1.2.3",
  "description": "Design system component library",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    },
    "./package.json": "./package.json"
  },
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch",
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    "release": "changeset publish"
  },
  "peerDependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@changesets/cli": "^2.27.0",
    "tsup": "^8.0.0"
  }
}
```

### 빌드 설정 (tsup)

```typescript
// tsup.config.ts
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'], // CommonJS와 ESM 모두 지원
  dts: true, // TypeScript 선언 파일 생성
  splitting: true,
  sourcemap: true,
  clean: true,
  external: ['react', 'react-dom'], // 번들에 포함하지 않음
  treeshake: true, // 사용하지 않는 코드 제거
});
```

### Changesets 설정

```bash
# Changesets 초기화
npx changeset init
```

```jsonc
// .changeset/config.json
{
  "$schema": "https://unpkg.com/@changesets/config@2.3.0/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

### 변경사항 기록

```bash
# 변경사항 추가
npx changeset

# 대화형 프롬프트:
# 1. 변경 타입 선택 (patch/minor/major)
# 2. 변경 내용 작성

# 버전 업데이트
npx changeset version

# 배포
npx changeset publish
```

### GitHub Actions CI/CD

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: npm run release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Storybook 배포

```yaml
# .github/workflows/storybook.yml
name: Deploy Storybook

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Build Storybook
        run: npm run build-storybook

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./storybook-static
```

---

## 실무 베스트 프랙티스

### 1. 컴포넌트 API 설계

```tsx
// ✅ Good: 명확하고 일관된 API
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

// ❌ Bad: 모호하거나 일관성 없는 API
interface BadButtonProps {
  type?: string; // 모호함
  big?: boolean; // size와 혼동
  isDisabled?: boolean; // disabled와 혼동
  text?: string; // children과 혼동
}
```

### 2. 합성 패턴 (Composition Pattern)

```tsx
// ✅ Good: 유연한 합성 패턴
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>Content</CardContent>
  <CardFooter>Footer</CardFooter>
</Card>

// ❌ Bad: 모든 것을 props로 전달
<Card
  title="Title"
  description="Description"
  content="Content"
  footer="Footer"
/>
```

### 3. Polymorphic 컴포넌트

```tsx
// as prop으로 다른 요소로 렌더링
<Button as="a" href="/about">
  Link Button
</Button>

<Button as={Link} to="/dashboard">
  React Router Link
</Button>

// 또는 asChild 패턴 (Radix UI 스타일)
<Button asChild>
  <a href="/about">Link Button</a>
</Button>
```

### 4. Controlled vs Uncontrolled

```tsx
// Controlled: 외부에서 상태 관리
function ControlledExample() {
  const [value, setValue] = useState('');

  return (
    <Input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Uncontrolled: 내부에서 상태 관리
function UncontrolledExample() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <Input
      ref={inputRef}
      defaultValue="initial"
    />
  );
}
```

### 5. 포워딩 Refs

```tsx
// ✅ Good: forwardRef로 ref 전달 가능
export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  (props, ref) => {
    return <input ref={ref} {...props} />;
  }
);

// 사용처에서 ref 접근
function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <Input ref={inputRef} />;
}
```

### 6. 타입 안전성

```typescript
// ✅ Good: 엄격한 타입 정의
type ButtonVariant = 'primary' | 'secondary' | 'outline';
type ButtonSize = 'sm' | 'md' | 'lg';

interface ButtonProps {
  variant: ButtonVariant;
  size: ButtonSize;
  children: React.ReactNode;
}

// ❌ Bad: 느슨한 타입
interface BadButtonProps {
  variant?: string;
  size?: any;
  children?: any;
}
```

### 7. 성능 최적화

```tsx
// ✅ Good: React.memo로 불필요한 리렌더링 방지
export const Button = React.memo(
  React.forwardRef<HTMLButtonElement, ButtonProps>(
    ({ children, ...props }, ref) => {
      return (
        <button ref={ref} {...props}>
          {children}
        </button>
      );
    }
  )
);

// props 비교 함수 커스터마이징
export const ComplexComponent = React.memo(
  Component,
  (prevProps, nextProps) => {
    // true를 반환하면 리렌더링 스킵
    return prevProps.id === nextProps.id;
  }
);
```

### 8. 에러 처리

```tsx
// ✅ Good: 명확한 에러 메시지와 fallback
export const Input = ({ value, onChange, ...props }: InputProps) => {
  if (value !== undefined && onChange === undefined) {
    console.warn(
      'Input: value prop을 사용하려면 onChange도 제공해야 합니다.'
    );
  }

  return <input value={value} onChange={onChange} {...props} />;
};
```

### 9. 테스트 용이성

```tsx
// ✅ Good: data-testid 제공
export const Button = ({ children, ...props }: ButtonProps) => {
  return (
    <button data-testid="button" {...props}>
      {children}
    </button>
  );
};

// 테스트
test('Button renders correctly', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByTestId('button')).toHaveTextContent('Click me');
});
```

### 10. 문서화

```tsx
/**
 * 재사용 가능한 버튼 컴포넌트
 *
 * @example
 * ```tsx
 * <Button variant="primary" size="md">
 *   Click me
 * </Button>
 * ```
 *
 * @see {@link https://storybook.com} Storybook 문서
 */
export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', ...props }, ref) => {
    return <button ref={ref} {...props} />;
  }
);
```

---

## 자주 묻는 질문 (FAQ)

### Q1. 디자인 시스템을 처음부터 만들어야 하나요?

**A:** 프로젝트 규모와 요구사항에 따라 다릅니다:

**처음부터 만드는 경우:**
- 브랜드 정체성이 강함
- 독특한 디자인 언어 필요
- 완전한 커스터마이징 필요

**기존 시스템 활용:**
- 빠른 개발 필요
- 일반적인 UI 패턴
- 리소스 제한적

**추천 접근법:**
1. shadcn/ui로 시작 (copy-paste)
2. 브랜드에 맞게 커스터마이징
3. 점진적으로 자체 컴포넌트 추가

---

### Q2. Tailwind vs CSS-in-JS, 어떤 것을 선택해야 하나요?

**A:** 각각의 장단점:

**Tailwind CSS:**
- ✅ 빠른 개발 속도
- ✅ 작은 번들 크기
- ✅ 일관된 디자인
- ❌ 클래스 이름이 길어짐
- ❌ 동적 스타일 제한적

**CSS-in-JS (Styled Components, Emotion):**
- ✅ 완전한 JavaScript 제어
- ✅ 동적 스타일 쉬움
- ✅ 타입 안전성
- ❌ 런타임 오버헤드
- ❌ 번들 크기 증가

**추천:** 대부분의 경우 Tailwind + CVA 조합이 최적

---

### Q3. 컴포넌트를 언제 만들어야 하나요?

**A:** 다음 기준을 따르세요:

**컴포넌트 만들기:**
- 3번 이상 반복 (DRY 원칙)
- 독립적인 기능
- 재사용 가능성
- 테스트 필요

**만들지 않기:**
- 한 번만 사용
- 너무 구체적
- 오버 엔지니어링

---

### Q4. 디자인 토큰을 어떻게 관리하나요?

**A:** 계층적 구조로 관리:

```typescript
// 1. 원시 토큰 (Raw Tokens)
const rawColors = {
  blue500: '#3b82f6',
  gray500: '#6b7280',
};

// 2. 의미론적 토큰 (Semantic Tokens)
const semanticColors = {
  primary: rawColors.blue500,
  textSecondary: rawColors.gray500,
};

// 3. 컴포넌트 토큰 (Component Tokens)
const buttonColors = {
  primaryBg: semanticColors.primary,
  primaryText: 'white',
};
```

---

### Q5. Storybook이 너무 느린데 어떻게 하나요?

**A:** 성능 최적화 방법:

```javascript
// 1. Vite 사용 (Webpack보다 빠름)
// .storybook/main.ts
framework: '@storybook/react-vite'

// 2. 특정 스토리만 로드
stories: [
  '../src/components/**/*.stories.tsx',
  // '../src/**/*.stories.tsx' 대신
]

// 3. 애드온 최소화
addons: [
  '@storybook/addon-essentials', // 필수만
]
```

---

### Q6. 접근성을 어떻게 보장하나요?

**A:** 단계별 접근:

**1. Radix UI 같은 Headless 컴포넌트 사용**
```tsx
import * as Dialog from '@radix-ui/react-dialog';
// 접근성이 이미 구현됨
```

**2. Storybook 접근성 애드온**
```bash
npm install @storybook/addon-a11y
```

**3. 자동 테스트**
```typescript
// playwright-axe로 접근성 테스트
await expect(page).toHaveNoViolations();
```

**4. 수동 테스트**
- 키보드만으로 탐색
- 스크린 리더 테스트
- 색상 대비 확인

---

### Q7. 버전 업그레이드 시 Breaking Change는 어떻게 관리하나요?

**A:** Semantic Versioning 준수:

```bash
# Patch (1.0.0 → 1.0.1): 버그 수정
- 기존 API 변경 없음
- 버그 수정만

# Minor (1.0.0 → 1.1.0): 새 기능 추가
- 기존 API 호환
- 새로운 기능/컴포넌트 추가

# Major (1.0.0 → 2.0.0): Breaking Change
- 기존 API 변경
- 마이그레이션 가이드 제공
```

**마이그레이션 가이드 작성:**
```markdown
# v2.0.0 마이그레이션 가이드

## Breaking Changes

### Button 컴포넌트
**변경 전:**
```tsx
<Button type="primary">Click</Button>
```

**변경 후:**
```tsx
<Button variant="primary">Click</Button>
```

**이유:** 네이티브 `type` 속성과 충돌 방지
```

---

## 결론

디자인 시스템은 팀의 생산성과 제품의 일관성을 크게 향상시키는 투자입니다.

### 핵심 요점

1. **디자인 토큰**: 색상, 타이포그래피, 스페이싱을 중앙에서 관리
2. **Tailwind + CVA**: 빠르고 유지보수 쉬운 스타일링
3. **Storybook**: 컴포넌트 문서화와 개발 환경
4. **접근성**: Radix UI로 기본부터 접근 가능하게
5. **버전 관리**: Changesets로 체계적인 릴리스

### 시작 체크리스트

**Phase 1: 기초 (1-2주)**
- [ ] 프로젝트 셋업 (React + TypeScript + Tailwind)
- [ ] Storybook 설치 및 설정
- [ ] 디자인 토큰 정의 (색상, 타이포그래피)
- [ ] 기본 컴포넌트 3개 (Button, Input, Card)

**Phase 2: 확장 (2-4주)**
- [ ] 더 많은 컴포넌트 추가
- [ ] MDX로 문서화
- [ ] 접근성 테스트 (a11y 애드온)
- [ ] 다크모드 지원

**Phase 3: 배포 (1-2주)**
- [ ] npm 패키지로 빌드
- [ ] Storybook 배포 (GitHub Pages)
- [ ] CI/CD 설정
- [ ] 버전 관리 (Changesets)

**Phase 4: 운영**
- [ ] 팀 교육 및 온보딩
- [ ] 피드백 수집 및 개선
- [ ] 정기적인 업데이트
- [ ] 커뮤니티 구축

### 다음 단계

디자인 시스템을 구축한 후:

1. **[시각적 회귀 테스트 가이드](/posts/visual-regression-testing-guide/)**로 UI 변경 자동 감지
2. **[React 상태 관리 가이드](/posts/react-state-management-guide/)**로 복잡한 컴포넌트 로직 관리
3. **[프론트엔드 테스팅 가이드](/posts/frontend-testing-guide/)**로 컴포넌트 테스트 작성

이제 확장 가능하고 일관된 디자인 시스템으로 제품을 빠르게 개발할 수 있습니다!

---

## 참고 자료

### 공식 문서
- [Storybook Documentation](https://storybook.js.org/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Radix UI Documentation](https://www.radix-ui.com/)
- [CVA (Class Variance Authority)](https://cva.style/docs)

### 디자인 시스템 예시
- [shadcn/ui](https://ui.shadcn.com/) - Copy-paste 컴포넌트 라이브러리
- [Chakra UI](https://chakra-ui.com/) - 접근성 중심 React 컴포넌트
- [Material UI](https://mui.com/) - Google Material Design
- [Ant Design](https://ant.design/) - 엔터프라이즈 디자인 시스템
- [Atlassian Design System](https://atlassian.design/) - 실무 디자인 시스템 사례

### 도구
- [Figma](https://www.figma.com/) - 디자인 협업 도구
- [Zeroheight](https://zeroheight.com/) - 디자인 시스템 문서화
- [Chromatic](https://www.chromatic.com/) - 시각적 테스트 및 리뷰
- [Percy](https://percy.io/) - 시각적 회귀 테스트

### 학습 자료
- [Design Systems Handbook](https://www.designbetter.co/design-systems-handbook) - 디자인 시스템 기초
- [Atomic Design](https://atomicdesign.bradfrost.com/) - 컴포넌트 구조 설계
- [The Component Gallery](https://component.gallery/) - 컴포넌트 패턴 모음
- [Accessible Components](https://web.dev/patterns/components/) - 접근 가능한 컴포넌트 패턴

### 관련 글
- [프론트엔드 테스팅 완벽 가이드](/posts/frontend-testing-guide/)
- [시각적 회귀 테스트 가이드](/posts/visual-regression-testing-guide/)
- [React 상태 관리 가이드](/posts/react-state-management-guide/)

### 커뮤니티
- [Storybook Discord](https://discord.gg/storybook)
- [Tailwind CSS Discord](https://tailwindcss.com/discord)
- [Radix UI Discord](https://discord.com/invite/7Xb99uG)
- [Design Systems Slack](https://design.systems/slack/)
