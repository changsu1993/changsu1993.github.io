---
title: "TDD(테스트 주도 개발) 실전 가이드: Red-Green-Refactor 사이클로 배우는 개발 방법론"
date: 2025-11-09 18:00:00 +0900
categories: [Testing, TDD]
tags: [testing, tdd]
description: "실전 예제로 배우는 TDD(테스트 주도 개발) 완벽 가이드. Red-Green-Refactor 사이클을 통해 Todo 앱을 단계별로 구현하며 TDD의 핵심 개념과 실무 활용법을 익힙니다."
author: changsu
---

## 개요

TDD(Test-Driven Development, 테스트 주도 개발)는 테스트를 먼저 작성하고 그 테스트를 통과하는 코드를 작성하는 개발 방법론입니다. 이 글에서는 실전 Todo 앱 구현을 통해 TDD의 Red-Green-Refactor 사이클을 실습하고, 실무에서 바로 적용할 수 있는 TDD 노하우를 공유합니다.

**이 글에서 배울 수 있는 것:**
- TDD의 핵심 개념과 Red-Green-Refactor 사이클
- 실전 예제로 배우는 TDD 적용 방법
- 점진적 개발과 리팩토링 기법
- TDD의 장단점과 실무 활용 전략

**사전 요구사항:**
- JavaScript/React 기본 지식
- Jest 및 React Testing Library 기초 ([프론트엔드 테스팅 완벽 가이드](https://changsu1993.github.io/posts/frontend-testing-guide/) 참고)
- 단위 테스트 작성 경험

**예상 소요 시간:** 약 25분

---

## 목차

1. [TDD란 무엇인가?](#tdd란-무엇인가)
2. [Red-Green-Refactor 사이클](#red-green-refactor-사이클)
3. [실전 예제: Todo 앱 TDD로 구현하기](#실전-예제-todo-앱-tdd로-구현하기)
4. [TDD 실전 팁](#tdd-실전-팁)
5. [TDD의 장단점](#tdd의-장단점)
6. [자주 묻는 질문 (FAQ)](#자주-묻는-질문-faq)
7. [참고 자료](#참고-자료)

---

## TDD란 무엇인가?

### TDD의 정의

TDD는 **테스트를 먼저 작성**하고, 그 테스트를 통과하는 **최소한의 코드**를 작성한 후, **리팩토링**을 통해 코드 품질을 개선하는 개발 방법론입니다.

### TDD의 핵심 원칙

1. **실패하는 테스트를 먼저 작성** (Red)
2. **테스트를 통과하는 최소한의 코드 작성** (Green)
3. **중복 제거 및 코드 개선** (Refactor)

### 일반적인 개발 vs TDD

**일반적인 개발 프로세스:**

```
요구사항 분석 → 설계 → 구현 → 테스트 작성 → 디버깅
```

**TDD 프로세스:**

```
요구사항 분석 → 테스트 작성 → 구현 → 리팩토링
```

TDD에서는 **테스트가 명세**가 되고, **구현이 테스트를 따라갑니다**.

---

## Red-Green-Refactor 사이클

TDD의 핵심은 Red-Green-Refactor 사이클입니다.

<div style="display: flex; justify-content: center; align-items: center; gap: 30px; margin: 30px 0; flex-wrap: wrap;">
  <div style="text-align: center;">
    <div style="width: 120px; height: 120px; border-radius: 50%; background: linear-gradient(135deg, #ef5350 0%, #e53935 100%); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 18px; box-shadow: 0 4px 15px rgba(239, 83, 80, 0.4); margin: 0 auto 10px;">
      RED
    </div>
    <div style="color: #666; font-size: 14px; max-width: 150px;">실패하는<br/>테스트 작성</div>
  </div>

  <div style="font-size: 30px; color: #999;">→</div>

  <div style="text-align: center;">
    <div style="width: 120px; height: 120px; border-radius: 50%; background: linear-gradient(135deg, #66bb6a 0%, #43a047 100%); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 18px; box-shadow: 0 4px 15px rgba(102, 187, 106, 0.4); margin: 0 auto 10px;">
      GREEN
    </div>
    <div style="color: #666; font-size: 14px; max-width: 150px;">최소한의 코드로<br/>테스트 통과</div>
  </div>

  <div style="font-size: 30px; color: #999;">→</div>

  <div style="text-align: center;">
    <div style="width: 120px; height: 120px; border-radius: 50%; background: linear-gradient(135deg, #42a5f5 0%, #1e88e5 100%); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 18px; box-shadow: 0 4px 15px rgba(66, 165, 245, 0.4); margin: 0 auto 10px;">
      REFACTOR
    </div>
    <div style="color: #666; font-size: 14px; max-width: 150px;">코드 개선 및<br/>중복 제거</div>
  </div>
</div>

### 1. Red 단계: 실패하는 테스트 작성

먼저 구현하려는 기능에 대한 테스트를 작성합니다. 이 단계에서는 **아직 구현이 없기 때문에 테스트가 실패**합니다.

**목적:**
- 요구사항을 명확히 정의
- 구현의 인터페이스(API) 설계
- 테스트 가능한 코드 유도

### 2. Green 단계: 테스트를 통과하는 최소한의 코드 작성

테스트를 통과할 수 있는 **최소한의 코드**를 작성합니다. 이 단계에서는 **코드 품질보다 테스트 통과가 우선**입니다.

**목적:**
- 빠른 피드백 루프
- 작은 단위로 진행
- 테스트가 실제로 동작하는지 확인

### 3. Refactor 단계: 코드 개선

테스트가 통과한 상태에서 **중복 제거, 가독성 개선, 성능 최적화** 등을 진행합니다. 테스트가 있기 때문에 **안전하게 리팩토링**할 수 있습니다.

**목적:**
- 코드 품질 향상
- 중복 제거
- 설계 개선
- 유지보수성 증대

---

## 실전 예제: Todo 앱 TDD로 구현하기

실제 Todo 앱을 TDD로 구현하며 Red-Green-Refactor 사이클을 체험해봅시다.

### 프로젝트 설정

**package.json 설정:**

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.1.4",
    "@testing-library/user-event": "^14.5.1",
    "jest": "^29.7.0",
    "jest-environment-jsdom": "^29.7.0"
  }
}
```

**Jest 설정 (jest.config.js):**

```javascript
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy'
  }
};
```

**setupTests.js:**

```javascript
import '@testing-library/jest-dom';
```

---

### 기능 1: Todo 추가하기

#### Red: 실패하는 테스트 작성

```javascript
// src/components/TodoApp.test.js
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import TodoApp from './TodoApp';

describe('TodoApp', () => {
  test('사용자가 새로운 Todo를 추가할 수 있다', async () => {
    const user = userEvent.setup();
    render(<TodoApp />);

    const input = screen.getByPlaceholderText('할 일을 입력하세요');
    const addButton = screen.getByRole('button', { name: '추가' });

    await user.type(input, '장보기');
    await user.click(addButton);

    expect(screen.getByText('장보기')).toBeInTheDocument();
  });
});
```

**실행 결과:** ❌ FAIL (TodoApp 컴포넌트가 없음)

#### Green: 최소한의 코드로 테스트 통과

```jsx
// src/components/TodoApp.jsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const handleAdd = () => {
    setTodos([...todos, input]);
    setInput('');
  };

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
    </div>
  );
}

export default TodoApp;
```

**실행 결과:** ✅ PASS

#### Refactor: 코드 개선

현재는 리팩토링할 부분이 적지만, 향후를 위해 Todo를 객체로 관리하도록 개선합니다.

```jsx
// src/components/TodoApp.jsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const handleAdd = () => {
    if (!input.trim()) return;

    const newTodo = {
      id: Date.now(),
      text: input,
      completed: false
    };

    setTodos([...todos, newTodo]);
    setInput('');
  };

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}

export default TodoApp;
```

**실행 결과:** ✅ PASS (리팩토링 후에도 테스트 통과)

---

### 기능 2: 빈 입력 검증

#### Red: 실패하는 테스트 작성

```javascript
// src/components/TodoApp.test.js
test('빈 문자열은 추가되지 않는다', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  const addButton = screen.getByRole('button', { name: '추가' });
  await user.click(addButton);

  const listItems = screen.queryAllByRole('listitem');
  expect(listItems).toHaveLength(0);
});

test('공백만 있는 문자열은 추가되지 않는다', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });

  await user.type(input, '   ');
  await user.click(addButton);

  const listItems = screen.queryAllByRole('listitem');
  expect(listItems).toHaveLength(0);
});
```

**실행 결과:** ✅ PASS (이미 `if (!input.trim())` 검증 로직이 있음)

> 💡 **TDD의 장점**: 리팩토링 단계에서 미리 검증 로직을 추가했기 때문에 새로운 테스트가 바로 통과합니다!

---

### 기능 3: Todo 완료 토글

#### Red: 실패하는 테스트 작성

```javascript
// src/components/TodoApp.test.js
test('Todo를 클릭하면 완료 상태가 토글된다', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });

  await user.type(input, '운동하기');
  await user.click(addButton);

  const todoItem = screen.getByText('운동하기');

  expect(todoItem).not.toHaveStyle({ textDecoration: 'line-through' });

  await user.click(todoItem);
  expect(todoItem).toHaveStyle({ textDecoration: 'line-through' });

  await user.click(todoItem);
  expect(todoItem).not.toHaveStyle({ textDecoration: 'line-through' });
});
```

**실행 결과:** ❌ FAIL (토글 기능 없음)

#### Green: 테스트를 통과하는 코드 작성

```jsx
// src/components/TodoApp.jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const handleAdd = () => {
    if (!input.trim()) return;

    const newTodo = {
      id: Date.now(),
      text: input,
      completed: false
    };

    setTodos([...todos, newTodo]);
    setInput('');
  };

  const handleToggle = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <li
            key={todo.id}
            onClick={() => handleToggle(todo.id)}
            style={{
              textDecoration: todo.completed ? 'line-through' : 'none',
              cursor: 'pointer'
            }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**실행 결과:** ✅ PASS

#### Refactor: TodoItem 컴포넌트 분리

```jsx
// src/components/TodoItem.jsx
function TodoItem({ todo, onToggle }) {
  return (
    <li
      onClick={() => onToggle(todo.id)}
      style={{
        textDecoration: todo.completed ? 'line-through' : 'none',
        cursor: 'pointer'
      }}
    >
      {todo.text}
    </li>
  );
}

export default TodoItem;
```

```jsx
// src/components/TodoApp.jsx
import TodoItem from './TodoItem';

function TodoApp() {
  // ... 이전 코드 동일

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <TodoItem key={todo.id} todo={todo} onToggle={handleToggle} />
        ))}
      </ul>
    </div>
  );
}
```

**실행 결과:** ✅ PASS (리팩토링 후에도 테스트 통과)

---

### 기능 4: Todo 삭제

#### Red: 실패하는 테스트 작성

```javascript
// src/components/TodoApp.test.js
test('삭제 버튼을 클릭하면 Todo가 제거된다', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });

  await user.type(input, '책 읽기');
  await user.click(addButton);

  expect(screen.getByText('책 읽기')).toBeInTheDocument();

  const deleteButton = screen.getByRole('button', { name: '삭제' });
  await user.click(deleteButton);

  expect(screen.queryByText('책 읽기')).not.toBeInTheDocument();
});
```

**실행 결과:** ❌ FAIL (삭제 버튼 없음)

#### Green: 테스트를 통과하는 코드 작성

```jsx
// src/components/TodoItem.jsx
function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li style={{ display: 'flex', gap: '10px', alignItems: 'center' }}>
      <span
        onClick={() => onToggle(todo.id)}
        style={{
          textDecoration: todo.completed ? 'line-through' : 'none',
          cursor: 'pointer',
          flex: 1
        }}
      >
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </li>
  );
}

export default TodoItem;
```

```jsx
// src/components/TodoApp.jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const handleAdd = () => {
    if (!input.trim()) return;

    const newTodo = {
      id: Date.now(),
      text: input,
      completed: false
    };

    setTodos([...todos, newTodo]);
    setInput('');
  };

  const handleToggle = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const handleDelete = (id) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>
      <ul>
        {todos.map((todo) => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}
```

**실행 결과:** ✅ PASS

---

### 기능 5: 필터링 (전체/완료/미완료)

#### Red: 실패하는 테스트 작성

```javascript
// src/components/TodoApp.test.js
test('필터링 기능이 정상 동작한다', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });

  await user.type(input, '운동하기');
  await user.click(addButton);
  await user.type(input, '공부하기');
  await user.click(addButton);

  const firstTodo = screen.getByText('운동하기');
  await user.click(firstTodo);

  const allButton = screen.getByRole('button', { name: '전체' });
  const activeButton = screen.getByRole('button', { name: '미완료' });
  const completedButton = screen.getByRole('button', { name: '완료' });

  await user.click(completedButton);
  expect(screen.getByText('운동하기')).toBeInTheDocument();
  expect(screen.queryByText('공부하기')).not.toBeInTheDocument();

  await user.click(activeButton);
  expect(screen.queryByText('운동하기')).not.toBeInTheDocument();
  expect(screen.getByText('공부하기')).toBeInTheDocument();

  await user.click(allButton);
  expect(screen.getByText('운동하기')).toBeInTheDocument();
  expect(screen.getByText('공부하기')).toBeInTheDocument();
});
```

**실행 결과:** ❌ FAIL (필터 버튼 없음)

#### Green: 테스트를 통과하는 코드 작성

```jsx
// src/components/TodoApp.jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  const [filter, setFilter] = useState('all');

  const handleAdd = () => {
    if (!input.trim()) return;

    const newTodo = {
      id: Date.now(),
      text: input,
      completed: false
    };

    setTodos([...todos, newTodo]);
    setInput('');
  };

  const handleToggle = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const handleDelete = (id) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  const getFilteredTodos = () => {
    if (filter === 'completed') {
      return todos.filter((todo) => todo.completed);
    }
    if (filter === 'active') {
      return todos.filter((todo) => !todo.completed);
    }
    return todos;
  };

  const filteredTodos = getFilteredTodos();

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>

      <div style={{ margin: '10px 0' }}>
        <button onClick={() => setFilter('all')}>전체</button>
        <button onClick={() => setFilter('active')}>미완료</button>
        <button onClick={() => setFilter('completed')}>완료</button>
      </div>

      <ul>
        {filteredTodos.map((todo) => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}
```

**실행 결과:** ✅ PASS

#### Refactor: Custom Hook으로 로직 분리

TodoApp 컴포넌트가 복잡해졌으므로 비즈니스 로직을 Custom Hook으로 분리합니다.

```javascript
// src/hooks/useTodos.js
import { useState } from 'react';

function useTodos() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  const addTodo = (text) => {
    if (!text.trim()) return;

    const newTodo = {
      id: Date.now(),
      text,
      completed: false
    };

    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  const getFilteredTodos = () => {
    if (filter === 'completed') {
      return todos.filter((todo) => todo.completed);
    }
    if (filter === 'active') {
      return todos.filter((todo) => !todo.completed);
    }
    return todos;
  };

  return {
    todos: getFilteredTodos(),
    filter,
    setFilter,
    addTodo,
    toggleTodo,
    deleteTodo
  };
}

export default useTodos;
```

```jsx
// src/components/TodoApp.jsx
import { useState } from 'react';
import TodoItem from './TodoItem';
import useTodos from '../hooks/useTodos';

function TodoApp() {
  const [input, setInput] = useState('');
  const { todos, filter, setFilter, addTodo, toggleTodo, deleteTodo } = useTodos();

  const handleAdd = () => {
    addTodo(input);
    setInput('');
  };

  return (
    <div>
      <input
        type="text"
        placeholder="할 일을 입력하세요"
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleAdd}>추가</button>

      <div style={{ margin: '10px 0' }}>
        <button onClick={() => setFilter('all')}>전체</button>
        <button onClick={() => setFilter('active')}>미완료</button>
        <button onClick={() => setFilter('completed')}>완료</button>
      </div>

      <ul>
        {todos.map((todo) => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={toggleTodo}
            onDelete={deleteTodo}
          />
        ))}
      </ul>
    </div>
  );
}

export default TodoApp;
```

**실행 결과:** ✅ PASS (리팩토링 후에도 모든 테스트 통과)

#### Refactor: Custom Hook 테스트 작성

Custom Hook을 분리했으므로 독립적인 테스트를 추가합니다.

```javascript
// src/hooks/useTodos.test.js
import { renderHook, act } from '@testing-library/react';
import useTodos from './useTodos';

describe('useTodos', () => {
  test('새로운 Todo를 추가할 수 있다', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('장보기');
    });

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.todos[0].text).toBe('장보기');
    expect(result.current.todos[0].completed).toBe(false);
  });

  test('빈 문자열은 추가되지 않는다', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('   ');
    });

    expect(result.current.todos).toHaveLength(0);
  });

  test('Todo를 완료 처리할 수 있다', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('운동하기');
    });

    const todoId = result.current.todos[0].id;

    act(() => {
      result.current.toggleTodo(todoId);
    });

    expect(result.current.todos[0].completed).toBe(true);
  });

  test('Todo를 삭제할 수 있다', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('책 읽기');
    });

    const todoId = result.current.todos[0].id;

    act(() => {
      result.current.deleteTodo(todoId);
    });

    expect(result.current.todos).toHaveLength(0);
  });

  test('필터링이 정상 동작한다', () => {
    const { result } = renderHook(() => useTodos());

    act(() => {
      result.current.addTodo('운동하기');
      result.current.addTodo('공부하기');
    });

    const firstTodoId = result.current.todos[0].id;

    act(() => {
      result.current.toggleTodo(firstTodoId);
    });

    act(() => {
      result.current.setFilter('completed');
    });

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.todos[0].text).toBe('운동하기');

    act(() => {
      result.current.setFilter('active');
    });

    expect(result.current.todos).toHaveLength(1);
    expect(result.current.todos[0].text).toBe('공부하기');

    act(() => {
      result.current.setFilter('all');
    });

    expect(result.current.todos).toHaveLength(2);
  });
});
```

**실행 결과:** ✅ PASS

---

## TDD 실전 팁

### 1. 작은 단위로 시작하기

TDD는 **작은 단위의 기능**부터 시작하는 것이 중요합니다.

**나쁜 예:**

```javascript
test('Todo 앱의 모든 기능이 동작한다', () => {
  // 추가, 삭제, 완료, 필터링을 한 번에 테스트
});
```

**좋은 예:**

```javascript
test('새로운 Todo를 추가할 수 있다', () => { ... });
test('Todo를 삭제할 수 있다', () => { ... });
test('Todo를 완료 처리할 수 있다', () => { ... });
```

### 2. AAA 패턴 활용

테스트는 **Arrange(준비) - Act(실행) - Assert(검증)** 패턴을 따릅니다.

```javascript
test('사용자가 새로운 Todo를 추가할 수 있다', async () => {
  // Arrange: 테스트 환경 준비
  const user = userEvent.setup();
  render(<TodoApp />);
  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });

  // Act: 동작 실행
  await user.type(input, '장보기');
  await user.click(addButton);

  // Assert: 결과 검증
  expect(screen.getByText('장보기')).toBeInTheDocument();
});
```

### 3. 테스트 이름을 명확하게

테스트 이름은 **무엇을 테스트하는지** 명확히 표현해야 합니다.

**나쁜 예:**

```javascript
test('works', () => { ... });
test('test1', () => { ... });
```

**좋은 예:**

```javascript
test('빈 문자열은 추가되지 않는다', () => { ... });
test('삭제 버튼을 클릭하면 Todo가 제거된다', () => { ... });
```

### 4. 하나의 테스트는 하나의 개념만

테스트는 **하나의 동작이나 개념**만 검증해야 합니다.

**나쁜 예:**

```javascript
test('Todo 추가 및 삭제', async () => {
  // 추가 테스트
  await user.type(input, '장보기');
  await user.click(addButton);
  expect(screen.getByText('장보기')).toBeInTheDocument();

  // 삭제 테스트
  const deleteButton = screen.getByRole('button', { name: '삭제' });
  await user.click(deleteButton);
  expect(screen.queryByText('장보기')).not.toBeInTheDocument();
});
```

**좋은 예:**

```javascript
test('사용자가 새로운 Todo를 추가할 수 있다', async () => {
  await user.type(input, '장보기');
  await user.click(addButton);
  expect(screen.getByText('장보기')).toBeInTheDocument();
});

test('삭제 버튼을 클릭하면 Todo가 제거된다', async () => {
  await user.type(input, '장보기');
  await user.click(addButton);
  const deleteButton = screen.getByRole('button', { name: '삭제' });
  await user.click(deleteButton);
  expect(screen.queryByText('장보기')).not.toBeInTheDocument();
});
```

### 5. Given-When-Then 패턴

BDD(Behavior-Driven Development) 스타일로 테스트를 작성할 수도 있습니다.

```javascript
test('Todo를 클릭하면 완료 상태가 토글된다', async () => {
  // Given: 운동하기 Todo가 추가된 상태
  const user = userEvent.setup();
  render(<TodoApp />);
  const input = screen.getByPlaceholderText('할 일을 입력하세요');
  const addButton = screen.getByRole('button', { name: '추가' });
  await user.type(input, '운동하기');
  await user.click(addButton);

  // When: Todo를 클릭하면
  const todoItem = screen.getByText('운동하기');
  await user.click(todoItem);

  // Then: 완료 상태가 된다
  expect(todoItem).toHaveStyle({ textDecoration: 'line-through' });
});
```

### 6. 테스트 주도로 리팩토링

테스트가 있으면 **안전하게 리팩토링**할 수 있습니다.

```javascript
// 리팩토링 전
const handleToggle = (id) => {
  const newTodos = [];
  for (let i = 0; i < todos.length; i++) {
    if (todos[i].id === id) {
      newTodos.push({ ...todos[i], completed: !todos[i].completed });
    } else {
      newTodos.push(todos[i]);
    }
  }
  setTodos(newTodos);
};

// 리팩토링 후 (테스트가 통과하는지 확인)
const handleToggle = (id) => {
  setTodos(
    todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  );
};
```

---

## TDD의 장단점

### TDD의 장점

#### 1. 버그 감소

테스트를 먼저 작성하면 **엣지 케이스**를 미리 고려하게 됩니다.

```javascript
test('빈 문자열은 추가되지 않는다', () => { ... });
test('공백만 있는 문자열은 추가되지 않는다', () => { ... });
```

#### 2. 리팩토링 자신감

테스트가 있으면 **코드 변경 후에도 안전**합니다.

```javascript
// Custom Hook으로 리팩토링해도 테스트가 통과하는지 확인 가능
const { todos, addTodo, toggleTodo } = useTodos();
```

#### 3. 문서화 효과

테스트는 **살아있는 문서** 역할을 합니다.

```javascript
describe('TodoApp', () => {
  test('사용자가 새로운 Todo를 추가할 수 있다', () => { ... });
  test('빈 문자열은 추가되지 않는다', () => { ... });
  test('Todo를 클릭하면 완료 상태가 토글된다', () => { ... });
});
```

#### 4. 설계 개선

테스트를 먼저 작성하면 **API 인터페이스**를 미리 설계하게 됩니다.

```jsx
// 테스트를 작성하며 자연스럽게 props 인터페이스 설계
<TodoItem todo={todo} onToggle={handleToggle} onDelete={handleDelete} />
```

#### 5. 빠른 피드백

작은 단위로 테스트하면 **빠르게 피드백**을 받을 수 있습니다.

### TDD의 단점

#### 1. 초기 시간 투자

TDD는 초기에 **테스트 작성 시간**이 필요합니다.

```javascript
// 구현 코드 10줄에 테스트 코드 30줄이 필요할 수 있음
```

**해결책:** 장기적으로는 버그 수정 시간을 줄여 전체 개발 시간이 감소합니다.

#### 2. 학습 곡선

TDD는 **새로운 사고방식**을 요구합니다.

**해결책:** 작은 기능부터 시작하여 점진적으로 익숙해지세요.

#### 3. 과도한 테스트

**모든 것을 테스트**하려다 비효율적일 수 있습니다.

**해결책:** 핵심 비즈니스 로직에 집중하고, UI 스타일 등은 선택적으로 테스트하세요.

#### 4. 테스트 유지보수

요구사항이 변경되면 **테스트도 함께 수정**해야 합니다.

**해결책:** 구현 세부사항이 아닌 **동작(behavior)**을 테스트하면 유지보수가 쉬워집니다.

---

## TDD의 리듬

TDD는 **일정한 리듬**을 유지하는 것이 중요합니다.

### TDD 작업 흐름

```
1. 다음 기능 선택 (1-2분)
   ↓
2. Red: 실패하는 테스트 작성 (2-5분)
   ↓
3. Green: 최소한의 코드로 통과 (2-10분)
   ↓
4. Refactor: 코드 개선 (2-10분)
   ↓
5. 커밋 (1분)
   ↓
반복
```

### TDD 사이클 시간

- **Red 단계:** 2-5분 (테스트 작성)
- **Green 단계:** 2-10분 (구현)
- **Refactor 단계:** 2-10분 (개선)
- **전체 사이클:** 10-25분

> 💡 **팁:** 한 사이클이 30분 이상 걸린다면 기능을 더 작게 나누세요.

---

## 실무에서 TDD 적용하기

### 1. 점진적 도입

처음부터 모든 코드에 TDD를 적용하지 마세요.

**단계별 적용:**

1. **1주차:** 새로운 유틸리티 함수에 TDD 적용
2. **2-3주차:** 새로운 컴포넌트에 TDD 적용
3. **4주차 이후:** 핵심 비즈니스 로직에 TDD 적용

### 2. 팀과 함께하기

**페어 프로그래밍:**
- 한 명은 테스트 작성
- 다른 한 명은 구현 작성

**코드 리뷰:**
- 테스트가 요구사항을 잘 표현하는지 확인
- 테스트가 구현 세부사항에 의존하지 않는지 확인

### 3. 레거시 코드에 적용

기존 코드에 TDD를 적용하려면:

1. **특성화 테스트(Characterization Test)** 작성
2. 리팩토링하며 테스트 추가
3. 새로운 기능은 TDD로 개발

```javascript
// 레거시 코드의 현재 동작을 테스트로 고정
test('기존 Todo 추가 기능이 동작한다', () => {
  // 현재 동작을 테스트로 문서화
});
```

---

## 자주 묻는 질문 (FAQ)

### Q1. TDD를 하면 개발 속도가 느려지지 않나요?

**A:** 초기에는 느려질 수 있지만, 장기적으로는 **더 빨라집니다**.

- 버그 수정 시간 감소
- 리팩토링 자신감 증가
- 디버깅 시간 감소

### Q2. 모든 코드에 TDD를 적용해야 하나요?

**A:** 아닙니다. **핵심 비즈니스 로직**에 집중하세요.

**TDD 적용 우선순위:**

1. ✅ 비즈니스 로직 (useTodos, 유효성 검증)
2. ✅ 복잡한 알고리즘
3. ✅ 자주 변경되는 코드
4. ❌ 단순 UI 스타일링
5. ❌ 서드파티 라이브러리 래핑

### Q3. UI 컴포넌트도 TDD로 개발하나요?

**A:** **핵심 동작**은 TDD로, **스타일링**은 선택적으로 테스트하세요.

```javascript
// ✅ 동작 테스트 (TDD)
test('Todo를 클릭하면 완료 상태가 토글된다', () => { ... });

// ❌ 스타일 테스트 (선택적)
test('완료된 Todo는 취소선이 표시된다', () => { ... });
```

### Q4. 리팩토링 단계를 건너뛰면 안 되나요?

**A:** 리팩토링을 건너뛰면 **기술 부채**가 쌓입니다.

**리팩토링 타이밍:**

- 중복 코드 발견 시
- 함수가 20줄 이상일 때
- 하나의 함수가 여러 역할을 할 때

### Q5. 테스트가 실패하는데 어떻게 해야 하나요?

**A:** **테스트와 코드 중 하나만** 수정하세요.

**Red 단계에서 실패:**
- ✅ 정상 (구현이 없으므로)

**Green 단계에서 실패:**
- 구현 코드를 수정하여 테스트 통과

**Refactor 단계에서 실패:**
- 리팩토링 되돌리기 (테스트는 수정하지 않음)

### Q6. 통합 테스트도 TDD로 작성하나요?

**A:** 네, TDD는 **모든 수준의 테스트**에 적용 가능합니다.

```javascript
// 통합 테스트도 TDD로 작성 가능
test('Todo 추가부터 삭제까지 전체 흐름이 동작한다', async () => {
  // Red → Green → Refactor 사이클 적용
});
```

---

## 결론

TDD(테스트 주도 개발)는 단순히 테스트를 먼저 작성하는 것이 아니라, **더 나은 설계**와 **안전한 리팩토링**을 가능하게 하는 개발 방법론입니다.

**TDD의 핵심 가치:**

1. ✅ **버그 감소** - 엣지 케이스를 미리 고려
2. ✅ **리팩토링 자신감** - 테스트가 안전망 역할
3. ✅ **설계 개선** - API 인터페이스를 먼저 설계
4. ✅ **빠른 피드백** - 작은 단위로 빠르게 검증
5. ✅ **문서화** - 테스트가 살아있는 문서

**TDD 시작하기:**

1. 작은 유틸리티 함수부터 시작
2. Red-Green-Refactor 사이클 익히기
3. 점진적으로 적용 범위 확대
4. 팀과 함께 학습하고 개선

TDD는 초기 학습 곡선이 있지만, **장기적으로 더 빠르고 안전한 개발**을 가능하게 합니다. 오늘 당장 작은 함수 하나부터 TDD를 시작해보세요!

---

**다음 글 예고:**

다음 포스트에서는 **E2E 테스트 실전 가이드**를 다룰 예정입니다. Playwright와 Cypress를 활용한 실전 E2E 테스트 작성법을 살펴보겠습니다.

**피드백 환영:**

이 글이 도움이 되셨나요? 댓글로 피드백을 남겨주시면 더 좋은 콘텐츠로 보답하겠습니다! 🚀

---

## 참고 자료

### 공식 문서

- [Jest 공식 문서](https://jestjs.io/)
- [React Testing Library 공식 문서](https://testing-library.com/react)
- [Testing Library 쿼리 우선순위](https://testing-library.com/docs/queries/about#priority)

### TDD 관련 서적

- **"테스트 주도 개발" (Kent Beck)** - TDD의 바이블
- **"클린 코드" (Robert C. Martin)** - TDD와 클린 코드
- **"리팩토링" (Martin Fowler)** - 안전한 리팩토링 기법

### 관련 아티클

- [프론트엔드 테스팅 완벽 가이드](https://changsu1993.github.io/posts/frontend-testing-guide/)
- [Jest와 React Testing Library로 시작하는 단위 테스트](https://jestjs.io/docs/tutorial-react)

### 온라인 강의

- [Test-Driven Development (Udemy)](https://www.udemy.com/topic/test-driven-development/)
- [Testing JavaScript (Kent C. Dodds)](https://testingjavascript.com/)

### 실습 자료

- [TDD Kata 연습 문제](https://github.com/garora/TDD-Katas)
- [React Testing Examples](https://github.com/testing-library/react-testing-library/tree/main/examples)
