---
title: "React Custom Hooks 패턴 완벽 가이드: 재사용 가능한 로직 설계"
description: "useToggle, useDebounce, useAsync 등 실무에서 자주 사용하는 React 커스텀 훅 15가지 패턴을 TypeScript 코드 예제와 함께 설계 원칙, 테스트 방법까지 상세히 다룹니다."
date: 2025-12-05 14:00:00 +0900
categories: [Frontend, React]
tags: [react, hooks, custom-hooks, typescript, usestate, useeffect, reusable-logic, patterns]
---

## 개요

React를 사용하다 보면 여러 컴포넌트에서 동일한 로직이 반복되는 경우를 자주 만나게 됩니다. 토글 상태 관리, API 호출, 로컬 스토리지 연동 등 비슷한 코드를 매번 작성하는 것은 비효율적입니다. 이러한 문제를 해결하는 것이 바로 **Custom Hook(커스텀 훅)** 입니다.

이 글에서는 실무에서 자주 사용되는 커스텀 훅 패턴들을 소개하고, 직접 구현하면서 설계 원칙과 베스트 프랙티스를 배워보겠습니다. 훅의 성능 최적화에 대해서는 [React Hooks 성능 최적화 가이드](/posts/react-hooks-performance-optimization/)를 참고하세요.

### 학습 목표

- Custom Hook의 개념과 필요성 이해
- 다양한 커스텀 훅 패턴 학습 및 구현
- TypeScript를 활용한 타입 안전한 훅 설계
- 테스트 작성법과 베스트 프랙티스 습득

### 사전 지식

- React 기본 Hooks (useState, useEffect, useRef) 이해
- TypeScript 기초 문법 ([TypeScript 고급 타입 패턴](/posts/typescript-advanced-type-patterns/) 참고)
- ES6+ JavaScript 문법

---

## Custom Hook이란?

Custom Hook은 React의 내장 훅(useState, useEffect 등)을 활용하여 **재사용 가능한 로직을 캡슐화**한 함수입니다. 컴포넌트에서 상태 관련 로직을 추출하여 독립적으로 테스트하고 재사용할 수 있게 해줍니다.

### 왜 Custom Hook이 필요한가?

다음과 같은 토글 로직이 여러 컴포넌트에서 사용된다고 가정해봅시다:

```tsx
function ModalComponent() {
  const [isOpen, setIsOpen] = useState(false);

  const toggle = () => setIsOpen(prev => !prev);
  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);

  return (
    <div>
      <button onClick={toggle}>Toggle Modal</button>
      {isOpen && <Modal onClose={close} />}
    </div>
  );
}
```

이 로직이 10개의 컴포넌트에서 필요하다면? 매번 같은 코드를 복사하는 것은 다음과 같은 문제를 야기합니다:

1. **코드 중복**: 동일한 로직이 여러 곳에 산재
2. **유지보수 어려움**: 로직 변경 시 모든 곳을 수정해야 함
3. **테스트 복잡성**: 각 컴포넌트마다 동일한 테스트 필요
4. **일관성 부재**: 구현 방식이 조금씩 달라질 수 있음

Custom Hook으로 추출하면 이 모든 문제가 해결됩니다:

```tsx
// useToggle 커스텀 훅
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(prev => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
}

// 사용하는 컴포넌트
function ModalComponent() {
  const modal = useToggle(false);

  return (
    <div>
      <button onClick={modal.toggle}>Toggle Modal</button>
      {modal.value && <Modal onClose={modal.setFalse} />}
    </div>
  );
}
```

---

## Custom Hook 작성 규칙

### 1. use 접두사 필수

모든 커스텀 훅은 반드시 `use`로 시작해야 합니다. 이는 단순한 컨벤션이 아니라 React가 훅 규칙 위반을 감지하기 위해 필요합니다.

```tsx
// 올바른 예
function useCounter() { /* ... */ }
function useLocalStorage() { /* ... */ }
function useWindowSize() { /* ... */ }

// 잘못된 예 - React가 훅으로 인식하지 못함
function counter() { /* ... */ }
function getLocalStorage() { /* ... */ }
function windowSizeHook() { /* ... */ }
```

### 2. 훅 규칙 준수

커스텀 훅 내부에서도 React의 훅 규칙을 반드시 준수해야 합니다:

```tsx
// 잘못된 예 - 조건문 안에서 훅 호출
function useBadExample(condition: boolean) {
  if (condition) {
    const [state, setState] = useState(0); // 위반!
  }
  // ...
}

// 올바른 예
function useGoodExample(condition: boolean) {
  const [state, setState] = useState(0);

  useEffect(() => {
    if (condition) {
      // 조건부 로직은 여기서 처리
    }
  }, [condition]);

  return state;
}
```

### 3. 반환값 설계

커스텀 훅의 반환값은 사용 패턴에 따라 설계합니다:

```tsx
// 배열 반환 - useState처럼 구조 분해 시 이름 자유롭게 지정
function useCounter(initial: number) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(c => c + 1);
  return [count, increment] as const;
}

const [clicks, addClick] = useCounter(0);
const [views, addView] = useCounter(0);

// 객체 반환 - 여러 값을 반환할 때 명확함
function useAsync<T>() {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  return { data, loading, error };
}

const { data, loading, error } = useAsync<User[]>();
```

---

## 상태 관리 훅 패턴

### useToggle - Boolean 토글

가장 기본적이면서도 자주 사용되는 훅입니다.

```tsx
import { useState, useCallback } from 'react';

interface UseToggleReturn {
  value: boolean;
  toggle: () => void;
  setTrue: () => void;
  setFalse: () => void;
  setValue: (value: boolean) => void;
}

function useToggle(initialValue = false): UseToggleReturn {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(prev => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse, setValue };
}

export default useToggle;
```

**사용 예시:**

```tsx
function DarkModeToggle() {
  const darkMode = useToggle(false);

  return (
    <div>
      <button onClick={darkMode.toggle}>
        {darkMode.value ? '라이트 모드로 전환' : '다크 모드로 전환'}
      </button>
      <p>현재 모드: {darkMode.value ? '다크' : '라이트'}</p>
    </div>
  );
}
```

### useCounter - 숫자 증감

숫자 상태를 다루는 다양한 기능을 제공합니다.

```tsx
import { useState, useCallback } from 'react';

interface UseCounterOptions {
  min?: number;
  max?: number;
  step?: number;
}

interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  set: (value: number) => void;
}

function useCounter(
  initialValue = 0,
  options: UseCounterOptions = {}
): UseCounterReturn {
  const { min = -Infinity, max = Infinity, step = 1 } = options;
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount(prev => Math.min(prev + step, max));
  }, [step, max]);

  const decrement = useCallback(() => {
    setCount(prev => Math.max(prev - step, min));
  }, [step, min]);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  const set = useCallback((value: number) => {
    setCount(Math.max(min, Math.min(value, max)));
  }, [min, max]);

  return { count, increment, decrement, reset, set };
}

export default useCounter;
```

**사용 예시:**

```tsx
function QuantitySelector() {
  const quantity = useCounter(1, { min: 1, max: 99, step: 1 });

  return (
    <div>
      <button onClick={quantity.decrement} disabled={quantity.count <= 1}>
        -
      </button>
      <span>{quantity.count}</span>
      <button onClick={quantity.increment} disabled={quantity.count >= 99}>
        +
      </button>
      <button onClick={quantity.reset}>초기화</button>
    </div>
  );
}
```

### useInput - 입력 필드 관리

폼 입력 필드의 상태와 이벤트 핸들러를 관리합니다.

{% raw %}
```tsx
import { useState, useCallback, ChangeEvent } from 'react';

interface UseInputOptions<T> {
  validate?: (value: T) => string | null;
  transform?: (value: string) => T;
}

interface UseInputReturn<T> {
  value: T;
  onChange: (e: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void;
  reset: () => void;
  setValue: (value: T) => void;
  error: string | null;
  isDirty: boolean;
  bind: {
    value: T;
    onChange: (e: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void;
  };
}

function useInput<T extends string | number = string>(
  initialValue: T,
  options: UseInputOptions<T> = {}
): UseInputReturn<T> {
  const { validate, transform } = options;
  const [value, setValue] = useState<T>(initialValue);
  const [error, setError] = useState<string | null>(null);
  const [isDirty, setIsDirty] = useState(false);

  const onChange = useCallback((
    e: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const rawValue = e.target.value;
    const newValue = transform
      ? transform(rawValue)
      : (rawValue as T);

    setValue(newValue);
    setIsDirty(true);

    if (validate) {
      setError(validate(newValue));
    }
  }, [validate, transform]);

  const reset = useCallback(() => {
    setValue(initialValue);
    setError(null);
    setIsDirty(false);
  }, [initialValue]);

  return {
    value,
    onChange,
    reset,
    setValue,
    error,
    isDirty,
    bind: { value, onChange },
  };
}

export default useInput;
```
{% endraw %}

**사용 예시:**

{% raw %}
```tsx
function LoginForm() {
  const email = useInput('', {
    validate: (value) => {
      if (!value.includes('@')) return '유효한 이메일을 입력하세요';
      return null;
    },
  });

  const password = useInput('', {
    validate: (value) => {
      if (value.length < 8) return '비밀번호는 8자 이상이어야 합니다';
      return null;
    },
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!email.error && !password.error) {
      console.log('로그인:', { email: email.value, password: password.value });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input type="email" placeholder="이메일" {...email.bind} />
        {email.isDirty && email.error && (
          <span style={{ color: 'red' }}>{email.error}</span>
        )}
      </div>
      <div>
        <input type="password" placeholder="비밀번호" {...password.bind} />
        {password.isDirty && password.error && (
          <span style={{ color: 'red' }}>{password.error}</span>
        )}
      </div>
      <button type="submit">로그인</button>
    </form>
  );
}
```
{% endraw %}

### useArray - 배열 조작

배열 상태를 다루는 다양한 유틸리티를 제공합니다.

```tsx
import { useState, useCallback } from 'react';

interface UseArrayReturn<T> {
  array: T[];
  set: (newArray: T[]) => void;
  push: (element: T) => void;
  filter: (callback: (element: T, index: number) => boolean) => void;
  update: (index: number, newElement: T) => void;
  remove: (index: number) => void;
  clear: () => void;
  isEmpty: boolean;
}

function useArray<T>(initialArray: T[] = []): UseArrayReturn<T> {
  const [array, setArray] = useState<T[]>(initialArray);

  const push = useCallback((element: T) => {
    setArray(prev => [...prev, element]);
  }, []);

  const filter = useCallback((
    callback: (element: T, index: number) => boolean
  ) => {
    setArray(prev => prev.filter(callback));
  }, []);

  const update = useCallback((index: number, newElement: T) => {
    setArray(prev => {
      const copy = [...prev];
      copy[index] = newElement;
      return copy;
    });
  }, []);

  const remove = useCallback((index: number) => {
    setArray(prev => prev.filter((_, i) => i !== index));
  }, []);

  const clear = useCallback(() => {
    setArray([]);
  }, []);

  return {
    array,
    set: setArray,
    push,
    filter,
    update,
    remove,
    clear,
    isEmpty: array.length === 0,
  };
}

export default useArray;
```

**사용 예시:**

```tsx
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

function TodoList() {
  const todos = useArray<Todo>([
    { id: 1, text: 'React 공부하기', completed: false },
    { id: 2, text: '커스텀 훅 만들기', completed: false },
  ]);
  const newTodo = useInput('');

  const addTodo = () => {
    if (newTodo.value.trim()) {
      todos.push({
        id: Date.now(),
        text: newTodo.value,
        completed: false,
      });
      newTodo.reset();
    }
  };

  const toggleTodo = (index: number) => {
    const todo = todos.array[index];
    todos.update(index, { ...todo, completed: !todo.completed });
  };

  return (
    <div>
      <input {...newTodo.bind} placeholder="새 할일 입력" />
      <button onClick={addTodo}>추가</button>
      <ul>
        {todos.array.map((todo, index) => (
          <li key={todo.id}>
            <span
              onClick={() => toggleTodo(index)}
              style={% raw %}{{ textDecoration: todo.completed ? 'line-through' : 'none' }}{% endraw %}
            >
              {todo.text}
            </span>
            <button onClick={() => todos.remove(index)}>삭제</button>
          </li>
        ))}
      </ul>
      {!todos.isEmpty && (
        <button onClick={todos.clear}>모두 삭제</button>
      )}
    </div>
  );
}
```

---

## 사이드 이펙트 훅 패턴

### useLocalStorage - 로컬 스토리지 연동

로컬 스토리지와 React 상태를 동기화합니다.

```tsx
import { useState, useEffect, useCallback } from 'react';

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
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  }, [initialValue, key]);

  const [storedValue, setStoredValue] = useState<T>(readValue);

  // 값 설정
  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    try {
      const valueToStore = value instanceof Function
        ? value(storedValue)
        : value;

      setStoredValue(valueToStore);

      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));

        // 다른 탭/창과 동기화를 위한 커스텀 이벤트
        window.dispatchEvent(new StorageEvent('storage', {
          key,
          newValue: JSON.stringify(valueToStore),
        }));
      }
    } catch (error) {
      console.warn(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  // 값 제거
  const removeValue = useCallback(() => {
    try {
      if (typeof window !== 'undefined') {
        window.localStorage.removeItem(key);
        setStoredValue(initialValue);
      }
    } catch (error) {
      console.warn(`Error removing localStorage key "${key}":`, error);
    }
  }, [key, initialValue]);

  // 다른 탭에서 변경 시 동기화
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

export default useLocalStorage;
```

**사용 예시:**

```tsx
interface UserPreferences {
  theme: 'light' | 'dark';
  fontSize: number;
  language: string;
}

function SettingsPanel() {
  const [preferences, setPreferences, resetPreferences] =
    useLocalStorage<UserPreferences>('user-preferences', {
      theme: 'light',
      fontSize: 16,
      language: 'ko',
    });

  return (
    <div>
      <h3>설정</h3>
      <div>
        <label>테마:</label>
        <select
          value={preferences.theme}
          onChange={(e) => setPreferences(prev => ({
            ...prev,
            theme: e.target.value as 'light' | 'dark',
          }))}
        >
          <option value="light">라이트</option>
          <option value="dark">다크</option>
        </select>
      </div>
      <div>
        <label>글자 크기: {preferences.fontSize}px</label>
        <input
          type="range"
          min="12"
          max="24"
          value={preferences.fontSize}
          onChange={(e) => setPreferences(prev => ({
            ...prev,
            fontSize: Number(e.target.value),
          }))}
        />
      </div>
      <button onClick={resetPreferences}>설정 초기화</button>
    </div>
  );
}
```

### useDebounce - 디바운스

값의 변경을 지연시켜 불필요한 연산을 줄입니다.

```tsx
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

export default useDebounce;
```

**함수 버전 (콜백 디바운스):**

```tsx
import { useCallback, useRef } from 'react';

function useDebouncedCallback<T extends (...args: unknown[]) => unknown>(
  callback: T,
  delay: number
): (...args: Parameters<T>) => void {
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  return useCallback((...args: Parameters<T>) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }

    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

export default useDebouncedCallback;
```

**사용 예시:**

```tsx
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearch = useDebounce(searchTerm, 500);
  const [results, setResults] = useState<string[]>([]);

  useEffect(() => {
    if (debouncedSearch) {
      // API 호출은 디바운스된 값이 변경될 때만 실행
      fetch(`/api/search?q=${debouncedSearch}`)
        .then(res => res.json())
        .then(data => setResults(data));
    }
  }, [debouncedSearch]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="검색어 입력..."
      />
      <p>검색 중: {debouncedSearch}</p>
      <ul>
        {results.map((result, i) => (
          <li key={i}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useThrottle - 쓰로틀

일정 시간 간격으로만 값을 갱신합니다.

```tsx
import { useState, useEffect, useRef } from 'react';

function useThrottle<T>(value: T, interval: number): T {
  const [throttledValue, setThrottledValue] = useState<T>(value);
  const lastUpdated = useRef<number>(Date.now());

  useEffect(() => {
    const now = Date.now();
    const timeSinceLastUpdate = now - lastUpdated.current;

    if (timeSinceLastUpdate >= interval) {
      lastUpdated.current = now;
      setThrottledValue(value);
    } else {
      const timer = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, interval - timeSinceLastUpdate);

      return () => clearTimeout(timer);
    }
  }, [value, interval]);

  return throttledValue;
}

export default useThrottle;
```

**사용 예시:**

```tsx
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const throttledPosition = useThrottle(position, 100);

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return (
    <div>
      <p>실제 위치: ({position.x}, {position.y})</p>
      <p>쓰로틀 위치: ({throttledPosition.x}, {throttledPosition.y})</p>
    </div>
  );
}
```

### useInterval - 인터벌

선언적으로 setInterval을 사용합니다.

```tsx
import { useEffect, useRef } from 'react';

function useInterval(callback: () => void, delay: number | null): void {
  const savedCallback = useRef<() => void>(callback);

  // 콜백이 변경되면 ref 업데이트
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // 인터벌 설정
  useEffect(() => {
    if (delay === null) {
      return;
    }

    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

export default useInterval;
```

**사용 예시:**

```tsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useInterval(
    () => setSeconds(s => s + 1),
    isRunning ? 1000 : null // null이면 인터벌 중지
  );

  return (
    <div>
      <p>경과 시간: {seconds}초</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? '정지' : '시작'}
      </button>
      <button onClick={() => setSeconds(0)}>리셋</button>
    </div>
  );
}
```

---

## DOM/이벤트 훅 패턴

### useClickOutside - 외부 클릭 감지

요소 외부 클릭을 감지하여 드롭다운, 모달 닫기 등에 활용합니다. 모달 구현에 대한 더 자세한 내용은 [React Portal 완벽 가이드](/posts/react-portal-complete-guide/)를 참고하세요.

```tsx
import { useEffect, useRef, RefObject } from 'react';

function useClickOutside<T extends HTMLElement>(
  handler: () => void,
  mouseEvent: 'mousedown' | 'mouseup' = 'mousedown'
): RefObject<T> {
  const ref = useRef<T>(null);

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (ref.current && !ref.current.contains(event.target as Node)) {
        handler();
      }
    };

    document.addEventListener(mouseEvent, handleClickOutside);
    return () => document.removeEventListener(mouseEvent, handleClickOutside);
  }, [handler, mouseEvent]);

  return ref;
}

export default useClickOutside;
```

**사용 예시:**

```tsx
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useClickOutside<HTMLDivElement>(() => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>
        메뉴 열기
      </button>
      {isOpen && (
        <ul>
          <li>옵션 1</li>
          <li>옵션 2</li>
          <li>옵션 3</li>
        </ul>
      )}
    </div>
  );
}
```

### useKeyPress - 키보드 이벤트

특정 키 입력을 감지합니다.

```tsx
import { useState, useEffect, useCallback } from 'react';

type KeyPressHandler = (event: KeyboardEvent) => void;

interface UseKeyPressOptions {
  targetKey: string;
  onKeyDown?: KeyPressHandler;
  onKeyUp?: KeyPressHandler;
  preventDefault?: boolean;
}

function useKeyPress(options: UseKeyPressOptions): boolean {
  const { targetKey, onKeyDown, onKeyUp, preventDefault = false } = options;
  const [keyPressed, setKeyPressed] = useState(false);

  const downHandler = useCallback((event: KeyboardEvent) => {
    if (event.key === targetKey) {
      if (preventDefault) event.preventDefault();
      setKeyPressed(true);
      onKeyDown?.(event);
    }
  }, [targetKey, onKeyDown, preventDefault]);

  const upHandler = useCallback((event: KeyboardEvent) => {
    if (event.key === targetKey) {
      if (preventDefault) event.preventDefault();
      setKeyPressed(false);
      onKeyUp?.(event);
    }
  }, [targetKey, onKeyUp, preventDefault]);

  useEffect(() => {
    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);

    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [downHandler, upHandler]);

  return keyPressed;
}

export default useKeyPress;
```

**사용 예시:**

```tsx
function Game() {
  const [position, setPosition] = useState({ x: 50, y: 50 });

  useKeyPress({
    targetKey: 'ArrowUp',
    onKeyDown: () => setPosition(p => ({ ...p, y: p.y - 10 })),
  });

  useKeyPress({
    targetKey: 'ArrowDown',
    onKeyDown: () => setPosition(p => ({ ...p, y: p.y + 10 })),
  });

  useKeyPress({
    targetKey: 'ArrowLeft',
    onKeyDown: () => setPosition(p => ({ ...p, x: p.x - 10 })),
  });

  useKeyPress({
    targetKey: 'ArrowRight',
    onKeyDown: () => setPosition(p => ({ ...p, x: p.x + 10 })),
  });

  const escPressed = useKeyPress({ targetKey: 'Escape' });

  return (
    <div>
      {escPressed && <p>ESC 키가 눌렸습니다!</p>}
      <div
        style={% raw %}{{
          position: 'absolute',
          left: position.x,
          top: position.y,
          width: 50,
          height: 50,
          backgroundColor: 'blue',
        }}{% endraw %}
      />
    </div>
  );
}
```

### useWindowSize - 윈도우 크기

브라우저 윈도우 크기 변화를 감지합니다.

```tsx
import { useState, useEffect } from 'react';

interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>(() => ({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0,
  }));

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

export default useWindowSize;
```

**사용 예시:**

```tsx
function ResponsiveComponent() {
  const { width, height } = useWindowSize();

  const getDeviceType = () => {
    if (width < 768) return 'mobile';
    if (width < 1024) return 'tablet';
    return 'desktop';
  };

  return (
    <div>
      <p>화면 크기: {width} x {height}</p>
      <p>디바이스 타입: {getDeviceType()}</p>
      {width < 768 ? (
        <MobileNavigation />
      ) : (
        <DesktopNavigation />
      )}
    </div>
  );
}
```

### useScrollPosition - 스크롤 위치

스크롤 위치를 추적합니다.

```tsx
import { useState, useEffect } from 'react';

interface ScrollPosition {
  x: number;
  y: number;
  direction: 'up' | 'down' | null;
  isAtTop: boolean;
  isAtBottom: boolean;
}

function useScrollPosition(): ScrollPosition {
  const [scrollPosition, setScrollPosition] = useState<ScrollPosition>({
    x: 0,
    y: 0,
    direction: null,
    isAtTop: true,
    isAtBottom: false,
  });

  useEffect(() => {
    let previousY = window.scrollY;

    const handleScroll = () => {
      const currentY = window.scrollY;
      const currentX = window.scrollX;
      const direction = currentY > previousY ? 'down' : 'up';

      const isAtTop = currentY === 0;
      const isAtBottom =
        window.innerHeight + currentY >= document.documentElement.scrollHeight;

      setScrollPosition({
        x: currentX,
        y: currentY,
        direction: currentY === previousY ? null : direction,
        isAtTop,
        isAtBottom,
      });

      previousY = currentY;
    };

    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return scrollPosition;
}

export default useScrollPosition;
```

**사용 예시:**

{% raw %}
```tsx
function Header() {
  const { y, direction, isAtTop } = useScrollPosition();
  const [isVisible, setIsVisible] = useState(true);

  useEffect(() => {
    if (direction === 'down' && y > 100) {
      setIsVisible(false);
    } else if (direction === 'up') {
      setIsVisible(true);
    }
  }, [direction, y]);

  return (
    <header
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        right: 0,
        transform: isVisible ? 'translateY(0)' : 'translateY(-100%)',
        transition: 'transform 0.3s ease',
        backgroundColor: isAtTop ? 'transparent' : 'white',
        boxShadow: isAtTop ? 'none' : '0 2px 4px rgba(0,0,0,0.1)',
      }}
    >
      <nav>내비게이션</nav>
    </header>
  );
}
```
{% endraw %}

---

## 비동기 훅 패턴

### useFetch - 데이터 패칭

API 호출을 위한 선언적 훅입니다. 더 복잡한 서버 상태 관리가 필요하다면 [React Query 완벽 가이드](/posts/react-query-guide/)를 참고하세요.

```tsx
import { useState, useEffect, useCallback } from 'react';

interface UseFetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

interface UseFetchReturn<T> extends UseFetchState<T> {
  refetch: () => void;
}

function useFetch<T>(url: string, options?: RequestInit): UseFetchReturn<T> {
  const [state, setState] = useState<UseFetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const response = await fetch(url, options);

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({
        data: null,
        loading: false,
        error: error instanceof Error ? error : new Error('Unknown error')
      });
    }
  }, [url, options]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { ...state, refetch: fetchData };
}

export default useFetch;
```

**사용 예시:**

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

function UserList() {
  const { data: users, loading, error, refetch } = useFetch<User[]>(
    'https://api.example.com/users'
  );

  if (loading) return <div>로딩 중...</div>;
  if (error) return <div>에러: {error.message}</div>;
  if (!users) return <div>데이터가 없습니다</div>;

  return (
    <div>
      <button onClick={refetch}>새로고침</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} ({user.email})
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### useAsync - 비동기 상태 관리

더 유연한 비동기 작업 관리를 위한 훅입니다.

```tsx
import { useState, useCallback } from 'react';

type AsyncStatus = 'idle' | 'pending' | 'success' | 'error';

interface UseAsyncState<T> {
  status: AsyncStatus;
  data: T | null;
  error: Error | null;
}

interface UseAsyncReturn<T, P extends unknown[]> extends UseAsyncState<T> {
  execute: (...params: P) => Promise<T | null>;
  reset: () => void;
  isIdle: boolean;
  isPending: boolean;
  isSuccess: boolean;
  isError: boolean;
}

function useAsync<T, P extends unknown[] = []>(
  asyncFunction: (...params: P) => Promise<T>
): UseAsyncReturn<T, P> {
  const [state, setState] = useState<UseAsyncState<T>>({
    status: 'idle',
    data: null,
    error: null,
  });

  const execute = useCallback(async (...params: P): Promise<T | null> => {
    setState({ status: 'pending', data: null, error: null });

    try {
      const data = await asyncFunction(...params);
      setState({ status: 'success', data, error: null });
      return data;
    } catch (error) {
      const err = error instanceof Error ? error : new Error('Unknown error');
      setState({ status: 'error', data: null, error: err });
      return null;
    }
  }, [asyncFunction]);

  const reset = useCallback(() => {
    setState({ status: 'idle', data: null, error: null });
  }, []);

  return {
    ...state,
    execute,
    reset,
    isIdle: state.status === 'idle',
    isPending: state.status === 'pending',
    isSuccess: state.status === 'success',
    isError: state.status === 'error',
  };
}

export default useAsync;
```

**사용 예시:**

```tsx
interface LoginResponse {
  token: string;
  user: { id: number; name: string };
}

async function loginAPI(email: string, password: string): Promise<LoginResponse> {
  const response = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });

  if (!response.ok) {
    throw new Error('로그인에 실패했습니다');
  }

  return response.json();
}

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const loginAsync = useAsync(loginAPI);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const result = await loginAsync.execute(email, password);

    if (result) {
      console.log('로그인 성공:', result.user.name);
      // 리다이렉트 등 처리
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="이메일"
        disabled={loginAsync.isPending}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="비밀번호"
        disabled={loginAsync.isPending}
      />
      <button type="submit" disabled={loginAsync.isPending}>
        {loginAsync.isPending ? '로그인 중...' : '로그인'}
      </button>
      {loginAsync.isError && (
        <p style={% raw %}{{ color: 'red' }}{% endraw %}>{loginAsync.error?.message}</p>
      )}
      {loginAsync.isSuccess && (
        <p style={% raw %}{{ color: 'green' }}{% endraw %}>환영합니다, {loginAsync.data?.user.name}님!</p>
      )}
    </form>
  );
}
```

---

## 고급 패턴

### 제네릭 활용

TypeScript 제네릭을 활용하면 타입 안전하고 재사용 가능한 훅을 만들 수 있습니다.

```tsx
import { useState, useCallback } from 'react';

// 제네릭 스택 훅
interface UseStackReturn<T> {
  items: T[];
  push: (item: T) => void;
  pop: () => T | undefined;
  peek: () => T | undefined;
  isEmpty: boolean;
  size: number;
  clear: () => void;
}

function useStack<T>(initialItems: T[] = []): UseStackReturn<T> {
  const [items, setItems] = useState<T[]>(initialItems);

  const push = useCallback((item: T) => {
    setItems(prev => [...prev, item]);
  }, []);

  const pop = useCallback((): T | undefined => {
    let poppedItem: T | undefined;
    setItems(prev => {
      if (prev.length === 0) return prev;
      poppedItem = prev[prev.length - 1];
      return prev.slice(0, -1);
    });
    return poppedItem;
  }, []);

  const peek = useCallback((): T | undefined => {
    return items[items.length - 1];
  }, [items]);

  const clear = useCallback(() => {
    setItems([]);
  }, []);

  return {
    items,
    push,
    pop,
    peek,
    isEmpty: items.length === 0,
    size: items.length,
    clear,
  };
}

// 제네릭 맵 훅
function useMap<K, V>(initialMap?: Map<K, V>) {
  const [map, setMap] = useState(new Map<K, V>(initialMap));

  const set = useCallback((key: K, value: V) => {
    setMap(prev => new Map(prev).set(key, value));
  }, []);

  const remove = useCallback((key: K) => {
    setMap(prev => {
      const next = new Map(prev);
      next.delete(key);
      return next;
    });
  }, []);

  const get = useCallback((key: K) => map.get(key), [map]);

  const has = useCallback((key: K) => map.has(key), [map]);

  const clear = useCallback(() => setMap(new Map()), []);

  return { map, set, remove, get, has, clear, size: map.size };
}
```

### 훅 조합 (Composition)

여러 훅을 조합하여 더 복잡한 기능을 구현합니다.

```tsx
// 기본 훅들을 조합한 useForm 훅
interface FormField<T> {
  value: T;
  error: string | null;
  touched: boolean;
}

interface UseFormOptions<T extends Record<string, unknown>> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit: (values: T) => void | Promise<void>;
}

function useForm<T extends Record<string, unknown>>(options: UseFormOptions<T>) {
  const { initialValues, validate, onSubmit } = options;

  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const submitAsync = useAsync(async () => {
    await onSubmit(values);
  });

  const handleChange = useCallback((
    name: keyof T,
    value: T[keyof T]
  ) => {
    setValues(prev => ({ ...prev, [name]: value }));

    if (validate) {
      const validationErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: validationErrors[name] }));
    }
  }, [values, validate]);

  const handleBlur = useCallback((name: keyof T) => {
    setTouched(prev => ({ ...prev, [name]: true }));
  }, []);

  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();

    // 모든 필드를 touched로 설정
    const allTouched = Object.keys(values).reduce(
      (acc, key) => ({ ...acc, [key]: true }),
      {} as Record<keyof T, boolean>
    );
    setTouched(allTouched);

    // 유효성 검사
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);

      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }

    await submitAsync.execute();
  }, [values, validate, submitAsync]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    submitAsync.reset();
  }, [initialValues, submitAsync]);

  const getFieldProps = useCallback((name: keyof T) => ({
    value: values[name],
    onChange: (e: React.ChangeEvent<HTMLInputElement>) =>
      handleChange(name, e.target.value as T[keyof T]),
    onBlur: () => handleBlur(name),
  }), [values, handleChange, handleBlur]);

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    getFieldProps,
    isSubmitting: submitAsync.isPending,
    isValid: Object.keys(errors).length === 0,
  };
}
```

**사용 예시:**

```tsx
interface SignupForm {
  username: string;
  email: string;
  password: string;
}

function SignupPage() {
  const form = useForm<SignupForm>({
    initialValues: {
      username: '',
      email: '',
      password: '',
    },
    validate: (values) => {
      const errors: Partial<Record<keyof SignupForm, string>> = {};

      if (values.username.length < 3) {
        errors.username = '사용자명은 3자 이상이어야 합니다';
      }
      if (!values.email.includes('@')) {
        errors.email = '유효한 이메일을 입력하세요';
      }
      if (values.password.length < 8) {
        errors.password = '비밀번호는 8자 이상이어야 합니다';
      }

      return errors;
    },
    onSubmit: async (values) => {
      await fetch('/api/signup', {
        method: 'POST',
        body: JSON.stringify(values),
      });
    },
  });

  return (
    <form onSubmit={form.handleSubmit}>
      <div>
        <input {...form.getFieldProps('username')} placeholder="사용자명" />
        {form.touched.username && form.errors.username && (
          <span>{form.errors.username}</span>
        )}
      </div>
      <div>
        <input {...form.getFieldProps('email')} placeholder="이메일" />
        {form.touched.email && form.errors.email && (
          <span>{form.errors.email}</span>
        )}
      </div>
      <div>
        <input
          {...form.getFieldProps('password')}
          type="password"
          placeholder="비밀번호"
        />
        {form.touched.password && form.errors.password && (
          <span>{form.errors.password}</span>
        )}
      </div>
      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? '가입 중...' : '가입하기'}
      </button>
    </form>
  );
}
```

### 조건부 훅 실행 주의사항

React 훅은 항상 동일한 순서로 호출되어야 합니다. 조건부로 훅을 호출하면 오류가 발생합니다.

```tsx
// 잘못된 예 - 조건부 훅 호출
function BadExample({ shouldFetch }: { shouldFetch: boolean }) {
  // 이 코드는 React 훅 규칙을 위반합니다!
  if (shouldFetch) {
    const { data } = useFetch('/api/data'); // 위반!
  }

  return <div>...</div>;
}

// 올바른 예 - 훅은 항상 호출하고, 조건은 내부에서 처리
function GoodExample({ shouldFetch }: { shouldFetch: boolean }) {
  const { data, loading } = useFetch(
    shouldFetch ? '/api/data' : null // null일 때 fetch 스킵하도록 훅 내부 구현
  );

  return <div>...</div>;
}

// 또는 enabled 옵션 패턴 사용
function useFetchWithEnabled<T>(url: string, enabled = true) {
  const [state, setState] = useState<{ data: T | null; loading: boolean }>({
    data: null,
    loading: enabled,
  });

  useEffect(() => {
    if (!enabled) {
      setState({ data: null, loading: false });
      return;
    }

    // fetch 로직...
  }, [url, enabled]);

  return state;
}
```

---

## 테스트 작성법

React Testing Library의 `renderHook`을 사용하여 커스텀 훅을 테스트합니다. 프론트엔드 테스트에 대한 더 자세한 내용은 [프론트엔드 테스트 완벽 가이드](/posts/frontend-testing-guide/)를 참고하세요.

### 테스트 환경 설정

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom vitest
```

### useToggle 테스트 예시

```tsx
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import useToggle from './useToggle';

describe('useToggle', () => {
  it('초기값이 false일 때 value가 false여야 한다', () => {
    const { result } = renderHook(() => useToggle(false));
    expect(result.current.value).toBe(false);
  });

  it('초기값이 true일 때 value가 true여야 한다', () => {
    const { result } = renderHook(() => useToggle(true));
    expect(result.current.value).toBe(true);
  });

  it('toggle 호출 시 값이 반전되어야 한다', () => {
    const { result } = renderHook(() => useToggle(false));

    act(() => {
      result.current.toggle();
    });

    expect(result.current.value).toBe(true);

    act(() => {
      result.current.toggle();
    });

    expect(result.current.value).toBe(false);
  });

  it('setTrue 호출 시 값이 true가 되어야 한다', () => {
    const { result } = renderHook(() => useToggle(false));

    act(() => {
      result.current.setTrue();
    });

    expect(result.current.value).toBe(true);
  });

  it('setFalse 호출 시 값이 false가 되어야 한다', () => {
    const { result } = renderHook(() => useToggle(true));

    act(() => {
      result.current.setFalse();
    });

    expect(result.current.value).toBe(false);
  });
});
```

### useAsync 테스트 예시

```tsx
import { renderHook, act, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import useAsync from './useAsync';

describe('useAsync', () => {
  it('초기 상태가 idle이어야 한다', () => {
    const asyncFn = vi.fn();
    const { result } = renderHook(() => useAsync(asyncFn));

    expect(result.current.isIdle).toBe(true);
    expect(result.current.status).toBe('idle');
  });

  it('execute 호출 시 pending 상태가 되어야 한다', async () => {
    const asyncFn = vi.fn(() => new Promise(resolve =>
      setTimeout(() => resolve('data'), 100)
    ));
    const { result } = renderHook(() => useAsync(asyncFn));

    act(() => {
      result.current.execute();
    });

    expect(result.current.isPending).toBe(true);
  });

  it('성공 시 success 상태와 데이터가 있어야 한다', async () => {
    const mockData = { id: 1, name: 'Test' };
    const asyncFn = vi.fn(() => Promise.resolve(mockData));
    const { result } = renderHook(() => useAsync(asyncFn));

    await act(async () => {
      await result.current.execute();
    });

    expect(result.current.isSuccess).toBe(true);
    expect(result.current.data).toEqual(mockData);
  });

  it('실패 시 error 상태와 에러가 있어야 한다', async () => {
    const error = new Error('Test error');
    const asyncFn = vi.fn(() => Promise.reject(error));
    const { result } = renderHook(() => useAsync(asyncFn));

    await act(async () => {
      await result.current.execute();
    });

    expect(result.current.isError).toBe(true);
    expect(result.current.error).toEqual(error);
  });

  it('reset 호출 시 idle 상태로 돌아가야 한다', async () => {
    const asyncFn = vi.fn(() => Promise.resolve('data'));
    const { result } = renderHook(() => useAsync(asyncFn));

    await act(async () => {
      await result.current.execute();
    });

    expect(result.current.isSuccess).toBe(true);

    act(() => {
      result.current.reset();
    });

    expect(result.current.isIdle).toBe(true);
    expect(result.current.data).toBeNull();
  });
});
```

---

## 베스트 프랙티스

### 1. 단일 책임 원칙

각 훅은 하나의 명확한 목적을 가져야 합니다.

```tsx
// 좋은 예 - 각 훅이 하나의 책임을 가짐
function useToggle() { /* 토글 로직만 */ }
function useLocalStorage() { /* 로컬 스토리지 로직만 */ }

// 나쁜 예 - 너무 많은 책임
function useEverything() {
  // 토글도 하고, API도 호출하고, 로컬 스토리지도 관리하고...
}
```

### 2. 일관된 반환값 구조

훅의 반환값은 일관된 패턴을 따라야 합니다.

```tsx
// 값 하나만 반환할 때
function useDebounce<T>(value: T, delay: number): T {
  // ...
  return debouncedValue;
}

// 배열 반환 (useState 스타일) - 값과 setter
function useToggle(initial: boolean): [boolean, () => void] {
  // ...
  return [value, toggle];
}

// 객체 반환 - 여러 값이나 메서드가 있을 때
function useAsync<T>() {
  // ...
  return { data, loading, error, execute, reset };
}
```

### 3. 의존성 최적화

불필요한 리렌더링을 방지하기 위해 의존성을 최적화합니다.

```tsx
function useOptimizedHook() {
  // useCallback으로 함수 메모이제이션
  const handleClick = useCallback(() => {
    // ...
  }, [/* 필요한 의존성만 */]);

  // useMemo로 값 메모이제이션
  const computedValue = useMemo(() => {
    return expensiveCalculation();
  }, [/* 필요한 의존성만 */]);

  return { handleClick, computedValue };
}
```

### 4. 에러 처리

에러 상황을 적절히 처리하고 사용자에게 피드백을 제공합니다.

```tsx
function useRobustAsync<T>(asyncFn: () => Promise<T>) {
  const [state, setState] = useState({
    data: null as T | null,
    error: null as Error | null,
    loading: false,
  });

  const execute = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));

    try {
      const data = await asyncFn();
      setState({ data, error: null, loading: false });
      return data;
    } catch (error) {
      const normalizedError = error instanceof Error
        ? error
        : new Error(String(error));
      setState({ data: null, error: normalizedError, loading: false });

      // 필요시 에러 리포팅
      console.error('Async operation failed:', normalizedError);

      return null;
    }
  }, [asyncFn]);

  return { ...state, execute };
}
```

### 5. TypeScript 타입 안전성

제네릭과 타입 가드를 활용하여 타입 안전성을 보장합니다.

```tsx
// 제네릭으로 유연한 타입 지원
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  // ...
}

// 반환 타입을 명시적으로 정의
interface UseCounterReturn {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

function useCounter(initial = 0): UseCounterReturn {
  // ...
}

// const assertion으로 튜플 타입 보장
function useTuple() {
  const state = useState(0);
  return [state[0], state[1]] as const;
}
```

### 6. 문서화

JSDoc 주석으로 훅의 사용법을 문서화합니다.

```tsx
/**
 * 불리언 값을 토글하는 커스텀 훅
 *
 * @param initialValue - 초기 불리언 값 (기본값: false)
 * @returns 현재 값과 토글 관련 함수들을 포함한 객체
 *
 * @example
 * ```tsx
 * function Modal() {
 *   const { value: isOpen, toggle, setFalse: close } = useToggle();
 *   return (
 *     <div>
 *       <button onClick={toggle}>Toggle</button>
 *       {isOpen && <ModalContent onClose={close} />}
 *     </div>
 *   );
 * }
 * ```
 */
function useToggle(initialValue = false) {
  // 구현...
}
```

---

## 마무리

이 글에서 다룬 커스텀 훅 패턴들을 정리하면 다음과 같습니다:

| 카테고리 | 훅 | 용도 |
|---------|-----|------|
| 상태 관리 | useToggle | 불리언 토글 |
| | useCounter | 숫자 증감 |
| | useInput | 입력 필드 관리 |
| | useArray | 배열 조작 |
| 사이드 이펙트 | useLocalStorage | 로컬 스토리지 연동 |
| | useDebounce | 값 디바운스 |
| | useThrottle | 값 쓰로틀 |
| | useInterval | 인터벌 관리 |
| DOM/이벤트 | useClickOutside | 외부 클릭 감지 |
| | useKeyPress | 키보드 이벤트 |
| | useWindowSize | 윈도우 크기 |
| | useScrollPosition | 스크롤 위치 |
| 비동기 | useFetch | 데이터 패칭 |
| | useAsync | 비동기 상태 관리 |

### 핵심 포인트

1. **재사용성**: 공통 로직을 훅으로 추출하여 중복을 제거합니다
2. **테스트 용이성**: 훅은 독립적으로 테스트할 수 있습니다
3. **타입 안전성**: TypeScript와 제네릭을 활용하여 타입 오류를 방지합니다
4. **조합 가능성**: 작은 훅들을 조합하여 복잡한 기능을 구현합니다
5. **규칙 준수**: React 훅 규칙을 반드시 지켜야 합니다

커스텀 훅은 React 개발에서 코드 품질과 생산성을 높이는 핵심 도구입니다. 이 글에서 소개한 패턴들을 기반으로 프로젝트에 맞는 커스텀 훅을 설계해보세요. React 19의 새로운 기능과 함께 활용하려면 [React 19 완벽 가이드](/posts/react-19-complete-guide/)도 참고하시기 바랍니다.

## 참고 자료

- [React 공식 문서 - Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [React 공식 문서 - Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [TypeScript Handbook - Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [Testing Library - renderHook](https://testing-library.com/docs/react-testing-library/api/#renderhook)
