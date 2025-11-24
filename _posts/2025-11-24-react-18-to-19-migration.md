---
title: "React 18에서 19로 마이그레이션하기 - 실전 가이드"
description: "React 18에서 19로 안전하게 마이그레이션하는 방법을 단계별로 알아봅니다. Breaking changes, API 변경사항, 그리고 실무에서 마주치는 이슈 해결책까지 상세히 다룹니다."
date: 2025-11-24 10:00:00 +0900
categories: [Frontend, React]
tags: [react, react-19, migration, upgrade, breaking-changes, frontend]
---

## 서론: React 19의 주요 변화점과 마이그레이션 필요성

2024년 12월 5일, React 팀은 **React 19**를 정식 릴리스했습니다. React 18 출시 이후 약 2년 만의 메이저 업데이트로, 비동기 작업 처리, 폼 핸들링, 개발자 경험에서 혁신적인 개선이 이루어졌습니다.

### 왜 마이그레이션해야 하는가?

React 19로 마이그레이션하면 다음과 같은 이점을 얻을 수 있습니다:

1. **코드 간소화**: `forwardRef` 제거, PropTypes 대신 TypeScript, Actions 패턴으로 보일러플레이트 감소
2. **성능 향상**: Server Components 최적화, 리소스 프리로딩, 번들 크기 감소
3. **개발자 경험**: 향상된 에러 메시지, 새로운 Hooks(`use()`, `useActionState()`, `useOptimistic()`)
4. **미래 대비**: React의 장기 로드맵에 맞춘 아키텍처 준비

### React 19 핵심 변경사항 요약

| 영역 | 변경사항 |
|------|----------|
| **새로운 Hooks** | `use()`, `useActionState()`, `useOptimistic()`, `useFormStatus()` |
| **API 제거** | PropTypes, defaultProps(함수형), String refs, Legacy Context |
| **단순화** | `forwardRef` 불필요, `<Context>` 직접 사용 |
| **Actions** | 폼 제출 및 비동기 작업의 새로운 패러다임 |
| **Server Components** | 안정화 및 성능 개선 |

---

## 사전 준비

마이그레이션을 시작하기 전에 충분한 준비가 필요합니다. 급하게 진행하면 예상치 못한 문제에 직면할 수 있습니다.

### 1. 현재 버전 확인

```bash
# package.json에서 React 버전 확인
npm list react react-dom

# 또는 직접 확인
cat package.json | grep -E '"react"|"react-dom"'
```

**예상 출력:**

```
react@18.2.0
react-dom@18.2.0
```

### 2. Node.js 버전 확인

React 19는 Node.js 18 이상을 권장합니다.

```bash
node --version
# v18.0.0 이상 권장
```

### 3. 의존성 호환성 체크

React 19와 호환되는지 주요 라이브러리를 확인하세요.

```bash
# 모든 React 관련 의존성 확인
npm ls react
```

**주요 라이브러리 호환성 현황:**

| 라이브러리 | 호환 버전 | 비고 |
|------------|-----------|------|
| React Router | v6.4+ | React 19 완전 지원 |
| React Redux | v8+ | Hooks API 완전 지원 |
| Next.js | v15+ | React 19 기본 지원 |
| Material-UI | v5.14+ | 공식 호환 |
| Ant Design | v5.11+ | 공식 호환 |
| Zustand | v4.4+ | 호환 |
| TanStack Query | v5+ | 호환 |

### 4. 프로젝트 백업

```bash
# Git 브랜치 생성
git checkout -b feature/react-19-migration

# 현재 상태 커밋
git add .
git commit -m "chore: React 19 마이그레이션 시작 전 상태"
```

### 5. 테스트 커버리지 확인

마이그레이션 후 검증을 위해 충분한 테스트 커버리지가 필요합니다.

```bash
# Jest 테스트 커버리지 확인
npm test -- --coverage
```

테스트 커버리지가 70% 미만이라면, 핵심 기능에 대한 테스트를 먼저 작성하는 것을 권장합니다.

---

## Breaking Changes 체크리스트

React 19 마이그레이션 시 반드시 수정해야 할 항목들입니다. 아래 체크리스트를 순서대로 확인하세요.

### 필수 수정 항목

- [ ] **PropTypes 제거**: TypeScript 또는 런타임 검증 라이브러리로 대체
- [ ] **defaultProps 제거**: ES6 기본 매개변수로 변경
- [ ] **String refs 제거**: `useRef` 또는 콜백 ref로 변경
- [ ] **Legacy Context 제거**: 새로운 Context API(`createContext`)로 변경
- [ ] **ReactDOM.render 제거**: `createRoot` 사용
- [ ] **ReactDOM.hydrate 제거**: `hydrateRoot` 사용
- [ ] **react-dom/test-utils 제거**: `@testing-library/react` 사용

### 권장 수정 항목

- [ ] **forwardRef 제거**: ref를 일반 props로 전달
- [ ] **useFormState 제거**: `useActionState`로 변경
- [ ] **Context.Provider 간소화**: `<Context>` 직접 사용

### 체크 스크립트

프로젝트에서 Breaking Changes가 있는지 빠르게 확인할 수 있는 명령어:

```bash
# PropTypes 사용 확인
grep -r "PropTypes\." --include="*.tsx" --include="*.jsx" src/

# defaultProps 사용 확인
grep -r "\.defaultProps" --include="*.tsx" --include="*.jsx" src/

# String refs 확인
grep -r 'ref="' --include="*.tsx" --include="*.jsx" src/

# ReactDOM.render 확인
grep -r "ReactDOM.render" --include="*.tsx" --include="*.jsx" src/

# Legacy Context 확인
grep -r "getChildContext\|childContextTypes\|contextTypes" --include="*.tsx" --include="*.jsx" src/
```

---

## 단계별 마이그레이션 가이드

### Step 1: React 18.3으로 먼저 업그레이드

React 팀은 **React 18.3**을 중간 단계로 권장합니다. React 18.3은 18.2와 기능적으로 동일하지만, React 19에서 제거될 기능에 대한 **deprecation 경고**를 제공합니다.

```bash
# React 18.3으로 업그레이드
npm install react@18.3.1 react-dom@18.3.1
```

업그레이드 후 앱을 실행하고 콘솔에서 경고 메시지를 확인하세요:

```bash
npm run dev
# 또는
npm start
```

**예상되는 경고 메시지:**

```
Warning: componentWillMount has been renamed, and is not recommended for use.
Warning: Using defaultProps on function components is deprecated.
Warning: String refs are deprecated.
```

이 경고들을 모두 해결한 후 React 19로 진행합니다.

### Step 2: React 및 ReactDOM 19로 업그레이드

```bash
# React 19 정식 버전 설치
npm install react@19 react-dom@19

# TypeScript 사용 시 타입도 업데이트
npm install --save-dev @types/react@19 @types/react-dom@19
```

**package.json 확인:**

```jsonc
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0"
  }
}
```

### Step 3: 새로운 JSX Transform 적용

React 19는 새로운 JSX Transform을 사용합니다. 대부분의 최신 빌드 도구(Vite, Next.js, CRA 5+)는 이미 지원합니다.

**Babel 설정 확인 (babel.config.js):**

```javascript
module.exports = {
  presets: [
    ['@babel/preset-react', {
      runtime: 'automatic' // 'classic' 대신 'automatic' 사용
    }]
  ]
};
```

**TypeScript 설정 확인 (tsconfig.json):**

```jsonc
{
  "compilerOptions": {
    "jsx": "react-jsx", // "react" 대신 "react-jsx" 사용
    "jsxImportSource": "react"
  }
}
```

새로운 JSX Transform을 사용하면 더 이상 파일 상단에 `import React from 'react'`가 필요하지 않습니다.

```tsx
// Before: React import 필요
import React from 'react';

function App() {
  return <div>Hello</div>;
}

// After: React import 불필요
function App() {
  return <div>Hello</div>;
}
```

### Step 4: Concurrent 기능 활성화 확인

React 18에서 이미 `createRoot`를 사용하고 있다면 Concurrent 기능이 활성화된 상태입니다.

**Before (React 17 방식 - 작동 안함):**

```tsx
// index.tsx - 더 이상 지원되지 않음
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));
```

**After (React 18/19 방식):**

```tsx
// index.tsx
import { createRoot } from 'react-dom/client';

const container = document.getElementById('root');
const root = createRoot(container!);
root.render(<App />);
```

**SSR의 경우:**

```tsx
// Before
import ReactDOM from 'react-dom';
ReactDOM.hydrate(<App />, document.getElementById('root'));

// After
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document.getElementById('root')!, <App />);
```

### Step 5: Suspense 및 에러 바운더리 업데이트

React 19에서 Suspense는 더욱 강화되었습니다. fallback이 실제로 마운트되어 CSS 애니메이션이 정상 작동합니다.

```tsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <ErrorBoundary
      fallback={<div>오류가 발생했습니다.</div>}
      onReset={() => window.location.reload()}
    >
      <Suspense fallback={<LoadingSkeleton />}>
        <MainContent />
      </Suspense>
    </ErrorBoundary>
  );
}

function LoadingSkeleton() {
  return (
    <div className="skeleton animate-pulse">
      로딩 중...
    </div>
  );
}
```

---

## 주요 API 변경 사항

### 1. forwardRef에서 ref prop 직접 사용으로

React 19의 가장 환영받는 변화 중 하나입니다. 더 이상 `forwardRef`로 컴포넌트를 감쌀 필요가 없습니다.

**Before (React 18):**

```tsx
import { forwardRef, useRef } from 'react';

interface InputProps {
  label: string;
  type?: string;
}

// forwardRef로 감싸야 함
const Input = forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, type = 'text' }, ref) {
    return (
      <label>
        {label}
        <input ref={ref} type={type} />
      </label>
    );
  }
);

function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <form>
      <Input ref={inputRef} label="이름" />
      <button onClick={() => inputRef.current?.focus()}>
        포커스
      </button>
    </form>
  );
}
```

**After (React 19):**

```tsx
import { useRef } from 'react';

interface InputProps {
  ref?: React.Ref<HTMLInputElement>;
  label: string;
  type?: string;
}

// forwardRef 없이 ref를 props로 직접 받음
function Input({ ref, label, type = 'text' }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} type={type} />
    </label>
  );
}

function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <form>
      <Input ref={inputRef} label="이름" />
      <button onClick={() => inputRef.current?.focus()}>
        포커스
      </button>
    </form>
  );
}
```

**마이그레이션 팁:**
- 기존 `forwardRef` 코드는 계속 작동합니다
- 새로운 컴포넌트는 새 방식으로 작성
- 시간이 날 때 점진적으로 마이그레이션

### 2. useContext 최적화

React 19에서는 `<Context.Provider>` 대신 `<Context>`를 직접 렌더링할 수 있습니다.

**Before (React 18):**

```tsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Main />
    </ThemeContext.Provider>
  );
}
```

**After (React 19):**

```tsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext value="dark">
      <Main />
    </ThemeContext>
  );
}
```

### 3. use() 훅 도입

`use()`는 React 19의 혁신적인 새 API입니다. 다른 Hooks와 달리 조건문 내에서 호출할 수 있으며, Promise와 Context를 읽을 수 있습니다.

**Promise 읽기:**

```tsx
import { use, Suspense } from 'react';

// 서버에서 Promise 생성
async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// 클라이언트 컴포넌트에서 use() 사용
function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise); // Suspense와 자동 통합

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 사용
function App() {
  const userPromise = fetchUser('123');

  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**조건부 Context 읽기:**

{% raw %}
```tsx
import { use } from 'react';
import { ThemeContext } from './ThemeContext';

function ThemedButton({ showTheme }: { showTheme: boolean }) {
  // 조건문 내에서 use() 호출 가능!
  if (showTheme) {
    const theme = use(ThemeContext);
    return (
      <button style={{ background: theme.primary }}>
        테마 적용됨
      </button>
    );
  }

  return <button>기본 버튼</button>;
}
```
{% endraw %}

### 4. Actions와 useActionState

Actions는 폼 제출 및 비동기 작업의 새로운 패러다임입니다. `useActionState`는 이전의 `useFormState`를 대체합니다.

**Before (React 18):**

```tsx
import { useState } from 'react';

function ContactForm() {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState(false);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();

    setIsPending(true);
    setError(null);
    setSuccess(false);

    const formData = new FormData(e.currentTarget);

    try {
      await fetch('/api/contact', {
        method: 'POST',
        body: formData
      });
      setSuccess(true);
      e.currentTarget.reset();
    } catch (err) {
      setError(err instanceof Error ? err.message : '오류 발생');
    } finally {
      setIsPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" disabled={isPending} required />
      <textarea name="message" disabled={isPending} required />

      <button type="submit" disabled={isPending}>
        {isPending ? '전송 중...' : '전송'}
      </button>

      {error && <div className="error">{error}</div>}
      {success && <div className="success">전송 완료!</div>}
    </form>
  );
}
```

**After (React 19):**

```tsx
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

interface FormState {
  success: boolean;
  error: string | null;
}

async function contactAction(
  prevState: FormState,
  formData: FormData
): Promise<FormState> {
  try {
    await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify({
        email: formData.get('email'),
        message: formData.get('message')
      })
    });
    return { success: true, error: null };
  } catch (err) {
    return {
      success: false,
      error: err instanceof Error ? err.message : '오류 발생'
    };
  }
}

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '전송 중...' : '전송'}
    </button>
  );
}

function ContactForm() {
  const [state, formAction, isPending] = useActionState(contactAction, {
    success: false,
    error: null
  });

  return (
    <form action={formAction}>
      <input name="email" type="email" disabled={isPending} required />
      <textarea name="message" disabled={isPending} required />

      <SubmitButton />

      {state.error && <div className="error">{state.error}</div>}
      {state.success && <div className="success">전송 완료!</div>}
    </form>
  );
}
```

### 5. useOptimistic으로 낙관적 업데이트

```tsx
import { useOptimistic, useTransition } from 'react';

interface Todo {
  id: number;
  text: string;
  pending?: boolean;
}

function TodoList({ initialTodos }: { initialTodos: Todo[] }) {
  const [todos, setTodos] = useState(initialTodos);
  const [isPending, startTransition] = useTransition();

  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state: Todo[], newTodo: Todo) => [...state, { ...newTodo, pending: true }]
  );

  async function handleAddTodo(formData: FormData) {
    const text = formData.get('text') as string;
    const newTodo: Todo = { id: Date.now(), text };

    startTransition(async () => {
      // 즉시 UI 업데이트
      addOptimisticTodo(newTodo);

      // 실제 서버 요청
      const savedTodo = await saveTodo(newTodo);
      setTodos((prev) => [...prev, savedTodo]);
    });
  }

  return (
    <div>
      <form action={handleAddTodo}>
        <input name="text" placeholder="할 일 입력" required />
        <button type="submit">추가</button>
      </form>

      <ul>
        {optimisticTodos.map((todo) => (
          <li
            key={todo.id}
            style={{ opacity: todo.pending ? 0.5 : 1 }}
          >
            {todo.text}
            {todo.pending && ' (저장 중...)'}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 일반적인 마이그레이션 이슈와 해결책

### 이슈 1: PropTypes 관련 에러

**문제:**

```
Error: prop-types is not defined
```

**해결책:**

PropTypes를 TypeScript 인터페이스로 변환:

```tsx
// Before
import PropTypes from 'prop-types';

function Button({ variant, size, children }) {
  return <button className={`btn-${variant} btn-${size}`}>{children}</button>;
}

Button.propTypes = {
  variant: PropTypes.oneOf(['primary', 'secondary']).isRequired,
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  children: PropTypes.node.isRequired
};

Button.defaultProps = {
  size: 'medium'
};

// After
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  children: React.ReactNode;
}

function Button({ variant, size = 'medium', children }: ButtonProps) {
  return <button className={`btn-${variant} btn-${size}`}>{children}</button>;
}
```

### 이슈 2: ref 콜백 반환 타입 에러

**문제:**

```
Type '() => void' is not assignable to type 'void'.
```

**원인:**
React 19에서 ref 콜백은 클린업 함수를 반환할 수 있게 되었습니다.

**해결책:**

```tsx
// Before - 암묵적 반환으로 인한 타입 에러
<div ref={(node) => (node?.focus())} />

// After - 명시적으로 void 반환
<div ref={(node) => { node?.focus(); }} />

// 또는 클린업 함수 활용
<div ref={(node) => {
  if (node) {
    node.focus();
    return () => {
      // 클린업 로직
    };
  }
}} />
```

### 이슈 3: Hydration 불일치 에러

**문제:**

```
Error: Hydration failed because the initial UI does not match what was rendered on the server.
```

**해결책:**

서버와 클라이언트에서 동일한 결과를 렌더링하도록 수정:

```tsx
// Before - Hydration 불일치
function Clock() {
  return <div>{new Date().toTimeString()}</div>;
}

// After - 클라이언트에서만 렌더링
function Clock() {
  const [time, setTime] = useState('');

  useEffect(() => {
    setTime(new Date().toTimeString());
    const interval = setInterval(() => {
      setTime(new Date().toTimeString());
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  // 서버에서는 빈 값, 클라이언트에서 실제 시간
  return <div>{time || '로딩 중...'}</div>;
}

// 또는 suppressHydrationWarning 사용 (권장하지 않음)
function Clock() {
  return (
    <div suppressHydrationWarning>
      {new Date().toTimeString()}
    </div>
  );
}
```

### 이슈 4: 써드파티 라이브러리 호환성 문제

**문제:**

```
Error: Cannot find module 'react' or React version mismatch
```

**해결책:**

1. 라이브러리 버전 업데이트:

```bash
# 주요 라이브러리 최신 버전으로 업데이트
npm update react-router-dom @tanstack/react-query zustand
```

2. peer dependency 확인:

```bash
npm ls react
```

3. 호환되지 않는 라이브러리 대체:

```bash
# react-helmet 대신 react-helmet-async 사용
npm uninstall react-helmet
npm install react-helmet-async
```

### 이슈 5: useFormState에서 useActionState로 변경

**문제:**

```
Warning: useFormState has been renamed to useActionState.
```

**해결책:**

```tsx
// Before
import { useFormState } from 'react-dom';

function Form() {
  const [state, formAction] = useFormState(action, initialState);
  // ...
}

// After
import { useActionState } from 'react';

function Form() {
  const [state, formAction, isPending] = useActionState(action, initialState);
  // isPending이 세 번째 반환값으로 추가됨
}
```

### 이슈 6: act() 경고 메시지

**문제:**

```
Warning: An update to Component inside a test was not wrapped in act(...)
```

**해결책:**

`@testing-library/react` 최신 버전 사용:

```bash
npm install --save-dev @testing-library/react@latest
```

```tsx
// Before
import { act } from 'react-dom/test-utils';

act(() => {
  render(<Component />);
});

// After
import { render, screen, waitFor } from '@testing-library/react';

// act가 자동으로 적용됨
render(<Component />);

// 비동기 작업은 waitFor 사용
await waitFor(() => {
  expect(screen.getByText('완료')).toBeInTheDocument();
});
```

---

## 테스트 및 검증

마이그레이션 후 철저한 검증이 필요합니다. 아래 체크리스트를 순서대로 확인하세요.

### 1. 빌드 테스트

```bash
# 프로덕션 빌드 테스트
npm run build

# 빌드 에러가 없는지 확인
# 번들 크기 비교 (React 19에서 약 8-12% 감소 예상)
```

### 2. 단위 테스트 실행

```bash
# 전체 테스트 실행
npm test

# 커버리지 포함 실행
npm test -- --coverage

# 특정 파일 테스트
npm test -- --testPathPattern="Component.test"
```

### 3. E2E 테스트 실행

```bash
# Playwright 또는 Cypress E2E 테스트
npm run test:e2e
```

### 4. 수동 테스트 체크리스트

다음 기능들을 수동으로 확인하세요:

- [ ] **라우팅**: 모든 페이지 이동이 정상 작동하는가?
- [ ] **폼 제출**: 모든 폼이 정상적으로 제출되는가?
- [ ] **인증**: 로그인/로그아웃이 정상 작동하는가?
- [ ] **데이터 로딩**: API 호출 및 데이터 표시가 정상인가?
- [ ] **에러 처리**: 에러 발생 시 적절한 메시지가 표시되는가?
- [ ] **반응형**: 모바일/태블릿/데스크톱에서 정상 작동하는가?

### 5. 성능 테스트

```bash
# Lighthouse CI 실행 (선택사항)
npx lighthouse http://localhost:3000 --output=json --output-path=./lighthouse-report.json
```

**확인할 성능 지표:**

| 지표 | React 18 기준 | React 19 예상 |
|------|---------------|---------------|
| First Contentful Paint | 1.5s | 1.3s (13% 개선) |
| Time to Interactive | 3.0s | 2.4s (20% 개선) |
| Bundle Size | 150KB | 135KB (10% 감소) |

### 6. 콘솔 에러 확인

개발자 도구 콘솔에서 다음 항목을 확인하세요:

```bash
# 확인할 경고/에러
- React 관련 경고 메시지가 없는지
- Hydration 관련 에러가 없는지
- Deprecated API 사용 경고가 없는지
```

---

## 결론: 마이그레이션 체크리스트 요약

### 마이그레이션 전 준비

- [ ] React 18.3으로 먼저 업그레이드하여 deprecation 경고 확인
- [ ] 모든 경고 메시지 해결
- [ ] 테스트 커버리지 확인 (최소 70% 권장)
- [ ] Git 브랜치 생성 및 백업

### 핵심 변경사항 적용

- [ ] `npm install react@19 react-dom@19`
- [ ] TypeScript 타입 업데이트: `@types/react@19 @types/react-dom@19`
- [ ] PropTypes를 TypeScript 인터페이스로 변환
- [ ] defaultProps를 ES6 기본 매개변수로 변환
- [ ] forwardRef를 ref props로 변환 (점진적)
- [ ] ReactDOM.render를 createRoot로 변환
- [ ] useFormState를 useActionState로 변환

### 새로운 기능 활용 (선택)

- [ ] Actions 패턴 도입 검토
- [ ] use() 훅 활용 검토
- [ ] useOptimistic 활용 검토
- [ ] Document Metadata 기능 활용 검토

### 검증

- [ ] 프로덕션 빌드 성공
- [ ] 단위 테스트 통과
- [ ] E2E 테스트 통과
- [ ] 수동 테스트 완료
- [ ] 콘솔 에러/경고 없음
- [ ] 성능 지표 확인

### Codemod 활용

마이그레이션을 자동화하려면 React 팀이 제공하는 codemod를 사용하세요:

```bash
# 전체 마이그레이션 레시피
npx codemod react/19/migration-recipe

# TypeScript 타입 마이그레이션
npx types-react-codemod@latest preset-19 ./src

# 개별 codemod
npx codemod react/19/replace-reactdom-render
npx codemod react/19/replace-default-props
npx codemod react/19/replace-string-ref
```

---

## 참고 자료

### 공식 문서
- [React 19 공식 릴리스 노트](https://react.dev/blog/2024/12/05/react-19)
- [React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)
- [useActionState 문서](https://react.dev/reference/react/useActionState)
- [use() API 문서](https://react.dev/reference/react/use)
- [useOptimistic 문서](https://react.dev/reference/react/useOptimistic)

### 마이그레이션 도구
- [React 19 Codemod](https://codemod.com/registry/react-19-migration-recipe)
- [TypeScript Types Codemod](https://github.com/eps1lon/types-react-codemod)

### 추가 학습 자료
- [React 19 Actions 패턴 이해하기](https://react.dev/reference/rsc/server-actions)
- [Server Components 가이드](https://react.dev/reference/rsc/server-components)
