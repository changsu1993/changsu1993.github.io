---
title: "React Suspense 완벽 가이드 - 데이터 페칭과 코드 스플리팅"
description: "React Suspense의 개념부터 실전 활용까지 완벽 가이드. React.lazy를 활용한 코드 스플리팅, React 19 use() 훅으로 데이터 페칭, TanStack Query 연동, Error Boundary 조합 패턴을 상세한 코드 예제와 함께 설명합니다."
date: 2025-12-04 09:00:00 +0900
categories: [Frontend, React]
tags: [react, suspense, code-splitting, lazy-loading, concurrent, error-boundary, tanstack-query, react-19, data-fetching]
---

## 개요

React 애플리케이션에서 비동기 작업을 처리할 때 로딩 상태를 관리하는 것은 복잡한 일입니다. **Suspense**는 이러한 비동기 작업의 로딩 상태를 **선언적으로** 처리할 수 있게 해주는 React의 핵심 기능입니다.

이 글에서는 Suspense의 기본 개념부터 코드 스플리팅, 데이터 페칭, 그리고 실전 패턴까지 상세히 알아봅니다.

---

## Suspense란 무엇인가?

### 선언적 로딩 UI

Suspense는 **자식 컴포넌트가 로딩을 완료할 때까지 폴백 UI를 표시**하는 React 컴포넌트입니다. 기존의 명령형 로딩 처리 방식과 비교해보겠습니다.

```tsx
// 기존 방식: 명령형 로딩 처리
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setIsLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [userId]);

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

```tsx
// Suspense 방식: 선언적 로딩 처리
function UserProfile({ userId }: { userId: string }) {
  const user = use(fetchUser(userId)); // React 19 use() 훅
  return <div>{user.name}</div>;
}

// 사용하는 곳
function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile userId="1" />
    </Suspense>
  );
}
```

Suspense를 사용하면 **로딩 상태 관리 로직이 컴포넌트에서 분리**되어 코드가 훨씬 깔끔해집니다.

### Suspense의 동작 원리

Suspense는 자식 컴포넌트 트리 전체를 **하나의 단위**로 취급합니다.

| 동작 | 설명 |
|------|------|
| 초기 렌더링 | 자식 중 하나라도 "suspend"되면 폴백 표시 |
| 데이터 로드 완료 | 모든 자식이 준비되면 실제 UI로 교체 |
| 재suspend | 이미 표시된 UI가 suspend되면 폴백으로 다시 전환 |
| Transition 사용 시 | `startTransition` 내에서는 이전 UI 유지 |

### Suspense가 활성화되는 조건

Suspense는 다음 소스에서만 활성화됩니다.

```tsx
// 1. React.lazy로 지연 로드된 컴포넌트
const LazyComponent = lazy(() => import('./MyComponent'));

// 2. use() 훅으로 읽는 Promise (React 19)
const data = use(fetchDataPromise);

// 3. Suspense 지원 프레임워크 (Next.js, Relay 등)
const data = useSuspenseQuery(queryOptions);
```

**주의**: `useEffect`나 이벤트 핸들러 내부의 데이터 페칭은 Suspense를 활성화하지 않습니다.

---

## 코드 스플리팅과 React.lazy

### React.lazy() 기본 사용법

`React.lazy()`는 컴포넌트 코드를 **첫 렌더링 시점까지 지연**시켜 초기 번들 크기를 줄입니다.

```tsx
import { lazy, Suspense } from 'react';

// 정적 import (번들에 포함)
// import HeavyEditor from './HeavyEditor';

// 동적 import (별도 청크로 분리)
const HeavyEditor = lazy(() => import('./HeavyEditor'));

function App() {
  const [showEditor, setShowEditor] = useState(false);

  return (
    <div>
      <button onClick={() => setShowEditor(true)}>
        에디터 열기
      </button>

      {showEditor && (
        <Suspense fallback={<div>에디터 로딩 중...</div>}>
          <HeavyEditor />
        </Suspense>
      )}
    </div>
  );
}
```

### lazy() 선언 위치

**중요**: `lazy()`는 반드시 **모듈 최상위 레벨**에서 선언해야 합니다.

```tsx
// 올바른 사용법
const MyComponent = lazy(() => import('./MyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <MyComponent />
    </Suspense>
  );
}
```

```tsx
// 잘못된 사용법 - 컴포넌트 내부 선언
function App() {
  // 매 렌더링마다 새로운 lazy 컴포넌트 생성
  // 상태가 리셋되는 문제 발생
  const MyComponent = lazy(() => import('./MyComponent'));

  return (
    <Suspense fallback={<Loading />}>
      <MyComponent />
    </Suspense>
  );
}
```

### 라우트 기반 코드 스플리팅

가장 효과적인 코드 스플리팅 전략은 **라우트 단위**입니다.

```tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// 각 페이지를 별도 청크로 분리
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));

function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner" />
      <p>페이지 로딩 중...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 컴포넌트 기반 코드 스플리팅

무거운 컴포넌트를 필요할 때만 로드합니다.

```tsx
import { lazy, Suspense, useState } from 'react';

// 무거운 라이브러리를 사용하는 컴포넌트들
const RichTextEditor = lazy(() => import('./RichTextEditor'));
const ChartDashboard = lazy(() => import('./ChartDashboard'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function ContentBlock({ type }: { type: 'editor' | 'chart' | 'video' }) {
  return (
    <Suspense fallback={<div className="skeleton" />}>
      {type === 'editor' && <RichTextEditor />}
      {type === 'chart' && <ChartDashboard />}
      {type === 'video' && <VideoPlayer />}
    </Suspense>
  );
}

function Dashboard() {
  const [activeTab, setActiveTab] = useState<'editor' | 'chart' | 'video'>('editor');

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab('editor')}>에디터</button>
        <button onClick={() => setActiveTab('chart')}>차트</button>
        <button onClick={() => setActiveTab('video')}>비디오</button>
      </nav>

      <ContentBlock type={activeTab} />
    </div>
  );
}
```

### Named Export 처리

`lazy()`는 기본적으로 **default export**만 지원합니다. named export를 사용하려면 중간 처리가 필요합니다.

```tsx
// MyComponents.tsx
export function ComponentA() { /* ... */ }
export function ComponentB() { /* ... */ }
```

```tsx
// 사용하는 곳
const ComponentA = lazy(() =>
  import('./MyComponents').then(module => ({
    default: module.ComponentA
  }))
);

const ComponentB = lazy(() =>
  import('./MyComponents').then(module => ({
    default: module.ComponentB
  }))
);
```

---

## Suspense로 데이터 페칭

### 전통적인 데이터 페칭의 문제점

#### Waterfall 문제

컴포넌트가 순차적으로 데이터를 요청하면 성능이 저하됩니다.

```tsx
// Waterfall 문제 발생
function Dashboard() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    // 1. 먼저 유저 로드 (500ms)
    fetchUser().then(setUser);
  }, []);

  useEffect(() => {
    if (user) {
      // 2. 유저 로드 후 포스트 로드 (500ms)
      fetchPosts(user.id).then(setPosts);
    }
  }, [user]);

  // 총 1000ms 소요 (순차적)
}
```

#### 분산된 로딩 상태

각 컴포넌트가 개별적으로 로딩 상태를 관리하면 UI가 들쭉날쭉합니다.

```tsx
function UserCard() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  // ... 로딩 상태 관리 로직

  if (loading) return <Spinner />; // 각자 스피너 표시
  return <div>{user.name}</div>;
}

function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  // ... 로딩 상태 관리 로직

  if (loading) return <Spinner />; // 각자 스피너 표시
  return <ul>{posts.map(/* ... */)}</ul>;
}
```

### React 19의 use() 훅

React 19에서 도입된 `use()` 훅은 Promise를 직접 읽을 수 있게 해줍니다. React 19의 새로운 기능들에 대해서는 [React 19 완벽 가이드](/posts/react-19-complete-guide/)를 참고하세요.

```tsx
import { use, Suspense } from 'react';

// 캐싱된 Promise를 반환하는 함수 (중요!)
const userPromise = fetchUser(1);
const postsPromise = fetchPosts(1);

function UserProfile() {
  const user = use(userPromise);
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
    </div>
  );
}

function PostList() {
  const posts = use(postsPromise);
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

function Dashboard() {
  return (
    <Suspense fallback={<DashboardSkeleton />}>
      <UserProfile />
      <PostList />
    </Suspense>
  );
}
```

### use() 훅의 특징

`use()`는 다른 훅들과 달리 **조건문 내부에서도 사용**할 수 있습니다.

```tsx
function Message({ messagePromise, shouldLoad }: Props) {
  // 조건부 사용 가능
  if (shouldLoad) {
    const message = use(messagePromise);
    return <p>{message.content}</p>;
  }

  return <p>메시지를 불러오려면 버튼을 클릭하세요.</p>;
}
```

**주의사항**:
- `use()`는 컴포넌트나 훅 내부에서만 호출 가능
- `try-catch` 블록 내부에서 사용 불가
- 에러는 가장 가까운 Error Boundary에서 처리

### TanStack Query와 Suspense 연동

TanStack Query(React Query)는 Suspense를 위한 전용 훅을 제공합니다. TanStack Query의 기본 사용법은 [React Query 완벽 가이드](/posts/react-query-guide/)에서 확인할 수 있습니다.

```tsx
import { useSuspenseQuery } from '@tanstack/react-query';
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

interface User {
  id: number;
  name: string;
  email: string;
}

function UserProfile({ userId }: { userId: number }) {
  // useSuspenseQuery는 data가 항상 정의됨을 보장
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  // isLoading, isError 체크 불필요
  return (
    <div className="profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function ProfilePage({ userId }: { userId: number }) {
  return (
    <ErrorBoundary fallback={<div>프로필을 불러올 수 없습니다.</div>}>
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile userId={userId} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### useSuspenseQuery vs useQuery

| 특징 | useQuery | useSuspenseQuery |
|------|----------|------------------|
| data 타입 | `T \| undefined` | `T` (항상 정의됨) |
| 로딩 처리 | `isLoading` 체크 필요 | Suspense가 처리 |
| 에러 처리 | `isError` 체크 필요 | Error Boundary가 처리 |
| enabled 옵션 | 사용 가능 | 사용 불가 |
| placeholderData | 사용 가능 | 사용 불가 |

### 여러 쿼리 병렬 실행

`useSuspenseQueries`로 여러 쿼리를 병렬로 실행합니다.

```tsx
import { useSuspenseQueries } from '@tanstack/react-query';

function Dashboard({ userId }: { userId: number }) {
  const [
    { data: user },
    { data: posts },
    { data: notifications }
  ] = useSuspenseQueries({
    queries: [
      {
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId),
      },
      {
        queryKey: ['posts', userId],
        queryFn: () => fetchUserPosts(userId),
      },
      {
        queryKey: ['notifications', userId],
        queryFn: () => fetchNotifications(userId),
      },
    ],
  });

  return (
    <div>
      <UserCard user={user} />
      <PostList posts={posts} />
      <NotificationBadge count={notifications.length} />
    </div>
  );
}
```

---

## Suspense 경계 설계

### 단일 Suspense vs 다중 Suspense

Suspense 경계를 어떻게 배치하느냐에 따라 사용자 경험이 달라집니다.

#### 단일 Suspense: 동시 표시

모든 컨텐츠가 함께 로드됩니다.

```tsx
function Dashboard() {
  return (
    <Suspense fallback={<DashboardSkeleton />}>
      <Header />
      <Sidebar />
      <MainContent />
      <Footer />
    </Suspense>
  );
}
```

**장점**: 모든 컨텐츠가 동시에 나타나 시각적 안정감
**단점**: 가장 느린 컴포넌트가 전체를 지연

#### 다중 Suspense: 점진적 표시

각 영역이 독립적으로 로드됩니다.

```tsx
function Dashboard() {
  return (
    <div className="dashboard">
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>

      <div className="main-area">
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>

        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />
        </Suspense>
      </div>

      <Suspense fallback={<FooterSkeleton />}>
        <Footer />
      </Suspense>
    </div>
  );
}
```

**장점**: 빠른 컨텐츠 먼저 표시, 체감 성능 향상
**단점**: UI가 점진적으로 나타나 산만할 수 있음

### 중첩 Suspense 패턴

중첩 Suspense로 **계층적 로딩**을 구현합니다.

```tsx
function App() {
  return (
    // 외부 Suspense: 페이지 전체 로딩
    <Suspense fallback={<PageLoader />}>
      <DashboardPage />
    </Suspense>
  );
}

function DashboardPage() {
  return (
    <div className="dashboard">
      <DashboardHeader />

      {/* 내부 Suspense: 위젯들의 로딩 */}
      <div className="widgets">
        <Suspense fallback={<WidgetSkeleton />}>
          <AnalyticsWidget />
        </Suspense>

        <Suspense fallback={<WidgetSkeleton />}>
          <RevenueWidget />
        </Suspense>

        <Suspense fallback={<WidgetSkeleton />}>
          <UsersWidget />
        </Suspense>
      </div>
    </div>
  );
}
```

### Suspense 경계 설계 원칙

| 원칙 | 설명 |
|------|------|
| 사용자 경험 중심 | 디자이너와 협업하여 로딩 시퀀스 설계 |
| 논리적 그룹화 | 관련 컴포넌트를 하나의 경계로 묶기 |
| 핵심 컨텐츠 우선 | 중요한 컨텐츠는 빠르게, 부가 기능은 나중에 |
| 과도한 분할 지양 | 모든 컴포넌트에 개별 Suspense는 비효율적 |

---

## Error Boundary와 Suspense 조합

### 기본 패턴

Suspense와 Error Boundary를 함께 사용하면 **로딩, 에러, 성공** 세 상태를 깔끔하게 처리합니다. Error Boundary에 대한 자세한 내용은 [React Error Boundary 완벽 가이드](/posts/react-error-boundary-complete-guide/)를 참고하세요.

```tsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }: {
  error: Error;
  resetErrorBoundary: () => void;
}) {
  return (
    <div className="error-container">
      <h2>문제가 발생했습니다</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>다시 시도</button>
    </div>
  );
}

function DataComponent() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Suspense fallback={<LoadingSpinner />}>
        <AsyncDataDisplay />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### 재사용 가능한 AsyncBoundary 컴포넌트

```tsx
import { Suspense, ReactNode } from 'react';
import { ErrorBoundary, FallbackProps } from 'react-error-boundary';

interface AsyncBoundaryProps {
  children: ReactNode;
  loadingFallback: ReactNode;
  errorFallback?: (props: FallbackProps) => ReactNode;
  onError?: (error: Error, info: { componentStack?: string }) => void;
  onReset?: () => void;
}

function DefaultErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
  return (
    <div className="p-4 bg-red-50 rounded-lg">
      <h3 className="text-red-800 font-bold">오류 발생</h3>
      <p className="text-red-600 text-sm mt-1">{error.message}</p>
      <button
        onClick={resetErrorBoundary}
        className="mt-3 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
      >
        다시 시도
      </button>
    </div>
  );
}

function AsyncBoundary({
  children,
  loadingFallback,
  errorFallback = DefaultErrorFallback,
  onError,
  onReset,
}: AsyncBoundaryProps) {
  return (
    <ErrorBoundary
      fallbackRender={errorFallback}
      onError={onError}
      onReset={onReset}
    >
      <Suspense fallback={loadingFallback}>
        {children}
      </Suspense>
    </ErrorBoundary>
  );
}

// 사용 예시
function Dashboard() {
  return (
    <div className="grid grid-cols-2 gap-4">
      <AsyncBoundary loadingFallback={<CardSkeleton />}>
        <RevenueCard />
      </AsyncBoundary>

      <AsyncBoundary loadingFallback={<CardSkeleton />}>
        <UsersCard />
      </AsyncBoundary>

      <AsyncBoundary loadingFallback={<ChartSkeleton />}>
        <AnalyticsChart />
      </AsyncBoundary>

      <AsyncBoundary loadingFallback={<TableSkeleton />}>
        <RecentOrders />
      </AsyncBoundary>
    </div>
  );
}
```

### TanStack Query의 QueryErrorResetBoundary

TanStack Query는 에러 리셋을 위한 전용 컴포넌트를 제공합니다.

```tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function QueryBoundary({ children }: { children: ReactNode }) {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div className="error-panel">
              <p>데이터를 불러올 수 없습니다: {error.message}</p>
              <button onClick={() => resetErrorBoundary()}>
                다시 시도
              </button>
            </div>
          )}
        >
          <Suspense fallback={<Loading />}>
            {children}
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

function App() {
  return (
    <QueryBoundary>
      <UserDashboard />
    </QueryBoundary>
  );
}
```

---

## 실전 예제

### 대시보드 페이지 구현

```tsx
import { lazy, Suspense } from 'react';
import { useSuspenseQuery, useSuspenseQueries } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

// 무거운 차트 컴포넌트는 lazy 로드
const AnalyticsChart = lazy(() => import('./AnalyticsChart'));
const RevenueChart = lazy(() => import('./RevenueChart'));

// API 함수들
async function fetchDashboardStats(): Promise<DashboardStats> {
  const response = await fetch('/api/dashboard/stats');
  if (!response.ok) throw new Error('통계를 불러올 수 없습니다');
  return response.json();
}

async function fetchRecentOrders(): Promise<Order[]> {
  const response = await fetch('/api/orders/recent');
  if (!response.ok) throw new Error('주문 목록을 불러올 수 없습니다');
  return response.json();
}

// 스켈레톤 컴포넌트들
function StatCardSkeleton() {
  return (
    <div className="p-4 bg-gray-100 rounded-lg animate-pulse">
      <div className="h-4 bg-gray-200 rounded w-1/2 mb-2" />
      <div className="h-8 bg-gray-200 rounded w-3/4" />
    </div>
  );
}

function ChartSkeleton() {
  return (
    <div className="p-4 bg-gray-100 rounded-lg h-64 animate-pulse">
      <div className="h-full bg-gray-200 rounded" />
    </div>
  );
}

function TableSkeleton() {
  return (
    <div className="p-4 bg-gray-100 rounded-lg animate-pulse">
      {[...Array(5)].map((_, i) => (
        <div key={i} className="h-12 bg-gray-200 rounded mb-2" />
      ))}
    </div>
  );
}

// 통계 카드 컴포넌트
function StatsCards() {
  const { data: stats } = useSuspenseQuery({
    queryKey: ['dashboard', 'stats'],
    queryFn: fetchDashboardStats,
  });

  return (
    <div className="grid grid-cols-4 gap-4 mb-6">
      <StatCard title="총 매출" value={`${stats.revenue.toLocaleString()}원`} />
      <StatCard title="주문 수" value={stats.orders.toString()} />
      <StatCard title="신규 고객" value={stats.newCustomers.toString()} />
      <StatCard title="전환율" value={`${stats.conversionRate}%`} />
    </div>
  );
}

function StatCard({ title, value }: { title: string; value: string }) {
  return (
    <div className="p-4 bg-white rounded-lg shadow">
      <p className="text-gray-500 text-sm">{title}</p>
      <p className="text-2xl font-bold mt-1">{value}</p>
    </div>
  );
}

// 최근 주문 테이블
function RecentOrdersTable() {
  const { data: orders } = useSuspenseQuery({
    queryKey: ['orders', 'recent'],
    queryFn: fetchRecentOrders,
  });

  return (
    <table className="w-full">
      <thead>
        <tr className="border-b">
          <th className="text-left p-2">주문 ID</th>
          <th className="text-left p-2">고객</th>
          <th className="text-left p-2">금액</th>
          <th className="text-left p-2">상태</th>
        </tr>
      </thead>
      <tbody>
        {orders.map(order => (
          <tr key={order.id} className="border-b">
            <td className="p-2">{order.id}</td>
            <td className="p-2">{order.customerName}</td>
            <td className="p-2">{order.amount.toLocaleString()}원</td>
            <td className="p-2">
              <span className={`badge badge-${order.status}`}>
                {order.statusLabel}
              </span>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// 에러 폴백 컴포넌트
function WidgetError({ error, resetErrorBoundary }: {
  error: Error;
  resetErrorBoundary: () => void;
}) {
  return (
    <div className="p-4 bg-red-50 rounded-lg text-center">
      <p className="text-red-600 mb-2">{error.message}</p>
      <button
        onClick={resetErrorBoundary}
        className="px-3 py-1 bg-red-500 text-white rounded text-sm"
      >
        다시 시도
      </button>
    </div>
  );
}

// 메인 대시보드
function Dashboard() {
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-6">대시보드</h1>

      {/* 통계 카드 영역 */}
      <ErrorBoundary FallbackComponent={WidgetError}>
        <Suspense fallback={
          <div className="grid grid-cols-4 gap-4 mb-6">
            {[...Array(4)].map((_, i) => <StatCardSkeleton key={i} />)}
          </div>
        }>
          <StatsCards />
        </Suspense>
      </ErrorBoundary>

      {/* 차트 영역 */}
      <div className="grid grid-cols-2 gap-6 mb-6">
        <ErrorBoundary FallbackComponent={WidgetError}>
          <Suspense fallback={<ChartSkeleton />}>
            <AnalyticsChart />
          </Suspense>
        </ErrorBoundary>

        <ErrorBoundary FallbackComponent={WidgetError}>
          <Suspense fallback={<ChartSkeleton />}>
            <RevenueChart />
          </Suspense>
        </ErrorBoundary>
      </div>

      {/* 최근 주문 테이블 */}
      <div className="bg-white rounded-lg shadow p-4">
        <h2 className="text-lg font-bold mb-4">최근 주문</h2>
        <ErrorBoundary FallbackComponent={WidgetError}>
          <Suspense fallback={<TableSkeleton />}>
            <RecentOrdersTable />
          </Suspense>
        </ErrorBoundary>
      </div>
    </div>
  );
}

export default Dashboard;
```

### Skeleton UI 패턴

사용자 경험을 높이는 Skeleton UI 구현 패턴입니다.

```tsx
// Skeleton 기본 컴포넌트
function Skeleton({ className }: { className?: string }) {
  return (
    <div
      className={`animate-pulse bg-gray-200 rounded ${className || ''}`}
    />
  );
}

// 프로필 카드 스켈레톤
function ProfileCardSkeleton() {
  return (
    <div className="p-4 bg-white rounded-lg shadow">
      <div className="flex items-center space-x-4">
        <Skeleton className="w-16 h-16 rounded-full" />
        <div className="flex-1">
          <Skeleton className="h-5 w-32 mb-2" />
          <Skeleton className="h-4 w-48" />
        </div>
      </div>
      <div className="mt-4 space-y-2">
        <Skeleton className="h-4 w-full" />
        <Skeleton className="h-4 w-3/4" />
      </div>
    </div>
  );
}

// 실제 프로필 카드
function ProfileCard({ userId }: { userId: string }) {
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  return (
    <div className="p-4 bg-white rounded-lg shadow">
      <div className="flex items-center space-x-4">
        <img
          src={user.avatar}
          alt={user.name}
          className="w-16 h-16 rounded-full"
        />
        <div>
          <h3 className="text-lg font-bold">{user.name}</h3>
          <p className="text-gray-500">{user.email}</p>
        </div>
      </div>
      <p className="mt-4 text-gray-600">{user.bio}</p>
    </div>
  );
}

// 사용
function UserSection({ userId }: { userId: string }) {
  return (
    <Suspense fallback={<ProfileCardSkeleton />}>
      <ProfileCard userId={userId} />
    </Suspense>
  );
}
```

---

## 성능 최적화 팁

### Preload 패턴

사용자가 필요로 하기 전에 미리 컴포넌트를 로드합니다.

```tsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

// preload 함수 정의
function preloadHeavyComponent() {
  // import()를 호출하면 로딩이 시작됨
  import('./HeavyComponent');
}

function App() {
  return (
    <div>
      {/* 호버 시 미리 로드 시작 */}
      <button
        onMouseEnter={preloadHeavyComponent}
        onFocus={preloadHeavyComponent}
      >
        무거운 컴포넌트 열기
      </button>

      <Suspense fallback={<Loading />}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}
```

### 라우트 Preload

```tsx
import { lazy } from 'react';
import { Link, useNavigate } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

// preload 맵
const preloadMap = {
  dashboard: () => import('./pages/Dashboard'),
  settings: () => import('./pages/Settings'),
};

function Navigation() {
  return (
    <nav>
      <Link
        to="/dashboard"
        onMouseEnter={() => preloadMap.dashboard()}
      >
        대시보드
      </Link>
      <Link
        to="/settings"
        onMouseEnter={() => preloadMap.settings()}
      >
        설정
      </Link>
    </nav>
  );
}
```

### TanStack Query Prefetch

```tsx
import { useQueryClient } from '@tanstack/react-query';

function UserList({ users }: { users: User[] }) {
  const queryClient = useQueryClient();

  const prefetchUser = (userId: string) => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 60000, // 1분간 fresh 상태 유지
    });
  };

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <Link
            to={`/users/${user.id}`}
            onMouseEnter={() => prefetchUser(user.id)}
          >
            {user.name}
          </Link>
        </li>
      ))}
    </ul>
  );
}
```

### 번들 크기 최적화

```tsx
// 조건부 import로 번들 최적화
const ChartComponent = lazy(() => {
  // 모바일에서는 가벼운 차트 사용
  if (window.innerWidth < 768) {
    return import('./LightChart');
  }
  return import('./FullChart');
});

// 기능별 청크 분리
const AdminPanel = lazy(() =>
  import(/* webpackChunkName: "admin" */ './AdminPanel')
);

const EditorSuite = lazy(() =>
  import(/* webpackChunkName: "editor" */ './EditorSuite')
);
```

### Transition으로 UX 개선

`startTransition`을 사용하면 로딩 중에도 이전 UI를 유지할 수 있습니다.

```tsx
import { useState, useTransition, Suspense } from 'react';

function TabNavigation() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const handleTabChange = (newTab: string) => {
    startTransition(() => {
      setTab(newTab);
    });
  };

  return (
    <div>
      <nav style={{ opacity: isPending ? 0.7 : 1 }}>
        <button onClick={() => handleTabChange('home')}>홈</button>
        <button onClick={() => handleTabChange('profile')}>프로필</button>
        <button onClick={() => handleTabChange('settings')}>설정</button>
        {isPending && <span className="ml-2">로딩 중...</span>}
      </nav>

      <Suspense fallback={<TabSkeleton />}>
        {tab === 'home' && <HomeTab />}
        {tab === 'profile' && <ProfileTab />}
        {tab === 'settings' && <SettingsTab />}
      </Suspense>
    </div>
  );
}
```

---

## 주의사항과 베스트 프랙티스

### SSR에서의 Suspense

#### Next.js App Router

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import { UserProfile } from './UserProfile';
import { RecentActivity } from './RecentActivity';

export default function DashboardPage() {
  return (
    <div>
      <h1>대시보드</h1>

      {/* 각각 독립적으로 스트리밍 */}
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile />
      </Suspense>

      <Suspense fallback={<ActivitySkeleton />}>
        <RecentActivity />
      </Suspense>
    </div>
  );
}
```

#### Streaming SSR

Next.js에서 Suspense 경계마다 HTML이 점진적으로 스트리밍됩니다.

```tsx
// 서버에서 데이터 페칭과 함께 스트리밍
async function UserProfile() {
  const user = await fetchUser(); // 서버에서 실행

  return (
    <div className="profile">
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}
```

### 테스트 작성 방법

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// 테스트용 QueryClient 생성
function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        gcTime: 0,
      },
    },
  });
}

// 테스트 래퍼
function TestWrapper({ children }: { children: React.ReactNode }) {
  const queryClient = createTestQueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <ErrorBoundary fallback={<div>Error</div>}>
        <Suspense fallback={<div>Loading...</div>}>
          {children}
        </Suspense>
      </ErrorBoundary>
    </QueryClientProvider>
  );
}

// 테스트 예시
describe('UserProfile', () => {
  it('로딩 후 사용자 정보를 표시한다', async () => {
    // API 모킹
    vi.spyOn(global, 'fetch').mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ name: '홍길동', email: 'hong@example.com' }),
    } as Response);

    render(
      <TestWrapper>
        <UserProfile userId="1" />
      </TestWrapper>
    );

    // 로딩 상태 확인
    expect(screen.getByText('Loading...')).toBeInTheDocument();

    // 데이터 로드 완료 확인
    await waitFor(() => {
      expect(screen.getByText('홍길동')).toBeInTheDocument();
    });
  });

  it('에러 시 에러 메시지를 표시한다', async () => {
    vi.spyOn(global, 'fetch').mockRejectedValueOnce(new Error('API Error'));

    render(
      <TestWrapper>
        <UserProfile userId="1" />
      </TestWrapper>
    );

    await waitFor(() => {
      expect(screen.getByText('Error')).toBeInTheDocument();
    });
  });
});
```

### 흔한 실수와 해결 방법

#### 1. lazy를 컴포넌트 내부에서 선언

```tsx
// 잘못된 방식
function App() {
  const LazyComponent = lazy(() => import('./Component')); // 매 렌더링마다 새로 생성
  return <LazyComponent />;
}

// 올바른 방식
const LazyComponent = lazy(() => import('./Component')); // 모듈 레벨에서 선언

function App() {
  return <LazyComponent />;
}
```

#### 2. Suspense 없이 lazy 컴포넌트 사용

```tsx
// 에러 발생
const LazyComponent = lazy(() => import('./Component'));

function App() {
  return <LazyComponent />; // Suspense 없음
}

// 올바른 방식
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <LazyComponent />
    </Suspense>
  );
}
```

#### 3. 매번 새로운 Promise 생성

```tsx
// 문제: 매 렌더링마다 새 Promise 생성
function UserProfile({ userId }: { userId: string }) {
  const user = use(fetchUser(userId)); // 무한 로딩 발생
  return <div>{user.name}</div>;
}

// 해결: 캐싱된 Promise 사용
const userCache = new Map();

function getCachedUser(userId: string) {
  if (!userCache.has(userId)) {
    userCache.set(userId, fetchUser(userId));
  }
  return userCache.get(userId);
}

function UserProfile({ userId }: { userId: string }) {
  const user = use(getCachedUser(userId));
  return <div>{user.name}</div>;
}
```

#### 4. Error Boundary 없이 Suspense 사용

```tsx
// 에러 시 앱 크래시
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <DataComponent /> {/* 에러 발생 시 처리 불가 */}
    </Suspense>
  );
}

// 올바른 방식
function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Suspense fallback={<Loading />}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 자주 묻는 질문 (FAQ)

### Q1. Suspense와 Error Boundary의 순서는?

**A:** 일반적으로 **Error Boundary가 Suspense를 감싸는 것**이 권장됩니다.

```tsx
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Loading />}>
    <Component />
  </Suspense>
</ErrorBoundary>
```

이렇게 하면 로딩 중 발생한 에러도 Error Boundary에서 처리됩니다.

---

### Q2. 왜 useEffect에서 fetch는 Suspense를 트리거하지 않나요?

**A:** Suspense는 **렌더링 중에** 발생하는 비동기 작업만 감지합니다. useEffect는 렌더링 **이후에** 실행되므로 Suspense가 이를 감지하지 못합니다.

```tsx
// Suspense 트리거 안됨
useEffect(() => {
  fetch('/api/data').then(setData);
}, []);

// Suspense 트리거됨
const data = use(fetchPromise); // 렌더링 중 호출
```

---

### Q3. 모든 컴포넌트에 Suspense를 사용해야 하나요?

**A:** 아닙니다. Suspense는 **비동기 작업이 있는 컴포넌트**에만 필요합니다. 동기적으로 렌더링되는 일반 컴포넌트에는 Suspense가 필요 없습니다.

---

### Q4. lazy와 Suspense의 성능 영향은?

**A:** `lazy()`는 번들 크기를 줄이고 초기 로딩을 빠르게 합니다. 단, 해당 컴포넌트를 사용할 때 네트워크 요청이 발생하므로 **preload 전략**을 함께 사용하면 좋습니다.

---

### Q5. Suspense fallback이 너무 자주 깜빡여요

**A:** `startTransition`이나 `useDeferredValue`를 사용하면 이전 컨텐츠를 유지하면서 새 컨텐츠를 준비할 수 있습니다.

```tsx
const [isPending, startTransition] = useTransition();

const handleChange = (value) => {
  startTransition(() => {
    setValue(value); // 이전 UI 유지하면서 업데이트
  });
};
```

---

## 마무리

Suspense는 React 애플리케이션에서 비동기 상태를 선언적으로 처리할 수 있게 해주는 강력한 도구입니다.

### 핵심 포인트 정리

| 개념 | 설명 |
|------|------|
| Suspense | 로딩 상태를 선언적으로 처리하는 컴포넌트 |
| React.lazy | 컴포넌트 코드 스플리팅을 위한 함수 |
| use() | React 19에서 Promise를 읽는 훅 |
| useSuspenseQuery | TanStack Query의 Suspense 지원 훅 |
| startTransition | 이전 UI를 유지하면서 업데이트 |
| Error Boundary | Suspense와 함께 에러 처리 |

### 권장 사항

1. **코드 스플리팅**: 라우트 단위로 `lazy()`를 사용하여 초기 번들 크기 감소
2. **점진적 로딩**: 중요한 컨텐츠부터 표시되도록 Suspense 경계 설계
3. **에러 처리**: 항상 Error Boundary와 함께 Suspense 사용
4. **Preload**: 사용자 상호작용 전에 미리 데이터/컴포넌트 로드
5. **Transition**: 급격한 UI 전환을 피하기 위해 startTransition 활용

Suspense를 적절히 활용하면 코드는 간결해지고, 사용자 경험은 크게 향상됩니다.

---

## 참고 자료

- [React 공식 문서 - Suspense](https://react.dev/reference/react/Suspense)
- [React 공식 문서 - lazy](https://react.dev/reference/react/lazy)
- [React 공식 문서 - use](https://react.dev/reference/react/use)
- [TanStack Query - Suspense](https://tanstack.com/query/latest/docs/framework/react/guides/suspense)
- [react-error-boundary](https://github.com/bvaughn/react-error-boundary)
