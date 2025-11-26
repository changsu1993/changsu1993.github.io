---
title: "React Actions 패턴 상세 가이드 - useActionState, useFormStatus, useOptimistic"
description: "React 19의 Actions 패턴을 완벽히 이해합니다. useActionState, useFormStatus, useOptimistic 훅의 사용법과 Server Actions, 낙관적 업데이트, 폼 처리 패턴을 실전 예제와 함께 다룹니다."
date: 2025-11-26 10:00:00 +0900
categories: [Frontend, React]
tags: [react, react-19, actions, useActionState, useFormStatus, useOptimistic, server-actions, forms]
---

React 19는 폼과 데이터 변이를 처리하는 새로운 패턴인 **Actions**를 도입했습니다. 이 가이드에서는 Actions의 핵심 개념부터 실전 활용 패턴까지 심층적으로 다룹니다.

## Actions란 무엇인가?

Actions는 React 19에서 도입된 비동기 데이터 변이를 처리하기 위한 새로운 패턴입니다. 전통적인 이벤트 핸들러와 달리, Actions는 다음과 같은 특징을 가집니다:

- **비동기 트랜지션**: `useTransition`과 통합되어 pending 상태를 자동으로 관리
- **폼과의 네이티브 통합**: `<form action={...}>`을 통한 선언적 데이터 제출
- **낙관적 업데이트**: 서버 응답 전에 UI를 즉시 업데이트
- **자동 에러 처리**: Error Boundary와 통합된 에러 관리

### Before vs After 비교

**React 18 이전 방식:**

```tsx
function UpdateNameForm() {
  const [name, setName] = useState('');
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setIsPending(true);
    setError(null);

    try {
      await updateName(name);
    } catch (err) {
      setError(err);
    } finally {
      setIsPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button disabled={isPending}>
        {isPending ? '업데이트 중...' : '저장'}
      </button>
      {error && <p>{error.message}</p>}
    </form>
  );
}
```

**React 19 Actions 방식:**

```tsx
function UpdateNameForm() {
  const [state, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const name = formData.get('name');
      try {
        await updateName(name);
        return { success: true };
      } catch (error) {
        return { error: error.message };
      }
    },
    { success: false }
  );

  return (
    <form action={submitAction}>
      <input name="name" />
      <SubmitButton />
      {state.error && <p>{state.error}</p>}
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? '업데이트 중...' : '저장'}</button>;
}
```

코드가 더 간결해지고, pending 상태와 에러 처리가 자동화됩니다.

## Actions의 핵심 개념

### 1. 폼 제출과 데이터 변이

Actions는 `<form>` 요소의 `action` 속성에 직접 전달할 수 있습니다:

```tsx
function CommentForm({ postId }: { postId: string }) {
  async function createComment(formData: FormData) {
    const content = formData.get('content') as string;
    await api.createComment(postId, content);
  }

  return (
    <form action={createComment}>
      <textarea name="content" required />
      <button type="submit">댓글 작성</button>
    </form>
  );
}
```

폼이 제출되면 React는 자동으로:
- `FormData` 객체를 생성하여 action에 전달
- 비동기 작업이 완료될 때까지 pending 상태 관리
- 에러가 발생하면 가장 가까운 Error Boundary로 전달

### 2. 비동기 트랜지션

Actions는 내부적으로 `useTransition`을 사용하여 비동기 상태를 관리합니다:

```tsx
function LikeButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();

  function handleLike() {
    startTransition(async () => {
      await api.likePost(postId);
    });
  }

  return (
    <button onClick={handleLike} disabled={isPending}>
      {isPending ? '처리 중...' : '좋아요'}
    </button>
  );
}
```

`startTransition`으로 감싸진 업데이트는:
- 중단 가능(interruptible)하여 사용자 입력에 반응
- 다른 긴급한 업데이트가 있으면 중단될 수 있음
- pending 상태를 추적하여 로딩 UI 표시 가능

### 3. 낙관적 업데이트

사용자 경험을 개선하기 위해 서버 응답 전에 UI를 즉시 업데이트할 수 있습니다:

```tsx
function TodoItem({ todo }: { todo: Todo }) {
  const [optimisticTodo, updateOptimisticTodo] = useOptimistic(
    todo,
    (state, newCompleted: boolean) => ({
      ...state,
      completed: newCompleted
    })
  );

  async function toggleTodo(formData: FormData) {
    updateOptimisticTodo(!optimisticTodo.completed);
    await api.updateTodo(todo.id, { completed: !todo.completed });
  }

  return (
    <form action={toggleTodo}>
      <button type="submit">
        {optimisticTodo.completed ? '✓' : '○'} {optimisticTodo.title}
      </button>
    </form>
  );
}
```

## useActionState 훅

`useActionState`는 폼 상태를 관리하고 Actions를 처리하는 핵심 훅입니다.

### 기본 사용법

```tsx
const [state, formAction, isPending] = useActionState(
  action,
  initialState,
  permalink?
);
```

**파라미터:**
- `action`: 폼이 제출될 때 호출될 함수
- `initialState`: 초기 상태 값
- `permalink` (선택): 서버에서 렌더링될 때 사용할 URL

**반환값:**
- `state`: 현재 상태
- `formAction`: 폼의 action 속성에 전달할 함수
- `isPending`: pending 상태 (boolean)

### 폼 상태 관리

실제 회원가입 폼 예제:

```tsx
interface SignupState {
  errors?: {
    email?: string[];
    password?: string[];
  };
  message?: string;
}

function SignupForm() {
  const [state, formAction, isPending] = useActionState<SignupState>(
    async (previousState, formData) => {
      const email = formData.get('email') as string;
      const password = formData.get('password') as string;

      // 유효성 검사
      const errors: SignupState['errors'] = {};

      if (!email.includes('@')) {
        errors.email = ['유효한 이메일 주소를 입력하세요.'];
      }

      if (password.length < 8) {
        errors.password = ['비밀번호는 최소 8자 이상이어야 합니다.'];
      }

      if (Object.keys(errors).length > 0) {
        return { errors };
      }

      // API 호출
      try {
        await api.signup({ email, password });
        return { message: '회원가입이 완료되었습니다!' };
      } catch (error) {
        return {
          errors: { email: ['이미 사용 중인 이메일입니다.'] }
        };
      }
    },
    { message: '' }
  );

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          name="email"
          type="email"
          required
          aria-describedby={state.errors?.email ? 'email-error' : undefined}
        />
        {state.errors?.email && (
          <p id="email-error" className="error">
            {state.errors.email.join(', ')}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          name="password"
          type="password"
          required
          aria-describedby={state.errors?.password ? 'password-error' : undefined}
        />
        {state.errors?.password && (
          <p id="password-error" className="error">
            {state.errors.password.join(', ')}
          </p>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? '처리 중...' : '회원가입'}
      </button>

      {state.message && <p className="success">{state.message}</p>}
    </form>
  );
}
```

### 에러 처리

Actions에서 발생한 에러는 두 가지 방식으로 처리할 수 있습니다:

**1. 반환값으로 처리 (권장):**

```tsx
const [state, formAction] = useActionState(
  async (previousState, formData) => {
    try {
      await someAsyncOperation(formData);
      return { success: true };
    } catch (error) {
      return {
        success: false,
        error: error instanceof Error ? error.message : '오류가 발생했습니다.'
      };
    }
  },
  { success: false }
);
```

**2. Error Boundary로 전파:**

```tsx
function FormWithErrorBoundary() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <FormComponent />
    </ErrorBoundary>
  );
}

function FormComponent() {
  const [state, formAction] = useActionState(
    async (previousState, formData) => {
      // 에러를 throw하면 Error Boundary가 처리
      const result = await riskyOperation(formData);
      if (!result.ok) {
        throw new Error(result.error);
      }
      return result.data;
    },
    null
  );

  return <form action={formAction}>{/* ... */}</form>;
}
```

### pending 상태 활용

`isPending`을 사용하여 다양한 로딩 UI를 구현할 수 있습니다:

```tsx
function SearchForm() {
  const [results, searchAction, isPending] = useActionState(
    async (previousState, formData) => {
      const query = formData.get('query') as string;
      const results = await api.search(query);
      return results;
    },
    []
  );

  return (
    <>
      <form action={searchAction}>
        <input
          name="query"
          placeholder="검색어를 입력하세요"
          disabled={isPending}
        />
        <button type="submit" disabled={isPending}>
          검색
        </button>
      </form>

      {isPending ? (
        <div className="loading">
          <Spinner />
          <p>검색 중...</p>
        </div>
      ) : (
        <SearchResults results={results} />
      )}
    </>
  );
}
```

## useFormStatus 훅

`useFormStatus`는 **폼 내부의 컴포넌트**에서 부모 폼의 상태를 읽을 수 있게 해주는 훅입니다.

### 기본 사용법

```tsx
const { pending, data, method, action } = useFormStatus();
```

**반환값:**
- `pending`: 폼이 제출 중인지 여부 (boolean)
- `data`: 제출 중인 `FormData` 객체
- `method`: HTTP 메서드 ('get' 또는 'post')
- `action`: action 함수에 대한 참조

**중요한 제약사항:**
- `useFormStatus`는 **반드시** `<form>` 태그 내부의 컴포넌트에서만 호출해야 합니다.
- 폼 컴포넌트 자체에서는 호출할 수 없으며, 자식 컴포넌트에서만 사용 가능합니다.

### 버튼 비활성화 패턴

```tsx
function SubmitButton({ children }: { children: React.ReactNode }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {children}
    </button>
  );
}

function CreatePostForm() {
  async function createPost(formData: FormData) {
    await api.createPost({
      title: formData.get('title'),
      content: formData.get('content')
    });
  }

  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <SubmitButton>게시글 작성</SubmitButton>
    </form>
  );
}
```

### 로딩 인디케이터

여러 버튼이 있는 폼에서 어떤 버튼이 눌렸는지 확인:

```tsx
function FormButtons() {
  const { pending, data } = useFormStatus();
  const action = data?.get('action');

  return (
    <>
      <button
        type="submit"
        name="action"
        value="save"
        disabled={pending}
      >
        {pending && action === 'save' ? (
          <>
            <Spinner /> 저장 중...
          </>
        ) : (
          '저장'
        )}
      </button>

      <button
        type="submit"
        name="action"
        value="publish"
        disabled={pending}
      >
        {pending && action === 'publish' ? (
          <>
            <Spinner /> 발행 중...
          </>
        ) : (
          '발행'
        )}
      </button>
    </>
  );
}

function PostForm() {
  async function handleSubmit(formData: FormData) {
    const action = formData.get('action');
    const post = {
      title: formData.get('title'),
      content: formData.get('content')
    };

    if (action === 'save') {
      await api.saveDraft(post);
    } else if (action === 'publish') {
      await api.publishPost(post);
    }
  }

  return (
    <form action={handleSubmit}>
      <input name="title" />
      <textarea name="content" />
      <FormButtons />
    </form>
  );
}
```

### 전체 폼 비활성화

pending 상태일 때 모든 입력 필드를 비활성화:

```tsx
function FormFields() {
  const { pending } = useFormStatus();

  return (
    <fieldset disabled={pending}>
      <legend>사용자 정보</legend>
      <input name="name" placeholder="이름" />
      <input name="email" placeholder="이메일" type="email" />
      <textarea name="bio" placeholder="자기소개" />
    </fieldset>
  );
}

function UserProfileForm() {
  async function updateProfile(formData: FormData) {
    await api.updateProfile({
      name: formData.get('name'),
      email: formData.get('email'),
      bio: formData.get('bio')
    });
  }

  return (
    <form action={updateProfile}>
      <FormFields />
      <SubmitButton>프로필 업데이트</SubmitButton>
    </form>
  );
}
```

`<fieldset disabled={pending}>`을 사용하면 내부의 모든 입력 요소가 자동으로 비활성화됩니다.

## useOptimistic 훅

`useOptimistic`은 비동기 작업이 진행되는 동안 UI를 낙관적으로 업데이트하는 훅입니다.

### 기본 사용법

```tsx
const [optimisticState, addOptimistic] = useOptimistic(
  state,
  updateFn
);
```

**파라미터:**
- `state`: 초기 값 및 작업이 완료되었을 때 반환될 값
- `updateFn`: 낙관적 업데이트를 수행하는 함수 `(currentState, optimisticValue) => newState`

**반환값:**
- `optimisticState`: 낙관적으로 업데이트된 상태
- `addOptimistic`: 낙관적 업데이트를 트리거하는 함수

### 낙관적 UI 업데이트

좋아요 버튼 예제:

```tsx
interface Post {
  id: string;
  likes: number;
  isLiked: boolean;
}

function LikeButton({ post }: { post: Post }) {
  const [optimisticPost, updateOptimisticPost] = useOptimistic(
    post,
    (currentPost, newIsLiked: boolean) => ({
      ...currentPost,
      isLiked: newIsLiked,
      likes: currentPost.likes + (newIsLiked ? 1 : -1)
    })
  );

  async function toggleLike() {
    const newIsLiked = !optimisticPost.isLiked;

    // 즉시 UI 업데이트
    updateOptimisticPost(newIsLiked);

    // 서버에 요청
    try {
      await api.toggleLike(post.id, newIsLiked);
    } catch (error) {
      // 에러 발생 시 원래 상태로 자동 롤백됨
      console.error('좋아요 처리 실패:', error);
    }
  }

  return (
    <button onClick={toggleLike} className={optimisticPost.isLiked ? 'liked' : ''}>
      {optimisticPost.isLiked ? '❤️' : '🤍'} {optimisticPost.likes}
    </button>
  );
}
```

### 롤백 처리

`useOptimistic`은 비동기 작업이 완료되면 자동으로 원래 상태로 돌아갑니다:

```tsx
function MessagesThread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages, newMessage: Message) => [
      ...currentMessages,
      { ...newMessage, sending: true }
    ]
  );

  async function sendMessage(formData: FormData) {
    const content = formData.get('message') as string;
    const tempMessage: Message = {
      id: crypto.randomUUID(),
      content,
      timestamp: new Date(),
      sending: true
    };

    // 낙관적 업데이트
    addOptimisticMessage(tempMessage);

    try {
      await api.sendMessage(content);
      // 성공하면 부모 컴포넌트가 새 메시지 목록으로 리렌더링
    } catch (error) {
      // 실패하면 optimisticMessages가 원래 messages로 자동 롤백
      alert('메시지 전송 실패');
    }
  }

  return (
    <>
      <ul>
        {optimisticMessages.map((message) => (
          <li
            key={message.id}
            className={message.sending ? 'sending' : ''}
          >
            {message.content}
            {message.sending && <Spinner />}
          </li>
        ))}
      </ul>
      <form action={sendMessage}>
        <input name="message" />
        <button type="submit">전송</button>
      </form>
    </>
  );
}
```

### 실전 예제: 장바구니

복잡한 낙관적 업데이트 예제:

```tsx
interface CartItem {
  id: string;
  name: string;
  quantity: number;
  price: number;
}

type CartAction =
  | { type: 'add'; item: CartItem }
  | { type: 'remove'; itemId: string }
  | { type: 'update'; itemId: string; quantity: number };

function ShoppingCart({ items }: { items: CartItem[] }) {
  const [optimisticItems, updateOptimisticItems] = useOptimistic(
    items,
    (currentItems, action: CartAction) => {
      switch (action.type) {
        case 'add':
          return [...currentItems, action.item];

        case 'remove':
          return currentItems.filter(item => item.id !== action.itemId);

        case 'update':
          return currentItems.map(item =>
            item.id === action.itemId
              ? { ...item, quantity: action.quantity }
              : item
          );

        default:
          return currentItems;
      }
    }
  );

  async function addToCart(product: CartItem) {
    updateOptimisticItems({ type: 'add', item: product });
    await api.addToCart(product);
  }

  async function removeFromCart(itemId: string) {
    updateOptimisticItems({ type: 'remove', itemId });
    await api.removeFromCart(itemId);
  }

  async function updateQuantity(itemId: string, quantity: number) {
    updateOptimisticItems({ type: 'update', itemId, quantity });
    await api.updateCartItem(itemId, quantity);
  }

  const total = optimisticItems.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  return (
    <div className="cart">
      <h2>장바구니</h2>
      {optimisticItems.length === 0 ? (
        <p>장바구니가 비어있습니다.</p>
      ) : (
        <>
          <ul>
            {optimisticItems.map((item) => (
              <li key={item.id}>
                <span>{item.name}</span>
                <input
                  type="number"
                  value={item.quantity}
                  min="1"
                  onChange={(e) => {
                    const newQuantity = parseInt(e.target.value);
                    if (newQuantity > 0) {
                      updateQuantity(item.id, newQuantity);
                    }
                  }}
                />
                <span>{(item.price * item.quantity).toLocaleString()}원</span>
                <button onClick={() => removeFromCart(item.id)}>
                  삭제
                </button>
              </li>
            ))}
          </ul>
          <div className="total">
            <strong>총액: {total.toLocaleString()}원</strong>
          </div>
        </>
      )}
    </div>
  );
}
```

## Server Actions (Next.js)

Next.js에서 Server Actions를 사용하면 클라이언트에서 서버 함수를 직접 호출할 수 있습니다.

### 'use server' 지시어

Server Actions는 파일 또는 함수 레벨에서 선언할 수 있습니다:

**파일 레벨:**

```tsx
// app/actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await db.post.create({
    data: { title, content }
  });

  revalidatePath('/posts');
}

export async function deletePost(postId: string) {
  await db.post.delete({
    where: { id: postId }
  });

  revalidatePath('/posts');
}
```

**함수 레벨:**

```tsx
// app/components/CommentForm.tsx
export function CommentForm({ postId }: { postId: string }) {
  async function createComment(formData: FormData) {
    'use server';

    const content = formData.get('content') as string;

    await db.comment.create({
      data: {
        postId,
        content
      }
    });

    revalidatePath(`/posts/${postId}`);
  }

  return (
    <form action={createComment}>
      <textarea name="content" required />
      <button type="submit">댓글 작성</button>
    </form>
  );
}
```

### 폼과 Server Actions 연동

Server Actions를 사용한 블로그 게시글 생성:

```tsx
// app/posts/new/page.tsx
import { redirect } from 'next/navigation';
import { revalidatePath } from 'next/cache';

async function createPost(formData: FormData) {
  'use server';

  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  const tags = (formData.get('tags') as string).split(',').map(t => t.trim());

  const post = await db.post.create({
    data: {
      title,
      content,
      tags: {
        connectOrCreate: tags.map(tag => ({
          where: { name: tag },
          create: { name: tag }
        }))
      }
    }
  });

  revalidatePath('/posts');
  redirect(`/posts/${post.id}`);
}

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <div>
        <label htmlFor="title">제목</label>
        <input id="title" name="title" required />
      </div>

      <div>
        <label htmlFor="content">내용</label>
        <textarea id="content" name="content" required />
      </div>

      <div>
        <label htmlFor="tags">태그 (쉼표로 구분)</label>
        <input id="tags" name="tags" placeholder="react, typescript" />
      </div>

      <button type="submit">게시글 작성</button>
    </form>
  );
}
```

### 재검증(Revalidation) 패턴

데이터가 변경된 후 캐시를 무효화하는 패턴:

**1. revalidatePath - 경로별 재검증:**

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function updateProfile(userId: string, formData: FormData) {
  const name = formData.get('name') as string;
  const bio = formData.get('bio') as string;

  await db.user.update({
    where: { id: userId },
    data: { name, bio }
  });

  // 특정 경로의 캐시 재검증
  revalidatePath(`/users/${userId}`);

  // 동적 세그먼트가 있는 모든 경로 재검증
  revalidatePath('/users/[id]', 'page');

  // 레이아웃 포함 재검증
  revalidatePath('/users', 'layout');
}
```

**2. revalidateTag - 태그별 재검증:**

```tsx
// app/actions.ts
'use server';

import { revalidateTag } from 'next/cache';

export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title') as string,
      content: formData.get('content') as string
    }
  });

  // 'posts' 태그가 있는 모든 fetch 요청 재검증
  revalidateTag('posts');
}

// 데이터 페칭 시 태그 지정
export async function getPosts() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  });
  return posts.json();
}
```

**3. redirect - 작업 후 리다이렉트:**

```tsx
'use server';

import { redirect } from 'next/navigation';

export async function deletePost(postId: string) {
  await db.post.delete({
    where: { id: postId }
  });

  revalidatePath('/posts');
  redirect('/posts'); // 목록 페이지로 리다이렉트
}
```

### Server Actions와 useActionState 통합

```tsx
// app/actions.ts
'use server';

interface LoginState {
  error?: string;
  success?: boolean;
}

export async function login(
  previousState: LoginState,
  formData: FormData
): Promise<LoginState> {
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;

  try {
    const user = await authenticate(email, password);

    if (!user) {
      return { error: '이메일 또는 비밀번호가 올바르지 않습니다.' };
    }

    await createSession(user.id);
    return { success: true };
  } catch (error) {
    return { error: '로그인 중 오류가 발생했습니다.' };
  }
}

// app/login/page.tsx
'use client';

import { useActionState } from 'react';
import { login } from './actions';

export default function LoginPage() {
  const [state, formAction, isPending] = useActionState(login, {});

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button disabled={isPending}>
        {isPending ? '로그인 중...' : '로그인'}
      </button>
      {state.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```

## 실전 패턴

### 폼 검증과 Actions

Zod를 사용한 스키마 검증:

```tsx
// app/actions.ts
'use server';

import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(1, '제목은 필수입니다.').max(100, '제목은 100자 이하여야 합니다.'),
  content: z.string().min(10, '내용은 최소 10자 이상이어야 합니다.'),
  category: z.enum(['tech', 'life', 'review'], {
    errorMap: () => ({ message: '올바른 카테고리를 선택하세요.' })
  })
});

interface FormState {
  errors?: {
    title?: string[];
    content?: string[];
    category?: string[];
  };
  message?: string;
}

export async function createPost(
  previousState: FormState,
  formData: FormData
): Promise<FormState> {
  // 검증
  const validatedFields = postSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    category: formData.get('category')
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors
    };
  }

  // 데이터 생성
  try {
    await db.post.create({
      data: validatedFields.data
    });

    revalidatePath('/posts');
    return { message: '게시글이 작성되었습니다.' };
  } catch (error) {
    return { message: '게시글 작성에 실패했습니다.' };
  }
}

// app/posts/new/page.tsx
'use client';

import { useActionState } from 'react';
import { createPost } from '../actions';

export default function NewPostPage() {
  const [state, formAction, isPending] = useActionState(createPost, {});

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="title">제목</label>
        <input id="title" name="title" />
        {state.errors?.title && (
          <p className="error">{state.errors.title.join(', ')}</p>
        )}
      </div>

      <div>
        <label htmlFor="content">내용</label>
        <textarea id="content" name="content" />
        {state.errors?.content && (
          <p className="error">{state.errors.content.join(', ')}</p>
        )}
      </div>

      <div>
        <label htmlFor="category">카테고리</label>
        <select id="category" name="category">
          <option value="">선택하세요</option>
          <option value="tech">기술</option>
          <option value="life">생활</option>
          <option value="review">리뷰</option>
        </select>
        {state.errors?.category && (
          <p className="error">{state.errors.category.join(', ')}</p>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? '작성 중...' : '게시글 작성'}
      </button>

      {state.message && <p className="success">{state.message}</p>}
    </form>
  );
}
```

### 중첩 폼 처리

여러 Actions를 가진 복잡한 폼:

```tsx
function ProductPage({ product }: { product: Product }) {
  async function updateProduct(formData: FormData) {
    'use server';
    // 제품 정보 업데이트
    await db.product.update({
      where: { id: product.id },
      data: {
        name: formData.get('name'),
        price: Number(formData.get('price'))
      }
    });
    revalidatePath(`/products/${product.id}`);
  }

  async function uploadImage(formData: FormData) {
    'use server';
    // 이미지 업로드
    const file = formData.get('image') as File;
    const url = await uploadToS3(file);
    await db.product.update({
      where: { id: product.id },
      data: { imageUrl: url }
    });
    revalidatePath(`/products/${product.id}`);
  }

  async function deleteProduct() {
    'use server';
    // 제품 삭제
    await db.product.delete({
      where: { id: product.id }
    });
    revalidatePath('/products');
    redirect('/products');
  }

  return (
    <div>
      {/* 제품 정보 수정 폼 */}
      <form action={updateProduct}>
        <input name="name" defaultValue={product.name} />
        <input name="price" type="number" defaultValue={product.price} />
        <button type="submit">정보 수정</button>
      </form>

      {/* 이미지 업로드 폼 */}
      <form action={uploadImage}>
        <input name="image" type="file" accept="image/*" />
        <button type="submit">이미지 업로드</button>
      </form>

      {/* 삭제 버튼 */}
      <form action={deleteProduct}>
        <button
          type="submit"
          onClick={(e) => {
            if (!confirm('정말 삭제하시겠습니까?')) {
              e.preventDefault();
            }
          }}
        >
          제품 삭제
        </button>
      </form>
    </div>
  );
}
```

### 파일 업로드

Server Actions를 사용한 파일 업로드:

```tsx
// app/actions.ts
'use server';

export async function uploadFile(formData: FormData) {
  const file = formData.get('file') as File;

  if (!file) {
    return { error: '파일을 선택하세요.' };
  }

  // 파일 크기 제한 (5MB)
  if (file.size > 5 * 1024 * 1024) {
    return { error: '파일 크기는 5MB 이하여야 합니다.' };
  }

  // 파일 타입 검증
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
  if (!allowedTypes.includes(file.type)) {
    return { error: '지원하지 않는 파일 형식입니다.' };
  }

  try {
    // ArrayBuffer로 변환
    const bytes = await file.arrayBuffer();
    const buffer = Buffer.from(bytes);

    // S3 또는 다른 스토리지에 업로드
    const fileName = `${Date.now()}-${file.name}`;
    const url = await uploadToStorage(fileName, buffer, file.type);

    // DB에 저장
    await db.upload.create({
      data: {
        fileName: file.name,
        fileSize: file.size,
        fileType: file.type,
        url
      }
    });

    return { success: true, url };
  } catch (error) {
    return { error: '파일 업로드에 실패했습니다.' };
  }
}

// app/upload/page.tsx
'use client';

import { useActionState } from 'react';
import { uploadFile } from './actions';

export default function UploadPage() {
  const [state, formAction, isPending] = useActionState(uploadFile, {});

  return (
    <form action={formAction}>
      <input
        type="file"
        name="file"
        accept="image/*"
        disabled={isPending}
      />
      <button type="submit" disabled={isPending}>
        {isPending ? '업로드 중...' : '업로드'}
      </button>

      {state.error && <p className="error">{state.error}</p>}
      {state.success && (
        <div>
          <p>업로드 완료!</p>
          <img src={state.url} alt="Uploaded" />
        </div>
      )}
    </form>
  );
}
```

### 다중 액션 처리

하나의 폼에서 여러 액션을 처리:

```tsx
// app/editor/page.tsx
'use client';

import { useActionState } from 'react';

type EditorAction = 'save' | 'publish' | 'preview';

interface EditorState {
  message?: string;
  previewUrl?: string;
}

export default function EditorPage() {
  const [state, formAction, isPending] = useActionState(
    async (previousState: EditorState, formData: FormData) => {
      const action = formData.get('action') as EditorAction;
      const title = formData.get('title') as string;
      const content = formData.get('content') as string;

      switch (action) {
        case 'save':
          await api.saveDraft({ title, content });
          return { message: '임시 저장되었습니다.' };

        case 'publish':
          await api.publishPost({ title, content });
          return { message: '게시글이 발행되었습니다.' };

        case 'preview':
          const previewUrl = await api.createPreview({ title, content });
          return { previewUrl };

        default:
          return previousState;
      }
    },
    {}
  );

  return (
    <form action={formAction}>
      <input name="title" placeholder="제목" required />
      <textarea name="content" placeholder="내용" required />

      <div className="actions">
        <button
          type="submit"
          name="action"
          value="save"
          disabled={isPending}
        >
          임시 저장
        </button>

        <button
          type="submit"
          name="action"
          value="preview"
          disabled={isPending}
        >
          미리보기
        </button>

        <button
          type="submit"
          name="action"
          value="publish"
          disabled={isPending}
        >
          발행
        </button>
      </div>

      {state.message && <p>{state.message}</p>}

      {state.previewUrl && (
        <iframe src={state.previewUrl} title="Preview" />
      )}
    </form>
  );
}
```

## 에러 처리와 사용자 피드백

### 에러 바운더리와 Actions

```tsx
// app/error.tsx
'use client';

export default function Error({
  error,
  reset
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="error-container">
      <h2>오류가 발생했습니다</h2>
      <p>{error.message}</p>
      {error.digest && <p>오류 ID: {error.digest}</p>}
      <button onClick={reset}>다시 시도</button>
    </div>
  );
}

// app/posts/[id]/page.tsx
async function deletePost(postId: string) {
  'use server';

  const post = await db.post.findUnique({
    where: { id: postId }
  });

  if (!post) {
    throw new Error('게시글을 찾을 수 없습니다.');
  }

  // 권한 확인
  const session = await getSession();
  if (post.authorId !== session.userId) {
    throw new Error('권한이 없습니다.');
  }

  await db.post.delete({
    where: { id: postId }
  });

  revalidatePath('/posts');
  redirect('/posts');
}
```

### Toast/알림 패턴

전역 토스트를 사용한 피드백:

```tsx
// app/components/ToastProvider.tsx
'use client';

import { createContext, useContext, useState } from 'react';

interface Toast {
  id: string;
  message: string;
  type: 'success' | 'error' | 'info';
}

interface ToastContextType {
  toasts: Toast[];
  addToast: (message: string, type: Toast['type']) => void;
  removeToast: (id: string) => void;
}

const ToastContext = createContext<ToastContextType | null>(null);

export function ToastProvider({ children }: { children: React.ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);

  function addToast(message: string, type: Toast['type']) {
    const id = crypto.randomUUID();
    setToasts(prev => [...prev, { id, message, type }]);

    // 3초 후 자동 제거
    setTimeout(() => removeToast(id), 3000);
  }

  function removeToast(id: string) {
    setToasts(prev => prev.filter(toast => toast.id !== id));
  }

  return (
    <ToastContext.Provider value={{ toasts, addToast, removeToast }}>
      {children}
      <div className="toast-container">
        {toasts.map(toast => (
          <div key={toast.id} className={`toast toast-${toast.type}`}>
            {toast.message}
            <button onClick={() => removeToast(toast.id)}>✕</button>
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

export function useToast() {
  const context = useContext(ToastContext);
  if (!context) {
    throw new Error('useToast must be used within ToastProvider');
  }
  return context;
}

// app/components/CommentForm.tsx
'use client';

import { useToast } from './ToastProvider';

export function CommentForm({ postId }: { postId: string }) {
  const { addToast } = useToast();

  const [state, formAction] = useActionState(
    async (previousState, formData: FormData) => {
      try {
        await createComment(postId, formData);
        addToast('댓글이 작성되었습니다.', 'success');
        return { success: true };
      } catch (error) {
        addToast('댓글 작성에 실패했습니다.', 'error');
        return { success: false };
      }
    },
    { success: false }
  );

  return (
    <form action={formAction}>
      <textarea name="content" required />
      <button type="submit">댓글 작성</button>
    </form>
  );
}
```

### 유효성 검사 피드백

실시간 검증 피드백:

```tsx
'use client';

import { useActionState, useState } from 'react';

interface ValidationErrors {
  email?: string;
  password?: string;
}

export function SignupForm() {
  const [clientErrors, setClientErrors] = useState<ValidationErrors>({});

  const [state, formAction, isPending] = useActionState(
    async (previousState, formData: FormData) => {
      // 서버 측 검증
      const email = formData.get('email') as string;
      const password = formData.get('password') as string;

      const errors: ValidationErrors = {};

      if (!email.includes('@')) {
        errors.email = '유효한 이메일 주소를 입력하세요.';
      }

      if (password.length < 8) {
        errors.password = '비밀번호는 최소 8자 이상이어야 합니다.';
      }

      if (Object.keys(errors).length > 0) {
        return { errors };
      }

      try {
        await api.signup({ email, password });
        return { success: true };
      } catch (error) {
        return {
          errors: { email: '이미 사용 중인 이메일입니다.' }
        };
      }
    },
    {}
  );

  // 클라이언트 측 실시간 검증
  function validateEmail(email: string) {
    if (!email) {
      setClientErrors(prev => ({ ...prev, email: undefined }));
    } else if (!email.includes('@')) {
      setClientErrors(prev => ({
        ...prev,
        email: '유효한 이메일 주소를 입력하세요.'
      }));
    } else {
      setClientErrors(prev => ({ ...prev, email: undefined }));
    }
  }

  function validatePassword(password: string) {
    if (!password) {
      setClientErrors(prev => ({ ...prev, password: undefined }));
    } else if (password.length < 8) {
      setClientErrors(prev => ({
        ...prev,
        password: '비밀번호는 최소 8자 이상이어야 합니다.'
      }));
    } else {
      setClientErrors(prev => ({ ...prev, password: undefined }));
    }
  }

  const errors = { ...clientErrors, ...state.errors };

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          name="email"
          type="email"
          onBlur={(e) => validateEmail(e.target.value)}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" className="error">
            {errors.email}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          name="password"
          type="password"
          onBlur={(e) => validatePassword(e.target.value)}
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <p id="password-error" className="error">
            {errors.password}
          </p>
        )}
      </div>

      <button
        type="submit"
        disabled={isPending || Object.keys(clientErrors).length > 0}
      >
        {isPending ? '처리 중...' : '회원가입'}
      </button>
    </form>
  );
}
```

## 성능 최적화

### 불필요한 리렌더링 방지

`useActionState`와 메모이제이션:

```tsx
'use client';

import { useActionState, memo } from 'react';

// 자식 컴포넌트 메모이제이션
const FormFields = memo(function FormFields({
  disabled
}: {
  disabled: boolean
}) {
  console.log('FormFields 렌더링');

  return (
    <fieldset disabled={disabled}>
      <input name="title" placeholder="제목" />
      <textarea name="content" placeholder="내용" />
    </fieldset>
  );
});

export function PostForm() {
  const [state, formAction, isPending] = useActionState(
    async (previousState, formData: FormData) => {
      await api.createPost({
        title: formData.get('title'),
        content: formData.get('content')
      });
      return { success: true };
    },
    { success: false }
  );

  // isPending이 변경될 때만 FormFields 리렌더링
  return (
    <form action={formAction}>
      <FormFields disabled={isPending} />
      <button type="submit" disabled={isPending}>
        게시글 작성
      </button>
    </form>
  );
}
```

### 디바운싱

검색 폼에 디바운싱 적용:

```tsx
'use client';

import { useActionState, useCallback, useRef } from 'react';

export function SearchForm() {
  const [results, searchAction, isPending] = useActionState(
    async (previousState, formData: FormData) => {
      const query = formData.get('query') as string;
      const results = await api.search(query);
      return results;
    },
    []
  );

  const debounceTimerRef = useRef<NodeJS.Timeout>();

  const handleSearch = useCallback((formElement: HTMLFormElement) => {
    // 이전 타이머 취소
    if (debounceTimerRef.current) {
      clearTimeout(debounceTimerRef.current);
    }

    // 300ms 후 검색 실행
    debounceTimerRef.current = setTimeout(() => {
      const formData = new FormData(formElement);
      searchAction(formData);
    }, 300);
  }, [searchAction]);

  return (
    <>
      <form
        action={searchAction}
        onChange={(e) => handleSearch(e.currentTarget)}
      >
        <input
          name="query"
          placeholder="검색어를 입력하세요"
          autoComplete="off"
        />
      </form>

      {isPending && <p>검색 중...</p>}

      <ul>
        {results.map((result) => (
          <li key={result.id}>{result.title}</li>
        ))}
      </ul>
    </>
  );
}
```

### 스로틀링

좋아요 버튼에 스로틀링 적용:

```tsx
'use client';

import { useOptimistic, useRef } from 'react';

export function LikeButton({
  postId,
  initialLikes,
  initialIsLiked
}: {
  postId: string;
  initialLikes: number;
  initialIsLiked: boolean;
}) {
  const [optimisticState, updateOptimistic] = useOptimistic(
    { likes: initialLikes, isLiked: initialIsLiked },
    (state, isLiked: boolean) => ({
      likes: state.likes + (isLiked ? 1 : -1),
      isLiked
    })
  );

  const lastCallRef = useRef(0);
  const throttleDelay = 1000; // 1초

  async function handleLike() {
    const now = Date.now();

    // 스로틀링: 마지막 호출로부터 1초 이내면 무시
    if (now - lastCallRef.current < throttleDelay) {
      return;
    }

    lastCallRef.current = now;

    const newIsLiked = !optimisticState.isLiked;
    updateOptimistic(newIsLiked);

    try {
      await api.toggleLike(postId, newIsLiked);
    } catch (error) {
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

### 요청 중복 방지

여러 번 클릭해도 한 번만 실행:

```tsx
'use client';

import { useActionState, useRef } from 'react';

export function PaymentForm({ amount }: { amount: number }) {
  const processingRef = useRef(false);

  const [state, formAction, isPending] = useActionState(
    async (previousState, formData: FormData) => {
      // 이미 처리 중이면 무시
      if (processingRef.current) {
        return previousState;
      }

      processingRef.current = true;

      try {
        const result = await api.processPayment({
          amount,
          cardNumber: formData.get('cardNumber')
        });

        return { success: true, orderId: result.orderId };
      } catch (error) {
        return { success: false, error: error.message };
      } finally {
        processingRef.current = false;
      }
    },
    { success: false }
  );

  return (
    <form action={formAction}>
      <input name="cardNumber" placeholder="카드 번호" required />
      <p>결제 금액: {amount.toLocaleString()}원</p>
      <button type="submit" disabled={isPending}>
        {isPending ? '결제 처리 중...' : '결제하기'}
      </button>

      {state.success && (
        <p>결제 완료! 주문번호: {state.orderId}</p>
      )}
      {state.error && (
        <p className="error">{state.error}</p>
      )}
    </form>
  );
}
```

## 결론

React Actions는 폼 처리와 데이터 변이를 크게 개선하는 강력한 패턴입니다. 다음 체크리스트로 올바르게 사용하고 있는지 확인하세요:

### Actions 패턴 체크리스트

**기본 사용:**
- [ ] 폼 제출 시 `action` 속성 사용
- [ ] `useActionState`로 폼 상태 관리
- [ ] `useFormStatus`는 폼 내부 컴포넌트에서만 사용
- [ ] pending 상태 시 버튼 비활성화

**에러 처리:**
- [ ] 예상 가능한 에러는 반환값으로 처리
- [ ] 예상하지 못한 에러는 Error Boundary로 처리
- [ ] 사용자에게 명확한 에러 메시지 제공
- [ ] 서버와 클라이언트 양쪽에서 검증

**낙관적 업데이트:**
- [ ] 빠른 피드백이 필요한 곳에 `useOptimistic` 사용
- [ ] 실패 시 자동 롤백 활용
- [ ] 네트워크 상태 고려

**Server Actions (Next.js):**
- [ ] 'use server' 지시어 올바르게 사용
- [ ] 데이터 변경 후 `revalidatePath` 또는 `revalidateTag` 호출
- [ ] 민감한 작업은 권한 확인
- [ ] 필요시 `redirect` 사용

**성능:**
- [ ] 불필요한 리렌더링 방지 (메모이제이션)
- [ ] 검색 등에는 디바운싱 적용
- [ ] 중복 요청 방지 (스로틀링, 플래그)
- [ ] 큰 폼은 컴포넌트 분리

**접근성:**
- [ ] 적절한 ARIA 속성 사용
- [ ] pending 상태 시 스크린 리더 피드백
- [ ] 에러 메시지와 입력 필드 연결 (`aria-describedby`)
- [ ] 키보드 네비게이션 지원

### Best Practices

1. **선언적 코드 작성**: 이벤트 핸들러보다 `action` 속성 우선 사용
2. **점진적 향상**: JavaScript 비활성화 시에도 작동하도록 구현
3. **명확한 피드백**: 사용자가 현재 상태를 항상 인지할 수 있도록
4. **에러 복구**: 에러 발생 시 다시 시도할 수 있는 방법 제공
5. **타입 안정성**: TypeScript로 폼 데이터와 상태 타입 정의

Actions 패턴을 익히면 더 간결하고 유지보수하기 쉬운 React 코드를 작성할 수 있습니다. 공식 문서와 함께 이 가이드를 참고하여 실전에 적용해보세요!

## 참고 자료

- [React 19 공식 문서 - Actions](https://react.dev/reference/react/use-server)
- [Next.js 문서 - Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [React useActionState 문서](https://react.dev/reference/react/useActionState)
- [React useFormStatus 문서](https://react.dev/reference/react-dom/hooks/useFormStatus)
- [React useOptimistic 문서](https://react.dev/reference/react/useOptimistic)
