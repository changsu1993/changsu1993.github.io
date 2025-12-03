---
title: "React Error Boundary 완벽 가이드: 에러 처리와 복구 전략"
description: "React Error Boundary의 핵심 개념부터 고급 패턴까지. 클래스 컴포넌트 구현, react-error-boundary 라이브러리, 에러 복구 전략을 실전 예제와 함께 알아봅니다."
date: 2025-12-03 09:00:00 +0900
categories: [Frontend, React]
tags: [react, error-boundary, error-handling, react-error-boundary, typescript]
---

## 개요

React 애플리케이션에서 예상치 못한 에러가 발생하면 전체 UI가 크래시되어 사용자에게 빈 화면만 보여줄 수 있습니다. **Error Boundary**는 이러한 상황을 방지하고, 에러가 발생한 부분만 우아하게 처리하여 나머지 애플리케이션은 정상적으로 동작하도록 합니다.

이 글에서는 Error Boundary의 기초부터 고급 복구 전략까지 실전 예제와 함께 상세히 알아봅니다.

---

## 에러 처리가 중요한 이유

### JavaScript 에러와 UI 크래시

React 16 이전에는 컴포넌트 내부의 JavaScript 에러가 React의 내부 상태를 손상시켜 이후 렌더링에서 알 수 없는 에러를 발생시켰습니다. React 16부터는 **에러가 발생하면 전체 컴포넌트 트리가 언마운트**됩니다.

```tsx
function BuggyComponent() {
  // 이 에러는 전체 앱을 크래시시킴
  throw new Error('예상치 못한 에러!');
  return <div>절대 렌더링되지 않음</div>;
}

function App() {
  return (
    <div>
      <Header />
      <BuggyComponent /> {/* 여기서 에러 발생 */}
      <Footer />
    </div>
  );
}
// 결과: 전체 앱이 빈 화면으로 크래시
```

### Error Boundary의 역할

Error Boundary는 **하위 컴포넌트 트리에서 발생한 JavaScript 에러를 캐치**하고, 에러를 로깅하며, **폴백 UI를 표시**합니다.

| 역할 | 설명 |
|------|------|
| 에러 캐치 | 렌더링 중 발생한 에러를 잡음 |
| 폴백 UI | 크래시 대신 대체 UI 표시 |
| 에러 로깅 | 에러 정보를 서비스에 전송 |
| 앱 보호 | 나머지 앱은 정상 동작 유지 |

---

## Error Boundary 구현하기

### 클래스 컴포넌트로 구현

현재 React에서 Error Boundary는 **클래스 컴포넌트로만 구현**할 수 있습니다. 두 가지 생명주기 메서드를 사용합니다.

```tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
    };
  }

  // 에러 발생 시 state 업데이트
  static getDerivedStateFromError(error: Error): State {
    return {
      hasError: true,
      error,
    };
  }

  // 에러 정보 로깅
  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    console.error('에러 발생:', error);
    console.error('컴포넌트 스택:', errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // 폴백 UI 렌더링
      return this.props.fallback || (
        <div className="error-fallback">
          <h2>문제가 발생했습니다</h2>
          <p>잠시 후 다시 시도해주세요.</p>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

### 사용 예시

```tsx
import ErrorBoundary from './ErrorBoundary';

function App() {
  return (
    <div className="app">
      <Header />

      <ErrorBoundary fallback={<p>위젯을 불러올 수 없습니다.</p>}>
        <DashboardWidget />
      </ErrorBoundary>

      <ErrorBoundary fallback={<p>사이드바를 불러올 수 없습니다.</p>}>
        <Sidebar />
      </ErrorBoundary>

      <Footer />
    </div>
  );
}
```

---

## getDerivedStateFromError vs componentDidCatch

두 메서드는 서로 다른 목적으로 사용됩니다.

### getDerivedStateFromError

**렌더링 단계**에서 호출되며, 폴백 UI를 렌더링하기 위한 state를 반환합니다.

```tsx
static getDerivedStateFromError(error: Error): State {
  // 반드시 state 객체를 반환해야 함
  return {
    hasError: true,
    error,
  };
}
```

**특징:**
- 순수 함수여야 함 (부수 효과 없음)
- 에러 객체만 받음
- 폴백 UI 표시에 사용

### componentDidCatch

**커밋 단계**에서 호출되며, 에러 로깅 등의 부수 효과를 처리합니다.

```tsx
componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
  // 에러 로깅 서비스에 전송
  logErrorToService(error, {
    componentStack: errorInfo.componentStack,
    timestamp: new Date().toISOString(),
    userAgent: navigator.userAgent,
    url: window.location.href,
  });
}
```

**특징:**
- 부수 효과 허용 (API 호출 등)
- `errorInfo.componentStack` 으로 컴포넌트 스택 추적 가능
- 에러 리포팅에 사용

### 비교 표

| 특성 | getDerivedStateFromError | componentDidCatch |
|------|--------------------------|-------------------|
| 호출 시점 | 렌더링 단계 | 커밋 단계 |
| 목적 | 폴백 UI 렌더링 | 에러 로깅 |
| 부수 효과 | 불가능 | 가능 |
| 매개변수 | error | error, errorInfo |
| 반환값 | state 객체 | void |

---

## Error Boundary가 잡지 못하는 에러

Error Boundary는 **모든 에러를 잡지 못합니다**. 다음 상황의 에러는 별도로 처리해야 합니다.

### 1. 이벤트 핸들러

```tsx
function Button() {
  const handleClick = () => {
    // Error Boundary가 잡지 못함!
    throw new Error('클릭 에러');
  };

  return <button onClick={handleClick}>클릭</button>;
}

// 해결책: try-catch 사용
function SafeButton() {
  const [error, setError] = useState<Error | null>(null);

  const handleClick = () => {
    try {
      riskyOperation();
    } catch (err) {
      setError(err as Error);
      // 또는 에러 로깅
      logError(err);
    }
  };

  if (error) {
    return <p>에러 발생: {error.message}</p>;
  }

  return <button onClick={handleClick}>클릭</button>;
}
```

### 2. 비동기 코드

```tsx
function AsyncComponent() {
  useEffect(() => {
    // Error Boundary가 잡지 못함!
    setTimeout(() => {
      throw new Error('타이머 에러');
    }, 1000);
  }, []);

  return <div>비동기 컴포넌트</div>;
}

// 해결책: async/await과 state 활용
function SafeAsyncComponent() {
  const [error, setError] = useState<Error | null>(null);
  const [data, setData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const response = await fetch('/api/data');
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err as Error);
      }
    }
    fetchData();
  }, []);

  if (error) {
    return <p>데이터를 불러올 수 없습니다.</p>;
  }

  return <div>{data}</div>;
}
```

### 3. 서버 사이드 렌더링

SSR 환경에서 발생하는 에러는 서버에서 별도로 처리해야 합니다.

### 4. Error Boundary 자체의 에러

Error Boundary 컴포넌트 내부에서 발생한 에러는 자기 자신이 잡지 못합니다.

```tsx
class BrokenErrorBoundary extends Component {
  render() {
    if (this.state.hasError) {
      // 이 에러는 잡히지 않음!
      throw new Error('폴백 렌더링 에러');
    }
    return this.props.children;
  }
}
```

### 에러 유형별 처리 방법

| 에러 유형 | Error Boundary | 해결 방법 |
|-----------|----------------|-----------|
| 렌더링 에러 | ✅ 캐치 | Error Boundary |
| 생명주기 메서드 | ✅ 캐치 | Error Boundary |
| 이벤트 핸들러 | ❌ 못잡음 | try-catch |
| 비동기 코드 | ❌ 못잡음 | try-catch + state |
| SSR | ❌ 못잡음 | 서버 에러 핸들링 |

---

## react-error-boundary 라이브러리

함수형 컴포넌트에서 더 쉽게 Error Boundary를 사용할 수 있게 해주는 라이브러리입니다.

### 설치

```bash
npm install react-error-boundary
```

### 기본 사용법

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert" className="error-fallback">
      <h2>문제가 발생했습니다</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>다시 시도</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => {
        // 에러 로깅
        logErrorToService(error, info);
      }}
      onReset={() => {
        // 리셋 시 실행할 로직
        // 예: 캐시 초기화, 상태 리셋
      }}
    >
      <Dashboard />
    </ErrorBoundary>
  );
}
```

### 폴백 렌더링 방법

#### 1. fallback prop (간단한 JSX)

```tsx
<ErrorBoundary fallback={<p>에러가 발생했습니다.</p>}>
  <Component />
</ErrorBoundary>
```

#### 2. FallbackComponent (컴포넌트)

```tsx
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <p>에러: {error.message}</p>
      <button onClick={resetErrorBoundary}>재시도</button>
    </div>
  );
}

<ErrorBoundary FallbackComponent={ErrorFallback}>
  <Component />
</ErrorBoundary>
```

#### 3. fallbackRender (렌더 프롭)

```tsx
<ErrorBoundary
  fallbackRender={({ error, resetErrorBoundary }) => (
    <div>
      <p>에러: {error.message}</p>
      <button onClick={resetErrorBoundary}>재시도</button>
    </div>
  )}
>
  <Component />
</ErrorBoundary>
```

### useErrorBoundary 훅

이벤트 핸들러나 비동기 코드에서 발생한 에러를 Error Boundary로 전달할 수 있습니다.

```tsx
import { useErrorBoundary } from 'react-error-boundary';

function DataFetcher() {
  const { showBoundary } = useErrorBoundary();
  const [data, setData] = useState(null);

  const fetchData = async () => {
    try {
      const response = await fetch('/api/data');
      if (!response.ok) {
        throw new Error('데이터를 불러올 수 없습니다');
      }
      const result = await response.json();
      setData(result);
    } catch (error) {
      // Error Boundary로 에러 전달
      showBoundary(error);
    }
  };

  return (
    <div>
      <button onClick={fetchData}>데이터 불러오기</button>
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}

// 사용
function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <DataFetcher />
    </ErrorBoundary>
  );
}
```

### withErrorBoundary HOC

```tsx
import { withErrorBoundary } from 'react-error-boundary';

function Dashboard() {
  return <div>대시보드 컨텐츠</div>;
}

const DashboardWithErrorBoundary = withErrorBoundary(Dashboard, {
  FallbackComponent: ErrorFallback,
  onError: (error, info) => {
    logErrorToService(error, info);
  },
});

export default DashboardWithErrorBoundary;
```

---

## 에러 복구 전략

### 1. 리셋 버튼

사용자가 직접 재시도할 수 있게 합니다.

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="p-4 bg-red-50 rounded-lg">
      <h3 className="text-red-800 font-bold">오류 발생</h3>
      <p className="text-red-600">{error.message}</p>
      <button
        onClick={resetErrorBoundary}
        className="mt-2 px-4 py-2 bg-red-500 text-white rounded"
      >
        다시 시도
      </button>
    </div>
  );
}

function App() {
  const [queryKey, setQueryKey] = useState(0);

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // 상태 초기화
        setQueryKey((prev) => prev + 1);
      }}
      resetKeys={[queryKey]} // 이 값이 변경되면 자동 리셋
    >
      <DataComponent key={queryKey} />
    </ErrorBoundary>
  );
}
```

### 2. 자동 재시도

특정 횟수만큼 자동으로 재시도합니다.

```tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  maxRetries?: number;
  onError?: (error: Error, retryCount: number) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
  retryCount: number;
}

class RetryErrorBoundary extends Component<Props, State> {
  static defaultProps = {
    maxRetries: 3,
  };

  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      retryCount: 0,
    };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error) {
    const { maxRetries, onError } = this.props;
    const { retryCount } = this.state;

    if (onError) {
      onError(error, retryCount);
    }

    // 자동 재시도
    if (retryCount < maxRetries!) {
      setTimeout(() => {
        this.setState((prev) => ({
          hasError: false,
          error: null,
          retryCount: prev.retryCount + 1,
        }));
      }, 1000 * (retryCount + 1)); // 점진적 대기
    }
  }

  render() {
    const { hasError, error, retryCount } = this.state;
    const { maxRetries, children } = this.props;

    if (hasError) {
      if (retryCount >= maxRetries!) {
        return (
          <div className="error-fallback">
            <h3>문제가 지속됩니다</h3>
            <p>{error?.message}</p>
            <button onClick={() => window.location.reload()}>
              페이지 새로고침
            </button>
          </div>
        );
      }

      return (
        <div className="loading">
          재시도 중... ({retryCount + 1}/{maxRetries})
        </div>
      );
    }

    return children;
  }
}
```

### 3. 폴백 컨텐츠 제공

에러 시 대체 컨텐츠를 보여줍니다.

```tsx
function UserProfileFallback() {
  return (
    <div className="user-profile-skeleton">
      <div className="avatar-placeholder" />
      <p>프로필을 불러올 수 없습니다.</p>
      <a href="/login">다시 로그인하기</a>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary fallback={<UserProfileFallback />}>
      <UserProfile />
    </ErrorBoundary>
  );
}
```

### 4. 부분적 기능 유지

에러가 발생해도 핵심 기능은 유지합니다.

```tsx
function Dashboard() {
  return (
    <div className="dashboard">
      {/* 핵심 기능 - 에러 시 전체 폴백 */}
      <ErrorBoundary fallback={<CriticalErrorPage />}>
        <MainContent />
      </ErrorBoundary>

      {/* 부가 기능 - 에러 시 숨김 */}
      <ErrorBoundary fallback={null}>
        <RecommendationWidget />
      </ErrorBoundary>

      {/* 부가 기능 - 에러 시 간단한 메시지 */}
      <ErrorBoundary fallback={<p>알림을 불러올 수 없습니다.</p>}>
        <NotificationPanel />
      </ErrorBoundary>
    </div>
  );
}
```

---

## 에러 로깅과 모니터링

### 기본 에러 로깅

```tsx
interface ErrorLog {
  message: string;
  stack?: string;
  componentStack?: string;
  timestamp: string;
  userAgent: string;
  url: string;
  userId?: string;
}

function logErrorToService(error: Error, errorInfo?: { componentStack?: string }) {
  const errorLog: ErrorLog = {
    message: error.message,
    stack: error.stack,
    componentStack: errorInfo?.componentStack,
    timestamp: new Date().toISOString(),
    userAgent: navigator.userAgent,
    url: window.location.href,
    userId: getCurrentUserId(), // 사용자 ID 가져오기
  };

  // 에러 로깅 서비스로 전송
  fetch('/api/log-error', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(errorLog),
  }).catch(console.error);
}
```

### Sentry 연동

```tsx
import * as Sentry from '@sentry/react';

// 초기화
Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  environment: process.env.NODE_ENV,
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
  tracesSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});

// Sentry Error Boundary 사용
function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={({ error, resetError }) => (
        <div>
          <h2>에러가 발생했습니다</h2>
          <button onClick={resetError}>다시 시도</button>
        </div>
      )}
      onError={(error, componentStack) => {
        // 추가 컨텍스트 설정
        Sentry.setContext('component', {
          stack: componentStack,
        });
      }}
    >
      <Router />
    </Sentry.ErrorBoundary>
  );
}
```

### 커스텀 에러 리포터

```tsx
class ErrorReporter {
  private static instance: ErrorReporter;
  private errorQueue: ErrorLog[] = [];
  private isOnline = navigator.onLine;

  private constructor() {
    window.addEventListener('online', () => {
      this.isOnline = true;
      this.flushQueue();
    });
    window.addEventListener('offline', () => {
      this.isOnline = false;
    });
  }

  static getInstance() {
    if (!ErrorReporter.instance) {
      ErrorReporter.instance = new ErrorReporter();
    }
    return ErrorReporter.instance;
  }

  report(error: Error, context?: Record<string, unknown>) {
    const errorLog: ErrorLog = {
      message: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      ...context,
    };

    if (this.isOnline) {
      this.sendError(errorLog);
    } else {
      this.errorQueue.push(errorLog);
      this.saveToLocalStorage();
    }
  }

  private async sendError(errorLog: ErrorLog) {
    try {
      await fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorLog),
      });
    } catch {
      this.errorQueue.push(errorLog);
      this.saveToLocalStorage();
    }
  }

  private flushQueue() {
    const queue = [...this.errorQueue];
    this.errorQueue = [];
    queue.forEach((error) => this.sendError(error));
  }

  private saveToLocalStorage() {
    localStorage.setItem('errorQueue', JSON.stringify(this.errorQueue));
  }
}

export const errorReporter = ErrorReporter.getInstance();
```

---

## 실전 예제

### 1. 전역 Error Boundary 설정

```tsx
import { ErrorBoundary } from 'react-error-boundary';
import { useNavigate } from 'react-router-dom';

function GlobalErrorFallback({ error, resetErrorBoundary }) {
  const navigate = useNavigate();

  const handleGoHome = () => {
    navigate('/');
    resetErrorBoundary();
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md p-8 bg-white rounded-lg shadow-lg text-center">
        <div className="text-6xl mb-4">😕</div>
        <h1 className="text-2xl font-bold text-gray-800 mb-2">
          문제가 발생했습니다
        </h1>
        <p className="text-gray-600 mb-6">
          예상치 못한 오류가 발생했습니다.
          불편을 드려 죄송합니다.
        </p>

        {process.env.NODE_ENV === 'development' && (
          <pre className="text-left text-sm bg-gray-100 p-4 rounded mb-4 overflow-auto">
            {error.message}
          </pre>
        )}

        <div className="space-x-4">
          <button
            onClick={resetErrorBoundary}
            className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
          >
            다시 시도
          </button>
          <button
            onClick={handleGoHome}
            className="px-4 py-2 border border-gray-300 rounded hover:bg-gray-50"
          >
            홈으로 이동
          </button>
        </div>
      </div>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={GlobalErrorFallback}
      onError={(error, info) => {
        errorReporter.report(error, {
          componentStack: info.componentStack,
          type: 'global',
        });
      }}
    >
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </BrowserRouter>
    </ErrorBoundary>
  );
}
```

### 2. 페이지별 Error Boundary

```tsx
import { ErrorBoundary } from 'react-error-boundary';
import { Suspense } from 'react';

function PageErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="p-8 text-center">
      <h2 className="text-xl font-bold text-red-600 mb-4">
        페이지를 불러올 수 없습니다
      </h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button
        onClick={resetErrorBoundary}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        다시 시도
      </button>
    </div>
  );
}

function PageWrapper({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundary
      FallbackComponent={PageErrorFallback}
      onReset={() => {
        // 페이지 상태 초기화 로직
      }}
    >
      <Suspense fallback={<PageSkeleton />}>
        {children}
      </Suspense>
    </ErrorBoundary>
  );
}

// 라우터에서 사용
function AppRoutes() {
  return (
    <Routes>
      <Route
        path="/dashboard"
        element={
          <PageWrapper>
            <Dashboard />
          </PageWrapper>
        }
      />
      <Route
        path="/settings"
        element={
          <PageWrapper>
            <Settings />
          </PageWrapper>
        }
      />
    </Routes>
  );
}
```

### 3. 위젯별 독립적인 Error Boundary

```tsx
function WidgetErrorFallback({ error, resetErrorBoundary, widgetName }) {
  return (
    <div className="p-4 border border-red-200 rounded-lg bg-red-50">
      <div className="flex items-center justify-between">
        <span className="text-red-600 text-sm">
          {widgetName} 로드 실패
        </span>
        <button
          onClick={resetErrorBoundary}
          className="text-sm text-blue-500 hover:underline"
        >
          재시도
        </button>
      </div>
    </div>
  );
}

function createWidgetBoundary(WidgetComponent: React.ComponentType, name: string) {
  return function WidgetWithBoundary(props: any) {
    return (
      <ErrorBoundary
        fallbackRender={({ error, resetErrorBoundary }) => (
          <WidgetErrorFallback
            error={error}
            resetErrorBoundary={resetErrorBoundary}
            widgetName={name}
          />
        )}
        onError={(error) => {
          errorReporter.report(error, { widget: name });
        }}
      >
        <WidgetComponent {...props} />
      </ErrorBoundary>
    );
  };
}

// 사용
const SafeWeatherWidget = createWidgetBoundary(WeatherWidget, '날씨');
const SafeStockWidget = createWidgetBoundary(StockWidget, '주식');
const SafeNewsWidget = createWidgetBoundary(NewsWidget, '뉴스');

function Dashboard() {
  return (
    <div className="grid grid-cols-3 gap-4">
      <SafeWeatherWidget />
      <SafeStockWidget />
      <SafeNewsWidget />
    </div>
  );
}
```

### 4. 비동기 에러 처리 통합

```tsx
import { useErrorBoundary, ErrorBoundary } from 'react-error-boundary';
import { useQuery } from '@tanstack/react-query';

function useQueryWithErrorBoundary<T>(
  queryKey: string[],
  queryFn: () => Promise<T>
) {
  const { showBoundary } = useErrorBoundary();

  return useQuery({
    queryKey,
    queryFn,
    throwOnError: true, // React Query v5
    // 또는 수동으로 에러 전파
    // onError: (error) => showBoundary(error),
  });
}

function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useQueryWithErrorBoundary(
    ['user', userId],
    () => fetchUser(userId)
  );

  if (isLoading) {
    return <ProfileSkeleton />;
  }

  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}

function ProfilePage({ userId }: { userId: string }) {
  return (
    <ErrorBoundary
      fallbackRender={({ error, resetErrorBoundary }) => (
        <div className="error">
          <p>프로필을 불러올 수 없습니다: {error.message}</p>
          <button onClick={resetErrorBoundary}>다시 시도</button>
        </div>
      )}
      onReset={() => {
        // React Query 캐시 무효화
        queryClient.invalidateQueries({ queryKey: ['user', userId] });
      }}
    >
      <UserProfile userId={userId} />
    </ErrorBoundary>
  );
}
```

---

## Error Boundary 배치 전략

### 계층적 Error Boundary

```
App
└─ GlobalErrorBoundary (전역: 치명적 에러)
   ├─ Header
   ├─ PageErrorBoundary (페이지별)
   │  └─ Dashboard
   │     ├─ WidgetErrorBoundary
   │     │  └─ ChartWidget
   │     ├─ WidgetErrorBoundary
   │     │  └─ TableWidget
   │     └─ WidgetErrorBoundary
   │        └─ StatsWidget
   └─ Footer
```

### 권장 배치 위치

| 위치 | 목적 | 폴백 UI |
|------|------|---------|
| 앱 최상위 | 전역 에러 캐치 | 전체 에러 페이지 |
| 라우트/페이지 | 페이지별 격리 | 페이지 에러 메시지 |
| 독립 위젯 | 위젯별 격리 | 위젯 에러 표시 |
| 데이터 로딩 영역 | 로딩 실패 처리 | 재시도 버튼 |
| 서드파티 컴포넌트 | 외부 라이브러리 격리 | 대체 UI |

---

## 자주 묻는 질문 (FAQ)

### Q1. 함수형 컴포넌트로 Error Boundary를 만들 수 있나요?

**A:** 현재 React에서는 **클래스 컴포넌트로만** Error Boundary를 만들 수 있습니다. `getDerivedStateFromError`와 `componentDidCatch`는 클래스 컴포넌트 생명주기 메서드입니다.

```tsx
// ❌ 불가능
function ErrorBoundary({ children }) {
  // getDerivedStateFromError를 훅으로 사용할 수 없음
}

// ✅ 가능
class ErrorBoundary extends Component {
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
}
```

함수형 컴포넌트에서 사용하려면 `react-error-boundary` 라이브러리를 사용하세요.

---

### Q2. Error Boundary를 어디에 배치해야 하나요?

**A:** 애플리케이션의 구조와 요구사항에 따라 다릅니다.

**권장 패턴:**
1. **앱 루트**: 최후의 방어선으로 전역 Error Boundary
2. **라우트별**: 각 페이지를 독립적으로 격리
3. **위젯별**: 실패해도 다른 부분에 영향 없도록

```tsx
// 계층적 배치
<GlobalErrorBoundary>
  <Routes>
    <Route
      path="/dashboard"
      element={
        <PageErrorBoundary>
          <Dashboard />
        </PageErrorBoundary>
      }
    />
  </Routes>
</GlobalErrorBoundary>
```

---

### Q3. 이벤트 핸들러 에러는 어떻게 처리하나요?

**A:** Error Boundary는 이벤트 핸들러 에러를 잡지 못합니다. `useErrorBoundary` 훅이나 try-catch를 사용하세요.

```tsx
import { useErrorBoundary } from 'react-error-boundary';

function Button() {
  const { showBoundary } = useErrorBoundary();

  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      showBoundary(error); // Error Boundary로 전달
    }
  };

  return <button onClick={handleClick}>클릭</button>;
}
```

---

### Q4. 개발 환경에서 Error Boundary가 작동하지 않는 것 같아요

**A:** React 개발 모드에서는 에러가 발생하면 **에러 오버레이**가 먼저 표시됩니다. ESC를 누르면 Error Boundary의 폴백 UI를 확인할 수 있습니다.

프로덕션 빌드에서는 에러 오버레이 없이 바로 폴백 UI가 표시됩니다.

---

### Q5. 에러 발생 후 컴포넌트를 리셋하려면?

**A:** `react-error-boundary`의 `resetErrorBoundary` 함수나 `resetKeys` prop을 사용합니다.

```tsx
function App() {
  const [retryKey, setRetryKey] = useState(0);

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => setRetryKey((k) => k + 1)}
      resetKeys={[retryKey]} // 이 값이 변경되면 자동 리셋
    >
      <Component key={retryKey} />
    </ErrorBoundary>
  );
}
```

---

### Q6. 중첩된 Error Boundary는 어떻게 동작하나요?

**A:** 에러는 **가장 가까운 상위 Error Boundary**에서 잡힙니다.

```tsx
<OuterErrorBoundary>
  <InnerErrorBoundary>
    <BuggyComponent /> {/* InnerErrorBoundary가 잡음 */}
  </InnerErrorBoundary>
</OuterErrorBoundary>
```

내부 Error Boundary가 에러를 잡으면 외부로 전파되지 않습니다.

---

### Q7. TypeScript에서 Error Boundary를 어떻게 타입 지정하나요?

**A:** 다음과 같이 타입을 정의합니다.

```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  state: State = {
    hasError: false,
    error: null,
  };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <DefaultFallback />;
    }
    return this.props.children;
  }
}
```

---

### Q8. Server Components에서 Error Boundary는 어떻게 사용하나요?

**A:** Next.js App Router에서는 `error.tsx` 파일을 사용합니다.

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>문제가 발생했습니다</h2>
      <button onClick={() => reset()}>다시 시도</button>
    </div>
  );
}
```

Server Components에서 발생한 에러도 클라이언트의 가장 가까운 `error.tsx`에서 처리됩니다.

---

## 마무리

Error Boundary는 React 애플리케이션의 안정성을 높이는 핵심 패턴입니다.

### 핵심 포인트 정리

| 개념 | 설명 |
|------|------|
| Error Boundary | 하위 컴포넌트의 렌더링 에러를 캐치 |
| getDerivedStateFromError | 폴백 UI 렌더링용 state 업데이트 |
| componentDidCatch | 에러 로깅 및 부수 효과 |
| react-error-boundary | 함수형 컴포넌트 친화적 라이브러리 |
| useErrorBoundary | 이벤트/비동기 에러를 Boundary로 전달 |
| resetKeys | 특정 값 변경 시 자동 리셋 |

### 권장 사항

1. **계층적 배치**: 전역, 페이지, 위젯 레벨로 Error Boundary 배치
2. **적절한 폴백**: 에러 수준에 맞는 폴백 UI 제공
3. **에러 로깅**: Sentry 등 모니터링 서비스 연동
4. **복구 전략**: 사용자가 재시도할 수 있는 방법 제공
5. **테스트**: 에러 상황을 시뮬레이션하여 폴백 UI 테스트

Error Boundary를 통해 예상치 못한 에러에도 우아하게 대응하고, 사용자 경험을 보호하세요.

---

## 참고 자료

- [React 공식 문서 - Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [react-error-boundary GitHub](https://github.com/bvaughn/react-error-boundary)
- [Sentry React SDK](https://docs.sentry.io/platforms/javascript/guides/react/)
