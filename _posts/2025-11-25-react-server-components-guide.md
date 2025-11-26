---
title: "React Server Components 완벽 가이드 - 개념부터 실전까지"
description: "React Server Components의 개념, 동작 원리, 작성법을 상세히 알아봅니다. Client Components와의 조합 방법, Next.js에서의 활용, 성능 최적화 전략까지 실전 예제와 함께 다룹니다."
date: 2025-11-25 10:00:00 +0900
categories: [Frontend, React]
tags: [react, server-components, nextjs, ssr]
---

React Server Components(RSC)는 React 팀이 제안한 새로운 패러다임으로, 서버와 클라이언트의 장점을 결합하여 웹 애플리케이션의 성능과 사용자 경험을 크게 향상시킵니다. 이 글에서는 RSC의 개념부터 실전 활용까지 모든 것을 다룹니다.

## Server Components란 무엇인가?

### 기존 React의 한계

전통적인 React 애플리케이션은 모든 컴포넌트가 클라이언트에서 실행됩니다. 이는 다음과 같은 문제를 야기합니다:

- **큰 번들 크기**: 모든 컴포넌트와 라이브러리가 브라우저로 전송됨
- **데이터 페칭 워터폴**: 컴포넌트 계층에 따라 순차적으로 데이터를 가져옴
- **민감 정보 노출**: API 키나 데이터베이스 접근 로직이 클라이언트에 노출될 위험

### Server Components의 등장

React Server Components는 컴포넌트를 서버에서 렌더링하고, 그 결과를 클라이언트로 전송합니다. 이를 통해:

- **제로 번들 크기**: Server Components 코드가 클라이언트 번들에 포함되지 않음
- **직접 데이터 접근**: 데이터베이스, 파일 시스템 등에 직접 접근 가능
- **자동 코드 분할**: 각 Server Component가 자동으로 분할됨
- **향상된 보안**: 민감한 로직을 서버에 안전하게 보관

```tsx
// Server Component 예시
// 이 컴포넌트는 서버에서만 실행되며, 클라이언트 번들에 포함되지 않습니다
async function BlogPost({ id }: { id: string }) {
  // 서버에서 직접 데이터베이스 접근
  const post = await db.posts.findOne({ id });

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

## Server Components vs Client Components

### 개념적 차이점

| 특성 | Server Components | Client Components |
|------|-------------------|-------------------|
| 실행 위치 | 서버에서만 실행 | 클라이언트에서 실행 (서버에서 pre-render 가능) |
| 번들 크기 | 클라이언트 번들에 미포함 | 클라이언트 번들에 포함 |
| 데이터 접근 | 직접 백엔드 리소스 접근 가능 | API를 통해서만 접근 |
| 상태 관리 | 불가능 (useState, useReducer 등) | 가능 |
| 이벤트 핸들러 | 불가능 | 가능 |
| 브라우저 API | 불가능 | 가능 |
| Hooks 사용 | 일부만 가능 (use, custom hooks 불가) | 모두 가능 |
| async/await | 가능 (async 컴포넌트) | 불가능 (컴포넌트 함수 자체는) |

### 렌더링 방식 비교

#### Server Component 렌더링

```tsx
// app/page.tsx (Server Component)
async function ProductList() {
  // 서버에서 실행 - 데이터베이스 직접 접근
  const products = await prisma.product.findMany();

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

렌더링 과정:
1. 서버에서 컴포넌트 실행
2. 데이터 페칭 완료
3. 결과를 특수 형식(RSC Payload)으로 직렬화
4. 클라이언트로 전송
5. 클라이언트에서 UI 재구성

#### Client Component 렌더링

```tsx
'use client';

import { useState, useEffect } from 'react';

// 클라이언트에서 실행되는 컴포넌트
function InteractiveCounter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

렌더링 과정:
1. 서버에서 초기 HTML 생성 (선택적)
2. 컴포넌트 코드를 클라이언트로 전송
3. 클라이언트에서 hydration
4. 이후 상호작용은 클라이언트에서 처리

### 언제 어떤 것을 사용해야 하는가?

#### Server Components를 사용해야 할 때

- 데이터 페칭이 필요한 경우
- 백엔드 리소스에 직접 접근해야 하는 경우
- 민감한 정보를 다루는 경우 (API 키, 토큰 등)
- 큰 의존성 라이브러리를 사용하는 경우 (번들 크기 절감)
- 정적이거나 사용자 상호작용이 없는 UI

```tsx
// 데이터 페칭 - Server Component 사용
async function UserProfile({ userId }: { userId: string }) {
  const user = await fetch(`https://api.example.com/users/${userId}`);
  const data = await user.json();

  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.bio}</p>
    </div>
  );
}
```

#### Client Components를 사용해야 할 때

- 상태(state)를 사용하는 경우
- 이벤트 핸들러가 필요한 경우 (onClick, onChange 등)
- 브라우저 API를 사용하는 경우 (localStorage, window 등)
- React Hooks를 사용하는 경우 (useState, useEffect 등)
- 사용자 상호작용이 필요한 경우

```tsx
'use client';

import { useState } from 'react';

// 상호작용 - Client Component 사용
function SearchBox() {
  const [query, setQuery] = useState('');

  return (
    <input
      type="text"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

## Server Components의 동작 원리

### RSC 프로토콜

React Server Components는 특수한 직렬화 형식(RSC Payload)을 사용하여 서버에서 클라이언트로 컴포넌트 트리를 전송합니다.

#### RSC Payload 구조

```tsx
// 서버 컴포넌트
async function Page() {
  const data = await fetchData();
  return (
    <div>
      <Header />
      <Content data={data} />
      <ClientButton />
    </div>
  );
}
```

위 컴포넌트가 렌더링되면 다음과 같은 RSC Payload가 생성됩니다:

```jsonc
// 실제 전송되는 데이터 (간소화된 버전)
{
  "type": "div",
  "props": {
    "children": [
      {
        "type": "Header",
        "props": {},
        "$$typeof": "react.element"
      },
      {
        "type": "Content",
        "props": { "data": { /* 실제 데이터 */ } },
        "$$typeof": "react.element"
      },
      {
        "type": "ClientButton",
        "props": {},
        "$$typeof": "react.module.reference",
        "module": "client-button.js"  // 클라이언트에서 로드할 모듈
      }
    ]
  }
}
```

### 서버에서 클라이언트로 전달되는 과정

```tsx
// 1. 서버에서 시작
// app/page.tsx
async function HomePage() {
  // 2. 서버에서 데이터 페칭
  const posts = await db.posts.findMany();

  // 3. 서버에서 렌더링
  return (
    <main>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        // 4. 각 포스트를 RSC Payload로 직렬화
        <PostCard key={post.id} post={post} />
      ))}
      {/* 5. Client Component는 참조로 전달 */}
      <CommentSection postId={posts[0].id} />
    </main>
  );
}
```

전달 과정:
1. **Request**: 클라이언트가 페이지 요청
2. **Server Rendering**: 서버에서 Server Components 실행
3. **Serialization**: 결과를 RSC Payload로 직렬화
4. **Streaming**: 준비된 부분부터 점진적으로 전송
5. **Client Reconstruction**: 클라이언트에서 UI 재구성
6. **Hydration**: Client Components만 hydrate

### Streaming과 Suspense

Server Components는 Suspense와 함께 사용하여 점진적 렌더링을 구현할 수 있습니다.

```tsx
import { Suspense } from 'react';

// app/page.tsx
function ProductPage() {
  return (
    <div>
      {/* 즉시 렌더링되는 부분 */}
      <header>
        <h1>Product Details</h1>
      </header>

      {/* 데이터 로딩 중 fallback 표시 */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductInfo />
      </Suspense>

      {/* 독립적으로 로딩 */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews />
      </Suspense>
    </div>
  );
}

// 느린 데이터 페칭
async function ProductInfo() {
  const product = await fetchProduct(); // 2초 소요
  return <div>{product.name}</div>;
}

// 더 느린 데이터 페칭
async function ProductReviews() {
  const reviews = await fetchReviews(); // 5초 소요
  return <div>{reviews.length} reviews</div>;
}
```

렌더링 타임라인:
```
0s:    HTML Shell 전송 (header + skeletons)
       사용자가 즉시 컨텐츠 볼 수 있음

2s:    ProductInfo 완료 → streaming으로 전송
       ProductSkeleton이 실제 내용으로 교체

5s:    ProductReviews 완료 → streaming으로 전송
       ReviewsSkeleton이 실제 내용으로 교체
```

#### Streaming의 장점

```tsx
// 전통적인 방식: 모든 데이터를 기다림
async function TraditionalPage() {
  const [product, reviews, recommendations] = await Promise.all([
    fetchProduct(),      // 2초
    fetchReviews(),      // 5초
    fetchRecommendations() // 3초
  ]);

  // 5초 후에야 사용자가 무언가를 볼 수 있음
  return <div>...</div>;
}

// Streaming 방식: 준비된 것부터 표시
function StreamingPage() {
  return (
    <>
      {/* 0초: 즉시 표시 */}
      <Header />

      {/* 2초: ProductInfo 표시 */}
      <Suspense fallback={<Skeleton />}>
        <ProductInfo />
      </Suspense>

      {/* 3초: Recommendations 표시 */}
      <Suspense fallback={<Skeleton />}>
        <Recommendations />
      </Suspense>

      {/* 5초: Reviews 표시 */}
      <Suspense fallback={<Skeleton />}>
        <Reviews />
      </Suspense>
    </>
  );
}
```

## Server Components 작성법

### 기본 문법과 규칙

Server Components는 기본적으로 일반 React 컴포넌트와 동일하게 작성하되, 몇 가지 규칙을 따라야 합니다.

```tsx
// app/components/UserList.tsx
// 별도의 지시어 없이 기본적으로 Server Component

// ✅ 올바른 Server Component
async function UserList() {
  const users = await fetchUsers();

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ❌ 잘못된 예시 - Server Component에서 사용 불가
function InvalidServerComponent() {
  // 에러: useState는 Server Component에서 사용 불가
  const [count, setCount] = useState(0);

  // 에러: onClick 이벤트 핸들러 사용 불가
  return <button onClick={() => setCount(count + 1)}>Click</button>;
}
```

### 'use server' 지시어

`'use server'`는 Server Actions를 정의할 때 사용합니다 (Server Components와는 다른 개념).

```tsx
// app/actions.ts
'use server';

// Server Action: 클라이언트에서 호출 가능한 서버 함수
export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // 서버에서만 실행되는 코드
  await db.posts.create({
    data: { title, content }
  });

  revalidatePath('/posts');
}

// 함수 내부에서도 사용 가능
export async function updatePost(id: string, data: any) {
  'use server';

  await db.posts.update({
    where: { id },
    data
  });
}
```

Client Component에서 Server Action 사용:

```tsx
'use client';

import { createPost } from './actions';

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### async 컴포넌트

Server Components는 비동기 함수로 정의할 수 있습니다. 이는 Client Components에서는 불가능합니다.

```tsx
// ✅ Server Component - async 가능
async function BlogPost({ id }: { id: string }) {
  const post = await db.posts.findUnique({
    where: { id }
  });

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}

// ✅ 여러 데이터 소스에서 페칭
async function Dashboard() {
  // 병렬로 데이터 페칭
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

// ❌ Client Component - async 불가능
'use client';

async function InvalidClientComponent() {
  // 에러: Client Component는 async 불가
  const data = await fetchData();
  return <div>{data}</div>;
}
```

### 데이터 페칭 패턴

#### 패턴 1: 컴포넌트 레벨 페칭

```tsx
// 각 컴포넌트가 자신의 데이터를 페칭
async function PostList() {
  const posts = await fetchPosts();

  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} postId={post.id} />
      ))}
    </div>
  );
}

async function PostCard({ postId }: { postId: string }) {
  // 각 카드가 독립적으로 데이터 페칭
  const post = await fetchPost(postId);
  const author = await fetchAuthor(post.authorId);

  return (
    <div>
      <h3>{post.title}</h3>
      <p>By {author.name}</p>
    </div>
  );
}
```

#### 패턴 2: 데이터 전달 패턴

```tsx
// 부모가 데이터를 페칭하여 자식에게 전달
async function PostList() {
  const posts = await fetchPosts();

  // 작성자 정보도 한 번에 가져오기
  const authorIds = [...new Set(posts.map(p => p.authorId))];
  const authors = await fetchAuthors(authorIds);
  const authorsMap = new Map(authors.map(a => [a.id, a]));

  return (
    <div>
      {posts.map(post => (
        <PostCard
          key={post.id}
          post={post}
          author={authorsMap.get(post.authorId)!}
        />
      ))}
    </div>
  );
}

// 이 컴포넌트는 데이터를 받기만 함
function PostCard({
  post,
  author
}: {
  post: Post;
  author: Author;
}) {
  return (
    <div>
      <h3>{post.title}</h3>
      <p>By {author.name}</p>
    </div>
  );
}
```

#### 패턴 3: 병렬 페칭과 Suspense

```tsx
// 독립적인 데이터는 병렬로 로딩
function ProfilePage({ userId }: { userId: string }) {
  return (
    <div>
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile userId={userId} />
      </Suspense>

      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={userId} />
      </Suspense>

      <Suspense fallback={<FollowersSkeleton />}>
        <UserFollowers userId={userId} />
      </Suspense>
    </div>
  );
}

async function UserProfile({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  return <div>...</div>;
}

async function UserPosts({ userId }: { userId: string }) {
  const posts = await fetchUserPosts(userId);
  return <div>...</div>;
}

async function UserFollowers({ userId }: { userId: string }) {
  const followers = await fetchFollowers(userId);
  return <div>...</div>;
}
```

#### 패턴 4: 캐싱을 활용한 중복 제거

```tsx
// React는 동일한 요청을 자동으로 중복 제거합니다
async function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
      <Content />
    </div>
  );
}

async function Header() {
  // 첫 번째 호출
  const user = await fetchUser();
  return <div>{user.name}</div>;
}

async function Sidebar() {
  // 캐시된 결과 사용 (중복 요청 안 함)
  const user = await fetchUser();
  return <div>{user.role}</div>;
}

async function Content() {
  // 역시 캐시된 결과 사용
  const user = await fetchUser();
  return <div>{user.bio}</div>;
}

// fetchUser 구현 (Next.js의 경우)
async function fetchUser() {
  const res = await fetch('https://api.example.com/user', {
    next: { revalidate: 3600 } // 1시간 캐싱
  });
  return res.json();
}
```

## Client Components와의 조합

### 'use client' 지시어

Client Component를 정의하려면 파일 최상단에 `'use client'` 지시어를 추가합니다.

```tsx
// app/components/Counter.tsx
'use client';

import { useState } from 'react';

// 이 파일의 모든 컴포넌트는 Client Component
export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

export function ResetButton({ onReset }: { onReset: () => void }) {
  return <button onClick={onReset}>Reset</button>;
}
```

### 경계(Boundary) 설정

Server와 Client Components의 경계는 신중하게 설정해야 합니다.

#### 원칙 1: Client Component는 잎(leaf)에 가깝게

```tsx
// ❌ 좋지 않은 예: 최상위에 'use client'
'use client';

export default function Page() {
  const [filter, setFilter] = useState('all');

  return (
    <div>
      <Header />  {/* 불필요하게 Client Component가 됨 */}
      <FilterBar filter={filter} onFilterChange={setFilter} />
      <ProductList filter={filter} />  {/* 불필요하게 Client Component가 됨 */}
    </div>
  );
}

// ✅ 좋은 예: 필요한 부분만 Client Component
export default function Page() {
  return (
    <div>
      <Header />  {/* Server Component 유지 */}
      <FilterableProductList />
    </div>
  );
}

function FilterableProductList() {
  return (
    <>
      <ClientFilterBar />  {/* 상태 관리 필요한 부분만 Client */}
      <ProductList />  {/* Server Component로 데이터 페칭 */}
    </>
  );
}
```

#### 원칙 2: Server Component를 Client Component의 children으로 전달

```tsx
// ❌ 불가능: Client Component에서 Server Component import
'use client';

import ServerComponent from './ServerComponent'; // 에러!

function ClientComponent() {
  return <ServerComponent />; // Server Component가 Client로 변환됨
}

// ✅ 가능: children prop으로 전달
// app/page.tsx (Server Component)
export default function Page() {
  return (
    <ClientWrapper>
      {/* Server Component를 children으로 전달 */}
      <ServerComponent />
    </ClientWrapper>
  );
}

// app/components/ClientWrapper.tsx
'use client';

function ClientWrapper({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children}  {/* Server Component 유지 */}
    </div>
  );
}
```

실제 활용 예시:

```tsx
// Server Component: 데이터 페칭
async function ProductPage({ id }: { id: string }) {
  const product = await fetchProduct(id);
  const recommendations = await fetchRecommendations(id);

  return (
    <div>
      {/* Server Component */}
      <ProductInfo product={product} />

      {/* Client Component with Server Component children */}
      <InteractiveModal>
        <ProductDetails product={product} />
      </InteractiveModal>

      {/* Server Component */}
      <RecommendationList items={recommendations} />
    </div>
  );
}

// Client Component
'use client';

function InteractiveModal({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>View Details</button>
      {isOpen && (
        <dialog open>
          {children}  {/* Server Component가 여기 렌더링 */}
          <button onClick={() => setIsOpen(false)}>Close</button>
        </dialog>
      )}
    </>
  );
}
```

### props 전달 규칙

#### 직렬화 가능한 데이터만 전달 가능

```tsx
// Server Component
async function ParentServer() {
  const data = await fetchData();

  // ✅ 직렬화 가능한 데이터
  const serializable = {
    string: 'hello',
    number: 42,
    boolean: true,
    array: [1, 2, 3],
    object: { key: 'value' },
    date: new Date().toISOString(), // Date는 문자열로 변환
    null: null,
    undefined: undefined
  };

  // ❌ 직렬화 불가능한 데이터
  const nonSerializable = {
    function: () => {},  // 함수 불가
    classInstance: new MyClass(),  // 클래스 인스턴스 불가
    symbol: Symbol('test'),  // Symbol 불가
    date: new Date()  // Date 객체 그대로는 불가 (toISOString() 필요)
  };

  return (
    <ClientChild
      data={serializable}  // ✅ OK
      // invalid={nonSerializable}  // ❌ 에러!
    />
  );
}
```

#### 함수 전달하기: Server Actions 활용

```tsx
// Server Component
import { deletePost } from './actions';

async function PostList() {
  const posts = await fetchPosts();

  return (
    <div>
      {posts.map(post => (
        <PostCard
          key={post.id}
          post={post}
          // Server Action을 prop으로 전달
          onDelete={deletePost}
        />
      ))}
    </div>
  );
}

// actions.ts
'use server';

export async function deletePost(formData: FormData) {
  const id = formData.get('id') as string;
  await db.posts.delete({ where: { id } });
  revalidatePath('/posts');
}

// Client Component
'use client';

function PostCard({
  post,
  onDelete
}: {
  post: Post;
  onDelete: (formData: FormData) => Promise<void>;
}) {
  return (
    <div>
      <h3>{post.title}</h3>
      <form action={onDelete}>
        <input type="hidden" name="id" value={post.id} />
        <button type="submit">Delete</button>
      </form>
    </div>
  );
}
```

### 직렬화 가능한 데이터

Server에서 Client로 전달 가능한 데이터 타입:

```tsx
// types.ts
interface SerializableData {
  // ✅ 기본 타입
  stringValue: string;
  numberValue: number;
  booleanValue: boolean;
  nullValue: null;
  undefinedValue: undefined;

  // ✅ 배열
  arrayOfStrings: string[];
  arrayOfObjects: Array<{ id: number; name: string }>;

  // ✅ 중첩 객체
  nestedObject: {
    level1: {
      level2: {
        value: string;
      };
    };
  };

  // ✅ Date는 문자열로 변환
  dateString: string; // new Date().toISOString()

  // ❌ 불가능한 타입들 (에러 발생)
  // functionValue: () => void;
  // classInstance: MyClass;
  // symbolValue: symbol;
  // mapValue: Map<string, any>;
  // setValue: Set<string>;
}
```

실전 예시:

```tsx
// Server Component
async function UserDashboard({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  const posts = await fetchUserPosts(userId);

  // 데이터를 직렬화 가능한 형태로 변환
  const dashboardData = {
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
      createdAt: user.createdAt.toISOString(), // Date → string
    },
    posts: posts.map(post => ({
      id: post.id,
      title: post.title,
      publishedAt: post.publishedAt.toISOString(),
      // 함수나 메서드는 제외
    })),
    stats: {
      totalPosts: posts.length,
      totalViews: posts.reduce((sum, p) => sum + p.views, 0),
    }
  };

  return <ClientDashboard data={dashboardData} />;
}

// Client Component
'use client';

function ClientDashboard({ data }: { data: typeof dashboardData }) {
  const [selectedPost, setSelectedPost] = useState(data.posts[0]);

  return (
    <div>
      <h1>{data.user.name}'s Dashboard</h1>
      <p>Member since: {new Date(data.user.createdAt).toLocaleDateString()}</p>
      {/* ... */}
    </div>
  );
}
```

## 실전 패턴과 Best Practices

### 컴포넌트 트리 구성

#### 패턴 1: 레이어드 아키텍처

```tsx
// app/posts/[id]/page.tsx (Server Component - 최상위)
export default async function PostPage({ params }: { params: { id: string } }) {
  // 데이터 페칭 레이어
  const post = await fetchPost(params.id);
  const author = await fetchAuthor(post.authorId);

  return (
    <article>
      {/* 프레젠테이션 레이어 (Server Components) */}
      <PostHeader post={post} author={author} />
      <PostContent content={post.content} />

      {/* 인터랙션 레이어 (Client Components) */}
      <PostInteractions postId={post.id} />

      {/* 추가 데이터 레이어 (비동기) */}
      <Suspense fallback={<CommentsSkeleton />}>
        <CommentsSection postId={post.id} />
      </Suspense>
    </article>
  );
}

// 프레젠테이션 레이어 - Server Components
function PostHeader({ post, author }: { post: Post; author: Author }) {
  return (
    <header>
      <h1>{post.title}</h1>
      <AuthorInfo author={author} />
      <PublishDate date={post.publishedAt} />
    </header>
  );
}

// 인터랙션 레이어 - Client Component
'use client';

function PostInteractions({ postId }: { postId: string }) {
  const [liked, setLiked] = useState(false);
  const [bookmarked, setBookmarked] = useState(false);

  return (
    <div className="interactions">
      <LikeButton liked={liked} onLike={() => setLiked(!liked)} />
      <BookmarkButton bookmarked={bookmarked} onBookmark={() => setBookmarked(!bookmarked)} />
      <ShareButton postId={postId} />
    </div>
  );
}
```

#### 패턴 2: 컴포지션 패턴

```tsx
// 재사용 가능한 컨테이너 컴포넌트
// app/components/Card.tsx
'use client';

interface CardProps {
  header?: React.ReactNode;
  children: React.ReactNode;
  footer?: React.ReactNode;
  collapsible?: boolean;
}

export function Card({ header, children, footer, collapsible = false }: CardProps) {
  const [isCollapsed, setIsCollapsed] = useState(false);

  return (
    <div className="card">
      {header && (
        <div className="card-header">
          {header}
          {collapsible && (
            <button onClick={() => setIsCollapsed(!isCollapsed)}>
              {isCollapsed ? 'Expand' : 'Collapse'}
            </button>
          )}
        </div>
      )}
      {!isCollapsed && <div className="card-body">{children}</div>}
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// Server Component에서 Card 사용
async function Dashboard() {
  const stats = await fetchStats();
  const activities = await fetchRecentActivities();

  return (
    <div className="dashboard">
      {/* Server Component를 Client Component의 슬롯에 넣기 */}
      <Card
        header={<h2>Statistics</h2>}
        footer={<LastUpdated date={stats.updatedAt} />}
        collapsible
      >
        <StatsDisplay data={stats} />
      </Card>

      <Card header={<h2>Recent Activities</h2>}>
        <ActivityList items={activities} />
      </Card>
    </div>
  );
}

// Server Components로 실제 데이터 렌더링
function StatsDisplay({ data }: { data: Stats }) {
  return (
    <div className="stats-grid">
      <StatItem label="Users" value={data.userCount} />
      <StatItem label="Revenue" value={`$${data.revenue}`} />
      <StatItem label="Growth" value={`${data.growth}%`} />
    </div>
  );
}
```

### 데이터 페칭 최적화

#### 워터폴 방지

```tsx
// ❌ 나쁜 예: 순차적 워터폴
async function BadProductPage({ id }: { id: string }) {
  // 1초 대기
  const product = await fetchProduct(id);

  // product를 받은 후 1초 대기
  const category = await fetchCategory(product.categoryId);

  // category를 받은 후 1초 대기
  const relatedProducts = await fetchRelatedProducts(category.id);

  // 총 3초 소요
  return <div>...</div>;
}

// ✅ 좋은 예 1: 병렬 페칭
async function GoodProductPage({ id }: { id: string }) {
  // 모두 동시에 시작
  const [product, categoryPromise, relatedPromise] = await Promise.all([
    fetchProduct(id),
    fetchProduct(id).then(p => fetchCategory(p.categoryId)),
    fetchProduct(id)
      .then(p => fetchCategory(p.categoryId))
      .then(c => fetchRelatedProducts(c.id))
  ]);

  // 하지만 여전히 중첩된 의존성 때문에 비효율적
}

// ✅ 좋은 예 2: 데이터 구조 최적화
async function OptimizedProductPage({ id }: { id: string }) {
  // 백엔드 API를 수정하여 필요한 모든 데이터를 한 번에 가져오기
  const productData = await fetchProductWithDetails(id);
  // {
  //   product: { ... },
  //   category: { ... },
  //   relatedProducts: [ ... ]
  // }

  return (
    <div>
      <ProductInfo product={productData.product} />
      <CategoryBadge category={productData.category} />
      <RelatedProducts items={productData.relatedProducts} />
    </div>
  );
}

// ✅ 좋은 예 3: Suspense로 독립적 로딩
function SuspenseProductPage({ id }: { id: string }) {
  return (
    <div>
      {/* 제품 정보는 빠르게 로드 */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductInfo id={id} />
      </Suspense>

      {/* 관련 제품은 독립적으로 로드 */}
      <Suspense fallback={<RelatedSkeleton />}>
        <RelatedProducts id={id} />
      </Suspense>
    </div>
  );
}
```

#### 데이터 프리페칭

```tsx
// app/posts/page.tsx
import { preloadPostList } from './preload';

export default function PostsPage() {
  // 렌더링 전에 데이터 프리페칭 시작
  preloadPostList();

  return (
    <Suspense fallback={<PostsSkeleton />}>
      <PostList />
    </Suspense>
  );
}

// preload.ts
import { cache } from 'react';

// cache()를 사용하여 중복 요청 방지
export const getPostList = cache(async () => {
  const res = await fetch('https://api.example.com/posts');
  return res.json();
});

// 프리페칭 함수
export function preloadPostList() {
  void getPostList(); // 결과를 기다리지 않고 요청만 시작
}

// PostList.tsx
import { getPostList } from './preload';

async function PostList() {
  // 이미 프리페칭된 데이터를 사용 (캐시에서 가져옴)
  const posts = await getPostList();

  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

### 캐싱 전략

#### Next.js fetch 캐싱

```tsx
// 1. 자동 캐싱 (기본값)
async function DefaultCache() {
  // 무기한 캐싱됨
  const data = await fetch('https://api.example.com/data');
  return data.json();
}

// 2. 시간 기반 재검증
async function TimeBasedCache() {
  // 60초마다 재검증
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }
  });
  return data.json();
}

// 3. 캐싱 비활성화
async function NoCache() {
  // 항상 최신 데이터 페칭
  const data = await fetch('https://api.example.com/data', {
    cache: 'no-store'
  });
  return data.json();
}

// 4. 태그 기반 재검증
async function TagBasedCache() {
  const data = await fetch('https://api.example.com/data', {
    next: {
      revalidate: 3600,  // 1시간
      tags: ['products']  // 태그 지정
    }
  });
  return data.json();
}

// Server Action에서 태그 기반 재검증
'use server';

import { revalidateTag } from 'next/cache';

export async function updateProduct() {
  await db.products.update(/* ... */);

  // 'products' 태그가 있는 모든 캐시 무효화
  revalidateTag('products');
}
```

#### React cache() 활용

```tsx
import { cache } from 'react';

// 동일한 렌더링 중 중복 호출 방지
export const getUser = cache(async (id: string) => {
  console.log(`Fetching user ${id}`); // 한 번만 출력됨

  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
});

// 여러 컴포넌트에서 호출해도 한 번만 실행됨
async function Header() {
  const user = await getUser('123');
  return <div>{user.name}</div>;
}

async function Sidebar() {
  const user = await getUser('123'); // 캐시에서 가져옴
  return <div>{user.email}</div>;
}

async function Main() {
  const user = await getUser('123'); // 캐시에서 가져옴
  return <div>{user.bio}</div>;
}
```

### 에러 처리

#### Error Boundary 활용

```tsx
// app/posts/error.tsx
'use client';

export default function PostsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}

// app/posts/page.tsx
async function PostsPage() {
  // 에러가 발생하면 error.tsx가 렌더링됨
  const posts = await fetchPosts(); // 에러 발생 가능

  return (
    <div>
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
    </div>
  );
}
```

#### 세밀한 에러 처리

```tsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function ProductPage({ id }: { id: string }) {
  return (
    <div>
      {/* 각 섹션에 독립적인 에러 처리 */}
      <ErrorBoundary fallback={<ProductInfoError />}>
        <Suspense fallback={<ProductInfoSkeleton />}>
          <ProductInfo id={id} />
        </Suspense>
      </ErrorBoundary>

      <ErrorBoundary fallback={<ReviewsError />}>
        <Suspense fallback={<ReviewsSkeleton />}>
          <ProductReviews id={id} />
        </Suspense>
      </ErrorBoundary>

      {/* 관련 상품 로딩 실패해도 페이지는 정상 작동 */}
      <ErrorBoundary fallback={null}>
        <Suspense fallback={<RelatedSkeleton />}>
          <RelatedProducts id={id} />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}

function ProductInfoError() {
  return (
    <div className="error">
      <p>Failed to load product information.</p>
      <button onClick={() => window.location.reload()}>Retry</button>
    </div>
  );
}
```

#### 에러 로깅

```tsx
// lib/error-logging.ts
export async function logError(error: Error, context: Record<string, any>) {
  // 서버에서만 실행되는 로깅
  if (typeof window === 'undefined') {
    console.error('Server Error:', {
      message: error.message,
      stack: error.stack,
      context,
      timestamp: new Date().toISOString()
    });

    // 외부 서비스로 전송 (예: Sentry, DataDog)
    // await sendToErrorTracking(error, context);
  }
}

// Server Component에서 에러 처리
async function DataComponent() {
  try {
    const data = await fetchData();
    return <div>{data}</div>;
  } catch (error) {
    // 서버 사이드 로깅
    await logError(error as Error, {
      component: 'DataComponent',
      action: 'fetchData'
    });

    // 에러를 다시 던져서 Error Boundary가 처리하게 함
    throw error;
  }
}
```

## Next.js에서의 Server Components

### App Router와 RSC

Next.js 13+의 App Router는 Server Components를 기본으로 사용합니다.

```tsx
// app/layout.tsx - Server Component (기본)
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}

// app/page.tsx - Server Component (기본)
export default async function HomePage() {
  const posts = await fetchPosts();

  return (
    <div>
      <h1>Latest Posts</h1>
      <PostList posts={posts} />
    </div>
  );
}

// app/posts/[id]/page.tsx - 동적 라우트 Server Component
export default async function PostPage({
  params,
}: {
  params: { id: string };
}) {
  const post = await fetchPost(params.id);

  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

#### 메타데이터 생성

```tsx
// app/posts/[id]/page.tsx
import { Metadata } from 'next';

// 정적 메타데이터
export const metadata: Metadata = {
  title: 'Blog Posts',
  description: 'Read our latest blog posts',
};

// 동적 메타데이터
export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata> {
  const post = await fetchPost(params.id);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}

export default async function PostPage({
  params,
}: {
  params: { id: string };
}) {
  const post = await fetchPost(params.id);
  return <article>...</article>;
}
```

### 서버 액션(Server Actions)

Server Actions를 사용하면 Client Components에서 서버 함수를 직접 호출할 수 있습니다.

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

// Form 데이터 처리
export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  // 서버에서만 실행되는 데이터베이스 작업
  const post = await db.posts.create({
    data: { title, content }
  });

  // 캐시 재검증
  revalidatePath('/posts');

  // 리다이렉트
  redirect(`/posts/${post.id}`);
}

// 일반 함수 호출 방식
export async function updatePost(id: string, data: { title: string; content: string }) {
  'use server';

  await db.posts.update({
    where: { id },
    data
  });

  revalidatePath(`/posts/${id}`);
  return { success: true };
}

// 낙관적 업데이트를 위한 액션
export async function toggleLike(postId: string, currentLiked: boolean) {
  'use server';

  await db.likes.upsert({
    where: { postId },
    update: { liked: !currentLiked },
    create: { postId, liked: true }
  });

  revalidatePath('/posts');
  return { liked: !currentLiked };
}
```

#### Form과 함께 사용하기

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input
        type="text"
        name="title"
        placeholder="Title"
        required
      />
      <textarea
        name="content"
        placeholder="Content"
        required
      />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

#### Client Component에서 사용하기

```tsx
'use client';

import { updatePost, toggleLike } from '@/app/actions';
import { useOptimistic, useTransition } from 'react';

function PostEditor({ post }: { post: Post }) {
  const [isPending, startTransition] = useTransition();

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;

    startTransition(async () => {
      await updatePost(post.id, { title, content });
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" defaultValue={post.title} />
      <textarea name="content" defaultValue={post.content} />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
}

// 낙관적 업데이트 예시
function LikeButton({ postId, initialLiked }: { postId: string; initialLiked: boolean }) {
  const [optimisticLiked, setOptimisticLiked] = useOptimistic(initialLiked);

  const handleLike = async () => {
    // UI를 즉시 업데이트 (낙관적)
    setOptimisticLiked(!optimisticLiked);

    // 서버 액션 실행
    await toggleLike(postId, optimisticLiked);
  };

  return (
    <button onClick={handleLike}>
      {optimisticLiked ? '❤️ Liked' : '🤍 Like'}
    </button>
  );
}
```

### 라우트 핸들러

API Routes를 Server Components와 함께 사용할 수 있습니다.

```tsx
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/posts
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = searchParams.get('page') || '1';

  const posts = await db.posts.findMany({
    skip: (parseInt(page) - 1) * 10,
    take: 10,
  });

  return NextResponse.json(posts);
}

// POST /api/posts
export async function POST(request: NextRequest) {
  const body = await request.json();

  const post = await db.posts.create({
    data: body,
  });

  return NextResponse.json(post, { status: 201 });
}

// app/api/posts/[id]/route.ts
// GET /api/posts/:id
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = await db.posts.findUnique({
    where: { id: params.id },
  });

  if (!post) {
    return NextResponse.json(
      { error: 'Post not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(post);
}

// PATCH /api/posts/:id
export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();

  const post = await db.posts.update({
    where: { id: params.id },
    data: body,
  });

  return NextResponse.json(post);
}

// DELETE /api/posts/:id
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await db.posts.delete({
    where: { id: params.id },
  });

  return NextResponse.json({ success: true });
}
```

#### 라우트 핸들러 vs Server Actions

```tsx
// Server Actions 사용 (권장)
// - Form 제출에 최적화
// - 자동 재검증
// - 간단한 구문

'use server';

export async function createComment(postId: string, content: string) {
  const comment = await db.comments.create({
    data: { postId, content }
  });

  revalidatePath(`/posts/${postId}`);
  return comment;
}

// Client에서 사용
'use client';

function CommentForm({ postId }: { postId: string }) {
  return (
    <form action={async (formData) => {
      await createComment(postId, formData.get('content') as string);
    }}>
      <textarea name="content" />
      <button type="submit">Submit</button>
    </form>
  );
}

// Route Handlers 사용
// - RESTful API 필요시
// - 외부 클라이언트 지원
// - 복잡한 HTTP 처리

// app/api/comments/route.ts
export async function POST(request: NextRequest) {
  const { postId, content } = await request.json();

  const comment = await db.comments.create({
    data: { postId, content }
  });

  return NextResponse.json(comment);
}

// Client에서 사용
'use client';

async function submitComment(postId: string, content: string) {
  const res = await fetch('/api/comments', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ postId, content })
  });

  return res.json();
}
```

## 성능 최적화

### 번들 크기 최소화

#### 전략 1: Server Components로 마이그레이션

```tsx
// Before: 모든 것이 Client Component
'use client';

import Chart from 'heavy-chart-library'; // 500KB
import { useState } from 'react';

export default function Dashboard() {
  const [data, setData] = useState([]);

  useEffect(() => {
    fetch('/api/data').then(r => r.json()).then(setData);
  }, []);

  return <Chart data={data} />; // 500KB가 클라이언트로 전송됨
}

// After: Server Component 활용
// app/dashboard/page.tsx (Server Component)
import Chart from 'heavy-chart-library'; // 서버에서만 실행, 0KB 번들

export default async function Dashboard() {
  const data = await fetchData(); // 서버에서 페칭

  return <Chart data={data} />; // Chart 코드가 번들에 포함되지 않음
}
```

실제 번들 크기 비교:
```
Before (Client Component):
- React: 50KB
- Chart Library: 500KB
- Component Code: 10KB
Total: 560KB

After (Server Component):
- React: 50KB
- Chart Library: 0KB (서버에서만 실행)
- Component Code: 0KB (서버에서만 실행)
- RSC Payload: 5KB (렌더링된 결과만)
Total: 55KB (90% 감소!)
```

#### 전략 2: 동적 임포트로 Client Components 분할

```tsx
'use client';

import dynamic from 'next/dynamic';

// 무거운 컴포넌트를 동적으로 로드
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false // 클라이언트에서만 로드
});

const VideoPlayer = dynamic(() => import('./VideoPlayer'), {
  loading: () => <div>Loading player...</div>,
});

export default function MediaPage() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>

      {/* 클릭할 때만 로드 */}
      {showChart && <HeavyChart data={data} />}

      {/* 뷰포트에 들어올 때 로드 */}
      <LazyLoad>
        <VideoPlayer src="/video.mp4" />
      </LazyLoad>
    </div>
  );
}
```

### 워터폴 방지

#### 문제: 순차적 데이터 페칭

```tsx
// ❌ 나쁜 예: 3초 워터폴
async function ProductPage({ id }: { id: string }) {
  const product = await fetchProduct(id);        // 1초
  const reviews = await fetchReviews(id);        // 1초
  const recommendations = await fetchRecommendations(id); // 1초

  // 총 3초 소요
  return <div>...</div>;
}
```

#### 해결책 1: Promise.all로 병렬 페칭

```tsx
// ✅ 좋은 예: 1초로 단축
async function ProductPage({ id }: { id: string }) {
  const [product, reviews, recommendations] = await Promise.all([
    fetchProduct(id),
    fetchReviews(id),
    fetchRecommendations(id)
  ]);

  // 가장 느린 요청(1초)만큼만 소요
  return <div>...</div>;
}
```

#### 해결책 2: Suspense로 분리

```tsx
// ✅ 더 좋은 예: 점진적 렌더링
function ProductPage({ id }: { id: string }) {
  return (
    <div>
      {/* 빠른 데이터는 먼저 표시 */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductInfo id={id} /> {/* 0.5초 */}
      </Suspense>

      {/* 느린 데이터는 나중에 표시 */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews id={id} /> {/* 2초 */}
      </Suspense>

      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations id={id} /> {/* 1초 */}
      </Suspense>
    </div>
  );
}

// 0.5초 후 ProductInfo 표시
// 1초 후 Recommendations 표시
// 2초 후 ProductReviews 표시
// 사용자는 즉시 일부 컨텐츠를 볼 수 있음!
```

### 병렬 데이터 페칭

#### 패턴 1: 최상위에서 병렬 페칭

```tsx
async function DashboardPage() {
  // 모든 데이터를 병렬로 페칭
  const [user, stats, activities, notifications] = await Promise.all([
    fetchUser(),
    fetchStats(),
    fetchActivities(),
    fetchNotifications()
  ]);

  return (
    <div>
      <UserHeader user={user} />
      <StatsGrid stats={stats} />
      <ActivityFeed activities={activities} />
      <NotificationsList notifications={notifications} />
    </div>
  );
}
```

#### 패턴 2: Suspense와 조합

```tsx
function DashboardPage() {
  return (
    <div>
      {/* 각 섹션이 독립적으로 로딩 */}
      <Suspense fallback={<HeaderSkeleton />}>
        <UserHeader />
      </Suspense>

      <div className="grid">
        <Suspense fallback={<StatsSkeleton />}>
          <StatsGrid />
        </Suspense>

        <Suspense fallback={<ActivitySkeleton />}>
          <ActivityFeed />
        </Suspense>
      </div>

      <Suspense fallback={<NotificationsSkeleton />}>
        <NotificationsList />
      </Suspense>
    </div>
  );
}

// 각 컴포넌트가 독립적으로 데이터 페칭
async function UserHeader() {
  const user = await fetchUser();
  return <header>...</header>;
}

async function StatsGrid() {
  const stats = await fetchStats();
  return <div>...</div>;
}

// 모든 요청이 병렬로 실행됨!
```

#### 패턴 3: preload로 프리페칭

```tsx
import { preload } from 'react-dom';

// 링크 호버 시 프리페칭
'use client';

function ProductLink({ id }: { id: string }) {
  return (
    <Link
      href={`/products/${id}`}
      onMouseEnter={() => {
        // 페이지 방문 전에 데이터 프리페칭
        preloadProductData(id);
      }}
    >
      View Product
    </Link>
  );
}

function preloadProductData(id: string) {
  void fetchProduct(id);
  void fetchReviews(id);
  void fetchRecommendations(id);
}
```

#### 성능 측정 결과

```tsx
// 측정 코드
async function MeasuredPage({ id }: { id: string }) {
  const start = Date.now();

  // 순차적 페칭
  // const product = await fetchProduct(id);
  // const reviews = await fetchReviews(id);
  // const recommendations = await fetchRecommendations(id);

  // 병렬 페칭
  const [product, reviews, recommendations] = await Promise.all([
    fetchProduct(id),
    fetchReviews(id),
    fetchRecommendations(id)
  ]);

  const duration = Date.now() - start;
  console.log(`Page loaded in ${duration}ms`);

  return <div>...</div>;
}
```

결과:
```
순차적 페칭:
- fetchProduct: 500ms
- fetchReviews: 800ms
- fetchRecommendations: 600ms
Total: 1900ms

병렬 페칭:
- All fetches start simultaneously
- Longest fetch (reviews): 800ms
Total: 800ms

성능 향상: 2.4배 빠름 (1900ms → 800ms)
```

## 주의사항과 제한점

### Server Components의 제한사항

```tsx
// ❌ Server Components에서 불가능한 것들

// 1. React Hooks 사용 불가
async function ServerComponent() {
  const [state, setState] = useState(0); // ❌ 에러!
  useEffect(() => {}, []); // ❌ 에러!
  const ref = useRef(null); // ❌ 에러!

  return <div>...</div>;
}

// 2. 이벤트 핸들러 사용 불가
function ServerComponent() {
  return (
    <button onClick={() => alert('Click!')}>  {/* ❌ 에러! */}
      Click me
    </button>
  );
}

// 3. 브라우저 API 사용 불가
function ServerComponent() {
  const value = localStorage.getItem('key'); // ❌ 에러!
  const width = window.innerWidth; // ❌ 에러!

  return <div>...</div>;
}

// 4. Context Provider 사용 불가
function ServerComponent() {
  return (
    <MyContext.Provider value={data}>  {/* ❌ 에러! */}
      <Children />
    </MyContext.Provider>
  );
}
```

### Client Components의 제한사항

```tsx
// ❌ Client Components에서 불가능한 것들

// 1. Server Component를 import 불가
'use client';

import ServerComponent from './ServerComponent'; // ❌ 에러!

function ClientComponent() {
  return <ServerComponent />; // Server Component가 Client로 변환됨
}

// ✅ 대신 children이나 props로 전달
'use client';

function ClientComponent({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>; // ✅ OK
}

// 사용
function ParentServer() {
  return (
    <ClientComponent>
      <ServerComponent /> {/* ✅ OK */}
    </ClientComponent>
  );
}

// 2. 비동기 컴포넌트 불가
'use client';

async function ClientComponent() { // ❌ 에러!
  const data = await fetchData();
  return <div>{data}</div>;
}

// ✅ 대신 useEffect 사용
'use client';

function ClientComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, []);

  return <div>{data}</div>;
}

// 3. 서버 전용 모듈 import 불가
'use client';

import fs from 'fs'; // ❌ 에러!
import { db } from './database'; // ❌ 에러!

function ClientComponent() {
  const file = fs.readFileSync('data.txt'); // ❌ 브라우저에서 실행 불가
  return <div>...</div>;
}
```

### Props 직렬화 제한

```tsx
// ❌ 직렬화 불가능한 props

function ParentServer() {
  const handleClick = () => console.log('click'); // 함수
  const date = new Date(); // Date 객체
  const regex = /test/g; // RegExp
  const map = new Map([['key', 'value']]); // Map

  return (
    <ClientChild
      onClick={handleClick}  // ❌ 에러!
      date={date}           // ❌ 에러!
      pattern={regex}       // ❌ 에러!
      data={map}            // ❌ 에러!
    />
  );
}

// ✅ 올바른 방법

function ParentServer() {
  // 1. Server Action 사용
  async function handleClick() {
    'use server';
    console.log('click');
  }

  // 2. Date는 문자열로 변환
  const dateString = new Date().toISOString();

  // 3. RegExp는 문자열로 전달
  const patternString = '/test/g';

  // 4. Map은 객체나 배열로 변환
  const dataObject = { key: 'value' };

  return (
    <ClientChild
      onClick={handleClick}      // ✅ OK (Server Action)
      date={dateString}          // ✅ OK (문자열)
      pattern={patternString}    // ✅ OK (문자열)
      data={dataObject}          // ✅ OK (평범한 객체)
    />
  );
}
```

### 흔한 실수와 해결책

#### 실수 1: 불필요하게 모든 것을 Client Component로 만들기

```tsx
// ❌ 나쁜 예
'use client';

export default function Page() {
  return (
    <div>
      <Header />  {/* 정적 컨텐츠인데 Client Component가 됨 */}
      <StaticContent />  {/* 불필요하게 번들에 포함됨 */}
      <Footer />  {/* 불필요하게 번들에 포함됨 */}
    </div>
  );
}

// ✅ 좋은 예: 필요한 부분만 Client Component
export default function Page() {
  return (
    <div>
      <Header />  {/* Server Component */}
      <InteractiveSection />  {/* 이것만 Client Component */}
      <Footer />  {/* Server Component */}
    </div>
  );
}
```

#### 실수 2: Context를 최상위에 배치

```tsx
// ❌ 나쁜 예: 모든 것이 Client Component가 됨
'use client';

export default function RootLayout({ children }) {
  return (
    <ThemeProvider>  {/* 이로 인해 전체 앱이 Client Component */}
      {children}
    </ThemeProvider>
  );
}

// ✅ 좋은 예: Provider를 분리
// app/layout.tsx (Server Component)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>
          {children}  {/* Server Component 유지 */}
        </Providers>
      </body>
    </html>
  );
}

// app/providers.tsx (Client Component)
'use client';

export function Providers({ children }) {
  return (
    <ThemeProvider>
      {children}  {/* children은 여전히 Server Component */}
    </ThemeProvider>
  );
}
```

#### 실수 3: 데이터 페칭 워터폴

```tsx
// ❌ 나쁜 예
async function Page({ params }) {
  const user = await fetchUser(params.id);
  const posts = await fetchPosts(user.id);  // user를 기다림
  const comments = await fetchComments(posts[0].id);  // posts를 기다림

  // 총 3번의 순차적 요청
}

// ✅ 좋은 예
async function Page({ params }) {
  // 독립적인 요청은 병렬로
  const [user, postsData] = await Promise.all([
    fetchUser(params.id),
    fetchUserPosts(params.id)  // user 정보 없이도 가능
  ]);

  // 또는 Suspense로 분리
  return (
    <div>
      <UserInfo user={user} />
      <Suspense fallback={<PostsSkeleton />}>
        <Posts userId={params.id} />
      </Suspense>
    </div>
  );
}
```

## 결론

React Server Components는 현대 웹 애플리케이션 개발의 패러다임을 변화시키는 강력한 기술입니다. 이 글에서 다룬 핵심 내용을 정리하면:

### 주요 장점

1. **성능 향상**
   - 클라이언트 번들 크기 90% 이상 감소 가능
   - 초기 페이지 로드 시간 대폭 단축
   - Streaming으로 사용자 경험 개선

2. **개발자 경험**
   - 서버 리소스 직접 접근
   - 데이터 페칭 코드 단순화
   - 자동 코드 분할

3. **보안**
   - 민감한 로직을 서버에 안전하게 보관
   - API 키 노출 위험 제거

### 사용 가이드

#### Server Components를 사용할 때
- 데이터 페칭
- 백엔드 리소스 접근
- 무거운 라이브러리 사용
- 정적 컨텐츠 렌더링

#### Client Components를 사용할 때
- 상태 관리 (useState, useReducer)
- 이벤트 핸들러 (onClick, onChange)
- 브라우저 API (localStorage, window)
- 사용자 인터랙션

### Best Practices 요약

```tsx
// 1. 가능한 한 Server Components 사용
async function Page() {
  const data = await fetchData();
  return <Content data={data} />;
}

// 2. Client Components는 잎(leaf)에 가깝게
function Page() {
  return (
    <div>
      <Header />  {/* Server */}
      <InteractiveButton />  {/* Client */}
      <Footer />  {/* Server */}
    </div>
  );
}

// 3. Suspense로 점진적 렌더링
function Page() {
  return (
    <>
      <Suspense fallback={<Skeleton />}>
        <SlowComponent />
      </Suspense>
    </>
  );
}

// 4. 병렬 데이터 페칭
async function Page() {
  const [a, b, c] = await Promise.all([
    fetchA(),
    fetchB(),
    fetchC()
  ]);
}

// 5. Server Actions로 mutations
'use server';

export async function createPost(data: FormData) {
  await db.posts.create({ data });
  revalidatePath('/posts');
}
```

### 다음 단계

React Server Components를 프로젝트에 도입하려면:

1. **Next.js 13+ 사용**: App Router로 시작
2. **점진적 마이그레이션**: 새 페이지부터 RSC 적용
3. **성능 측정**: Lighthouse로 개선 효과 확인
4. **팀 교육**: 팀원들과 개념 공유

React Server Components는 아직 진화 중이지만, 이미 프로덕션에서 검증된 기술입니다. 올바르게 활용하면 더 빠르고, 더 안전하고, 더 유지보수하기 쉬운 애플리케이션을 만들 수 있습니다.

## 참고 자료

- [React Server Components 공식 문서](https://react.dev/reference/react/use-client)
- [Next.js App Router 문서](https://nextjs.org/docs/app)
- [Server Actions 가이드](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [RSC From Scratch](https://github.com/reactwg/server-components/discussions/5)
- [Making Sense of React Server Components](https://www.joshwcomeau.com/react/server-components/)
