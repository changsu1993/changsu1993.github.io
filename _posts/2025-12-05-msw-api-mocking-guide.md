---
title: "MSW (Mock Service Worker) 완벽 가이드 - API 모킹과 테스트 전략"
description: "MSW 2.0을 활용한 API 모킹의 모든 것. 설치부터 브라우저/Node.js 설정, REST/GraphQL 핸들러, Vitest/Jest/Playwright 테스트 환경 연동까지 실전 예제와 함께 완벽 정리합니다."
date: 2025-12-05 18:30:00 +0900
categories: [Frontend, Testing]
tags: [msw, mock-service-worker, api-mocking, testing, react, typescript, vitest, jest, service-worker, frontend-testing]
---

## 개요

API 개발이 완료되지 않았는데 프론트엔드 개발을 시작해야 한다면 어떻게 해야 할까요? 테스트 환경에서 실제 API 호출을 어떻게 격리할 수 있을까요?

**MSW(Mock Service Worker)**는 이런 문제를 우아하게 해결하는 API 모킹 라이브러리입니다. Service Worker를 활용해 네트워크 레벨에서 요청을 가로채기 때문에, fetch나 axios 같은 HTTP 클라이언트를 수정하지 않고도 API 응답을 모킹할 수 있습니다.

> MSW는 Google, Microsoft, Spotify, Amazon, Netflix 등 수많은 기업에서 사용하는 업계 표준 API 모킹 라이브러리입니다.
{: .prompt-info }

### MSW를 사용하는 이유

**1. 애플리케이션 무결성 유지**

기존의 API 모킹 방식은 fetch나 axios를 직접 패치(monkey-patch)하는 방식이었습니다. 이 방식은 애플리케이션의 런타임을 변경하므로 테스트 환경과 실제 환경의 동작이 다를 수 있습니다.

```typescript
// 기존 방식: fetch를 직접 mock
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve({ data: 'mocked' }),
  })
);

// MSW 방식: 네트워크 레벨에서 가로채기
// fetch는 그대로 동작하고, 네트워크 요청만 가로챔
```

**2. 재사용 가능한 모킹 레이어**

한 번 정의한 핸들러를 개발, 테스트, Storybook, 데모 등 다양한 환경에서 재사용할 수 있습니다.

**3. 브라우저와 Node.js 모두 지원**

동일한 핸들러 코드로 브라우저 개발 환경과 Node.js 테스트 환경 모두에서 사용할 수 있습니다.

**4. REST, GraphQL, WebSocket 모두 지원**

다양한 프로토콜의 API를 일관된 방식으로 모킹할 수 있습니다.

> 프론트엔드 테스트에 대한 전반적인 이해가 필요하다면 [프론트엔드 테스트 완벽 가이드](/posts/frontend-testing-guide/)를 먼저 읽어보시는 것을 추천합니다.
{: .prompt-tip }

## MSW의 동작 원리

MSW는 Service Worker API를 활용하여 브라우저에서 발생하는 네트워크 요청을 가로챕니다.

### 브라우저 환경

```
[애플리케이션] → [fetch 요청] → [Service Worker] → [MSW 핸들러]
                                        ↓
                              [모킹된 응답 반환]
```

1. 애플리케이션이 일반적인 fetch 요청을 보냄
2. Service Worker가 네트워크 요청을 가로챔
3. MSW 핸들러가 요청을 분석하고 적절한 응답을 생성
4. 모킹된 응답이 애플리케이션에 전달됨

### Node.js 환경

Node.js에서는 Service Worker가 없으므로, MSW는 Node.js의 HTTP/HTTPS 모듈을 확장하여 요청을 가로챕니다. 중요한 점은 모듈을 패치하는 것이 아니라 **확장**한다는 것입니다.

```typescript
// Node.js 환경에서는 setupServer 사용
import { setupServer } from 'msw/node';
```

## 설치 및 기본 설정

### 패키지 설치

```bash
npm install msw --save-dev
```

### 핸들러 정의

먼저 API 핸들러를 정의합니다. 이 핸들러는 브라우저와 Node.js 환경 모두에서 공유됩니다.

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

// 사용자 타입 정의
interface User {
  id: number;
  name: string;
  email: string;
}

// 모킹할 사용자 데이터
const users: User[] = [
  { id: 1, name: '홍길동', email: 'hong@example.com' },
  { id: 2, name: '김철수', email: 'kim@example.com' },
];

export const handlers = [
  // GET /api/users - 사용자 목록 조회
  http.get('/api/users', () => {
    return HttpResponse.json(users);
  }),

  // GET /api/users/:id - 특정 사용자 조회
  http.get('/api/users/:id', ({ params }) => {
    const { id } = params;
    const user = users.find((u) => u.id === Number(id));

    if (!user) {
      return HttpResponse.json(
        { error: '사용자를 찾을 수 없습니다' },
        { status: 404 }
      );
    }

    return HttpResponse.json(user);
  }),

  // POST /api/users - 사용자 생성
  http.post('/api/users', async ({ request }) => {
    const newUser = (await request.json()) as Omit<User, 'id'>;

    const user: User = {
      id: users.length + 1,
      ...newUser,
    };

    users.push(user);

    return HttpResponse.json(user, { status: 201 });
  }),
];
```

### 브라우저 환경 설정

**1. Service Worker 스크립트 생성**

MSW CLI를 사용하여 Service Worker 스크립트를 생성합니다.

```bash
# public 폴더가 정적 파일 디렉토리인 경우
npx msw init public/ --save
```

이 명령은 `public/mockServiceWorker.js` 파일을 생성합니다.

**2. 브라우저 워커 설정**

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

**3. 애플리케이션 진입점에서 워커 시작**

```tsx
// src/main.tsx (React + Vite 프로젝트 예시)
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

async function enableMocking() {
  // 개발 환경에서만 MSW 활성화
  if (process.env.NODE_ENV !== 'development') {
    return;
  }

  const { worker } = await import('./mocks/browser');

  // 워커 시작 (반드시 await 필요!)
  return worker.start({
    // Service Worker 요청은 콘솔에 표시하지 않음
    quiet: true,
    // 핸들러가 없는 요청은 실제 네트워크로 전달
    onUnhandledRequest: 'bypass',
  });
}

enableMocking().then(() => {
  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
});
```

> `worker.start()`는 반드시 `await`해야 합니다. 그렇지 않으면 Service Worker 등록 전에 API 요청이 발생하여 모킹이 적용되지 않을 수 있습니다.
{: .prompt-warning }

> Vite 프로젝트 설정에 대한 자세한 내용은 [Vite 완벽 가이드](/posts/vite-complete-guide/)를 참고하세요.
{: .prompt-tip }

### Node.js 환경 설정 (테스트용)

```typescript
// src/mocks/node.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

## 핸들러 작성 기초

### REST API 핸들러

MSW 2.0에서는 `http` 네임스페이스를 사용하여 REST API 핸들러를 정의합니다.

```typescript
import { http, HttpResponse } from 'msw';

export const handlers = [
  // GET 요청
  http.get('/api/posts', () => {
    return HttpResponse.json([
      { id: 1, title: '첫 번째 글' },
      { id: 2, title: '두 번째 글' },
    ]);
  }),

  // POST 요청 - 요청 본문 처리
  http.post('/api/posts', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      { id: 3, ...body },
      { status: 201 }
    );
  }),

  // PUT 요청 - 경로 파라미터와 본문 처리
  http.put('/api/posts/:id', async ({ params, request }) => {
    const { id } = params;
    const updates = await request.json();
    return HttpResponse.json({ id: Number(id), ...updates });
  }),

  // DELETE 요청
  http.delete('/api/posts/:id', ({ params }) => {
    const { id } = params;
    return new HttpResponse(null, { status: 204 });
  }),

  // PATCH 요청
  http.patch('/api/posts/:id', async ({ params, request }) => {
    const { id } = params;
    const updates = await request.json();
    return HttpResponse.json({ id: Number(id), ...updates });
  }),

  // 모든 HTTP 메서드 처리
  http.all('/api/health', () => {
    return HttpResponse.json({ status: 'ok' });
  }),
];
```

### 요청 정보 활용하기

핸들러의 resolver 함수는 다양한 요청 정보에 접근할 수 있습니다.

```typescript
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/search', async ({ request, params, cookies }) => {
    // URL 객체로 쿼리 파라미터 접근
    const url = new URL(request.url);
    const query = url.searchParams.get('q');
    const page = url.searchParams.get('page') || '1';

    // 요청 헤더 접근
    const authHeader = request.headers.get('Authorization');

    // 쿠키 접근
    const sessionId = cookies.session_id;

    // 조건부 응답
    if (!authHeader) {
      return HttpResponse.json(
        { error: '인증이 필요합니다' },
        { status: 401 }
      );
    }

    return HttpResponse.json({
      query,
      page: Number(page),
      results: [`${query} 검색 결과 1`, `${query} 검색 결과 2`],
    });
  }),
];
```

### GraphQL 핸들러

MSW는 GraphQL 요청도 모킹할 수 있습니다.

```typescript
import { graphql, HttpResponse } from 'msw';

// 타입 정의
interface User {
  id: string;
  name: string;
  email: string;
}

interface GetUserVariables {
  id: string;
}

interface GetUserData {
  user: User | null;
}

interface CreateUserVariables {
  input: {
    name: string;
    email: string;
  };
}

interface CreateUserData {
  createUser: User;
}

export const graphqlHandlers = [
  // Query 핸들러
  graphql.query<GetUserData, GetUserVariables>('GetUser', ({ variables }) => {
    const { id } = variables;

    return HttpResponse.json({
      data: {
        user: {
          id,
          name: '홍길동',
          email: 'hong@example.com',
        },
      },
    });
  }),

  // Mutation 핸들러
  graphql.mutation<CreateUserData, CreateUserVariables>(
    'CreateUser',
    ({ variables }) => {
      const { input } = variables;

      return HttpResponse.json({
        data: {
          createUser: {
            id: 'new-user-id',
            name: input.name,
            email: input.email,
          },
        },
      });
    }
  ),

  // 특정 엔드포인트로 스코프 지정
  graphql.link('https://api.github.com/graphql').query('GetRepository', () => {
    return HttpResponse.json({
      data: {
        repository: {
          name: 'msw',
          stargazerCount: 15000,
        },
      },
    });
  }),
];
```

### HttpResponse 활용

`HttpResponse` 클래스는 다양한 응답 형식을 지원합니다.

```typescript
import { http, HttpResponse } from 'msw';

export const handlers = [
  // JSON 응답
  http.get('/api/json', () => {
    return HttpResponse.json({ message: 'Hello' });
  }),

  // 텍스트 응답
  http.get('/api/text', () => {
    return HttpResponse.text('Plain text response');
  }),

  // HTML 응답
  http.get('/api/html', () => {
    return HttpResponse.html('<h1>Hello World</h1>');
  }),

  // XML 응답
  http.get('/api/xml', () => {
    return HttpResponse.xml('<root><message>Hello</message></root>');
  }),

  // 바이너리 응답
  http.get('/api/image', () => {
    const buffer = new ArrayBuffer(8);
    return HttpResponse.arrayBuffer(buffer, {
      headers: {
        'Content-Type': 'image/png',
      },
    });
  }),

  // 커스텀 상태 코드와 헤더
  http.post('/api/created', () => {
    return HttpResponse.json(
      { id: 1, created: true },
      {
        status: 201,
        statusText: 'Created',
        headers: {
          'X-Custom-Header': 'custom-value',
          'Set-Cookie': 'session=abc123; Path=/; HttpOnly',
        },
      }
    );
  }),

  // 네트워크 에러
  http.get('/api/network-error', () => {
    return HttpResponse.error();
  }),
];
```

## 실전 예제

### React + TypeScript 프로젝트에서 MSW 사용

실제 프로젝트에서 MSW를 활용하는 예제를 살펴보겠습니다.

**1. API 타입 정의**

```typescript
// src/types/api.ts
export interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  user: User;
  token: string;
}

export interface ApiError {
  error: string;
  code: string;
}
```

**2. 핸들러 정의**

```typescript
// src/mocks/handlers/auth.ts
import { http, HttpResponse, delay } from 'msw';
import type { LoginRequest, LoginResponse, User } from '../../types/api';

// 모킹용 사용자 데이터
const mockUsers: (User & { password: string })[] = [
  {
    id: 1,
    name: '관리자',
    email: 'admin@example.com',
    password: 'admin123',
    role: 'admin',
  },
  {
    id: 2,
    name: '홍길동',
    email: 'hong@example.com',
    password: 'user123',
    role: 'user',
  },
];

let currentUser: User | null = null;

export const authHandlers = [
  // 로그인
  http.post('/api/auth/login', async ({ request }) => {
    // 실제 네트워크 지연 시뮬레이션
    await delay(500);

    const { email, password } = (await request.json()) as LoginRequest;

    const user = mockUsers.find(
      (u) => u.email === email && u.password === password
    );

    if (!user) {
      return HttpResponse.json(
        { error: '이메일 또는 비밀번호가 올바르지 않습니다', code: 'INVALID_CREDENTIALS' },
        { status: 401 }
      );
    }

    // 비밀번호 제외한 사용자 정보
    const { password: _, ...userWithoutPassword } = user;
    currentUser = userWithoutPassword;

    const response: LoginResponse = {
      user: userWithoutPassword,
      token: `mock-jwt-token-${user.id}`,
    };

    return HttpResponse.json(response, {
      headers: {
        'Set-Cookie': `auth-token=mock-jwt-token-${user.id}; Path=/; HttpOnly`,
      },
    });
  }),

  // 로그아웃
  http.post('/api/auth/logout', async () => {
    await delay(200);
    currentUser = null;

    return new HttpResponse(null, {
      status: 204,
      headers: {
        'Set-Cookie': 'auth-token=; Path=/; Max-Age=0',
      },
    });
  }),

  // 현재 사용자 정보
  http.get('/api/auth/me', async ({ cookies }) => {
    await delay(100);

    const token = cookies['auth-token'];

    if (!token || !currentUser) {
      return HttpResponse.json(
        { error: '인증이 필요합니다', code: 'UNAUTHORIZED' },
        { status: 401 }
      );
    }

    return HttpResponse.json(currentUser);
  }),
];
```

**3. 로그인 컴포넌트**

```tsx
// src/components/LoginForm.tsx
import { useState } from 'react';
import type { LoginRequest, LoginResponse, ApiError } from '../types/api';

export function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setLoading(true);

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password } as LoginRequest),
      });

      if (!response.ok) {
        const errorData: ApiError = await response.json();
        throw new Error(errorData.error);
      }

      const data: LoginResponse = await response.json();
      console.log('로그인 성공:', data.user.name);
      // 로그인 성공 후 처리 (예: 리다이렉트, 상태 업데이트)
    } catch (err) {
      setError(err instanceof Error ? err.message : '로그인에 실패했습니다');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
      </div>
      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
      </div>
      {error && <div role="alert">{error}</div>}
      <button type="submit" disabled={loading}>
        {loading ? '로그인 중...' : '로그인'}
      </button>
    </form>
  );
}
```

### 페이지네이션 API 모킹

```typescript
// src/mocks/handlers/posts.ts
import { http, HttpResponse, delay } from 'msw';

interface Post {
  id: number;
  title: string;
  content: string;
  author: string;
  createdAt: string;
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// 대량의 모킹 데이터 생성
const generatePosts = (count: number): Post[] => {
  return Array.from({ length: count }, (_, i) => ({
    id: i + 1,
    title: `게시글 제목 ${i + 1}`,
    content: `게시글 ${i + 1}의 내용입니다. Lorem ipsum dolor sit amet.`,
    author: `작성자${(i % 5) + 1}`,
    createdAt: new Date(Date.now() - i * 86400000).toISOString(),
  }));
};

const allPosts = generatePosts(100);

export const postHandlers = [
  http.get('/api/posts', async ({ request }) => {
    await delay(300);

    const url = new URL(request.url);
    const page = parseInt(url.searchParams.get('page') || '1', 10);
    const limit = parseInt(url.searchParams.get('limit') || '10', 10);
    const search = url.searchParams.get('search') || '';

    // 검색 필터링
    let filteredPosts = allPosts;
    if (search) {
      filteredPosts = allPosts.filter(
        (post) =>
          post.title.includes(search) || post.content.includes(search)
      );
    }

    // 페이지네이션 계산
    const total = filteredPosts.length;
    const totalPages = Math.ceil(total / limit);
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    const paginatedPosts = filteredPosts.slice(startIndex, endIndex);

    const response: PaginatedResponse<Post> = {
      data: paginatedPosts,
      pagination: {
        page,
        limit,
        total,
        totalPages,
      },
    };

    return HttpResponse.json(response);
  }),

  http.get('/api/posts/:id', async ({ params }) => {
    await delay(200);

    const { id } = params;
    const post = allPosts.find((p) => p.id === Number(id));

    if (!post) {
      return HttpResponse.json(
        { error: '게시글을 찾을 수 없습니다' },
        { status: 404 }
      );
    }

    return HttpResponse.json(post);
  }),
];
```

### 에러 응답 시뮬레이션

다양한 에러 상황을 시뮬레이션하는 핸들러입니다.

```typescript
// src/mocks/handlers/errors.ts
import { http, HttpResponse, delay } from 'msw';

export const errorHandlers = [
  // 400 Bad Request
  http.post('/api/validate', async ({ request }) => {
    const body = await request.json();

    const errors: Record<string, string> = {};

    if (!body.email) {
      errors.email = '이메일은 필수입니다';
    } else if (!body.email.includes('@')) {
      errors.email = '올바른 이메일 형식이 아닙니다';
    }

    if (!body.password) {
      errors.password = '비밀번호는 필수입니다';
    } else if (body.password.length < 8) {
      errors.password = '비밀번호는 8자 이상이어야 합니다';
    }

    if (Object.keys(errors).length > 0) {
      return HttpResponse.json(
        { errors, message: '유효성 검사에 실패했습니다' },
        { status: 400 }
      );
    }

    return HttpResponse.json({ success: true });
  }),

  // 403 Forbidden
  http.delete('/api/admin/users/:id', ({ request }) => {
    const authHeader = request.headers.get('Authorization');

    if (!authHeader?.includes('admin')) {
      return HttpResponse.json(
        { error: '관리자 권한이 필요합니다', code: 'FORBIDDEN' },
        { status: 403 }
      );
    }

    return new HttpResponse(null, { status: 204 });
  }),

  // 429 Too Many Requests (Rate Limiting)
  http.get('/api/rate-limited', () => {
    return HttpResponse.json(
      {
        error: '요청이 너무 많습니다. 잠시 후 다시 시도해주세요.',
        retryAfter: 60,
      },
      {
        status: 429,
        headers: {
          'Retry-After': '60',
        },
      }
    );
  }),

  // 500 Internal Server Error
  http.get('/api/unstable', () => {
    // 50% 확률로 에러 발생
    if (Math.random() < 0.5) {
      return HttpResponse.json(
        { error: '서버 내부 오류가 발생했습니다' },
        { status: 500 }
      );
    }

    return HttpResponse.json({ data: 'success' });
  }),

  // 503 Service Unavailable
  http.get('/api/maintenance', () => {
    return HttpResponse.json(
      {
        error: '서비스 점검 중입니다',
        expectedEndTime: '2025-12-06T00:00:00Z',
      },
      { status: 503 }
    );
  }),

  // 네트워크 에러 (연결 실패)
  http.get('/api/network-failure', () => {
    return HttpResponse.error();
  }),
];
```

## 테스트 환경 연동

### Vitest와 함께 사용

Vitest에서 MSW를 설정하는 방법입니다.

**1. 테스트 설정 파일**

```typescript
// src/mocks/setup.ts
import { beforeAll, afterEach, afterAll } from 'vitest';
import { server } from './node';

// 모든 테스트 시작 전 서버 시작
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// 각 테스트 후 핸들러 리셋
afterEach(() => server.resetHandlers());

// 모든 테스트 완료 후 서버 종료
afterAll(() => server.close());
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/mocks/setup.ts'],
    globals: true,
  },
});
```

**2. 테스트 작성**

```tsx
// src/components/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from '../mocks/node';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('성공적으로 로그인하면 사용자 정보를 표시한다', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    await user.type(screen.getByLabelText('이메일'), 'admin@example.com');
    await user.type(screen.getByLabelText('비밀번호'), 'admin123');
    await user.click(screen.getByRole('button', { name: '로그인' }));

    // 로딩 상태 확인
    expect(screen.getByText('로그인 중...')).toBeInTheDocument();

    // 성공 후 로딩 상태 해제
    await waitFor(() => {
      expect(screen.queryByText('로그인 중...')).not.toBeInTheDocument();
    });
  });

  it('잘못된 자격 증명으로 로그인하면 에러를 표시한다', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    await user.type(screen.getByLabelText('이메일'), 'wrong@example.com');
    await user.type(screen.getByLabelText('비밀번호'), 'wrongpassword');
    await user.click(screen.getByRole('button', { name: '로그인' }));

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent(
        '이메일 또는 비밀번호가 올바르지 않습니다'
      );
    });
  });

  it('서버 에러 시 적절한 에러 메시지를 표시한다', async () => {
    // 이 테스트에서만 핸들러 오버라이드
    server.use(
      http.post('/api/auth/login', () => {
        return HttpResponse.json(
          { error: '서버 오류가 발생했습니다' },
          { status: 500 }
        );
      })
    );

    const user = userEvent.setup();
    render(<LoginForm />);

    await user.type(screen.getByLabelText('이메일'), 'test@example.com');
    await user.type(screen.getByLabelText('비밀번호'), 'password123');
    await user.click(screen.getByRole('button', { name: '로그인' }));

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('서버 오류가 발생했습니다');
    });
  });
});
```

> TDD(테스트 주도 개발) 방법론에 대해 더 알고 싶다면 [TDD 실전 가이드](/posts/tdd-practical-guide/)를 참고하세요.
{: .prompt-tip }

### Jest와 함께 사용

Jest에서도 유사하게 설정할 수 있습니다.

```typescript
// jest.setup.ts
import { server } from './src/mocks/node';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
};
```

### Playwright E2E 테스트 연동

Playwright에서 MSW를 사용하려면 브라우저 환경에서 모킹을 설정해야 합니다.

```typescript
// e2e/fixtures.ts
import { test as base } from '@playwright/test';

export const test = base.extend({
  // 각 테스트 전에 MSW 활성화
  page: async ({ page }, use) => {
    // 개발 서버가 이미 MSW를 사용하고 있다면 그대로 사용
    await page.goto('/');

    // MSW가 활성화되었는지 확인
    await page.waitForFunction(() => {
      return (window as any).__MSW_READY__ === true;
    });

    await use(page);
  },
});

export { expect } from '@playwright/test';
```

```tsx
// src/main.tsx - MSW 준비 상태 표시
enableMocking().then(() => {
  // MSW 준비 완료 표시
  (window as any).__MSW_READY__ = true;

  ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
});
```

```typescript
// e2e/login.spec.ts
import { test, expect } from './fixtures';

test.describe('로그인 페이지', () => {
  test('유효한 자격 증명으로 로그인할 수 있다', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'admin@example.com');
    await page.fill('input[name="password"]', 'admin123');
    await page.click('button[type="submit"]');

    // 로그인 성공 후 대시보드로 이동
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=관리자')).toBeVisible();
  });
});
```

> E2E 테스트에 대한 더 자세한 내용은 [E2E 테스트 실전 가이드](/posts/e2e-testing-guide/)를 참고하세요.
{: .prompt-tip }

## 고급 패턴

### 동적 응답 생성

상태를 유지하며 CRUD 작업을 시뮬레이션하는 패턴입니다.

```typescript
// src/mocks/handlers/dynamic.ts
import { http, HttpResponse } from 'msw';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

// 메모리 내 데이터 저장소
let todos: Todo[] = [
  { id: 1, title: '리액트 공부하기', completed: false },
  { id: 2, title: 'MSW 익히기', completed: true },
];

let nextId = 3;

// 데이터 리셋 함수 (테스트 간 격리용)
export const resetTodos = () => {
  todos = [
    { id: 1, title: '리액트 공부하기', completed: false },
    { id: 2, title: 'MSW 익히기', completed: true },
  ];
  nextId = 3;
};

export const todoHandlers = [
  // 목록 조회
  http.get('/api/todos', () => {
    return HttpResponse.json(todos);
  }),

  // 생성
  http.post('/api/todos', async ({ request }) => {
    const { title } = (await request.json()) as { title: string };

    const newTodo: Todo = {
      id: nextId++,
      title,
      completed: false,
    };

    todos.push(newTodo);
    return HttpResponse.json(newTodo, { status: 201 });
  }),

  // 수정
  http.patch('/api/todos/:id', async ({ params, request }) => {
    const { id } = params;
    const updates = (await request.json()) as Partial<Todo>;

    const todoIndex = todos.findIndex((t) => t.id === Number(id));

    if (todoIndex === -1) {
      return HttpResponse.json(
        { error: 'Todo not found' },
        { status: 404 }
      );
    }

    todos[todoIndex] = { ...todos[todoIndex], ...updates };
    return HttpResponse.json(todos[todoIndex]);
  }),

  // 삭제
  http.delete('/api/todos/:id', ({ params }) => {
    const { id } = params;
    const todoIndex = todos.findIndex((t) => t.id === Number(id));

    if (todoIndex === -1) {
      return HttpResponse.json(
        { error: 'Todo not found' },
        { status: 404 }
      );
    }

    todos.splice(todoIndex, 1);
    return new HttpResponse(null, { status: 204 });
  }),
];
```

### 지연(delay) 시뮬레이션

```typescript
import { http, HttpResponse, delay } from 'msw';

export const handlers = [
  // 고정 지연
  http.get('/api/slow', async () => {
    await delay(2000); // 2초 지연
    return HttpResponse.json({ message: '느린 응답' });
  }),

  // 현실적인 지연 (100-400ms 랜덤)
  http.get('/api/realistic', async () => {
    await delay(); // 기본값: 현실적인 지연
    return HttpResponse.json({ message: '현실적인 응답 시간' });
  }),

  // 조건부 지연
  http.get('/api/conditional', async ({ request }) => {
    const url = new URL(request.url);
    const slow = url.searchParams.get('slow') === 'true';

    if (slow) {
      await delay(3000);
    } else {
      await delay(100);
    }

    return HttpResponse.json({ slow });
  }),

  // 무한 지연 (타임아웃 테스트용)
  http.get('/api/timeout', async () => {
    await delay('infinite');
    return HttpResponse.json({ message: '이 응답은 도착하지 않음' });
  }),
];
```

### 네트워크 에러 시뮬레이션

```typescript
import { http, HttpResponse, delay } from 'msw';

export const networkErrorHandlers = [
  // 네트워크 연결 실패
  http.get('/api/offline', () => {
    return HttpResponse.error();
  }),

  // 확률적 네트워크 에러
  http.get('/api/flaky', () => {
    // 30% 확률로 네트워크 에러
    if (Math.random() < 0.3) {
      return HttpResponse.error();
    }
    return HttpResponse.json({ status: 'success' });
  }),

  // 타임아웃 시뮬레이션
  http.get('/api/timeout-test', async () => {
    // 클라이언트 타임아웃보다 긴 지연
    await delay(30000);
    return HttpResponse.json({ message: 'Too late' });
  }),

  // 점진적 실패 (재시도 테스트용)
  (() => {
    let attempts = 0;

    return http.get('/api/retry-test', () => {
      attempts++;

      // 처음 2번은 실패, 3번째부터 성공
      if (attempts < 3) {
        return HttpResponse.json(
          { error: 'Temporary failure', attempt: attempts },
          { status: 503 }
        );
      }

      attempts = 0; // 리셋
      return HttpResponse.json({ success: true, attempts: 3 });
    });
  })(),
];
```

### 조건부 핸들러와 once 옵션

```typescript
import { http, HttpResponse } from 'msw';

export const handlers = [
  // 한 번만 실행되는 핸들러
  http.get(
    '/api/one-time',
    () => {
      return HttpResponse.json({ special: 'first-response' });
    },
    { once: true }
  ),

  // 위 핸들러 실행 후 이 핸들러가 매칭됨
  http.get('/api/one-time', () => {
    return HttpResponse.json({ regular: 'response' });
  }),
];
```

## MSW 2.0 주요 변경사항

MSW 1.x에서 2.0으로 업그레이드할 때 알아야 할 주요 변경사항입니다.

### 환경 요구사항 변경

- Node.js: 최소 버전 18.0.0
- TypeScript: 최소 버전 4.7

### Import 경로 변경

```typescript
// MSW 1.x
import { setupWorker, rest } from 'msw';

// MSW 2.0
import { setupWorker } from 'msw/browser';
import { http, HttpResponse } from 'msw';
```

### 핸들러 API 변경

```typescript
// MSW 1.x
rest.get('/api/users', (req, res, ctx) => {
  return res(ctx.status(200), ctx.json({ users: [] }));
});

// MSW 2.0
http.get('/api/users', () => {
  return HttpResponse.json({ users: [] });
});
```

### 요청 접근 방식 변경

```typescript
// MSW 1.x
rest.get('/api/users/:id', (req, res, ctx) => {
  const { id } = req.params;
  const authHeader = req.headers.get('Authorization');
  const body = req.body;
  return res(ctx.json({ id }));
});

// MSW 2.0
http.get('/api/users/:id', async ({ params, request, cookies }) => {
  const { id } = params;
  const authHeader = request.headers.get('Authorization');
  const body = await request.json();
  return HttpResponse.json({ id });
});
```

### 주요 API 변경 매핑

| MSW 1.x | MSW 2.0 |
|---------|---------|
| `rest` | `http` |
| `res(ctx.json(...))` | `HttpResponse.json(...)` |
| `res(ctx.status(404))` | `HttpResponse.json(..., { status: 404 })` |
| `ctx.delay(1000)` | `await delay(1000)` |
| `ctx.fetch(req)` | `bypass(request)` |
| `res.once()` | `{ once: true }` 옵션 |
| `res.networkError()` | `HttpResponse.error()` |

## 실무 팁과 베스트 프랙티스

### 1. 핸들러 구조화

도메인별로 핸들러를 분리하여 관리합니다.

```
src/mocks/
  handlers/
    auth.ts
    users.ts
    posts.ts
    comments.ts
    index.ts      # 모든 핸들러 통합
  browser.ts
  node.ts
```

```typescript
// src/mocks/handlers/index.ts
import { authHandlers } from './auth';
import { userHandlers } from './users';
import { postHandlers } from './posts';

export const handlers = [
  ...authHandlers,
  ...userHandlers,
  ...postHandlers,
];
```

### 2. 타입 안전성 확보

```typescript
import { http, HttpResponse } from 'msw';
import type { User, CreateUserRequest } from '../types';

// 요청/응답 타입 명시
http.post<never, CreateUserRequest, User>('/api/users', async ({ request }) => {
  const body = await request.json();
  // body는 CreateUserRequest 타입

  const user: User = {
    id: 1,
    name: body.name,
    email: body.email,
  };

  return HttpResponse.json(user);
});
```

### 3. 실제 API 스펙과 동기화

OpenAPI/Swagger 스펙을 기반으로 핸들러를 생성하면 실제 API와의 일관성을 유지할 수 있습니다.

```typescript
// 실제 API 응답 구조와 동일하게 모킹
interface ApiResponse<T> {
  success: boolean;
  data: T;
  meta?: {
    timestamp: string;
    requestId: string;
  };
}

function createApiResponse<T>(data: T): ApiResponse<T> {
  return {
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
    },
  };
}

http.get('/api/users', () => {
  return HttpResponse.json(createApiResponse(users));
});
```

### 4. 개발 환경에서 디버깅

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);

// 디버깅용 이벤트 리스너
worker.events.on('request:start', ({ request }) => {
  console.log('MSW 요청 시작:', request.method, request.url);
});

worker.events.on('request:match', ({ request }) => {
  console.log('MSW 핸들러 매칭:', request.method, request.url);
});

worker.events.on('request:unhandled', ({ request }) => {
  console.warn('MSW 미처리 요청:', request.method, request.url);
});
```

### 5. 테스트에서 핸들러 오버라이드

```typescript
import { server } from '../mocks/node';
import { http, HttpResponse } from 'msw';

describe('에러 처리', () => {
  it('네트워크 에러 시 재시도 로직이 동작한다', async () => {
    let attempts = 0;

    server.use(
      http.get('/api/data', () => {
        attempts++;
        if (attempts < 3) {
          return HttpResponse.error();
        }
        return HttpResponse.json({ success: true });
      })
    );

    // 테스트 로직...
    expect(attempts).toBe(3);
  });
});
```

### 6. 환경별 설정 분리

```typescript
// src/mocks/browser.ts
export const worker = setupWorker(...handlers);

export const startWorker = () => {
  return worker.start({
    onUnhandledRequest(request, print) {
      // 정적 자원 요청은 무시
      if (request.url.includes('/assets/') || request.url.includes('.svg')) {
        return;
      }

      // 개발 환경에서는 경고만
      if (import.meta.env.DEV) {
        print.warning();
      }
    },
  });
};
```

### 7. Storybook과 함께 사용

MSW는 Storybook에서 API 모킹에 매우 유용합니다. 개발, 테스트, Storybook 환경에서 동일한 핸들러를 재사용할 수 있습니다.

> Storybook을 활용한 컴포넌트 문서화에 대해서는 [Storybook 컴포넌트 문서화 완벽 가이드](/posts/storybook-component-documentation-guide/)를 참고하세요.
{: .prompt-tip }

## 마치며

MSW는 프론트엔드 개발 워크플로우를 크게 개선할 수 있는 강력한 도구입니다. API 개발이 완료되기 전에 프론트엔드 개발을 시작할 수 있고, 테스트 환경에서 네트워크 요청을 완벽하게 제어할 수 있습니다.

**MSW 도입의 핵심 이점**:

1. **개발 속도 향상**: 백엔드 API 완성을 기다리지 않고 개발 가능
2. **테스트 신뢰성**: 네트워크 레벨에서 모킹하여 실제 환경과 유사한 테스트
3. **재사용성**: 개발, 테스트, Storybook 등에서 동일한 핸들러 사용
4. **타입 안전성**: TypeScript와 완벽하게 통합

**시작 팁**:

1. 먼저 간단한 GET 요청부터 모킹해보세요
2. 점진적으로 핸들러를 추가하세요
3. 실제 API 스펙과 동기화를 유지하세요
4. 테스트 환경에서 적극 활용하세요

MSW를 활용하면 프론트엔드 개발과 테스트가 한층 수월해집니다. 특히 팀 규모가 크고 백엔드와 프론트엔드가 병렬로 개발되는 환경에서 그 진가를 발휘합니다.

## 참고 자료

### 공식 문서

- [MSW 공식 문서](https://mswjs.io/docs/)
- [MSW GitHub 저장소](https://github.com/mswjs/msw)
- [MSW Quick Start](https://mswjs.io/docs/quick-start/)
- [MSW 1.x to 2.0 마이그레이션 가이드](https://mswjs.io/docs/migrations/1.x-to-2.x)

### 관련 학습 자료

- [Kent C. Dodds - Stop Mocking fetch](https://kentcdodds.com/blog/stop-mocking-fetch)
- [Testing Library 공식 문서](https://testing-library.com/)
- [Vitest 공식 문서](https://vitest.dev/)

### 관련 포스트

- [프론트엔드 테스트 완벽 가이드](/posts/frontend-testing-guide/) - Jest와 React Testing Library로 시작하기
- [TDD 실전 가이드](/posts/tdd-practical-guide/) - Red-Green-Refactor 사이클로 배우는 개발 방법론
- [E2E 테스트 실전 가이드](/posts/e2e-testing-guide/) - Playwright와 Cypress로 사용자 시나리오 테스트
- [Storybook 컴포넌트 문서화 완벽 가이드](/posts/storybook-component-documentation-guide/) - 디자인 시스템과 연계한 UI 컴포넌트 개발
- [Vite 완벽 가이드](/posts/vite-complete-guide/) - 차세대 번들러로 개발 경험 혁신하기
