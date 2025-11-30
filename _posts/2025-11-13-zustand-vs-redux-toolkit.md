---
title: "Zustand vs Redux Toolkit - 2025년 상태 관리 라이브러리 완벽 비교"
date: 2025-11-13 16:00:00 +0900
categories: [React, State Management]
tags: [react, state-management, zustand, redux-toolkit]
author: changsu
toc: true
comments: true
description: Zustand와 Redux Toolkit을 실전 예제로 비교합니다. 번들 사이즈, 러닝 커브, 성능을 분석하고 프로젝트 규모별 최적의 상태 관리 라이브러리 선택 가이드를 제공합니다.
---

## 들어가며

[이전 글](/posts/react-state-management-guide/)에서 React의 기본 상태 관리 도구인 useState, useReducer, Context API를 살펴봤습니다. 하지만 실무 프로젝트가 복잡해지면 이런 질문들이 생깁니다:

- "Context API만으로는 부족한데, 더 나은 방법이 없을까?"
- "Redux는 너무 복잡하다는데, 꼭 써야 하나?"
- "Zustand가 핫하다는데, Redux와 뭐가 다른가?"
- "프로젝트 규모에 따라 어떤 도구를 선택해야 할까?"

2025년 현재, 전역 상태 관리 생태계는 매우 다양합니다. Redux가 여전히 강력하지만, Zustand, Jotai, Recoil 같은 새로운 라이브러리들이 등장하며 선택의 폭이 넓어졌습니다.

이번 포스팅에서는 실전에서 가장 많이 쓰이는 **Zustand**와 **Redux Toolkit**을 중심으로 전역 상태 관리 라이브러리들을 철저히 비교하고, 여러분의 프로젝트에 맞는 최적의 선택을 돕겠습니다.

## 전역 상태 관리가 필요한 시점

Context API도 전역 상태 관리가 가능한데, 왜 별도의 라이브러리가 필요할까요?

### Context API의 한계

```jsx
// Context API의 성능 문제
const AppContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [cart, setCart] = useState([]);
  const [theme, setTheme] = useState('light');

  const value = useMemo(() => ({
    user, setUser,
    cart, setCart,
    theme, setTheme
  }), [user, cart, theme]);

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// 문제: theme만 필요한 컴포넌트도 user나 cart 변경 시 리렌더링됨
function ThemeButton() {
  const { theme, setTheme } = useContext(AppContext);
  console.log('ThemeButton 렌더링'); // user 변경 시에도 출력됨!

  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
    {theme} 모드
  </button>;
}
```

**Context API의 문제점**:
1. **불필요한 리렌더링**: Context 값이 변경되면 모든 구독자가 리렌더링
2. **성능 최적화 어려움**: 부분 구독이 불가능
3. **보일러플레이트**: Provider 중첩, useMemo 필수
4. **디버깅 어려움**: Redux DevTools 같은 도구 없음
5. **미들웨어 부재**: 로깅, 영속화 등 추가 기능 구현 어려움

### 전역 상태 관리 라이브러리가 필요한 경우

다음 상황이라면 전역 상태 관리 라이브러리를 고려하세요:

```jsx
// ✅ 1. 많은 컴포넌트가 같은 상태를 공유
// 장바구니를 Header, ProductList, Checkout에서 모두 접근

// ✅ 2. 복잡한 상태 로직과 비동기 처리
// 사용자 인증, 권한 체크, API 데이터 캐싱

// ✅ 3. 디버깅 도구 필요
// 상태 변화 추적, 시간 여행 디버깅

// ✅ 4. 성능 최적화 필수
// 대규모 애플리케이션, 부분 구독 필요

// ✅ 5. 팀 협업
// 일관된 상태 관리 패턴, 타입 안정성
```

**간단한 기준**:
- 전역 상태가 3개 이하, 단순 구조 → Context API
- 전역 상태가 5개 이상, 복잡한 로직 → 상태 관리 라이브러리
- 서버 데이터 위주 → React Query / SWR
- 복잡한 클라이언트 상태 → Zustand / Redux Toolkit

## Redux의 역사와 Redux Toolkit

### Redux의 등장 (2015)

Redux는 Flux 아키텍처에서 영감을 받아 2015년에 Dan Abramov가 만들었습니다. 당시 React 생태계에는 체계적인 상태 관리 솔루션이 없었고, Redux는 다음을 제공했습니다:

```jsx
// 전통적인 Redux (Legacy)
// 1. 액션 타입 정의
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';

// 2. 액션 생성자
function addTodo(text) {
  return { type: ADD_TODO, payload: text };
}

// 3. 리듀서
function todoReducer(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [...state, { id: Date.now(), text: action.payload, completed: false }];
    case TOGGLE_TODO:
      return state.map(todo =>
        todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
      );
    default:
      return state;
  }
}

// 4. 스토어 생성
const store = createStore(todoReducer);

// 5. 컴포넌트에서 사용
function TodoList() {
  const todos = useSelector(state => state);
  const dispatch = useDispatch();

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id} onClick={() => dispatch({ type: TOGGLE_TODO, payload: todo.id })}>
          {todo.text}
        </li>
      ))}
      <button onClick={() => dispatch(addTodo('New Todo'))}>추가</button>
    </ul>
  );
}
```

**Redux의 장점**:
- 예측 가능한 상태 관리
- 강력한 DevTools
- 시간 여행 디버깅
- 미들웨어 생태계

**Redux의 문제점**:
```jsx
// ❌ 엄청난 보일러플레이트
// 하나의 기능을 위해 5개 파일 필요:
// - actionTypes.js
// - actionCreators.js
// - reducer.js
// - selectors.js
// - container.js

// ❌ 복잡한 설정
const store = createStore(
  rootReducer,
  initialState,
  compose(
    applyMiddleware(thunk, logger),
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
  )
);

// ❌ 불변성 유지의 어려움
case UPDATE_USER:
  return {
    ...state,
    users: state.users.map(user =>
      user.id === action.payload.id
        ? { ...user, ...action.payload.updates }
        : user
    )
  };
```

### Redux Toolkit의 등장 (2019)

Redux 팀이 공식적으로 만든 Redux Toolkit(RTK)은 Redux의 복잡성을 대폭 줄였습니다:

```jsx
// ✅ Redux Toolkit - 같은 기능을 훨씬 간결하게
import { createSlice, configureStore } from '@reduxjs/toolkit';

// 1. Slice 생성 (액션 + 리듀서 통합)
const todoSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      // Immer 덕분에 직접 수정 가능!
      state.push({ id: Date.now(), text: action.payload, completed: false });
    },
    toggleTodo: (state, action) => {
      const todo = state.find(t => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    }
  }
});

// 2. 액션과 리듀서 자동 생성
export const { addTodo, toggleTodo } = todoSlice.actions;

// 3. 스토어 생성 (DevTools 자동 설정)
const store = configureStore({
  reducer: {
    todos: todoSlice.reducer
  }
});

// 4. 컴포넌트에서 사용 (동일)
function TodoList() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id} onClick={() => dispatch(toggleTodo(todo.id))}>
          {todo.text}
        </li>
      ))}
      <button onClick={() => dispatch(addTodo('New Todo'))}>추가</button>
    </ul>
  );
}
```

**Redux Toolkit의 개선사항**:
- 80% 적은 코드
- Immer로 불변성 자동 처리
- DevTools 자동 설정
- 타입스크립트 완벽 지원
- RTK Query로 API 통합

## Zustand 소개

Zustand(독일어로 '상태')는 2019년 Poimandres(three.js 생태계)에서 만든 초경량 상태 관리 라이브러리입니다.

### Zustand의 철학

"React 상태 관리는 간단해야 한다."

```jsx
// ✅ Zustand - 믿을 수 없을 만큼 간단
import { create } from 'zustand';

// 1. 스토어 생성 (끝!)
const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text, completed: false }]
  })),
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  }))
}));

// 2. 컴포넌트에서 사용
function TodoList() {
  const todos = useTodoStore(state => state.todos);
  const addTodo = useTodoStore(state => state.addTodo);
  const toggleTodo = useTodoStore(state => state.toggleTodo);

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id} onClick={() => toggleTodo(todo.id)}>
          {todo.text}
        </li>
      ))}
      <button onClick={() => addTodo('New Todo')}>추가</button>
    </ul>
  );
}
```

**Zustand의 특징**:

1. **Provider 불필요**: 바로 import해서 사용
2. **극도로 작은 번들**: 압축 시 1KB
3. **부분 구독**: 필요한 부분만 선택적으로 구독
4. **React 외부 사용 가능**: Vanilla JS에서도 사용
5. **미들웨어 지원**: persist, devtools, immer 등

```jsx
// Provider 없이 어디서든 접근 가능
import { useTodoStore } from './store';

// React 외부에서도 사용
const currentTodos = useTodoStore.getState().todos;
useTodoStore.setState({ todos: [] });

// 구독 (React 없이)
const unsubscribe = useTodoStore.subscribe(
  state => console.log('Todos changed:', state.todos)
);
```

### 왜 Zustand가 인기인가?

**npm 다운로드 추이** (2025년 기준):
- Redux: 약 8,000,000/주
- Redux Toolkit: 약 5,000,000/주
- Zustand: 약 3,000,000/주 (빠르게 성장 중)
- Jotai: 약 800,000/주
- Recoil: 약 400,000/주

```jsx
// 개발자들이 Zustand를 선택하는 이유

// 1. 러닝 커브가 거의 없음
const useStore = create((set) => ({
  count: 0,
  inc: () => set(state => ({ count: state.count + 1 }))
}));
// 이게 전부입니다. 5분이면 배웁니다.

// 2. Redux보다 훨씬 적은 코드
// Redux Toolkit: 30줄
// Zustand: 10줄

// 3. 뛰어난 성능
function Counter() {
  // count가 변경될 때만 리렌더링
  const count = useStore(state => state.count);
  return <div>{count}</div>;
}

function IncrementButton() {
  // inc는 변경되지 않으므로 리렌더링 안 됨
  const inc = useStore(state => state.inc);
  return <button onClick={inc}>+1</button>;
}
```

## Recoil과 Jotai 간단 소개

### Recoil (Facebook, 2020)

Recoil은 Facebook(현 Meta)에서 만든 원자적 상태 관리 라이브러리입니다.

```jsx
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil';

// Atom: 상태의 단위
const todoListState = atom({
  key: 'todoList',
  default: []
});

// Selector: 파생 상태
const todoStatsState = selector({
  key: 'todoStats',
  get: ({ get }) => {
    const todos = get(todoListState);
    return {
      total: todos.length,
      completed: todos.filter(t => t.completed).length
    };
  }
});

function TodoList() {
  const [todos, setTodos] = useRecoilState(todoListState);
  const stats = useRecoilValue(todoStatsState);

  return (
    <div>
      <p>완료: {stats.completed} / {stats.total}</p>
      {/* ... */}
    </div>
  );
}
```

**Recoil의 장단점**:
- ✅ React처럼 사용 (Hooks 스타일)
- ✅ 세밀한 구독과 최적화
- ✅ 비동기 지원
- ❌ 아직 실험 단계 (Experimental)
- ❌ Facebook 내부에서도 제한적 사용
- ❌ 번들 크기가 큼 (14KB)

### Jotai (Poimandres, 2020)

Jotai는 Zustand를 만든 팀의 원자적 상태 관리 라이브러리입니다.

```jsx
import { atom, useAtom } from 'jotai';

// Primitive atom
const countAtom = atom(0);

// Derived atom
const doubleCountAtom = atom(get => get(countAtom) * 2);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubleCount] = useAtom(doubleCountAtom);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Double: {doubleCount}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

**Jotai의 장단점**:
- ✅ 매우 작은 번들 (3KB)
- ✅ TypeScript 완벽 지원
- ✅ Recoil보다 간단
- ✅ 안정적인 API
- ❌ 생태계가 작음
- ❌ 러닝 커브 있음 (원자적 개념)

### MobX

MobX는 Observable 패턴 기반의 상태 관리 라이브러리입니다.

```jsx
import { makeObservable, observable, action } from 'mobx';
import { observer } from 'mobx-react-lite';

class TodoStore {
  todos = [];

  constructor() {
    makeObservable(this, {
      todos: observable,
      addTodo: action
    });
  }

  addTodo = (text) => {
    this.todos.push({ id: Date.now(), text, completed: false });
  };
}

const TodoList = observer(({ store }) => {
  return (
    <ul>
      {store.todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
});
```

**MobX 특징**:
- ✅ 직관적 (일반 객체처럼 사용)
- ✅ 자동 최적화
- ❌ 클래스 기반 (React Hooks와 이질적)
- ❌ 마법 같은 동작 (디버깅 어려움)

## 주요 라이브러리 상세 비교

### 비교표

| 항목 | Zustand | Redux Toolkit | Recoil | Jotai |
|------|---------|---------------|--------|-------|
| **번들 크기** | 1.2 KB | 11 KB | 14 KB | 3 KB |
| **러닝 커브** | ⭐ 매우 쉬움 | ⭐⭐⭐ 중간 | ⭐⭐⭐ 중간 | ⭐⭐ 쉬움 |
| **보일러플레이트** | 최소 | 중간 (Slice 필요) | 중간 (Atom 정의) | 중간 |
| **DevTools** | ✅ (미들웨어) | ✅ 내장 | ✅ | ✅ |
| **TypeScript** | ✅ 우수 | ✅ 완벽 | ✅ 우수 | ✅ 완벽 |
| **미들웨어** | ✅ 다양 | ✅ 풍부 | ❌ 제한적 | ⚠️ 있음 |
| **비동기 처리** | ✅ 자유로움 | ✅ createAsyncThunk | ✅ 내장 | ✅ 내장 |
| **Provider 필요** | ❌ | ✅ | ✅ | ✅ |
| **성능** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **생태계** | ⭐⭐⭐ 중간 | ⭐⭐⭐⭐⭐ 최고 | ⭐⭐ 작음 | ⭐⭐ 작음 |
| **커뮤니티** | 성장 중 | 매우 큼 | 중간 | 중간 |
| **안정성** | ✅ 안정 | ✅ 매우 안정 | ⚠️ 실험적 | ✅ 안정 |
| **React 외부 사용** | ✅ 가능 | ⚠️ 제한적 | ❌ 불가 | ❌ 불가 |

### 번들 크기 실측

```bash
# 실제 프로덕션 번들 크기 (minified + gzipped)
zustand: 1.2 KB
jotai: 3.0 KB
redux-toolkit: 11.0 KB
  - @reduxjs/toolkit: 8.5 KB
  - react-redux: 2.5 KB
recoil: 14.0 KB

# 참고: Context API는 React에 포함되어 있으므로 0 KB
```

작은 프로젝트나 모바일 환경에서는 Zustand의 1KB가 큰 장점입니다.

### 러닝 커브

```jsx
// 1. Zustand - 5분이면 배움
const useStore = create((set) => ({
  count: 0,
  inc: () => set(state => ({ count: state.count + 1 }))
}));

// 2. Jotai - 15분
const countAtom = atom(0);
const useCount = () => useAtom(countAtom);

// 3. Redux Toolkit - 1시간
const slice = createSlice({
  name: 'counter',
  initialState: { count: 0 },
  reducers: {
    increment: state => { state.count += 1; }
  }
});
const store = configureStore({ reducer: { counter: slice.reducer } });

// 4. Recoil - 30분
const countState = atom({ key: 'count', default: 0 });
```

### 개발자 경험 (DX)

```jsx
// Zustand: 가장 간결
const bears = useBearStore(state => state.bears);
const increasePopulation = useBearStore(state => state.increasePopulation);

// Redux Toolkit: 타입 안정성 최고
const count = useSelector((state: RootState) => state.counter.count);
const dispatch = useDispatch<AppDispatch>();

// Jotai: React Hooks처럼 느껴짐
const [count, setCount] = useAtom(countAtom);

// Recoil: React Hooks + 추가 개념
const [todos, setTodos] = useRecoilState(todoListState);
```

## Zustand 실습

### 설치 및 기본 설정

```bash
npm install zustand
# 또는
yarn add zustand
# 또는
pnpm add zustand
```

### 기본 사용법

```jsx
// store/todoStore.js
import { create } from 'zustand';

export const useTodoStore = create((set) => ({
  // 상태
  todos: [],
  filter: 'all',

  // 액션
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, {
      id: Date.now(),
      text,
      completed: false
    }]
  })),

  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),

  deleteTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),

  setFilter: (filter) => set({ filter })
}));
```

### 컴포넌트에서 사용

{% raw %}
```jsx
// components/TodoList.jsx
import { useTodoStore } from '../store/todoStore';

function TodoList() {
  // 필요한 부분만 선택적으로 구독 (성능 최적화)
  const todos = useTodoStore(state => state.todos);
  const filter = useTodoStore(state => state.filter);
  const toggleTodo = useTodoStore(state => state.toggleTodo);
  const deleteTodo = useTodoStore(state => state.deleteTodo);

  // 필터링된 할일 목록
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <ul>
      {filteredTodos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.text}
          </span>
          <button onClick={() => deleteTodo(todo.id)}>삭제</button>
        </li>
      ))}
    </ul>
  );
}

function AddTodo() {
  const addTodo = useTodoStore(state => state.addTodo);
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text);
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="할 일을 입력하세요"
      />
      <button type="submit">추가</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useTodoStore(state => state.filter);
  const setFilter = useTodoStore(state => state.setFilter);

  return (
    <div>
      <button
        disabled={filter === 'all'}
        onClick={() => setFilter('all')}
      >
        전체
      </button>
      <button
        disabled={filter === 'active'}
        onClick={() => setFilter('active')}
      >
        진행중
      </button>
      <button
        disabled={filter === 'completed'}
        onClick={() => setFilter('completed')}
      >
        완료
      </button>
    </div>
  );
}
```
{% endraw %}

### 고급 패턴

#### 1. 비동기 액션

```jsx
const useUserStore = create((set, get) => ({
  user: null,
  loading: false,
  error: null,

  // 비동기 액션
  fetchUser: async (userId) => {
    set({ loading: true, error: null });

    try {
      const response = await fetch(`/api/users/${userId}`);
      const user = await response.json();
      set({ user, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },

  // 다른 상태 참조 (get 사용)
  updateUserName: (name) => {
    const currentUser = get().user;
    if (currentUser) {
      set({ user: { ...currentUser, name } });
    }
  }
}));
```

#### 2. Immer 미들웨어로 불변성 간편화

```jsx
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

const useStore = create(
  immer((set) => ({
    todos: [],

    // Immer 덕분에 직접 수정 가능!
    addTodo: (text) => set((state) => {
      state.todos.push({
        id: Date.now(),
        text,
        completed: false
      });
    }),

    toggleTodo: (id) => set((state) => {
      const todo = state.todos.find(t => t.id === id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    })
  }))
);
```

#### 3. Persist 미들웨어로 LocalStorage 자동 저장

```jsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useStore = create(
  persist(
    (set) => ({
      todos: [],
      addTodo: (text) => set((state) => ({
        todos: [...state.todos, { id: Date.now(), text, completed: false }]
      }))
    }),
    {
      name: 'todo-storage', // localStorage 키
      storage: createJSONStorage(() => localStorage)
    }
  )
);

// 이제 새로고침해도 데이터가 유지됩니다!
```

#### 4. DevTools 미들웨어

```jsx
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), false, 'increment')
    }),
    { name: 'CounterStore' }
  )
);

// Redux DevTools에서 상태 변화를 추적할 수 있습니다!
```

#### 5. 미들웨어 조합

```jsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        todos: [],
        addTodo: (text) => set((state) => {
          state.todos.push({ id: Date.now(), text, completed: false });
        })
      })),
      { name: 'todo-storage' }
    ),
    { name: 'TodoStore' }
  )
);
```

#### 6. Slice 패턴 (대규모 앱)

```jsx
// store/slices/userSlice.js
export const createUserSlice = (set, get) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null })
});

// store/slices/cartSlice.js
export const createCartSlice = (set, get) => ({
  items: [],
  addItem: (item) => set((state) => ({
    items: [...state.items, item]
  })),
  total: () => get().items.reduce((sum, item) => sum + item.price, 0)
});

// store/index.js
import { create } from 'zustand';
import { createUserSlice } from './slices/userSlice';
import { createCartSlice } from './slices/cartSlice';

export const useStore = create((...args) => ({
  ...createUserSlice(...args),
  ...createCartSlice(...args)
}));

// 사용
const user = useStore(state => state.user);
const addItem = useStore(state => state.addItem);
```

## Redux Toolkit 실습

### 설치 및 기본 설정

```bash
npm install @reduxjs/toolkit react-redux
```

### 기본 사용법

```jsx
// store/todoSlice.js
import { createSlice } from '@reduxjs/toolkit';

const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    filter: 'all'
  },
  reducers: {
    addTodo: (state, action) => {
      // Immer 덕분에 직접 수정 가능!
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false
      });
    },

    toggleTodo: (state, action) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },

    deleteTodo: (state, action) => {
      state.items = state.items.filter(t => t.id !== action.payload);
    },

    setFilter: (state, action) => {
      state.filter = action.payload;
    }
  }
});

export const { addTodo, toggleTodo, deleteTodo, setFilter } = todoSlice.actions;
export default todoSlice.reducer;

// Selector 정의
export const selectTodos = state => state.todos.items;
export const selectFilter = state => state.todos.filter;
export const selectFilteredTodos = state => {
  const { items, filter } = state.todos;
  if (filter === 'active') return items.filter(t => !t.completed);
  if (filter === 'completed') return items.filter(t => t.completed);
  return items;
};
```

### 스토어 설정

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import todoReducer from './todoSlice';

export const store = configureStore({
  reducer: {
    todos: todoReducer
  },
  // DevTools 자동 설정됨
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Provider 설정

```jsx
// index.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import { store } from './store';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

### 컴포넌트에서 사용

{% raw %}
```jsx
// components/TodoList.jsx
import { useSelector, useDispatch } from 'react-redux';
import { toggleTodo, deleteTodo, selectFilteredTodos } from '../store/todoSlice';

function TodoList() {
  const todos = useSelector(selectFilteredTodos);
  const dispatch = useDispatch();

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => dispatch(toggleTodo(todo.id))}
          />
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.text}
          </span>
          <button onClick={() => dispatch(deleteTodo(todo.id))}>삭제</button>
        </li>
      ))}
    </ul>
  );
}

function AddTodo() {
  const dispatch = useDispatch();
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch(addTodo(text));
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="할 일을 입력하세요"
      />
      <button type="submit">추가</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useSelector(selectFilter);
  const dispatch = useDispatch();

  return (
    <div>
      <button
        disabled={filter === 'all'}
        onClick={() => dispatch(setFilter('all'))}
      >
        전체
      </button>
      <button
        disabled={filter === 'active'}
        onClick={() => dispatch(setFilter('active'))}
      >
        진행중
      </button>
      <button
        disabled={filter === 'completed'}
        onClick={() => dispatch(setFilter('completed'))}
      >
        완료
      </button>
    </div>
  );
}
```
{% endraw %}

### createAsyncThunk로 비동기 처리

```jsx
// store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 비동기 Thunk 생성
export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error('Failed to fetch');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const updateUser = createAsyncThunk(
  'user/updateUser',
  async ({ userId, updates }) => {
    const response = await fetch(`/api/users/${userId}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates)
    });
    return await response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  reducers: {
    logout: (state) => {
      state.data = null;
    }
  },
  extraReducers: (builder) => {
    builder
      // fetchUser
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      // updateUser
      .addCase(updateUser.fulfilled, (state, action) => {
        state.data = action.payload;
      });
  }
});

export const { logout } = userSlice.actions;
export default userSlice.reducer;

// 사용
function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const { data: user, loading, error } = useSelector(state => state.user);

  useEffect(() => {
    dispatch(fetchUser(userId));
  }, [dispatch, userId]);

  if (loading) return <div>로딩 중...</div>;
  if (error) return <div>오류: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.name}</h2>
      <button onClick={() => dispatch(updateUser({ userId, updates: { name: 'New Name' } }))}>
        이름 변경
      </button>
      <button onClick={() => dispatch(logout())}>로그아웃</button>
    </div>
  );
}
```

### RTK Query로 API 통합 (보너스)

```jsx
// store/api.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Todo'],
  endpoints: (builder) => ({
    getTodos: builder.query({
      query: () => 'todos',
      providesTags: ['Todo']
    }),
    addTodo: builder.mutation({
      query: (todo) => ({
        url: 'todos',
        method: 'POST',
        body: todo
      }),
      invalidatesTags: ['Todo']
    }),
    updateTodo: builder.mutation({
      query: ({ id, ...updates }) => ({
        url: `todos/${id}`,
        method: 'PATCH',
        body: updates
      }),
      invalidatesTags: ['Todo']
    })
  })
});

export const { useGetTodosQuery, useAddTodoMutation, useUpdateTodoMutation } = api;

// 사용
function TodoList() {
  const { data: todos, isLoading, error } = useGetTodosQuery();
  const [addTodo] = useAddTodoMutation();
  const [updateTodo] = useUpdateTodoMutation();

  if (isLoading) return <div>로딩 중...</div>;
  if (error) return <div>오류 발생</div>;

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => updateTodo({ id: todo.id, completed: true })}>
            완료
          </button>
        </li>
      ))}
      <button onClick={() => addTodo({ text: 'New Todo' })}>추가</button>
    </ul>
  );
}
```

## 같은 Todo 앱을 두 도구로 구현

### Zustand 버전 (완전한 예제)

```jsx
// store/todoStore.js
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export const useTodoStore = create(
  devtools(
    persist(
      immer((set, get) => ({
        // 상태
        todos: [],
        filter: 'all',

        // 액션
        addTodo: (text) => set((state) => {
          state.todos.push({
            id: Date.now(),
            text,
            completed: false,
            createdAt: new Date().toISOString()
          });
        }),

        toggleTodo: (id) => set((state) => {
          const todo = state.todos.find(t => t.id === id);
          if (todo) todo.completed = !todo.completed;
        }),

        deleteTodo: (id) => set((state) => {
          state.todos = state.todos.filter(t => t.id !== id);
        }),

        editTodo: (id, text) => set((state) => {
          const todo = state.todos.find(t => t.id === id);
          if (todo) todo.text = text;
        }),

        setFilter: (filter) => set({ filter }),

        clearCompleted: () => set((state) => {
          state.todos = state.todos.filter(t => !t.completed);
        }),

        // 셀렉터 (computed)
        getFilteredTodos: () => {
          const { todos, filter } = get();
          if (filter === 'active') return todos.filter(t => !t.completed);
          if (filter === 'completed') return todos.filter(t => t.completed);
          return todos;
        },

        getStats: () => {
          const todos = get().todos;
          return {
            total: todos.length,
            active: todos.filter(t => !t.completed).length,
            completed: todos.filter(t => t.completed).length
          };
        }
      })),
      { name: 'todo-storage' }
    ),
    { name: 'TodoStore' }
  )
);

// components/TodoApp.jsx
import { useTodoStore } from '../store/todoStore';
import { useState } from 'react';

function TodoApp() {
  return (
    <div className="todo-app">
      <h1>Todo App (Zustand)</h1>
      <AddTodoForm />
      <FilterButtons />
      <TodoStats />
      <TodoList />
      <ClearCompleted />
    </div>
  );
}

function AddTodoForm() {
  const addTodo = useTodoStore(state => state.addTodo);
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text);
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="할 일을 입력하세요"
      />
      <button type="submit">추가</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useTodoStore(state => state.filter);
  const setFilter = useTodoStore(state => state.setFilter);

  return (
    <div className="filters">
      {['all', 'active', 'completed'].map(f => (
        <button
          key={f}
          className={filter === f ? 'active' : ''}
          onClick={() => setFilter(f)}
        >
          {f === 'all' ? '전체' : f === 'active' ? '진행중' : '완료'}
        </button>
      ))}
    </div>
  );
}

function TodoStats() {
  const getStats = useTodoStore(state => state.getStats);
  const stats = getStats();

  return (
    <div className="stats">
      전체: {stats.total} | 진행중: {stats.active} | 완료: {stats.completed}
    </div>
  );
}

function TodoList() {
  const getFilteredTodos = useTodoStore(state => state.getFilteredTodos);
  const todos = getFilteredTodos();

  return (
    <ul className="todo-list">
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

function TodoItem({ todo }) {
  const toggleTodo = useTodoStore(state => state.toggleTodo);
  const deleteTodo = useTodoStore(state => state.deleteTodo);
  const editTodo = useTodoStore(state => state.editTodo);
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleEdit = () => {
    if (editText.trim()) {
      editTodo(todo.id, editText);
      setIsEditing(false);
    }
  };

  return (
    <li className={todo.completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleTodo(todo.id)}
      />

      {isEditing ? (
        <input
          type="text"
          value={editText}
          onChange={e => setEditText(e.target.value)}
          onBlur={handleEdit}
          onKeyPress={e => e.key === 'Enter' && handleEdit()}
          autoFocus
        />
      ) : (
        <span onDoubleClick={() => setIsEditing(true)}>
          {todo.text}
        </span>
      )}

      <button onClick={() => deleteTodo(todo.id)}>삭제</button>
    </li>
  );
}

function ClearCompleted() {
  const clearCompleted = useTodoStore(state => state.clearCompleted);
  const getStats = useTodoStore(state => state.getStats);
  const stats = getStats();

  if (stats.completed === 0) return null;

  return (
    <button onClick={clearCompleted}>
      완료된 항목 삭제 ({stats.completed})
    </button>
  );
}

export default TodoApp;
```

**Zustand 코드 특징**:
- 총 라인 수: ~150줄
- Provider 불필요
- 직관적인 API
- 미들웨어로 기능 확장

### Redux Toolkit 버전 (완전한 예제)

```jsx
// store/todoSlice.js
import { createSlice, createSelector } from '@reduxjs/toolkit';

const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    filter: 'all'
  },
  reducers: {
    addTodo: (state, action) => {
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
        createdAt: new Date().toISOString()
      });
    },

    toggleTodo: (state, action) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },

    deleteTodo: (state, action) => {
      state.items = state.items.filter(t => t.id !== action.payload);
    },

    editTodo: (state, action) => {
      const todo = state.items.find(t => t.id === action.payload.id);
      if (todo) todo.text = action.payload.text;
    },

    setFilter: (state, action) => {
      state.filter = action.payload;
    },

    clearCompleted: (state) => {
      state.items = state.items.filter(t => !t.completed);
    }
  }
});

export const {
  addTodo,
  toggleTodo,
  deleteTodo,
  editTodo,
  setFilter,
  clearCompleted
} = todoSlice.actions;

// Selectors (with memoization)
export const selectTodos = state => state.todos.items;
export const selectFilter = state => state.todos.filter;

export const selectFilteredTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    if (filter === 'active') return todos.filter(t => !t.completed);
    if (filter === 'completed') return todos.filter(t => t.completed);
    return todos;
  }
);

export const selectStats = createSelector(
  [selectTodos],
  (todos) => ({
    total: todos.length,
    active: todos.filter(t => !t.completed).length,
    completed: todos.filter(t => t.completed).length
  })
);

export default todoSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import todoReducer from './todoSlice';

export const store = configureStore({
  reducer: {
    todos: todoReducer
  }
});

// index.jsx
import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <TodoApp />
  </Provider>
);

// components/TodoApp.jsx
import { useSelector, useDispatch } from 'react-redux';
import {
  addTodo,
  toggleTodo,
  deleteTodo,
  editTodo,
  setFilter,
  clearCompleted,
  selectFilteredTodos,
  selectFilter,
  selectStats
} from '../store/todoSlice';
import { useState } from 'react';

function TodoApp() {
  return (
    <div className="todo-app">
      <h1>Todo App (Redux Toolkit)</h1>
      <AddTodoForm />
      <FilterButtons />
      <TodoStats />
      <TodoList />
      <ClearCompleted />
    </div>
  );
}

function AddTodoForm() {
  const dispatch = useDispatch();
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch(addTodo(text));
      setText('');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="할 일을 입력하세요"
      />
      <button type="submit">추가</button>
    </form>
  );
}

function FilterButtons() {
  const dispatch = useDispatch();
  const filter = useSelector(selectFilter);

  return (
    <div className="filters">
      {['all', 'active', 'completed'].map(f => (
        <button
          key={f}
          className={filter === f ? 'active' : ''}
          onClick={() => dispatch(setFilter(f))}
        >
          {f === 'all' ? '전체' : f === 'active' ? '진행중' : '완료'}
        </button>
      ))}
    </div>
  );
}

function TodoStats() {
  const stats = useSelector(selectStats);

  return (
    <div className="stats">
      전체: {stats.total} | 진행중: {stats.active} | 완료: {stats.completed}
    </div>
  );
}

function TodoList() {
  const todos = useSelector(selectFilteredTodos);

  return (
    <ul className="todo-list">
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

function TodoItem({ todo }) {
  const dispatch = useDispatch();
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleEdit = () => {
    if (editText.trim()) {
      dispatch(editTodo({ id: todo.id, text: editText }));
      setIsEditing(false);
    }
  };

  return (
    <li className={todo.completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => dispatch(toggleTodo(todo.id))}
      />

      {isEditing ? (
        <input
          type="text"
          value={editText}
          onChange={e => setEditText(e.target.value)}
          onBlur={handleEdit}
          onKeyPress={e => e.key === 'Enter' && handleEdit()}
          autoFocus
        />
      ) : (
        <span onDoubleClick={() => setIsEditing(true)}>
          {todo.text}
        </span>
      )}

      <button onClick={() => dispatch(deleteTodo(todo.id))}>삭제</button>
    </li>
  );
}

function ClearCompleted() {
  const dispatch = useDispatch();
  const stats = useSelector(selectStats);

  if (stats.completed === 0) return null;

  return (
    <button onClick={() => dispatch(clearCompleted())}>
      완료된 항목 삭제 ({stats.completed})
    </button>
  );
}

export default TodoApp;
```

**Redux Toolkit 코드 특징**:
- 총 라인 수: ~180줄
- Provider 필수
- createSelector로 메모이제이션
- 타입 안정성 높음

### 코드 비교 요약

| 측면 | Zustand | Redux Toolkit |
|------|---------|---------------|
| **라인 수** | ~150줄 | ~180줄 |
| **파일 수** | 2개 (store, component) | 3개 (slice, store, component) |
| **설정 복잡도** | 매우 간단 | 중간 (Provider 필요) |
| **API 복잡도** | 간단 | 중간 |
| **타입 안정성** | 좋음 | 우수 |
| **가독성** | 매우 높음 | 높음 |

## 복잡한 예제: 인증 + 장바구니

실무에서 흔히 마주치는 복잡한 시나리오를 구현해봅시다.

### Zustand 버전

```jsx
// store/authStore.js
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

export const useAuthStore = create(
  devtools(
    persist(
      (set, get) => ({
        user: null,
        token: null,
        loading: false,
        error: null,

        login: async (email, password) => {
          set({ loading: true, error: null });

          try {
            const response = await fetch('/api/auth/login', {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ email, password })
            });

            if (!response.ok) throw new Error('Login failed');

            const { user, token } = await response.json();
            set({ user, token, loading: false });
            return { success: true };
          } catch (error) {
            set({ error: error.message, loading: false });
            return { success: false, error: error.message };
          }
        },

        logout: () => {
          set({ user: null, token: null });
          // 장바구니도 초기화
          useCartStore.getState().clearCart();
        },

        updateProfile: async (updates) => {
          const { user, token } = get();

          try {
            const response = await fetch(`/api/users/${user.id}`, {
              method: 'PATCH',
              headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
              },
              body: JSON.stringify(updates)
            });

            const updatedUser = await response.json();
            set({ user: updatedUser });
            return { success: true };
          } catch (error) {
            return { success: false, error: error.message };
          }
        }
      }),
      {
        name: 'auth-storage',
        partialize: (state) => ({ token: state.token, user: state.user })
      }
    ),
    { name: 'AuthStore' }
  )
);

// store/cartStore.js
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import { useAuthStore } from './authStore';

export const useCartStore = create(
  devtools(
    persist(
      immer((set, get) => ({
        items: [],

        addItem: (product) => set((state) => {
          const existingItem = state.items.find(item => item.id === product.id);

          if (existingItem) {
            existingItem.quantity += 1;
          } else {
            state.items.push({ ...product, quantity: 1 });
          }
        }),

        removeItem: (productId) => set((state) => {
          state.items = state.items.filter(item => item.id !== productId);
        }),

        updateQuantity: (productId, quantity) => set((state) => {
          const item = state.items.find(item => item.id === productId);
          if (item) {
            if (quantity <= 0) {
              state.items = state.items.filter(item => item.id !== productId);
            } else {
              item.quantity = quantity;
            }
          }
        }),

        clearCart: () => set({ items: [] }),

        getTotal: () => {
          return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
        },

        getItemCount: () => {
          return get().items.reduce((sum, item) => sum + item.quantity, 0);
        },

        checkout: async () => {
          const { items } = get();
          const { user, token } = useAuthStore.getState();

          if (!user) {
            return { success: false, error: 'Please login first' };
          }

          try {
            const response = await fetch('/api/orders', {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${token}`
              },
              body: JSON.stringify({ items })
            });

            const order = await response.json();
            set({ items: [] });
            return { success: true, order };
          } catch (error) {
            return { success: false, error: error.message };
          }
        }
      })),
      { name: 'cart-storage' }
    ),
    { name: 'CartStore' }
  )
);

// components/App.jsx
function App() {
  const user = useAuthStore(state => state.user);

  return (
    <div>
      <Header />
      <main>
        {user ? <Dashboard /> : <LoginPage />}
      </main>
    </div>
  );
}

function Header() {
  const user = useAuthStore(state => state.user);
  const logout = useAuthStore(state => state.logout);
  const itemCount = useCartStore(state => state.getItemCount());

  return (
    <header>
      <h1>My Shop</h1>
      {user && (
        <div>
          <span>안녕하세요, {user.name}님</span>
          <CartIcon count={itemCount} />
          <button onClick={logout}>로그아웃</button>
        </div>
      )}
    </header>
  );
}

function ProductCard({ product }) {
  const addItem = useCartStore(state => state.addItem);

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.price.toLocaleString()}원</p>
      <button onClick={() => addItem(product)}>
        장바구니에 추가
      </button>
    </div>
  );
}

function Cart() {
  const items = useCartStore(state => state.items);
  const removeItem = useCartStore(state => state.removeItem);
  const updateQuantity = useCartStore(state => state.updateQuantity);
  const getTotal = useCartStore(state => state.getTotal);
  const checkout = useCartStore(state => state.checkout);

  const handleCheckout = async () => {
    const result = await checkout();
    if (result.success) {
      alert('주문이 완료되었습니다!');
    } else {
      alert(result.error);
    }
  };

  return (
    <div className="cart">
      <h2>장바구니</h2>
      {items.map(item => (
        <div key={item.id} className="cart-item">
          <img src={item.image} alt={item.name} />
          <div>
            <h4>{item.name}</h4>
            <p>{item.price.toLocaleString()}원</p>
          </div>
          <input
            type="number"
            value={item.quantity}
            onChange={e => updateQuantity(item.id, parseInt(e.target.value))}
            min="1"
          />
          <button onClick={() => removeItem(item.id)}>삭제</button>
        </div>
      ))}
      <div className="cart-total">
        <strong>총액: {getTotal().toLocaleString()}원</strong>
        <button onClick={handleCheckout}>결제하기</button>
      </div>
    </div>
  );
}
```

### Redux Toolkit 버전

```jsx
// store/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { cartSlice } from './cartSlice';

export const login = createAsyncThunk(
  'auth/login',
  async ({ email, password }, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });

      if (!response.ok) throw new Error('Login failed');

      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const updateProfile = createAsyncThunk(
  'auth/updateProfile',
  async (updates, { getState }) => {
    const { auth } = getState();
    const response = await fetch(`/api/users/${auth.user.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${auth.token}`
      },
      body: JSON.stringify(updates)
    });

    return await response.json();
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    loading: false,
    error: null
  },
  reducers: {
    logout: (state) => {
      state.user = null;
      state.token = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(login.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      .addCase(updateProfile.fulfilled, (state, action) => {
        state.user = action.payload;
      });
  }
});

export const { logout } = authSlice.actions;
export default authSlice.reducer;

// store/cartSlice.js
import { createSlice, createAsyncThunk, createSelector } from '@reduxjs/toolkit';

export const checkout = createAsyncThunk(
  'cart/checkout',
  async (_, { getState }) => {
    const { cart, auth } = getState();

    const response = await fetch('/api/orders', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${auth.token}`
      },
      body: JSON.stringify({ items: cart.items })
    });

    return await response.json();
  }
);

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: []
  },
  reducers: {
    addItem: (state, action) => {
      const existingItem = state.items.find(item => item.id === action.payload.id);

      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },

    removeItem: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    },

    updateQuantity: (state, action) => {
      const item = state.items.find(item => item.id === action.payload.id);
      if (item) {
        if (action.payload.quantity <= 0) {
          state.items = state.items.filter(item => item.id !== action.payload.id);
        } else {
          item.quantity = action.payload.quantity;
        }
      }
    },

    clearCart: (state) => {
      state.items = [];
    }
  },
  extraReducers: (builder) => {
    builder.addCase(checkout.fulfilled, (state) => {
      state.items = [];
    });
  }
});

export const { addItem, removeItem, updateQuantity, clearCart } = cartSlice.actions;

// Selectors
export const selectCartItems = state => state.cart.items;

export const selectCartTotal = createSelector(
  [selectCartItems],
  (items) => items.reduce((sum, item) => sum + item.price * item.quantity, 0)
);

export const selectItemCount = createSelector(
  [selectCartItems],
  (items) => items.reduce((sum, item) => sum + item.quantity, 0)
);

export default cartSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import authReducer, { logout } from './authSlice';
import cartReducer, { clearCart } from './cartSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer
  }
});

// 로그아웃 시 장바구니도 초기화하는 리스너
store.subscribe(() => {
  const state = store.getState();
  if (!state.auth.user) {
    store.dispatch(clearCart());
  }
});

// components/App.jsx
import { useSelector, useDispatch } from 'react-redux';
import { login, logout } from '../store/authSlice';
import { addItem, removeItem, updateQuantity, checkout, selectCartItems, selectCartTotal, selectItemCount } from '../store/cartSlice';

function App() {
  const user = useSelector(state => state.auth.user);

  return (
    <div>
      <Header />
      <main>
        {user ? <Dashboard /> : <LoginPage />}
      </main>
    </div>
  );
}

function Header() {
  const dispatch = useDispatch();
  const user = useSelector(state => state.auth.user);
  const itemCount = useSelector(selectItemCount);

  return (
    <header>
      <h1>My Shop</h1>
      {user && (
        <div>
          <span>안녕하세요, {user.name}님</span>
          <CartIcon count={itemCount} />
          <button onClick={() => dispatch(logout())}>로그아웃</button>
        </div>
      )}
    </header>
  );
}

function ProductCard({ product }) {
  const dispatch = useDispatch();

  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>{product.price.toLocaleString()}원</p>
      <button onClick={() => dispatch(addItem(product))}>
        장바구니에 추가
      </button>
    </div>
  );
}

function Cart() {
  const dispatch = useDispatch();
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);

  const handleCheckout = async () => {
    const result = await dispatch(checkout()).unwrap();
    alert('주문이 완료되었습니다!');
  };

  return (
    <div className="cart">
      <h2>장바구니</h2>
      {items.map(item => (
        <div key={item.id} className="cart-item">
          <img src={item.image} alt={item.name} />
          <div>
            <h4>{item.name}</h4>
            <p>{item.price.toLocaleString()}원</p>
          </div>
          <input
            type="number"
            value={item.quantity}
            onChange={e => dispatch(updateQuantity({ id: item.id, quantity: parseInt(e.target.value) }))}
            min="1"
          />
          <button onClick={() => dispatch(removeItem(item.id))}>삭제</button>
        </div>
      ))}
      <div className="cart-total">
        <strong>총액: {total.toLocaleString()}원</strong>
        <button onClick={handleCheckout}>결제하기</button>
      </div>
    </div>
  );
}
```

### 복잡한 예제 비교

| 측면 | Zustand | Redux Toolkit |
|------|---------|---------------|
| **스토어 간 통신** | 직접 접근 가능 | createAsyncThunk 사용 |
| **코드 복잡도** | 간단 | 중간 |
| **타입 안정성** | 좋음 | 우수 |
| **확장성** | 좋음 | 우수 |

## 프로젝트 규모별 선택 가이드

### 소규모 프로젝트 (1-5명, 3개월 이하)

**추천: Zustand 또는 Context API**

```jsx
// Zustand로 충분한 경우
- 간단한 전역 상태 (3-5개)
- 빠른 프로토타이핑
- 팀원의 React 경험 부족
- 번들 크기 최소화 중요

// 예시 프로젝트
- 개인 블로그
- 포트폴리오 사이트
- 간단한 대시보드
- MVP 제품
```

**장점**:
- 러닝 커브 거의 없음
- 빠른 개발 속도
- 작은 번들 크기

### 중규모 프로젝트 (5-15명, 6개월-1년)

**추천: Zustand 또는 Redux Toolkit**

```jsx
// Zustand 선택 시
- 복잡도가 중간 정도
- 빠른 개발 속도 중요
- 팀이 유연한 패턴 선호
- 미들웨어로 충분

// Redux Toolkit 선택 시
- 복잡한 상태 로직
- 엄격한 패턴 필요
- 타입 안정성 최우선
- 팀이 Redux 경험 있음

// 예시 프로젝트
- SaaS 제품
- E-commerce 사이트
- 관리자 대시보드
- 소셜 미디어 앱
```

**판단 기준**:
```jsx
// Zustand를 선택하세요
if (
  팀_러닝커브_중요 &&
  개발_속도_우선 &&
  번들크기_신경쓰임
) {
  return 'Zustand';
}

// Redux Toolkit을 선택하세요
if (
  복잡한_비즈니스로직 &&
  타입안정성_필수 &&
  팀이_Redux_경험있음
) {
  return 'Redux Toolkit';
}
```

### 대규모 프로젝트 (15명 이상, 1년 이상)

**추천: Redux Toolkit**

```jsx
// Redux Toolkit이 필수적인 경우
- 매우 복잡한 상태 로직
- 엄격한 코드 리뷰 필요
- 일관된 패턴 강제
- 대규모 팀 협업
- 장기적 유지보수

// 예시 프로젝트
- 엔터프라이즈 애플리케이션
- 금융 플랫폼
- 대형 E-commerce
- 복잡한 데이터 시각화 도구
```

**Redux Toolkit의 강점**:
- 표준화된 패턴
- 강력한 타입 시스템
- 풍부한 생태계
- 검증된 아키텍처
- Redux DevTools

### 특수 상황별 추천

```text
// 1. 서버 상태가 대부분인 경우
→ React Query + Zustand (클라이언트 상태)

// 2. 실시간 데이터 많은 경우
→ Redux Toolkit + RTK Query

// 3. 복잡한 폼이 많은 경우
→ React Hook Form + Zustand

// 4. 원자적 상태 관리 선호
→ Jotai

// 5. 마이크로 프론트엔드
→ Zustand (각 앱이 독립적)
```

## 마이그레이션 가이드

### Context API → Zustand

```jsx
// Before: Context API
const TodoContext = createContext(null);

export function TodoProvider({ children }) {
  const [todos, setTodos] = useState([]);

  const addTodo = (text) => {
    setTodos(prev => [...prev, { id: Date.now(), text }]);
  };

  const value = useMemo(() => ({ todos, addTodo }), [todos]);

  return <TodoContext.Provider value={value}>{children}</TodoContext.Provider>;
}

export function useTodos() {
  return useContext(TodoContext);
}

// After: Zustand
import { create } from 'zustand';

export const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text }]
  }))
}));

// 마이그레이션 단계:
// 1. Provider 제거 (App.jsx에서)
// 2. useContext를 useTodoStore로 변경
// 3. useMemo 제거 (Zustand가 자동 처리)
```

### Context API → Redux Toolkit

```jsx
// Before: Context API
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (email, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    const data = await response.json();
    setUser(data.user);
  };

  const value = useMemo(() => ({ user, login }), [user]);

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// After: Redux Toolkit
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const login = createAsyncThunk(
  'auth/login',
  async ({ email, password }) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    return await response.json();
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null },
  extraReducers: (builder) => {
    builder.addCase(login.fulfilled, (state, action) => {
      state.user = action.payload.user;
    });
  }
});

// 마이그레이션 단계:
// 1. Provider를 Redux Provider로 변경
// 2. useContext를 useSelector/useDispatch로 변경
// 3. 비동기 로직을 createAsyncThunk로 변경
```

### Redux → Zustand

```jsx
// Before: Redux
const ADD_TODO = 'ADD_TODO';

function addTodo(text) {
  return { type: ADD_TODO, payload: text };
}

function todoReducer(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [...state, { id: Date.now(), text: action.payload }];
    default:
      return state;
  }
}

// After: Zustand
const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text }]
  }))
}));

// 마이그레이션 전략:
// 1. 점진적 마이그레이션 (슬라이스 단위)
// 2. 먼저 단순한 상태부터
// 3. 복잡한 로직은 나중에
// 4. 일정 기간 공존 가능
```

## Best Practices

### Zustand 베스트 프랙티스

```jsx
// 1. 스토어 분리
// ❌ 하나의 거대한 스토어
const useStore = create((set) => ({
  user: null,
  todos: [],
  cart: [],
  // ... 100개의 상태
}));

// ✅ 관심사별로 분리
const useUserStore = create((set) => ({ /* ... */ }));
const useTodoStore = create((set) => ({ /* ... */ }));
const useCartStore = create((set) => ({ /* ... */ }));

// 2. Computed 값 활용
const useStore = create((set, get) => ({
  todos: [],

  // ❌ 나쁜 방법: 상태로 저장
  completedCount: 0,

  // ✅ 좋은 방법: Getter 함수
  getCompletedCount: () => get().todos.filter(t => t.completed).length
}));

// 3. 액션 그룹화
const useStore = create((set) => ({
  todos: [],

  // ✅ 액션을 객체로 그룹화
  actions: {
    addTodo: (text) => set((state) => ({
      todos: [...state.todos, { id: Date.now(), text }]
    })),
    deleteTodo: (id) => set((state) => ({
      todos: state.todos.filter(t => t.id !== id)
    }))
  }
}));

// 사용
const { addTodo, deleteTodo } = useStore(state => state.actions);

// 4. 선택적 구독으로 최적화
// ❌ 전체 상태 구독
const state = useStore();

// ✅ 필요한 부분만 구독
const todos = useStore(state => state.todos);
const addTodo = useStore(state => state.addTodo);

// ✅ 얕은 비교로 최적화
import { shallow } from 'zustand/shallow';
const { todos, addTodo } = useStore(
  state => ({ todos: state.todos, addTodo: state.addTodo }),
  shallow
);

// 5. TypeScript와 함께 사용
interface TodoStore {
  todos: Todo[];
  addTodo: (text: string) => void;
  deleteTodo: (id: number) => void;
}

const useTodoStore = create<TodoStore>((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text, completed: false }]
  })),
  deleteTodo: (id) => set((state) => ({
    todos: state.todos.filter(t => t.id !== id)
  }))
}));
```

### Redux Toolkit 베스트 프랙티스

```jsx
// 1. Feature 폴더 구조
/*
src/
  features/
    todos/
      todosSlice.js
      TodoList.jsx
      TodoItem.jsx
    auth/
      authSlice.js
      Login.jsx
    cart/
      cartSlice.js
      Cart.jsx
*/

// 2. createSelector로 메모이제이션
import { createSelector } from '@reduxjs/toolkit';

// ❌ 매번 새 배열 생성
export const selectActiveTodos = state =>
  state.todos.filter(t => !t.completed);

// ✅ 메모이제이션
export const selectActiveTodos = createSelector(
  [state => state.todos],
  (todos) => todos.filter(t => !t.completed)
);

// 3. extraReducers builder 패턴
const todoSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: { /* ... */ },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.items = action.payload;
        state.loading = false;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.error = action.error.message;
        state.loading = false;
      });
  }
});

// 4. 커스텀 Hook으로 타입 안전성
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// 사용
const todos = useAppSelector(state => state.todos);
const dispatch = useAppDispatch();

// 5. RTK Query 활용
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Todo'],
  endpoints: (builder) => ({
    getTodos: builder.query({
      query: () => 'todos',
      providesTags: ['Todo']
    }),
    addTodo: builder.mutation({
      query: (todo) => ({
        url: 'todos',
        method: 'POST',
        body: todo
      }),
      invalidatesTags: ['Todo']
    })
  })
});

// 6. 정규화된 상태 구조
// ❌ 중첩된 배열
{
  users: [
    { id: 1, name: 'Alice', posts: [...] }
  ]
}

// ✅ 정규화
{
  users: {
    byId: { 1: { id: 1, name: 'Alice', postIds: [1, 2] } },
    allIds: [1]
  },
  posts: {
    byId: { 1: { id: 1, title: '...' }, 2: { ... } },
    allIds: [1, 2]
  }
}

// createEntityAdapter 사용
import { createEntityAdapter } from '@reduxjs/toolkit';

const todosAdapter = createEntityAdapter();

const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState(),
  reducers: {
    todoAdded: todosAdapter.addOne,
    todosReceived: todosAdapter.setAll
  }
});

// 자동 생성된 selectors
export const {
  selectAll: selectAllTodos,
  selectById: selectTodoById,
  selectIds: selectTodoIds
} = todosAdapter.getSelectors(state => state.todos);
```

## FAQ

### Q1: Zustand와 Redux Toolkit 중 어떤 것을 선택해야 하나요?

**A**: 프로젝트 규모와 팀 상황에 따라 다릅니다:

**Zustand를 선택하세요**:
- 빠른 프로토타이핑이 필요할 때
- 팀이 작거나 Redux 경험이 없을 때
- 번들 크기가 중요할 때
- 유연한 패턴을 선호할 때
- 학습 시간이 부족할 때

**Redux Toolkit을 선택하세요**:
- 복잡한 비즈니스 로직이 많을 때
- 대규모 팀 협업이 필요할 때
- 엄격한 패턴과 타입 안정성이 중요할 때
- 팀이 Redux 경험이 있을 때
- 장기적 유지보수를 고려할 때

### Q2: Context API만으로는 부족한가요?

**A**: 프로젝트에 따라 다릅니다:

**Context API로 충분한 경우**:
- 전역 상태가 3개 이하
- 상태 변경이 자주 일어나지 않음
- 디버깅 도구가 필요 없음
- 매우 간단한 로직

**라이브러리가 필요한 경우**:
- 전역 상태가 5개 이상
- 상태 변경이 빈번함
- 성능 최적화가 중요함
- 디버깅 도구가 필요함
- 미들웨어 기능이 필요함

### Q3: Zustand는 작아서 기능이 부족하지 않나요?

**A**: 오히려 그 반대입니다:

```jsx
// Zustand는 작지만 강력합니다
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// DevTools, 영속화, Immer 모두 지원
const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        todos: [],
        addTodo: (text) => set((state) => {
          state.todos.push({ id: Date.now(), text });
        })
      })),
      { name: 'storage' }
    )
  )
);

// React 외부에서도 사용 가능
const state = useStore.getState();
useStore.setState({ todos: [] });

// 구독
const unsubscribe = useStore.subscribe(console.log);
```

필요한 기능은 미들웨어로 추가할 수 있습니다.

### Q4: Redux Toolkit은 여전히 복잡하지 않나요?

**A**: Legacy Redux에 비하면 80% 간단해졌습니다:

```jsx
// Legacy Redux: ~50줄
// - actionTypes.js
// - actionCreators.js
// - reducer.js
// - 불변성 수동 관리
// - 복잡한 스토어 설정

// Redux Toolkit: ~15줄
const slice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push({ id: Date.now(), text: action.payload }); // Immer!
    }
  }
});

const store = configureStore({ reducer: { todos: slice.reducer } });
```

### Q5: 여러 상태 관리 라이브러리를 함께 쓸 수 있나요?

**A**: 가능하며, 실무에서 자주 사용하는 패턴입니다:

```jsx
// 1. React Query + Zustand (추천 조합)
// - React Query: 서버 상태
// - Zustand: 클라이언트 상태

// 서버 상태
const { data: todos } = useQuery('todos', fetchTodos);

// 클라이언트 상태
const filter = useStore(state => state.filter);

// 2. Redux Toolkit + RTK Query
// - Redux: 복잡한 클라이언트 상태
// - RTK Query: API 호출

// 3. Zustand + Jotai
// - Zustand: 글로벌 상태
// - Jotai: 컴포넌트 레벨 상태
```

### Q6: 성능 차이가 큰가요?

**A**: 올바르게 사용하면 모두 훌륭한 성능을 제공합니다:

```jsx
// Zustand: 선택적 구독으로 최적화
const todos = useStore(state => state.todos); // todos 변경 시만 리렌더링

// Redux: createSelector로 메모이제이션
const selectTodos = createSelector(
  [state => state.todos],
  (todos) => todos.filter(t => !t.completed)
);

// 실측 결과 (10,000개 아이템):
// Zustand: ~5ms
// Redux Toolkit: ~6ms
// Context API: ~15ms (최적화 없이)
```

차이는 미미하며, 코드 품질이 더 중요합니다.

### Q7: TypeScript 지원은 어떤가요?

**A**: 모두 훌륭한 TypeScript 지원을 제공합니다:

```typescript
// Zustand
interface TodoStore {
  todos: Todo[];
  addTodo: (text: string) => void;
}

const useStore = create<TodoStore>((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({ todos: [...state.todos, { id: Date.now(), text }] }))
}));

// Redux Toolkit - 타입 추론 자동
const slice = createSlice({
  name: 'todos',
  initialState: [] as Todo[],
  reducers: {
    addTodo: (state, action: PayloadAction<string>) => {
      state.push({ id: Date.now(), text: action.payload });
    }
  }
});

// 둘 다 완벽한 타입 안정성을 제공합니다
```

### Q8: 기존 프로젝트에 도입하기 쉬운가요?

**A**: Zustand가 더 쉽습니다:

```jsx
// Zustand: Provider 불필요, 즉시 사용 가능
import { create } from 'zustand';
const useStore = create((set) => ({ count: 0 }));
// 끝!

// Redux: Provider 설정 필요
import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  root
);
```

점진적 도입도 Zustand가 더 유리합니다.

## 결론

2025년 현재, React 전역 상태 관리는 다양한 선택지가 있습니다. 정답은 없으며, **프로젝트의 요구사항과 팀의 상황**에 따라 결정해야 합니다.

### 핵심 요약

**Zustand를 선택하세요**:
- 빠른 개발이 필요할 때
- 작은 번들 크기가 중요할 때
- 러닝 커브를 최소화하고 싶을 때
- 유연한 패턴을 선호할 때

**Redux Toolkit을 선택하세요**:
- 복잡한 비즈니스 로직이 많을 때
- 엄격한 패턴이 필요할 때
- 대규모 팀 협업을 할 때
- 타입 안정성을 최우선으로 할 때

**Jotai를 선택하세요**:
- 원자적 상태 관리를 선호할 때
- React Hooks 스타일을 좋아할 때
- 작은 번들 크기가 중요할 때

**Recoil은 아직 신중하게**:
- 실험적 단계이므로 프로덕션에 주의
- Facebook도 제한적으로 사용 중

### 실전 추천

```text
// 소규모 프로젝트
Context API → 충분하면 그대로, 부족하면 Zustand

// 중규모 프로젝트
Zustand (빠른 개발) vs Redux Toolkit (안정성)

// 대규모 프로젝트
Redux Toolkit (표준) + RTK Query (API)

// 모든 프로젝트
서버 상태는 React Query / SWR 사용 권장
```

### 최종 조언

1. **일찍 결정하지 마세요**: 처음에는 Context API로 시작하고, 필요할 때 라이브러리 도입
2. **서버 상태 분리하세요**: React Query로 서버 상태를 관리하면 전역 상태가 줄어듭니다
3. **팀과 논의하세요**: 팀의 경험과 선호도가 가장 중요합니다
4. **마이그레이션 고려하세요**: 나중에 변경할 수 있으니 너무 걱정하지 마세요

### 다음 단계

이제 전역 상태 관리 라이브러리를 마스터했으니, 다음 포스팅에서는 **서버 상태 관리의 혁명**을 다룰 예정입니다:

- **React Query (TanStack Query)**: 서버 상태 관리의 표준
- **SWR**: Vercel의 경량 대안
- 캐싱, 리페칭, 옵티미스틱 업데이트
- 실시간 데이터 동기화

서버 상태를 올바르게 관리하면 전역 상태 관리 코드가 80% 줄어듭니다!

상태 관리 시리즈의 다음 글도 기대해주세요. 궁금한 점이나 프로젝트 경험을 댓글로 공유해주시면 감사하겠습니다!

## 참고 자료

- [Zustand 공식 문서](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [Redux Toolkit 공식 문서](https://redux-toolkit.js.org/)
- [Jotai 공식 문서](https://jotai.org/)
- [Recoil 공식 문서](https://recoiljs.org/)
- [Redux vs Context API vs Zustand](https://blog.logrocket.com/zustand-vs-redux-vs-context-api/)
- [State Management in React 2024](https://2023.stateofjs.com/en-US/libraries/state-management/)
- [When to use Redux](https://redux.js.org/faq/general#when-should-i-use-redux)
- [React 공식 문서 - Managing State](https://react.dev/learn/managing-state)
