---
title: "Next.js 15 완벽 가이드 - App Router 마스터하기"
date: 2025-11-15 16:00:00 +0900
categories: [Next.js, React]
tags: [nextjs, nextjs15, app-router, react-server-components, server-actions, streaming, suspense, next, 넥스트, 리액트]
author: changsu
toc: true
comments: true
description: Next.js 15의 모든 것을 다룹니다. App Router의 핵심 개념부터 React Server Components, Server Actions, 최신 데이터 페칭 전략까지. Turbopack과 성능 최적화를 포함한 실전 예제로 Next.js 15를 완벽히 마스터하세요.
---

## 들어가며

Next.js 15가 정식 출시되면서 React 기반 웹 개발의 새로운 장이 열렸습니다. React 19 지원, 안정화된 Turbopack, 그리고 완전히 새로워진 캐싱 전략까지 많은 변화가 있었습니다.

특히 **App Router**는 Pages Router와는 완전히 다른 패러다임을 제시합니다. "언제 Server Component를 쓰고 언제 Client Component를 써야 하나요?", "Server Actions는 어떻게 활용하나요?", "데이터 페칭 전략이 왜 바뀌었나요?" 같은 질문들이 끊임없이 나옵니다.

이번 포스팅에서는 Next.js 15의 핵심 개념부터 실전 활용법까지 모든 것을 다룹니다. Pages Router에서 마이그레이션하는 분들도, 처음 Next.js를 시작하는 분들도 이 글 하나면 충분할 것입니다.

## Next.js 15 주요 변경사항

### React 19 RC 지원

Next.js 15는 React 19 Release Candidate를 지원합니다:

```json
{
  "dependencies": {
    "react": "^19.0.0-rc",
    "react-dom": "^19.0.0-rc",
    "next": "^15.0.0"
  }
}
```

**주요 개선사항:**
- **React Compiler (실험적)**: 자동 최적화로 `useMemo`, `useCallback` 수동 작성 감소
- **Improved Hydration Errors**: 더욱 명확한 에러 메시지와 디버깅 정보
- **Server Components HMR**: 개발 중 fetch 응답 재사용으로 API 호출 감소

### Turbopack Dev 안정화

Turbopack이 개발 서버에서 정식 사용 가능해졌습니다:

```bash
# package.json
{
  "scripts": {
    "dev": "next dev --turbo"
  }
}
```

**성능 향상:**
- 로컬 서버 시작 속도 최대 76.7% 개선
- Fast Refresh 코드 업데이트 최대 96.3% 개선
- 초기 라우트 컴파일 시간 단축

### 캐싱 모델 변경 (중요!)

Next.js 15는 **명시적 캐싱**으로 전환되었습니다:

```typescript
// ❌ Next.js 14: 기본적으로 캐시됨
const data = await fetch('https://api.example.com/data');

// ✅ Next.js 15: 기본적으로 캐시 안 됨
const data = await fetch('https://api.example.com/data');

// 캐싱을 원한다면 명시적으로 지정
const cachedData = await fetch('https://api.example.com/data', {
  cache: 'force-cache'
});
```

**변경된 기본값:**
- `fetch` 요청: 캐시 안 됨 (이전: 캐시됨)
- GET Route Handlers: 캐시 안 됨 (이전: 캐시됨)
- Client Router Cache: `staleTime`이 `0`으로 변경 (이전: `30초`)

### Async Request APIs

동적 API들이 이제 비동기로 동작합니다:

```typescript
// ❌ Next.js 14
import { cookies, headers } from 'next/headers';

export default function Page({ params, searchParams }) {
  const cookieStore = cookies();
  const headersList = headers();
  const { id } = params;
}

// ✅ Next.js 15
import { cookies, headers } from 'next/headers';

export default async function Page({
  params,
  searchParams
}: {
  params: Promise<{ id: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}) {
  const cookieStore = await cookies();
  const headersList = await headers();
  const { id } = await params;
  const query = await searchParams;
}
```

## App Router vs Pages Router

### 구조적 차이

```bash
# Pages Router (기존)
pages/
  ├── index.tsx          # /
  ├── about.tsx          # /about
  ├── blog/
  │   ├── [id].tsx       # /blog/:id
  │   └── index.tsx      # /blog
  └── api/
      └── users.ts       # /api/users

# App Router (새로운 방식)
app/
  ├── page.tsx           # /
  ├── layout.tsx         # 루트 레이아웃
  ├── about/
  │   └── page.tsx       # /about
  ├── blog/
  │   ├── [id]/
  │   │   └── page.tsx   # /blog/:id
  │   ├── page.tsx       # /blog
  │   ├── layout.tsx     # /blog 레이아웃
  │   └── loading.tsx    # 로딩 UI
  └── api/
      └── users/
          └── route.ts   # /api/users
```

### 주요 차이점

| 구분 | Pages Router | App Router |
|------|-------------|-----------|
| **라우팅** | 파일 기반 | 폴더 기반 |
| **렌더링** | SSR, SSG, ISR | RSC + 기존 방식 |
| **레이아웃** | `_app.tsx`로 관리 | 중첩 `layout.tsx` |
| **데이터 페칭** | `getServerSideProps` 등 | `async/await` 직접 사용 |
| **API Routes** | `pages/api/*.ts` | `app/*/route.ts` |
| **로딩 상태** | 수동 구현 | `loading.tsx` |
| **에러 처리** | `_error.tsx` | `error.tsx`, `not-found.tsx` |

### 언제 App Router를 사용해야 할까?

**App Router 추천:**
- 새로운 프로젝트 시작
- Server Components의 이점 활용 (번들 크기 감소, 서버 데이터 직접 접근)
- 복잡한 레이아웃 구조
- 스트리밍과 Suspense 활용

**Pages Router 유지:**
- 기존 프로젝트 (점진적 마이그레이션 가능)
- Incremental Static Regeneration (ISR) 의존도가 높은 경우
- App Router로 마이그레이션할 시간이 없는 경우

## React Server Components (RSC) 심층 이해

### Server Component vs Client Component

```tsx
// app/components/ServerComponent.tsx
// 기본적으로 모든 컴포넌트는 Server Component

import { db } from '@/lib/database';

export default async function UserList() {
  // 서버에서 직접 데이터베이스 접근 가능
  const users = await db.user.findMany();

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

```tsx
// app/components/ClientComponent.tsx
'use client'; // 이 지시어가 있으면 Client Component

import { useState } from 'react';

export default function Counter() {
  // 상태, 이벤트 핸들러 등 브라우저 API 사용 가능
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      클릭 횟수: {count}
    </button>
  );
}
```

### Server Component의 장점

**1. 번들 크기 감소**

```tsx
// Server Component에서 대용량 라이브러리 사용
import { marked } from 'marked'; // 큰 마크다운 라이브러리
import hljs from 'highlight.js';  // 큰 코드 하이라이팅 라이브러리

export default async function BlogPost({ slug }: { slug: string }) {
  const post = await getPost(slug);

  // 서버에서 처리되므로 클라이언트 번들에 포함되지 않음
  const html = marked(post.content);
  const highlighted = hljs.highlightAuto(html).value;

  return <article dangerouslySetInnerHTML={{ __html: highlighted }} />;
}
```

**2. 서버 리소스 직접 접근**

```tsx
// app/dashboard/page.tsx
import { cookies } from 'next/headers';
import { db } from '@/lib/database';
import { redis } from '@/lib/redis';

export default async function Dashboard() {
  const cookieStore = await cookies();
  const sessionId = cookieStore.get('sessionId')?.value;

  // 데이터베이스, Redis, 파일 시스템 등 직접 접근
  const user = await db.user.findUnique({
    where: { sessionId }
  });

  const cachedData = await redis.get(`user:${user.id}`);

  return <div>환영합니다, {user.name}님!</div>;
}
```

**3. 자동 코드 스플리팅**

```tsx
// app/page.tsx
import HeavyComponent from './HeavyComponent'; // Server Component
import LightComponent from './LightComponent'; // Client Component

export default function Home() {
  return (
    <div>
      {/* HeavyComponent는 서버에서 렌더링되어 HTML로 전달 */}
      <HeavyComponent />

      {/* LightComponent만 클라이언트 번들에 포함 */}
      <LightComponent />
    </div>
  );
}
```

### Client Component는 언제 사용할까?

**필수 사용 케이스:**

```tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';

export default function InteractiveComponent() {
  // 1. 상태 관리
  const [value, setValue] = useState('');

  // 2. 라이프사이클 훅
  useEffect(() => {
    console.log('컴포넌트 마운트됨');
  }, []);

  // 3. 이벤트 핸들러
  const handleClick = () => {
    alert('클릭!');
  };

  // 4. 브라우저 API
  const handleShare = async () => {
    await navigator.share({
      title: '공유하기',
      url: window.location.href,
    });
  };

  // 5. Custom Hooks
  const router = useRouter();

  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      <button onClick={handleClick}>클릭</button>
      <button onClick={handleShare}>공유</button>
    </div>
  );
}
```

### 컴포넌트 구성 패턴

**패턴 1: Server Component 내에 Client Component**

```tsx
// app/page.tsx (Server Component)
import ClientCounter from './ClientCounter';

export default async function Page() {
  const data = await fetch('https://api.example.com/data');

  return (
    <div>
      <h1>서버에서 렌더링</h1>
      {/* Client Component 삽입 */}
      <ClientCounter initialCount={data.count} />
    </div>
  );
}
```

**패턴 2: Client Component에 Server Component 전달 (Composition)**

```tsx
// app/components/ClientWrapper.tsx
'use client';

export default function ClientWrapper({
  children
}: {
  children: React.ReactNode
}) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>토글</button>
      {isOpen && children}
    </div>
  );
}

// app/page.tsx
import ClientWrapper from './components/ClientWrapper';
import ServerComponent from './components/ServerComponent';

export default function Page() {
  return (
    <ClientWrapper>
      {/* Server Component를 children으로 전달 */}
      <ServerComponent />
    </ClientWrapper>
  );
}
```

**패턴 3: Context Provider는 별도 Client Component로**

```tsx
// app/providers/ThemeProvider.tsx
'use client';

import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext<{
  theme: string;
  setTheme: (theme: string) => void;
}>({
  theme: 'light',
  setTheme: () => {},
});

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);

// app/layout.tsx
import { ThemeProvider } from './providers/ThemeProvider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

## Server Actions 실전 활용

### Server Actions란?

Server Actions는 클라이언트에서 서버 함수를 직접 호출할 수 있게 하는 기능입니다. API Route를 만들지 않고도 서버 로직을 실행할 수 있습니다.

### 기본 사용법

```tsx
// app/actions/user.ts
'use server';

import { db } from '@/lib/database';
import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  // 서버에서만 실행되는 코드
  const user = await db.user.create({
    data: { name, email }
  });

  // 캐시 재검증
  revalidatePath('/users');

  return { success: true, user };
}

export async function deleteUser(userId: string) {
  await db.user.delete({
    where: { id: userId }
  });

  revalidatePath('/users');
  return { success: true };
}
```

### Form과 함께 사용하기

```tsx
// app/users/new/page.tsx
import { createUser } from '@/app/actions/user';

export default function NewUserPage() {
  return (
    <form action={createUser}>
      <input
        type="text"
        name="name"
        placeholder="이름"
        required
      />
      <input
        type="email"
        name="email"
        placeholder="이메일"
        required
      />
      <button type="submit">사용자 생성</button>
    </form>
  );
}
```

### Client Component에서 사용하기

```tsx
// app/components/UserForm.tsx
'use client';

import { createUser } from '@/app/actions/user';
import { useFormStatus } from 'react-dom';
import { useState } from 'react';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '생성 중...' : '사용자 생성'}
    </button>
  );
}

export default function UserForm() {
  const [message, setMessage] = useState('');

  async function handleSubmit(formData: FormData) {
    const result = await createUser(formData);

    if (result.success) {
      setMessage('사용자가 생성되었습니다!');
    }
  }

  return (
    <form action={handleSubmit}>
      <input type="text" name="name" placeholder="이름" required />
      <input type="email" name="email" placeholder="이메일" required />
      <SubmitButton />
      {message && <p>{message}</p>}
    </form>
  );
}
```

### Progressive Enhancement

Server Actions는 JavaScript 없이도 작동합니다:

```tsx
// app/components/TodoForm.tsx
import { addTodo } from '@/app/actions/todo';

export default function TodoForm() {
  return (
    <form action={addTodo}>
      <input
        type="text"
        name="title"
        placeholder="할 일 추가"
        required
      />
      {/* JavaScript가 비활성화되어도 작동 */}
      <button type="submit">추가</button>
    </form>
  );
}
```

### 에러 처리와 검증

```tsx
// app/actions/user.ts
'use server';

import { z } from 'zod';
import { db } from '@/lib/database';

const userSchema = z.object({
  name: z.string().min(2, '이름은 최소 2자 이상이어야 합니다'),
  email: z.string().email('유효한 이메일을 입력하세요'),
  age: z.number().min(18, '18세 이상이어야 합니다'),
});

export async function createUser(formData: FormData) {
  try {
    // 검증
    const data = userSchema.parse({
      name: formData.get('name'),
      email: formData.get('email'),
      age: Number(formData.get('age')),
    });

    // 중복 확인
    const existing = await db.user.findUnique({
      where: { email: data.email }
    });

    if (existing) {
      return {
        success: false,
        error: '이미 존재하는 이메일입니다'
      };
    }

    // 생성
    const user = await db.user.create({ data });

    return { success: true, user };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return {
        success: false,
        error: error.errors[0].message
      };
    }

    return {
      success: false,
      error: '사용자 생성에 실패했습니다'
    };
  }
}
```

### useActionState로 상태 관리

```tsx
// app/components/UserForm.tsx
'use client';

import { useActionState } from 'react';
import { createUser } from '@/app/actions/user';

const initialState = {
  success: false,
  error: '',
  user: null,
};

export default function UserForm() {
  const [state, formAction, isPending] = useActionState(
    createUser,
    initialState
  );

  return (
    <form action={formAction}>
      <input type="text" name="name" placeholder="이름" required />
      <input type="email" name="email" placeholder="이메일" required />
      <input type="number" name="age" placeholder="나이" required />

      <button type="submit" disabled={isPending}>
        {isPending ? '생성 중...' : '사용자 생성'}
      </button>

      {state.error && (
        <p className="error">{state.error}</p>
      )}

      {state.success && (
        <p className="success">
          사용자 {state.user?.name}님이 생성되었습니다!
        </p>
      )}
    </form>
  );
}
```

## 라우팅 시스템

### 파일 시스템 라우팅

```bash
app/
  ├── page.tsx              # / (홈)
  ├── layout.tsx            # 루트 레이아웃
  ├── loading.tsx           # 루트 로딩
  ├── error.tsx             # 루트 에러
  ├── not-found.tsx         # 404 페이지
  ├── about/
  │   └── page.tsx          # /about
  ├── blog/
  │   ├── page.tsx          # /blog
  │   ├── layout.tsx        # /blog 레이아웃
  │   ├── [slug]/
  │   │   ├── page.tsx      # /blog/:slug
  │   │   └── loading.tsx   # 로딩 UI
  │   └── [...slug]/
  │       └── page.tsx      # /blog/* (Catch-all)
  └── dashboard/
      ├── layout.tsx        # 대시보드 레이아웃
      ├── page.tsx          # /dashboard
      ├── settings/
      │   └── page.tsx      # /dashboard/settings
      └── [team]/
          ├── page.tsx      # /dashboard/:team
          └── [project]/
              └── page.tsx  # /dashboard/:team/:project
```

### 동적 라우트

**단일 동적 세그먼트:**

```tsx
// app/blog/[slug]/page.tsx
interface PageProps {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

export default async function BlogPost({ params }: PageProps) {
  const { slug } = await params;
  const post = await getPost(slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}

// 정적 경로 생성
export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

**다중 동적 세그먼트:**

```tsx
// app/shop/[category]/[product]/page.tsx
interface PageProps {
  params: Promise<{
    category: string;
    product: string;
  }>;
}

export default async function ProductPage({ params }: PageProps) {
  const { category, product } = await params;

  return (
    <div>
      <h1>{category} - {product}</h1>
    </div>
  );
}

export async function generateStaticParams() {
  const products = await getProducts();

  return products.map((product) => ({
    category: product.category,
    product: product.slug,
  }));
}
```

**Catch-all 세그먼트:**

```tsx
// app/docs/[...slug]/page.tsx
interface PageProps {
  params: Promise<{ slug: string[] }>;
}

export default async function DocsPage({ params }: PageProps) {
  const { slug } = await params;
  // /docs/a -> slug = ['a']
  // /docs/a/b -> slug = ['a', 'b']
  // /docs/a/b/c -> slug = ['a', 'b', 'c']

  const doc = await getDoc(slug.join('/'));

  return <div>{doc.content}</div>;
}
```

**Optional Catch-all:**

```tsx
// app/shop/[[...slug]]/page.tsx
interface PageProps {
  params: Promise<{ slug?: string[] }>;
}

export default async function ShopPage({ params }: PageProps) {
  const { slug } = await params;

  // /shop -> slug = undefined
  // /shop/clothes -> slug = ['clothes']
  // /shop/clothes/tops -> slug = ['clothes', 'tops']

  return <div>Shop: {slug?.join('/') || 'All Products'}</div>;
}
```

### Route Groups (그룹화)

괄호를 사용하여 라우트를 그룹화하면 URL에 영향을 주지 않습니다:

```bash
app/
  ├── (marketing)/
  │   ├── layout.tsx        # 마케팅 레이아웃
  │   ├── page.tsx          # / (홈)
  │   ├── about/
  │   │   └── page.tsx      # /about
  │   └── contact/
  │       └── page.tsx      # /contact
  ├── (shop)/
  │   ├── layout.tsx        # 쇼핑 레이아웃
  │   ├── products/
  │   │   └── page.tsx      # /products
  │   └── cart/
  │       └── page.tsx      # /cart
  └── (dashboard)/
      ├── layout.tsx        # 대시보드 레이아웃
      └── dashboard/
          └── page.tsx      # /dashboard
```

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <nav>마케팅 네비게이션</nav>
      {children}
      <footer>마케팅 푸터</footer>
    </div>
  );
}

// app/(dashboard)/layout.tsx
export default function DashboardLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <aside>대시보드 사이드바</aside>
      <main>{children}</main>
    </div>
  );
}
```

### 병렬 라우트 (Parallel Routes)

동시에 여러 페이지를 렌더링할 수 있습니다:

```bash
app/
  └── dashboard/
      ├── layout.tsx
      ├── page.tsx
      ├── @analytics/
      │   └── page.tsx
      ├── @team/
      │   └── page.tsx
      └── @revenue/
          └── page.tsx
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
  revenue,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
  revenue: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <div className="main">{children}</div>
      <div className="grid">
        <div className="analytics">{analytics}</div>
        <div className="team">{team}</div>
        <div className="revenue">{revenue}</div>
      </div>
    </div>
  );
}
```

### 인터셉팅 라우트 (Intercepting Routes)

현재 레이아웃 내에서 다른 라우트의 내용을 보여줄 수 있습니다:

```bash
app/
  ├── feed/
  │   └── page.tsx
  └── photos/
      ├── [id]/
      │   └── page.tsx      # /photos/:id
      └── (..)feed/
          └── page.tsx      # /feed를 모달로 가로채기
```

**인터셉팅 규칙:**
- `(.)` - 같은 레벨
- `(..)` - 한 레벨 위
- `(..)(..)` - 두 레벨 위
- `(...)` - 루트 `app` 디렉토리부터

```tsx
// app/photos/[id]/page.tsx
export default async function PhotoPage({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
  const photo = await getPhoto(id);

  return (
    <div>
      <img src={photo.url} alt={photo.title} />
      <h1>{photo.title}</h1>
    </div>
  );
}

// app/photos/(.)feed/page.tsx
import Modal from '@/components/Modal';

export default function FeedModal() {
  return (
    <Modal>
      <div>Feed content in modal</div>
    </Modal>
  );
}
```

## 레이아웃과 템플릿

### Layout vs Template

```tsx
// app/layout.tsx - 네비게이션 시 상태 유지
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        {/* 한 번만 렌더링되고 재사용됨 */}
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}

// app/template.tsx - 네비게이션 시 재생성
export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <div>
      {/* 페이지 이동할 때마다 새로 렌더링됨 */}
      <AnimatedWrapper>{children}</AnimatedWrapper>
    </div>
  );
}
```

**언제 Template을 사용할까?**
- 페이지 전환 애니메이션
- useEffect 기반 로깅
- 페이지별 독립적인 상태

### 중첩 레이아웃

```tsx
// app/layout.tsx (루트)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <Header />
        {children}
      </body>
    </html>
  );
}

// app/blog/layout.tsx (블로그)
export default function BlogLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="blog-container">
      <aside className="sidebar">
        <BlogSidebar />
      </aside>
      <main>{children}</main>
    </div>
  );
}

// app/blog/[slug]/layout.tsx (블로그 포스트)
export default async function PostLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const post = await getPost(slug);

  return (
    <article>
      <header>
        <h1>{post.title}</h1>
        <time>{post.date}</time>
      </header>
      {children}
      <footer>
        <Comments postId={post.id} />
      </footer>
    </article>
  );
}
```

## 로딩 UI와 Streaming

### loading.tsx

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="loading-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content" />
      <div className="skeleton-sidebar" />
    </div>
  );
}
```

이것은 자동으로 Suspense 경계를 생성합니다:

```tsx
// Next.js가 자동으로 생성하는 구조
<Suspense fallback={<Loading />}>
  <Page />
</Suspense>
```

### 수동 Suspense 경계

더 세밀한 제어가 필요할 때:

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import RevenueChart from './RevenueChart';
import UserList from './UserList';
import Analytics from './Analytics';

function RevenueSkeleton() {
  return <div className="skeleton">수익 로딩 중...</div>;
}

function UserSkeleton() {
  return <div className="skeleton">사용자 로딩 중...</div>;
}

export default function Dashboard() {
  return (
    <div className="dashboard">
      {/* 각 컴포넌트가 독립적으로 스트리밍됨 */}
      <Suspense fallback={<RevenueSkeleton />}>
        <RevenueChart />
      </Suspense>

      <Suspense fallback={<UserSkeleton />}>
        <UserList />
      </Suspense>

      {/* Analytics는 즉시 렌더링 (데이터 페칭 없음) */}
      <Analytics />
    </div>
  );
}
```

### Streaming의 이점

```tsx
// app/products/page.tsx
import { Suspense } from 'react';

async function ProductList() {
  // 3초 걸리는 데이터
  const products = await getProducts(); // 느림

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

async function FeaturedProduct() {
  // 5초 걸리는 데이터
  const featured = await getFeaturedProduct(); // 매우 느림

  return <FeaturedCard product={featured} />;
}

async function Categories() {
  // 0.5초 걸리는 데이터
  const categories = await getCategories(); // 빠름

  return <CategoryList categories={categories} />;
}

export default function ProductsPage() {
  return (
    <div>
      {/* Categories는 0.5초 후 즉시 표시 */}
      <Suspense fallback={<div>카테고리 로딩...</div>}>
        <Categories />
      </Suspense>

      {/* ProductList는 3초 후 표시 */}
      <Suspense fallback={<div>상품 로딩...</div>}>
        <ProductList />
      </Suspense>

      {/* FeaturedProduct는 5초 후 표시 */}
      <Suspense fallback={<div>추천 상품 로딩...</div>}>
        <FeaturedProduct />
      </Suspense>
    </div>
  );
}

// Streaming 없이는 5초를 모두 기다려야 하지만,
// Streaming으로 0.5초부터 점진적으로 화면이 표시됨
```

## 에러 핸들링

### error.tsx

```tsx
// app/dashboard/error.tsx
'use client'; // Error 컴포넌트는 반드시 Client Component

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // 에러 로깅 서비스로 전송
    console.error('Error:', error);
  }, [error]);

  return (
    <div className="error-container">
      <h2>문제가 발생했습니다!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>다시 시도</button>
    </div>
  );
}
```

### global-error.tsx

루트 레이아웃의 에러를 처리합니다:

```tsx
// app/global-error.tsx
'use client';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>전역 에러가 발생했습니다!</h2>
        <p>{error.message}</p>
        <button onClick={reset}>다시 시도</button>
      </body>
    </html>
  );
}
```

### not-found.tsx

```tsx
// app/blog/[slug]/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="not-found">
      <h2>포스트를 찾을 수 없습니다</h2>
      <p>요청하신 블로그 포스트가 존재하지 않습니다.</p>
      <Link href="/blog">블로그 목록으로 돌아가기</Link>
    </div>
  );
}
```

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';

export default async function BlogPost({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    notFound(); // not-found.tsx 렌더링
  }

  return <article>{post.content}</article>;
}
```

### 에러 경계 계층

```bash
app/
  ├── global-error.tsx      # 최상위 에러
  ├── error.tsx             # 루트 에러
  ├── not-found.tsx         # 루트 404
  └── dashboard/
      ├── error.tsx         # /dashboard 에러
      ├── not-found.tsx     # /dashboard 404
      └── settings/
          ├── error.tsx     # /dashboard/settings 에러
          └── not-found.tsx # /dashboard/settings 404
```

## 데이터 페칭 전략

### Server Component에서 직접 페칭

```tsx
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    // Next.js 15: 기본적으로 캐시 안 됨
    // 캐싱을 원한다면 명시적으로 지정
    cache: 'force-cache', // 또는 'no-store'
  });

  if (!res.ok) {
    throw new Error('Failed to fetch posts');
  }

  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();

  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

### Revalidation 전략

**1. Time-based Revalidation:**

```tsx
// 60초마다 재검증
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }
  });

  return res.json();
}

// 또는 페이지 레벨에서
export const revalidate = 60; // 60초

export default async function PostsPage() {
  const posts = await getPosts();
  return <div>...</div>;
}
```

**2. On-demand Revalidation:**

```tsx
// app/actions/posts.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content'),
    }
  });

  // 특정 경로 재검증
  revalidatePath('/posts');

  // 또는 특정 태그 재검증
  revalidateTag('posts');

  return post;
}
```

```tsx
// 태그 기반 캐싱
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  });

  return res.json();
}

async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`, {
    next: { tags: ['posts', `post-${id}`] }
  });

  return res.json();
}
```

### 병렬 데이터 페칭

```tsx
// ❌ 순차 페칭 (느림)
export default async function Page() {
  const user = await getUser();          // 1초
  const posts = await getPosts();        // 1초
  const comments = await getComments();  // 1초
  // 총 3초

  return <div>...</div>;
}

// ✅ 병렬 페칭 (빠름)
export default async function Page() {
  // 동시에 시작
  const userPromise = getUser();
  const postsPromise = getPosts();
  const commentsPromise = getComments();

  // Promise.all로 대기
  const [user, posts, comments] = await Promise.all([
    userPromise,
    postsPromise,
    commentsPromise,
  ]);
  // 총 1초 (가장 느린 요청 기준)

  return <div>...</div>;
}
```

### 순차 페칭 (의존성 있을 때)

```tsx
export default async function Page() {
  // 1. 먼저 사용자 정보 가져오기
  const user = await getUser();

  // 2. 사용자 ID로 포스트 가져오기 (의존성)
  const posts = await getUserPosts(user.id);

  // 3. 첫 번째 포스트의 댓글 가져오기 (의존성)
  const comments = await getPostComments(posts[0].id);

  return <div>...</div>;
}
```

### 데이터 중복 제거 (자동)

Next.js는 같은 요청을 자동으로 중복 제거합니다:

```tsx
// app/page.tsx
async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

async function Header() {
  const user = await getUser('123'); // 첫 번째 요청
  return <header>{user.name}</header>;
}

async function Sidebar() {
  const user = await getUser('123'); // 중복 제거됨 (실제 요청 안 함)
  return <aside>{user.bio}</aside>;
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  );
}
// '123' 사용자 정보는 한 번만 요청됨
```

### unstable_cache 활용

```tsx
import { unstable_cache } from 'next/cache';

const getCachedPosts = unstable_cache(
  async () => {
    const posts = await db.post.findMany();
    return posts;
  },
  ['posts'],                    // 캐시 키
  {
    revalidate: 3600,          // 1시간
    tags: ['posts']            // 태그
  }
);

export default async function PostsPage() {
  const posts = await getCachedPosts();
  return <div>...</div>;
}
```

## 메타데이터 API

### 정적 메타데이터

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: '블로그 포스트',
  description: '블로그 포스트 설명',
  openGraph: {
    title: '블로그 포스트',
    description: '블로그 포스트 설명',
    images: ['/og-image.jpg'],
  },
  twitter: {
    card: 'summary_large_image',
    title: '블로그 포스트',
    description: '블로그 포스트 설명',
    images: ['/twitter-image.jpg'],
  },
};

export default function BlogPost() {
  return <article>...</article>;
}
```

### 동적 메타데이터

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

interface PageProps {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata({
  params
}: PageProps): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
      images: [
        {
          url: post.coverImage,
          width: 1200,
          height: 630,
          alt: post.title,
        }
      ],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
      creator: `@${post.author.twitter}`,
    },
    alternates: {
      canonical: `https://example.com/blog/${slug}`,
    },
  };
}

export default async function BlogPost({ params }: PageProps) {
  const { slug } = await params;
  const post = await getPost(slug);

  return <article>{post.content}</article>;
}
```

### JSON-LD 구조화 데이터

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params;
  const post = await getPost(slug);

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author.name,
      url: post.author.website,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>
        <h1>{post.title}</h1>
        <div>{post.content}</div>
      </article>
    </>
  );
}
```

### 루트 레이아웃 메타데이터

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: '내 사이트',
    template: '%s | 내 사이트', // 하위 페이지에서 사용
  },
  description: '사이트 기본 설명',
  metadataBase: new URL('https://example.com'),
  keywords: ['Next.js', 'React', 'TypeScript'],
  authors: [{ name: '홍길동', url: 'https://example.com' }],
  creator: '홍길동',
  publisher: '내 사이트',
  formatDetection: {
    email: false,
    address: false,
    telephone: false,
  },
  openGraph: {
    type: 'website',
    locale: 'ko_KR',
    siteName: '내 사이트',
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  icons: {
    icon: '/favicon.ico',
    shortcut: '/favicon-16x16.png',
    apple: '/apple-touch-icon.png',
  },
  manifest: '/site.webmanifest',
};

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>{children}</body>
    </html>
  );
}
```

## 성능 최적화

### Partial Prerendering (실험적)

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  experimental: {
    ppr: true, // Partial Prerendering 활성화
  },
};

export default nextConfig;
```

```tsx
// app/products/page.tsx
export const experimental_ppr = true;

// 정적 부분 (빌드 시 생성)
export default async function ProductsPage() {
  return (
    <div>
      <header>
        <h1>상품 목록</h1>
      </header>

      {/* 동적 부분 (요청 시 생성) */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductList />
      </Suspense>

      {/* 정적 부분 */}
      <footer>
        <p>Copyright 2025</p>
      </footer>
    </div>
  );
}
```

### Image 최적화

```tsx
import Image from 'next/image';

export default function ProductCard({ product }) {
  return (
    <div>
      {/* 자동 최적화, lazy loading, responsive */}
      <Image
        src={product.image}
        alt={product.name}
        width={500}
        height={300}
        priority={false} // 첫 화면이 아니면 false
        placeholder="blur" // 블러 플레이스홀더
        blurDataURL={product.blurDataUrl}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      />
      <h3>{product.name}</h3>
    </div>
  );
}

// 외부 이미지 사용 시
// next.config.ts
const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
        pathname: '/products/**',
      },
    ],
  },
};
```

### Dynamic Import (코드 스플리팅)

```tsx
// app/page.tsx
import dynamic from 'next/dynamic';

// Client Component 동적 임포트
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>로딩 중...</div>,
  ssr: false, // 클라이언트에서만 렌더링
});

const Chart = dynamic(() => import('./Chart'), {
  loading: () => <div>차트 로딩 중...</div>,
});

export default function Page() {
  return (
    <div>
      <h1>대시보드</h1>

      {/* 필요할 때만 로드 */}
      <Suspense fallback={<div>로딩...</div>}>
        <Chart />
      </Suspense>

      <HeavyComponent />
    </div>
  );
}
```

### Font 최적화

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko" className={`${inter.variable} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

```css
/* globals.css */
body {
  font-family: var(--font-inter), sans-serif;
}

code {
  font-family: var(--font-roboto-mono), monospace;
}
```

### Third-party Script 최적화

```tsx
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html>
      <body>
        {children}

        {/* Google Analytics */}
        <Script
          src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"
          strategy="afterInteractive" // 페이지 인터랙티브 후 로드
        />
        <Script id="google-analytics" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', 'GA_MEASUREMENT_ID');
          `}
        </Script>
      </body>
    </html>
  );
}
```

### Bundle Analyzer

```bash
npm install @next/bundle-analyzer
```

```typescript
// next.config.ts
import type { NextConfig } from 'next';
import bundleAnalyzer from '@next/bundle-analyzer';

const withBundleAnalyzer = bundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
});

const nextConfig: NextConfig = {
  // 설정...
};

export default withBundleAnalyzer(nextConfig);
```

```json
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true next build"
  }
}
```

## 실무 프로젝트 구조

### 추천 폴더 구조

```bash
project/
├── app/
│   ├── (auth)/                    # Route Group
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── dashboard/
│   │   │   ├── page.tsx
│   │   │   ├── loading.tsx
│   │   │   └── error.tsx
│   │   └── settings/
│   │       └── page.tsx
│   ├── (marketing)/
│   │   ├── layout.tsx
│   │   ├── page.tsx               # 홈
│   │   ├── about/
│   │   │   └── page.tsx
│   │   └── contact/
│   │       └── page.tsx
│   ├── api/                       # API Routes
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts
│   │   └── webhooks/
│   │       └── stripe/
│   │           └── route.ts
│   ├── actions/                   # Server Actions
│   │   ├── user.ts
│   │   ├── post.ts
│   │   └── comment.ts
│   ├── layout.tsx                 # 루트 레이아웃
│   ├── global-error.tsx
│   ├── not-found.tsx
│   └── sitemap.ts                 # 동적 Sitemap
├── components/                    # 공유 컴포넌트
│   ├── ui/                        # UI 컴포넌트
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Modal.tsx
│   ├── forms/                     # 폼 컴포넌트
│   │   ├── LoginForm.tsx
│   │   └── RegisterForm.tsx
│   ├── layout/                    # 레이아웃 컴포넌트
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Sidebar.tsx
│   └── providers/                 # Context Providers
│       ├── ThemeProvider.tsx
│       └── AuthProvider.tsx
├── lib/                           # 유틸리티, 설정
│   ├── db.ts                      # 데이터베이스
│   ├── auth.ts                    # 인증
│   ├── utils.ts                   # 헬퍼 함수
│   └── constants.ts               # 상수
├── hooks/                         # Custom Hooks
│   ├── useUser.ts
│   └── useAuth.ts
├── types/                         # TypeScript 타입
│   ├── user.ts
│   ├── post.ts
│   └── index.ts
├── styles/
│   └── globals.css
├── public/
│   ├── images/
│   └── fonts/
├── middleware.ts                  # 미들웨어
├── next.config.ts
├── tsconfig.json
└── package.json
```

### 환경 변수 관리

```bash
# .env.local
DATABASE_URL="postgresql://..."
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000"

NEXT_PUBLIC_API_URL="https://api.example.com"
NEXT_PUBLIC_GA_ID="G-XXXXXXXXXX"
```

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(1),
  NEXTAUTH_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
  NEXT_PUBLIC_GA_ID: z.string().min(1),
});

export const env = envSchema.parse(process.env);
```

### Middleware 활용

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 인증 체크
  const token = request.cookies.get('session')?.value;

  // 대시보드 접근 시 인증 필요
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  // 로그인 상태에서 로그인 페이지 접근 방지
  if (request.nextUrl.pathname.startsWith('/login')) {
    if (token) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
  }

  // 커스텀 헤더 추가
  const response = NextResponse.next();
  response.headers.set('X-Custom-Header', 'value');

  return response;
}

export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

### TypeScript 설정

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"],
      "@/components/*": ["./components/*"],
      "@/lib/*": ["./lib/*"],
      "@/hooks/*": ["./hooks/*"],
      "@/types/*": ["./types/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Next.js 설정

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  // TypeScript 엄격 모드
  typescript: {
    ignoreBuildErrors: false,
  },

  // ESLint 설정
  eslint: {
    ignoreDuringBuilds: false,
  },

  // 실험적 기능
  experimental: {
    ppr: true,                    // Partial Prerendering
    reactCompiler: true,          // React Compiler
    serverActions: {
      bodySizeLimit: '2mb',
    },
  },

  // 이미지 최적화
  images: {
    formats: ['image/avif', 'image/webp'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
      },
    ],
  },

  // 리다이렉트
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },

  // 헤더 설정
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};

export default nextConfig;
```

## Pages Router에서 마이그레이션하기

### 단계별 마이그레이션

**1단계: Next.js 15로 업그레이드**

```bash
npm install next@latest react@rc react-dom@rc
```

**2단계: 점진적 채택 (Incremental Adoption)**

Next.js는 `app`과 `pages` 디렉토리를 동시에 지원합니다:

```bash
project/
├── app/              # 새로운 페이지는 여기에
│   └── dashboard/
│       └── page.tsx
└── pages/            # 기존 페이지 유지
    ├── index.tsx
    └── blog/
        └── [slug].tsx
```

**3단계: 레이아웃 마이그레이션**

```tsx
// pages/_app.tsx (기존)
export default function App({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}

// app/layout.tsx (새로운 방식)
export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ko">
      <body>
        <Layout>{children}</Layout>
      </body>
    </html>
  );
}
```

**4단계: 데이터 페칭 마이그레이션**

```tsx
// pages/posts/[id].tsx (기존)
export async function getServerSideProps({ params }) {
  const post = await getPost(params.id);
  return { props: { post } };
}

export default function Post({ post }) {
  return <article>{post.content}</article>;
}

// app/posts/[id]/page.tsx (새로운 방식)
export default async function Post({
  params
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params;
  const post = await getPost(id);
  return <article>{post.content}</article>;
}
```

**5단계: API Routes 마이그레이션**

```typescript
// pages/api/users.ts (기존)
export default async function handler(req, res) {
  if (req.method === 'GET') {
    const users = await getUsers();
    res.status(200).json(users);
  }
}

// app/api/users/route.ts (새로운 방식)
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await getUsers();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const data = await request.json();
  const user = await createUser(data);
  return NextResponse.json(user, { status: 201 });
}
```

## 실전 예제: 블로그 시스템

### 완전한 블로그 구현

```bash
app/
├── (blog)/
│   ├── layout.tsx
│   ├── blog/
│   │   ├── page.tsx              # 목록
│   │   ├── [slug]/
│   │   │   ├── page.tsx          # 포스트
│   │   │   ├── loading.tsx
│   │   │   ├── error.tsx
│   │   │   └── not-found.tsx
│   │   └── category/
│   │       └── [category]/
│   │           └── page.tsx
│   └── admin/
│       ├── layout.tsx
│       └── posts/
│           ├── page.tsx          # 관리자 목록
│           ├── new/
│           │   └── page.tsx      # 새 포스트
│           └── [id]/
│               └── edit/
│                   └── page.tsx  # 수정
├── actions/
│   └── blog.ts
└── api/
    └── posts/
        └── route.ts
```

**포스트 목록 페이지:**

```tsx
// app/(blog)/blog/page.tsx
import Link from 'next/link';
import { Suspense } from 'react';

async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }
  });
  return res.json();
}

async function PostList() {
  const posts = await getPosts();

  return (
    <div className="post-grid">
      {posts.map(post => (
        <article key={post.slug} className="post-card">
          <Link href={`/blog/${post.slug}`}>
            <h2>{post.title}</h2>
            <p>{post.excerpt}</p>
            <time>{new Date(post.publishedAt).toLocaleDateString('ko-KR')}</time>
          </Link>
        </article>
      ))}
    </div>
  );
}

function PostSkeleton() {
  return (
    <div className="post-grid">
      {[...Array(6)].map((_, i) => (
        <div key={i} className="skeleton-card" />
      ))}
    </div>
  );
}

export default function BlogPage() {
  return (
    <div className="container">
      <h1>블로그</h1>
      <Suspense fallback={<PostSkeleton />}>
        <PostList />
      </Suspense>
    </div>
  );
}

export const metadata = {
  title: '블로그',
  description: '최신 기술 블로그 포스트',
};
```

**포스트 상세 페이지:**

```tsx
// app/(blog)/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';
import { Suspense } from 'react';
import type { Metadata } from 'next';

interface PageProps {
  params: Promise<{ slug: string }>;
}

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { tags: [`post-${slug}`] }
  });

  if (!res.ok) {
    return null;
  }

  return res.json();
}

async function getRelatedPosts(category: string, currentSlug: string) {
  const res = await fetch(
    `https://api.example.com/posts?category=${category}&exclude=${currentSlug}`,
    { next: { revalidate: 3600 } }
  );
  return res.json();
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    return {
      title: '포스트를 찾을 수 없습니다',
    };
  }

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
      images: [post.coverImage],
    },
  };
}

export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

async function RelatedPosts({ category, currentSlug }: { category: string; currentSlug: string }) {
  const posts = await getRelatedPosts(category, currentSlug);

  if (posts.length === 0) {
    return null;
  }

  return (
    <aside className="related-posts">
      <h3>관련 포스트</h3>
      <ul>
        {posts.map(post => (
          <li key={post.slug}>
            <Link href={`/blog/${post.slug}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </aside>
  );
}

export default async function BlogPost({ params }: PageProps) {
  const { slug } = await params;
  const post = await getPost(slug);

  if (!post) {
    notFound();
  }

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author.name,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />

      <article className="blog-post">
        <header>
          <h1>{post.title}</h1>
          <div className="meta">
            <time>{new Date(post.publishedAt).toLocaleDateString('ko-KR')}</time>
            <span>·</span>
            <span>{post.author.name}</span>
            <span>·</span>
            <span>{post.readingTime}분</span>
          </div>
        </header>

        <div
          className="content"
          dangerouslySetInnerHTML={{ __html: post.content }}
        />

        <footer>
          <div className="tags">
            {post.tags.map(tag => (
              <Link key={tag} href={`/blog/tag/${tag}`}>
                #{tag}
              </Link>
            ))}
          </div>
        </footer>
      </article>

      <Suspense fallback={<div>관련 포스트 로딩 중...</div>}>
        <RelatedPosts category={post.category} currentSlug={slug} />
      </Suspense>
    </>
  );
}
```

**Server Actions로 포스트 생성:**

```tsx
// app/actions/blog.ts
'use server';

import { revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';

const postSchema = z.object({
  title: z.string().min(1, '제목을 입력하세요'),
  content: z.string().min(1, '내용을 입력하세요'),
  excerpt: z.string().max(200),
  category: z.string(),
  tags: z.array(z.string()),
});

export async function createPost(formData: FormData) {
  try {
    const data = postSchema.parse({
      title: formData.get('title'),
      content: formData.get('content'),
      excerpt: formData.get('excerpt'),
      category: formData.get('category'),
      tags: formData.get('tags')?.toString().split(',').map(t => t.trim()),
    });

    const response = await fetch('https://api.example.com/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      throw new Error('포스트 생성 실패');
    }

    const post = await response.json();

    revalidateTag('posts');
    redirect(`/blog/${post.slug}`);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return {
        success: false,
        error: error.errors[0].message,
      };
    }

    return {
      success: false,
      error: '포스트 생성에 실패했습니다',
    };
  }
}

export async function updatePost(id: string, formData: FormData) {
  try {
    const data = postSchema.parse({
      title: formData.get('title'),
      content: formData.get('content'),
      excerpt: formData.get('excerpt'),
      category: formData.get('category'),
      tags: formData.get('tags')?.toString().split(',').map(t => t.trim()),
    });

    const response = await fetch(`https://api.example.com/posts/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      throw new Error('포스트 수정 실패');
    }

    const post = await response.json();

    revalidateTag('posts');
    revalidateTag(`post-${post.slug}`);
    redirect(`/blog/${post.slug}`);
  } catch (error) {
    return {
      success: false,
      error: '포스트 수정에 실패했습니다',
    };
  }
}

export async function deletePost(id: string) {
  try {
    const response = await fetch(`https://api.example.com/posts/${id}`, {
      method: 'DELETE',
    });

    if (!response.ok) {
      throw new Error('포스트 삭제 실패');
    }

    revalidateTag('posts');
    redirect('/blog');
  } catch (error) {
    return {
      success: false,
      error: '포스트 삭제에 실패했습니다',
    };
  }
}
```

**포스트 작성 폼:**

```tsx
// app/(blog)/admin/posts/new/page.tsx
'use client';

import { useActionState } from 'react';
import { createPost } from '@/app/actions/blog';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '생성 중...' : '포스트 생성'}
    </button>
  );
}

const initialState = {
  success: false,
  error: '',
};

export default function NewPostPage() {
  const [state, formAction] = useActionState(createPost, initialState);

  return (
    <div className="container">
      <h1>새 포스트 작성</h1>

      <form action={formAction} className="post-form">
        <div className="form-group">
          <label htmlFor="title">제목</label>
          <input
            type="text"
            id="title"
            name="title"
            required
            placeholder="포스트 제목을 입력하세요"
          />
        </div>

        <div className="form-group">
          <label htmlFor="excerpt">요약</label>
          <textarea
            id="excerpt"
            name="excerpt"
            rows={3}
            required
            maxLength={200}
            placeholder="포스트 요약 (최대 200자)"
          />
        </div>

        <div className="form-group">
          <label htmlFor="category">카테고리</label>
          <select id="category" name="category" required>
            <option value="">선택하세요</option>
            <option value="react">React</option>
            <option value="nextjs">Next.js</option>
            <option value="typescript">TypeScript</option>
            <option value="javascript">JavaScript</option>
          </select>
        </div>

        <div className="form-group">
          <label htmlFor="tags">태그 (쉼표로 구분)</label>
          <input
            type="text"
            id="tags"
            name="tags"
            placeholder="react, hooks, performance"
          />
        </div>

        <div className="form-group">
          <label htmlFor="content">내용</label>
          <textarea
            id="content"
            name="content"
            rows={20}
            required
            placeholder="포스트 내용을 Markdown으로 작성하세요"
          />
        </div>

        {state.error && (
          <div className="error-message">{state.error}</div>
        )}

        <div className="form-actions">
          <SubmitButton />
        </div>
      </form>
    </div>
  );
}
```

## FAQ (자주 묻는 질문)

### Q1: Server Component와 Client Component 중 무엇을 기본으로 사용해야 하나요?

**A:** Server Component를 기본으로 사용하고, 상호작용이 필요한 경우에만 Client Component로 전환하세요. Server Component는 번들 크기를 줄이고 서버 리소스에 직접 접근할 수 있습니다.

### Q2: Next.js 15에서 캐싱이 기본적으로 비활성화되었는데, 성능에 문제가 없나요?

**A:** 명시적 캐싱으로의 전환은 더 예측 가능한 동작을 제공합니다. 필요한 곳에 `cache: 'force-cache'`나 `revalidate` 옵션을 추가하면 됩니다. 이는 불필요한 캐싱으로 인한 버그를 방지합니다.

### Q3: Server Actions는 언제 사용하고 API Routes는 언제 사용하나요?

**A:**
- **Server Actions**: 폼 제출, 데이터 변경 등 내부 로직
- **API Routes**: 외부 API로 노출해야 하는 경우, Webhook, 서드파티 통합

### Q4: Pages Router 프로젝트를 App Router로 완전히 마이그레이션해야 하나요?

**A:** 아니요. Next.js는 점진적 마이그레이션을 지원합니다. 두 시스템을 동시에 사용할 수 있으므로 새 기능은 App Router로 개발하고, 기존 페이지는 필요할 때 마이그레이션하세요.

### Q5: Turbopack을 프로덕션에서 사용할 수 있나요?

**A:** Next.js 15에서 Turbopack Dev는 안정화되었지만, 프로덕션 빌드는 아직 실험적입니다. Next.js 15.4부터 Turbopack 빌드가 베타로 제공되며, 점진적으로 안정화되고 있습니다.

### Q6: 'use client'를 최상위에 두면 하위 컴포넌트도 모두 Client Component가 되나요?

**A:** 네. 'use client'가 있는 파일이 import하는 모든 컴포넌트는 Client Component로 취급됩니다. 하지만 `children` props로 전달된 Server Component는 여전히 서버에서 렌더링됩니다.

### Q7: Dynamic Route에서 generateStaticParams를 꼭 사용해야 하나요?

**A:** 아니요. 하지만 사용하면 빌드 시 정적 페이지를 미리 생성하여 성능이 향상됩니다. 동적 경로는 런타임에도 생성됩니다.

### Q8: Server Component에서 useState를 사용할 수 없는데 상태는 어떻게 관리하나요?

**A:** Server Component는 상태가 필요 없습니다. 상태가 필요한 부분만 Client Component로 분리하세요. Composition 패턴을 활용하여 Server Component 안에 Client Component를 배치할 수 있습니다.

## 마치며

Next.js 15와 App Router는 React 웹 개발의 새로운 패러다임을 제시합니다. Server Components로 번들 크기를 줄이고, Server Actions로 간단하게 서버 로직을 실행하며, Streaming으로 빠른 사용자 경험을 제공할 수 있습니다.

처음에는 Pages Router와의 차이 때문에 혼란스러울 수 있지만, App Router의 철학을 이해하면 훨씬 강력하고 유지보수하기 쉬운 애플리케이션을 만들 수 있습니다.

**핵심 포인트:**
- Server Component를 기본으로, 필요할 때만 Client Component 사용
- 명시적 캐싱으로 예측 가능한 동작 보장
- Server Actions로 간단한 서버 로직 처리
- Suspense와 Streaming으로 사용자 경험 개선
- 점진적 마이그레이션으로 안전한 전환

이 글이 Next.js 15와 App Router를 마스터하는 데 도움이 되기를 바랍니다. 실제 프로젝트에 적용하면서 더 많은 것을 배워나가세요!

## 참고 자료

- [Next.js 15 공식 문서](https://nextjs.org/docs)
- [Next.js 15 Release Notes](https://nextjs.org/blog/next-15)
- [Next.js 15.4 Release](https://nextjs.org/blog/next-15-4)
- [Next.js 15.5 Release](https://nextjs.org/blog/next-15-5)
- [React 19 RC 문서](https://react.dev/blog/2024/04/25/react-19)
- [React Server Components 설명](https://react.dev/reference/rsc/server-components)
- [Vercel 블로그 - Next.js 15](https://vercel.com/blog/next-js-15)
- [App Router 마이그레이션 가이드](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration)
- [Server Actions 문서](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [Turbopack 문서](https://nextjs.org/docs/architecture/turbopack)
