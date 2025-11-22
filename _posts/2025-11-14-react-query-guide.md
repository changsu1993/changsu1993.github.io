---
title: "React Query 완벽 가이드 - 서버 상태 관리의 정석 (TanStack Query)"
date: 2025-11-14 14:00:00 +0900
categories: [React, State Management]
tags: [react, react-query, tanstack-query, server-state, data-fetching, cache, 서버상태, 상태관리]
author: changsu
toc: true
comments: true
description: React Query로 서버 상태 관리를 마스터하세요. useQuery부터 useMutation, 무한 스크롤, 낙관적 업데이트까지. 캐싱 전략과 실전 예제로 데이터 페칭의 모든 것을 다룹니다. TanStack Query v5 완벽 가이드.
---

## 들어가며

[이전 글](/posts/react-state-management-guide/)에서 클라이언트 상태 관리를, [두 번째 글](/posts/zustand-vs-redux-toolkit/)에서 전역 상태 관리 라이브러리를 다뤘습니다. 하지만 실무에서 가장 많이 다루는 상태는 따로 있습니다. 바로 **서버 상태**입니다.

블로그 포스트 목록, 사용자 프로필, 상품 정보 등 서버에서 가져온 데이터를 관리하는 것은 생각보다 복잡합니다:

- "API 요청은 언제 다시 해야 할까?"
- "로딩 상태와 에러 처리는 어떻게 하지?"
- "캐싱은 어떻게 구현하나?"
- "낙관적 업데이트는 어떻게 하지?"

이런 고민들을 해결하기 위해 등장한 것이 바로 **React Query (TanStack Query)**입니다. 2025년 현재 가장 인기 있는 서버 상태 관리 라이브러리로, 복잡한 데이터 페칭 로직을 선언적이고 간단하게 만들어줍니다.

이번 포스팅에서는 React Query의 모든 것을 다룹니다. 이 글을 읽고 나면 서버 상태 관리를 완벽히 마스터할 수 있을 것입니다.

## 클라이언트 상태 vs 서버 상태

상태 관리를 제대로 하려면 먼저 **상태의 종류**를 구분해야 합니다.

### 클라이언트 상태 (Client State)

애플리케이션이 **완전히 제어**하는 상태입니다:

```jsx
// 클라이언트 상태 예시
const [isModalOpen, setIsModalOpen] = useState(false); // 모달 열림/닫힘
const [selectedTab, setSelectedTab] = useState('home'); // 선택된 탭
const [theme, setTheme] = useState('light'); // 테마
const [formData, setFormData] = useState({}); // 폼 입력 데이터
```

**특징**:
- ✅ 애플리케이션이 완전히 제어
- ✅ 동기적 (Synchronous)
- ✅ 예측 가능한 상태 변화
- ✅ 로컬에만 존재

### 서버 상태 (Server State)

서버에 저장되어 있고, **비동기적으로 가져오는** 상태입니다:

```jsx
// 서버 상태 예시
const [posts, setPosts] = useState([]); // 블로그 포스트 목록
const [user, setUser] = useState(null); // 사용자 프로필
const [products, setProducts] = useState([]); // 상품 목록
```

**특징**:
- 🔄 서버가 소유 (우리는 "스냅샷"만 가짐)
- 🔄 비동기적 (Asynchronous)
- 🔄 언제든 오래될 수 있음 (Stale)
- 🔄 여러 곳에서 같은 데이터 필요
- 🔄 로딩, 에러 처리 필요
- 🔄 캐싱, 동기화, 재시도 필요

### 비교표

| 구분 | 클라이언트 상태 | 서버 상태 |
|------|---------------|----------|
| **소유권** | 클라이언트 완전 제어 | 서버가 소유 |
| **동기화** | 필요 없음 | 서버와 동기화 필요 |
| **캐싱** | 필요 없음 | 필수 |
| **로딩** | 즉시 사용 가능 | 비동기 로딩 필요 |
| **에러** | 거의 없음 | 네트워크 에러 가능 |
| **예시** | 모달 상태, 폼 입력 | API 응답 데이터 |
| **도구** | useState, Zustand | React Query, SWR |

### 왜 구분이 중요한가?

많은 개발자들이 서버 상태를 클라이언트 상태처럼 다루면서 문제를 겪습니다:

```jsx
// ❌ 나쁜 예: 서버 상태를 useState로 관리
function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, []);

  // 문제:
  // 1. 컴포넌트가 다시 마운트되면 재요청 (캐싱 없음)
  // 2. 여러 컴포넌트에서 같은 데이터 요청 시 중복 요청
  // 3. 데이터가 오래되었는지 알 수 없음
  // 4. 백그라운드 업데이트 불가능
  // 5. 페이지네이션, 무한 스크롤 구현 복잡
}
```

서버 상태는 **전용 도구**로 관리해야 합니다. 그것이 바로 React Query입니다.

## React Query 없이 데이터 페칭하기 (문제점)

React Query의 가치를 이해하려면, 먼저 전통적인 방식의 문제점을 봐야 합니다.

### 전통적인 방식: useState + useEffect

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        if (!cancelled) {
          setUser(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true; // 클린업
    };
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

### 문제점

**1. 보일러플레이트 코드가 너무 많음**
- 로딩, 에러, 데이터 상태를 매번 선언
- useEffect 클린업 로직 필수
- 매 컴포넌트마다 반복

**2. 캐싱이 없음**
```jsx
// 같은 userId를 가진 UserProfile이 여러 개 렌더링되면?
<UserProfile userId={1} /> {/* API 요청 1 */}
<UserProfile userId={1} /> {/* API 요청 2 - 중복! */}
<UserProfile userId={1} /> {/* API 요청 3 - 중복! */}
```

**3. 데이터 동기화 어려움**
```jsx
// 한 컴포넌트에서 데이터를 수정하면, 다른 컴포넌트는 어떻게 아나?
function EditProfile() {
  const updateUser = async (newData) => {
    await fetch('/api/users/1', {
      method: 'PUT',
      body: JSON.stringify(newData)
    });
    // ❌ 문제: 다른 컴포넌트의 UserProfile은 옛날 데이터를 보여줌
  };
}
```

**4. 재시도 로직 없음**
- 네트워크 에러 발생 시 수동으로 재시도 구현 필요
- Exponential backoff 같은 전략 구현 복잡

**5. 성능 최적화 어려움**
- 창 포커스 시 데이터 재검증 불가능
- 주기적인 폴링 구현 복잡
- 백그라운드 업데이트 어려움

### Redux + Thunk/Saga로 해결?

Redux로 서버 상태를 관리할 수도 있지만:

```jsx
// Redux로 서버 상태 관리
const postsSlice = createSlice({
  name: 'posts',
  initialState: { data: [], loading: false, error: null },
  reducers: {
    fetchPostsStart: (state) => {
      state.loading = true;
    },
    fetchPostsSuccess: (state, action) => {
      state.loading = false;
      state.data = action.payload;
    },
    fetchPostsFailure: (state, action) => {
      state.loading = false;
      state.error = action.payload;
    }
  }
});

// 여전히 문제:
// 1. 캐싱 로직을 직접 구현해야 함
// 2. 데이터 신선도 관리를 직접 해야 함
// 3. 재시도, 폴링, 무효화 모두 직접 구현
// 4. 보일러플레이트가 여전히 많음
```

Redux는 **클라이언트 상태 관리**에는 훌륭하지만, **서버 상태 관리**에는 과도합니다.

## React Query 소개 및 핵심 개념

### React Query란?

**TanStack Query** (이전 React Query)는 서버 상태 관리를 위한 강력한 라이브러리입니다.

**핵심 철학**:
> "서버 상태는 캐시다. 원본은 서버에 있고, 우리는 그것의 스냅샷을 가지고 있을 뿐이다."

### 주요 특징

**1. 자동 캐싱**
```jsx
// 같은 쿼리 키를 사용하면 자동으로 캐시 공유
<UserProfile userId={1} /> {/* API 요청 1회만 */}
<UserProfile userId={1} /> {/* 캐시에서 가져옴 */}
<UserProfile userId={1} /> {/* 캐시에서 가져옴 */}
```

**2. 자동 재검증**
- 창 포커스 시 자동 재검증
- 네트워크 재연결 시 자동 재검증
- 주기적 폴링 지원

**3. 데이터 동기화**
```jsx
// 한 곳에서 데이터를 수정하면
queryClient.invalidateQueries(['user', 1]);
// 모든 UserProfile 컴포넌트가 자동으로 업데이트됨!
```

**4. 성능 최적화**
- 중복 요청 자동 제거
- 백그라운드 업데이트
- 메모리 관리 (가비지 컬렉션)
- Structural sharing으로 불필요한 리렌더링 방지

**5. 개발자 경험**
- React DevTools 같은 전용 DevTools
- TypeScript 완벽 지원
- 최소한의 보일러플레이트

### 핵심 개념

#### 1. Query (쿼리)

**데이터를 읽는** 모든 작업:

```jsx
// GET 요청 = Query
const { data, isLoading, error } = useQuery({
  queryKey: ['posts'],
  queryFn: () => fetch('/api/posts').then(res => res.json())
});
```

#### 2. Mutation (뮤테이션)

**데이터를 변경하는** 모든 작업:

```jsx
// POST, PUT, DELETE = Mutation
const mutation = useMutation({
  mutationFn: (newPost) =>
    fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(newPost)
    })
});
```

#### 3. Query Key (쿼리 키)

쿼리를 식별하는 **고유한 키**:

```jsx
// 배열 형태의 쿼리 키
['posts']                    // 모든 포스트
['posts', 1]                 // ID가 1인 포스트
['posts', { userId: 1 }]     // userId가 1인 포스트들
['posts', 1, 'comments']     // 포스트 1의 댓글들
```

#### 4. Cache (캐시)

React Query는 모든 쿼리 결과를 **자동으로 캐싱**합니다:

```
[Query Key] -> [Cached Data]
['posts'] -> [{ id: 1, title: '...' }, ...]
['user', 1] -> { id: 1, name: 'John' }
```

#### 5. Query State (쿼리 상태)

각 쿼리는 다음 상태를 가집니다:

```jsx
{
  data,           // 쿼리 결과 데이터
  error,          // 에러 객체
  isLoading,      // 최초 로딩 중
  isFetching,     // 백그라운드 페칭 중
  isError,        // 에러 발생
  isSuccess,      // 성공
  refetch,        // 수동 재요청 함수
  // ...
}
```

**isLoading vs isFetching**:
```jsx
// isLoading: 최초 로딩 (캐시 없음 + 로딩 중)
// isFetching: 백그라운드 로딩 (캐시 있음 + 로딩 중)

// 시나리오 1: 컴포넌트 최초 마운트
isLoading: true, isFetching: true  // 둘 다 true

// 시나리오 2: 캐시된 데이터 보여주면서 백그라운드 업데이트
isLoading: false, isFetching: true  // 캐시 보여주면서 업데이트
```

## 설치 및 기본 설정

### 설치

```bash
# npm
npm install @tanstack/react-query

# yarn
yarn add @tanstack/react-query

# pnpm
pnpm add @tanstack/react-query

# DevTools (선택사항이지만 강력 추천!)
npm install @tanstack/react-query-devtools
```

### 기본 설정

```jsx
// app.jsx 또는 index.jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// QueryClient 생성
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1분
      gcTime: 5 * 60 * 1000, // 5분 (v5에서 cacheTime -> gcTime)
      retry: 1, // 실패 시 1회 재시도
      refetchOnWindowFocus: true, // 창 포커스 시 재검증
      refetchOnReconnect: true, // 네트워크 재연결 시 재검증
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      {/* DevTools: 개발 환경에서만 표시됨 */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

**v5 변경사항**:
- ✅ `cacheTime` → `gcTime` (Garbage Collection Time)
- ✅ `useQuery`의 구조 변경 (객체 파라미터로 통일)

### TypeScript 설정

```tsx
// types.ts
export interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: string;
}

export interface User {
  id: number;
  name: string;
  email: string;
}

// api.ts
export const api = {
  getPosts: async (): Promise<Post[]> => {
    const res = await fetch('/api/posts');
    if (!res.ok) throw new Error('Failed to fetch posts');
    return res.json();
  },

  getPost: async (id: number): Promise<Post> => {
    const res = await fetch(`/api/posts/${id}`);
    if (!res.ok) throw new Error('Failed to fetch post');
    return res.json();
  },
};
```

## useQuery 완벽 가이드

`useQuery`는 React Query의 핵심입니다. **데이터를 가져오는** 모든 작업에 사용합니다.

### 기본 사용법

```jsx
import { useQuery } from '@tanstack/react-query';

function PostList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts'],
    queryFn: async () => {
      const res = await fetch('/api/posts');
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

### 파라미터를 포함한 쿼리

```jsx
function PostDetail({ postId }) {
  const { data: post, isLoading, error } = useQuery({
    queryKey: ['posts', postId], // postId를 키에 포함!
    queryFn: async () => {
      const res = await fetch(`/api/posts/${postId}`);
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
  });

  // postId가 변경되면 자동으로 새로운 쿼리 실행!
}
```

**쿼리 키 규칙**:
```jsx
// ✅ 좋은 예: 계층적 구조
['posts']                     // 모든 포스트
['posts', postId]             // 특정 포스트
['posts', postId, 'comments'] // 특정 포스트의 댓글

// ✅ 좋은 예: 파라미터를 객체로
['posts', { page: 1, limit: 10 }]
['posts', { userId: 1, status: 'published' }]

// ❌ 나쁜 예: 순서가 중요한 키
['posts', 1, 10] // 무엇이 page이고 무엇이 limit인지 불명확
```

### 주요 옵션

#### staleTime (신선도 시간)

데이터가 "신선한" 상태로 유지되는 시간:

```jsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 60 * 1000, // 1분
});

// 동작:
// 0초: API 요청 → 데이터 fresh
// 60초: 데이터 stale
// 61초: 컴포넌트 재렌더링 시 백그라운드 재검증

// staleTime이 길수록:
// ✅ API 요청 횟수 감소 (성능 향상)
// ❌ 데이터가 오래될 수 있음
```

**staleTime 설정 가이드**:
```jsx
// 자주 변하는 데이터 (실시간 주식, 채팅)
staleTime: 0 // 기본값

// 가끔 변하는 데이터 (블로그 포스트, 상품 목록)
staleTime: 60 * 1000 // 1분

// 거의 안 변하는 데이터 (카테고리, 국가 목록)
staleTime: 10 * 60 * 1000 // 10분

// 절대 안 변하는 데이터 (정적 설정)
staleTime: Infinity
```

#### gcTime (가비지 컬렉션 시간)

캐시된 데이터가 메모리에 유지되는 시간:

```jsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  gcTime: 5 * 60 * 1000, // 5분 (기본값)
});

// 동작:
// 1. 모든 컴포넌트가 언마운트됨 (쿼리 사용 중단)
// 2. gcTime 동안 캐시 유지
// 3. gcTime 경과 후 가비지 컬렉션 (메모리에서 제거)
```

**staleTime vs gcTime**:
```jsx
// 예시 시나리오
staleTime: 60 * 1000,  // 1분
gcTime: 5 * 60 * 1000, // 5분

// 타임라인:
// 0:00 - API 요청, 데이터 fresh
// 1:00 - 데이터 stale (백그라운드 재검증 시작 가능)
// 2:00 - 컴포넌트 언마운트 (캐시는 유지)
// 7:00 - gcTime 경과, 캐시 제거

// 6:00에 컴포넌트 재마운트하면?
// → 캐시에서 즉시 보여주고 백그라운드 재검증

// 8:00에 컴포넌트 재마운트하면?
// → 캐시 없음, 새로 API 요청
```

#### enabled (조건부 쿼리)

쿼리 실행을 제어:

```jsx
function UserProfile({ userId }) {
  // userId가 있을 때만 쿼리 실행
  const { data: user } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId, // userId가 없으면 쿼리 실행 안 함
  });

  return user ? <div>{user.name}</div> : null;
}

// 실전 활용: 로그인 여부에 따라
const { data: privateData } = useQuery({
  queryKey: ['private-data'],
  queryFn: fetchPrivateData,
  enabled: isLoggedIn, // 로그인 시에만 실행
});
```

#### refetchOnWindowFocus

창 포커스 시 재검증:

```jsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  refetchOnWindowFocus: true, // 기본값
});

// 사용자 시나리오:
// 1. 사용자가 블로그 목록 페이지 열람
// 2. 다른 탭으로 이동 (10분 후)
// 3. 다시 블로그 탭으로 돌아옴
// → 자동으로 최신 데이터 가져옴!

// 언제 끄나?
// - 데이터가 거의 안 변하는 경우
// - 불필요한 API 요청을 줄이고 싶을 때
refetchOnWindowFocus: false
```

#### refetchInterval (폴링)

주기적으로 데이터 가져오기:

```jsx
// 실시간 데이터 (주식, 암호화폐)
const { data: stockPrice } = useQuery({
  queryKey: ['stock', symbol],
  queryFn: () => fetchStockPrice(symbol),
  refetchInterval: 5000, // 5초마다
});

// 폴링 + 창 포커스 조건
const { data } = useQuery({
  queryKey: ['live-data'],
  queryFn: fetchLiveData,
  refetchInterval: 10000, // 10초마다
  refetchIntervalInBackground: false, // 백그라운드에서는 중지
});

// 사용자가 다른 탭으로 가면 폴링 중지 → 불필요한 요청 방지
```

#### retry (재시도)

실패 시 재시도 전략:

```jsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: 3, // 실패 시 3회 재시도
  retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
  // 1초, 2초, 4초, 8초... (exponential backoff)
});

// 조건부 재시도
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: (failureCount, error) => {
    // 404는 재시도 안 함
    if (error.status === 404) return false;
    // 500 에러는 3회까지 재시도
    return failureCount < 3;
  },
});
```

### useQuery 반환 값

```jsx
const {
  // 데이터 관련
  data,              // 쿼리 결과 데이터
  error,             // 에러 객체

  // 상태 관련
  isLoading,         // 최초 로딩 중 (캐시 없음)
  isFetching,        // 백그라운드 페칭 중
  isError,           // 에러 발생
  isSuccess,         // 성공
  isPending,         // 대기 중 (isLoading과 유사, v5)
  isRefetching,      // 재검증 중

  // 상태 조합
  status,            // 'pending' | 'error' | 'success'
  fetchStatus,       // 'fetching' | 'paused' | 'idle'

  // 함수
  refetch,           // 수동 재요청

  // 메타데이터
  dataUpdatedAt,     // 데이터 마지막 업데이트 시간
  errorUpdatedAt,    // 에러 마지막 업데이트 시간
  failureCount,      // 실패 횟수
  failureReason,     // 실패 이유
} = useQuery({ ... });
```

### 실전 패턴

#### 1. 초기 데이터 제공

```jsx
function PostDetail({ postId, initialData }) {
  const { data: post } = useQuery({
    queryKey: ['posts', postId],
    queryFn: () => fetchPost(postId),
    initialData: initialData, // SSR이나 prefetch된 데이터
    staleTime: 60 * 1000,
  });

  return <div>{post.title}</div>;
}
```

#### 2. Placeholder 데이터

```jsx
const { data: posts = [] } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  placeholderData: [], // 로딩 중에도 빈 배열 사용 가능
});

// map 사용 시 에러 방지
return (
  <ul>
    {posts.map(post => <li key={post.id}>{post.title}</li>)}
  </ul>
);
```

#### 3. Select (데이터 변환)

```jsx
const { data: postTitles } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  select: (data) => data.map(post => post.title), // 제목만 추출
});

// 장점:
// 1. 컴포넌트에서 추가 변환 불필요
// 2. 메모이제이션 자동 적용
// 3. 불필요한 리렌더링 방지
```

#### 4. TypeScript와 함께

```tsx
interface Post {
  id: number;
  title: string;
  content: string;
}

function PostList() {
  const { data, isLoading, error } = useQuery<Post[], Error>({
    queryKey: ['posts'],
    queryFn: async () => {
      const res = await fetch('/api/posts');
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
  });

  // data는 자동으로 Post[] 타입
  // error는 자동으로 Error 타입
}
```

## useMutation 완벽 가이드

`useMutation`은 **데이터를 변경하는** 모든 작업에 사용합니다 (POST, PUT, DELETE).

### 기본 사용법

```jsx
import { useMutation } from '@tanstack/react-query';

function CreatePost() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const mutation = useMutation({
    mutationFn: async (newPost) => {
      const res = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newPost),
      });
      if (!res.ok) throw new Error('Failed to create post');
      return res.json();
    },
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    mutation.mutate({ title, content });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={e => setTitle(e.target.value)} />
      <textarea value={content} onChange={e => setContent(e.target.value)} />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create Post'}
      </button>
      {mutation.isError && <div>Error: {mutation.error.message}</div>}
      {mutation.isSuccess && <div>Post created!</div>}
    </form>
  );
}
```

### useMutation 반환 값

```jsx
const {
  // 함수
  mutate,            // 뮤테이션 실행 (성공/실패 콜백 없음)
  mutateAsync,       // 뮤테이션 실행 (Promise 반환)
  reset,             // 상태 초기화

  // 데이터
  data,              // 뮤테이션 결과 데이터
  error,             // 에러 객체

  // 상태
  isPending,         // 실행 중 (v5에서 isLoading → isPending)
  isError,           // 에러 발생
  isSuccess,         // 성공
  isIdle,            // 아직 실행 안 함
  status,            // 'idle' | 'pending' | 'error' | 'success'

  // 메타
  submittedAt,       // 제출 시간
  variables,         // mutate에 전달된 변수
} = useMutation({ ... });
```

### mutate vs mutateAsync

```jsx
// mutate: 콜백 스타일
mutation.mutate(newPost, {
  onSuccess: (data) => {
    console.log('Success!', data);
  },
  onError: (error) => {
    console.error('Error!', error);
  },
});

// mutateAsync: Promise 스타일
try {
  const data = await mutation.mutateAsync(newPost);
  console.log('Success!', data);
} catch (error) {
  console.error('Error!', error);
}
```

### 라이프사이클 콜백

```jsx
const mutation = useMutation({
  mutationFn: createPost,

  // 1. 뮤테이션 시작 전
  onMutate: async (newPost) => {
    console.log('Mutating...', newPost);
    // 낙관적 업데이트에 사용 (아래에서 자세히)
    return { /* 롤백을 위한 컨텍스트 */ };
  },

  // 2. 성공 시
  onSuccess: (data, variables, context) => {
    console.log('Success!', data);
    // 쿼리 무효화 (자동 재검증)
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },

  // 3. 실패 시
  onError: (error, variables, context) => {
    console.error('Error!', error);
    // 롤백 로직
  },

  // 4. 성공/실패 관계없이 항상 실행
  onSettled: (data, error, variables, context) => {
    console.log('Settled!');
    // 로딩 상태 정리 등
  },
});
```

### 실전 예제

#### 1. 포스트 생성 (POST)

```jsx
function CreatePostForm() {
  const queryClient = useQueryClient();

  const createMutation = useMutation({
    mutationFn: async (newPost) => {
      const res = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newPost),
      });
      if (!res.ok) throw new Error('Failed to create');
      return res.json();
    },
    onSuccess: () => {
      // 포스트 목록 재조회
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      alert('포스트가 생성되었습니다!');
    },
  });

  const handleSubmit = (data) => {
    createMutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 필드 */}
      <button disabled={createMutation.isPending}>
        {createMutation.isPending ? '생성 중...' : '생성'}
      </button>
    </form>
  );
}
```

#### 2. 포스트 수정 (PUT)

```jsx
function EditPostForm({ postId, initialData }) {
  const queryClient = useQueryClient();

  const updateMutation = useMutation({
    mutationFn: async ({ id, data }) => {
      const res = await fetch(`/api/posts/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      if (!res.ok) throw new Error('Failed to update');
      return res.json();
    },
    onSuccess: (data, variables) => {
      // 특정 포스트만 무효화
      queryClient.invalidateQueries({ queryKey: ['posts', variables.id] });
      // 포스트 목록도 무효화
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });

  const handleSubmit = (formData) => {
    updateMutation.mutate({ id: postId, data: formData });
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 폼 필드 */}
    </form>
  );
}
```

#### 3. 포스트 삭제 (DELETE)

```jsx
function DeletePostButton({ postId }) {
  const queryClient = useQueryClient();
  const navigate = useNavigate();

  const deleteMutation = useMutation({
    mutationFn: async (id) => {
      const res = await fetch(`/api/posts/${id}`, {
        method: 'DELETE',
      });
      if (!res.ok) throw new Error('Failed to delete');
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      navigate('/posts'); // 목록 페이지로 이동
    },
  });

  const handleDelete = () => {
    if (confirm('정말 삭제하시겠습니까?')) {
      deleteMutation.mutate(postId);
    }
  };

  return (
    <button onClick={handleDelete} disabled={deleteMutation.isPending}>
      {deleteMutation.isPending ? '삭제 중...' : '삭제'}
    </button>
  );
}
```

#### 4. 여러 뮤테이션 조합

```jsx
function PostEditor({ postId }) {
  const queryClient = useQueryClient();

  // 수정 뮤테이션
  const updateMutation = useMutation({
    mutationFn: updatePost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts', postId] });
    },
  });

  // 삭제 뮤테이션
  const deleteMutation = useMutation({
    mutationFn: deletePost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      navigate('/posts');
    },
  });

  // 발행 뮤테이션
  const publishMutation = useMutation({
    mutationFn: publishPost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts', postId] });
    },
  });

  return (
    <div>
      <button onClick={() => updateMutation.mutate(data)}>수정</button>
      <button onClick={() => deleteMutation.mutate(postId)}>삭제</button>
      <button onClick={() => publishMutation.mutate(postId)}>발행</button>
    </div>
  );
}
```

## 캐싱 전략

React Query의 캐싱 시스템은 매우 정교합니다. 올바르게 이해하면 성능을 크게 향상시킬 수 있습니다.

### 캐시 라이프사이클

```
1. Fresh (신선함)
   ↓ staleTime 경과
2. Stale (오래됨)
   ↓ 모든 컴포넌트 언마운트
3. Inactive (비활성)
   ↓ gcTime 경과
4. Garbage Collected (삭제)
```

### staleTime 전략

**staleTime**은 데이터가 얼마나 "신선한" 상태로 유지될지 결정합니다.

```jsx
// 예시 1: 실시간 데이터 (주식 가격)
const { data } = useQuery({
  queryKey: ['stock', symbol],
  queryFn: () => fetchStockPrice(symbol),
  staleTime: 0, // 항상 stale, 즉시 재검증
  refetchInterval: 5000, // 5초마다 자동 갱신
});

// 예시 2: 일반 데이터 (블로그 포스트)
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 5 * 60 * 1000, // 5분간 신선
});

// 예시 3: 정적 데이터 (국가 목록)
const { data } = useQuery({
  queryKey: ['countries'],
  queryFn: fetchCountries,
  staleTime: Infinity, // 절대 stale 안 됨
});
```

**시나리오별 staleTime**:

| 데이터 유형 | staleTime | 이유 |
|-----------|----------|-----|
| 실시간 데이터 (주식, 채팅) | 0ms | 항상 최신 데이터 필요 |
| 자주 변하는 데이터 (뉴스 피드) | 30초 - 1분 | 적절한 신선도와 성능 균형 |
| 일반 데이터 (블로그, 상품) | 5분 - 10분 | 대부분의 경우 적절 |
| 거의 안 변하는 데이터 (설정) | 30분 - 1시간 | 불필요한 요청 최소화 |
| 정적 데이터 (국가, 언어) | Infinity | API 요청 완전 제거 |

### gcTime 전략

**gcTime** (Garbage Collection Time)은 캐시가 메모리에 얼마나 유지될지 결정합니다.

```jsx
// 예시 1: 자주 재방문하는 페이지
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  gcTime: 30 * 60 * 1000, // 30분
});
// 사용자가 다른 페이지 갔다가 돌아와도 캐시 유지

// 예시 2: 일회성 페이지
const { data } = useQuery({
  queryKey: ['report', id],
  queryFn: () => fetchReport(id),
  gcTime: 0, // 언마운트 즉시 캐시 제거
});
// 메모리 절약

// 예시 3: 기본값 (대부분의 경우 적절)
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  gcTime: 5 * 60 * 1000, // 5분 (기본값)
});
```

### refetchOnWindowFocus 전략

창 포커스 시 자동 재검증 여부:

```jsx
// 예시 1: 중요한 실시간 데이터
const { data: notifications } = useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  refetchOnWindowFocus: true, // 탭 전환 시 재검증 (기본값)
});
// 사용자가 다른 탭 갔다가 돌아오면 최신 알림 확인

// 예시 2: 정적 데이터
const { data: countries } = useQuery({
  queryKey: ['countries'],
  queryFn: fetchCountries,
  refetchOnWindowFocus: false, // 불필요한 요청 방지
  staleTime: Infinity,
});

// 예시 3: 조건부 재검증
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  refetchOnWindowFocus: 'always', // 항상 재검증
  // 또는 false, true
});
```

### 전역 설정 vs 개별 설정

```jsx
// app.jsx - 전역 기본값
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,      // 1분 (일반적인 경우)
      gcTime: 5 * 60 * 1000,     // 5분
      retry: 1,
      refetchOnWindowFocus: true,
    },
  },
});

// 개별 쿼리에서 오버라이드
const { data } = useQuery({
  queryKey: ['realtime-data'],
  queryFn: fetchRealtimeData,
  staleTime: 0, // 전역 설정 무시, 0으로 설정
  refetchInterval: 5000, // 추가 설정
});
```

### 캐싱 시각화

```jsx
// 타임라인 예시
// staleTime: 2분, gcTime: 5분

// 0:00 - 컴포넌트 마운트, API 요청
// 0:01 - 데이터 도착, fresh 상태
// 2:00 - staleTime 경과, stale 상태
// 2:30 - 컴포넌트 재렌더링 → 백그라운드 재검증 시작
// 2:31 - 새 데이터 도착, 다시 fresh 상태
// 3:00 - 컴포넌트 언마운트 (inactive 상태 시작)
// 8:00 - gcTime 경과 (3:00 + 5분), 캐시 제거
```

## 페이지네이션 구현

페이지네이션은 대량의 데이터를 나눠서 보여주는 일반적인 패턴입니다.

### 기본 페이지네이션

```jsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

function PostListPaginated() {
  const [page, setPage] = useState(1);
  const limit = 10;

  const { data, isLoading, isFetching, isError, error } = useQuery({
    queryKey: ['posts', { page, limit }], // page를 키에 포함!
    queryFn: async () => {
      const res = await fetch(`/api/posts?page=${page}&limit=${limit}`);
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
    placeholderData: (previousData) => previousData, // 이전 데이터 유지
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      <ul>
        {data.posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>

      {/* 페이지네이션 UI */}
      <div>
        <button
          onClick={() => setPage(prev => Math.max(prev - 1, 1))}
          disabled={page === 1}
        >
          이전
        </button>
        <span>페이지 {page} / {data.totalPages}</span>
        <button
          onClick={() => setPage(prev => prev + 1)}
          disabled={page >= data.totalPages}
        >
          다음
        </button>
      </div>

      {/* 백그라운드 페칭 인디케이터 */}
      {isFetching && <div>Updating...</div>}
    </div>
  );
}
```

### placeholderData를 활용한 부드러운 전환

```jsx
const { data, isLoading } = useQuery({
  queryKey: ['posts', page],
  queryFn: () => fetchPosts(page),
  placeholderData: (previousData) => previousData, // 중요!
});

// placeholderData 덕분에:
// 1. 페이지 전환 시 이전 데이터를 계속 보여줌
// 2. 백그라운드에서 새 데이터 로딩
// 3. 새 데이터 도착하면 부드럽게 전환
// → 사용자 경험 크게 향상!
```

### Prefetching (미리 가져오기)

다음 페이지를 미리 가져와 즉시 표시:

```jsx
import { useQuery, useQueryClient } from '@tanstack/react-query';

function PostListPaginated() {
  const queryClient = useQueryClient();
  const [page, setPage] = useState(1);

  const { data } = useQuery({
    queryKey: ['posts', page],
    queryFn: () => fetchPosts(page),
  });

  // 다음 페이지 미리 가져오기
  useEffect(() => {
    if (page < data?.totalPages) {
      queryClient.prefetchQuery({
        queryKey: ['posts', page + 1],
        queryFn: () => fetchPosts(page + 1),
      });
    }
  }, [page, data, queryClient]);

  // 사용자가 "다음" 클릭하면 이미 캐시에 있어서 즉시 표시!
}
```

### 커서 기반 페이지네이션

```jsx
// API: /api/posts?cursor=abc123&limit=10
function PostListCursor() {
  const [cursor, setCursor] = useState(null);

  const { data, isLoading } = useQuery({
    queryKey: ['posts', { cursor }],
    queryFn: async () => {
      const params = new URLSearchParams();
      if (cursor) params.append('cursor', cursor);
      params.append('limit', '10');

      const res = await fetch(`/api/posts?${params}`);
      return res.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <ul>
        {data.posts.map(post => <li key={post.id}>{post.title}</li>)}
      </ul>

      {data.nextCursor && (
        <button onClick={() => setCursor(data.nextCursor)}>
          더 보기
        </button>
      )}
    </div>
  );
}
```

## 무한 스크롤 구현 (useInfiniteQuery)

무한 스크롤은 SNS 피드, 검색 결과 등에서 흔히 사용됩니다.

### 기본 사용법

```jsx
import { useInfiniteQuery } from '@tanstack/react-query';

function InfinitePostList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
    error,
  } = useInfiniteQuery({
    queryKey: ['posts', 'infinite'],
    queryFn: async ({ pageParam = 1 }) => {
      const res = await fetch(`/api/posts?page=${pageParam}&limit=10`);
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
    getNextPageParam: (lastPage, allPages) => {
      // 다음 페이지 번호 계산
      return lastPage.hasMore ? allPages.length + 1 : undefined;
    },
    initialPageParam: 1, // v5에서 필수!
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      {/* 모든 페이지의 데이터 렌더링 */}
      {data.pages.map((page, pageIndex) => (
        <div key={pageIndex}>
          {page.posts.map(post => (
            <div key={post.id}>
              <h3>{post.title}</h3>
              <p>{post.excerpt}</p>
            </div>
          ))}
        </div>
      ))}

      {/* 더 보기 버튼 */}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Loading...'
          : hasNextPage
          ? '더 보기'
          : '마지막 페이지입니다'}
      </button>
    </div>
  );
}
```

### useInfiniteQuery 데이터 구조

```jsx
{
  data: {
    pages: [
      { posts: [...], hasMore: true },  // 페이지 1
      { posts: [...], hasMore: true },  // 페이지 2
      { posts: [...], hasMore: false }, // 페이지 3
    ],
    pageParams: [1, 2, 3], // 각 페이지의 파라미터
  },
  hasNextPage: false,
  hasPreviousPage: false,
  fetchNextPage: () => {},
  fetchPreviousPage: () => {},
  isFetchingNextPage: false,
  isFetchingPreviousPage: false,
}
```

### Intersection Observer로 자동 로딩

```jsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';
import { useEffect } from 'react';

function InfinitePostList() {
  const { ref, inView } = useInView();

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['posts', 'infinite'],
    queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
    initialPageParam: 1,
  });

  // 스크롤이 하단에 도달하면 자동으로 다음 페이지 로딩
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.posts.map(post => (
            <div key={post.id}>{post.title}</div>
          ))}
        </div>
      ))}

      {/* 로딩 트리거 */}
      <div ref={ref}>
        {isFetchingNextPage && <div>Loading more...</div>}
      </div>
    </div>
  );
}
```

### 커서 기반 무한 스크롤

```jsx
const {
  data,
  fetchNextPage,
  hasNextPage,
} = useInfiniteQuery({
  queryKey: ['posts', 'infinite'],
  queryFn: async ({ pageParam = null }) => {
    const params = new URLSearchParams();
    if (pageParam) params.append('cursor', pageParam);
    params.append('limit', '20');

    const res = await fetch(`/api/posts?${params}`);
    return res.json();
  },
  getNextPageParam: (lastPage) => {
    // 다음 커서 반환 (없으면 undefined)
    return lastPage.nextCursor ?? undefined;
  },
  initialPageParam: null,
});

// API 응답 형식:
// {
//   posts: [...],
//   nextCursor: 'eyJpZCI6MTIzfQ==', // Base64 encoded cursor
// }
```

### 양방향 무한 스크롤

```jsx
const {
  data,
  fetchNextPage,
  fetchPreviousPage,
  hasNextPage,
  hasPreviousPage,
} = useInfiniteQuery({
  queryKey: ['posts', 'infinite'],
  queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
  getNextPageParam: (lastPage, pages) => {
    return lastPage.hasMore ? pages.length + 1 : undefined;
  },
  getPreviousPageParam: (firstPage, pages) => {
    return pages[0] > 1 ? pages[0] - 1 : undefined;
  },
  initialPageParam: 1,
  maxPages: 5, // 최대 5페이지만 메모리에 유지 (메모리 절약)
});

return (
  <div>
    {hasPreviousPage && (
      <button onClick={() => fetchPreviousPage()}>
        이전 페이지 로드
      </button>
    )}

    {data?.pages.map(page => (
      // 렌더링
    ))}

    {hasNextPage && (
      <button onClick={() => fetchNextPage()}>
        다음 페이지 로드
      </button>
    )}
  </div>
);
```

### 실전 SNS 피드 예제

```jsx
function SocialFeed() {
  const { ref, inView } = useInView();

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    refetch,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: async ({ pageParam = null }) => {
      const res = await fetch(
        `/api/feed?cursor=${pageParam || ''}&limit=20`
      );
      return res.json();
    },
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    initialPageParam: null,
    staleTime: 60 * 1000, // 1분
    refetchOnWindowFocus: true, // 탭 전환 시 새 게시물 확인
  });

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) return <FeedSkeleton />;

  return (
    <div>
      {/* Pull to refresh */}
      <button onClick={() => refetch()}>새 게시물 확인</button>

      {/* Feed items */}
      {data.pages.map((page, i) => (
        <div key={i}>
          {page.items.map(item => (
            <FeedItem key={item.id} item={item} />
          ))}
        </div>
      ))}

      {/* Loading trigger */}
      <div ref={ref} style={{ height: 20 }}>
        {isFetchingNextPage && <Spinner />}
      </div>

      {!hasNextPage && <div>모든 게시물을 확인했습니다</div>}
    </div>
  );
}
```

## 낙관적 업데이트 (Optimistic Updates)

낙관적 업데이트는 서버 응답을 기다리지 않고 **즉시 UI를 업데이트**하는 기법입니다. 사용자 경험을 크게 향상시킵니다.

### 개념

```text
// 일반적인 방식 (비관적)
좋아요 클릭 → API 요청 → 응답 대기 (1-2초) → UI 업데이트
// 사용자는 1-2초간 기다려야 함

// 낙관적 방식
좋아요 클릭 → 즉시 UI 업데이트 → API 요청 → 실패 시 롤백
// 사용자는 즉시 피드백을 받음!
```

### 기본 낙관적 업데이트

```jsx
function LikeButton({ postId, initialLikes }) {
  const queryClient = useQueryClient();

  const likeMutation = useMutation({
    mutationFn: async (postId) => {
      const res = await fetch(`/api/posts/${postId}/like`, {
        method: 'POST',
      });
      if (!res.ok) throw new Error('Failed to like');
      return res.json();
    },

    // 1. 뮤테이션 시작 전 (낙관적 업데이트)
    onMutate: async (postId) => {
      // 진행 중인 쿼리 취소 (충돌 방지)
      await queryClient.cancelQueries({ queryKey: ['posts', postId] });

      // 현재 데이터 백업 (롤백용)
      const previousPost = queryClient.getQueryData(['posts', postId]);

      // 낙관적 업데이트 (즉시 UI 변경)
      queryClient.setQueryData(['posts', postId], (old) => ({
        ...old,
        likes: old.likes + 1,
        isLiked: true,
      }));

      // 컨텍스트 반환 (onError에서 사용)
      return { previousPost };
    },

    // 2. 에러 발생 시 (롤백)
    onError: (error, postId, context) => {
      // 이전 데이터로 복구
      queryClient.setQueryData(
        ['posts', postId],
        context.previousPost
      );
      alert('좋아요 실패: ' + error.message);
    },

    // 3. 성공 시
    onSuccess: (data, postId) => {
      // 서버 데이터로 최종 업데이트
      queryClient.setQueryData(['posts', postId], data);
    },

    // 4. 성공/실패 관계없이 (정리)
    onSettled: (data, error, postId) => {
      // 쿼리 무효화 (서버 데이터와 동기화)
      queryClient.invalidateQueries({ queryKey: ['posts', postId] });
    },
  });

  return (
    <button onClick={() => likeMutation.mutate(postId)}>
      ❤️ {initialLikes}
    </button>
  );
}
```

### 낙관적 업데이트 패턴 분석

```jsx
// 핵심 패턴
onMutate: async (variables) => {
  // 1. 진행 중인 refetch 취소
  await queryClient.cancelQueries({ queryKey: ['key'] });

  // 2. 현재 데이터 스냅샷 저장
  const previous = queryClient.getQueryData(['key']);

  // 3. 낙관적으로 캐시 업데이트
  queryClient.setQueryData(['key'], (old) => {
    // 즉시 변경
  });

  // 4. 롤백을 위한 컨텍스트 반환
  return { previous };
},

onError: (err, variables, context) => {
  // 에러 시 롤백
  queryClient.setQueryData(['key'], context.previous);
},

onSettled: () => {
  // 서버와 재동기화
  queryClient.invalidateQueries({ queryKey: ['key'] });
},
```

### 실전 예제 1: 장바구니

```jsx
function ProductCard({ product }) {
  const queryClient = useQueryClient();

  const addToCartMutation = useMutation({
    mutationFn: async (productId) => {
      const res = await fetch('/api/cart', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ productId, quantity: 1 }),
      });
      if (!res.ok) throw new Error('Failed to add to cart');
      return res.json();
    },

    onMutate: async (productId) => {
      await queryClient.cancelQueries({ queryKey: ['cart'] });

      const previousCart = queryClient.getQueryData(['cart']);

      // 낙관적 업데이트: 장바구니에 즉시 추가
      queryClient.setQueryData(['cart'], (old) => ({
        ...old,
        items: [
          ...old.items,
          { productId, quantity: 1, product },
        ],
        totalCount: old.totalCount + 1,
      }));

      return { previousCart };
    },

    onError: (error, productId, context) => {
      queryClient.setQueryData(['cart'], context.previousCart);
      alert('장바구니 추가 실패');
    },

    onSuccess: () => {
      // 성공 시 토스트 메시지
      toast.success('장바구니에 추가되었습니다');
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  return (
    <button
      onClick={() => addToCartMutation.mutate(product.id)}
      disabled={addToCartMutation.isPending}
    >
      장바구니 담기
    </button>
  );
}
```

### 실전 예제 2: 댓글 작성

```jsx
function CommentForm({ postId }) {
  const queryClient = useQueryClient();
  const [comment, setComment] = useState('');

  const addCommentMutation = useMutation({
    mutationFn: async ({ postId, content }) => {
      const res = await fetch(`/api/posts/${postId}/comments`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ content }),
      });
      if (!res.ok) throw new Error('Failed to add comment');
      return res.json();
    },

    onMutate: async ({ postId, content }) => {
      await queryClient.cancelQueries({
        queryKey: ['posts', postId, 'comments']
      });

      const previousComments = queryClient.getQueryData([
        'posts',
        postId,
        'comments',
      ]);

      // 낙관적 댓글 추가 (임시 ID 사용)
      const optimisticComment = {
        id: `temp-${Date.now()}`,
        content,
        author: { name: '나', avatar: '/my-avatar.jpg' },
        createdAt: new Date().toISOString(),
        isOptimistic: true, // 낙관적 업데이트 표시
      };

      queryClient.setQueryData(
        ['posts', postId, 'comments'],
        (old) => [...old, optimisticComment]
      );

      return { previousComments };
    },

    onError: (error, variables, context) => {
      queryClient.setQueryData(
        ['posts', variables.postId, 'comments'],
        context.previousComments
      );
      alert('댓글 작성 실패');
    },

    onSuccess: (newComment, variables) => {
      // 임시 댓글을 실제 댓글로 교체
      queryClient.setQueryData(
        ['posts', variables.postId, 'comments'],
        (old) =>
          old.map((comment) =>
            comment.id.startsWith('temp-') ? newComment : comment
          )
      );
      setComment(''); // 폼 초기화
    },
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!comment.trim()) return;
    addCommentMutation.mutate({ postId, content: comment });
  };

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        placeholder="댓글을 입력하세요"
      />
      <button type="submit">작성</button>
    </form>
  );
}
```

### 실전 예제 3: 할 일 목록 (Todo)

```jsx
function TodoList() {
  const queryClient = useQueryClient();

  // 할 일 토글 (완료/미완료)
  const toggleMutation = useMutation({
    mutationFn: async ({ id, completed }) => {
      const res = await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed }),
      });
      if (!res.ok) throw new Error('Failed to toggle');
      return res.json();
    },

    onMutate: async ({ id, completed }) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      const previousTodos = queryClient.getQueryData(['todos']);

      // 낙관적 업데이트
      queryClient.setQueryData(['todos'], (old) =>
        old.map((todo) =>
          todo.id === id ? { ...todo, completed } : todo
        )
      );

      return { previousTodos };
    },

    onError: (error, variables, context) => {
      queryClient.setQueryData(['todos'], context.previousTodos);
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  // 할 일 삭제
  const deleteMutation = useMutation({
    mutationFn: async (id) => {
      const res = await fetch(`/api/todos/${id}`, { method: 'DELETE' });
      if (!res.ok) throw new Error('Failed to delete');
    },

    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      const previousTodos = queryClient.getQueryData(['todos']);

      // 낙관적 삭제
      queryClient.setQueryData(['todos'], (old) =>
        old.filter((todo) => todo.id !== id)
      );

      return { previousTodos };
    },

    onError: (error, id, context) => {
      queryClient.setQueryData(['todos'], context.previousTodos);
      alert('삭제 실패');
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  const { data: todos } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });

  return (
    <ul>
      {todos?.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={(e) =>
              toggleMutation.mutate({
                id: todo.id,
                completed: e.target.checked,
              })
            }
          />
          <span>{todo.title}</span>
          <button onClick={() => deleteMutation.mutate(todo.id)}>
            삭제
          </button>
        </li>
      ))}
    </ul>
  );
}
```

## Query Invalidation (쿼리 무효화)

쿼리 무효화는 캐시된 데이터를 "stale"로 만들어 재검증을 유도합니다.

### 기본 무효화

```jsx
import { useQueryClient } from '@tanstack/react-query';

function CreatePost() {
  const queryClient = useQueryClient();

  const createMutation = useMutation({
    mutationFn: createPost,
    onSuccess: () => {
      // 포스트 목록 무효화 → 자동 재검증
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });

  // 사용자가 포스트를 생성하면:
  // 1. 뮤테이션 성공
  // 2. ['posts'] 쿼리가 stale로 표시
  // 3. 해당 쿼리를 사용하는 컴포넌트가 자동으로 재검증
}
```

### 정확한 무효화 vs 부분 무효화

```jsx
// 정확한 무효화: 특정 쿼리만
queryClient.invalidateQueries({ queryKey: ['posts', 1] });
// ['posts', 1]만 무효화

// 부분 무효화: 접두사 매칭
queryClient.invalidateQueries({ queryKey: ['posts'] });
// ['posts'], ['posts', 1], ['posts', 2], ... 모두 무효화

// 예시
queryClient.setQueryData(['posts'], [...]);
queryClient.setQueryData(['posts', 1], {...});
queryClient.setQueryData(['posts', 2], {...});
queryClient.setQueryData(['users'], [...]);

queryClient.invalidateQueries({ queryKey: ['posts'] });
// → ['posts'], ['posts', 1], ['posts', 2] 무효화
// → ['users']는 영향 없음
```

### 무효화 옵션

```jsx
// 1. 즉시 재검증
queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'active', // 현재 활성화된 쿼리만 재검증 (기본값)
});

queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'all', // 모든 쿼리 재검증 (비활성 쿼리 포함)
});

queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'none', // stale로만 표시, 재검증은 안 함
});

// 2. 특정 조건의 쿼리만
queryClient.invalidateQueries({
  queryKey: ['posts'],
  predicate: (query) => {
    // 마지막 업데이트가 10분 이상 된 쿼리만 무효화
    return Date.now() - query.dataUpdatedAt > 10 * 60 * 1000;
  },
});
```

### 실전 무효화 패턴

#### 1. 생성 후 목록 무효화

```jsx
const createMutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    // 포스트 목록 재조회
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});
```

#### 2. 수정 후 상세 & 목록 무효화

```jsx
const updateMutation = useMutation({
  mutationFn: ({ id, data }) => updatePost(id, data),
  onSuccess: (data, variables) => {
    // 특정 포스트 무효화
    queryClient.invalidateQueries({
      queryKey: ['posts', variables.id],
    });
    // 포스트 목록도 무효화 (제목이 바뀌었을 수 있음)
    queryClient.invalidateQueries({
      queryKey: ['posts'],
    });
  },
});
```

#### 3. 삭제 후 목록 무효화

```jsx
const deleteMutation = useMutation({
  mutationFn: deletePost,
  onSuccess: (data, postId) => {
    // 삭제된 포스트 제거
    queryClient.removeQueries({ queryKey: ['posts', postId] });
    // 목록 재조회
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});
```

#### 4. 관련 쿼리 모두 무효화

```jsx
const likeMutation = useMutation({
  mutationFn: ({ postId }) => likePost(postId),
  onSuccess: (data, variables) => {
    // 포스트 상세
    queryClient.invalidateQueries({
      queryKey: ['posts', variables.postId],
    });
    // 포스트 목록
    queryClient.invalidateQueries({
      queryKey: ['posts'],
    });
    // 사용자 좋아요 목록
    queryClient.invalidateQueries({
      queryKey: ['user', 'likes'],
    });
  },
});
```

### setQueryData vs invalidateQueries

```jsx
// setQueryData: 캐시를 직접 업데이트
queryClient.setQueryData(['posts', 1], (old) => ({
  ...old,
  likes: old.likes + 1,
}));
// 장점: 즉시 업데이트, API 요청 없음
// 단점: 서버 데이터와 불일치 가능

// invalidateQueries: 재검증 유도
queryClient.invalidateQueries({ queryKey: ['posts', 1] });
// 장점: 서버와 동기화 보장
// 단점: API 요청 필요

// 조합 (Best Practice)
useMutation({
  mutationFn: likePost,
  onMutate: async (postId) => {
    // 즉시 UI 업데이트 (낙관적)
    queryClient.setQueryData(['posts', postId], (old) => ({
      ...old,
      likes: old.likes + 1,
    }));
  },
  onSettled: (data, error, postId) => {
    // 서버와 재동기화 (최종 확인)
    queryClient.invalidateQueries({ queryKey: ['posts', postId] });
  },
});
```

## Dependent Queries & Parallel Queries

### Dependent Queries (종속 쿼리)

한 쿼리의 결과가 다른 쿼리의 조건이 되는 경우:

```jsx
function UserPosts({ userId }) {
  // 1. 사용자 정보 먼저 가져오기
  const { data: user } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
  });

  // 2. 사용자 정보를 사용하여 포스트 가져오기
  const { data: posts } = useQuery({
    queryKey: ['posts', { authorId: user?.id }],
    queryFn: () => fetchPostsByAuthor(user.id),
    enabled: !!user, // user가 있을 때만 실행!
  });

  if (!user) return <div>Loading user...</div>;
  if (!posts) return <div>Loading posts...</div>;

  return (
    <div>
      <h1>{user.name}의 포스트</h1>
      <ul>
        {posts.map(post => <li key={post.id}>{post.title}</li>)}
      </ul>
    </div>
  );
}
```

**실전 패턴**:
```jsx
function PostDetail({ postId }) {
  // 포스트 정보
  const { data: post } = useQuery({
    queryKey: ['posts', postId],
    queryFn: () => fetchPost(postId),
  });

  // 포스트 작성자 정보 (포스트에 authorId가 있어야 조회 가능)
  const { data: author } = useQuery({
    queryKey: ['users', post?.authorId],
    queryFn: () => fetchUser(post.authorId),
    enabled: !!post?.authorId,
  });

  // 댓글 (포스트가 있어야 조회 가능)
  const { data: comments } = useQuery({
    queryKey: ['posts', postId, 'comments'],
    queryFn: () => fetchComments(postId),
    enabled: !!post,
  });

  // 로딩 처리
  if (!post) return <div>Loading...</div>;

  return (
    <article>
      <h1>{post.title}</h1>
      {author && <p>By {author.name}</p>}
      <div>{post.content}</div>
      {comments && <CommentList comments={comments} />}
    </article>
  );
}
```

### Parallel Queries (병렬 쿼리)

여러 쿼리를 동시에 실행:

```jsx
function Dashboard() {
  // 세 쿼리가 동시에 실행됨!
  const { data: stats } = useQuery({
    queryKey: ['stats'],
    queryFn: fetchStats,
  });

  const { data: recentPosts } = useQuery({
    queryKey: ['posts', 'recent'],
    queryFn: fetchRecentPosts,
  });

  const { data: notifications } = useQuery({
    queryKey: ['notifications'],
    queryFn: fetchNotifications,
  });

  // 병렬 실행으로 로딩 시간 단축!
  // 순차 실행: 1초 + 1초 + 1초 = 3초
  // 병렬 실행: max(1초, 1초, 1초) = 1초
}
```

### useQueries (동적 병렬 쿼리)

```jsx
import { useQueries } from '@tanstack/react-query';

function MultipleUsers({ userIds }) {
  // userIds 배열의 각 ID에 대해 쿼리 실행
  const userQueries = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['users', id],
      queryFn: () => fetchUser(id),
      staleTime: 60 * 1000,
    })),
  });

  // 모든 쿼리의 상태 확인
  const isLoading = userQueries.some((q) => q.isLoading);
  const isError = userQueries.some((q) => q.isError);

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error loading users</div>;

  return (
    <ul>
      {userQueries.map((query, i) => (
        <li key={userIds[i]}>{query.data?.name}</li>
      ))}
    </ul>
  );
}
```

**실전 활용: 대시보드**
```jsx
function AdminDashboard() {
  const queries = useQueries({
    queries: [
      {
        queryKey: ['stats', 'users'],
        queryFn: fetchUserStats,
      },
      {
        queryKey: ['stats', 'posts'],
        queryFn: fetchPostStats,
      },
      {
        queryKey: ['stats', 'revenue'],
        queryFn: fetchRevenueStats,
      },
      {
        queryKey: ['recent-activity'],
        queryFn: fetchRecentActivity,
      },
    ],
  });

  // 모든 데이터 추출
  const [userStats, postStats, revenueStats, recentActivity] =
    queries.map((q) => q.data);

  // 로딩 상태
  const isLoading = queries.some((q) => q.isLoading);

  if (isLoading) return <DashboardSkeleton />;

  return (
    <div>
      <StatCard title="Users" data={userStats} />
      <StatCard title="Posts" data={postStats} />
      <StatCard title="Revenue" data={revenueStats} />
      <ActivityFeed activities={recentActivity} />
    </div>
  );
}
```

## 에러 처리 및 재시도 전략

### 기본 에러 처리

```jsx
function PostList() {
  const { data, isError, error } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  if (isError) {
    return (
      <div>
        <h2>에러가 발생했습니다</h2>
        <p>{error.message}</p>
      </div>
    );
  }

  return <ul>{/* 렌더링 */}</ul>;
}
```

### 커스텀 에러 클래스

```jsx
// api.js
class ApiError extends Error {
  constructor(message, status, data) {
    super(message);
    this.status = status;
    this.data = data;
  }
}

export async function fetchPosts() {
  const res = await fetch('/api/posts');

  if (!res.ok) {
    const error = await res.json();
    throw new ApiError(
      error.message || 'Failed to fetch',
      res.status,
      error
    );
  }

  return res.json();
}

// component.jsx
function PostList() {
  const { data, isError, error } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  if (isError) {
    // 상태 코드별 처리
    if (error.status === 404) {
      return <div>포스트를 찾을 수 없습니다</div>;
    }
    if (error.status === 401) {
      return <div>로그인이 필요합니다</div>;
    }
    if (error.status >= 500) {
      return <div>서버 오류가 발생했습니다</div>;
    }
    return <div>오류: {error.message}</div>;
  }
}
```

### Error Boundary와 통합

```jsx
// ErrorBoundary.jsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <div>
              <h2>에러가 발생했습니다</h2>
              <p>{error.message}</p>
              <button onClick={resetErrorBoundary}>다시 시도</button>
            </div>
          )}
        >
          <YourApp />
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

// 컴포넌트에서 에러 던지기
function PostDetail({ postId }) {
  const { data } = useQuery({
    queryKey: ['posts', postId],
    queryFn: () => fetchPost(postId),
    useErrorBoundary: true, // 에러를 ErrorBoundary로 전파
  });

  // 에러 발생 시 ErrorBoundary가 처리
  return <div>{data.title}</div>;
}
```

### 재시도 전략

```jsx
// 기본 재시도 (3회)
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: 3, // 실패 시 최대 3회 재시도
});

// 조건부 재시도
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: (failureCount, error) => {
    // 404는 재시도 안 함
    if (error.status === 404) return false;

    // 500대 에러는 3회까지 재시도
    if (error.status >= 500 && failureCount < 3) return true;

    // 그 외는 재시도 안 함
    return false;
  },
});

// Exponential backoff (지수 백오프)
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: 3,
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  // 1초, 2초, 4초, 8초, ... (최대 30초)
});

// 재시도 시 콜백
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: 3,
  onError: (error) => {
    console.error('Query failed:', error);
    // 로깅, 알림 등
  },
});
```

### 글로벌 에러 처리

```jsx
// app.jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,
      onError: (error) => {
        // 모든 쿼리 에러를 여기서 처리
        console.error('Query error:', error);

        if (error.status === 401) {
          // 로그아웃 처리
          logout();
        }

        // 에러 로깅 서비스 (Sentry 등)
        logErrorToService(error);
      },
    },
    mutations: {
      onError: (error) => {
        // 모든 뮤테이션 에러 처리
        toast.error(error.message);
        logErrorToService(error);
      },
    },
  },
});
```

### 수동 재시도

```jsx
function PostList() {
  const { data, isError, error, refetch, isRefetching } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  if (isError) {
    return (
      <div>
        <p>에러: {error.message}</p>
        <button onClick={() => refetch()} disabled={isRefetching}>
          {isRefetching ? '재시도 중...' : '다시 시도'}
        </button>
      </div>
    );
  }

  return <ul>{/* 렌더링 */}</ul>;
}
```

## DevTools 활용법

React Query DevTools는 개발을 크게 도와주는 강력한 도구입니다.

### 설치 및 설정

```jsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      {/* 개발 환경에서만 표시됨 */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### 주요 기능

**1. 쿼리 목록 확인**
- 현재 캐시된 모든 쿼리 표시
- 각 쿼리의 상태 (fresh, stale, inactive 등)
- 데이터 크기, 마지막 업데이트 시간

**2. 쿼리 상세 정보**
```jsx
// DevTools에서 확인 가능한 정보
{
  queryKey: ['posts', 1],
  status: 'success',
  fetchStatus: 'idle',
  data: { ... },
  dataUpdatedAt: 1640000000000,
  error: null,
  errorUpdatedAt: 0,
  isFetching: false,
  isStale: false,
  observerCount: 2, // 이 쿼리를 사용하는 컴포넌트 수
}
```

**3. 수동 액션**
- Refetch: 쿼리 재실행
- Invalidate: 쿼리 무효화
- Reset: 쿼리 상태 초기화
- Remove: 캐시에서 제거

**4. 뮤테이션 추적**
- 진행 중인 뮤테이션 확인
- 뮤테이션 변수, 결과, 에러 확인

### DevTools 활용 팁

**디버깅 시나리오 1: 쿼리가 실행 안 됨**
```jsx
// DevTools에서 확인:
// - enabled: false가 설정되어 있나?
// - queryKey가 올바른가?
// - 컴포넌트가 실제로 마운트되었나? (observerCount 확인)

const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  enabled: false, // ← 이것 때문에 실행 안 됨!
});
```

**디버깅 시나리오 2: 캐시가 업데이트 안 됨**
```jsx
// DevTools에서 확인:
// - invalidateQueries가 제대로 호출되었나?
// - queryKey가 일치하는가?
// - staleTime이 너무 길게 설정되어 있나?

// 잘못된 무효화
queryClient.invalidateQueries({ queryKey: ['post'] }); // 's' 빠짐!
// 올바른 키: ['posts']
```

**디버깅 시나리오 3: 불필요한 재요청**
```jsx
// DevTools에서 확인:
// - refetchOnWindowFocus: true인가?
// - staleTime이 너무 짧은가?
// - 컴포넌트가 불필요하게 리마운트되나?

const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 0, // ← 너무 짧음!
  refetchOnWindowFocus: true,
});
// 창 전환할 때마다 재요청 발생
```

### Production 환경

```jsx
// DevTools는 자동으로 production에서 제외됨
// 하지만 명시적으로 제외하려면:

import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      {process.env.NODE_ENV === 'development' && (
        <ReactQueryDevtools initialIsOpen={false} />
      )}
    </QueryClientProvider>
  );
}
```

## Best Practices

### 1. 쿼리 키 설계

```jsx
// ✅ 좋은 예: 계층적 구조
const queryKeys = {
  posts: ['posts'],
  postList: (filters) => ['posts', 'list', filters],
  postDetail: (id) => ['posts', 'detail', id],
  postComments: (id) => ['posts', 'detail', id, 'comments'],
};

// 사용
useQuery({
  queryKey: queryKeys.postDetail(1),
  queryFn: () => fetchPost(1),
});

// 무효화
queryClient.invalidateQueries({ queryKey: queryKeys.posts }); // 모든 포스트 관련 쿼리

// ❌ 나쁜 예: 일관성 없는 구조
['posts']
['post-1']
['getPostById', 1]
['posts_comments_1']
```

### 2. API 함수 분리

```jsx
// api/posts.js
export const postsApi = {
  getAll: async (filters) => {
    const res = await fetch(`/api/posts?${new URLSearchParams(filters)}`);
    if (!res.ok) throw new Error('Failed to fetch posts');
    return res.json();
  },

  getById: async (id) => {
    const res = await fetch(`/api/posts/${id}`);
    if (!res.ok) throw new Error('Failed to fetch post');
    return res.json();
  },

  create: async (data) => {
    const res = await fetch('/api/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    if (!res.ok) throw new Error('Failed to create post');
    return res.json();
  },
};

// hooks/usePosts.js
export function usePosts(filters) {
  return useQuery({
    queryKey: ['posts', 'list', filters],
    queryFn: () => postsApi.getAll(filters),
  });
}

export function usePost(id) {
  return useQuery({
    queryKey: ['posts', 'detail', id],
    queryFn: () => postsApi.getById(id),
    enabled: !!id,
  });
}

export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: postsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts', 'list'] });
    },
  });
}

// 컴포넌트에서 사용
function PostList() {
  const { data: posts } = usePosts({ status: 'published' });
  const createPost = useCreatePost();

  // ...
}
```

### 3. TypeScript 타입 안정성

```tsx
// types.ts
export interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: string;
}

export interface PostFilters {
  status?: 'draft' | 'published';
  authorId?: number;
  page?: number;
  limit?: number;
}

// api/posts.ts
export const postsApi = {
  getAll: async (filters: PostFilters): Promise<Post[]> => {
    const res = await fetch(`/api/posts?${new URLSearchParams(filters as any)}`);
    if (!res.ok) throw new Error('Failed to fetch posts');
    return res.json();
  },
};

// hooks/usePosts.ts
export function usePosts(filters: PostFilters) {
  return useQuery<Post[], Error>({
    queryKey: ['posts', 'list', filters],
    queryFn: () => postsApi.getAll(filters),
  });
}

// 컴포넌트
function PostList() {
  const { data: posts } = usePosts({ status: 'published' });

  // posts는 자동으로 Post[] 타입!
  return (
    <ul>
      {posts?.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

### 4. 쿼리 클라이언트 설정 최적화

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // 대부분의 앱에 적합한 기본값
      staleTime: 60 * 1000, // 1분
      gcTime: 5 * 60 * 1000, // 5분
      retry: 1,
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,

      // 에러 처리
      onError: (error) => {
        console.error('Query error:', error);
      },
    },
    mutations: {
      // 뮤테이션 에러 처리
      onError: (error) => {
        toast.error(error.message);
      },
    },
  },
});
```

### 5. Suspense 모드 (실험적)

```jsx
// App.jsx
import { Suspense } from 'react';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true, // Suspense 모드 활성화
    },
  },
});

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <PostList />
    </Suspense>
  );
}

// PostList.jsx
function PostList() {
  // isLoading 불필요! Suspense가 처리
  const { data: posts } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  // posts는 항상 존재 (로딩 중이면 Suspense가 잡음)
  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

### 6. 쿼리 키 팩토리 패턴

```jsx
// queryKeys.js
export const queryKeys = {
  // Posts
  posts: {
    all: ['posts'],
    lists: () => [...queryKeys.posts.all, 'list'],
    list: (filters) => [...queryKeys.posts.lists(), filters],
    details: () => [...queryKeys.posts.all, 'detail'],
    detail: (id) => [...queryKeys.posts.details(), id],
  },

  // Users
  users: {
    all: ['users'],
    detail: (id) => [...queryKeys.users.all, id],
  },
};

// 사용
useQuery({
  queryKey: queryKeys.posts.list({ status: 'published' }),
  queryFn: () => fetchPosts({ status: 'published' }),
});

// 무효화
queryClient.invalidateQueries({ queryKey: queryKeys.posts.lists() });
// 모든 포스트 목록 쿼리 무효화
```

### 7. Select로 데이터 변환

```jsx
// 컴포넌트에서 변환하지 말고 select 사용
// ❌ 나쁜 예
function PostTitles() {
  const { data: posts } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  // 매 렌더링마다 map 실행 (비효율)
  const titles = posts?.map(p => p.title);
}

// ✅ 좋은 예
function PostTitles() {
  const { data: titles } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    select: (posts) => posts.map(p => p.title), // 메모이제이션 자동 적용
  });
}
```

### 8. 조건부 쿼리 패턴

```jsx
// ✅ 좋은 예: enabled 사용
function UserPosts({ userId }) {
  const { data: posts } = useQuery({
    queryKey: ['posts', { userId }],
    queryFn: () => fetchUserPosts(userId),
    enabled: !!userId, // userId가 있을 때만 실행
  });
}

// ❌ 나쁜 예: 조건문으로 분기
function UserPosts({ userId }) {
  if (!userId) {
    return null;
  }

  // Hook이 조건부로 호출됨 (React 규칙 위반!)
  const { data: posts } = useQuery({
    queryKey: ['posts', { userId }],
    queryFn: () => fetchUserPosts(userId),
  });
}
```

## FAQ

### Q1: React Query vs Redux, 언제 무엇을 사용하나?

**React Query**를 사용하세요:
- ✅ 서버 데이터 페칭 (API 호출)
- ✅ 캐싱, 동기화, 재시도 필요
- ✅ 실시간 업데이트 필요
- ✅ 예: 블로그 포스트, 사용자 프로필, 상품 목록

**Redux/Zustand**를 사용하세요:
- ✅ 클라이언트 전용 상태 (UI 상태)
- ✅ 복잡한 상태 로직
- ✅ 예: 테마 설정, 모달 상태, 폼 데이터

**둘 다 사용하는 경우**:
```jsx
// Redux: 클라이언트 상태
const theme = useSelector(state => state.theme);
const isModalOpen = useSelector(state => state.ui.isModalOpen);

// React Query: 서버 상태
const { data: posts } = useQuery(['posts'], fetchPosts);
const { data: user } = useQuery(['user'], fetchUser);
```

### Q2: staleTime과 gcTime의 차이는?

```jsx
// staleTime: 데이터가 "신선한" 시간
// - 이 시간 동안은 재요청 안 함
// - 컴포넌트 재렌더링 시에도 API 호출 없음

// gcTime: 캐시 유지 시간
// - 모든 컴포넌트가 언마운트된 후 이 시간 동안 캐시 유지
// - 시간 경과 후 메모리에서 제거

// 예시
staleTime: 1분, gcTime: 5분

// 0:00 - API 요청, fresh
// 1:00 - stale (백그라운드 재검증 가능)
// 2:00 - 컴포넌트 언마운트
// 7:00 - gcTime 경과, 캐시 제거
```

### Q3: useQuery가 무한 루프에 빠졌어요!

```jsx
// ❌ 원인: 쿼리 키가 매번 새로운 객체
function PostList() {
  const [filters, setFilters] = useState({ status: 'published' });

  const { data } = useQuery({
    queryKey: ['posts', filters], // filters 객체가 매번 새로 생성!
    queryFn: () => fetchPosts(filters),
  });
}

// ✅ 해결 1: 필터를 개별 값으로
const [status, setStatus] = useState('published');
useQuery({
  queryKey: ['posts', status],
  queryFn: () => fetchPosts({ status }),
});

// ✅ 해결 2: useMemo로 메모이제이션
const filters = useMemo(() => ({ status }), [status]);
useQuery({
  queryKey: ['posts', filters],
  queryFn: () => fetchPosts(filters),
});
```

### Q4: 쿼리 결과를 다른 쿼리에 사용하려면?

```jsx
// Dependent Queries 사용
function UserPosts({ userId }) {
  // 1단계: 사용자 정보
  const { data: user } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
  });

  // 2단계: 사용자의 포스트 (enabled로 의존성 표현)
  const { data: posts } = useQuery({
    queryKey: ['posts', { authorId: user?.id }],
    queryFn: () => fetchPosts(user.id),
    enabled: !!user, // user가 있을 때만 실행
  });
}
```

### Q5: 여러 컴포넌트에서 같은 쿼리를 사용하면 중복 요청되나?

```jsx
// ❌ 걱정: 중복 요청?
function Header() {
  const { data: user } = useQuery(['user'], fetchUser);
}

function Sidebar() {
  const { data: user } = useQuery(['user'], fetchUser);
}

// ✅ 실제: 자동으로 중복 제거됨!
// - 같은 queryKey를 사용하면 하나의 요청만 실행
// - 결과는 두 컴포넌트가 공유
// - 캐싱 덕분에 성능 걱정 없음
```

### Q6: mutation 후 쿼리가 자동으로 업데이트 안 돼요!

```jsx
// ❌ 문제: invalidateQueries 안 함
const createMutation = useMutation({
  mutationFn: createPost,
  // 이것만으로는 목록이 업데이트 안 됨
});

// ✅ 해결: onSuccess에서 invalidateQueries
const createMutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});

// 또는 낙관적 업데이트
const createMutation = useMutation({
  mutationFn: createPost,
  onMutate: async (newPost) => {
    await queryClient.cancelQueries({ queryKey: ['posts'] });
    const previous = queryClient.getQueryData(['posts']);

    queryClient.setQueryData(['posts'], (old) => [...old, newPost]);

    return { previous };
  },
  onError: (err, newPost, context) => {
    queryClient.setQueryData(['posts'], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});
```

### Q7: 페이지네이션에서 이전 데이터를 유지하려면?

```jsx
// placeholderData 사용
const { data } = useQuery({
  queryKey: ['posts', page],
  queryFn: () => fetchPosts(page),
  placeholderData: (previousData) => previousData, // 핵심!
});

// 결과:
// - 페이지 전환 시 이전 데이터를 보여줌
// - 백그라운드에서 새 데이터 로딩
// - 새 데이터 도착하면 부드럽게 전환
```

### Q8: React Query를 Next.js에서 어떻게 사용하나?

```jsx
// app/providers.jsx (App Router)
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';

export function Providers({ children }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

// app/layout.jsx
import { Providers } from './providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}

// SSR/SSG with Hydration
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query';

export async function getServerSideProps() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
}

function Page({ dehydratedState }) {
  return (
    <HydrationBoundary state={dehydratedState}>
      <PostList />
    </HydrationBoundary>
  );
}
```

## 결론

이번 포스팅에서는 React Query (TanStack Query)의 모든 것을 다뤘습니다. 서버 상태 관리의 복잡함을 React Query가 어떻게 해결하는지, 그리고 실전에서 어떻게 활용하는지 살펴봤습니다.

### 핵심 정리

**1. 상태 구분이 중요하다**
- 클라이언트 상태: useState, Zustand, Redux
- 서버 상태: React Query, SWR

**2. React Query의 강점**
- ✅ 자동 캐싱 및 중복 제거
- ✅ 백그라운드 업데이트
- ✅ 낙관적 업데이트
- ✅ 페이지네이션/무한 스크롤
- ✅ 강력한 DevTools

**3. 주요 개념 복습**
```jsx
// 데이터 읽기: useQuery
const { data, isLoading, error } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 60 * 1000,
});

// 데이터 변경: useMutation
const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});

// 무한 스크롤: useInfiniteQuery
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ['posts', 'infinite'],
  queryFn: ({ pageParam }) => fetchPosts(pageParam),
  getNextPageParam: (lastPage) => lastPage.nextCursor,
  initialPageParam: null,
});
```

**4. Best Practices**
- 쿼리 키를 계층적으로 설계
- API 함수와 Hook을 분리
- TypeScript로 타입 안정성 확보
- 낙관적 업데이트로 UX 향상
- DevTools로 적극 디버깅

### 상태 관리 시리즈 마무리

이제 React 상태 관리 시리즈 3부작을 모두 마쳤습니다:

1. [React 상태 관리 완벽 가이드](/posts/react-state-management-guide/) - useState, useReducer, Context API
2. [Zustand vs Redux Toolkit](/posts/zustand-vs-redux-toolkit/) - 전역 상태 관리
3. **React Query 완벽 가이드** - 서버 상태 관리

이 세 가지를 조합하면 어떤 규모의 React 애플리케이션도 효과적으로 관리할 수 있습니다:

```jsx
// 종합 예시
function App() {
  // 1. 클라이언트 상태 (useState)
  const [isModalOpen, setIsModalOpen] = useState(false);

  // 2. 전역 상태 (Zustand)
  const theme = useThemeStore(state => state.theme);

  // 3. 서버 상태 (React Query)
  const { data: user } = useQuery(['user'], fetchUser);
  const { data: posts } = useQuery(['posts'], fetchPosts);

  return (
    <div className={theme}>
      {user && <Welcome user={user} />}
      <PostList posts={posts} />
      {isModalOpen && <Modal />}
    </div>
  );
}
```

### 다음 단계

React Query를 마스터했다면:
1. **프로젝트에 적용하기**: 기존 useEffect + fetch를 React Query로 마이그레이션
2. **SWR과 비교하기**: 다른 서버 상태 관리 라이브러리 살펴보기
3. **GraphQL과 통합**: Apollo Client 대신 React Query + GraphQL 조합 시도
4. **실시간 기능**: WebSocket, Server-Sent Events와 통합

React Query는 단순한 라이브러리가 아니라 서버 상태 관리의 **새로운 패러다임**입니다. 이를 마스터하면 개발 생산성과 애플리케이션 품질이 크게 향상될 것입니다.

여러분의 React 앱에 React Query를 도입하고, 서버 상태 관리의 복잡함에서 벗어나세요!

## 참고 자료

### 공식 문서
- [TanStack Query 공식 문서](https://tanstack.com/query/latest)
- [React Query v5 마이그레이션 가이드](https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5)
- [React Query DevTools](https://tanstack.com/query/latest/docs/framework/react/devtools)

### 추천 글
- [React 상태 관리 완벽 가이드](/posts/react-state-management-guide/)
- [Zustand vs Redux Toolkit](/posts/zustand-vs-redux-toolkit/)

### 참고 자료
- [React Query vs SWR 비교](https://tanstack.com/query/latest/docs/framework/react/comparison)
- [Optimistic Updates 패턴](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates)
- [TanStack Query GitHub](https://github.com/TanStack/query)
