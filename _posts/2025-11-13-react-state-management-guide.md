---
title: "React 상태 관리 완벽 가이드 - useState부터 Context API까지"
date: 2025-11-13 14:00:00 +0900
categories: [React, State Management]
tags: [react, state-management, hooks, context-api]
author: changsu
toc: true
comments: true
description: React 상태 관리의 모든 것을 다룹니다. useState의 함수형 업데이트부터 useReducer의 복잡한 상태 로직, Context API로 Props Drilling 해결까지. 실전 예제와 성능 최적화 기법을 통해 React 상태 관리를 완벽히 마스터하세요.
---

## 들어가며

React 애플리케이션을 개발하다 보면 가장 먼저 마주하는 것이 바로 **상태 관리**입니다. 버튼 클릭 횟수부터 복잡한 폼 데이터, 사용자 인증 정보까지 모든 동적인 데이터는 상태로 관리됩니다.

하지만 상태 관리는 단순해 보이면서도 실제로는 많은 개발자들이 어려움을 겪는 영역입니다. "언제 useState를 쓰고 언제 useReducer를 써야 하나요?", "Context API는 언제 사용해야 하나요?", "Props Drilling은 왜 나쁜가요?" 같은 질문들이 끊임없이 나옵니다.

이번 포스팅에서는 React 상태 관리의 기초부터 실전 활용법까지 모든 것을 다룹니다. 이 글을 읽고 나면 어떤 상황에서 어떤 상태 관리 방법을 사용해야 하는지 명확히 알 수 있을 것입니다.

## 상태(State)란?

### 상태의 정의

상태는 **컴포넌트가 기억해야 하는 정보**입니다. 사용자의 입력, 네트워크 응답, 현재 선택된 탭 등 시간이 지남에 따라 변하는 모든 데이터를 의미합니다.

```jsx
function Counter() {
  // count는 상태 변수
  const [count, setCount] = useState(0);

  // 버튼 클릭 시 상태 변경
  return (
    <button onClick={() => setCount(count + 1)}>
      클릭 횟수: {count}
    </button>
  );
}
```

### 상태 vs Props

많은 초보자들이 헷갈려 하는 개념입니다:

| 구분 | 상태(State) | Props |
|------|------------|-------|
| **소유** | 컴포넌트 자신이 소유 | 부모 컴포넌트가 전달 |
| **변경** | 컴포넌트 내부에서 변경 가능 | 읽기 전용 (변경 불가) |
| **목적** | 동적 데이터 관리 | 컴포넌트 간 데이터 전달 |
| **예시** | 폼 입력 값, 토글 상태 | 사용자 정보, 설정 옵션 |

### 왜 상태 관리가 필요한가?

상태 관리를 제대로 하지 않으면 다음과 같은 문제가 발생합니다:

```jsx
// ❌ 나쁜 예: 일반 변수 사용
function BadCounter() {
  let count = 0; // 변경해도 화면이 업데이트되지 않음

  return (
    <button onClick={() => {
      count++; // 값은 증가하지만 리렌더링 안 됨
      console.log(count); // 콘솔에는 증가한 값이 출력됨
    }}>
      클릭 횟수: {count} {/* 항상 0 */}
    </button>
  );
}

// ✅ 좋은 예: useState 사용
function GoodCounter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      클릭 횟수: {count} {/* 정상적으로 증가 */}
    </button>
  );
}
```

React는 상태가 변경될 때만 컴포넌트를 다시 렌더링합니다. 일반 변수를 사용하면 화면이 업데이트되지 않습니다.

## useState 완벽 가이드

### 기본 사용법

`useState`는 가장 기본적인 상태 관리 Hook입니다:

```jsx
import { useState } from 'react';

function Example() {
  // [현재 상태, 상태 변경 함수] = useState(초기값)
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  const [isVisible, setIsVisible] = useState(true);

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

### 함수형 업데이트 (Functional Update)

상태 업데이트가 이전 상태에 의존할 때는 **함수형 업데이트**를 사용해야 합니다:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  // ❌ 문제가 있는 코드
  const handleBadIncrement = () => {
    setCount(count + 1); // count는 현재 렌더링의 스냅샷
    setCount(count + 1); // 같은 값으로 두 번 호출
    setCount(count + 1); // 결과: +1만 증가
  };

  // ✅ 올바른 코드
  const handleGoodIncrement = () => {
    setCount(prev => prev + 1); // 이전 상태를 기반으로 계산
    setCount(prev => prev + 1); // 각각 정확히 +1씩
    setCount(prev => prev + 1); // 결과: +3 증가
  };

  return (
    <div>
      <p>카운트: {count}</p>
      <button onClick={handleBadIncrement}>잘못된 증가 (+1)</button>
      <button onClick={handleGoodIncrement}>올바른 증가 (+3)</button>
    </div>
  );
}
```

**왜 이런 차이가 날까요?**

React는 상태 업데이트를 **배치(batch) 처리**합니다. 이벤트 핸들러의 모든 코드가 실행된 후에 한 번에 리렌더링을 수행합니다. 따라서:

- `setCount(count + 1)`: 현재 `count` 값(예: 0)을 세 번 사용 → 0 + 1 = 1
- `setCount(prev => prev + 1)`: 이전 업데이트 결과를 사용 → 0 + 1 + 1 + 1 = 3

### 지연 초기화 (Lazy Initialization)

초기 상태 계산이 비용이 클 때 사용합니다:

```jsx
// ❌ 나쁜 예: 매 렌더링마다 함수 실행
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos()); // 불필요한 재계산
  // ...
}

// ✅ 좋은 예: 초기 렌더링에만 함수 실행
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos); // 함수 참조 전달
  // 또는
  const [todos, setTodos] = useState(() => {
    console.log('초기화 한 번만 실행');
    return JSON.parse(localStorage.getItem('todos')) || [];
  });
  // ...
}

function createInitialTodos() {
  // 비용이 큰 계산
  console.log('초기 Todo 생성 중...');
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: `Todo ${i}`,
      completed: false
    });
  }
  return initialTodos;
}
```

### 객체와 배열 상태 관리

**중요**: React에서는 상태를 **절대 직접 변경하면 안 됩니다**. 항상 새로운 객체나 배열을 생성해야 합니다.

```jsx
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  // ❌ 잘못된 방법: 직접 수정
  const handleBadChange = (field, value) => {
    user[field] = value; // 상태 직접 변경 - 리렌더링 안 됨!
    setUser(user); // 같은 참조라서 변경 감지 안 됨
  };

  // ✅ 올바른 방법: 새 객체 생성
  const handleGoodChange = (field, value) => {
    setUser(prev => ({
      ...prev, // 기존 속성 복사
      [field]: value // 특정 속성만 변경
    }));
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={e => handleGoodChange('name', e.target.value)}
        placeholder="이름"
      />
      <input
        value={user.email}
        onChange={e => handleGoodChange('email', e.target.value)}
        placeholder="이메일"
      />
      <p>입력된 정보: {JSON.stringify(user)}</p>
    </div>
  );
}
```

배열 상태 관리도 마찬가지입니다:

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  // ✅ 추가
  const addTodo = (text) => {
    setTodos(prev => [...prev, { id: Date.now(), text }]);
  };

  // ✅ 삭제
  const removeTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };

  // ✅ 수정
  const updateTodo = (id, newText) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, text: newText } : todo
    ));
  };

  // ❌ 잘못된 방법들
  const badAddTodo = (text) => {
    todos.push({ id: Date.now(), text }); // 직접 변경
    setTodos(todos); // 리렌더링 안 됨
  };

  return (
    <div>
      <button onClick={() => addTodo('새 할일')}>추가</button>
      {todos.map(todo => (
        <div key={todo.id}>
          {todo.text}
          <button onClick={() => removeTodo(todo.id)}>삭제</button>
        </div>
      ))}
    </div>
  );
}
```

### 실전 예제: 카운터 앱

모든 개념을 활용한 카운터 앱입니다:

```jsx
function AdvancedCounter() {
  const [count, setCount] = useState(() => {
    // 지연 초기화: localStorage에서 값 불러오기
    const saved = localStorage.getItem('count');
    return saved ? parseInt(saved, 10) : 0;
  });

  const [step, setStep] = useState(1);

  // localStorage에 자동 저장
  useEffect(() => {
    localStorage.setItem('count', count);
  }, [count]);

  // 함수형 업데이트로 정확한 증가/감소
  const increment = () => setCount(prev => prev + step);
  const decrement = () => setCount(prev => prev - step);
  const reset = () => setCount(0);

  return (
    <div>
      <h2>현재 카운트: {count}</h2>
      <div>
        <label>
          증가/감소 단위:
          <input
            type="number"
            value={step}
            onChange={e => setStep(Number(e.target.value))}
          />
        </label>
      </div>
      <button onClick={increment}>+ {step}</button>
      <button onClick={decrement}>- {step}</button>
      <button onClick={reset}>초기화</button>
    </div>
  );
}
```

## useReducer 완벽 가이드

### useReducer란?

`useReducer`는 **복잡한 상태 로직을 관리**할 때 사용하는 Hook입니다. 여러 필드가 함께 변경되거나, 상태 업데이트 로직이 복잡할 때 유용합니다.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### 기본 구조

```jsx
import { useReducer } from 'react';

// 1. 리듀서 함수 정의
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// 2. 컴포넌트에서 사용
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>카운트: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>초기화</button>
    </div>
  );
}
```

### Reducer 함수 작성 원칙

리듀서는 **순수 함수**여야 합니다:

```jsx
// ✅ 좋은 리듀서
function goodReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      // 새 객체 반환
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    default:
      return state;
  }
}

// ❌ 나쁜 리듀서
function badReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      // 상태 직접 변경 - 절대 금지!
      state.todos.push(action.payload);
      return state;
    case 'FETCH_DATA':
      // 부수 효과(API 호출) - 리듀서에서 금지!
      fetch('/api/data');
      return state;
    default:
      return state;
  }
}
```

**리듀서 함수 규칙**:
1. **순수 함수**: 같은 입력에 항상 같은 출력
2. **상태 직접 수정 금지**: 항상 새 객체 반환
3. **부수 효과 금지**: API 호출, 타이머 등 불가
4. **동기적 실행**: 비동기 작업 불가

### useState vs useReducer 비교

언제 무엇을 사용해야 할까요?

| 기준 | useState | useReducer |
|------|----------|------------|
| **상태 복잡도** | 단순한 값 (숫자, 문자열, 불리언) | 복잡한 객체, 여러 하위 필드 |
| **업데이트 로직** | 간단한 변경 | 복잡한 로직, 여러 케이스 |
| **관련 상태** | 독립적인 상태들 | 서로 연관된 상태들 |
| **로직 위치** | 컴포넌트 내부 | 외부 함수 (테스트 용이) |
| **코드량** | 적음 | 많음 (보일러플레이트) |
| **디버깅** | 단순 | 액션 로깅 가능 |

**간단한 기준**:
- 상태가 1-2개 필드만 있고 독립적 → `useState`
- 상태가 여러 필드이고 함께 변경됨 → `useReducer`
- 여러 이벤트 핸들러에서 복잡한 로직 → `useReducer`

### 실전 예제: Todo 앱

복잡한 상태 로직의 완벽한 예시입니다:

```jsx
// 액션 타입 상수 정의 (오타 방지)
const ACTIONS = {
  ADD_TODO: 'ADD_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
  DELETE_TODO: 'DELETE_TODO',
  EDIT_TODO: 'EDIT_TODO',
  SET_FILTER: 'SET_FILTER'
};

// 리듀서 함수
function todoReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false
          }
        ]
      };

    case ACTIONS.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };

    case ACTIONS.DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };

    case ACTIONS.EDIT_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        )
      };

    case ACTIONS.SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };

    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// 초기 상태
const initialState = {
  todos: [],
  filter: 'all' // 'all' | 'active' | 'completed'
};

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [inputValue, setInputValue] = useState('');

  // 필터링된 할일 목록
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      dispatch({ type: ACTIONS.ADD_TODO, payload: inputValue });
      setInputValue('');
    }
  };

  return (
    <div>
      <h1>Todo 앱</h1>

      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={e => setInputValue(e.target.value)}
          placeholder="할 일을 입력하세요"
        />
        <button type="submit">추가</button>
      </form>

      <div>
        <button onClick={() => dispatch({ type: ACTIONS.SET_FILTER, payload: 'all' })}>
          전체 ({state.todos.length})
        </button>
        <button onClick={() => dispatch({ type: ACTIONS.SET_FILTER, payload: 'active' })}>
          진행중 ({state.todos.filter(t => !t.completed).length})
        </button>
        <button onClick={() => dispatch({ type: ACTIONS.SET_FILTER, payload: 'completed' })}>
          완료 ({state.todos.filter(t => t.completed).length})
        </button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: ACTIONS.TOGGLE_TODO, payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: ACTIONS.DELETE_TODO, payload: todo.id })}>
              삭제
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**왜 useReducer를 사용했을까?**
- 상태가 `todos`와 `filter` 두 개로 연관되어 있음
- 5가지 다른 액션 타입이 있음
- 각 액션의 로직이 복잡함 (배열 변환)
- 액션을 로깅하거나 디버깅하기 쉬움

### 지연 초기화

useReducer도 지연 초기화를 지원합니다:

```jsx
function init(initialCount) {
  // localStorage에서 불러오기 등 비용이 큰 작업
  return {
    count: initialCount,
    history: []
  };
}

function MyComponent({ initialCount }) {
  // 세 번째 인자로 초기화 함수 전달
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
}
```

## Props Drilling 문제

### Props Drilling이란?

Props Drilling은 **중간 컴포넌트들을 거쳐 props를 여러 단계로 전달해야 하는 문제**입니다.

```jsx
// ❌ Props Drilling의 문제점
function App() {
  const [user, setUser] = useState({ name: 'Alice', theme: 'dark' });

  return <Layout user={user} />;
}

function Layout({ user }) {
  // Layout은 user를 사용하지 않지만 Header로 전달하기 위해 받음
  return (
    <div>
      <Header user={user} />
      <Main />
    </div>
  );
}

function Header({ user }) {
  // Header도 사용하지 않지만 UserMenu로 전달
  return (
    <header>
      <Logo />
      <Navigation user={user} />
    </header>
  );
}

function Navigation({ user }) {
  // Navigation도 사용하지 않지만 UserMenu로 전달
  return (
    <nav>
      <Menu />
      <UserMenu user={user} />
    </nav>
  );
}

function UserMenu({ user }) {
  // 드디어 실제로 user를 사용!
  return <div>환영합니다, {user.name}님!</div>;
}
```

**문제점**:
1. **중간 컴포넌트가 불필요한 props를 받음**: `Layout`, `Header`, `Navigation`은 `user`를 사용하지 않음
2. **리팩토링 어려움**: `user` 구조 변경 시 모든 중간 컴포넌트 수정 필요
3. **코드 가독성 저하**: 데이터 흐름 추적이 어려움
4. **타입 관리 복잡**: TypeScript 사용 시 모든 중간 컴포넌트에 타입 정의 필요

### Props Drilling 시각화

```
App (user 생성)
  │
  ├─ user props 전달
  │
  └─> Layout (user 미사용, 단순 전달)
        │
        ├─ user props 전달
        │
        └─> Header (user 미사용, 단순 전달)
              │
              ├─ user props 전달
              │
              └─> Navigation (user 미사용, 단순 전달)
                    │
                    ├─ user props 전달
                    │
                    └─> UserMenu (user 실제 사용!) ✅
```

3-4단계를 거쳐야 비로소 데이터를 사용할 수 있습니다.

## Context API 완벽 가이드

### Context API란?

Context API는 **컴포넌트 트리 전체에 데이터를 전달할 수 있는 React 내장 기능**입니다. Props Drilling을 해결하는 완벽한 솔루션입니다.

### 기본 사용법

Context API는 3단계로 사용합니다:

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Context 생성
const ThemeContext = createContext('light'); // 기본값

// 2. Provider로 값 제공
function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={theme}>
      <Layout />
      <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
        테마 변경
      </button>
    </ThemeContext.Provider>
  );
}

// 3. useContext로 값 읽기
function SomeDeepComponent() {
  const theme = useContext(ThemeContext);

  return (
    <div style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      현재 테마: {theme}
    </div>
  );
}
```

### Props Drilling 해결

이제 Context로 이전 예제를 개선해봅시다:

```jsx
// ✅ Context API로 해결
import { createContext, useContext, useState } from 'react';

const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState({ name: 'Alice', theme: 'dark' });

  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function Layout() {
  // user를 props로 받지 않음!
  return (
    <div>
      <Header />
      <Main />
    </div>
  );
}

function Header() {
  // user를 props로 받지 않음!
  return (
    <header>
      <Logo />
      <Navigation />
    </header>
  );
}

function Navigation() {
  // user를 props로 받지 않음!
  return (
    <nav>
      <Menu />
      <UserMenu />
    </nav>
  );
}

function UserMenu() {
  // 어디서든 직접 Context에서 가져옴!
  const user = useContext(UserContext);

  return <div>환영합니다, {user.name}님!</div>;
}
```

**Context 사용 후 데이터 흐름**:

```
App (UserContext.Provider)
  │
  └─> Layout
        └─> Header
              └─> Navigation
                    └─> UserMenu
                          │
                          └─ useContext(UserContext) ✅
```

중간 컴포넌트를 거치지 않고 직접 접근합니다!

### 실전 예제 1: 테마 전환

실무에서 가장 많이 사용하는 패턴입니다:

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

// Context 생성
const ThemeContext = createContext(null);

// Provider 컴포넌트 (재사용을 위해 분리)
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    // localStorage에서 저장된 테마 불러오기
    return localStorage.getItem('theme') || 'light';
  });

  // 테마 변경 시 localStorage에 저장
  useEffect(() => {
    localStorage.setItem('theme', theme);
    document.documentElement.setAttribute('data-theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    toggleTheme
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom Hook (사용 편의성)
export function useTheme() {
  const context = useContext(ThemeContext);

  if (context === null) {
    throw new Error('useTheme must be used within ThemeProvider');
  }

  return context;
}

// 사용 예시
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
      <Footer />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header className={theme}>
      <h1>내 웹사이트</h1>
      <button onClick={toggleTheme}>
        {theme === 'light' ? '🌙 다크 모드' : '☀️ 라이트 모드'}
      </button>
    </header>
  );
}

function Main() {
  const { theme } = useTheme();

  return (
    <main className={theme}>
      <p>현재 테마: {theme}</p>
    </main>
  );
}
```

**핵심 패턴**:
1. Provider 컴포넌트 분리로 재사용성 향상
2. Custom Hook으로 사용 편의성 향상
3. 에러 처리로 안전성 향상

### 실전 예제 2: 사용자 인증 상태

복잡한 상태와 함수를 함께 관리하는 예제입니다:

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // 초기 로드 시 인증 상태 확인
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const token = localStorage.getItem('token');
        if (token) {
          // API로 사용자 정보 가져오기
          const response = await fetch('/api/me', {
            headers: { Authorization: `Bearer ${token}` }
          });
          if (response.ok) {
            const userData = await response.json();
            setUser(userData);
          }
        }
      } catch (error) {
        console.error('Auth check failed:', error);
      } finally {
        setLoading(false);
      }
    };

    checkAuth();
  }, []);

  const login = async (email, password) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });

      if (response.ok) {
        const { user, token } = await response.json();
        localStorage.setItem('token', token);
        setUser(user);
        return { success: true };
      } else {
        return { success: false, error: '로그인 실패' };
      }
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  const value = {
    user,
    loading,
    login,
    logout,
    isAuthenticated: !!user
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);

  if (context === null) {
    throw new Error('useAuth must be used within AuthProvider');
  }

  return context;
}

// 사용 예시
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}

function LoginPage() {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    const result = await login(email, password);
    if (result.success) {
      // 대시보드로 이동
    } else {
      alert(result.error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">로그인</button>
    </form>
  );
}

function Dashboard() {
  const { user, logout } = useAuth();

  return (
    <div>
      <h1>환영합니다, {user.name}님!</h1>
      <button onClick={logout}>로그아웃</button>
    </div>
  );
}

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();

  if (loading) return <div>로딩 중...</div>;
  if (!isAuthenticated) return <Navigate to="/login" />;

  return children;
}
```

### 여러 Context 조합

관심사를 분리하여 여러 Context를 사용하면 유연성이 증가합니다:

```jsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          <Router>
            <Routes>
              {/* ... */}
            </Routes>
          </Router>
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// 더 깔끔하게 만들기
function AppProviders({ children }) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          {children}
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

function App() {
  return (
    <AppProviders>
      <Router>
        <Routes>
          {/* ... */}
        </Routes>
      </Router>
    </AppProviders>
  );
}
```

### Context API 주의사항

#### 1. 불필요한 리렌더링

Context 값이 변경되면 **해당 Context를 구독하는 모든 컴포넌트가 리렌더링**됩니다:

```jsx
// ❌ 문제가 있는 코드
function BadProvider({ children }) {
  const [user, setUser] = useState(null);

  // 매 렌더링마다 새 객체 생성 → 모든 구독 컴포넌트 리렌더링
  const value = {
    user,
    setUser
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// ✅ 해결책: useMemo 사용
function GoodProvider({ children }) {
  const [user, setUser] = useState(null);

  // user가 변경될 때만 새 객체 생성
  const value = useMemo(() => ({
    user,
    setUser
  }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

#### 2. Context 분리로 최적화

하나의 Context에 모든 것을 넣으면 불필요한 리렌더링이 발생합니다:

```jsx
// ❌ 나쁜 예: 모든 것을 하나의 Context에
const AppContext = createContext(null);

function BadProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('ko');

  const value = useMemo(() => ({
    user, setUser,
    theme, setTheme,
    language, setLanguage
  }), [user, theme, language]);

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// theme만 필요한 컴포넌트도 user나 language 변경 시 리렌더링됨!

// ✅ 좋은 예: Context 분리
const UserContext = createContext(null);
const ThemeContext = createContext(null);
const LanguageContext = createContext(null);

function GoodProviders({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <LanguageProvider>
          {children}
        </LanguageProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// 이제 theme만 필요한 컴포넌트는 theme 변경 시만 리렌더링됨!
```

#### 3. Provider 위치

Provider는 useContext 호출보다 위에 있어야 합니다:

```jsx
// ❌ 작동하지 않음
function App() {
  const theme = useContext(ThemeContext); // Provider가 없음!

  return (
    <ThemeContext.Provider value="dark">
      <Component />
    </ThemeContext.Provider>
  );
}

// ✅ 올바른 사용
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Component />
    </ThemeContext.Provider>
  );
}

function Component() {
  const theme = useContext(ThemeContext); // Provider 안에 있음
  return <div>{theme}</div>;
}
```

## 언제 어떤 방법을 사용할까?

### 의사결정 플로우차트

```
상태가 필요한가?
  │
  ├─ Yes → 상태가 컴포넌트 내부에서만 사용되는가?
  │          │
  │          ├─ Yes → 상태가 단순한가? (1-2개 필드)
  │          │          │
  │          │          ├─ Yes → useState 사용 ✅
  │          │          │
  │          │          └─ No → 상태 업데이트 로직이 복잡한가?
  │          │                   │
  │          │                   ├─ Yes → useReducer 사용 ✅
  │          │                   │
  │          │                   └─ No → useState 사용 (복잡해지면 리팩토링)
  │          │
  │          └─ No → 여러 컴포넌트가 공유하는가?
  │                   │
  │                   ├─ 2-3개 레벨 → Props로 전달 ✅
  │                   │
  │                   └─ 3개 이상 또는 많은 컴포넌트 → Context API 사용 ✅
  │
  └─ No → 계산으로 유도 가능 (파생 상태) ✅
```

### 실전 가이드라인

#### useState를 사용하세요

```jsx
// ✅ useState가 적합한 경우
function SimpleForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  return (
    <form>
      <input value={name} onChange={e => setName(e.target.value)} />
      <input value={email} onChange={e => setEmail(e.target.value)} />
    </form>
  );
}
```

**사용 시기**:
- 단순한 값 (문자열, 숫자, 불리언)
- 독립적인 상태 변수들
- 간단한 업데이트 로직
- 로컬 UI 상태 (열림/닫힘, 선택 여부 등)

#### useReducer를 사용하세요

```jsx
// ✅ useReducer가 적합한 경우
function ComplexForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  // 여러 필드가 함께 변경되고, 로직이 복잡함
  return (
    <form onSubmit={() => dispatch({ type: 'SUBMIT' })}>
      {/* ... */}
    </form>
  );
}
```

**사용 시기**:
- 여러 필드가 있는 복잡한 객체
- 상태 업데이트가 여러 단계
- 이전 상태에 기반한 복잡한 계산
- 로직을 테스트하고 싶을 때

#### Context API를 사용하세요

```jsx
// ✅ Context가 적합한 경우
<ThemeProvider>
  <App />
</ThemeProvider>

function DeepComponent() {
  const { theme } = useTheme();
  // ...
}
```

**사용 시기**:
- 전역 설정 (테마, 언어, 인증)
- 많은 컴포넌트가 공유하는 데이터
- Props Drilling이 3단계 이상
- 자주 변경되지 않는 데이터

#### 조합해서 사용하세요

가장 강력한 패턴은 **useReducer + Context**입니다:

```jsx
// ✅ useReducer + Context 조합
const TodoContext = createContext(null);

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  const value = useMemo(() => ({ state, dispatch }), [state]);

  return (
    <TodoContext.Provider value={value}>
      {children}
    </TodoContext.Provider>
  );
}

export function useTodos() {
  const context = useContext(TodoContext);

  if (context === null) {
    throw new Error('useTodos must be used within TodoProvider');
  }

  return context;
}

// 어디서든 사용 가능
function TodoList() {
  const { state, dispatch } = useTodos();

  return (
    <ul>
      {state.todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
            삭제
          </button>
        </li>
      ))}
    </ul>
  );
}
```

**이 조합의 장점**:
- 복잡한 상태 로직 (useReducer)
- 전역 접근성 (Context)
- 테스트 용이성 (reducer는 순수 함수)
- 디버깅 편의 (액션 로깅 가능)

### 빠른 참고표

| 상황 | 추천 방법 |
|------|----------|
| 컴포넌트 내부 토글 상태 | `useState` |
| 폼 입력 1-3개 필드 | `useState` |
| 폼 입력 4개 이상 필드 | `useReducer` |
| 장바구니, Todo 리스트 | `useReducer` |
| 테마, 언어 설정 | `Context + useState` |
| 사용자 인증 상태 | `Context + useState` |
| 복잡한 앱 전역 상태 | `Context + useReducer` |
| 서버 상태 (API 데이터) | React Query, SWR 등 |

## 성능 최적화

상태 관리를 올바르게 해도 성능 문제가 발생할 수 있습니다. 주요 최적화 기법을 알아봅시다.

### React.memo로 불필요한 리렌더링 방지

```jsx
// ❌ 최적화 안 된 컴포넌트
function TodoItem({ todo, onToggle, onDelete }) {
  console.log('TodoItem 렌더링:', todo.id);
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </li>
  );
}

// ✅ React.memo로 최적화
const TodoItem = React.memo(function TodoItem({ todo, onToggle, onDelete }) {
  console.log('TodoItem 렌더링:', todo.id);
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </li>
  );
});

// 이제 todo, onToggle, onDelete가 변경되지 않으면 리렌더링 안 됨!
```

### useCallback으로 함수 메모이제이션

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  // ❌ 매 렌더링마다 새 함수 생성 → TodoItem 리렌더링
  const handleToggle = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  // ✅ useCallback으로 함수 메모이제이션
  const handleToggleOptimized = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []); // 의존성 없음 (함수형 업데이트)

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggleOptimized}
        />
      ))}
    </ul>
  );
}
```

### useMemo로 계산 비용 절감

```jsx
function TodoList({ todos, filter }) {
  // ❌ 매 렌더링마다 필터링 실행
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  // ✅ useMemo로 최적화
  const filteredTodosOptimized = useMemo(() => {
    console.log('필터링 실행');
    return todos.filter(todo => {
      if (filter === 'active') return !todo.completed;
      if (filter === 'completed') return todo.completed;
      return true;
    });
  }, [todos, filter]); // todos나 filter 변경 시만 재계산

  return (
    <ul>
      {filteredTodosOptimized.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}
```

### Context 최적화 전략

```jsx
// ❌ 비효율적: 모든 것을 하나의 값으로
function BadTodoProvider({ children }) {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  // 매 렌더링마다 새 객체 생성
  const value = {
    todos,
    setTodos,
    filter,
    setFilter
  };

  return <TodoContext.Provider value={value}>{children}</TodoContext.Provider>;
}

// ✅ 효율적: useMemo + 관심사 분리
const TodoStateContext = createContext(null);
const TodoDispatchContext = createContext(null);

function GoodTodoProvider({ children }) {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  // 상태만 포함 (읽기 전용)
  const state = useMemo(() => ({ todos, filter }), [todos, filter]);

  // dispatch 함수만 포함 (거의 변경 안 됨)
  const dispatch = useMemo(() => ({ setTodos, setFilter }), []);

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        {children}
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}

// 상태만 필요한 컴포넌트
function TodoCount() {
  const { todos } = useContext(TodoStateContext);
  return <div>전체: {todos.length}</div>;
}

// dispatch만 필요한 컴포넌트
function AddTodoButton() {
  const { setTodos } = useContext(TodoDispatchContext);
  // todos 변경 시에도 리렌더링 안 됨!
  return <button onClick={() => setTodos(prev => [...prev, newTodo])}>추가</button>;
}
```

### 최적화 체크리스트

| 최적화 기법 | 사용 시기 | 우선순위 |
|------------|----------|----------|
| **함수형 업데이트** | 상태가 이전 상태에 의존 | 높음 |
| **React.memo** | 자식 컴포넌트가 자주 리렌더링 | 중간 |
| **useCallback** | 함수를 자식에게 전달 | 중간 |
| **useMemo** | 복잡한 계산, 큰 배열 필터링 | 중간 |
| **Context 분리** | 여러 상태가 하나의 Context에 | 높음 |
| **State/Dispatch 분리** | Context 최적화 | 높음 |

> **주의**: 모든 곳에 최적화를 적용하지 마세요! 성능 문제가 있을 때만 최적화하세요. 과도한 최적화는 코드를 복잡하게 만들고 오히려 성능을 저하시킬 수 있습니다.
{: .prompt-warning }

## Best Practices

### 1. 상태 구조화 원칙

**중복되거나 파생 가능한 상태를 만들지 마세요**:

```jsx
// ❌ 나쁜 예: 중복 상태
function BadUserForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState(''); // 중복!

  const handleFirstNameChange = (value) => {
    setFirstName(value);
    setFullName(value + ' ' + lastName); // 수동 동기화
  };
  // ...
}

// ✅ 좋은 예: 파생 상태
function GoodUserForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // 렌더링 중 계산 (파생 상태)
  const fullName = `${firstName} ${lastName}`;

  return <div>Full Name: {fullName}</div>;
}
```

**연관된 상태는 함께 관리하세요**:

```jsx
// ❌ 나쁜 예: 독립적인 상태들
function BadPosition() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);

  const moveRight = () => {
    setX(x + 1);
    setY(y); // 불필요
  };
}

// ✅ 좋은 예: 객체로 함께 관리
function GoodPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const moveRight = () => {
    setPosition(prev => ({ ...prev, x: prev.x + 1 }));
  };
}
```

### 2. 네이밍 컨벤션

일관된 네이밍으로 가독성을 높이세요:

```jsx
// ✅ 상태: 명사
const [user, setUser] = useState(null);
const [isLoading, setIsLoading] = useState(false);
const [hasError, setHasError] = useState(false);

// ✅ 이벤트 핸들러: handle + 동사
const handleClick = () => {};
const handleSubmit = () => {};
const handleChange = () => {};

// ✅ 액션 타입: 대문자 + 동사
const ACTIONS = {
  ADD_TODO: 'ADD_TODO',
  DELETE_TODO: 'DELETE_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO'
};

// ✅ Custom Hook: use + 명사/동사
function useAuth() {}
function useLocalStorage() {}
function useFetch() {}
```

### 3. 초기 상태 설정

의미 있는 초기값을 설정하세요:

```jsx
// ❌ 나쁜 예
const [user, setUser] = useState(); // undefined
const [todos, setTodos] = useState(); // undefined

// ✅ 좋은 예
const [user, setUser] = useState(null); // 명시적 null
const [todos, setTodos] = useState([]); // 빈 배열
const [isLoading, setIsLoading] = useState(true); // 초기 로딩 상태
```

### 4. 상태 업데이트 안전하게 하기

항상 불변성을 유지하세요:

```jsx
// ❌ 나쁜 예: 직접 변경
state.todos.push(newTodo);
setState(state);

// ✅ 좋은 예: 새 배열/객체 생성
setState(prev => ({
  ...prev,
  todos: [...prev.todos, newTodo]
}));
```

### 5. 에러 처리

Context 사용 시 Provider 존재 여부를 확인하세요:

```jsx
export function useAuth() {
  const context = useContext(AuthContext);

  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }

  return context;
}
```

### 6. 타입 안정성 (TypeScript)

TypeScript를 사용한다면 타입을 명시하세요:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

const [user, setUser] = useState<User | null>(null);

type Action =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'DELETE_TODO'; payload: number }
  | { type: 'TOGGLE_TODO'; payload: number };

function todoReducer(state: TodoState, action: Action): TodoState {
  // ...
}
```

## 자주 하는 실수와 해결법

### 1. 상태 업데이트 후 즉시 값 사용

```jsx
// ❌ 문제: 상태는 다음 렌더링에 업데이트됨
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
    console.log(count); // 여전히 이전 값!
  };
}

// ✅ 해결: 변수에 저장하거나 useEffect 사용
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    const newCount = count + 1;
    setCount(newCount);
    console.log(newCount); // 새 값 사용
  };
}
```

### 2. 렌더링 중 상태 업데이트

```jsx
// ❌ 무한 루프 발생
function BadComponent() {
  const [count, setCount] = useState(0);

  setCount(count + 1); // 렌더링 중 호출 → 다시 렌더링 → 무한 반복

  return <div>{count}</div>;
}

// ✅ 이벤트 핸들러에서 호출
function GoodComponent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

### 3. 함수를 상태로 저장

```jsx
// ❌ 함수가 즉시 실행됨
const [fn, setFn] = useState(myFunction); // myFunction()이 호출됨

// ✅ 함수를 반환하는 함수로 래핑
const [fn, setFn] = useState(() => myFunction);
```

### 4. 의존성 배열 누락

```jsx
// ❌ searchTerm 변경 시 업데이트 안 됨
const filtered = useMemo(() => {
  return items.filter(item => item.includes(searchTerm));
}, [items]); // searchTerm 누락!

// ✅ 모든 의존성 포함
const filtered = useMemo(() => {
  return items.filter(item => item.includes(searchTerm));
}, [items, searchTerm]);
```

### 5. Context 값으로 객체 직접 전달

```jsx
// ❌ 매 렌더링마다 새 객체 → 모든 구독자 리렌더링
function BadProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// ✅ useMemo로 메모이제이션
function GoodProvider({ children }) {
  const [user, setUser] = useState(null);

  const value = useMemo(() => ({ user, setUser }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

## FAQ

### Q1: useState와 useReducer 중 무엇을 선택해야 하나요?

**A**: 대부분의 경우 useState로 시작하세요. 다음 상황에서 useReducer로 전환하세요:
- 상태가 5개 이상의 필드를 가진 복잡한 객체일 때
- 여러 이벤트 핸들러가 같은 상태를 다른 방식으로 업데이트할 때
- 상태 업데이트 로직을 컴포넌트 밖으로 분리하고 싶을 때
- 상태 업데이트를 로깅하거나 디버깅하고 싶을 때

### Q2: Context를 사용하면 성능 문제가 있나요?

**A**: Context 자체는 문제가 아닙니다. 문제는 **불필요한 리렌더링**입니다:
- Context 값이 변경되면 모든 구독 컴포넌트가 리렌더링됩니다
- 객체나 함수를 값으로 전달할 때 useMemo/useCallback을 사용하세요
- 자주 변경되는 상태와 그렇지 않은 상태를 분리하세요
- State와 Dispatch Context를 분리하는 것도 좋은 전략입니다

### Q3: Props Drilling은 항상 나쁜가요?

**A**: 아닙니다. 2-3단계 정도의 Props 전달은 괜찮습니다:
- 명시적이고 추적하기 쉽습니다
- Context보다 간단합니다
- TypeScript와 잘 작동합니다

하지만 3단계 이상이거나, 많은 컴포넌트가 같은 데이터를 필요로 한다면 Context를 고려하세요.

### Q4: 언제 상태를 리프팅(lifting)해야 하나요?

**A**: 두 컴포넌트가 같은 상태를 공유해야 할 때입니다:
```jsx
// 두 자식이 count를 공유해야 함
function Parent() {
  const [count, setCount] = useState(0); // 상태를 부모로 리프팅

  return (
    <>
      <ChildA count={count} setCount={setCount} />
      <ChildB count={count} />
    </>
  );
}
```

### Q5: 로컬 스토리지와 상태를 어떻게 동기화하나요?

**A**: useEffect와 지연 초기화를 조합하세요:
```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

### Q6: 여러 상태를 동시에 업데이트하려면?

**A**: React는 상태 업데이트를 자동으로 배치 처리합니다 (React 18+):
```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  setText('updated');
  // 한 번만 리렌더링됨!
}
```

React 17 이하에서는 이벤트 핸들러 내부에서만 배치 처리됩니다.

### Q7: Context 없이 전역 상태를 관리할 수 있나요?

**A**: 다음 방법들이 있습니다:
- **Zustand, Jotai**: 간단한 전역 상태 라이브러리
- **Redux Toolkit**: 복잡한 앱을 위한 강력한 상태 관리
- **React Query, SWR**: 서버 상태 관리 전용

다음 포스팅에서 이들을 자세히 다룰 예정입니다!

### Q8: 상태가 너무 많아서 관리하기 어려워요

**A**: 상태를 카테고리별로 분류하세요:
- **UI 상태**: 모달 열림/닫힘, 선택된 탭 등 → 로컬 useState
- **폼 상태**: 입력 값, 유효성 검사 → useReducer 또는 React Hook Form
- **서버 상태**: API 데이터 → React Query, SWR
- **전역 설정**: 테마, 언어 → Context
- **복잡한 전역 상태**: 장바구니, 사용자 세션 → Zustand, Redux

## 결론

React 상태 관리는 처음에는 복잡해 보이지만, 각 도구의 용도를 이해하면 어렵지 않습니다.

**핵심 요약**:

1. **useState**: 단순한 로컬 상태 관리의 기본
   - 함수형 업데이트로 정확한 상태 변경
   - 지연 초기화로 성능 최적화
   - 객체/배열은 항상 새로 생성

2. **useReducer**: 복잡한 상태 로직의 해결사
   - 여러 필드가 연관된 상태
   - 복잡한 업데이트 로직
   - 테스트 가능한 순수 함수

3. **Context API**: Props Drilling의 완벽한 대안
   - 전역 설정과 테마 관리
   - 사용자 인증 상태
   - useMemo로 성능 최적화 필수

4. **조합의 힘**: useReducer + Context
   - 복잡한 앱의 상태 관리
   - 확장 가능한 아키텍처
   - 디버깅과 테스트 용이

**실전 팁**:
- 항상 간단한 것부터 시작하세요 (useState)
- 복잡해지면 리팩토링하세요 (useReducer)
- 3단계 이상 Props 전달이 필요하면 Context를 고려하세요
- 성능 문제가 발생하면 측정 후 최적화하세요

### 다음 단계

이제 React 기본 상태 관리를 마스터했으니, 다음 포스팅에서는 **더 강력한 상태 관리 라이브러리**를 다룰 예정입니다:

- **Zustand vs Redux Toolkit**: 언제 무엇을 선택해야 하나?
- **React Query & SWR**: 서버 상태 관리의 혁명
- **Recoil & Jotai**: 원자적 상태 관리의 새로운 패러다임

상태 관리 시리즈의 다음 글도 기대해주세요! 궁금한 점이나 추가로 다뤘으면 하는 주제가 있다면 댓글로 남겨주세요.

## 참고 자료

- [React 공식 문서 - Managing State](https://react.dev/learn/managing-state)
- [React 공식 문서 - useState](https://react.dev/reference/react/useState)
- [React 공식 문서 - useReducer](https://react.dev/reference/react/useReducer)
- [React 공식 문서 - useContext](https://react.dev/reference/react/useContext)
- [React 공식 문서 - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)
