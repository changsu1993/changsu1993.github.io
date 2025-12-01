---
title: "Storybook 컴포넌트 문서화 완벽 가이드 - 디자인 시스템과 연계한 UI 컴포넌트 개발/테스트"
description: "Storybook 8로 컴포넌트 문서화, 인터랙티브 테스트, 접근성 검증, CI/CD 통합까지. 디자인 시스템과 연계한 실전 가이드입니다."
date: 2025-12-01 10:00:00 +0900
categories: [Frontend, DevTools]
tags: [storybook, component, documentation, design-system, testing]
---

## 개요

Storybook은 UI 컴포넌트를 독립적으로 개발하고 문서화하는 오픈소스 도구입니다. 컴포넌트를 격리된 환경에서 개발하고 테스트하며, 자동으로 문서를 생성하여 팀 전체가 공유할 수 있습니다.

**이 글에서 배울 수 있는 것:**
- Storybook의 핵심 개념과 활용 방법
- CSF 3.0 형식으로 Story 작성하기
- Args, ArgTypes, Controls로 인터랙티브 문서 만들기
- MDX로 상세한 컴포넌트 가이드 작성
- 접근성 테스트와 시각적 회귀 테스트
- CI/CD 파이프라인에 Storybook 통합
- 디자인 시스템과의 연계 전략

**사전 요구사항:**
- React 18+ 기본 지식
- TypeScript 기본 문법
- 컴포넌트 기반 개발 이해
- [디자인 시스템 구축 가이드](/posts/design-system-guide/) 읽기 권장

**예상 소요 시간:** 약 50분

---

## Storybook이란?

### 정의

Storybook은 **UI 컴포넌트를 독립적으로 개발하고 문서화하는 프론트엔드 워크벤치**입니다. 컴포넌트를 애플리케이션과 분리하여 다양한 상태를 시각화하고 테스트할 수 있습니다.

```text
Storybook = 컴포넌트 개발 환경 + 인터랙티브 문서 + 테스트 플랫폼
```

### 왜 Storybook이 필요한가?

**Storybook 없이 개발할 때의 문제점:**

```tsx
// ❌ 문제 1: 컴포넌트 테스트를 위해 전체 앱 실행
function App() {
  return (
    <div>
      {/* 여러 페이지와 라우팅... */}
      {/* 복잡한 상태 관리... */}
      {/* 인증 로직... */}
      <Button variant="primary">테스트할 버튼</Button>
    </div>
  );
}

// ❌ 문제 2: 다양한 상태를 확인하려면 코드 수정 반복
<Button variant="primary">Primary</Button>
// → 코드 수정
<Button variant="secondary">Secondary</Button>
// → 코드 수정
<Button disabled>Disabled</Button>
// → 반복...

// ❌ 문제 3: 문서화가 코드와 분리되어 관리
// README.md 어딘가에...
// 업데이트 누락 가능성 높음
```

**Storybook을 사용하면:**

```tsx
// ✅ 해결 1: 컴포넌트만 독립적으로 개발
export default {
  title: 'Components/Button',
  component: Button,
};

// ✅ 해결 2: 모든 변형을 한 번에 시각화
export const Primary = { args: { variant: 'primary' } };
export const Secondary = { args: { variant: 'secondary' } };
export const Disabled = { args: { disabled: true } };

// ✅ 해결 3: 코드와 문서가 함께 관리됨
// Story 파일 = 실행 가능한 문서
```

### 핵심 이점

1. **격리된 개발 환경**
   - 컴포넌트를 애플리케이션 없이 개발
   - 외부 의존성 제거로 빠른 피드백
   - 특정 상태 재현이 쉬움

2. **시각적 문서화**
   - 코드와 함께 업데이트되는 살아있는 문서
   - 인터랙티브 테스트 가능
   - 디자이너-개발자 협업 도구

3. **품질 보증**
   - 접근성 테스트 자동화
   - 시각적 회귀 테스트
   - 다양한 상태 검증

4. **팀 협업**
   - 컴포넌트 재사용성 향상
   - 디자인 시스템 중앙 관리
   - PR 리뷰 시 시각적 확인

### Storybook을 사용하는 기업

- **Airbnb**: 디자인 시스템 관리
- **Uber**: 컴포넌트 라이브러리 문서화
- **Microsoft**: Fluent UI 개발
- **Shopify**: Polaris 디자인 시스템
- **BBC**: 뉴스 플랫폼 컴포넌트

---

## Storybook 8.x 설치 및 설정

### React + TypeScript + Vite 환경 설정

```bash
# 1. React + TypeScript 프로젝트 생성 (Vite 사용)
npm create vite@latest my-storybook-app -- --template react-ts
cd my-storybook-app
npm install

# 2. Storybook 8.x 설치
npx storybook@latest init

# 3. Storybook 실행
npm run storybook
```

Storybook CLI가 자동으로:
- 필요한 패키지 설치
- 설정 파일 생성 (`.storybook/` 디렉토리)
- 예제 Story 생성
- `package.json`에 스크립트 추가

### 프로젝트 구조

```text
my-storybook-app/
├── .storybook/
│   ├── main.ts          # Storybook 설정
│   └── preview.ts       # 전역 설정 (데코레이터, 파라미터)
├── src/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.stories.tsx  # Story 파일
│   │   │   ├── Button.mdx          # 상세 문서 (선택)
│   │   │   └── Button.test.tsx     # 단위 테스트
│   │   ├── Input/
│   │   └── Card/
│   └── ...
└── package.json
```

### Storybook 설정 파일

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  // Story 파일 위치 지정
  stories: [
    '../src/**/*.mdx',
    '../src/**/*.stories.@(js|jsx|mjs|ts|tsx)',
  ],

  // Addon 설정
  addons: [
    '@storybook/addon-links',        // Story 간 링크
    '@storybook/addon-essentials',   // 필수 Addon 모음
    '@storybook/addon-interactions', // 인터랙션 테스트
    '@storybook/addon-a11y',         // 접근성 테스트
  ],

  // 프레임워크 설정
  framework: {
    name: '@storybook/react-vite',
    options: {},
  },

  // 자동 문서 생성
  docs: {
    autodocs: 'tag', // 'autodocs' 태그가 있는 Story만 문서화
  },

  // TypeScript 설정
  typescript: {
    check: true, // 타입 체크 활성화
    reactDocgen: 'react-docgen-typescript', // Props 문서 자동 생성
  },
};

export default config;
```

```typescript
// .storybook/preview.ts
import type { Preview } from '@storybook/react';
import '../src/index.css'; // 전역 CSS 임포트 (Tailwind 등)

const preview: Preview = {
  // 전역 파라미터
  parameters: {
    // Actions addon 설정
    actions: { argTypesRegex: '^on[A-Z].*' }, // on으로 시작하는 props 자동 감지

    // Controls addon 설정
    controls: {
      matchers: {
        color: /(background|color)$/i, // color로 끝나는 props를 컬러 피커로
        date: /Date$/,                 // Date로 끝나는 props를 날짜 선택기로
      },
    },

    // 레이아웃 설정
    layout: 'centered', // 'centered' | 'fullscreen' | 'padded'

    // 배경색 설정
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#1a202c' },
        { name: 'gray', value: '#f3f4f6' },
      ],
    },

    // 뷰포트 설정
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: { width: '375px', height: '667px' },
        },
        tablet: {
          name: 'Tablet',
          styles: { width: '768px', height: '1024px' },
        },
        desktop: {
          name: 'Desktop',
          styles: { width: '1440px', height: '900px' },
        },
      },
    },
  },
};

export default preview;
```

### 필수 Addon 설치

```bash
# 접근성 테스트
npm install --save-dev @storybook/addon-a11y

# 인터랙션 테스트
npm install --save-dev @storybook/addon-interactions @storybook/testing-library

# Chromatic (시각적 회귀 테스트) - 선택사항
npm install --save-dev chromatic
```

---

## Story 작성법 - CSF 3.0 형식

### CSF (Component Story Format)란?

CSF는 Storybook의 표준 Story 작성 형식입니다. CSF 3.0은 더 간결하고 타입 안전한 방식을 제공합니다.

### 기본 Story 구조

```tsx
// src/components/Button/Button.tsx
import React from 'react';

export interface ButtonProps {
  /**
   * 버튼의 시각적 스타일
   */
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';

  /**
   * 버튼의 크기
   */
  size?: 'sm' | 'md' | 'lg';

  /**
   * 비활성화 여부
   */
  disabled?: boolean;

  /**
   * 버튼 내용
   */
  children: React.ReactNode;

  /**
   * 클릭 이벤트 핸들러
   */
  onClick?: () => void;
}

export const Button = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
  onClick,
}: ButtonProps) => {
  const baseClasses = 'inline-flex items-center justify-center rounded-md font-medium transition-colors';

  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
    outline: 'border border-blue-600 text-blue-600 hover:bg-blue-50',
    ghost: 'text-blue-600 hover:bg-blue-50',
  };

  const sizeClasses = {
    sm: 'h-8 px-3 text-xs',
    md: 'h-10 px-4 text-sm',
    lg: 'h-12 px-6 text-base',
  };

  const className = [
    baseClasses,
    variantClasses[variant],
    sizeClasses[size],
    disabled && 'opacity-50 cursor-not-allowed',
  ].filter(Boolean).join(' ');

  return (
    <button
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

{% raw %}
```tsx
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from '@storybook/test';
import { Button } from './Button';

// Meta 정의: 컴포넌트의 메타데이터
const meta = {
  title: 'Components/Button',     // Storybook 사이드바 위치
  component: Button,               // 대상 컴포넌트
  parameters: {
    layout: 'centered',            // 레이아웃: centered, fullscreen, padded
  },
  tags: ['autodocs'],              // 자동 문서 생성 활성화

  // ArgTypes: Props 제어 방식 정의
  argTypes: {
    variant: {
      control: 'select',           // 드롭다운 선택
      options: ['primary', 'secondary', 'outline', 'ghost'],
      description: '버튼의 시각적 스타일',
      table: {
        type: { summary: 'string' },
        defaultValue: { summary: 'primary' },
      },
    },
    size: {
      control: 'radio',            // 라디오 버튼
      options: ['sm', 'md', 'lg'],
      description: '버튼의 크기',
    },
    disabled: {
      control: 'boolean',          // 체크박스
      description: '비활성화 여부',
    },
    children: {
      control: 'text',             // 텍스트 입력
      description: '버튼 내용',
    },
    onClick: {
      description: '클릭 이벤트 핸들러',
    },
  },

  // Args: 기본 Props 값
  args: {
    onClick: fn(),                 // Actions addon에서 이벤트 로깅
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Story 정의: 컴포넌트의 다양한 상태
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Outline: Story = {
  args: {
    variant: 'outline',
    children: 'Outline Button',
  },
};

export const Ghost: Story = {
  args: {
    variant: 'ghost',
    children: 'Ghost Button',
  },
};

// 모든 크기 한 번에 표시
export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="md">Medium</Button>
      <Button size="lg">Large</Button>
    </div>
  ),
};

// 비활성화 상태
export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

// 로딩 상태
export const Loading: Story = {
  args: {
    disabled: true,
    children: (
      <>
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
      </>
    ),
  },
};

// 전체 너비
export const FullWidth: Story = {
  args: {
    children: 'Full Width Button',
  },
  decorators: [
    (Story) => (
      <div style={{ width: '100%', maxWidth: '400px' }}>
        <Story />
      </div>
    ),
  ],
};
```
{% endraw %}

### Story 작성 패턴

**1. 기본 Story (단일 상태)**

```tsx
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Click me',
  },
};
```

**2. Render 함수 사용 (복잡한 렌더링)**

```tsx
export const AllVariants: Story = {
  render: () => (
    <div className="space-y-4">
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
    </div>
  ),
};
```

**3. Play 함수 (인터랙션 테스트)**

```tsx
import { userEvent, within, expect } from '@storybook/test';

export const ClickTest: Story = {
  args: {
    children: 'Click me',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button', { name: /click me/i });

    // 버튼 클릭 시뮬레이션
    await userEvent.click(button);

    // 기대값 검증
    await expect(button).toBeInTheDocument();
  },
};
```

**4. Decorators (공통 래퍼)**

```tsx
// 개별 Story에 Decorator 적용
export const WithContainer: Story = {
  args: {
    children: 'Button',
  },
  decorators: [
    (Story) => (
      <div className="p-8 bg-gray-100 rounded">
        <Story />
      </div>
    ),
  ],
};

// 모든 Story에 Decorator 적용
const meta = {
  // ...
  decorators: [
    (Story) => (
      <div className="font-sans">
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof Button>;
```

---

## Args와 ArgTypes로 Props 문서화

### Args: 컴포넌트 Props 값

Args는 컴포넌트에 전달되는 Props의 실제 값입니다.

```tsx
// 기본 Args (모든 Story에 적용)
const meta = {
  // ...
  args: {
    variant: 'primary',
    size: 'md',
    disabled: false,
  },
} satisfies Meta<typeof Button>;

// Story별 Args (기본 Args 오버라이드)
export const Large: Story = {
  args: {
    size: 'lg', // 'md'를 'lg'로 오버라이드
  },
};
```

### ArgTypes: Props 제어 방식 정의

ArgTypes는 Controls addon에서 Props를 어떻게 제어할지 정의합니다.

```tsx
const meta = {
  // ...
  argTypes: {
    // Select: 드롭다운 선택
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost'],
      description: '버튼의 시각적 스타일',
      table: {
        type: { summary: 'string' },
        defaultValue: { summary: 'primary' },
        category: 'Appearance', // 카테고리로 그룹화
      },
    },

    // Radio: 라디오 버튼
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
      description: '버튼의 크기',
      table: {
        type: { summary: 'string' },
        defaultValue: { summary: 'md' },
        category: 'Appearance',
      },
    },

    // Boolean: 체크박스
    disabled: {
      control: 'boolean',
      description: '비활성화 여부',
      table: {
        type: { summary: 'boolean' },
        defaultValue: { summary: false },
        category: 'State',
      },
    },

    // Text: 텍스트 입력
    children: {
      control: 'text',
      description: '버튼 내용',
      table: {
        type: { summary: 'ReactNode' },
        category: 'Content',
      },
    },

    // Color: 컬러 피커
    backgroundColor: {
      control: 'color',
      description: '배경색',
    },

    // Number: 숫자 입력
    padding: {
      control: { type: 'number', min: 0, max: 100, step: 4 },
      description: '패딩 (px)',
    },

    // Range: 슬라이더
    opacity: {
      control: { type: 'range', min: 0, max: 1, step: 0.1 },
      description: '투명도',
    },

    // Object: JSON 에디터
    style: {
      control: 'object',
      description: '커스텀 스타일',
    },

    // Date: 날짜 선택기
    createdAt: {
      control: 'date',
      description: '생성 날짜',
    },

    // 제어 비활성화
    onClick: {
      control: false, // Controls 패널에 표시하지 않음
      description: '클릭 이벤트 핸들러',
      table: {
        category: 'Events',
      },
    },
  },
} satisfies Meta<typeof Button>;
```

### TypeScript로 타입 안전하게 작성

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button, type ButtonProps } from './Button';

// Meta의 제네릭으로 ButtonProps 타입 전달
const meta: Meta<ButtonProps> = {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    // ✅ TypeScript가 ButtonProps의 속성을 자동 완성
    variant: {
      control: 'select',
      options: ['primary', 'secondary'], // ✅ 타입 체크됨
    },
    // ❌ typo 같은 오류를 컴파일 타임에 잡아냄
    // varinat: { ... } // 오류!
  },
};

export default meta;
type Story = StoryObj<ButtonProps>;

// Story도 타입 안전
export const Primary: Story = {
  args: {
    variant: 'primary', // ✅ 타입 체크됨
    children: 'Button',
    // ❌ 존재하지 않는 props는 오류
    // invalidProp: true, // 오류!
  },
};
```

---

## Controls Addon으로 인터랙티브 테스트

Controls addon을 사용하면 Storybook UI에서 실시간으로 Props를 변경하며 컴포넌트를 테스트할 수 있습니다.

### Controls 패널 활용

```tsx
// 모든 ArgTypes가 자동으로 Controls 패널에 표시됨
const meta = {
  // ...
  argTypes: {
    variant: { control: 'select', options: ['primary', 'secondary'] },
    size: { control: 'radio', options: ['sm', 'md', 'lg'] },
    disabled: { control: 'boolean' },
  },
} satisfies Meta<typeof Button>;
```

**Controls 패널에서 할 수 있는 것:**
1. 드롭다운으로 `variant` 변경 → 즉시 버튼 스타일 변경
2. 라디오 버튼으로 `size` 변경 → 즉시 크기 변경
3. 체크박스로 `disabled` 토글 → 비활성화 상태 확인

### 고급 Controls 설정

```tsx
// Input 컴포넌트 예제
interface InputProps {
  label?: string;
  placeholder?: string;
  type?: 'text' | 'email' | 'password' | 'number';
  disabled?: boolean;
  error?: string;
  helperText?: string;
  maxLength?: number;
  minLength?: number;
}

const meta = {
  title: 'Components/Input',
  component: Input,
  argTypes: {
    // 텍스트 입력
    label: {
      control: 'text',
      description: '입력 필드 라벨',
    },
    placeholder: {
      control: 'text',
      description: '플레이스홀더 텍스트',
    },

    // 선택 옵션
    type: {
      control: 'select',
      options: ['text', 'email', 'password', 'number'],
      description: '입력 타입',
    },

    // 불린 값
    disabled: {
      control: 'boolean',
      description: '비활성화 여부',
    },

    // 조건부 표시
    error: {
      control: 'text',
      description: '에러 메시지',
      if: { arg: 'disabled', truthy: false }, // disabled가 false일 때만 표시
    },

    // 숫자 범위
    maxLength: {
      control: { type: 'number', min: 0, max: 1000 },
      description: '최대 길이',
    },
  },
} satisfies Meta<typeof Input>;
```

### Preset Args로 빠른 테스트

```tsx
export const Default: Story = {
  args: {
    label: 'Email',
    placeholder: 'you@example.com',
    type: 'email',
  },
};

export const WithError: Story = {
  args: {
    label: 'Email',
    placeholder: 'you@example.com',
    type: 'email',
    error: 'Invalid email address',
  },
};

export const Disabled: Story = {
  args: {
    label: 'Email',
    placeholder: 'you@example.com',
    disabled: true,
  },
};
```

각 Story는 프리셋처럼 작동하여, Controls 패널에서 추가로 조정 가능합니다.

---

## Actions Addon으로 이벤트 핸들러 테스트

Actions addon은 이벤트 핸들러 호출을 로깅하여 컴포넌트 인터랙션을 추적합니다.

### 기본 사용법

```tsx
import { fn } from '@storybook/test';

const meta = {
  title: 'Components/Button',
  component: Button,
  args: {
    onClick: fn(), // Actions 패널에 로그 출력
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    children: 'Click me',
  },
};
```

버튼을 클릭하면 Actions 패널에 다음과 같이 표시됩니다:
```text
onClick: []
```

### 복잡한 이벤트 로깅

```tsx
// Form 컴포넌트 예제
interface FormProps {
  onSubmit?: (data: { email: string; password: string }) => void;
  onChange?: (field: string, value: string) => void;
  onReset?: () => void;
}

const meta = {
  title: 'Components/Form',
  component: Form,
  args: {
    onSubmit: fn((data) => {
      console.log('Submitted:', data);
    }),
    onChange: fn((field, value) => {
      console.log(`Field ${field} changed to:`, value);
    }),
    onReset: fn(() => {
      console.log('Form reset');
    }),
  },
} satisfies Meta<typeof Form>;
```

Actions 패널에서 각 이벤트와 전달된 인자를 확인할 수 있습니다:
```text
onSubmit: [{ email: "test@example.com", password: "••••••" }]
onChange: ["email", "test@example.com"]
onChange: ["password", "••••••"]
onReset: []
```

### 자동 Action 감지

```typescript
// .storybook/preview.ts
const preview: Preview = {
  parameters: {
    actions: {
      // 'on'으로 시작하는 모든 props를 자동으로 Actions로 처리
      argTypesRegex: '^on[A-Z].*',
    },
  },
};
```

이제 `onClick`, `onChange`, `onSubmit` 등 모든 이벤트 핸들러가 자동으로 로깅됩니다.

---

## 디자인 시스템 컴포넌트 문서화 실전 예제

실무에서 자주 사용하는 컴포넌트들을 Storybook으로 문서화하는 방법을 알아봅니다.

### 1. Button 컴포넌트

```tsx
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from '@storybook/test';
import { Button } from './Button';
import { Mail, Download, Plus, X } from 'lucide-react';

const meta = {
  title: 'Design System/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost', 'destructive'],
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg', 'icon'],
    },
  },
  args: {
    onClick: fn(),
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// 모든 변형 한눈에 보기
export const Showcase: Story = {
  render: () => (
    <div className="space-y-8">
      <div>
        <h3 className="text-lg font-semibold mb-4">Variants</h3>
        <div className="flex flex-wrap gap-4">
          <Button variant="primary">Primary</Button>
          <Button variant="secondary">Secondary</Button>
          <Button variant="outline">Outline</Button>
          <Button variant="ghost">Ghost</Button>
          <Button variant="destructive">Destructive</Button>
        </div>
      </div>

      <div>
        <h3 className="text-lg font-semibold mb-4">Sizes</h3>
        <div className="flex items-center gap-4">
          <Button size="sm">Small</Button>
          <Button size="md">Medium</Button>
          <Button size="lg">Large</Button>
        </div>
      </div>

      <div>
        <h3 className="text-lg font-semibold mb-4">With Icons</h3>
        <div className="flex flex-wrap gap-4">
          <Button>
            <Mail className="w-4 h-4 mr-2" />
            Send Email
          </Button>
          <Button variant="secondary">
            <Download className="w-4 h-4 mr-2" />
            Download
          </Button>
          <Button variant="outline">
            Add Item
            <Plus className="w-4 h-4 ml-2" />
          </Button>
        </div>
      </div>

      <div>
        <h3 className="text-lg font-semibold mb-4">Icon Only</h3>
        <div className="flex gap-4">
          <Button size="icon" variant="primary">
            <Plus className="w-5 h-5" />
          </Button>
          <Button size="icon" variant="secondary">
            <X className="w-5 h-5" />
          </Button>
          <Button size="icon" variant="outline">
            <Mail className="w-5 h-5" />
          </Button>
        </div>
      </div>

      <div>
        <h3 className="text-lg font-semibold mb-4">States</h3>
        <div className="flex gap-4">
          <Button disabled>Disabled</Button>
          <Button disabled>
            <svg className="animate-spin -ml-1 mr-2 h-4 w-4" fill="none" viewBox="0 0 24 24">
              <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
              <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
            </svg>
            Loading...
          </Button>
        </div>
      </div>
    </div>
  ),
};

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};
```

### 2. Input 컴포넌트

```tsx
// src/components/Input/Input.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Input } from './Input';

const meta = {
  title: 'Design System/Input',
  component: Input,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
} satisfies Meta<typeof Input>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Showcase: Story = {
  render: () => (
    <div className="w-[400px] space-y-6">
      <Input
        label="Email"
        type="email"
        placeholder="you@example.com"
      />

      <Input
        label="Password"
        type="password"
        placeholder="Enter password"
        helperText="At least 8 characters"
      />

      <Input
        label="Email"
        type="email"
        placeholder="you@example.com"
        error="Invalid email address"
      />

      <Input
        label="Disabled Input"
        placeholder="Cannot edit"
        disabled
      />

      <Input
        label="Required Field"
        placeholder="Enter value"
        required
      />
    </div>
  ),
};

export const Default: Story = {
  args: {
    label: 'Email',
    type: 'email',
    placeholder: 'you@example.com',
  },
};

export const WithError: Story = {
  args: {
    label: 'Email',
    type: 'email',
    placeholder: 'you@example.com',
    error: 'Invalid email address',
  },
};
```

### 3. Card 컴포넌트

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
import { Input } from '../Input/Input';

const meta = {
  title: 'Design System/Card',
  component: Card,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
} satisfies Meta<typeof Card>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Showcase: Story = {
  render: () => (
    <div className="grid grid-cols-1 md:grid-cols-2 gap-6 max-w-4xl">
      {/* 기본 카드 */}
      <Card>
        <CardHeader>
          <CardTitle>Basic Card</CardTitle>
          <CardDescription>
            This is a basic card with header and content.
          </CardDescription>
        </CardHeader>
        <CardContent>
          <p className="text-sm text-gray-600">
            Card content goes here. You can put any content you want.
          </p>
        </CardContent>
      </Card>

      {/* 폼 카드 */}
      <Card>
        <CardHeader>
          <CardTitle>Sign In</CardTitle>
          <CardDescription>
            Enter your credentials to access your account
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <Input
            label="Email"
            type="email"
            placeholder="you@example.com"
          />
          <Input
            label="Password"
            type="password"
            placeholder="••••••••"
          />
        </CardContent>
        <CardFooter>
          <Button className="w-full">Sign In</Button>
        </CardFooter>
      </Card>

      {/* 통계 카드 */}
      <Card>
        <CardHeader>
          <CardTitle>Total Revenue</CardTitle>
          <CardDescription>
            Year to date performance
          </CardDescription>
        </CardHeader>
        <CardContent>
          <div className="text-3xl font-bold">$45,231.89</div>
          <p className="text-sm text-green-600 mt-2">
            +20.1% from last month
          </p>
        </CardContent>
      </Card>

      {/* 액션 카드 */}
      <Card>
        <CardHeader>
          <CardTitle>Delete Account</CardTitle>
          <CardDescription>
            This action cannot be undone
          </CardDescription>
        </CardHeader>
        <CardContent>
          <p className="text-sm text-gray-600">
            Are you sure you want to delete your account? All of your data will be permanently removed.
          </p>
        </CardContent>
        <CardFooter className="gap-2">
          <Button variant="outline">Cancel</Button>
          <Button variant="destructive">Delete</Button>
        </CardFooter>
      </Card>
    </div>
  ),
};

export const Basic: Story = {
  render: () => (
    <Card className="w-[380px]">
      <CardHeader>
        <CardTitle>Card Title</CardTitle>
        <CardDescription>Card description goes here</CardDescription>
      </CardHeader>
      <CardContent>
        <p>Card content with some meaningful information.</p>
      </CardContent>
    </Card>
  ),
};
```

### 4. Modal 컴포넌트

```tsx
// src/components/Modal/Modal.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { useState } from 'react';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogDescription,
  DialogFooter,
} from './Modal';
import { Button } from '../Button/Button';
import { Input } from '../Input/Input';

const meta = {
  title: 'Design System/Modal',
  component: Dialog,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
} satisfies Meta<typeof Dialog>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Basic: Story = {
  render: () => {
    const [open, setOpen] = useState(false);

    return (
      <>
        <Button onClick={() => setOpen(true)}>
          Open Modal
        </Button>

        <Dialog open={open} onOpenChange={setOpen}>
          <DialogContent>
            <DialogHeader>
              <DialogTitle>Modal Title</DialogTitle>
              <DialogDescription>
                This is a basic modal dialog. You can add any content here.
              </DialogDescription>
            </DialogHeader>

            <div className="py-4">
              <p className="text-sm text-gray-600">
                Modal content goes here. This could be a form, information, or any other content.
              </p>
            </div>

            <DialogFooter>
              <Button variant="outline" onClick={() => setOpen(false)}>
                Cancel
              </Button>
              <Button onClick={() => setOpen(false)}>
                Confirm
              </Button>
            </DialogFooter>
          </DialogContent>
        </Dialog>
      </>
    );
  },
};

export const WithForm: Story = {
  render: () => {
    const [open, setOpen] = useState(false);

    return (
      <>
        <Button onClick={() => setOpen(true)}>
          Create New Item
        </Button>

        <Dialog open={open} onOpenChange={setOpen}>
          <DialogContent>
            <DialogHeader>
              <DialogTitle>Create New Item</DialogTitle>
              <DialogDescription>
                Fill in the details to create a new item.
              </DialogDescription>
            </DialogHeader>

            <div className="space-y-4 py-4">
              <Input
                label="Name"
                placeholder="Enter item name"
                required
              />
              <Input
                label="Description"
                placeholder="Enter description"
              />
            </div>

            <DialogFooter>
              <Button variant="outline" onClick={() => setOpen(false)}>
                Cancel
              </Button>
              <Button onClick={() => setOpen(false)}>
                Create
              </Button>
            </DialogFooter>
          </DialogContent>
        </Dialog>
      </>
    );
  },
};

export const Destructive: Story = {
  render: () => {
    const [open, setOpen] = useState(false);

    return (
      <>
        <Button variant="destructive" onClick={() => setOpen(true)}>
          Delete Account
        </Button>

        <Dialog open={open} onOpenChange={setOpen}>
          <DialogContent>
            <DialogHeader>
              <DialogTitle>Are you absolutely sure?</DialogTitle>
              <DialogDescription>
                This action cannot be undone. This will permanently delete your account and remove your data from our servers.
              </DialogDescription>
            </DialogHeader>

            <DialogFooter>
              <Button variant="outline" onClick={() => setOpen(false)}>
                Cancel
              </Button>
              <Button variant="destructive" onClick={() => setOpen(false)}>
                Delete Account
              </Button>
            </DialogFooter>
          </DialogContent>
        </Dialog>
      </>
    );
  },
};
```

---

## MDX를 활용한 상세 문서 작성

MDX는 Markdown과 JSX를 결합하여 더 풍부한 문서를 작성할 수 있게 합니다.

### 기본 MDX 문서 구조

````jsx
{/* src/components/Button/Button.mdx */}
import { Canvas, Meta, Story, Controls, Primary, Stories } from '@storybook/blocks';
import * as ButtonStories from './Button.stories';

<Meta of={ButtonStories} />

# Button

재사용 가능한 버튼 컴포넌트입니다. 다양한 변형과 크기를 지원하며, 접근성을 고려하여 설계되었습니다.

## 개요

Button 컴포넌트는 사용자 액션을 트리거하는 인터랙티브 요소입니다. 5가지 시각적 변형과 4가지 크기를 제공합니다.

<Canvas of={ButtonStories.Showcase} />

## 사용법

```tsx
import { Button } from '@/components/Button';

function MyComponent() {
  return (
    <Button variant="primary" size="md" onClick={() => alert('Clicked!')}>
      Click me
    </Button>
  );
}
```

## Props

<Controls of={ButtonStories.Primary} />

## 변형 (Variants)

### Primary
주요 액션에 사용합니다. 페이지에서 가장 중요한 액션에만 사용하세요.

<Canvas of={ButtonStories.Primary} />

### Secondary
보조 액션에 사용합니다. Primary 버튼과 함께 사용할 수 있습니다.

<Canvas of={ButtonStories.Secondary} />

### Outline
덜 중요한 액션이나 대체 액션에 사용합니다.

<Canvas of={ButtonStories.Outline} />

### Ghost
최소한의 시각적 강조가 필요한 액션에 사용합니다.

<Canvas of={ButtonStories.Ghost} />

### Destructive
삭제나 취소 같은 위험한 액션에 사용합니다.

<Canvas of={ButtonStories.Destructive} />

## 크기 (Sizes)

<Canvas of={ButtonStories.Sizes} />

- **Small (sm)**: 제한된 공간에서 사용 (높이 32px)
- **Medium (md)**: 기본 크기 (높이 40px)
- **Large (lg)**: 중요한 액션 강조 (높이 48px)
- **Icon**: 아이콘 전용 정사각형 버튼 (40x40px)

## 아이콘 사용

### 텍스트와 함께
<Canvas of={ButtonStories.WithIcon} />

```tsx
import { Mail, Download, Plus } from 'lucide-react';

<Button>
  <Mail className="w-4 h-4 mr-2" />
  Send Email
</Button>

<Button>
  Download
  <Download className="w-4 h-4 ml-2" />
</Button>
```

### 아이콘만
<Canvas of={ButtonStories.IconOnly} />

```tsx
<Button size="icon" variant="primary">
  <Plus className="w-5 h-5" />
</Button>
```

## 상태 (States)

### Disabled
비활성화 상태는 사용자가 액션을 수행할 수 없음을 나타냅니다.

<Canvas of={ButtonStories.Disabled} />

### Loading
로딩 상태는 백그라운드 작업이 진행 중임을 나타냅니다.

<Canvas of={ButtonStories.Loading} />

```tsx
<Button disabled>
  <svg className="animate-spin -ml-1 mr-2 h-4 w-4" /* ... */>
    {/* spinner SVG */}
  </svg>
  Loading...
</Button>
```

## 접근성

- ✅ **키보드 탐색**: Tab, Enter, Space 키로 조작 가능
- ✅ **포커스 인디케이터**: 키보드 포커스 시 명확한 아웃라인 표시
- ✅ **비활성화 상태**: `disabled` 속성으로 스크린 리더에 전달
- ✅ **의미론적 HTML**: `<button>` 요소 사용

### 아이콘 전용 버튼

아이콘만 있는 버튼은 스크린 리더 사용자를 위해 `aria-label`을 제공해야 합니다:

```tsx
<Button size="icon" aria-label="메일 보내기">
  <Mail className="w-5 h-5" />
</Button>
```

## 베스트 프랙티스

### ✅ Good

```tsx
// Primary 버튼은 페이지당 하나만
<>
  <Button variant="primary">제출</Button>
  <Button variant="outline">취소</Button>
</>

// 아이콘 전용 버튼에 aria-label 제공
<Button size="icon" aria-label="닫기">
  <X className="w-5 h-5" />
</Button>

// 로딩 중에는 disabled 처리
<Button disabled={isLoading}>
  {isLoading ? 'Loading...' : 'Submit'}
</Button>
```

### ❌ Bad

```tsx
// Primary 버튼이 너무 많음
<>
  <Button variant="primary">저장</Button>
  <Button variant="primary">취소</Button>
  <Button variant="primary">삭제</Button>
</>

// 아이콘만 있는데 aria-label 없음
<Button size="icon">
  <X className="w-5 h-5" />
</Button>

// 로딩 중에도 클릭 가능
<Button onClick={handleSubmit}>
  {isLoading ? 'Loading...' : 'Submit'}
</Button>
```

## 모든 Story

<Stories />
````

### MDX 주요 블록

```jsx
{/* Meta: Story 메타데이터 연결 */}
<Meta of={ButtonStories} />

{/* Canvas: Story 실행 및 소스 코드 표시 */}
<Canvas of={ButtonStories.Primary} />

{/* Controls: Props 제어 패널 */}
<Controls of={ButtonStories.Primary} />

{/* Stories: 모든 Story 목록 표시 */}
<Stories />

{/* Primary: 첫 번째 Story의 큰 미리보기 */}
<Primary />
```

### 코드 블록 강조

````jsx
```tsx
// 코드 블록에 언어 지정
import { Button } from '@/components/Button';

function App() {
  return <Button>Click me</Button>;
}
```
````

---

## 접근성 테스트 (a11y addon)

### a11y Addon 설정

```bash
npm install --save-dev @storybook/addon-a11y
```

```typescript
// .storybook/main.ts
const config: StorybookConfig = {
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y', // 접근성 테스트 추가
  ],
};
```

### 자동 접근성 검사

a11y addon은 Axe 엔진을 사용하여 다음을 자동으로 검사합니다:
- 색상 대비 (WCAG AA/AAA)
- ARIA 속성 올바른 사용
- 키보드 접근성
- 폼 라벨
- 이미지 대체 텍스트

### Story별 접근성 설정

```tsx
// 특정 규칙 비활성화
export const AccessibilityTest: Story = {
  parameters: {
    a11y: {
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: true, // 색상 대비 검사 활성화
          },
          {
            id: 'button-name',
            enabled: true, // 버튼 이름 검사 활성화
          },
          {
            id: 'landmark-one-main',
            enabled: false, // 이 규칙은 비활성화
          },
        ],
      },
    },
  },
  render: () => (
    <div>
      <Button>Accessible Button</Button>
      <Button disabled>Disabled Button</Button>
      <Button size="icon" aria-label="Close">
        <X className="w-5 h-5" />
      </Button>
    </div>
  ),
};
```

### 접근성 체크리스트

```tsx
// 접근성이 좋은 컴포넌트 예제
export const AccessibleForm: Story = {
  render: () => (
    <form>
      {/* ✅ 라벨과 입력 필드 연결 */}
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        aria-describedby="email-helper"
        aria-required="true"
      />
      <span id="email-helper">We'll never share your email.</span>

      {/* ✅ 에러 메시지 연결 */}
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        aria-invalid="true"
        aria-describedby="password-error"
      />
      <span id="password-error" role="alert">
        Password must be at least 8 characters
      </span>

      {/* ✅ 버튼 타입 명시 */}
      <button type="submit">Submit</button>
      <button type="reset">Reset</button>
    </form>
  ),
  parameters: {
    a11y: {
      element: '#storybook-root', // 검사할 요소
      config: {
        rules: [
          { id: 'label', enabled: true },
          { id: 'aria-required-attr', enabled: true },
        ],
      },
    },
  },
};
```

---

## 시각적 회귀 테스트 (Chromatic 연동)

Chromatic은 Storybook과 통합되어 시각적 변경을 자동으로 감지합니다.

### Chromatic 설정

```bash
# 1. Chromatic 설치
npm install --save-dev chromatic

# 2. Chromatic 프로젝트 생성 (https://www.chromatic.com)
# 프로젝트 토큰 받기

# 3. 첫 스냅샷 생성
npx chromatic --project-token=<your-project-token>
```

### package.json에 스크립트 추가

```json
{
  "scripts": {
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    "chromatic": "chromatic --exit-zero-on-changes"
  }
}
```

### Story별 스냅샷 설정

```tsx
// 특정 Story 스냅샷 비활성화
export const DynamicContent: Story = {
  parameters: {
    chromatic: { disableSnapshot: true }, // 스냅샷 제외
  },
};

// 애니메이션 일시 중지
export const AnimatedButton: Story = {
  parameters: {
    chromatic: { pauseAnimationAtEnd: true },
  },
};

// 지연 시간 추가 (데이터 로딩 대기)
export const AsyncContent: Story = {
  parameters: {
    chromatic: { delay: 3000 }, // 3초 대기 후 스냅샷
  },
};

// 특정 뷰포트만 테스트
export const ResponsiveCard: Story = {
  parameters: {
    chromatic: {
      viewports: [320, 768, 1200], // 모바일, 태블릿, 데스크톱
    },
  },
};
```

### GitHub Actions로 자동화

```yaml
# .github/workflows/chromatic.yml
name: Chromatic

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  chromatic:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Chromatic이 전체 히스토리 필요

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run Chromatic
        uses: chromaui/action@latest
        with:
          projectToken: {% raw %}${{ secrets.CHROMATIC_PROJECT_TOKEN }}{% endraw %}
          exitZeroOnChanges: true # 변경사항이 있어도 빌드 성공
```

### 시각적 회귀 테스트 워크플로우

1. **PR 생성** → Chromatic이 자동으로 스냅샷 생성
2. **변경 감지** → 이전 스냅샷과 비교
3. **리뷰** → 변경사항 확인 및 승인/거부
4. **Merge** → 승인된 스냅샷이 새로운 베이스라인

```text
┌─────────────┐
│  PR 생성    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Chromatic   │ ← 자동 스냅샷 생성
│ 실행        │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 변경 감지?  │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
변경 있음  변경 없음
   │       │
   ▼       ▼
 리뷰 요청  통과
```

---

## Storybook을 CI/CD에 통합하기

### GitHub Actions로 Storybook 빌드 및 배포

```yaml
# .github/workflows/storybook.yml
name: Build and Deploy Storybook

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Storybook
        run: npm run build-storybook

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
          publish_dir: ./storybook-static
          cname: storybook.yourdomain.com # 커스텀 도메인 (선택)
```

### PR마다 Storybook 프리뷰 생성

```yaml
# .github/workflows/storybook-preview.yml
name: Storybook Preview

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  preview:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Storybook
        run: npm run build-storybook

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: {% raw %}${{ secrets.VERCEL_TOKEN }}{% endraw %}
          vercel-org-id: {% raw %}${{ secrets.VERCEL_ORG_ID }}{% endraw %}
          vercel-project-id: {% raw %}${{ secrets.VERCEL_PROJECT_ID }}{% endraw %}
          working-directory: ./storybook-static
```

### Storybook 테스트 실행

```yaml
# .github/workflows/storybook-tests.yml
name: Storybook Tests

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Storybook
        run: npm run build-storybook --quiet

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run Storybook tests
        run: npx test-storybook
```

### test-storybook 설정

```bash
# Storybook Test Runner 설치
npm install --save-dev @storybook/test-runner playwright
```

```typescript
// .storybook/test-runner.ts
import type { TestRunnerConfig } from '@storybook/test-runner';

const config: TestRunnerConfig = {
  // 접근성 테스트 자동 실행
  async postRender(page, context) {
    const storyContext = await page.evaluate(() => {
      // @ts-ignore
      return window.__STORYBOOK_PREVIEW__.storyStore.storyFromId(context.id);
    });

    // Axe 접근성 테스트
    await page.evaluate(() => {
      // @ts-ignore
      return window.__axe__();
    });
  },
};

export default config;
```

```json
// package.json
{
  "scripts": {
    "test-storybook": "test-storybook",
    "test-storybook:ci": "concurrently -k -s first -n \"SB,TEST\" -c \"magenta,blue\" \"npm run build-storybook --quiet && npx http-server storybook-static --port 6006 --silent\" \"wait-on tcp:6006 && npm run test-storybook\""
  }
}
```

---

## 디자인 시스템과의 연계 전략

### 1. 디자인 토큰 시각화

{% raw %}
```tsx
// src/stories/Tokens.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';

const meta = {
  title: 'Design Tokens/Colors',
  parameters: {
    layout: 'fullscreen',
  },
} satisfies Meta;

export default meta;
type Story = StoryObj<typeof meta>;

// 색상 토큰 시각화
export const ColorPalette: Story = {
  render: () => {
    const colors = {
      primary: {
        50: '#eff6ff',
        100: '#dbeafe',
        200: '#bfdbfe',
        300: '#93c5fd',
        400: '#60a5fa',
        500: '#3b82f6',
        600: '#2563eb',
        700: '#1d4ed8',
        800: '#1e40af',
        900: '#1e3a8a',
      },
      // ... 다른 색상
    };

    return (
      <div className="p-8">
        <h1 className="text-3xl font-bold mb-8">Color Palette</h1>

        {Object.entries(colors).map(([name, shades]) => (
          <div key={name} className="mb-8">
            <h2 className="text-xl font-semibold mb-4 capitalize">{name}</h2>
            <div className="grid grid-cols-10 gap-2">
              {Object.entries(shades).map(([shade, hex]) => (
                <div key={shade} className="space-y-2">
                  <div
                    className="h-20 rounded-lg shadow-sm"
                    style={{ backgroundColor: hex }}
                  />
                  <div className="text-xs">
                    <div className="font-medium">{shade}</div>
                    <div className="text-gray-500">{hex}</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        ))}
      </div>
    );
  },
};
```
{% endraw %}

### 2. 타이포그래피 시스템

```tsx
// src/stories/Typography.stories.tsx
export const Typography: Story = {
  render: () => (
    <div className="p-8 space-y-8">
      <h1 className="text-4xl font-bold">Heading 1</h1>
      <h2 className="text-3xl font-bold">Heading 2</h2>
      <h3 className="text-2xl font-bold">Heading 3</h3>
      <h4 className="text-xl font-bold">Heading 4</h4>
      <p className="text-base">Body text - 16px / 1.5 line height</p>
      <p className="text-sm">Small text - 14px / 1.25 line height</p>
      <p className="text-xs">Extra small - 12px / 1 line height</p>

      <div className="space-y-2">
        <p className="font-light">Light (300)</p>
        <p className="font-normal">Normal (400)</p>
        <p className="font-medium">Medium (500)</p>
        <p className="font-semibold">Semibold (600)</p>
        <p className="font-bold">Bold (700)</p>
      </div>
    </div>
  ),
};
```

### 3. 컴포넌트 계층 구조

```text
src/
├── stories/
│   ├── Introduction.stories.tsx       # 소개 페이지
│   ├── GettingStarted.stories.tsx    # 시작 가이드
│   └── Changelog.stories.tsx         # 변경 이력
├── components/
│   ├── Primitives/                   # 기초 컴포넌트
│   │   ├── Button/
│   │   ├── Input/
│   │   └── Text/
│   ├── Composite/                    # 합성 컴포넌트
│   │   ├── Card/
│   │   ├── Form/
│   │   └── Table/
│   ├── Layout/                       # 레이아웃 컴포넌트
│   │   ├── Container/
│   │   ├── Grid/
│   │   └── Stack/
│   └── Patterns/                     # 패턴 (여러 컴포넌트 조합)
│       ├── LoginForm/
│       ├── DashboardHeader/
│       └── ProductCard/
└── tokens/
    ├── Colors.stories.tsx
    ├── Typography.stories.tsx
    └── Spacing.stories.tsx
```

### 4. 통합 소개 페이지

```tsx
// src/stories/Introduction.stories.tsx
export const Introduction = () => (
  <div className="p-10 max-w-4xl">
    <h1 className="text-4xl font-bold mb-4">
      Design System
    </h1>

    <p className="text-lg mb-8 text-gray-600">
      확장 가능하고 접근 가능한 React 컴포넌트 라이브러리입니다.
      Tailwind CSS 기반으로 일관된 디자인을 제공합니다.
    </p>

    <div className="grid md:grid-cols-2 gap-6 mb-8">
      <div className="border rounded-lg p-6">
        <h2 className="text-xl font-semibold mb-2">설치</h2>
        <pre className="bg-gray-100 p-4 rounded-md text-sm">
          npm install @your-org/design-system
        </pre>
      </div>

      <div className="border rounded-lg p-6">
        <h2 className="text-xl font-semibold mb-2">사용법</h2>
        <pre className="bg-gray-100 p-4 rounded-md text-sm">
{`import { Button } from '@your-org/ds';

<Button variant="primary">
  Click me
</Button>`}
        </pre>
      </div>
    </div>

    <h2 className="text-2xl font-semibold mb-4">컴포넌트</h2>
    <div className="grid md:grid-cols-3 gap-4">
      <a href="?path=/docs/components-button--docs" className="border rounded-lg p-4 hover:shadow-md transition-shadow">
        <h3 className="font-semibold mb-1">Button</h3>
        <p className="text-sm text-gray-600">다양한 변형의 버튼</p>
      </a>
      <a href="?path=/docs/components-input--docs" className="border rounded-lg p-4 hover:shadow-md transition-shadow">
        <h3 className="font-semibold mb-1">Input</h3>
        <p className="text-sm text-gray-600">폼 입력 필드</p>
      </a>
      <a href="?path=/docs/components-card--docs" className="border rounded-lg p-4 hover:shadow-md transition-shadow">
        <h3 className="font-semibold mb-1">Card</h3>
        <p className="text-sm text-gray-600">컨텐츠 컨테이너</p>
      </a>
    </div>

    <div className="mt-8 p-6 bg-blue-50 rounded-lg">
      <h2 className="text-xl font-semibold mb-2">관련 글</h2>
      <p className="text-sm text-gray-700">
        더 자세한 디자인 시스템 구축 방법은{' '}
        <a href="/posts/design-system-guide/" className="text-blue-600 hover:underline">
          디자인 시스템 구축 완벽 가이드
        </a>
        를 참고하세요.
      </p>
    </div>
  </div>
);
```

---

## 자주 묻는 질문 (FAQ)

### Q1. Storybook을 언제 도입해야 하나요?

**A:** 다음 상황에서 도입을 고려하세요:

**도입이 유리한 경우:**
- 재사용 가능한 컴포넌트 라이브러리 구축
- 디자인 시스템 관리
- 여러 프로젝트에서 같은 컴포넌트 사용
- 디자이너-개발자 협업 필요
- 컴포넌트 문서화 자동화

**도입이 불필요한 경우:**
- 컴포넌트가 10개 미만의 작은 프로젝트
- 일회성 프로젝트
- 빠른 프로토타입 단계

---

### Q2. CSF 2.0과 CSF 3.0의 차이는?

**A:** CSF 3.0이 더 간결하고 타입 안전합니다.

**CSF 2.0 (구버전):**
```tsx
export const Primary = () => <Button variant="primary">Click me</Button>;
Primary.args = {
  variant: 'primary',
};
```

**CSF 3.0 (신버전):**
```tsx
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Click me',
  },
};
```

**CSF 3.0 장점:**
- 더 간결한 문법
- 타입 안전성 향상
- Args 재사용 용이
- 자동 완성 지원

---

### Q3. Storybook이 느린데 어떻게 최적화하나요?

**A:** 다음 방법으로 성능을 개선할 수 있습니다:

**1. Vite 사용 (Webpack보다 빠름)**
```typescript
// .storybook/main.ts
framework: '@storybook/react-vite'
```

**2. Story 로딩 최적화**
```typescript
stories: [
  '../src/components/**/*.stories.tsx', // 특정 경로만
  // '../src/**/*.stories.tsx' 대신
]
```

**3. Addon 최소화**
```typescript
addons: [
  '@storybook/addon-essentials', // 필수만
  // 사용하지 않는 addon은 제거
]
```

**4. 자동 문서 비활성화 (필요한 경우)**
```typescript
docs: {
  autodocs: false, // 수동으로 'autodocs' 태그 추가
}
```

---

### Q4. 접근성 테스트에서 계속 오류가 나요.

**A:** 흔한 접근성 오류와 해결 방법:

**1. 색상 대비 부족**
```tsx
// ❌ Bad: 대비율 낮음
<button className="bg-gray-300 text-gray-400">Click</button>

// ✅ Good: 대비율 높음 (4.5:1 이상)
<button className="bg-blue-600 text-white">Click</button>
```

**2. 버튼 이름 없음**
```tsx
// ❌ Bad: 스크린 리더가 읽을 내용 없음
<button><X /></button>

// ✅ Good: aria-label 제공
<button aria-label="닫기"><X /></button>
```

**3. 폼 라벨 누락**
```tsx
// ❌ Bad: 라벨 없음
<input type="email" />

// ✅ Good: 라벨 연결
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

**4. 의미 없는 링크 텍스트**
```tsx
// ❌ Bad
<a href="/docs">여기를 클릭하세요</a>

// ✅ Good
<a href="/docs">문서 보기</a>
```

---

### Q5. Chromatic이 비싸지 않나요?

**A:** Chromatic의 가격 정책:

**무료 플랜:**
- 월 5,000 스냅샷
- 1명의 사용자
- 오픈소스 프로젝트는 무제한

**대안:**
1. **Percy** (무료 5,000 스냅샷/월)
2. **Playwright** (자체 호스팅 무료)
3. **BackstopJS** (완전 무료, 설정 복잡)

**비용 절감 팁:**
```tsx
// 불필요한 Story는 스냅샷 제외
export const Dynamic: Story = {
  parameters: {
    chromatic: { disableSnapshot: true },
  },
};

// 중요한 뷰포트만 테스트
export const Mobile: Story = {
  parameters: {
    chromatic: { viewports: [375] }, // 모바일만
  },
};
```

---

### Q6. 모든 컴포넌트에 Story를 만들어야 하나요?

**A:** 아니요, 선택적으로 만드세요.

**Story가 필요한 컴포넌트:**
- 재사용 가능한 컴포넌트 (Button, Input 등)
- 디자인 시스템 컴포넌트
- 복잡한 상태를 가진 컴포넌트
- 팀원들이 자주 사용하는 컴포넌트

**Story가 불필요한 컴포넌트:**
- 페이지 컴포넌트 (특정 라우트에만 사용)
- 일회성 컴포넌트
- 단순한 래퍼 컴포넌트
- 내부 구현 컴포넌트 (export되지 않음)

---

### Q7. 디자인 시스템과 Storybook의 관계는?

**A:** Storybook은 디자인 시스템의 **문서화 및 개발 도구**입니다.

```text
디자인 시스템 = 컴포넌트 + 디자인 토큰 + 가이드라인
                    ↓
              Storybook으로 문서화
                    ↓
         팀 전체가 쉽게 접근 가능
```

**디자인 시스템 없이 Storybook 사용:**
- 가능하지만 효과가 제한적
- 체계가 없으면 관리가 어려움

**추천 순서:**
1. [디자인 시스템 가이드](/posts/design-system-guide/)로 토큰과 컴포넌트 구축
2. Storybook으로 문서화 (이 글)
3. 팀에 배포 및 교육

---

## 결론

Storybook은 컴포넌트 개발과 문서화를 혁신적으로 개선하는 도구입니다. 초기 설정 비용이 있지만, 장기적으로 팀 생산성과 제품 품질을 크게 향상시킵니다.

### 핵심 요점

1. **격리된 개발**: 컴포넌트를 독립적으로 개발하고 테스트
2. **살아있는 문서**: 코드와 함께 업데이트되는 인터랙티브 문서
3. **자동화된 테스트**: 접근성, 시각적 회귀 테스트 자동화
4. **팀 협업**: 디자이너-개발자 간 공통 언어 제공
5. **디자인 시스템 연계**: 중앙 집중식 컴포넌트 관리

### 시작 체크리스트

**1단계: 설치 및 설정 (1시간)**
- [x] Storybook 설치
- [x] 첫 Story 작성
- [x] Controls addon 테스트

**2단계: 핵심 컴포넌트 문서화 (1-2일)**
- [x] Button, Input, Card 등 기초 컴포넌트
- [x] ArgTypes와 Controls 설정
- [x] Actions addon으로 이벤트 로깅

**3단계: 고급 기능 (1주)**
- [x] MDX로 상세 문서 작성
- [x] a11y addon으로 접근성 검증
- [x] Chromatic으로 시각적 테스트

**4단계: CI/CD 통합 (1-2일)**
- [x] GitHub Actions 설정
- [x] 자동 배포 구성
- [x] PR 프리뷰 생성

**5단계: 팀 온보딩 (진행 중)**
- [x] 팀 교육 및 가이드 작성
- [x] 피드백 수집 및 개선
- [x] 정기적인 업데이트

### 다음 단계

Storybook 구축을 마쳤다면:

1. **[디자인 시스템 구축 가이드](/posts/design-system-guide/)**로 체계적인 컴포넌트 라이브러리 구축
2. **프론트엔드 테스팅 가이드**로 단위 테스트와 E2E 테스트 추가
3. **컴포넌트 최적화**로 성능 개선

이제 Storybook으로 컴포넌트를 효율적으로 개발하고, 팀 전체가 쉽게 접근할 수 있는 문서를 제공할 수 있습니다!

---

## 참고 자료

### 공식 문서
- [Storybook Documentation](https://storybook.js.org/docs) - 공식 문서
- [Component Story Format (CSF)](https://storybook.js.org/docs/api/csf) - CSF 3.0 스펙
- [Storybook Addons](https://storybook.js.org/addons) - 공식 Addon 목록
- [Test Runner](https://github.com/storybookjs/test-runner) - 자동 테스트

### 접근성
- [Storybook a11y Addon](https://storybook.js.org/addons/@storybook/addon-a11y) - 접근성 테스트
- [Axe Core](https://github.com/dequelabs/axe-core) - 접근성 검사 엔진
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/) - 웹 접근성 표준
- [ARIA Practices](https://www.w3.org/WAI/ARIA/apg/) - ARIA 패턴 가이드

### 시각적 테스트
- [Chromatic](https://www.chromatic.com/) - 시각적 회귀 테스트 (Storybook 제작사)
- [Percy](https://percy.io/) - 시각적 테스트 대안
- [Playwright](https://playwright.dev/) - E2E 테스트 프레임워크
- [BackstopJS](https://github.com/garris/BackstopJS) - 오픈소스 시각적 테스트

### 디자인 시스템 예제
- [Chakra UI Storybook](https://storybook.chakra-ui.com/) - Chakra UI 예제
- [Shopify Polaris](https://polaris.shopify.com/) - Shopify 디자인 시스템
- [Material-UI Storybook](https://v4.mui.com/components/app-bar/) - Material-UI 예제
- [GitHub Primer](https://primer.style/) - GitHub 디자인 시스템
- [IBM Carbon](https://www.carbondesignsystem.com/) - IBM 디자인 시스템

### 학습 자료
- [Storybook Tutorials](https://storybook.js.org/tutorials/) - 공식 튜토리얼
- [Component Encyclopedia](https://storybook.js.org/showcase) - 실제 사용 사례
- [Storybook Blog](https://storybook.js.org/blog) - 최신 소식 및 팁
- [Design Systems for Developers](https://storybook.js.org/tutorials/design-systems-for-developers/) - 디자인 시스템 구축 튜토리얼

### 도구
- [Figma](https://www.figma.com/) - 디자인 협업 도구
- [Zeplin](https://zeplin.io/) - 디자인 핸드오프
- [Supernova](https://www.supernova.io/) - 디자인 시스템 자동화

### 커뮤니티
- [Storybook Discord](https://discord.gg/storybook) - 공식 디스코드
- [GitHub Discussions](https://github.com/storybookjs/storybook/discussions) - 질문 및 토론
- [Stack Overflow](https://stackoverflow.com/questions/tagged/storybook) - Q&A

### 관련 글
- [디자인 시스템 구축 완벽 가이드](/posts/design-system-guide/) - Storybook과 Tailwind로 디자인 시스템 구축
- React Server Components 완벽 가이드
- 프론트엔드 테스팅 완벽 가이드
