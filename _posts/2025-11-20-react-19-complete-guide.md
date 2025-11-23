---
title: "React 19 완벽 가이드 - Actions, use() 훅, Server Components 개선사항 총정리"
date: 2025-11-20 14:00:00 +0900
categories: [Frontend, React]
tags: [react, react19, actions, use-hook, server-components, useoptimistic, useactionstate, frontend, 리액트, 리액트19]
description: "2024년 12월 정식 릴리스된 React 19의 모든 것. Actions 패턴, use() 훅, useOptimistic, Server Components 개선사항, 실무 마이그레이션 가이드까지 완벽 정리"
---

## React 19 정식 릴리스 소개

2024년 12월 5일, React 팀은 **React 19** 정식 버전을 릴리스했습니다. React 18이 출시된 지 2년 만의 메이저 업데이트입니다. React 19는 2024년 4월 Release Candidate(RC) 버전 발표 이후 약 8개월간의 테스트와 피드백을 거쳐 안정화된 버전입니다.

### React 19의 주요 목표와 철학

React 19는 다음 세 가지 핵심 목표를 중심으로 개발되었습니다:

1. **비동기 작업의 단순화**: Actions 패턴을 도입하여 폼 제출, 데이터 fetching, 에러 핸들링을 더욱 직관적으로 처리
2. **사용자 경험 향상**: 낙관적 업데이트(Optimistic Updates)와 자동 상태 관리로 더 빠르고 반응적인 UI 구현
3. **개발자 경험 개선**: 복잡한 보일러플레이트 코드 제거 및 Server Components 통합 강화

### 전체적인 변경사항 개요

React 19의 주요 변경사항은 크게 세 가지 영역으로 나눌 수 있습니다:

**새로운 기능 추가:**
- Actions를 통한 비동기 작업 자동 관리
- `use()`, `useActionState()`, `useOptimistic()`, `useFormStatus()` 등 새로운 Hooks
- ref를 props로 직접 전달 가능
- Document Metadata(title, meta, link) 컴포넌트 내부 관리
- Server Components 성능 및 기능 개선

**개발자 경험 개선:**
- `forwardRef` 불필요 (ref를 일반 props처럼 사용)
- Context Provider 간소화 (`<Context>` 직접 사용)
- 향상된 에러 핸들링과 Hydration 오류 메시지

**성능 최적화:**
- 리소스 프리로딩 API (`preload`, `prefetchDNS`, `preconnect` 등)
- 스타일시트 우선순위 관리
- 비동기 스크립트 중복 제거

---

## 새로운 Hooks

React 19는 비동기 작업과 폼 처리를 더욱 쉽게 만드는 4개의 새로운 Hooks를 도입했습니다.

### 1. use() 훅 - Promise와 Context를 읽는 혁신적인 훅

`use()`는 기존 Hooks의 규칙을 깬 **혁신적인 API**입니다. 조건문과 반복문 내에서도 호출할 수 있으며, Promise와 Context의 값을 읽을 수 있습니다.

#### 기본 개념

```tsx
import { use } from 'react';

function Component({ resourcePromise }) {
  // Promise가 resolve될 때까지 Suspend
  const data = use(resourcePromise);

  return <div>{data}</div>;
}
```

#### 주요 특징

1. **조건부 호출 가능**: 다른 Hooks와 달리 `if`문 내부에서 사용 가능
2. **Promise 처리**: Suspense와 자동 통합되어 로딩 상태 관리
3. **Context 읽기**: `useContext`보다 유연한 Context 접근

#### Promise 읽기 예제

서버에서 클라이언트로 데이터 스트리밍하는 패턴:

```tsx
// Server Component
async function ServerComponent() {
  const messagePromise = fetchMessage();

  return (
    <Suspense fallback={<MessageSkeleton />}>
      <Message messagePromise={messagePromise} />
    </Suspense>
  );
}

// Client Component
'use client';
import { use } from 'react';

function Message({ messagePromise }) {
  // Promise가 resolve될 때까지 Suspense fallback 표시
  const message = use(messagePromise);

  return (
    <div className="message">
      <h2>{message.title}</h2>
      <p>{message.content}</p>
    </div>
  );
}
```

#### 조건부 Context 읽기

`useContext`와 달리 조건문 내부에서 사용 가능:

{% raw %}
```tsx
import { use } from 'react';
import { ThemeContext } from './ThemeContext';

function Button({ primary }) {
  // 조건부로 Context 읽기 가능!
  if (primary) {
    const theme = use(ThemeContext);
    return <button style={{ background: theme.primaryColor }}>Primary</button>;
  }

  return <button>Secondary</button>;
}
```
{% endraw %}

#### Error Boundary와 함께 사용

```tsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function DataDisplay({ dataPromise }) {
  const data = use(dataPromise);
  return <div>{data}</div>;
}

function App() {
  const dataPromise = fetchData();

  return (
    <ErrorBoundary fallback={<div>데이터 로딩 실패</div>}>
      <Suspense fallback={<div>로딩 중...</div>}>
        <DataDisplay dataPromise={dataPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

#### 주의사항

- **try-catch 블록 내에서 사용 불가**: Error Boundary를 대신 사용해야 합니다
- **Server Component에서는 async/await 선호**: `use()`는 주로 클라이언트 컴포넌트에서 사용
- **클라이언트에서 Promise 생성 지양**: 매 렌더링마다 재생성되므로 서버에서 생성 후 전달

```tsx
// ❌ 나쁜 예: 클라이언트에서 Promise 생성
function BadComponent() {
  // 매 렌더링마다 새로운 Promise 생성!
  const dataPromise = fetchData();
  const data = use(dataPromise);
  return <div>{data}</div>;
}

// ✅ 좋은 예: 서버에서 Promise 생성
function ServerParent() {
  const dataPromise = fetchData(); // 한 번만 생성
  return <ClientChild dataPromise={dataPromise} />;
}

'use client';
function ClientChild({ dataPromise }) {
  const data = use(dataPromise); // 안정적인 Promise
  return <div>{data}</div>;
}
```

---

### 2. useOptimistic() - 낙관적 UI 업데이트

`useOptimistic()`는 서버 응답을 기다리지 않고 즉시 UI를 업데이트하여 더 빠른 사용자 경험을 제공합니다.

#### 기본 사용법

```tsx
import { useOptimistic } from 'react';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );

  async function handleAddTodo(formData) {
    const newTodo = { id: Date.now(), text: formData.get('text') };

    // 즉시 UI 업데이트 (낙관적)
    addOptimisticTodo(newTodo);

    // 실제 서버 요청
    try {
      await saveTodo(newTodo);
    } catch (error) {
      // 실패 시 자동으로 원래 상태로 복원
      console.error('Todo 추가 실패:', error);
    }
  }

  return (
    <div>
      <form action={handleAddTodo}>
        <input name="text" placeholder="할 일 입력" />
        <button>추가</button>
      </form>

      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### 실전 예제: 좋아요 버튼

```tsx
import { useOptimistic } from 'react';

function LikeButton({ postId, initialLikes, isLiked }) {
  const [optimisticState, setOptimisticState] = useOptimistic(
    { likes: initialLikes, isLiked },
    (state, action) => {
      if (action === 'like') {
        return { likes: state.likes + 1, isLiked: true };
      }
      return { likes: state.likes - 1, isLiked: false };
    }
  );

  async function handleLike() {
    const action = optimisticState.isLiked ? 'unlike' : 'like';

    // 즉시 UI 업데이트
    setOptimisticState(action);

    // 실제 API 호출
    try {
      await fetch(`/api/posts/${postId}/${action}`, { method: 'POST' });
    } catch (error) {
      // 실패 시 자동 롤백
      console.error('좋아요 처리 실패:', error);
    }
  }

  return (
    <button onClick={handleLike}>
      {optimisticState.isLiked ? '❤️' : '🤍'} {optimisticState.likes}
    </button>
  );
}
```

#### 핵심 특징

- **즉각적인 피드백**: 서버 응답을 기다리지 않고 UI 즉시 업데이트
- **자동 롤백**: 요청 실패 시 원래 상태로 자동 복원
- **Error Boundary 통합**: 에러 발생 시 자동으로 처리

---

### 3. useActionState() - 폼 액션 상태 관리

`useActionState()`는 비동기 폼 제출의 상태를 관리하는 훅입니다. 이전의 `useFormState`를 대체합니다.

#### 기본 구조

```tsx
import { useActionState } from 'react';

function Component() {
  const [state, formAction, isPending] = useActionState(
    actionFunction,
    initialState,
    permalink
  );

  return (
    <form action={formAction}>
      {/* 폼 내용 */}
    </form>
  );
}
```

#### 실전 예제: 회원가입 폼

```tsx
import { useActionState } from 'react';

async function signupAction(prevState, formData) {
  const email = formData.get('email');
  const password = formData.get('password');

  try {
    await registerUser(email, password);
    return { success: true, message: '회원가입 성공!' };
  } catch (error) {
    return { success: false, message: error.message };
  }
}

function SignupForm() {
  const [state, action, isPending] = useActionState(signupAction, {
    success: false,
    message: ''
  });

  return (
    <form action={action}>
      <input
        type="email"
        name="email"
        required
        disabled={isPending}
      />
      <input
        type="password"
        name="password"
        required
        disabled={isPending}
      />

      <button type="submit" disabled={isPending}>
        {isPending ? '처리 중...' : '회원가입'}
      </button>

      {state.message && (
        <div className={state.success ? 'success' : 'error'}>
          {state.message}
        </div>
      )}
    </form>
  );
}
```

#### 점진적 향상(Progressive Enhancement) 예제

```tsx
import { useActionState } from 'react';

async function updateProfileAction(prevState, formData) {
  const name = formData.get('name');
  const bio = formData.get('bio');

  try {
    const updatedProfile = await updateProfile({ name, bio });
    return {
      profile: updatedProfile,
      error: null,
      timestamp: Date.now()
    };
  } catch (error) {
    return {
      profile: prevState.profile,
      error: error.message,
      timestamp: Date.now()
    };
  }
}

function ProfileEditor({ initialProfile }) {
  const [state, action, isPending] = useActionState(
    updateProfileAction,
    { profile: initialProfile, error: null }
  );

  return (
    <form action={action}>
      <label>
        이름:
        <input
          name="name"
          defaultValue={state.profile.name}
          disabled={isPending}
        />
      </label>

      <label>
        자기소개:
        <textarea
          name="bio"
          defaultValue={state.profile.bio}
          disabled={isPending}
        />
      </label>

      <button type="submit" disabled={isPending}>
        {isPending ? '저장 중...' : '프로필 업데이트'}
      </button>

      {state.error && (
        <div className="error">오류: {state.error}</div>
      )}
    </form>
  );
}
```

#### 핵심 특징

- **자동 상태 관리**: 이전 상태를 자동으로 전달받아 처리
- **폼 데이터 접근**: FormData 객체로 폼 입력값 자동 수집
- **대기 상태**: `isPending` 플래그로 제출 중 상태 추적
- **점진적 향상**: JavaScript 비활성화 시에도 작동

---

### 4. useFormStatus() - 폼 제출 상태 확인

`useFormStatus()`는 부모 `<form>`의 제출 상태를 자식 컴포넌트에서 읽을 수 있게 해주는 훅입니다. 디자인 시스템 컴포넌트를 만들 때 특히 유용합니다.

#### 기본 사용법

```tsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '제출 중...' : '제출'}
    </button>
  );
}
```

#### 실전 예제: 재사용 가능한 로딩 버튼

```tsx
import { useFormStatus } from 'react-dom';

// 디자인 시스템 컴포넌트
function LoadingButton({ children, loadingText = '처리 중...' }) {
  const { pending } = useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
      className={pending ? 'loading' : ''}
    >
      {pending ? (
        <>
          <Spinner />
          {loadingText}
        </>
      ) : (
        children
      )}
    </button>
  );
}

// 실제 사용
function CommentForm() {
  async function submitComment(formData) {
    await postComment(formData.get('comment'));
  }

  return (
    <form action={submitComment}>
      <textarea name="comment" placeholder="댓글 입력" />
      <LoadingButton loadingText="댓글 작성 중...">
        댓글 작성
      </LoadingButton>
    </form>
  );
}
```

#### 폼 데이터 접근 예제

```tsx
import { useFormStatus } from 'react-dom';

function FormDebugger() {
  const { pending, data, method } = useFormStatus();

  if (!pending) return null;

  return (
    <div className="debug-info">
      <p>제출 중...</p>
      <p>Method: {method}</p>
      <p>데이터:</p>
      <pre>
        {JSON.stringify(Object.fromEntries(data?.entries() || []), null, 2)}
      </pre>
    </div>
  );
}

function SearchForm() {
  return (
    <form action="/search" method="get">
      <input name="query" placeholder="검색어 입력" />
      <button type="submit">검색</button>
      <FormDebugger />
    </form>
  );
}
```

#### 조건부 필드 비활성화

```tsx
import { useFormStatus } from 'react-dom';

function FormInput({ name, label, ...props }) {
  const { pending } = useFormStatus();

  return (
    <label>
      {label}
      <input
        name={name}
        disabled={pending}
        {...props}
      />
    </label>
  );
}

function ContactForm() {
  async function sendMessage(formData) {
    await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify(Object.fromEntries(formData))
    });
  }

  return (
    <form action={sendMessage}>
      <FormInput name="email" label="이메일" type="email" required />
      <FormInput name="subject" label="제목" required />
      <FormInput name="message" label="메시지" required />
      <LoadingButton>전송</LoadingButton>
    </form>
  );
}
```

#### 주의사항

- **반드시 부모 `<form>` 필요**: `<form>` 내부의 컴포넌트에서만 작동
- **직접 자식만 가능**: `<form>` 컴포넌트 자체 내부에서는 사용 불가
- **react-dom에서 import**: `react`가 아닌 `react-dom`에서 가져오기

```tsx
// ❌ 작동하지 않음
function Form() {
  const { pending } = useFormStatus(); // Form 자체 내부에서는 작동 안함
  return <form>...</form>;
}

// ✅ 올바른 사용
function Form() {
  return (
    <form>
      <SubmitButton /> {/* 자식 컴포넌트에서 사용 */}
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus(); // 여기서는 작동함
  return <button disabled={pending}>제출</button>;
}
```

---

## Actions 패턴

Actions는 React 19의 가장 중요한 새 기능 중 하나입니다. 비동기 함수를 트랜지션에서 사용하여 대기 상태, 에러, 폼 제출, 낙관적 업데이트를 자동으로 처리합니다.

### Actions란 무엇인가?

Actions는 **비동기 작업을 관습적으로(conventionally) 처리하는 패턴**입니다. React는 다음을 자동으로 관리합니다:

- **대기 상태(Pending State)**: 요청 시작 시 자동으로 시작, 완료 시 자동 리셋
- **낙관적 업데이트**: 즉시 UI 업데이트, 실패 시 자동 롤백
- **에러 처리**: Error Boundary와 자동 통합
- **폼 리셋**: 제출 성공 시 자동으로 폼 초기화

### Server Actions와 Client Actions

#### Server Actions

서버에서 실행되는 비동기 함수로, `'use server'` 지시문으로 정의합니다.

```tsx
// actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // 서버에서만 실행되는 코드
  const post = await db.post.create({
    data: { title, content }
  });

  return { success: true, post };
}
```

```tsx
// CreatePostForm.tsx
import { createPost } from './actions';

function CreatePostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="제목" required />
      <textarea name="content" placeholder="내용" required />
      <button type="submit">작성</button>
    </form>
  );
}
```

#### Client Actions

클라이언트에서 실행되는 비동기 함수로, `'use client'` 컴포넌트 내부에서 정의합니다.

```tsx
'use client';
import { useState } from 'react';

function SearchForm() {
  const [results, setResults] = useState([]);

  async function searchAction(formData: FormData) {
    const query = formData.get('query');

    const response = await fetch(`/api/search?q=${query}`);
    const data = await response.json();

    setResults(data.results);
  }

  return (
    <div>
      <form action={searchAction}>
        <input name="query" placeholder="검색어" />
        <button type="submit">검색</button>
      </form>

      <ul>
        {results.map(item => (
          <li key={item.id}>{item.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 폼 처리의 새로운 패러다임

React 19 이전에는 폼 제출을 처리하기 위해 많은 보일러플레이트 코드가 필요했습니다.

#### Before: React 18 방식

```tsx
// React 18 - 복잡한 폼 처리
function ContactForm() {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();

    setIsPending(true);
    setError(null);
    setSuccess(false);

    const formData = new FormData(e.target);

    try {
      await fetch('/api/contact', {
        method: 'POST',
        body: formData
      });

      setSuccess(true);
      e.target.reset(); // 수동으로 폼 리셋
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" disabled={isPending} />
      <textarea name="message" disabled={isPending} />

      <button type="submit" disabled={isPending}>
        {isPending ? '전송 중...' : '전송'}
      </button>

      {error && <div className="error">{error}</div>}
      {success && <div className="success">전송 완료!</div>}
    </form>
  );
}
```

#### After: React 19 Actions

```tsx
// React 19 - 간결한 Actions 패턴
import { useActionState } from 'react';

async function contactAction(prevState, formData) {
  const email = formData.get('email');
  const message = formData.get('message');

  try {
    await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify({ email, message })
    });

    return { success: true, error: null };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

function ContactForm() {
  const [state, action, isPending] = useActionState(contactAction, {
    success: false,
    error: null
  });

  return (
    <form action={action}>
      <input name="email" />
      <textarea name="message" />

      <button type="submit" disabled={isPending}>
        {isPending ? '전송 중...' : '전송'}
      </button>

      {state.error && <div className="error">{state.error}</div>}
      {state.success && <div className="success">전송 완료!</div>}
    </form>
  );
}
```

### 실전 예제: 쇼핑카트 추가

Actions와 `useOptimistic`을 결합한 실무 예제:

```tsx
'use client';
import { useOptimistic, useTransition } from 'react';

function ProductCard({ product, cartItems, addToCart }) {
  const [optimisticCart, addOptimisticItem] = useOptimistic(
    cartItems,
    (state, newItem) => [...state, newItem]
  );

  const [isPending, startTransition] = useTransition();

  async function handleAddToCart() {
    const cartItem = {
      id: Date.now(),
      productId: product.id,
      name: product.name,
      price: product.price,
      pending: true
    };

    startTransition(async () => {
      // 즉시 UI 업데이트
      addOptimisticItem(cartItem);

      // 실제 서버 요청
      await addToCart(product.id);
    });
  }

  const isInCart = optimisticCart.some(
    item => item.productId === product.id
  );

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>{product.price}원</p>

      <button
        onClick={handleAddToCart}
        disabled={isPending || isInCart}
      >
        {isPending ? '추가 중...' : isInCart ? '장바구니에 있음' : '장바구니 추가'}
      </button>
    </div>
  );
}
```

### 에러 핸들링과 로딩 상태 관리

Actions는 Error Boundary와 Suspense와 자연스럽게 통합됩니다.

```tsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

async function deletePostAction(postId) {
  const response = await fetch(`/api/posts/${postId}`, {
    method: 'DELETE'
  });

  if (!response.ok) {
    throw new Error('게시글 삭제 실패');
  }

  return { success: true };
}

function DeleteButton({ postId }) {
  const [isPending, startTransition] = useTransition();

  function handleDelete() {
    if (!confirm('정말 삭제하시겠습니까?')) return;

    startTransition(async () => {
      await deletePostAction(postId);
      // 성공 시 자동으로 리다이렉트 또는 UI 업데이트
    });
  }

  return (
    <button onClick={handleDelete} disabled={isPending}>
      {isPending ? '삭제 중...' : '삭제'}
    </button>
  );
}

function PostPage({ postId }) {
  return (
    <ErrorBoundary
      fallback={<div>오류가 발생했습니다.</div>}
      onReset={() => window.location.reload()}
    >
      <Suspense fallback={<div>로딩 중...</div>}>
        <DeleteButton postId={postId} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## React Server Components 개선사항

React 19는 Server Components의 안정성과 성능을 대폭 개선했습니다.

### Server Components란?

Server Components는 번들링 전 서버 환경에서 미리 렌더링되는 컴포넌트입니다. 클라이언트 번들에 포함되지 않아 초기 로딩 속도가 빠르고, 서버 리소스에 직접 접근할 수 있습니다.

### React 19의 주요 개선사항

#### 1. Async/Await 지원 강화

```tsx
// Server Component - 직접 비동기 처리
async function BlogPost({ id }) {
  // 서버에서 직접 데이터 페칭
  const post = await db.post.findUnique({ where: { id } });
  const comments = await db.comment.findMany({
    where: { postId: id }
  });

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>

      <CommentList comments={comments} />
    </article>
  );
}
```

#### 2. 클라이언트-서버 경계 최적화

Server Component에서 Client Component로 데이터 전달이 더욱 효율적입니다.

```tsx
// Server Component
async function ProductPage({ productId }) {
  const product = await fetchProduct(productId);
  const relatedProducts = await fetchRelatedProducts(productId);

  return (
    <div>
      {/* Server Component - 서버에서 렌더링 */}
      <ProductInfo product={product} />

      {/* Client Component - 클라이언트에서 hydrate */}
      <InteractiveCart product={product} />

      {/* Server Component */}
      <RelatedProducts products={relatedProducts} />
    </div>
  );
}

// Client Component
'use client';
function InteractiveCart({ product }) {
  const [quantity, setQuantity] = useState(1);

  return (
    <div>
      <button onClick={() => setQuantity(q => q - 1)}>-</button>
      <span>{quantity}</span>
      <button onClick={() => setQuantity(q => q + 1)}>+</button>
      <button>장바구니 담기</button>
    </div>
  );
}
```

#### 3. 성능 개선

React 19는 Server Components의 렌더링 성능을 크게 개선했습니다:

- **스트리밍 최적화**: 데이터를 받는 즉시 클라이언트로 전송
- **병렬 데이터 페칭**: 여러 데이터 소스를 동시에 요청
- **자동 코드 스플리팅**: 필요한 컴포넌트만 클라이언트로 전송

```tsx
// 병렬 데이터 페칭
async function Dashboard() {
  // 모든 요청이 병렬로 실행됨
  const [user, stats, notifications] = await Promise.all([
    fetchUser(),
    fetchStats(),
    fetchNotifications()
  ]);

  return (
    <div>
      <UserProfile user={user} />
      <Stats data={stats} />
      <Notifications items={notifications} />
    </div>
  );
}
```

### Server Actions와의 통합

Server Components와 Server Actions를 함께 사용하면 완전한 서버 중심 데이터 흐름을 구현할 수 있습니다.

```tsx
// actions.ts
'use server';

export async function updateUserProfile(userId: string, formData: FormData) {
  const name = formData.get('name') as string;
  const bio = formData.get('bio') as string;

  await db.user.update({
    where: { id: userId },
    data: { name, bio }
  });

  revalidatePath('/profile');
}

// ProfilePage.tsx (Server Component)
async function ProfilePage({ userId }) {
  const user = await db.user.findUnique({ where: { id: userId } });

  return (
    <div>
      <h1>프로필</h1>
      <ProfileForm user={user} userId={userId} />
    </div>
  );
}

// ProfileForm.tsx (Client Component)
'use client';
import { updateUserProfile } from './actions';

function ProfileForm({ user, userId }) {
  const updateWithId = updateUserProfile.bind(null, userId);

  return (
    <form action={updateWithId}>
      <input name="name" defaultValue={user.name} />
      <textarea name="bio" defaultValue={user.bio} />
      <button type="submit">저장</button>
    </form>
  );
}
```

---

## Document Metadata 지원

React 19부터 `<title>`, `<meta>`, `<link>` 태그를 컴포넌트 내부에서 직접 사용할 수 있습니다. React가 자동으로 이들을 document의 `<head>`로 이동시킵니다.

### 기본 사용법

```tsx
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title} - My Blog</title>
      <meta name="description" content={post.excerpt} />
      <meta property="og:title" content={post.title} />
      <meta property="og:image" content={post.coverImage} />

      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### 실전 예제: SEO 최적화

```tsx
function ProductPage({ product }) {
  return (
    <div>
      {/* SEO 메타데이터 */}
      <title>{product.name} - 최저가 {product.price}원</title>
      <meta name="description" content={product.description} />

      {/* Open Graph */}
      <meta property="og:type" content="product" />
      <meta property="og:title" content={product.name} />
      <meta property="og:description" content={product.description} />
      <meta property="og:image" content={product.image} />
      <meta property="og:price:amount" content={product.price} />

      {/* Twitter Card */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={product.name} />
      <meta name="twitter:image" content={product.image} />

      {/* Canonical URL */}
      <link rel="canonical" href={`https://example.com/products/${product.id}`} />

      {/* 실제 컨텐츠 */}
      <h1>{product.name}</h1>
      <img src={product.image} alt={product.name} />
      <p>{product.description}</p>
      <p>{product.price}원</p>
    </div>
  );
}
```

### 외부 스타일시트 및 폰트

```tsx
function App() {
  return (
    <html>
      <head>
        {/* 폰트 프리로드 */}
        <link
          rel="preload"
          href="/fonts/roboto.woff2"
          as="font"
          type="font/woff2"
          crossOrigin="anonymous"
        />

        {/* 외부 스타일시트 */}
        <link
          rel="stylesheet"
          href="/styles/theme.css"
          precedence="default"
        />
      </head>
      <body>
        <Main />
      </body>
    </html>
  );
}
```

### 스타일시트 우선순위 관리

`precedence` prop으로 스타일시트 로딩 순서를 제어할 수 있습니다.

```tsx
function ThemeProvider({ theme, children }) {
  return (
    <>
      {/* 기본 스타일 - 먼저 로드 */}
      <link
        rel="stylesheet"
        href="/styles/reset.css"
        precedence="reset"
      />

      {/* 테마 스타일 - 중간 */}
      <link
        rel="stylesheet"
        href={`/styles/themes/${theme}.css`}
        precedence="theme"
      />

      {/* 컴포넌트 스타일 - 마지막 */}
      <link
        rel="stylesheet"
        href="/styles/components.css"
        precedence="default"
      />

      {children}
    </>
  );
}
```

### Next.js Metadata API와의 관계

Next.js를 사용하는 경우, Next.js의 Metadata API를 사용하는 것이 권장됩니다. Next.js Metadata API는 React 19의 기능을 기반으로 구축되어 더 많은 기능을 제공합니다.

```tsx
// Next.js 방식 (권장)
export const metadata = {
  title: 'Product Name',
  description: 'Product description',
  openGraph: {
    title: 'Product Name',
    description: 'Product description',
    images: ['/product-image.jpg']
  }
};

export default function ProductPage() {
  return <div>Product content</div>;
}
```

---

## ref를 props로 전달

React 19부터는 `forwardRef` 없이 `ref`를 일반 props처럼 전달할 수 있습니다.

### Before: React 18 방식

```tsx
import { forwardRef } from 'react';

// React 18 - forwardRef 필요
const Input = forwardRef(function Input(props, ref) {
  return <input ref={ref} {...props} />;
});

function Form() {
  const inputRef = useRef(null);

  return (
    <form>
      <Input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        포커스
      </button>
    </form>
  );
}
```

### After: React 19 방식

```tsx
// React 19 - forwardRef 불필요!
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

function Form() {
  const inputRef = useRef(null);

  return (
    <form>
      <Input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        포커스
      </button>
    </form>
  );
}
```

### TypeScript 사용 시

```tsx
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  ref?: React.Ref<HTMLInputElement>;
  label?: string;
}

function Input({ ref, label, ...props }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}
```

### 실전 예제: 커스텀 컴포넌트

```tsx
interface ButtonProps {
  ref?: React.Ref<HTMLButtonElement>;
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

function Button({ ref, variant = 'primary', children, ...props }: ButtonProps) {
  return (
    <button
      ref={ref}
      className={`btn btn-${variant}`}
      {...props}
    >
      {children}
    </button>
  );
}

function App() {
  const buttonRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    // 자동 포커스
    buttonRef.current?.focus();
  }, []);

  return (
    <Button ref={buttonRef} variant="primary">
      클릭하세요
    </Button>
  );
}
```

### 주의사항

- **클래스 컴포넌트**: 여전히 `React.createRef()` 또는 콜백 ref 사용
- **기존 라이브러리**: forwardRef를 사용하는 기존 코드는 계속 작동
- **마이그레이션**: 점진적으로 forwardRef 제거 가능

---

## 기타 개선사항

### 1. ref 클린업 함수

ref 콜백에서 클린업 함수를 반환할 수 있습니다.

```tsx
function VideoPlayer({ src }) {
  return (
    <video
      src={src}
      ref={(node) => {
        if (node) {
          // ref가 설정될 때
          node.play();

          // 클린업 함수 반환
          return () => {
            node.pause();
          };
        }
      }}
    />
  );
}
```

### 2. useDeferredValue 초기값 지원

`useDeferredValue`에 초기값을 지정할 수 있습니다.

```tsx
function SearchResults({ query }) {
  // 초기 렌더링 시 빈 문자열 사용
  const deferredQuery = useDeferredValue(query, '');

  const results = useSearchResults(deferredQuery);

  return (
    <div>
      {results.map(result => (
        <ResultItem key={result.id} {...result} />
      ))}
    </div>
  );
}
```

### 3. Context를 Provider 없이 사용

`<Context.Provider>` 대신 `<Context>`를 직접 사용할 수 있습니다.

```tsx
// Before
<ThemeContext.Provider value={theme}>
  <App />
</ThemeContext.Provider>

// After - 더 간결!
<ThemeContext value={theme}>
  <App />
</ThemeContext>
```

### 4. 리소스 프리로딩 API

페이지 로딩 속도를 개선하는 새로운 API들이 추가되었습니다.

```tsx
import { preload, preinit, prefetchDNS, preconnect } from 'react-dom';

function App() {
  useEffect(() => {
    // DNS 프리페치
    prefetchDNS('https://api.example.com');

    // 연결 사전 설정
    preconnect('https://cdn.example.com');

    // 스크립트 프리로드 및 실행
    preinit('/scripts/analytics.js', { as: 'script' });

    // 폰트 프리로드
    preload('/fonts/roboto.woff2', { as: 'font' });
  }, []);

  return <Main />;
}
```

### 5. Suspense 개선

Suspense의 fallback이 더 이상 `display: none`으로 숨겨지지 않고 실제로 마운트됩니다. 이로 인해 더 나은 레이아웃 안정성을 제공합니다.

```tsx
function App() {
  return (
    <Suspense fallback={<Skeleton />}>
      <DataDisplay />
    </Suspense>
  );
}

// fallback이 실제로 렌더링되므로 CSS 애니메이션이 정상 작동
function Skeleton() {
  return (
    <div className="skeleton animate-pulse">
      Loading...
    </div>
  );
}
```

---

## Breaking Changes & 마이그레이션 가이드

React 19는 몇 가지 Breaking Changes를 포함하지만, React 팀은 마이그레이션을 최대한 쉽게 만들기 위해 노력했습니다.

### 마이그레이션 전략

React 팀이 권장하는 단계별 마이그레이션:

1. **React 18.3으로 먼저 업그레이드** (React 18.2와 동일하지만 deprecated 경고 포함)
2. **콘솔 경고 확인 및 수정**
3. **React 19로 업그레이드**
4. **Codemod 실행**

### 주요 Breaking Changes

#### 1. PropTypes 제거

```tsx
// ❌ React 19에서 제거됨
import PropTypes from 'prop-types';

function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

Button.propTypes = {
  onClick: PropTypes.func.isRequired,
  children: PropTypes.node
};

// ✅ TypeScript 또는 다른 타입 체커 사용
interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
}

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

#### 2. defaultProps 제거 (함수 컴포넌트)

```tsx
// ❌ React 19에서 제거됨
function Button({ variant, children }) {
  return <button className={variant}>{children}</button>;
}

Button.defaultProps = {
  variant: 'primary'
};

// ✅ ES6 기본 매개변수 사용
function Button({ variant = 'primary', children }) {
  return <button className={variant}>{children}</button>;
}

// ✅ TypeScript
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

function Button({ variant = 'primary', children }: ButtonProps) {
  return <button className={variant}>{children}</button>;
}
```

#### 3. Legacy Context 제거

```tsx
// ❌ getChildContext 제거됨
class LegacyProvider extends React.Component {
  getChildContext() {
    return { theme: 'dark' };
  }
}

// ✅ 새로운 Context API 사용
const ThemeContext = React.createContext('light');

function ModernProvider({ children }) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );
}
```

#### 4. String Refs 제거

```tsx
// ❌ String refs 제거됨
class OldComponent extends React.Component {
  componentDidMount() {
    this.refs.input.focus();
  }

  render() {
    return <input ref="input" />;
  }
}

// ✅ useRef 또는 createRef 사용
function NewComponent() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

#### 5. ReactDOM.render 제거

```tsx
// ❌ ReactDOM.render 제거됨
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));

// ✅ createRoot 사용
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

#### 6. react-test-utils 제거

```tsx
// ❌ react-dom/test-utils 제거됨
import { act } from 'react-dom/test-utils';

// ✅ @testing-library/react 사용 (권장)
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('button click', async () => {
  const user = userEvent.setup();
  render(<Button />);

  await user.click(screen.getByRole('button'));

  await waitFor(() => {
    expect(screen.getByText('Clicked')).toBeInTheDocument();
  });
});
```

### TypeScript 타입 변경사항

#### ref 콜백 반환 타입

```tsx
// React 18: void
// React 19: void | (() => void)

// ❌ React 19에서 타입 오류
<div ref={(node) => (node?.focus())} />

// ✅ 명시적으로 undefined 반환
<div ref={(node) => { node?.focus(); }} />

// ✅ 또는 클린업 함수 반환
<div ref={(node) => {
  if (node) {
    node.focus();
    return () => console.log('cleanup');
  }
}} />
```

### Codemod 도구 활용

React 팀은 자동 마이그레이션을 위한 codemod를 제공합니다.

#### 전체 마이그레이션 레시피 실행

```bash
npx codemod react/19/migration-recipe
```

이 명령은 다음 작업을 자동으로 수행합니다:

- `ReactDOM.render` → `createRoot` 변환
- PropTypes 제거 경고 추가
- defaultProps를 기본 매개변수로 변환
- Legacy Context를 새 Context API로 변환

#### 개별 codemod 실행

```bash
# React DOM 업그레이드
npx codemod react/19/replace-reactdom-render

# PropTypes 제거
npx codemod react/19/replace-prop-types-with-typescript

# defaultProps 변환
npx codemod react/19/replace-default-props

# string refs 변환
npx codemod react/19/replace-string-ref
```

#### TypeScript 타입 codemod

```bash
npx types-react-codemod@latest preset-19 ./src
```

### 주의사항

#### 1. 써드파티 라이브러리 호환성 확인

React 19로 업그레이드하기 전에 사용 중인 라이브러리들이 React 19와 호환되는지 확인하세요.

```bash
# package.json에서 peer dependency 확인
npm ls react
```

주요 라이브러리 호환성:
- **React Router**: v6.4+ 호환
- **Redux**: React-Redux v8+ 호환
- **Material-UI**: v5.14+ 호환
- **Ant Design**: v5.11+ 호환

#### 2. Hydration 오류 처리 변경

React 19는 hydration 오류를 더 엄격하게 처리합니다. 서버와 클라이언트의 HTML이 일치하지 않으면 에러가 발생합니다.

```tsx
// ❌ Hydration 불일치
function Clock() {
  return <div>{new Date().toTimeString()}</div>;
}

// ✅ useEffect로 클라이언트 전용 렌더링
function Clock() {
  const [time, setTime] = useState('');

  useEffect(() => {
    setTime(new Date().toTimeString());
  }, []);

  return <div>{time || 'Loading...'}</div>;
}
```

#### 3. useFormState → useActionState 변경

```tsx
// ❌ useFormState (deprecated)
import { useFormState } from 'react-dom';

// ✅ useActionState
import { useActionState } from 'react';
```

---

## 실전 마이그레이션 예제

실제 프로젝트를 React 18에서 React 19로 마이그레이션하는 전체 과정을 살펴보겠습니다.

### 예제: 사용자 프로필 편집 폼

#### Before: React 18 버전

```tsx
// UserProfileForm.tsx (React 18)
import { useState, useRef, forwardRef } from 'react';
import PropTypes from 'prop-types';

// forwardRef 필요
const Input = forwardRef(function Input({ label, ...props }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
});

Input.propTypes = {
  label: PropTypes.string.isRequired,
  type: PropTypes.string
};

Input.defaultProps = {
  type: 'text'
};

function UserProfileForm({ user, onSave }) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  const nameInputRef = useRef(null);

  async function handleSubmit(e) {
    e.preventDefault();

    setLoading(true);
    setError(null);
    setSuccess(false);

    const formData = new FormData(e.target);
    const data = {
      name: formData.get('name'),
      email: formData.get('email'),
      bio: formData.get('bio')
    };

    try {
      await fetch(`/api/users/${user.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });

      setSuccess(true);
      if (onSave) onSave(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <Input
        ref={nameInputRef}
        label="이름"
        name="name"
        defaultValue={user.name}
        disabled={loading}
        required
      />

      <Input
        label="이메일"
        name="email"
        type="email"
        defaultValue={user.email}
        disabled={loading}
        required
      />

      <label>
        자기소개
        <textarea
          name="bio"
          defaultValue={user.bio}
          disabled={loading}
        />
      </label>

      <button type="submit" disabled={loading}>
        {loading ? '저장 중...' : '저장'}
      </button>

      {error && <div className="error">{error}</div>}
      {success && <div className="success">저장 완료!</div>}
    </form>
  );
}

UserProfileForm.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.string.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired,
    bio: PropTypes.string
  }).isRequired,
  onSave: PropTypes.func
};

export default UserProfileForm;
```

#### After: React 19 버전

```tsx
// UserProfileForm.tsx (React 19)
import { useActionState, useOptimistic } from 'react';

// forwardRef 불필요 - ref를 직접 props로 받음
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  ref?: React.Ref<HTMLInputElement>;
  label: string;
}

function Input({ ref, label, ...props }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}

// PropTypes 대신 TypeScript
interface User {
  id: string;
  name: string;
  email: string;
  bio: string;
}

interface UserProfileFormProps {
  user: User;
  onSave?: (data: Partial<User>) => void;
}

// Action 함수 분리
async function updateProfileAction(prevState: any, formData: FormData) {
  const data = {
    name: formData.get('name') as string,
    email: formData.get('email') as string,
    bio: formData.get('bio') as string
  };

  try {
    const response = await fetch(`/api/users/${prevState.userId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });

    if (!response.ok) throw new Error('저장 실패');

    return {
      success: true,
      error: null,
      userId: prevState.userId,
      data
    };
  } catch (error) {
    return {
      success: false,
      error: error.message,
      userId: prevState.userId
    };
  }
}

function UserProfileForm({ user, onSave }: UserProfileFormProps) {
  // useActionState로 상태 관리 단순화
  const [state, action, isPending] = useActionState(
    updateProfileAction,
    {
      success: false,
      error: null,
      userId: user.id
    }
  );

  // 낙관적 업데이트
  const [optimisticUser, setOptimisticUser] = useOptimistic(
    user,
    (state, newData: Partial<User>) => ({ ...state, ...newData })
  );

  // 저장 성공 시 콜백
  if (state.success && state.data && onSave) {
    onSave(state.data);
  }

  return (
    <form action={action}>
      <Input
        label="이름"
        name="name"
        defaultValue={optimisticUser.name}
        disabled={isPending}
        required
      />

      <Input
        label="이메일"
        name="email"
        type="email"
        defaultValue={optimisticUser.email}
        disabled={isPending}
        required
      />

      <label>
        자기소개
        <textarea
          name="bio"
          defaultValue={optimisticUser.bio}
          disabled={isPending}
        />
      </label>

      <SubmitButton />

      {state.error && <div className="error">{state.error}</div>}
      {state.success && <div className="success">저장 완료!</div>}
    </form>
  );
}

// useFormStatus를 사용하는 재사용 가능한 버튼
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '저장 중...' : '저장'}
    </button>
  );
}

export default UserProfileForm;
```

### 주요 변경사항 요약

| 항목 | React 18 | React 19 |
|------|----------|----------|
| PropTypes | `PropTypes.shape()` | TypeScript 인터페이스 |
| defaultProps | `Component.defaultProps` | ES6 기본 매개변수 |
| ref 전달 | `forwardRef` 필요 | `ref` props 직접 사용 |
| 폼 상태 | `useState` × 3개 | `useActionState` 1개 |
| 로딩 상태 | 수동 관리 | 자동 관리 |
| 에러 처리 | try-catch + state | Action 자동 처리 |
| 낙관적 업데이트 | 수동 구현 | `useOptimistic` |
| 코드 라인 수 | ~120줄 | ~90줄 |

---

## FAQ

### Q1: React 19는 언제 프로덕션에 사용할 수 있나요?

**A:** React 19는 2024년 12월 5일 정식 릴리스되었으며, 프로덕션에서 안전하게 사용할 수 있습니다. 다만 다음 사항을 확인하세요:

- 사용 중인 써드파티 라이브러리가 React 19와 호환되는지 확인
- React 18.3으로 먼저 업그레이드하여 deprecation 경고 확인
- 테스트 커버리지가 충분한지 확인 후 단계적으로 마이그레이션

### Q2: use() 훅과 useEffect의 차이점은 무엇인가요?

**A:** `use()`는 Promise나 Context를 동기적으로 읽는 API이고, `useEffect`는 부수 효과를 처리하는 훅입니다.

```tsx
// use() - Promise를 동기적으로 읽음 (Suspense와 통합)
function Component({ dataPromise }) {
  const data = use(dataPromise); // Suspend됨
  return <div>{data}</div>;
}

// useEffect - 비동기 부수 효과 처리
function Component() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;
  return <div>{data}</div>;
}
```

**주요 차이:**
- `use()`는 조건문에서 사용 가능, `useEffect`는 불가능
- `use()`는 Suspense와 자동 통합, `useEffect`는 수동 로딩 상태 관리
- `use()`는 서버에서 전달받은 Promise에 최적화됨

### Q3: Server Components와 Server Actions의 차이는?

**A:**

- **Server Components**: 서버에서 렌더링되는 컴포넌트 (데이터 표시)
- **Server Actions**: 서버에서 실행되는 함수 (데이터 변경)

```tsx
// Server Component - 데이터 읽기
async function BlogPost({ id }) {
  const post = await db.post.findUnique({ where: { id } });
  return <article>{post.content}</article>;
}

// Server Action - 데이터 쓰기
'use server';
async function createPost(formData) {
  const post = await db.post.create({
    data: { title: formData.get('title') }
  });
  return post;
}
```

**사용 시나리오:**
- Server Components: 블로그 포스트, 상품 목록 등 정적 데이터 표시
- Server Actions: 폼 제출, 데이터 수정, 삭제 등 변경 작업

### Q4: useOptimistic을 언제 사용해야 하나요?

**A:** 사용자 피드백이 즉시 필요한 작업에 사용하세요:

**적합한 경우:**
- 좋아요/팔로우 버튼
- 댓글/리뷰 추가
- 장바구니 아이템 추가/삭제
- 간단한 폼 제출

**부적합한 경우:**
- 결제 처리 (실패 시 심각한 문제)
- 데이터 삭제 (복구 불가능)
- 복잡한 비즈니스 로직 (실패 가능성 높음)

```tsx
// ✅ 좋은 예: 좋아요 버튼
function LikeButton() {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(likes);
  // 즉시 UI 업데이트, 실패 시 자동 롤백
}

// ❌ 나쁜 예: 결제 처리
function PaymentButton() {
  const [optimisticPayment, setOptimisticPayment] = useOptimistic(payment);
  // 결제는 낙관적 업데이트 하면 안됨!
}
```

### Q5: forwardRef를 계속 사용해도 되나요?

**A:** 네, forwardRef는 계속 작동하며 당장 제거할 필요는 없습니다. 하지만 새로운 컴포넌트는 ref를 props로 직접 받는 방식을 권장합니다.

```tsx
// 기존 코드 - 계속 작동함
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// 새로운 방식 - 더 간단
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

**마이그레이션 전략:**
1. 기존 코드는 그대로 유지 (작동함)
2. 새로운 컴포넌트는 새 방식 사용
3. 시간이 날 때 점진적으로 마이그레이션

### Q6: React 19로 업그레이드하면 성능이 개선되나요?

**A:** 네, 여러 영역에서 성능이 개선되었습니다:

**자동 성능 개선:**
- Server Components 렌더링 최적화
- 번들 크기 감소 (PropTypes, forwardRef 제거)
- Hydration 성능 개선

**새로운 최적화 기회:**
- 리소스 프리로딩 API로 초기 로딩 속도 개선
- Actions로 불필요한 상태 관리 제거
- 스타일시트 우선순위 관리로 렌더 블로킹 감소

**실제 벤치마크 (Meta 사례):**
- 초기 로딩 시간: 15% 개선
- Time to Interactive: 20% 개선
- 번들 크기: 8-12% 감소

### Q7: TypeScript 프로젝트는 추가 작업이 필요한가요?

**A:** 네, 타입 정의 업데이트가 필요합니다.

```bash
# React 19 타입 설치
npm install --save-dev @types/react@19 @types/react-dom@19

# 타입 codemod 실행
npx types-react-codemod@latest preset-19 ./src
```

**주요 타입 변경:**
- ref 콜백 반환 타입: `void | (() => void)`
- `React.FC`의 암묵적 children 제거
- `JSX.Element` vs `React.ReactNode` 구분 명확화

```tsx
// React 18 타입
interface Props {
  children?: React.ReactNode; // 명시적으로 선언 필요
}

// React 19 타입 (동일)
interface Props {
  children?: React.ReactNode; // 여전히 명시적 선언 필요
}

// ref 타입
interface InputProps {
  ref?: React.Ref<HTMLInputElement>; // 추가됨
}
```

---

## 마치며

React 19는 비동기 작업 처리를 혁신적으로 개선하고, 개발자 경험을 향상시키는 중요한 업데이트입니다. Actions 패턴, 새로운 Hooks, Server Components 개선사항은 더 간결하고 유지보수하기 쉬운 코드를 작성할 수 있게 해줍니다.

### 다음 단계

1. **React 18.3으로 업그레이드**: deprecated 경고 확인
2. **Codemod 실행**: 자동 마이그레이션 적용
3. **새로운 기능 학습**: Actions, use() 등 실험
4. **점진적 적용**: 새 기능을 한 번에 하나씩 도입
5. **커뮤니티 참여**: 피드백 공유 및 베스트 프랙티스 학습

React 19로의 마이그레이션이 순조롭게 진행되길 바랍니다!

---

## 참고 자료

### 공식 문서
- [React 19 공식 릴리스 노트](https://react.dev/blog/2024/12/05/react-19)
- [React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)
- [use() API 문서](https://react.dev/reference/react/use)
- [useOptimistic 문서](https://react.dev/reference/react/useOptimistic)
- [React Server Components 문서](https://react.dev/reference/rsc/server-components)

### 마이그레이션 도구
- [React 19 Codemod](https://codemod.com/registry/react-19-migration-recipe)
- [TypeScript Types Codemod](https://github.com/eps1lon/types-react-codemod)
