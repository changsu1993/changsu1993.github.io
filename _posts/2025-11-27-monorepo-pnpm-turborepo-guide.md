---
title: "Monorepo 구축 완벽 가이드 - pnpm + Turborepo로 대규모 프로젝트 관리하기"
description: "pnpm workspace와 Turborepo를 활용한 효율적인 Monorepo 구축 방법을 실전 예제와 함께 상세히 알아봅니다."
date: 2025-11-27 10:00:00 +0900
categories: [Frontend, DevOps]
tags: [monorepo, pnpm, turborepo, workspace, build-optimization]
---

대규모 프로젝트를 관리하다 보면 여러 패키지 간의 의존성, 중복 코드, 일관성 없는 빌드 프로세스로 인해 어려움을 겪게 됩니다. Monorepo는 이러한 문제를 해결하기 위한 강력한 솔루션입니다. 이 글에서는 pnpm과 Turborepo를 활용하여 효율적인 Monorepo를 구축하는 방법을 상세히 알아보겠습니다.

## Monorepo란?

Monorepo(Monolithic Repository)는 여러 프로젝트나 패키지를 하나의 저장소에서 관리하는 소프트웨어 개발 전략입니다. Google, Facebook, Microsoft 등 많은 대형 기업들이 채택하고 있는 방식입니다.

### Monorepo vs Polyrepo

**Polyrepo (전통적 방식)**
```bash
my-company/
├── frontend-app/       # 별도 저장소
├── backend-api/        # 별도 저장소
├── shared-ui/          # 별도 저장소
└── shared-utils/       # 별도 저장소
```

**Monorepo**
```bash
my-company/
├── apps/
│   ├── frontend/
│   └── backend/
├── packages/
│   ├── ui/
│   └── utils/
└── package.json
```

### Monorepo의 장점

**1. 코드 공유 용이성**
- 공통 컴포넌트, 유틸리티, 타입 정의를 쉽게 공유
- import 경로 간소화: `import { Button } from '@company/ui'`
- 버전 불일치 문제 해결

**2. 원자적 커밋 (Atomic Commits)**
- 여러 패키지의 변경사항을 하나의 커밋으로 관리
- API 변경 시 모든 관련 코드를 동시에 업데이트
- 코드 리뷰가 더 명확해짐

**3. 개발 환경 일관성**
- 통일된 린트, 포맷터, 테스트 설정
- 단일 CI/CD 파이프라인
- 개발자 온보딩 시간 단축

**4. 대규모 리팩토링 용이**
- 전체 코드베이스를 한 번에 검색 및 수정
- 자동화된 마이그레이션 스크립트 실행 가능
- 영향 범위 파악 용이

### Monorepo의 단점

**1. 저장소 크기**
- 클론 시간 증가 (Git shallow clone으로 완화 가능)
- 디스크 공간 필요

**2. 접근 권한 관리**
- 세밀한 권한 설정 어려움
- CODEOWNERS 파일로 부분적 해결 가능

**3. 빌드 시간**
- 전체 빌드 시간 증가 (캐싱으로 해결 가능)
- 적절한 도구 없이는 비효율적

### 언제 Monorepo를 선택해야 하는가?

**Monorepo가 적합한 경우:**
- 여러 프로젝트가 코드를 공유하는 경우
- 팀 간 협업이 빈번한 경우
- 일관된 개발 환경이 중요한 경우
- 마이크로 프론트엔드 아키텍처
- 디자인 시스템과 여러 앱을 함께 관리

**Polyrepo가 적합한 경우:**
- 완전히 독립적인 프로젝트
- 각 팀이 완전히 다른 기술 스택 사용
- 외부 공개가 필요한 오픈소스 프로젝트
- 매우 작은 팀 (2-3명)

## 도구 선택: pnpm + Turborepo

### pnpm이 npm/yarn보다 좋은 이유

**1. 디스크 공간 효율성**

pnpm은 content-addressable storage를 사용하여 의존성을 전역 저장소에 한 번만 저장합니다.

```bash
# 일반적인 npm/yarn
~/projects/
├── project1/node_modules/react (100MB)
├── project2/node_modules/react (100MB)
└── project3/node_modules/react (100MB)
# 총 300MB 사용

# pnpm
~/.pnpm-store/
└── react@18.2.0 (100MB)
~/projects/
├── project1/node_modules/react -> symlink
├── project2/node_modules/react -> symlink
└── project3/node_modules/react -> symlink
# 총 100MB 사용 (70% 절약)
```

**2. 엄격한 의존성 관리**

pnpm은 `package.json`에 명시되지 않은 패키지에 접근할 수 없도록 합니다.

```tsx
// npm/yarn: 동작함 (잘못된 동작)
import _ from 'lodash'; // package.json에 없어도 부모가 설치했다면 동작

// pnpm: 에러 발생 (올바른 동작)
// Error: Cannot find module 'lodash'
```

**3. workspace 프로토콜**

```json
{
  "dependencies": {
    "@company/ui": "workspace:*",
    "@company/utils": "workspace:^"
  }
}
```

**4. 성능 비교**

실제 벤치마크 (React 프로젝트 기준):

| 작업 | npm | yarn | pnpm |
|------|-----|------|------|
| 첫 설치 | 51초 | 39초 | 24초 |
| 캐시 있는 설치 | 27초 | 13초 | 7초 |
| 디스크 공간 | 178MB | 165MB | 105MB |

### Turborepo의 핵심 기능

**1. 스마트 캐싱**

Turborepo는 태스크 입력(소스 코드, 의존성)을 기반으로 해시를 생성하고, 결과를 캐시합니다.

```bash
# 첫 번째 실행
turbo run build
# >>> FULL TURBO (43.2초)

# 변경사항 없이 다시 실행
turbo run build
# >>> CACHED (0.1초)
```

**2. 병렬 실행**

```bash
# 일반적인 실행 (순차적)
npm run build  # packages/ui: 10초
npm run build  # packages/utils: 8초
npm run build  # apps/web: 15초
# 총 33초

# Turborepo (병렬)
turbo run build
# packages/ui + packages/utils 동시 실행: 10초
# apps/web (의존성 대기 후 실행): 15초
# 총 25초
```

**3. 의존성 기반 실행 순서**

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"]
    }
  }
}
```

`^build`는 "이 패키지를 빌드하기 전에 의존하는 모든 패키지를 먼저 빌드하라"는 의미입니다.

### Nx vs Turborepo 비교

| 항목 | Turborepo | Nx |
|------|-----------|-----|
| **학습 곡선** | 낮음 | 중간 |
| **설정 복잡도** | 간단 | 복잡 |
| **캐싱** | 로컬/리모트 | 로컬/리모트 |
| **플러그인 생태계** | 제한적 | 풍부 |
| **프레임워크 통합** | 프레임워크 독립적 | Angular, React 등 내장 |
| **마이그레이션** | 쉬움 | 어려움 |
| **성능** | 매우 빠름 | 빠름 |
| **적합한 경우** | 단순하고 빠른 설정 | 복잡한 엔터프라이즈 |

**선택 가이드:**
- 빠르게 시작하고 싶다면: **Turborepo**
- Angular 프로젝트: **Nx**
- 복잡한 빌드 파이프라인: **Nx**
- Vercel 에코시스템: **Turborepo**

## 실전 프로젝트 구축

실제 동작하는 Monorepo를 처음부터 구축해보겠습니다.

### 프로젝트 초기 설정

**1. 프로젝트 생성**

```bash
# 디렉토리 생성
mkdir my-monorepo
cd my-monorepo

# pnpm 초기화
pnpm init

# Turborepo 설치
pnpm add -Dw turbo

# Git 초기화
git init
```

**2. 디렉토리 구조 생성**

```bash
mkdir -p apps/web apps/docs
mkdir -p packages/ui packages/utils packages/tsconfig
```

최종 구조:

```bash
my-monorepo/
├── apps/
│   ├── web/              # Next.js 앱
│   └── docs/             # 문서 사이트
├── packages/
│   ├── ui/               # 공통 UI 컴포넌트
│   ├── utils/            # 유틸리티 함수
│   └── tsconfig/         # 공유 TypeScript 설정
├── turbo.json
├── package.json
└── pnpm-workspace.yaml
```

### pnpm workspace 구성

**pnpm-workspace.yaml 생성**

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**루트 package.json 설정**

```json
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "turbo": "^2.3.1",
    "prettier": "^3.3.3",
    "typescript": "^5.6.3"
  },
  "packageManager": "pnpm@9.14.2"
}
```

### Turborepo 설정

**turbo.json 생성**

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    }
  }
}
```

**주요 설정 설명:**

- `dependsOn: ["^build"]`: 의존하는 패키지를 먼저 빌드
- `outputs`: 캐시할 결과물 지정
- `cache: false`: 개발 서버는 캐시하지 않음
- `persistent: true`: 백그라운드에서 계속 실행

### 패키지 구조 설계

**1. Web 앱 (apps/web)**

```bash
cd apps/web
pnpm create next-app@latest . --typescript --eslint --tailwind --app
```

**apps/web/package.json:**

```json
{
  "name": "@my-monorepo/web",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "@my-monorepo/ui": "workspace:*",
    "@my-monorepo/utils": "workspace:*"
  },
  "devDependencies": {
    "@my-monorepo/tsconfig": "workspace:*",
    "typescript": "^5.6.3"
  }
}
```

**2. UI 패키지 (packages/ui)**

**packages/ui/package.json:**

```json
{
  "name": "@my-monorepo/ui",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "lint": "eslint src/"
  },
  "devDependencies": {
    "@my-monorepo/tsconfig": "workspace:*",
    "tsup": "^8.3.5",
    "typescript": "^5.6.3",
    "react": "^19.0.0"
  },
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0"
  }
}
```

**packages/ui/tsconfig.json:**

```json
{
  "extends": "@my-monorepo/tsconfig/react-library.json",
  "compilerOptions": {
    "outDir": "dist"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**3. Utils 패키지 (packages/utils)**

**packages/utils/package.json:**

```json
{
  "name": "@my-monorepo/utils",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "test": "vitest",
    "lint": "eslint src/"
  },
  "devDependencies": {
    "@my-monorepo/tsconfig": "workspace:*",
    "tsup": "^8.3.5",
    "typescript": "^5.6.3",
    "vitest": "^2.1.8"
  }
}
```

## 공유 패키지 만들기

### 공통 UI 컴포넌트 패키지

**packages/ui/src/Button.tsx:**

```tsx
import React from 'react';

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
}

const variantStyles = {
  primary: 'bg-blue-600 hover:bg-blue-700 text-white',
  secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-900',
  danger: 'bg-red-600 hover:bg-red-700 text-white',
};

const sizeStyles = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
};

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  className = '',
  ...props
}: ButtonProps) {
  return (
    <button
      className={`
        rounded-md font-medium transition-colors
        ${variantStyles[variant]}
        ${sizeStyles[size]}
        ${className}
      `}
      {...props}
    >
      {children}
    </button>
  );
}
```

**packages/ui/src/index.ts:**

```typescript
export { Button, type ButtonProps } from './Button';
```

**실제 사용 (apps/web/app/page.tsx):**

```tsx
import { Button } from '@my-monorepo/ui';

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <h1 className="text-4xl font-bold mb-8">My Monorepo</h1>
      <div className="flex gap-4">
        <Button variant="primary">Primary Button</Button>
        <Button variant="secondary">Secondary Button</Button>
        <Button variant="danger" size="lg">
          Large Danger Button
        </Button>
      </div>
    </main>
  );
}
```

### 공통 설정 패키지

**1. TypeScript 설정 (packages/tsconfig)**

**packages/tsconfig/package.json:**

```json
{
  "name": "@my-monorepo/tsconfig",
  "version": "1.0.0",
  "private": true,
  "files": ["base.json", "nextjs.json", "react-library.json"]
}
```

**packages/tsconfig/base.json:**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "moduleResolution": "Bundler",
    "resolveJsonModule": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  }
}
```

**packages/tsconfig/react-library.json:**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "target": "ES2022",
    "module": "ESNext",
    "jsx": "react-jsx"
  }
}
```

**packages/tsconfig/nextjs.json:**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "target": "ES2022",
    "module": "ESNext",
    "jsx": "preserve",
    "plugins": [
      {
        "name": "next"
      }
    ],
    "incremental": true
  },
  "include": ["src", "next-env.d.ts"],
  "exclude": ["node_modules"]
}
```

**2. ESLint 설정 (packages/eslint-config)**

**packages/eslint-config/package.json:**

```json
{
  "name": "@my-monorepo/eslint-config",
  "version": "1.0.0",
  "private": true,
  "main": "index.js",
  "files": ["index.js", "next.js", "react.js"],
  "dependencies": {
    "eslint-config-next": "^15.0.0",
    "eslint-config-prettier": "^9.1.0"
  }
}
```

**packages/eslint-config/next.js:**

```javascript
module.exports = {
  extends: ['next/core-web-vitals', 'prettier'],
  rules: {
    '@next/next/no-html-link-for-pages': 'off',
    'react/jsx-key': 'warn',
  },
};
```

### 공통 유틸리티 패키지

**packages/utils/src/format.ts:**

```typescript
/**
 * 숫자를 천 단위 구분 기호로 포맷팅합니다.
 * @example
 * formatNumber(1234567) // "1,234,567"
 */
export function formatNumber(num: number): string {
  return new Intl.NumberFormat('ko-KR').format(num);
}

/**
 * 날짜를 한국 형식으로 포맷팅합니다.
 * @example
 * formatDate(new Date()) // "2025년 11월 27일"
 */
export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('ko-KR', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}
```

**packages/utils/src/validation.ts:**

```typescript
/**
 * 이메일 주소 유효성을 검증합니다.
 */
export function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * 한국 휴대폰 번호 유효성을 검증합니다.
 * @example
 * isValidPhoneNumber("010-1234-5678") // true
 */
export function isValidPhoneNumber(phone: string): boolean {
  const phoneRegex = /^01[0-9]-\d{3,4}-\d{4}$/;
  return phoneRegex.test(phone);
}
```

**packages/utils/src/index.ts:**

```typescript
export { formatNumber, formatDate } from './format';
export { isValidEmail, isValidPhoneNumber } from './validation';
```

**테스트 작성 (packages/utils/src/format.test.ts):**

```typescript
import { describe, it, expect } from 'vitest';
import { formatNumber, formatDate } from './format';

describe('formatNumber', () => {
  it('천 단위 구분 기호를 추가해야 함', () => {
    expect(formatNumber(1234567)).toBe('1,234,567');
  });

  it('0을 올바르게 포맷팅해야 함', () => {
    expect(formatNumber(0)).toBe('0');
  });
});

describe('formatDate', () => {
  it('한국 형식으로 날짜를 포맷팅해야 함', () => {
    const date = new Date('2025-11-27');
    const formatted = formatDate(date);
    expect(formatted).toContain('2025년');
    expect(formatted).toContain('11월');
    expect(formatted).toContain('27일');
  });
});
```

## 빌드 & 개발 최적화

### Turborepo 캐싱 전략

**캐싱 작동 원리**

Turborepo는 다음을 기반으로 해시를 생성합니다:

1. **태스크 입력**
   - 소스 코드 파일 내용
   - 의존하는 패키지의 출력물
   - 환경 변수 (globalDependencies에 지정된 것)
   - turbo.json의 태스크 설정

2. **캐시 키 생성**

```bash
# 해시 계산 예시
hash(
  "packages/ui/src/**/*.{ts,tsx}" +
  "packages/ui/package.json" +
  "packages/tsconfig/**/*" +
  "turbo.json[pipeline.build]"
) = "a1b2c3d4"
```

3. **캐시 적중/미스**

```bash
# 첫 번째 빌드
turbo run build
# packages/ui:build: cache miss, executing...
# packages/ui:build: 10.2s

# 변경 없이 재실행
turbo run build
# packages/ui:build: cache hit, replaying...
# packages/ui:build: 0.1s (cached)
```

**캐싱 최적화 전략**

**1. 입력 범위 좁히기**

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": [
        "src/**/*.{ts,tsx}",
        "!src/**/*.test.{ts,tsx}",
        "!src/**/*.stories.{ts,tsx}"
      ],
      "outputs": ["dist/**"]
    }
  }
}
```

**2. 환경 변수 관리**

```json
{
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "env": ["NODE_ENV", "API_URL"],
      "outputs": ["dist/**"]
    }
  }
}
```

**3. 캐시 무효화 트리거**

```bash
# 특정 파일 변경 시에만 캐시 무효화
{
  "pipeline": {
    "build": {
      "inputs": [
        "$TURBO_DEFAULT$",
        ".env.production"
      ]
    }
  }
}
```

### Remote Caching (Vercel)

로컬 캐시는 개발자 개인에게만 유용합니다. Remote Caching을 통해 팀 전체가 캐시를 공유할 수 있습니다.

**1. Vercel Remote Cache 설정**

```bash
# Turborepo 계정 연결
npx turbo login

# 프로젝트 링크
npx turbo link
```

**2. 환경 변수 설정**

```bash
# .env.local
TURBO_TOKEN=your_token_here
TURBO_TEAM=your_team_slug
```

**3. CI/CD에서 Remote Cache 활용**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9.14.2

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm turbo build
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

      - name: Test
        run: pnpm turbo test
```

**Remote Cache 효과**

| 상황 | 로컬 캐시만 | Remote Cache |
|------|-----------|--------------|
| 본인 재빌드 | ✅ 0.1초 | ✅ 0.1초 |
| 다른 개발자 빌드 | ❌ 43초 | ✅ 2초 (다운로드) |
| CI 빌드 | ❌ 43초 | ✅ 2초 (다운로드) |

### 의존성 그래프 이해

**그래프 시각화**

```bash
# 의존성 그래프 생성
npx turbo run build --graph=graph.html
```

예시 그래프:

```
packages/tsconfig
       ↓
packages/utils ─────→ packages/ui
       ↓                    ↓
       └────────────────────┴───→ apps/web
```

**병렬 실행 분석**

```bash
# --dry 플래그로 실행 계획 확인
npx turbo run build --dry=json

# 출력:
{
  "tasks": [
    {
      "task": "packages/tsconfig#build",
      "dependencies": [],
      "duration": 0
    },
    {
      "task": "packages/utils#build",
      "dependencies": ["packages/tsconfig#build"],
      "duration": 8234
    },
    {
      "task": "packages/ui#build",
      "dependencies": ["packages/tsconfig#build"],
      "duration": 10142
    },
    {
      "task": "apps/web#build",
      "dependencies": ["packages/utils#build", "packages/ui#build"],
      "duration": 15890
    }
  ]
}
```

**최적화 포인트:**

1. **불필요한 의존성 제거**: `packages/ui`가 `packages/utils`를 사용하지 않는다면 의존성 제거
2. **태스크 분리**: 무거운 태스크를 여러 단계로 분리하여 병렬화
3. **Lazy Loading**: 모든 패키지를 한 번에 빌드하지 말고 필요한 것만 빌드

## CI/CD 파이프라인

### GitHub Actions와 Turborepo 연동

**변경된 패키지만 빌드/테스트**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.filter.outputs.changes }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            ui:
              - 'packages/ui/**'
            utils:
              - 'packages/utils/**'
            web:
              - 'apps/web/**'

  build-and-test:
    needs: changes
    runs-on: ubuntu-latest
    if: ${{ needs.changes.outputs.packages != '[]' }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: 9.14.2

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build changed packages
        run: |
          pnpm turbo build --filter=...[origin/main]
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

      - name: Test changed packages
        run: |
          pnpm turbo test --filter=...[origin/main]

      - name: Lint changed packages
        run: |
          pnpm turbo lint --filter=...[origin/main]
```

**Turbo 필터 설명:**

- `--filter=...[origin/main]`: main 브랜치 이후 변경된 패키지와 그에 의존하는 모든 패키지
- `--filter=...@my-monorepo/ui`: ui 패키지와 그에 의존하는 모든 패키지
- `--filter=@my-monorepo/web^...`: web 패키지가 의존하는 모든 패키지 (web 자신 제외)

### 배포 전략

**1. 개별 앱 배포 (Vercel)**

```yaml
# .github/workflows/deploy-web.yml
name: Deploy Web

on:
  push:
    branches: [main]
    paths:
      - 'apps/web/**'
      - 'packages/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9.14.2

      - name: Build
        run: |
          pnpm install --frozen-lockfile
          pnpm turbo build --filter=@my-monorepo/web
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: apps/web
          production: true
```

**2. 공유 패키지 npm 배포**

```yaml
# .github/workflows/publish.yml
name: Publish Packages

on:
  push:
    branches: [main]
    paths:
      - 'packages/**'

jobs:
  version-check:
    runs-on: ubuntu-latest
    outputs:
      changed-packages: ${{ steps.check.outputs.packages }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2

      - id: check
        run: |
          CHANGED=$(git diff HEAD^ HEAD --name-only | grep 'packages/.*/package.json' | xargs -I {} dirname {})
          echo "packages=${CHANGED}" >> $GITHUB_OUTPUT

  publish:
    needs: version-check
    if: needs.version-check.outputs.changed-packages != ''
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 9.14.2

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'

      - name: Install and Build
        run: |
          pnpm install --frozen-lockfile
          pnpm turbo build --filter=./packages/*

      - name: Publish to npm
        run: |
          for package in ${{ needs.version-check.outputs.changed-packages }}; do
            cd $package
            pnpm publish --no-git-checks --access public
            cd -
          done
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 실무 팁과 주의사항

### 버전 관리 전략

**1. workspace 프로토콜 사용**

```json
{
  "dependencies": {
    "@my-monorepo/ui": "workspace:*",
    "@my-monorepo/utils": "workspace:^"
  }
}
```

- `workspace:*`: 현재 workspace의 정확한 버전 사용
- `workspace:^`: 현재 workspace의 호환 가능한 버전 사용

**배포 시 자동 변환:**

```bash
# 개발 시
"@my-monorepo/ui": "workspace:*"

# pnpm publish 후
"@my-monorepo/ui": "1.2.3"
```

**2. Changesets를 사용한 버전 관리**

```bash
# Changesets 설치
pnpm add -Dw @changesets/cli
pnpm changeset init
```

**.changeset/config.json:**

```json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.3/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": ["@my-monorepo/web"]
}
```

**버전 업데이트 워크플로우:**

```bash
# 1. 변경사항 추가
pnpm changeset
# ✔ 어떤 패키지가 변경되었나요? · @my-monorepo/ui
# ✔ 변경 유형은? · minor
# ✔ 변경사항 요약: Added new Button variant

# 2. 버전 업데이트
pnpm changeset version
# packages/ui/CHANGELOG.md 생성
# package.json 버전 업데이트: 1.0.0 -> 1.1.0

# 3. 배포
pnpm changeset publish
```

### 패키지 간 의존성 관리

**1. 순환 의존성 방지**

```bash
# ❌ 나쁜 예: 순환 의존성
packages/ui -> packages/utils
packages/utils -> packages/ui

# ✅ 좋은 예: 단방향 의존성
packages/core (공통 타입, 상수)
    ↓
packages/utils
    ↓
packages/ui
```

**순환 의존성 감지:**

```bash
# madge 설치
pnpm add -Dw madge

# 순환 의존성 검사
npx madge --circular --extensions ts,tsx packages/
```

**package.json에 스크립트 추가:**

```json
{
  "scripts": {
    "check-circular": "madge --circular --extensions ts,tsx packages/"
  }
}
```

**2. Peer Dependencies 올바르게 사용**

```json
{
  "name": "@my-monorepo/ui",
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0",
    "react-dom": "^18.0.0 || ^19.0.0"
  },
  "devDependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  }
}
```

**이유:**
- `peerDependencies`: 사용자가 설치해야 하는 의존성
- `devDependencies`: 개발/테스트 시 필요한 의존성
- 버전 중복 설치 방지

### 흔한 실수와 해결법

**1. TypeScript 타입 찾기 실패**

**문제:**
```typescript
// apps/web에서
import { Button } from '@my-monorepo/ui'; // TS2307: Cannot find module
```

**해결:**

```json
// apps/web/tsconfig.json
{
  "extends": "@my-monorepo/tsconfig/nextjs.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@my-monorepo/ui": ["../../packages/ui/src"],
      "@my-monorepo/utils": ["../../packages/utils/src"]
    }
  }
}
```

또는 packages/ui에서 타입 파일 제대로 빌드:

```json
// packages/ui/package.json
{
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    }
  }
}
```

**2. 개발 서버에서 변경사항 반영 안 됨**

**문제:** packages/ui를 수정했는데 apps/web에 반영 안 됨

**해결책 1: Watch 모드 활용**

```json
// turbo.json
{
  "pipeline": {
    "dev": {
      "dependsOn": ["^dev"],
      "cache": false,
      "persistent": true
    }
  }
}
```

```json
// packages/ui/package.json
{
  "scripts": {
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch"
  }
}
```

**해결책 2: Next.js transpilePackages**

```javascript
// apps/web/next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  transpilePackages: ['@my-monorepo/ui', '@my-monorepo/utils'],
};

module.exports = nextConfig;
```

**3. 빌드 캐시 문제**

**문제:** 파일을 변경했는데 캐시가 적용되어 빌드가 스킵됨

**해결:**

```bash
# 캐시 삭제 후 재빌드
pnpm turbo build --force

# 특정 패키지만 캐시 무효화
pnpm turbo build --filter=@my-monorepo/ui --force
```

**캐시 디렉토리 정리:**

```bash
# Turborepo 캐시 삭제
rm -rf .turbo

# pnpm 스토어 정리
pnpm store prune

# node_modules 재설치
rm -rf node_modules
pnpm install
```

**4. 의존성 설치 위치 혼동**

```bash
# ❌ 잘못된 방법
cd packages/ui
pnpm add react

# ✅ 올바른 방법 (루트에서)
pnpm add react --filter @my-monorepo/ui

# ✅ 모든 패키지에 동일한 dev 의존성 추가
pnpm add -Dw typescript

# ✅ workspace 프로토콜로 로컬 패키지 추가
pnpm add @my-monorepo/ui --filter @my-monorepo/web --workspace
```

**5. 환경 변수 공유 실수**

**문제:** 각 앱의 .env가 다른 앱에 영향을 줌

**해결:**

```json
// turbo.json
{
  "globalDependencies": [
    "**/.env.*local"
  ],
  "pipeline": {
    "build": {
      "env": [
        "NODE_ENV",
        "NEXT_PUBLIC_API_URL"
      ],
      "outputs": [".next/**", "!.next/cache/**"]
    }
  }
}
```

**.env 파일 구조:**

```bash
# 루트 .env (공통 설정)
NODE_ENV=development

# apps/web/.env.local (앱별 설정)
NEXT_PUBLIC_API_URL=https://api.example.com
```

### 성능 최적화 체크리스트

**빌드 속도 개선:**

- [ ] Turborepo Remote Caching 활성화
- [ ] 불필요한 `dependsOn` 제거하여 병렬화 극대화
- [ ] 개발 시 필요한 패키지만 빌드 (`--filter` 사용)
- [ ] TypeScript `incremental` 옵션 활성화
- [ ] ESLint 캐시 활성화 (`--cache` 플래그)

**개발 경험 개선:**

- [ ] 모든 패키지에 `dev` 스크립트 추가
- [ ] Hot Module Replacement 설정
- [ ] TypeScript `paths` 설정으로 빠른 타입 체크
- [ ] `transpilePackages`로 실시간 반영

**CI/CD 최적화:**

- [ ] 변경된 패키지만 테스트 (`--filter=...[HEAD^1]`)
- [ ] GitHub Actions 캐시 활용
- [ ] 병렬 작업 활용 (build, test, lint 동시 실행)
- [ ] Docker 레이어 캐싱

## 마치며

Monorepo는 초기 설정 비용이 들지만, 다음과 같은 상황에서 큰 가치를 제공합니다:

**Monorepo의 진정한 가치:**

1. **코드 재사용성**: 공통 컴포넌트, 유틸리티를 쉽게 공유
2. **일관성**: 단일 설정 파일로 모든 프로젝트 관리
3. **개발 속도**: 캐싱과 병렬 실행으로 빠른 빌드
4. **협업 효율**: 원자적 커밋으로 안전한 대규모 리팩토링
5. **유지보수성**: 의존성 버전 통일로 업그레이드 간소화

**시작하기 전 고려사항:**

- 팀 크기: 3명 이상의 팀에서 효과적
- 프로젝트 복잡도: 공유할 코드가 충분한가?
- 학습 곡선: 팀원들이 새로운 도구를 배울 준비가 되었는가?
- 마이그레이션 비용: 기존 프로젝트 이전에 드는 시간

**다음 단계:**

1. 작은 프로젝트로 시작 (2-3개 패키지)
2. 팀과 컨벤션 합의 (네이밍, 버전 관리 등)
3. 문서화 (README, CONTRIBUTING 가이드)
4. CI/CD 파이프라인 구축
5. 점진적으로 패키지 추가

pnpm과 Turborepo의 조합은 현대적인 Monorepo 구축에 최적화된 도구입니다. 이 가이드를 바탕으로 팀에 맞는 Monorepo를 구축하고, 생산성을 크게 향상시킬 수 있기를 바랍니다.

## 참고 자료

- [pnpm 공식 문서](https://pnpm.io/)
- [Turborepo 공식 문서](https://turbo.build/repo/docs)
- [pnpm workspace 가이드](https://pnpm.io/workspaces)
- [Turborepo 핸드북](https://turbo.build/repo/docs/handbook)
- [Changesets 문서](https://github.com/changesets/changesets)
- [Vercel Remote Caching](https://vercel.com/docs/monorepos/remote-caching)
