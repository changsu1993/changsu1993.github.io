---
title: "JavaScript URL과 URLSearchParams 완벽 가이드 - URL 파싱부터 쿼리 파라미터 관리까지"
description: "JavaScript URL API와 URLSearchParams 완벽 가이드. URL 파싱, 쿼리 스트링 조작, 딥링크 생성부터 React/Next.js 연동, History API를 활용한 SPA 라우팅까지 실무 코드 예제로 상세히 설명합니다."
date: 2025-12-29 15:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, URL, URLSearchParams, Query-String, Browser-API, TypeScript, React, Next.js, History-API, SPA, 딥링크, 라우팅, 웹개발, URL-파싱, 상태관리, REST-API, 프론트엔드, URL-인코딩]
---

## 개요

웹 개발에서 URL을 다루는 것은 필수적인 기술입니다. 사용자를 올바른 페이지로 안내하고, 검색 필터를 적용하고, API 요청을 구성하는 모든 작업에서 URL을 조작해야 합니다. **URL API**와 **URLSearchParams API**는 이러한 작업을 안전하고 편리하게 수행할 수 있게 해주는 브라우저 내장 기능입니다.

이 글에서는 URL 객체의 구조부터 URLSearchParams를 활용한 쿼리 파라미터 관리, React/Next.js와의 연동, History API를 활용한 SPA URL 관리까지 실무에서 필요한 모든 내용을 다룹니다.

### 학습 목표

- URL 객체의 구조와 각 구성 요소 이해
- URLSearchParams를 활용한 쿼리 파라미터 조작 방법 습득
- URL 유효성 검증과 안전한 URL 처리 방법 학습
- React/Next.js에서 URL 기반 상태 관리 패턴 적용
- History API와 URL을 연동한 SPA 라우팅 구현

### 사전 지식

- JavaScript 기초 문법
- [JavaScript ES6 Modules](/posts/javascript-modules-and-namespaces/) 이해
- TypeScript 기초 (고급 패턴 섹션)
- React 기초 (React 연동 섹션)

---

## URL API 소개

### URL 객체 생성

URL 생성자는 URL 문자열을 파싱하여 URL 객체를 생성합니다.

```javascript
// 절대 URL로 생성
const url = new URL('https://example.com:8080/path/page?name=value#section');

// 상대 URL + 기준 URL로 생성
const relativeUrl = new URL('/api/users', 'https://example.com');
console.log(relativeUrl.href); // 'https://example.com/api/users'

// 현재 페이지 기준 상대 URL
const currentUrl = new URL('/about', window.location.origin);
```

### URL의 구성 요소

URL 객체는 URL의 각 부분에 쉽게 접근할 수 있는 속성들을 제공합니다.

```javascript
const url = new URL('https://user:pass@api.example.com:8080/v1/users?page=1&limit=10#results');

console.log(url.href);       // 'https://user:pass@api.example.com:8080/v1/users?page=1&limit=10#results'
console.log(url.protocol);   // 'https:'
console.log(url.username);   // 'user'
console.log(url.password);   // 'pass'
console.log(url.host);       // 'api.example.com:8080'
console.log(url.hostname);   // 'api.example.com'
console.log(url.port);       // '8080'
console.log(url.pathname);   // '/v1/users'
console.log(url.search);     // '?page=1&limit=10'
console.log(url.hash);       // '#results'
console.log(url.origin);     // 'https://api.example.com:8080'
```

다음 표는 URL의 각 구성 요소를 정리한 것입니다.

| 속성 | 설명 | 예시 |
|------|------|------|
| `href` | 전체 URL 문자열 | `https://example.com/path?q=1` |
| `protocol` | 프로토콜 (콜론 포함) | `https:` |
| `host` | 호스트명 + 포트 | `example.com:8080` |
| `hostname` | 호스트명만 | `example.com` |
| `port` | 포트 번호 | `8080` |
| `pathname` | 경로 | `/path/to/page` |
| `search` | 쿼리 스트링 (? 포함) | `?page=1&limit=10` |
| `searchParams` | URLSearchParams 객체 | `URLSearchParams` |
| `hash` | 해시 (# 포함) | `#section` |
| `origin` | 출처 (protocol + host) | `https://example.com` |
| `username` | 인증 사용자명 | `user` |
| `password` | 인증 비밀번호 | `pass` |

### URL 속성 수정

URL 객체의 속성은 읽기 전용이 아니라 수정 가능합니다.

```javascript
const url = new URL('https://example.com/old-path');

// 경로 변경
url.pathname = '/new-path';
console.log(url.href); // 'https://example.com/new-path'

// 프로토콜 변경
url.protocol = 'http:';
console.log(url.href); // 'http://example.com/new-path'

// 호스트 변경
url.hostname = 'api.example.com';
url.port = '3000';
console.log(url.href); // 'http://api.example.com:3000/new-path'

// 해시 추가
url.hash = 'section1';
console.log(url.href); // 'http://api.example.com:3000/new-path#section1'
```

### 상대 URL과 절대 URL 처리

URL 생성자의 두 번째 인자를 활용하면 상대 경로를 절대 URL로 변환할 수 있습니다.

```javascript
const baseUrl = 'https://api.example.com/v1';

// 다양한 상대 경로 처리
console.log(new URL('/users', baseUrl).href);
// 'https://api.example.com/users' (루트 기준)

console.log(new URL('users', baseUrl).href);
// 'https://api.example.com/users' (현재 경로의 마지막 세그먼트 교체)

console.log(new URL('./users', baseUrl).href);
// 'https://api.example.com/users'

console.log(new URL('../users', baseUrl).href);
// 'https://api.example.com/users' (상위 디렉토리)

// 기준 URL의 경로가 /로 끝나는 경우
const baseWithSlash = 'https://api.example.com/v1/';
console.log(new URL('users', baseWithSlash).href);
// 'https://api.example.com/v1/users' (현재 디렉토리 기준)
```

> 기준 URL의 경로가 `/`로 끝나는지 여부에 따라 상대 경로 해석이 달라집니다. API 엔드포인트를 구성할 때 이 점을 주의해야 합니다.
{: .prompt-warning }

### URL 유효성 검증

잘못된 URL 문자열을 전달하면 `TypeError`가 발생합니다. 이를 활용하여 URL 유효성을 검증할 수 있습니다.

```javascript
// 유효성 검증 함수
function isValidUrl(urlString) {
  try {
    new URL(urlString);
    return true;
  } catch (error) {
    return false;
  }
}

console.log(isValidUrl('https://example.com')); // true
console.log(isValidUrl('not a url'));            // false
console.log(isValidUrl('example.com'));          // false (프로토콜 필수)
console.log(isValidUrl('//example.com'));        // false

// URL.canParse() - 최신 브라우저 지원 (Chrome 120+, Firefox 115+)
if (typeof URL.canParse === 'function') {
  console.log(URL.canParse('https://example.com')); // true
  console.log(URL.canParse('not a url'));           // false
}
```

```typescript
// TypeScript 타입 가드
function parseUrl(urlString: string): URL | null {
  try {
    return new URL(urlString);
  } catch {
    return null;
  }
}

const url = parseUrl('https://example.com');
if (url) {
  // url은 URL 타입으로 추론됨
  console.log(url.hostname);
}
```

---

## URLSearchParams API

### URLSearchParams 객체 생성

URLSearchParams는 URL의 쿼리 문자열을 다루기 위한 전용 객체입니다.

```javascript
// 1. 문자열로 생성
const params1 = new URLSearchParams('page=1&limit=10');

// 2. 객체로 생성
const params2 = new URLSearchParams({
  page: '1',
  limit: '10',
  sort: 'name'
});

// 3. 배열로 생성 (동일 키에 여러 값 가능)
const params3 = new URLSearchParams([
  ['color', 'red'],
  ['color', 'blue'],
  ['size', 'large']
]);

// 4. URL 객체에서 추출
const url = new URL('https://example.com?page=1&limit=10');
const params4 = url.searchParams;

// 5. 현재 페이지 URL에서 추출
const currentParams = new URLSearchParams(window.location.search);
```

### 파라미터 조작 메서드

URLSearchParams는 쿼리 파라미터를 조작하기 위한 다양한 메서드를 제공합니다.

```javascript
const params = new URLSearchParams('page=1');

// get() - 첫 번째 값 반환
console.log(params.get('page'));   // '1'
console.log(params.get('limit'));  // null (존재하지 않음)

// getAll() - 모든 값을 배열로 반환
const multiParams = new URLSearchParams('color=red&color=blue');
console.log(multiParams.getAll('color')); // ['red', 'blue']

// has() - 존재 여부 확인
console.log(params.has('page'));  // true
console.log(params.has('limit')); // false

// set() - 값 설정 (기존 값 덮어쓰기)
params.set('page', '2');
console.log(params.get('page')); // '2'

// append() - 값 추가 (기존 값 유지)
params.append('filter', 'active');
params.append('filter', 'verified');
console.log(params.getAll('filter')); // ['active', 'verified']

// delete() - 파라미터 삭제
params.delete('filter');
console.log(params.has('filter')); // false

// 특정 값만 삭제 (값 지정)
const colors = new URLSearchParams('color=red&color=blue&color=green');
colors.delete('color', 'blue');
console.log(colors.getAll('color')); // ['red', 'green']
```

### 순회 메서드

URLSearchParams는 이터러블이므로 다양한 방법으로 순회할 수 있습니다.

```javascript
const params = new URLSearchParams('page=1&limit=10&sort=name');

// entries() - [key, value] 쌍 순회
for (const [key, value] of params.entries()) {
  console.log(`${key}: ${value}`);
}
// page: 1
// limit: 10
// sort: name

// keys() - 키만 순회
for (const key of params.keys()) {
  console.log(key);
}
// page, limit, sort

// values() - 값만 순회
for (const value of params.values()) {
  console.log(value);
}
// 1, 10, name

// forEach() - 콜백으로 순회
params.forEach((value, key) => {
  console.log(`${key}=${value}`);
});

// 기본 이터레이터 (entries()와 동일)
for (const [key, value] of params) {
  console.log(`${key}: ${value}`);
}

// 객체로 변환
const paramsObject = Object.fromEntries(params);
console.log(paramsObject);
// { page: '1', limit: '10', sort: 'name' }

// 배열로 변환
const paramsArray = [...params];
console.log(paramsArray);
// [['page', '1'], ['limit', '10'], ['sort', 'name']]
```

### toString() 활용

`toString()` 메서드는 쿼리 문자열을 생성합니다.

```javascript
const params = new URLSearchParams();
params.set('search', '검색어');
params.set('category', 'books');
params.set('page', '1');

console.log(params.toString());
// 'search=%EA%B2%80%EC%83%89%EC%96%B4&category=books&page=1'

// URL과 조합
const baseUrl = 'https://api.example.com/search';
const fullUrl = `${baseUrl}?${params.toString()}`;
console.log(fullUrl);
// 'https://api.example.com/search?search=%EA%B2%80%EC%83%89%EC%96%B4&category=books&page=1'

// URL 객체와 조합
const url = new URL('https://api.example.com/search');
url.search = params.toString();
console.log(url.href);
```

> URLSearchParams는 자동으로 URL 인코딩을 처리합니다. 한글이나 특수문자도 안전하게 인코딩됩니다.
{: .prompt-info }

### sort() 메서드

`sort()` 메서드는 파라미터를 키 이름 기준으로 알파벳순 정렬합니다.

```javascript
const params = new URLSearchParams('z=last&a=first&m=middle');

params.sort();
console.log(params.toString());
// 'a=first&m=middle&z=last'
```

---

## 실전 활용 패턴

### 쿼리 스트링 파싱 및 생성

```javascript
// 현재 URL에서 쿼리 파라미터 읽기
function getQueryParams() {
  return Object.fromEntries(
    new URLSearchParams(window.location.search)
  );
}

// 객체를 쿼리 스트링으로 변환
function buildQueryString(params) {
  const searchParams = new URLSearchParams();

  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null && value !== '') {
      searchParams.set(key, String(value));
    }
  });

  return searchParams.toString();
}

// 사용 예시
const filters = {
  category: 'electronics',
  minPrice: 100,
  maxPrice: 500,
  inStock: true,
  brand: undefined // undefined는 제외됨
};

console.log(buildQueryString(filters));
// 'category=electronics&minPrice=100&maxPrice=500&inStock=true'
```

### API 엔드포인트 동적 생성

URL과 URLSearchParams를 활용하면 [Fetch API](/posts/javascript-fetch-api-network-guide/)와 함께 안전하게 API 요청 URL을 구성할 수 있습니다.

```typescript
interface ApiOptions {
  baseUrl: string;
  endpoint: string;
  params?: Record<string, string | number | boolean | undefined>;
}

function buildApiUrl(options: ApiOptions): string {
  const { baseUrl, endpoint, params } = options;

  // URL 객체로 안전하게 경로 결합
  const url = new URL(endpoint, baseUrl);

  // 파라미터 추가
  if (params) {
    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        url.searchParams.set(key, String(value));
      }
    });
  }

  return url.href;
}

// 사용 예시
const usersUrl = buildApiUrl({
  baseUrl: 'https://api.example.com',
  endpoint: '/v1/users',
  params: {
    page: 1,
    limit: 20,
    active: true
  }
});

console.log(usersUrl);
// 'https://api.example.com/v1/users?page=1&limit=20&active=true'
```

### 필터/정렬 쿼리 파라미터 관리

```typescript
interface FilterState {
  search?: string;
  category?: string;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
  page?: number;
  limit?: number;
}

class FilterManager {
  private url: URL;

  constructor(baseUrl: string = window.location.href) {
    this.url = new URL(baseUrl);
  }

  getFilters(): FilterState {
    const params = this.url.searchParams;

    return {
      search: params.get('search') || undefined,
      category: params.get('category') || undefined,
      sortBy: params.get('sortBy') || undefined,
      sortOrder: (params.get('sortOrder') as 'asc' | 'desc') || undefined,
      page: params.has('page') ? Number(params.get('page')) : undefined,
      limit: params.has('limit') ? Number(params.get('limit')) : undefined
    };
  }

  setFilters(filters: Partial<FilterState>): string {
    Object.entries(filters).forEach(([key, value]) => {
      if (value !== undefined && value !== null && value !== '') {
        this.url.searchParams.set(key, String(value));
      } else {
        this.url.searchParams.delete(key);
      }
    });

    return this.url.search;
  }

  resetFilters(): void {
    this.url.search = '';
  }

  getUrl(): string {
    return this.url.href;
  }
}

// 사용 예시
const filterManager = new FilterManager('https://shop.example.com/products');

filterManager.setFilters({
  category: 'laptops',
  sortBy: 'price',
  sortOrder: 'asc',
  page: 1
});

console.log(filterManager.getUrl());
// 'https://shop.example.com/products?category=laptops&sortBy=price&sortOrder=asc&page=1'
```

### 딥링크 및 공유 URL 생성

딥링크를 통해 특정 상태를 URL로 공유할 수 있습니다. 이는 [Web Storage API](/posts/javascript-web-storage-api-guide/)와 함께 사용하면 사용자 경험을 더욱 향상시킬 수 있습니다.

```typescript
interface ShareableState {
  productId: string;
  selectedColor?: string;
  selectedSize?: string;
  quantity?: number;
}

function createShareableUrl(state: ShareableState): string {
  const url = new URL(window.location.origin + '/product');

  url.searchParams.set('id', state.productId);

  if (state.selectedColor) {
    url.searchParams.set('color', state.selectedColor);
  }
  if (state.selectedSize) {
    url.searchParams.set('size', state.selectedSize);
  }
  if (state.quantity && state.quantity > 1) {
    url.searchParams.set('qty', String(state.quantity));
  }

  return url.href;
}

function parseShareableUrl(urlString: string): ShareableState | null {
  try {
    const url = new URL(urlString);
    const params = url.searchParams;

    const productId = params.get('id');
    if (!productId) return null;

    return {
      productId,
      selectedColor: params.get('color') || undefined,
      selectedSize: params.get('size') || undefined,
      quantity: params.has('qty') ? Number(params.get('qty')) : undefined
    };
  } catch {
    return null;
  }
}

// 사용 예시
const shareUrl = createShareableUrl({
  productId: 'abc123',
  selectedColor: 'navy',
  selectedSize: 'L',
  quantity: 2
});

console.log(shareUrl);
// 'https://example.com/product?id=abc123&color=navy&size=L&qty=2'

const parsed = parseShareableUrl(shareUrl);
console.log(parsed);
// { productId: 'abc123', selectedColor: 'navy', selectedSize: 'L', quantity: 2 }
```

---

## React/Next.js 연동

### Next.js App Router의 useSearchParams

[Next.js 15 App Router](/posts/nextjs-15-app-router-guide/)에서는 `useSearchParams` 훅을 사용하여 URL 쿼리 파라미터를 관리합니다.

```tsx
'use client';

import { useSearchParams, useRouter, usePathname } from 'next/navigation';
import { useCallback } from 'react';

function ProductFilters() {
  const searchParams = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();

  // 현재 필터값 읽기
  const category = searchParams.get('category') || 'all';
  const sortBy = searchParams.get('sortBy') || 'newest';
  const page = Number(searchParams.get('page')) || 1;

  // 필터 업데이트 함수
  const updateFilters = useCallback((updates: Record<string, string | null>) => {
    const params = new URLSearchParams(searchParams.toString());

    Object.entries(updates).forEach(([key, value]) => {
      if (value === null) {
        params.delete(key);
      } else {
        params.set(key, value);
      }
    });

    // 페이지가 변경되지 않은 경우 1페이지로 리셋
    if (!updates.hasOwnProperty('page')) {
      params.set('page', '1');
    }

    router.push(`${pathname}?${params.toString()}`);
  }, [searchParams, router, pathname]);

  return (
    <div>
      <select
        value={category}
        onChange={(e) => updateFilters({ category: e.target.value })}
      >
        <option value="all">전체</option>
        <option value="electronics">전자기기</option>
        <option value="clothing">의류</option>
      </select>

      <select
        value={sortBy}
        onChange={(e) => updateFilters({ sortBy: e.target.value })}
      >
        <option value="newest">최신순</option>
        <option value="price-asc">가격 낮은순</option>
        <option value="price-desc">가격 높은순</option>
      </select>

      <div>
        <button
          disabled={page <= 1}
          onClick={() => updateFilters({ page: String(page - 1) })}
        >
          이전
        </button>
        <span>페이지 {page}</span>
        <button
          onClick={() => updateFilters({ page: String(page + 1) })}
        >
          다음
        </button>
      </div>
    </div>
  );
}
```

### Next.js에서 재사용 가능한 훅

URL 상태 관리를 위한 [Custom Hook 패턴](/posts/react-custom-hooks-patterns-guide/)을 활용하면 코드 재사용성을 높일 수 있습니다.

```tsx
'use client';

import { useSearchParams, useRouter, usePathname } from 'next/navigation';
import { useCallback, useMemo } from 'react';

type ParamValue = string | number | boolean | null | undefined;

interface UseUrlStateOptions {
  shallow?: boolean;
}

function useUrlState<T extends Record<string, ParamValue>>(
  defaultValues: T,
  options: UseUrlStateOptions = {}
) {
  const searchParams = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();

  // URL에서 현재 상태 읽기
  const state = useMemo(() => {
    const result = { ...defaultValues } as T;

    Object.keys(defaultValues).forEach((key) => {
      const value = searchParams.get(key);
      if (value !== null) {
        const defaultValue = defaultValues[key];

        // 타입에 맞게 변환
        if (typeof defaultValue === 'number') {
          (result as Record<string, ParamValue>)[key] = Number(value);
        } else if (typeof defaultValue === 'boolean') {
          (result as Record<string, ParamValue>)[key] = value === 'true';
        } else {
          (result as Record<string, ParamValue>)[key] = value;
        }
      }
    });

    return result;
  }, [searchParams, defaultValues]);

  // 상태 업데이트 함수
  const setState = useCallback((updates: Partial<T>) => {
    const params = new URLSearchParams(searchParams.toString());

    Object.entries(updates).forEach(([key, value]) => {
      if (value === null || value === undefined || value === defaultValues[key]) {
        params.delete(key);
      } else {
        params.set(key, String(value));
      }
    });

    const queryString = params.toString();
    const url = queryString ? `${pathname}?${queryString}` : pathname;

    router.push(url, { scroll: false });
  }, [searchParams, router, pathname, defaultValues]);

  // 상태 초기화 함수
  const resetState = useCallback(() => {
    router.push(pathname, { scroll: false });
  }, [router, pathname]);

  return [state, setState, resetState] as const;
}

// 사용 예시
function ProductList() {
  const [filters, setFilters, resetFilters] = useUrlState({
    search: '',
    category: 'all',
    page: 1,
    limit: 20
  });

  return (
    <div>
      <input
        type="text"
        value={filters.search}
        onChange={(e) => setFilters({ search: e.target.value, page: 1 })}
        placeholder="검색..."
      />

      <select
        value={filters.category}
        onChange={(e) => setFilters({ category: e.target.value, page: 1 })}
      >
        <option value="all">전체</option>
        <option value="tech">기술</option>
      </select>

      <button onClick={resetFilters}>필터 초기화</button>

      <p>현재 페이지: {filters.page}</p>
    </div>
  );
}
```

### React Router와 URL 파라미터

React Router를 사용하는 경우의 패턴입니다.

```tsx
import { useSearchParams } from 'react-router-dom';
import { useCallback, useMemo } from 'react';

function useQueryParams<T extends Record<string, string>>(defaults: T) {
  const [searchParams, setSearchParams] = useSearchParams();

  const params = useMemo(() => {
    const result = { ...defaults };

    Object.keys(defaults).forEach((key) => {
      const value = searchParams.get(key);
      if (value !== null) {
        result[key as keyof T] = value as T[keyof T];
      }
    });

    return result;
  }, [searchParams, defaults]);

  const updateParams = useCallback((updates: Partial<T>) => {
    setSearchParams((prev) => {
      const newParams = new URLSearchParams(prev);

      Object.entries(updates).forEach(([key, value]) => {
        if (value === null || value === undefined || value === '') {
          newParams.delete(key);
        } else {
          newParams.set(key, value);
        }
      });

      return newParams;
    });
  }, [setSearchParams]);

  return [params, updateParams] as const;
}

// 사용 예시
function SearchPage() {
  const [params, updateParams] = useQueryParams({
    q: '',
    type: 'all',
    page: '1'
  });

  return (
    <div>
      <input
        value={params.q}
        onChange={(e) => updateParams({ q: e.target.value, page: '1' })}
      />
      <p>검색어: {params.q}</p>
    </div>
  );
}
```

---

## 고급 패턴

### TypeScript URL 빌더 클래스

```typescript
type QueryValue = string | number | boolean | null | undefined;
type QueryParams = Record<string, QueryValue | QueryValue[]>;

class UrlBuilder {
  private url: URL;

  constructor(baseUrl: string) {
    this.url = new URL(baseUrl);
  }

  static from(baseUrl: string): UrlBuilder {
    return new UrlBuilder(baseUrl);
  }

  path(...segments: string[]): UrlBuilder {
    const cleanSegments = segments
      .map((s) => s.replace(/^\/+|\/+$/g, ''))
      .filter(Boolean);

    this.url.pathname = '/' + cleanSegments.join('/');
    return this;
  }

  appendPath(segment: string): UrlBuilder {
    const clean = segment.replace(/^\/+|\/+$/g, '');
    this.url.pathname = this.url.pathname.replace(/\/?$/, '/') + clean;
    return this;
  }

  query(params: QueryParams): UrlBuilder {
    Object.entries(params).forEach(([key, value]) => {
      if (value === null || value === undefined) {
        this.url.searchParams.delete(key);
        return;
      }

      if (Array.isArray(value)) {
        // 배열 값 처리
        this.url.searchParams.delete(key);
        value.forEach((v) => {
          if (v !== null && v !== undefined) {
            this.url.searchParams.append(key, String(v));
          }
        });
      } else {
        this.url.searchParams.set(key, String(value));
      }
    });

    return this;
  }

  hash(fragment: string): UrlBuilder {
    this.url.hash = fragment.startsWith('#') ? fragment : `#${fragment}`;
    return this;
  }

  toString(): string {
    return this.url.href;
  }

  toURL(): URL {
    return new URL(this.url.href);
  }
}

// 사용 예시
const apiUrl = UrlBuilder
  .from('https://api.example.com')
  .path('v1', 'products')
  .query({
    category: 'electronics',
    tags: ['featured', 'sale'],
    page: 1,
    limit: 20,
    inactive: null // null은 제외됨
  })
  .toString();

console.log(apiUrl);
// 'https://api.example.com/v1/products?category=electronics&tags=featured&tags=sale&page=1&limit=20'

const productUrl = UrlBuilder
  .from('https://shop.example.com')
  .path('products', 'laptop-abc123')
  .query({ ref: 'homepage' })
  .hash('reviews')
  .toString();

console.log(productUrl);
// 'https://shop.example.com/products/laptop-abc123?ref=homepage#reviews'
```

### 배열/객체 파라미터 직렬화

```typescript
// 배열 직렬화 방식들
const colors = ['red', 'blue', 'green'];

// 1. 반복 키 방식 (PHP 스타일)
const params1 = new URLSearchParams();
colors.forEach((color) => params1.append('colors[]', color));
console.log(params1.toString());
// 'colors%5B%5D=red&colors%5B%5D=blue&colors%5B%5D=green'

// 2. 동일 키 반복 방식 (일반적)
const params2 = new URLSearchParams();
colors.forEach((color) => params2.append('colors', color));
console.log(params2.toString());
// 'colors=red&colors=blue&colors=green'

// 3. 쉼표 구분 방식
const params3 = new URLSearchParams();
params3.set('colors', colors.join(','));
console.log(params3.toString());
// 'colors=red%2Cblue%2Cgreen'

// 4. JSON 방식
const params4 = new URLSearchParams();
params4.set('colors', JSON.stringify(colors));
console.log(params4.toString());
// 'colors=%5B%22red%22%2C%22blue%22%2C%22green%22%5D'

// 객체 직렬화 유틸리티
function serializeParams(
  obj: Record<string, unknown>,
  prefix: string = ''
): URLSearchParams {
  const params = new URLSearchParams();

  function serialize(value: unknown, key: string): void {
    if (value === null || value === undefined) {
      return;
    }

    if (Array.isArray(value)) {
      value.forEach((item, index) => {
        serialize(item, `${key}[${index}]`);
      });
    } else if (typeof value === 'object') {
      Object.entries(value as Record<string, unknown>).forEach(([k, v]) => {
        serialize(v, key ? `${key}[${k}]` : k);
      });
    } else {
      params.append(key, String(value));
    }
  }

  serialize(obj, prefix);
  return params;
}

// 사용 예시
const filters = {
  price: { min: 100, max: 500 },
  categories: ['electronics', 'computers'],
  inStock: true
};

console.log(serializeParams(filters).toString());
// 'price%5Bmin%5D=100&price%5Bmax%5D=500&categories%5B0%5D=electronics&categories%5B1%5D=computers&inStock=true'
```

### URL 정규화 및 비교

```typescript
function normalizeUrl(urlString: string): string {
  try {
    const url = new URL(urlString);

    // 1. 프로토콜 소문자
    url.protocol = url.protocol.toLowerCase();

    // 2. 호스트명 소문자
    url.hostname = url.hostname.toLowerCase();

    // 3. 기본 포트 제거
    if (
      (url.protocol === 'http:' && url.port === '80') ||
      (url.protocol === 'https:' && url.port === '443')
    ) {
      url.port = '';
    }

    // 4. 경로 정규화 (후행 슬래시 일관성)
    if (url.pathname !== '/' && url.pathname.endsWith('/')) {
      url.pathname = url.pathname.slice(0, -1);
    }

    // 5. 쿼리 파라미터 정렬
    url.searchParams.sort();

    // 6. 빈 해시 제거
    if (url.hash === '#') {
      url.hash = '';
    }

    return url.href;
  } catch {
    return urlString;
  }
}

function urlsEqual(url1: string, url2: string): boolean {
  return normalizeUrl(url1) === normalizeUrl(url2);
}

// 사용 예시
console.log(normalizeUrl('HTTP://Example.COM:80/path/?b=2&a=1'));
// 'http://example.com/path?a=1&b=2'

console.log(urlsEqual(
  'https://example.com/path?a=1&b=2',
  'https://example.com/path/?b=2&a=1'
));
// true
```

### 보안 고려사항

URL을 다룰 때 보안에 주의해야 합니다. 더 자세한 보안 가이드는 [프론트엔드 보안 가이드](/posts/frontend-security-guide/)를 참고하세요.

```typescript
// 1. 사용자 입력 URL 검증
function isSafeUrl(urlString: string, allowedHosts: string[]): boolean {
  try {
    const url = new URL(urlString);

    // 허용된 프로토콜만
    if (!['http:', 'https:'].includes(url.protocol)) {
      return false;
    }

    // 허용된 호스트만
    if (!allowedHosts.includes(url.hostname)) {
      return false;
    }

    return true;
  } catch {
    return false;
  }
}

// 2. Open Redirect 방지
function safeRedirect(targetUrl: string, fallback: string = '/'): string {
  try {
    const url = new URL(targetUrl, window.location.origin);

    // 같은 출처인지 확인
    if (url.origin !== window.location.origin) {
      console.warn('외부 URL로의 리다이렉트 차단:', targetUrl);
      return fallback;
    }

    return url.pathname + url.search + url.hash;
  } catch {
    return fallback;
  }
}

// 3. XSS 방지를 위한 안전한 URL 생성
function createSafeLink(userInput: string): string {
  const url = new URL('https://search.example.com/results');

  // URLSearchParams가 자동으로 인코딩 처리
  url.searchParams.set('q', userInput);

  return url.href;
}

// 위험한 예 (직접 문자열 결합)
// const dangerous = `https://example.com/search?q=${userInput}`; // XSS 위험!

// 안전한 예
const safeUrl = createSafeLink('<script>alert("xss")</script>');
console.log(safeUrl);
// 'https://search.example.com/results?q=%3Cscript%3Ealert%28%22xss%22%29%3C%2Fscript%3E'

// 4. JavaScript URL 스킴 차단
function sanitizeHref(href: string): string {
  const dangerous = /^(javascript|data|vbscript):/i;

  if (dangerous.test(href.trim())) {
    return '#';
  }

  return href;
}

console.log(sanitizeHref('javascript:alert(1)')); // '#'
console.log(sanitizeHref('https://example.com')); // 'https://example.com'
```

> 사용자 입력을 URL에 포함할 때는 반드시 URLSearchParams나 URL 객체를 사용하세요. 직접 문자열을 결합하면 XSS 공격에 취약해질 수 있습니다.
{: .prompt-danger }

---

## 브라우저 History API 연동

### pushState/replaceState와 URL 업데이트

History API를 사용하면 페이지 새로고침 없이 URL을 변경할 수 있습니다.

```typescript
interface AppState {
  page: string;
  filters: Record<string, string>;
}

// URL 업데이트 (히스토리에 추가)
function navigateTo(state: AppState): void {
  const url = new URL(window.location.href);

  url.pathname = `/${state.page}`;
  url.search = '';

  Object.entries(state.filters).forEach(([key, value]) => {
    if (value) {
      url.searchParams.set(key, value);
    }
  });

  // 히스토리에 새 항목 추가
  window.history.pushState(state, '', url.href);

  // 페이지 콘텐츠 업데이트
  updatePageContent(state);
}

// URL 교체 (히스토리 항목 교체)
function replaceCurrentState(state: AppState): void {
  const url = new URL(window.location.href);

  url.pathname = `/${state.page}`;
  url.search = '';

  Object.entries(state.filters).forEach(([key, value]) => {
    if (value) {
      url.searchParams.set(key, value);
    }
  });

  // 현재 히스토리 항목 교체 (뒤로가기에 남지 않음)
  window.history.replaceState(state, '', url.href);

  updatePageContent(state);
}

function updatePageContent(state: AppState): void {
  console.log('페이지 업데이트:', state);
  // 실제 DOM 업데이트 로직
}
```

### popstate 이벤트 처리

사용자가 브라우저의 뒤로가기/앞으로가기 버튼을 클릭할 때 `popstate` 이벤트가 발생합니다.

```typescript
// 현재 URL에서 상태 파싱
function parseCurrentUrl(): AppState {
  const url = new URL(window.location.href);

  return {
    page: url.pathname.slice(1) || 'home',
    filters: Object.fromEntries(url.searchParams)
  };
}

// popstate 이벤트 리스너
function handlePopState(event: PopStateEvent): void {
  // state가 있으면 사용, 없으면 URL에서 파싱
  const state = event.state || parseCurrentUrl();

  console.log('뒤로가기/앞으로가기:', state);
  updatePageContent(state);
}

// 이벤트 리스너 등록
window.addEventListener('popstate', handlePopState);

// 초기 페이지 로드 시 상태 설정
function initializeState(): void {
  const initialState = parseCurrentUrl();

  // 현재 URL의 상태를 히스토리에 저장 (replace 사용)
  window.history.replaceState(initialState, '', window.location.href);

  updatePageContent(initialState);
}

// 앱 시작 시 호출
initializeState();
```

### SPA 라우터 구현 예시

```typescript
type RouteHandler = (params: Record<string, string>) => void;

interface Route {
  pattern: RegExp;
  keys: string[];
  handler: RouteHandler;
}

class SimpleRouter {
  private routes: Route[] = [];

  add(path: string, handler: RouteHandler): void {
    // 경로 패턴을 정규식으로 변환
    const keys: string[] = [];
    const pattern = path.replace(/:(\w+)/g, (_, key) => {
      keys.push(key);
      return '([^/]+)';
    });

    this.routes.push({
      pattern: new RegExp(`^${pattern}$`),
      keys,
      handler
    });
  }

  navigate(path: string, replace: boolean = false): void {
    const url = new URL(path, window.location.origin);

    if (replace) {
      window.history.replaceState({ path }, '', url.href);
    } else {
      window.history.pushState({ path }, '', url.href);
    }

    this.handleRoute(url.pathname, url.searchParams);
  }

  private handleRoute(pathname: string, searchParams: URLSearchParams): void {
    for (const route of this.routes) {
      const match = pathname.match(route.pattern);

      if (match) {
        // URL 파라미터 추출
        const params: Record<string, string> = {};
        route.keys.forEach((key, index) => {
          params[key] = match[index + 1];
        });

        // 쿼리 파라미터 추가
        searchParams.forEach((value, key) => {
          params[key] = value;
        });

        route.handler(params);
        return;
      }
    }

    console.warn('404: 라우트를 찾을 수 없습니다:', pathname);
  }

  init(): void {
    // popstate 이벤트 처리
    window.addEventListener('popstate', () => {
      const url = new URL(window.location.href);
      this.handleRoute(url.pathname, url.searchParams);
    });

    // 초기 라우트 처리
    const url = new URL(window.location.href);
    this.handleRoute(url.pathname, url.searchParams);
  }
}

// 사용 예시
const router = new SimpleRouter();

router.add('/', (params) => {
  console.log('홈페이지');
});

router.add('/products', (params) => {
  console.log('상품 목록, 필터:', params);
});

router.add('/products/:id', (params) => {
  console.log('상품 상세, ID:', params.id);
});

router.add('/users/:userId/posts/:postId', (params) => {
  console.log('사용자 포스트:', params.userId, params.postId);
});

router.init();

// 네비게이션
router.navigate('/products?category=electronics&page=1');
router.navigate('/products/abc123');
```

---

## 실용적인 유틸리티 함수

### 쿼리 파라미터 병합 함수

```typescript
type MergeMode = 'replace' | 'append' | 'keep';

function mergeSearchParams(
  base: URLSearchParams | string,
  updates: Record<string, string | string[] | null | undefined>,
  mode: MergeMode = 'replace'
): URLSearchParams {
  const result = new URLSearchParams(
    typeof base === 'string' ? base : base.toString()
  );

  Object.entries(updates).forEach(([key, value]) => {
    if (value === null || value === undefined) {
      result.delete(key);
      return;
    }

    switch (mode) {
      case 'replace':
        // 기존 값을 새 값으로 교체
        result.delete(key);
        if (Array.isArray(value)) {
          value.forEach((v) => result.append(key, v));
        } else {
          result.set(key, value);
        }
        break;

      case 'append':
        // 기존 값에 추가
        if (Array.isArray(value)) {
          value.forEach((v) => result.append(key, v));
        } else {
          result.append(key, value);
        }
        break;

      case 'keep':
        // 기존 값이 없을 때만 설정
        if (!result.has(key)) {
          if (Array.isArray(value)) {
            value.forEach((v) => result.append(key, v));
          } else {
            result.set(key, value);
          }
        }
        break;
    }
  });

  return result;
}

// 사용 예시
const current = new URLSearchParams('page=1&category=all');

const merged = mergeSearchParams(current, {
  page: '2',
  category: null, // 삭제
  sort: 'price',
  tags: ['featured', 'new']
});

console.log(merged.toString());
// 'page=2&sort=price&tags=featured&tags=new'
```

### URL 비교 함수

```typescript
interface UrlCompareOptions {
  ignoreHash?: boolean;
  ignoreSearch?: boolean;
  ignorePort?: boolean;
  ignoreProtocol?: boolean;
  ignoreTrailingSlash?: boolean;
}

function compareUrls(
  url1: string,
  url2: string,
  options: UrlCompareOptions = {}
): boolean {
  try {
    const a = new URL(url1);
    const b = new URL(url2);

    // 프로토콜 비교
    if (!options.ignoreProtocol && a.protocol !== b.protocol) {
      return false;
    }

    // 호스트명 비교 (항상 소문자로)
    if (a.hostname.toLowerCase() !== b.hostname.toLowerCase()) {
      return false;
    }

    // 포트 비교
    if (!options.ignorePort && a.port !== b.port) {
      return false;
    }

    // 경로 비교
    let pathA = a.pathname;
    let pathB = b.pathname;

    if (options.ignoreTrailingSlash) {
      pathA = pathA.replace(/\/+$/, '');
      pathB = pathB.replace(/\/+$/, '');
    }

    if (pathA !== pathB) {
      return false;
    }

    // 쿼리 스트링 비교
    if (!options.ignoreSearch) {
      a.searchParams.sort();
      b.searchParams.sort();

      if (a.searchParams.toString() !== b.searchParams.toString()) {
        return false;
      }
    }

    // 해시 비교
    if (!options.ignoreHash && a.hash !== b.hash) {
      return false;
    }

    return true;
  } catch {
    return url1 === url2;
  }
}

// 현재 URL과 일치하는지 확인 (네비게이션 하이라이트 용도)
function isCurrentPath(path: string): boolean {
  return compareUrls(window.location.href, path, {
    ignoreHash: true,
    ignoreSearch: true,
    ignoreTrailingSlash: true
  });
}

// 사용 예시
console.log(compareUrls(
  'https://example.com/path/',
  'https://example.com/path',
  { ignoreTrailingSlash: true }
)); // true

console.log(compareUrls(
  'https://example.com/path?a=1&b=2',
  'https://example.com/path?b=2&a=1'
)); // true (정렬 후 비교)
```

### 안전한 URL 파싱 함수

```typescript
interface ParsedUrl {
  isValid: boolean;
  url: URL | null;
  protocol: string | null;
  hostname: string | null;
  port: string | null;
  pathname: string | null;
  search: string | null;
  searchParams: Record<string, string> | null;
  hash: string | null;
  origin: string | null;
  error: string | null;
}

function safeParseUrl(urlString: string, base?: string): ParsedUrl {
  try {
    const url = base
      ? new URL(urlString, base)
      : new URL(urlString);

    return {
      isValid: true,
      url,
      protocol: url.protocol,
      hostname: url.hostname,
      port: url.port,
      pathname: url.pathname,
      search: url.search,
      searchParams: Object.fromEntries(url.searchParams),
      hash: url.hash,
      origin: url.origin,
      error: null
    };
  } catch (error) {
    return {
      isValid: false,
      url: null,
      protocol: null,
      hostname: null,
      port: null,
      pathname: null,
      search: null,
      searchParams: null,
      hash: null,
      origin: null,
      error: error instanceof Error ? error.message : 'Invalid URL'
    };
  }
}

// 사용 예시
const result1 = safeParseUrl('https://example.com/path?key=value');
if (result1.isValid) {
  console.log('호스트:', result1.hostname);
  console.log('파라미터:', result1.searchParams);
}

const result2 = safeParseUrl('not a valid url');
if (!result2.isValid) {
  console.log('파싱 실패:', result2.error);
}

// 상대 URL 파싱
const result3 = safeParseUrl('/api/users', 'https://api.example.com');
console.log(result3.url?.href); // 'https://api.example.com/api/users'
```

### URL 템플릿 함수

```typescript
type PathParams = Record<string, string | number>;

function buildUrlFromTemplate(
  template: string,
  pathParams: PathParams = {},
  queryParams: Record<string, string | number | boolean | null | undefined> = {}
): string {
  // 1. 경로 파라미터 치환
  let path = template.replace(/:(\w+)/g, (_, key) => {
    const value = pathParams[key];
    if (value === undefined) {
      throw new Error(`Missing path parameter: ${key}`);
    }
    return encodeURIComponent(String(value));
  });

  // 2. 쿼리 파라미터 추가
  const searchParams = new URLSearchParams();

  Object.entries(queryParams).forEach(([key, value]) => {
    if (value !== null && value !== undefined) {
      searchParams.set(key, String(value));
    }
  });

  const queryString = searchParams.toString();

  return queryString ? `${path}?${queryString}` : path;
}

// API 엔드포인트 정의
const API_ENDPOINTS = {
  users: '/api/users',
  user: '/api/users/:id',
  userPosts: '/api/users/:userId/posts',
  post: '/api/users/:userId/posts/:postId'
} as const;

// 사용 예시
const userUrl = buildUrlFromTemplate(
  API_ENDPOINTS.user,
  { id: 123 }
);
console.log(userUrl); // '/api/users/123'

const userPostsUrl = buildUrlFromTemplate(
  API_ENDPOINTS.userPosts,
  { userId: 456 },
  { page: 1, limit: 10, published: true }
);
console.log(userPostsUrl);
// '/api/users/456/posts?page=1&limit=10&published=true'
```

---

## 브라우저 지원 및 호환성

### 지원 현황

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| URL | 32+ | 26+ | 7+ | 12+ |
| URLSearchParams | 49+ | 44+ | 10.1+ | 17+ |
| URL.canParse() | 120+ | 115+ | 17+ | 120+ |

### 폴리필

구형 브라우저를 지원해야 할 경우 폴리필을 사용할 수 있습니다.

```javascript
// core-js 사용
import 'core-js/features/url';
import 'core-js/features/url-search-params';

// 또는 whatwg-url 패키지 사용
import { URL, URLSearchParams } from 'whatwg-url';
```

---

## 정리

URL API와 URLSearchParams는 웹 개발에서 URL을 다루는 표준적이고 안전한 방법을 제공합니다.

### 핵심 요약

| 기능 | 설명 |
|------|------|
| URL 객체 | URL 파싱 및 각 구성 요소 접근 |
| URLSearchParams | 쿼리 파라미터 조작 |
| 자동 인코딩 | 특수문자 자동 처리로 XSS 방지 |
| 불변성 없음 | 속성 수정 가능 (주의 필요) |
| History API 연동 | SPA에서 URL 기반 상태 관리 |

### 모범 사례

1. **문자열 결합 대신 URL/URLSearchParams 사용**: XSS 공격을 방지하고 인코딩을 자동화합니다.
2. **사용자 입력 URL 검증**: `try-catch`로 유효성을 확인하고 허용된 도메인만 허용합니다.
3. **URL 정규화**: 비교 전에 정규화하여 일관성을 유지합니다.
4. **React/Next.js 훅 활용**: 프레임워크가 제공하는 훅으로 URL 상태를 관리합니다.
5. **TypeScript 활용**: 타입 안전한 URL 빌더와 파서를 구현합니다.

---

## 참고 자료

- [MDN - URL API](https://developer.mozilla.org/ko/docs/Web/API/URL)
- [MDN - URLSearchParams](https://developer.mozilla.org/ko/docs/Web/API/URLSearchParams)
- [MDN - History API](https://developer.mozilla.org/ko/docs/Web/API/History_API)
- [WHATWG URL Standard](https://url.spec.whatwg.org/)
- [Next.js - useSearchParams](https://nextjs.org/docs/app/api-reference/functions/use-search-params)
- [React Router - useSearchParams](https://reactrouter.com/en/main/hooks/use-search-params)
