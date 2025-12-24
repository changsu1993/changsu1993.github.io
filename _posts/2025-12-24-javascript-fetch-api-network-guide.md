---
title: "JavaScript Fetch API 완벽 가이드 - HTTP 요청부터 에러 처리, 인터셉터 패턴까지"
description: "Fetch API의 GET/POST 요청부터 에러 처리, AbortController 타임아웃, 재시도 로직, 인터셉터 패턴까지 TypeScript 예제와 함께 실무에서 바로 적용 가능한 네트워크 통신 완벽 가이드입니다."
date: 2025-12-24 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, Fetch-API, HTTP, REST-API, AbortController, 네트워크, 비동기, TypeScript, 에러처리, 타임아웃, 인터셉터, CORS, Promise, async-await, XMLHttpRequest]
---

## 개요

웹 애플리케이션에서 서버와 통신하는 것은 가장 기본적이면서도 중요한 기능입니다. **Fetch API**는 XMLHttpRequest를 대체하는 모던 JavaScript의 네트워크 요청 표준으로, Promise 기반의 깔끔한 인터페이스를 제공합니다.

이 글에서는 Fetch API의 기본 사용법부터 에러 처리, 요청 취소, 재시도 로직, 인터셉터 패턴까지 실무에서 필요한 모든 내용을 다룹니다.

### 학습 목표

- Fetch API의 기본 문법과 HTTP 메서드별 사용법 이해
- Response 객체와 다양한 응답 형식 처리 방법 학습
- AbortController를 활용한 요청 취소와 타임아웃 구현
- 재시도 로직과 인터셉터 패턴 등 고급 패턴 습득
- TypeScript와 함께 타입 안전하게 사용하는 방법 익히기

### 사전 지식

- JavaScript 기초 문법
- [Promise와 async/await](/posts/javascript-promise-async-await-guide/)
- [JavaScript Event Loop](/posts/javascript-event-loop/) - 비동기 처리 메커니즘 이해
- HTTP 기본 개념 (GET, POST, 상태 코드 등)

---

## XMLHttpRequest vs Fetch API

Fetch API가 등장하기 전에는 **XMLHttpRequest(XHR)**가 브라우저에서 HTTP 요청을 보내는 유일한 방법이었습니다.

### XMLHttpRequest의 문제점

```javascript
// XMLHttpRequest 방식 - 콜백 기반
const xhr = new XMLHttpRequest();

xhr.open('GET', 'https://api.example.com/users');

xhr.onload = function() {
  if (xhr.status === 200) {
    const data = JSON.parse(xhr.responseText);
    console.log(data);
  } else {
    console.error('요청 실패:', xhr.status);
  }
};

xhr.onerror = function() {
  console.error('네트워크 에러');
};

xhr.send();
```

### Fetch API의 장점

```javascript
// Fetch API 방식 - Promise 기반
fetch('https://api.example.com/users')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  })
  .then(data => console.log(data))
  .catch(error => console.error('에러:', error));
```

### 비교 표

| 특성 | XMLHttpRequest | Fetch API |
|------|---------------|-----------|
| 인터페이스 | 콜백 기반 | Promise 기반 |
| 문법 | 복잡함 | 간결함 |
| 스트리밍 | 제한적 | ReadableStream 지원 |
| 요청 취소 | abort() 메서드 | AbortController |
| 진행률 추적 | onprogress 이벤트 | 직접 구현 필요 |
| 쿠키 전송 | 기본 포함 | credentials 옵션 필요 |
| 크로스 오리진 | withCredentials | mode, credentials 옵션 |

> Fetch API는 **HTTP 에러 상태(4xx, 5xx)**에서도 reject되지 않습니다. 네트워크 장애 같은 요청 자체가 실패한 경우에만 reject됩니다. 이 점이 면접에서 자주 나오는 포인트입니다.
{: .prompt-warning }

---

## Fetch API 기본 사용법

### GET 요청

가장 기본적인 데이터 조회 요청입니다.

```javascript
// 기본 GET 요청
async function getUsers() {
  try {
    const response = await fetch('https://api.example.com/users');

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const users = await response.json();
    console.log(users);
    return users;
  } catch (error) {
    console.error('사용자 조회 실패:', error);
    throw error;
  }
}

// 쿼리 파라미터 추가
async function getUsersWithParams(page, limit) {
  const params = new URLSearchParams({
    page: page.toString(),
    limit: limit.toString(),
    sort: 'createdAt'
  });

  const response = await fetch(`https://api.example.com/users?${params}`);
  return response.json();
}
```

### POST 요청

새로운 리소스를 생성할 때 사용합니다.

```javascript
async function createUser(userData) {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.message || '사용자 생성 실패');
    }

    const newUser = await response.json();
    console.log('생성된 사용자:', newUser);
    return newUser;
  } catch (error) {
    console.error('사용자 생성 에러:', error);
    throw error;
  }
}

// 사용 예시
createUser({
  name: '홍길동',
  email: 'hong@example.com',
  age: 30
});
```

### PUT 요청

기존 리소스를 완전히 교체할 때 사용합니다.

```javascript
async function updateUser(userId, userData) {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(userData),
  });

  if (!response.ok) {
    throw new Error(`사용자 수정 실패: ${response.status}`);
  }

  return response.json();
}
```

### PATCH 요청

리소스의 일부만 수정할 때 사용합니다.

```javascript
async function patchUser(userId, partialData) {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(partialData),
  });

  if (!response.ok) {
    throw new Error(`사용자 부분 수정 실패: ${response.status}`);
  }

  return response.json();
}

// 이메일만 수정
patchUser(123, { email: 'newemail@example.com' });
```

### DELETE 요청

리소스를 삭제할 때 사용합니다.

```javascript
async function deleteUser(userId) {
  const response = await fetch(`https://api.example.com/users/${userId}`, {
    method: 'DELETE',
  });

  if (!response.ok) {
    throw new Error(`사용자 삭제 실패: ${response.status}`);
  }

  // DELETE는 보통 204 No Content를 반환
  if (response.status === 204) {
    return { success: true };
  }

  return response.json();
}
```

---

## Response 객체와 메서드

`fetch()`는 **Response 객체**를 반환하는 Promise를 리턴합니다. Response 객체는 다양한 속성과 메서드를 제공합니다.

### Response 속성

```javascript
async function inspectResponse() {
  const response = await fetch('https://api.example.com/users');

  // 주요 속성들
  console.log('ok:', response.ok);           // 상태 코드가 200-299면 true
  console.log('status:', response.status);   // HTTP 상태 코드 (예: 200, 404)
  console.log('statusText:', response.statusText); // 상태 메시지 (예: 'OK', 'Not Found')
  console.log('url:', response.url);         // 요청 URL
  console.log('type:', response.type);       // 응답 타입 (basic, cors, opaque 등)
  console.log('redirected:', response.redirected); // 리다이렉트 여부

  // Headers 객체
  console.log('Content-Type:', response.headers.get('content-type'));
  console.log('모든 헤더:');
  response.headers.forEach((value, key) => {
    console.log(`  ${key}: ${value}`);
  });
}
```

### 응답 본문 읽기 메서드

Response 본문은 **한 번만 읽을 수 있습니다**. 여러 번 읽어야 한다면 `clone()`을 사용하세요.

```javascript
async function readResponseBody() {
  const response = await fetch('https://api.example.com/data');

  // 1. JSON으로 파싱 (가장 많이 사용)
  const jsonData = await response.json();

  // 2. 텍스트로 읽기
  const textData = await response.text();

  // 3. Blob으로 읽기 (이미지, 파일 등 바이너리 데이터)
  const blobData = await response.blob();

  // 4. ArrayBuffer로 읽기 (바이너리 데이터 직접 처리)
  const bufferData = await response.arrayBuffer();

  // 5. FormData로 읽기 (multipart/form-data 응답)
  const formData = await response.formData();
}

// 응답을 여러 번 읽어야 할 때
async function readMultipleTimes() {
  const response = await fetch('https://api.example.com/data');

  // 복제본 생성
  const clonedResponse = response.clone();

  // 원본은 JSON으로
  const jsonData = await response.json();

  // 복제본은 텍스트로
  const textData = await clonedResponse.text();

  console.log('JSON:', jsonData);
  console.log('Text:', textData);
}
```

### Content-Type별 응답 처리

```javascript
async function handleDifferentContentTypes(url) {
  const response = await fetch(url);
  const contentType = response.headers.get('content-type');

  if (contentType?.includes('application/json')) {
    return await response.json();
  } else if (contentType?.includes('text/html')) {
    return await response.text();
  } else if (contentType?.includes('image/')) {
    return await response.blob();
  } else if (contentType?.includes('application/octet-stream')) {
    return await response.arrayBuffer();
  } else {
    // 기본적으로 텍스트로 처리
    return await response.text();
  }
}
```

---

## 요청 옵션 상세

fetch의 두 번째 인자로 다양한 옵션을 설정할 수 있습니다.

### headers 설정

```javascript
// Headers 객체 사용
const headers = new Headers();
headers.append('Content-Type', 'application/json');
headers.append('Authorization', 'Bearer token123');
headers.append('X-Custom-Header', 'custom-value');

await fetch('https://api.example.com/data', {
  method: 'POST',
  headers: headers,
  body: JSON.stringify({ data: 'test' })
});

// 객체 리터럴로 직접 설정 (더 간단)
await fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123',
    'X-Custom-Header': 'custom-value'
  },
  body: JSON.stringify({ data: 'test' })
});
```

### body와 Content-Type

| Content-Type | body 형식 | 사용 사례 |
|--------------|-----------|-----------|
| application/json | JSON.stringify(obj) | REST API |
| application/x-www-form-urlencoded | URLSearchParams | HTML form |
| multipart/form-data | FormData | 파일 업로드 |
| text/plain | 문자열 | 단순 텍스트 |

```javascript
// 1. JSON 데이터
await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: '홍길동', age: 30 })
});

// 2. Form 데이터 (URL encoded)
const params = new URLSearchParams();
params.append('username', 'hong');
params.append('password', 'secret123');

await fetch('/api/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: params
});

// 3. FormData (파일 업로드 시 Content-Type 자동 설정)
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('description', '프로필 이미지');

await fetch('/api/upload', {
  method: 'POST',
  // Content-Type 헤더를 설정하지 않음 (자동으로 multipart/form-data 설정)
  body: formData
});
```

### credentials (쿠키 전송)

```javascript
// 'same-origin': 동일 출처 요청에만 쿠키 전송 (기본값)
// 'include': 크로스 오리진 요청에도 쿠키 전송
// 'omit': 쿠키 전송하지 않음

// 크로스 오리진 API에 인증 쿠키 전송
await fetch('https://api.other-domain.com/user', {
  credentials: 'include'
});

// 쿠키 없이 요청
await fetch('https://api.example.com/public', {
  credentials: 'omit'
});
```

> 크로스 오리진 요청에서 `credentials: 'include'`를 사용하려면 서버에서 `Access-Control-Allow-Credentials: true` 헤더를 설정해야 합니다.
{: .prompt-warning }

### mode (CORS 관련)

```javascript
// 'cors': CORS 요청 허용 (기본값)
// 'same-origin': 동일 출처만 허용
// 'no-cors': CORS 없이 요청 (응답 읽기 제한)

// 기본 CORS 요청
await fetch('https://api.other-domain.com/data', {
  mode: 'cors'
});

// 동일 출처만 허용
await fetch('/api/internal', {
  mode: 'same-origin'
});

// no-cors (응답 본문에 접근 불가, opaque 응답)
await fetch('https://external-api.com/data', {
  mode: 'no-cors'
});
```

### cache 정책

```javascript
// 'default': 브라우저 기본 캐시 동작
// 'no-store': 캐시 완전 무시
// 'reload': 캐시 무시하고 새로 요청, 응답으로 캐시 갱신
// 'no-cache': 서버에 검증 후 캐시 사용
// 'force-cache': 캐시 우선, 없으면 네트워크 요청
// 'only-if-cached': 캐시만 사용 (same-origin 모드에서만)

// 항상 최신 데이터 (캐시 무시)
await fetch('https://api.example.com/live-data', {
  cache: 'no-store'
});

// 캐시 우선
await fetch('https://api.example.com/static-data', {
  cache: 'force-cache'
});
```

### 전체 옵션 예시

```javascript
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ name: '홍길동' }),
  credentials: 'include',
  mode: 'cors',
  cache: 'no-cache',
  redirect: 'follow',      // 리다이렉트 자동 따라가기
  referrer: 'no-referrer', // Referrer 헤더 제어
  referrerPolicy: 'no-referrer-when-downgrade',
  integrity: 'sha256-abc123...',  // Subresource Integrity
  keepalive: false,        // 페이지 언로드 후에도 요청 유지
  signal: abortController.signal  // 요청 취소용
});
```

---

## 에러 처리

Fetch API의 에러 처리는 일반적인 예상과 다를 수 있어 주의가 필요합니다.

### 네트워크 에러 vs HTTP 에러

```javascript
async function fetchWithProperErrorHandling(url) {
  try {
    const response = await fetch(url);

    // HTTP 에러 상태 체크 (4xx, 5xx)
    if (!response.ok) {
      // HTTP 에러는 여기서 처리
      const errorBody = await response.text();
      throw new Error(`HTTP ${response.status}: ${response.statusText}. ${errorBody}`);
    }

    return await response.json();
  } catch (error) {
    // 네트워크 에러 또는 위에서 throw한 HTTP 에러
    if (error.name === 'TypeError') {
      // 네트워크 에러 (연결 실패, CORS 문제 등)
      console.error('네트워크 연결 실패:', error.message);
    } else {
      // HTTP 에러 또는 기타 에러
      console.error('요청 실패:', error.message);
    }
    throw error;
  }
}
```

### HTTP 상태 코드별 처리

```javascript
async function handleHttpStatus(url) {
  const response = await fetch(url);

  switch (response.status) {
    case 200:
    case 201:
      return await response.json();

    case 204:
      // No Content
      return null;

    case 400:
      const badRequestError = await response.json();
      throw new Error(`잘못된 요청: ${badRequestError.message}`);

    case 401:
      // 인증 필요 - 로그인 페이지로 리다이렉트
      window.location.href = '/login';
      throw new Error('인증이 필요합니다');

    case 403:
      throw new Error('접근 권한이 없습니다');

    case 404:
      throw new Error('리소스를 찾을 수 없습니다');

    case 409:
      throw new Error('리소스 충돌이 발생했습니다');

    case 422:
      const validationError = await response.json();
      throw new Error(`유효성 검사 실패: ${JSON.stringify(validationError.errors)}`);

    case 429:
      throw new Error('요청이 너무 많습니다. 잠시 후 다시 시도해주세요');

    case 500:
    case 502:
    case 503:
      throw new Error('서버 오류가 발생했습니다. 잠시 후 다시 시도해주세요');

    default:
      throw new Error(`예상치 못한 응답: ${response.status}`);
  }
}
```

### 커스텀 에러 클래스

```javascript
// 커스텀 HTTP 에러 클래스
class HttpError extends Error {
  constructor(response, message) {
    super(message || `HTTP Error: ${response.status}`);
    this.name = 'HttpError';
    this.response = response;
    this.status = response.status;
    this.statusText = response.statusText;
  }
}

class NetworkError extends Error {
  constructor(message) {
    super(message || '네트워크 연결에 실패했습니다');
    this.name = 'NetworkError';
  }
}

// 사용
async function safeFetch(url, options = {}) {
  let response;

  try {
    response = await fetch(url, options);
  } catch (error) {
    throw new NetworkError(error.message);
  }

  if (!response.ok) {
    const errorMessage = await response.text().catch(() => '');
    throw new HttpError(response, errorMessage);
  }

  return response;
}

// 에러 타입별 처리
async function apiCall() {
  try {
    const response = await safeFetch('https://api.example.com/data');
    return await response.json();
  } catch (error) {
    if (error instanceof NetworkError) {
      console.error('네트워크 문제:', error.message);
      // 오프라인 UI 표시
    } else if (error instanceof HttpError) {
      console.error(`HTTP ${error.status}:`, error.message);
      if (error.status === 401) {
        // 로그인 페이지로 이동
      }
    }
    throw error;
  }
}
```

---

## AbortController로 요청 취소

**AbortController**는 하나 이상의 웹 요청을 취소할 수 있는 컨트롤러 객체입니다.

### 기본 사용법

```javascript
// AbortController 생성
const controller = new AbortController();
const signal = controller.signal;

// fetch에 signal 전달
fetch('https://api.example.com/large-data', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('요청이 취소되었습니다');
    } else {
      console.error('요청 실패:', error);
    }
  });

// 5초 후 요청 취소
setTimeout(() => {
  controller.abort();
}, 5000);
```

### 타임아웃 구현

```javascript
async function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal
    });
    clearTimeout(timeoutId);
    return response;
  } catch (error) {
    clearTimeout(timeoutId);
    if (error.name === 'AbortError') {
      throw new Error(`요청 시간 초과 (${timeout}ms)`);
    }
    throw error;
  }
}

// 사용
try {
  const response = await fetchWithTimeout('https://api.example.com/slow-api', {}, 3000);
  const data = await response.json();
  console.log(data);
} catch (error) {
  console.error(error.message); // '요청 시간 초과 (3000ms)'
}
```

### AbortSignal.timeout() (최신 API)

최신 브라우저에서는 `AbortSignal.timeout()`을 사용할 수 있습니다.

```javascript
// AbortSignal.timeout() 사용 (Chrome 103+, Firefox 100+)
async function fetchWithModernTimeout(url) {
  try {
    const response = await fetch(url, {
      signal: AbortSignal.timeout(5000)
    });
    return await response.json();
  } catch (error) {
    if (error.name === 'TimeoutError') {
      console.error('요청 시간 초과');
    } else if (error.name === 'AbortError') {
      console.error('요청이 취소됨');
    } else {
      console.error('기타 에러:', error);
    }
    throw error;
  }
}
```

### 여러 요청 동시 취소

```javascript
// 하나의 controller로 여러 요청 관리
const controller = new AbortController();

Promise.all([
  fetch('/api/users', { signal: controller.signal }),
  fetch('/api/posts', { signal: controller.signal }),
  fetch('/api/comments', { signal: controller.signal })
])
  .then(responses => Promise.all(responses.map(r => r.json())))
  .then(([users, posts, comments]) => {
    console.log({ users, posts, comments });
  })
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('모든 요청이 취소되었습니다');
    }
  });

// 필요시 모든 요청 취소
document.getElementById('cancelBtn').addEventListener('click', () => {
  controller.abort();
});
```

### React에서 cleanup 활용

React 컴포넌트에서 fetch를 사용할 때 cleanup 함수에서 요청을 취소하는 것은 메모리 누수 방지에 필수적입니다. 더 다양한 커스텀 훅 패턴은 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)에서 확인할 수 있습니다.

```tsx
import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

function useUser(userId: number) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(`/api/users/${userId}`, {
          signal: controller.signal
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        setUser(data);
      } catch (err) {
        // AbortError는 무시 (컴포넌트 언마운트로 인한 취소)
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        // AbortError가 아닐 때만 loading 상태 변경
        if (!controller.signal.aborted) {
          setLoading(false);
        }
      }
    }

    fetchUser();

    // cleanup: 컴포넌트 언마운트 또는 userId 변경 시 요청 취소
    return () => {
      controller.abort();
    };
  }, [userId]);

  return { user, loading, error };
}

// 사용
function UserProfile({ userId }: { userId: number }) {
  const { user, loading, error } = useUser(userId);

  if (loading) return <div>로딩 중...</div>;
  if (error) return <div>에러: {error.message}</div>;
  if (!user) return <div>사용자를 찾을 수 없습니다</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

> React 컴포넌트에서 fetch를 사용할 때는 반드시 cleanup 함수에서 요청을 취소해야 합니다. 그렇지 않으면 언마운트된 컴포넌트에서 상태를 업데이트하려고 시도하여 메모리 누수가 발생할 수 있습니다. 메모리 관리에 대한 자세한 내용은 [JavaScript 메모리 관리와 가비지 컬렉션 가이드](/posts/javascript-memory-management-garbage-collection-guide/)를 참고하세요.
{: .prompt-tip }

---

## 고급 패턴

### 재시도(Retry) 로직 구현

네트워크 불안정이나 일시적인 서버 오류에 대응하기 위한 재시도 로직입니다.

```javascript
async function fetchWithRetry(url, options = {}, retries = 3, delay = 1000) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url, options);

      // 5xx 서버 에러는 재시도
      if (response.status >= 500 && attempt < retries) {
        console.log(`서버 에러 (${response.status}), ${attempt}번째 시도...`);
        await new Promise(resolve => setTimeout(resolve, delay * attempt));
        continue;
      }

      return response;
    } catch (error) {
      // 네트워크 에러는 재시도
      if (attempt === retries) {
        throw error;
      }
      console.log(`네트워크 에러, ${attempt}번째 재시도...`);
      await new Promise(resolve => setTimeout(resolve, delay * attempt));
    }
  }
}

// 지수 백오프(Exponential Backoff) 적용
async function fetchWithExponentialBackoff(url, options = {}, maxRetries = 3) {
  const baseDelay = 1000;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.status >= 500 && attempt < maxRetries - 1) {
        const delay = baseDelay * Math.pow(2, attempt); // 1s, 2s, 4s...
        console.log(`재시도 ${attempt + 1}/${maxRetries}, ${delay}ms 대기...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      return response;
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;

      const delay = baseDelay * Math.pow(2, attempt);
      console.log(`네트워크 에러, ${delay}ms 후 재시도...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### 인터셉터 패턴

요청/응답을 가로채서 공통 로직을 처리하는 패턴입니다.

```javascript
// 인터셉터 기반 Fetch 래퍼
class FetchClient {
  constructor(baseURL = '') {
    this.baseURL = baseURL;
    this.requestInterceptors = [];
    this.responseInterceptors = [];
  }

  // 요청 인터셉터 추가
  addRequestInterceptor(interceptor) {
    this.requestInterceptors.push(interceptor);
    return this;
  }

  // 응답 인터셉터 추가
  addResponseInterceptor(interceptor) {
    this.responseInterceptors.push(interceptor);
    return this;
  }

  async request(url, options = {}) {
    let config = {
      url: this.baseURL + url,
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    };

    // 요청 인터셉터 실행
    for (const interceptor of this.requestInterceptors) {
      config = await interceptor(config);
    }

    let response = await fetch(config.url, config);

    // 응답 인터셉터 실행
    for (const interceptor of this.responseInterceptors) {
      response = await interceptor(response, config);
    }

    return response;
  }

  get(url, options = {}) {
    return this.request(url, { ...options, method: 'GET' });
  }

  post(url, data, options = {}) {
    return this.request(url, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  put(url, data, options = {}) {
    return this.request(url, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  delete(url, options = {}) {
    return this.request(url, { ...options, method: 'DELETE' });
  }
}

// 사용 예시
const api = new FetchClient('https://api.example.com');

// 인증 토큰 추가 인터셉터
api.addRequestInterceptor(async (config) => {
  const token = localStorage.getItem('authToken');
  if (token) {
    config.headers = {
      ...config.headers,
      'Authorization': `Bearer ${token}`
    };
  }
  return config;
});

// 요청 로깅 인터셉터
api.addRequestInterceptor(async (config) => {
  console.log(`[Request] ${config.method || 'GET'} ${config.url}`);
  return config;
});

// 401 에러 처리 인터셉터
api.addResponseInterceptor(async (response, config) => {
  if (response.status === 401) {
    // 토큰 갱신 시도
    const refreshed = await refreshToken();
    if (refreshed) {
      // 원래 요청 재시도
      const token = localStorage.getItem('authToken');
      config.headers['Authorization'] = `Bearer ${token}`;
      return fetch(config.url, config);
    } else {
      // 로그인 페이지로 이동
      window.location.href = '/login';
    }
  }
  return response;
});

// 응답 로깅 인터셉터
api.addResponseInterceptor(async (response) => {
  console.log(`[Response] ${response.status} ${response.url}`);
  return response;
});

// API 호출
const users = await api.get('/users');
const newUser = await api.post('/users', { name: '홍길동' });
```

### Wrapper 함수 만들기

실무에서 자주 사용하는 형태의 fetch 래퍼입니다.

```javascript
const API_BASE_URL = 'https://api.example.com';

async function apiRequest(endpoint, options = {}) {
  const url = `${API_BASE_URL}${endpoint}`;

  const config = {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
  };

  // 인증 토큰 자동 추가
  const token = localStorage.getItem('token');
  if (token) {
    config.headers['Authorization'] = `Bearer ${token}`;
  }

  // body가 객체면 JSON 문자열로 변환
  if (config.body && typeof config.body === 'object') {
    config.body = JSON.stringify(config.body);
  }

  try {
    const response = await fetch(url, config);

    // 응답이 JSON인지 확인
    const contentType = response.headers.get('content-type');
    const isJson = contentType?.includes('application/json');

    // 응답 본문 파싱
    const data = isJson ? await response.json() : await response.text();

    if (!response.ok) {
      const error = new Error(data.message || `HTTP Error ${response.status}`);
      error.status = response.status;
      error.data = data;
      throw error;
    }

    return data;
  } catch (error) {
    // 에러 로깅
    console.error(`API Error [${endpoint}]:`, error);
    throw error;
  }
}

// 편의 메서드
const api = {
  get: (endpoint, options = {}) =>
    apiRequest(endpoint, { ...options, method: 'GET' }),

  post: (endpoint, body, options = {}) =>
    apiRequest(endpoint, { ...options, method: 'POST', body }),

  put: (endpoint, body, options = {}) =>
    apiRequest(endpoint, { ...options, method: 'PUT', body }),

  patch: (endpoint, body, options = {}) =>
    apiRequest(endpoint, { ...options, method: 'PATCH', body }),

  delete: (endpoint, options = {}) =>
    apiRequest(endpoint, { ...options, method: 'DELETE' }),
};

// 사용
const users = await api.get('/users');
const newUser = await api.post('/users', { name: '홍길동', email: 'hong@example.com' });
await api.patch('/users/1', { name: '홍길순' });
await api.delete('/users/1');
```

---

## 실전 예제

### 파일 업로드

```javascript
async function uploadFile(file, onProgress) {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('type', file.type);
  formData.append('name', file.name);

  // XMLHttpRequest를 사용한 진행률 표시 (fetch는 직접 지원하지 않음)
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (event) => {
      if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        onProgress?.(percentComplete);
      }
    });

    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.response));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });

    xhr.addEventListener('error', () => {
      reject(new Error('Network error during upload'));
    });

    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  });
}

// Fetch API만 사용 (진행률 없음)
async function uploadFileSimple(file) {
  const formData = new FormData();
  formData.append('file', file);

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
    // Content-Type 헤더를 설정하지 않음!
    // 브라우저가 boundary를 포함한 올바른 헤더를 자동 설정
  });

  if (!response.ok) {
    throw new Error('업로드 실패');
  }

  return response.json();
}

// 여러 파일 동시 업로드
async function uploadMultipleFiles(files) {
  const formData = new FormData();

  Array.from(files).forEach((file, index) => {
    formData.append(`files`, file);
  });

  const response = await fetch('/api/upload-multiple', {
    method: 'POST',
    body: formData
  });

  return response.json();
}
```

### 스트리밍 응답 처리

대용량 데이터나 실시간 데이터를 스트리밍으로 처리합니다.

```javascript
async function streamResponse(url) {
  const response = await fetch(url);

  if (!response.body) {
    throw new Error('ReadableStream not supported');
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  let result = '';

  while (true) {
    const { done, value } = await reader.read();

    if (done) break;

    const chunk = decoder.decode(value, { stream: true });
    result += chunk;
    console.log('Received chunk:', chunk);
  }

  return result;
}

// Server-Sent Events (SSE) 스타일 스트리밍
async function streamSSE(url, onMessage) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });

    // 줄 단위로 파싱
    const lines = buffer.split('\n');
    buffer = lines.pop(); // 마지막 불완전한 줄은 버퍼에 유지

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = JSON.parse(line.slice(6));
        onMessage(data);
      }
    }
  }
}

// 사용
streamSSE('/api/events', (data) => {
  console.log('New event:', data);
});
```

### 이미지 다운로드 및 표시

```javascript
async function downloadAndDisplayImage(imageUrl, imgElement) {
  const response = await fetch(imageUrl);

  if (!response.ok) {
    throw new Error('이미지 다운로드 실패');
  }

  const blob = await response.blob();
  const objectUrl = URL.createObjectURL(blob);

  imgElement.src = objectUrl;

  // 더 이상 필요 없을 때 메모리 해제
  imgElement.onload = () => {
    // 필요하다면 나중에 해제: URL.revokeObjectURL(objectUrl);
  };
}

// 진행률과 함께 다운로드
async function downloadWithProgress(url, onProgress) {
  const response = await fetch(url);
  const contentLength = response.headers.get('Content-Length');
  const total = parseInt(contentLength, 10);

  const reader = response.body.getReader();
  const chunks = [];
  let loaded = 0;

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    chunks.push(value);
    loaded += value.length;

    if (total) {
      const progress = (loaded / total) * 100;
      onProgress?.(progress);
    }
  }

  // 청크들을 하나의 Blob으로 합치기
  const blob = new Blob(chunks);
  return blob;
}

// 사용
const imageBlob = await downloadWithProgress(
  'https://example.com/large-image.jpg',
  (progress) => console.log(`${progress.toFixed(1)}% 완료`)
);
```

---

## TypeScript와 함께 사용하기

TypeScript를 활용하면 API 응답에 대한 타입 안전성을 확보할 수 있습니다. 더 고급 타입 패턴은 [TypeScript 고급 타입 패턴 가이드](/posts/typescript-advanced-type-patterns/)를 참고하세요.

### 응답 타입 정의

```typescript
// API 응답 타입 정의
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: string;
}

interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
}

interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

// 에러 응답 타입
interface ApiError {
  message: string;
  code: string;
  details?: Record<string, string[]>;
}
```

### 제네릭 Fetch 함수

```typescript
class HttpError extends Error {
  constructor(
    public status: number,
    public statusText: string,
    public data: unknown
  ) {
    super(`HTTP Error ${status}: ${statusText}`);
    this.name = 'HttpError';
  }
}

async function fetchJson<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  const response = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
    ...options,
  });

  const data = await response.json();

  if (!response.ok) {
    throw new HttpError(response.status, response.statusText, data);
  }

  return data as T;
}

// 사용
const user = await fetchJson<User>('/api/users/1');
console.log(user.name); // 타입 안전

const users = await fetchJson<PaginatedResponse<User>>('/api/users?page=1');
console.log(users.data[0].email); // 타입 안전
```

### 완전한 타입 안전 API 클라이언트

```typescript
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';

interface RequestConfig<TBody = unknown> {
  method?: HttpMethod;
  headers?: HeadersInit;
  body?: TBody;
  signal?: AbortSignal;
  credentials?: RequestCredentials;
}

class ApiClient {
  constructor(private baseUrl: string) {}

  private async request<TResponse, TBody = unknown>(
    endpoint: string,
    config: RequestConfig<TBody> = {}
  ): Promise<TResponse> {
    const { method = 'GET', headers, body, signal, credentials } = config;

    const url = `${this.baseUrl}${endpoint}`;

    const fetchConfig: RequestInit = {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...headers,
      },
      signal,
      credentials,
    };

    if (body !== undefined) {
      fetchConfig.body = JSON.stringify(body);
    }

    const token = localStorage.getItem('token');
    if (token) {
      (fetchConfig.headers as Record<string, string>)['Authorization'] =
        `Bearer ${token}`;
    }

    const response = await fetch(url, fetchConfig);

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new HttpError(response.status, response.statusText, errorData);
    }

    // 204 No Content
    if (response.status === 204) {
      return undefined as TResponse;
    }

    return response.json();
  }

  get<TResponse>(
    endpoint: string,
    config?: Omit<RequestConfig, 'method' | 'body'>
  ): Promise<TResponse> {
    return this.request<TResponse>(endpoint, { ...config, method: 'GET' });
  }

  post<TResponse, TBody>(
    endpoint: string,
    body: TBody,
    config?: Omit<RequestConfig, 'method' | 'body'>
  ): Promise<TResponse> {
    return this.request<TResponse, TBody>(endpoint, {
      ...config,
      method: 'POST',
      body,
    });
  }

  put<TResponse, TBody>(
    endpoint: string,
    body: TBody,
    config?: Omit<RequestConfig, 'method' | 'body'>
  ): Promise<TResponse> {
    return this.request<TResponse, TBody>(endpoint, {
      ...config,
      method: 'PUT',
      body,
    });
  }

  patch<TResponse, TBody>(
    endpoint: string,
    body: TBody,
    config?: Omit<RequestConfig, 'method' | 'body'>
  ): Promise<TResponse> {
    return this.request<TResponse, TBody>(endpoint, {
      ...config,
      method: 'PATCH',
      body,
    });
  }

  delete<TResponse>(
    endpoint: string,
    config?: Omit<RequestConfig, 'method' | 'body'>
  ): Promise<TResponse> {
    return this.request<TResponse>(endpoint, { ...config, method: 'DELETE' });
  }
}

// 사용
const api = new ApiClient('https://api.example.com');

// 완전한 타입 추론
interface CreateUserRequest {
  name: string;
  email: string;
}

const newUser = await api.post<User, CreateUserRequest>('/users', {
  name: '홍길동',
  email: 'hong@example.com',
});

console.log(newUser.id); // number 타입으로 추론

// 사용자 목록 조회
const users = await api.get<PaginatedResponse<User>>('/users');
users.data.forEach(user => {
  console.log(user.name); // 타입 안전
});
```

---

## Fetch vs Axios 비교

### 기능 비교

| 기능 | Fetch API | Axios |
|------|-----------|-------|
| 브라우저 내장 | O | X (설치 필요) |
| 번들 크기 | 0 KB | ~13 KB (gzip) |
| Promise 기반 | O | O |
| HTTP 에러 자동 reject | X | O |
| 요청 취소 | AbortController | CancelToken / AbortController |
| 요청/응답 인터셉터 | 직접 구현 | 내장 지원 |
| 자동 JSON 변환 | X (response.json() 필요) | O |
| 타임아웃 설정 | 직접 구현 | timeout 옵션 |
| 진행률 추적 | ReadableStream 활용 | onUploadProgress / onDownloadProgress |
| XSRF 보호 | X | O |
| Node.js 지원 | 18+ / polyfill 필요 | O |
| 요청 데이터 변환 | 직접 구현 | transformRequest |
| 응답 데이터 변환 | 직접 구현 | transformResponse |

### 코드 비교

```javascript
// Fetch API
async function fetchExample() {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: '홍길동' })
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error('에러:', error);
    throw error;
  }
}

// Axios
async function axiosExample() {
  try {
    const { data } = await axios.post('https://api.example.com/users', {
      name: '홍길동'
    });
    return data;
  } catch (error) {
    // 4xx, 5xx 에러도 여기서 처리
    console.error('에러:', error.response?.data || error.message);
    throw error;
  }
}
```

### 언제 무엇을 사용할까?

**Fetch API를 선택하는 경우:**
- 번들 크기를 최소화하고 싶을 때
- 의존성을 줄이고 싶을 때
- 간단한 요청만 필요할 때
- 브라우저 표준 API를 사용하고 싶을 때
- 이미 충분한 추상화 레이어가 있을 때

**Axios를 선택하는 경우:**
- 인터셉터가 복잡하게 필요할 때
- 구버전 브라우저 지원이 필요할 때
- Node.js와 브라우저에서 동일한 코드 사용 시
- 자동 JSON 변환, 에러 처리가 편리하길 원할 때
- 요청/응답 변환이 빈번할 때

> 최근에는 Fetch API의 기능이 많이 개선되어, 간단한 프로젝트에서는 Fetch만으로도 충분합니다. 복잡한 요구사항이 있다면 Axios나 ky, ofetch 같은 래퍼 라이브러리를 고려해보세요.
{: .prompt-tip }

---

## 마무리

### 핵심 정리

1. **Fetch API 기본**: `fetch()`는 Promise를 반환하며, Response 객체로 결과를 처리합니다.

2. **HTTP 에러 주의**: Fetch는 HTTP 에러(4xx, 5xx)에서 reject되지 않습니다. `response.ok`를 확인해야 합니다.

3. **요청 옵션**: method, headers, body, credentials, mode, cache 등 다양한 옵션으로 요청을 제어합니다.

4. **AbortController**: 요청 취소와 타임아웃 구현에 필수적입니다. React cleanup에서 활용하세요.

5. **고급 패턴**: 재시도 로직, 인터셉터 패턴, Wrapper 함수로 실무에서 효율적으로 사용할 수 있습니다.

### Best Practices

```javascript
// 1. 항상 에러 처리하기
if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}

// 2. AbortController로 요청 관리
const controller = new AbortController();
fetch(url, { signal: controller.signal });

// 3. 타임아웃 설정
const response = await fetch(url, {
  signal: AbortSignal.timeout(5000)
});

// 4. 타입 안전한 API 클라이언트 사용 (TypeScript)
const data = await api.get<User>('/users/1');

// 5. 인터셉터로 공통 로직 분리
api.addRequestInterceptor(addAuthToken);
api.addResponseInterceptor(handleUnauthorized);
```

### 면접 포인트

| 질문 | 핵심 답변 |
|------|-----------|
| Fetch와 XHR의 차이는? | Promise 기반, 스트리밍 지원, 깔끔한 API |
| Fetch가 reject되는 경우는? | 네트워크 장애만. HTTP 에러는 reject 안 됨 |
| 요청 취소 방법은? | AbortController.abort() |
| 타임아웃 구현 방법은? | AbortSignal.timeout() 또는 setTimeout + abort |
| Axios와의 차이점은? | 내장/외부, 자동 JSON 변환, 인터셉터 지원 |

Fetch API는 모던 웹 개발의 기본입니다. 이 가이드에서 다룬 내용을 바탕으로 실무에서 안정적이고 효율적인 네트워크 통신을 구현하시기 바랍니다.

---

## 관련 포스트

Fetch API와 함께 알면 좋은 주제들입니다:

- [React Query 완벽 가이드](/posts/react-query-guide/) - 서버 상태 관리 라이브러리로 Fetch를 더 효율적으로 사용하기
- [MSW API 모킹 가이드](/posts/msw-api-mocking-guide/) - 개발 및 테스트 환경에서 API 모킹하기
- [웹 성능 최적화 가이드](/posts/web-performance-optimization-guide/) - 네트워크 요청 최적화 전략

---

## 참고 자료

- [MDN - Fetch API](https://developer.mozilla.org/ko/docs/Web/API/Fetch_API)
- [MDN - Using the Fetch API](https://developer.mozilla.org/ko/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN - AbortController](https://developer.mozilla.org/ko/docs/Web/API/AbortController)
- [WHATWG Fetch Standard](https://fetch.spec.whatwg.org/)
- [Can I use - Fetch](https://caniuse.com/fetch)
