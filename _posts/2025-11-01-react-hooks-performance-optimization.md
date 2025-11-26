---
title: React Hooks 성능 최적화 완벽 가이드 - useMemo와 useCallback 활용법
date: 2025-11-01 22:00:00 +0900
categories: [개발, React]
tags: [react, hooks, performance]
author: changsu
toc: true
comments: true
# 이미지를 추가한 후 아래 주석을 해제하세요
# image:
#   path: /assets/img/posts/react-hooks-optimization.jpg
#   alt: React Hooks 성능 최적화
description: React Hooks를 활용한 실전 성능 최적화 기법을 알아봅니다. useMemo와 useCallback의 올바른 사용법과 성능 측정 방법을 실용적인 코드 예시와 함께 설명합니다. 불필요한 리렌더링을 방지하고 연산 비용을 줄이는 메모이제이션 기법을 배우며, React DevTools Profiler로 성능을 측정하고 개선하는 방법을 마스터합니다.
---

## 들어가며

React 애플리케이션이 커지면서 성능 문제에 직면한 적이 있나요? 불필요한 리렌더링과 연산으로 인해 사용자 경험이 저하되는 것을 경험하셨을 겁니다. 이번 포스팅에서는 React Hooks를 활용한 실전 성능 최적화 기법을 다룹니다.

## 성능 문제의 주범: 불필요한 리렌더링

React 컴포넌트는 다음과 같은 경우에 리렌더링됩니다:

- 상태(state)가 변경될 때
- Props가 변경될 때
- 부모 컴포넌트가 리렌더링될 때

문제는 **필요하지 않은 경우에도 리렌더링이 발생**한다는 점입니다.

```jsx
function ParentComponent() {
  const [count, setCount] = useState(0);

  // 매 렌더링마다 새로운 함수 생성
  const handleClick = () => {
    console.log('Clicked!');
  };

  // 매 렌더링마다 새로운 배열 생성
  const items = [1, 2, 3].map(n => n * 2);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ChildComponent onClick={handleClick} items={items} />
    </div>
  );
}
```

위 코드의 문제점은 `count`가 변경될 때마다 `handleClick` 함수와 `items` 배열이 새로 생성되어 `ChildComponent`가 불필요하게 리렌더링된다는 것입니다.

## useMemo: 값 메모이제이션

`useMemo`는 **계산 비용이 높은 연산의 결과를 메모이제이션**합니다.

### 기본 사용법

```jsx
const memoizedValue = useMemo(() => {
  // 계산 비용이 높은 연산
  return computeExpensiveValue(a, b);
}, [a, b]); // 의존성 배열
```

### 실전 예시: 필터링 최적화

```jsx
function UserList({ users, searchTerm }) {
  // searchTerm이나 users가 변경될 때만 재계산
  const filteredUsers = useMemo(() => {
    console.log('필터링 실행...');
    return users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);

  return (
    <ul>
      {filteredUsers.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### useMemo 활용 팁

> **언제 사용해야 할까?**
> - 복잡한 계산이나 변환 작업
> - 큰 배열/객체의 필터링, 정렬
> - 자식 컴포넌트에 전달되는 객체/배열
{: .prompt-tip }

> **주의사항**
> - 단순한 연산에는 오히려 오버헤드 발생
> - 의존성 배열을 정확하게 설정해야 함
{: .prompt-warning }

## useCallback: 함수 메모이제이션

`useCallback`은 **함수를 메모이제이션하여 참조를 유지**합니다.

### 기본 사용법

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### 실전 예시: 이벤트 핸들러 최적화

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  // filter가 변경될 때만 새 함수 생성
  const handleAddTodo = useCallback((text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  }, []); // 의존성 없음

  const handleToggle = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []); // 함수형 업데이트로 의존성 제거

  const filteredTodos = useMemo(() => {
    if (filter === 'active') return todos.filter(t => !t.completed);
    if (filter === 'completed') return todos.filter(t => t.completed);
    return todos;
  }, [todos, filter]);

  return (
    <div>
      <TodoInput onAdd={handleAddTodo} />
      <FilterButtons activeFilter={filter} onChange={setFilter} />
      <TodoItems todos={filteredTodos} onToggle={handleToggle} />
    </div>
  );
}

// React.memo로 Props 비교 최적화
const TodoItems = React.memo(({ todos, onToggle }) => {
  console.log('TodoItems 렌더링');
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} onToggle={onToggle} />
      ))}
    </ul>
  );
});
```

### useCallback과 React.memo 조합

```jsx
const ChildComponent = React.memo(({ onClick, data }) => {
  console.log('ChildComponent 렌더링');
  return <button onClick={onClick}>{data.title}</button>;
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [data] = useState({ title: 'Click me' });

  // useCallback 없이: count 변경 시마다 ChildComponent 리렌더링
  // useCallback 사용: count 변경 시에도 ChildComponent 리렌더링 안 됨
  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ChildComponent onClick={handleClick} data={data} />
    </div>
  );
}
```

## 성능 측정 방법

### 1. React DevTools Profiler

React DevTools의 Profiler 탭을 사용하면 컴포넌트 렌더링 성능을 시각적으로 확인할 수 있습니다.

```jsx
import { Profiler } from 'react';

function onRenderCallback(
  id, // 프로파일러 ID
  phase, // "mount" 또는 "update"
  actualDuration, // 렌더링에 걸린 시간
  baseDuration, // 메모이제이션 없이 걸릴 시간
  startTime,
  commitTime
) {
  console.log(`${id} ${phase} 렌더링 시간: ${actualDuration}ms`);
}

<Profiler id="TodoList" onRender={onRenderCallback}>
  <TodoList />
</Profiler>
```

### 2. Performance API 활용

```jsx
function ExpensiveComponent() {
  useEffect(() => {
    const start = performance.now();

    // 렌더링 완료 후 측정
    requestAnimationFrame(() => {
      const end = performance.now();
      console.log(`렌더링 시간: ${end - start}ms`);
    });
  });

  return <div>{/* ... */}</div>;
}
```

### 3. 커스텀 Hook으로 렌더링 추적

```jsx
function useRenderCount(componentName) {
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current += 1;
    console.log(`${componentName} 렌더링 횟수: ${renderCount.current}`);
  });
}

// 사용 예시
function MyComponent() {
  useRenderCount('MyComponent');
  // ...
}
```

## 최적화 체크리스트

| 항목 | 설명 | 우선순위 |
|------|------|----------|
| **불필요한 리렌더링 확인** | React DevTools로 렌더링 추적 | 높음 |
| **큰 리스트 최적화** | 가상화(react-window) 고려 | 높음 |
| **useMemo 적용** | 복잡한 계산, 필터링, 정렬 | 중간 |
| **useCallback 적용** | 자식 컴포넌트에 전달되는 함수 | 중간 |
| **React.memo 적용** | Props가 자주 바뀌지 않는 컴포넌트 | 중간 |
| **코드 스플리팅** | React.lazy와 Suspense 활용 | 낮음 |

## 안티패턴 주의

❌ **모든 곳에 최적화 적용하지 마세요**

```jsx
// 나쁜 예: 간단한 계산에 useMemo 사용
const doubled = useMemo(() => count * 2, [count]);

// 좋은 예: 그냥 계산
const doubled = count * 2;
```

❌ **의존성 배열 누락**

```jsx
// 나쁜 예: searchTerm을 의존성에 포함하지 않음
const filtered = useMemo(() => {
  return items.filter(item => item.includes(searchTerm));
}, [items]); // searchTerm 변경 시 업데이트 안 됨!

// 좋은 예
const filtered = useMemo(() => {
  return items.filter(item => item.includes(searchTerm));
}, [items, searchTerm]);
```

## 마치며

React Hooks를 활용한 성능 최적화는 **측정 → 분석 → 최적화**의 과정을 거쳐야 합니다.

> 성급한 최적화는 모든 악의 근원입니다. 먼저 측정하고, 병목을 찾은 후, 필요한 곳에만 최적화를 적용하세요.
{: .prompt-info }

**핵심 정리:**
- `useMemo`: 값 메모이제이션 (계산 결과 캐싱)
- `useCallback`: 함수 메모이제이션 (함수 참조 유지)
- `React.memo`: 컴포넌트 메모이제이션 (Props 비교)
- 항상 성능을 측정하고 최적화하세요

다음 포스팅에서는 React Query와 Recoil을 활용한 상태 관리 최적화를 다루겠습니다. 궁금한 점이나 추가로 다뤘으면 하는 주제가 있다면 댓글로 남겨주세요!

## 참고 자료

- [React 공식 문서 - Hooks API Reference](https://react.dev/reference/react)
- [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
