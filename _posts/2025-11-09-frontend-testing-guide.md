---
title: 프론트엔드 테스트 완벽 가이드 - Jest와 React Testing Library로 시작하기
description: 프론트엔드 개발에서 테스트 코드의 필요성부터 Jest, React Testing Library를 활용한 실전 테스트 작성까지. Unit 테스트, Integration 테스트, E2E 테스트의 차이와 좋은 테스트 코드 작성 팁을 실제 예제와 함께 완벽하게 정리합니다.
author: changsu
date: 2025-11-09 15:00:00 +0900
categories: [개발, Testing]
tags: [testing, react]
toc: true
comments: true
---

## 들어가며

"테스트 코드는 시간 낭비다"라고 생각하시나요? 실제로 많은 개발자들이 빠듯한 일정 속에서 테스트 코드 작성을 미루거나 생략합니다.

하지만 **테스트 코드는 버그를 조기에 발견하고, 리팩토링을 안전하게 하며, 코드 품질을 향상시키는 필수 도구**입니다.

이 글에서는 프론트엔드 테스트의 기초부터 실전 활용까지 모든 것을 다룹니다.

> 테스트 코드 작성에 투자한 시간은 디버깅 시간을 극적으로 줄여줍니다.
{: .prompt-tip }

## 왜 테스트 코드가 필요한가?

### 테스트 코드의 이점

**1. 버그 조기 발견**

```javascript
// 테스트 없이 개발
function calculateDiscount(price, discountRate) {
  return price * discountRate; // 버그: 할인율이 아니라 할인 금액을 곱함
}

// 프로덕션에서 발견 → 고객 불만, 매출 손실
```

```javascript
// 테스트 코드로 개발
describe('calculateDiscount', () => {
  test('10% 할인이 올바르게 적용되어야 함', () => {
    expect(calculateDiscount(1000, 0.1)).toBe(900); // ❌ 실패: 100이 반환됨
  });
});

// 개발 단계에서 발견 → 즉시 수정
function calculateDiscount(price, discountRate) {
  return price * (1 - discountRate); // ✅ 수정
}
```

**2. 안전한 리팩토링**

```javascript
// 리팩토링 전
function validateEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// 테스트 코드 작성
test('유효한 이메일을 검증해야 함', () => {
  expect(validateEmail('test@example.com')).toBe(true);
  expect(validateEmail('invalid-email')).toBe(false);
  expect(validateEmail('test@')).toBe(false);
});

// 리팩토링 후 - 테스트가 기존 동작 보장
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

**3. 문서화 역할**

```javascript
describe('ShoppingCart', () => {
  test('상품을 추가할 수 있어야 함', () => {
    const cart = new ShoppingCart();
    cart.addItem({ id: 1, name: 'Book', price: 10000 });
    expect(cart.items).toHaveLength(1);
  });

  test('동일한 상품은 수량만 증가해야 함', () => {
    const cart = new ShoppingCart();
    cart.addItem({ id: 1, name: 'Book', price: 10000 });
    cart.addItem({ id: 1, name: 'Book', price: 10000 });
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].quantity).toBe(2);
  });
});

// 테스트 코드가 ShoppingCart의 동작을 명확하게 설명
```

**4. 개발 속도 향상**

```
테스트 없는 개발:
코딩 → 수동 테스트 → 버그 발견 → 수정 → 다시 수동 테스트 → ...
(시간: 2-3시간)

테스트 코드와 함께 개발:
코딩 → 자동 테스트 → 버그 발견 → 수정 → 자동 테스트 → 완료
(시간: 30분)
```

## 테스트의 종류

### 테스트 피라미드

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="display: flex; flex-direction: column; align-items: center; gap: 0;">
    <!-- E2E Tests -->
    <div style="background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%); color: white; padding: 15px 60px; border-radius: 10px 10px 0 0; text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1); width: 200px;">
      <div style="font-weight: bold; font-size: 14px; margin-bottom: 5px;">E2E Tests</div>
      <div style="font-size: 12px; opacity: 0.9;">느림, 비용 높음</div>
    </div>

    <!-- Integration Tests -->
    <div style="background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%); color: white; padding: 20px 80px; text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1); width: 300px;">
      <div style="font-weight: bold; font-size: 15px; margin-bottom: 5px;">Integration Tests</div>
      <div style="font-size: 13px; opacity: 0.9;">보통 속도, 적절한 비용</div>
    </div>

    <!-- Unit Tests -->
    <div style="background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%); color: white; padding: 25px 100px; border-radius: 0 0 10px 10px; text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1); width: 400px;">
      <div style="font-weight: bold; font-size: 16px; margin-bottom: 5px;">Unit Tests</div>
      <div style="font-size: 14px; opacity: 0.9;">빠름, 비용 낮음, 많은 수</div>
    </div>
  </div>
</div>

### 1. Unit Test (단위 테스트)

**개별 함수나 컴포넌트를 독립적으로 테스트**

```javascript
// utils/math.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

// utils/math.test.js
import { add, multiply } from './math';

describe('Math Utils', () => {
  describe('add', () => {
    test('두 숫자를 더해야 함', () => {
      expect(add(2, 3)).toBe(5);
    });

    test('음수를 처리해야 함', () => {
      expect(add(-5, 3)).toBe(-2);
    });

    test('0을 처리해야 함', () => {
      expect(add(0, 0)).toBe(0);
    });
  });

  describe('multiply', () => {
    test('두 숫자를 곱해야 함', () => {
      expect(multiply(3, 4)).toBe(12);
    });

    test('0을 곱하면 0이어야 함', () => {
      expect(multiply(5, 0)).toBe(0);
    });
  });
});
```

**React 컴포넌트 Unit Test:**

```jsx
// components/Button.jsx
export function Button({ children, onClick, disabled }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// components/Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  test('텍스트를 렌더링해야 함', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  test('클릭 이벤트를 처리해야 함', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);

    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('비활성화 상태를 처리해야 함', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick} disabled>Click</Button>);

    const button = screen.getByText('Click');
    expect(button).toBeDisabled();

    fireEvent.click(button);
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

### 2. Integration Test (통합 테스트)

**여러 모듈이 함께 동작하는지 테스트**

```jsx
// components/TodoList.jsx
import { useState } from 'react';

export function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input, completed: false }]);
      setInput('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="할 일 입력"
      />
      <button onClick={addTodo}>추가</button>

      <ul>
        {todos.map(todo => (
          <li
            key={todo.id}
            onClick={() => toggleTodo(todo.id)}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}

// components/TodoList.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { TodoList } from './TodoList';

describe('TodoList Integration', () => {
  test('할 일을 추가하고 완료 처리할 수 있어야 함', () => {
    render(<TodoList />);

    // 1. 할 일 추가
    const input = screen.getByPlaceholderText('할 일 입력');
    const addButton = screen.getByText('추가');

    fireEvent.change(input, { target: { value: '테스트 작성하기' } });
    fireEvent.click(addButton);

    // 2. 추가된 할 일 확인
    const todoItem = screen.getByText('테스트 작성하기');
    expect(todoItem).toBeInTheDocument();
    expect(todoItem).toHaveStyle({ textDecoration: 'none' });

    // 3. 완료 처리
    fireEvent.click(todoItem);
    expect(todoItem).toHaveStyle({ textDecoration: 'line-through' });

    // 4. 다시 클릭하면 미완료로 변경
    fireEvent.click(todoItem);
    expect(todoItem).toHaveStyle({ textDecoration: 'none' });
  });

  test('빈 입력은 추가되지 않아야 함', () => {
    render(<TodoList />);

    const addButton = screen.getByText('추가');
    fireEvent.click(addButton);

    expect(screen.queryByRole('listitem')).not.toBeInTheDocument();
  });
});
```

### 3. E2E Test (End-to-End 테스트)

**실제 사용자 시나리오를 시뮬레이션**

```javascript
// e2e/shopping.spec.js (Playwright 예시)
import { test, expect } from '@playwright/test';

test.describe('쇼핑몰 구매 프로세스', () => {
  test('상품 검색부터 결제까지 전체 플로우', async ({ page }) => {
    // 1. 홈페이지 방문
    await page.goto('https://example-shop.com');

    // 2. 상품 검색
    await page.fill('input[name="search"]', 'JavaScript 책');
    await page.click('button[type="submit"]');

    // 3. 검색 결과 확인
    await expect(page.locator('.product-list')).toBeVisible();

    // 4. 첫 번째 상품 클릭
    await page.click('.product-item:first-child');

    // 5. 상품 상세 페이지 확인
    await expect(page.locator('h1')).toContainText('JavaScript');

    // 6. 장바구니 담기
    await page.click('button:has-text("장바구니 담기")');
    await expect(page.locator('.cart-count')).toHaveText('1');

    // 7. 장바구니로 이동
    await page.click('a:has-text("장바구니")');

    // 8. 결제 페이지로 이동
    await page.click('button:has-text("구매하기")');

    // 9. 결제 정보 입력
    await page.fill('input[name="name"]', '홍길동');
    await page.fill('input[name="phone"]', '010-1234-5678');
    await page.fill('input[name="address"]', '서울시 강남구');

    // 10. 결제 완료
    await page.click('button:has-text("결제하기")');

    // 11. 주문 완료 페이지 확인
    await expect(page.locator('h1')).toContainText('주문이 완료되었습니다');
  });
});
```

## Jest 시작하기

### 설치

```bash
# Jest 설치
npm install --save-dev jest

# Babel 지원 (ES6+ 문법 사용 시)
npm install --save-dev @babel/preset-env

# React Testing Library (React 프로젝트)
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

### 설정

**package.json 설정:**

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "jsdom",
    "setupFilesAfterEnv": ["<rootDir>/jest.setup.js"]
  }
}
```

```javascript
// jest.setup.js
import '@testing-library/jest-dom';
```

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: { node: 'current' } }],
    '@babel/preset-react'
  ]
};
```

## React Testing Library 핵심 개념

### 쿼리 우선순위

**1. Accessible by Everyone (최우선)**

```jsx
// ✅ 좋은 예: getByRole
test('버튼을 클릭할 수 있어야 함', () => {
  render(<button>제출</button>);
  const button = screen.getByRole('button', { name: '제출' });
  fireEvent.click(button);
});

// ✅ 좋은 예: getByLabelText (폼 요소)
test('이메일 입력이 가능해야 함', () => {
  render(
    <>
      <label htmlFor="email">이메일</label>
      <input id="email" type="email" />
    </>
  );
  const input = screen.getByLabelText('이메일');
  fireEvent.change(input, { target: { value: 'test@example.com' } });
});
```

**2. Semantic Queries**

```jsx
// ✅ getByPlaceholderText
test('검색어를 입력할 수 있어야 함', () => {
  render(<input placeholder="검색어 입력" />);
  const input = screen.getByPlaceholderText('검색어 입력');
});

// ✅ getByText
test('에러 메시지가 표시되어야 함', () => {
  render(<div>비밀번호가 일치하지 않습니다</div>);
  expect(screen.getByText('비밀번호가 일치하지 않습니다')).toBeInTheDocument();
});
```

**3. Test IDs (최후의 수단)**

```jsx
// ❌ 피해야 할 방법: getByTestId
test('결과가 표시되어야 함', () => {
  render(<div data-testid="result">100</div>);
  expect(screen.getByTestId('result')).toHaveTextContent('100');
});

// ✅ 대신 이렇게
test('결과가 표시되어야 함', () => {
  render(<div role="status">100</div>);
  expect(screen.getByRole('status')).toHaveTextContent('100');
});
```

### 비동기 테스트

```jsx
// components/UserProfile.jsx
import { useState, useEffect } from 'react';

export function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>로딩중...</div>;
  if (error) return <div>에러: {error}</div>;
  return <div>{user.name}</div>;
}

// components/UserProfile.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock fetch
global.fetch = jest.fn();

describe('UserProfile', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('사용자 정보를 불러와 표시해야 함', async () => {
    // Mock 응답 설정
    fetch.mockResolvedValueOnce({
      json: async () => ({ id: 1, name: '홍길동' })
    });

    render(<UserProfile userId={1} />);

    // 로딩 상태 확인
    expect(screen.getByText('로딩중...')).toBeInTheDocument();

    // 사용자 정보가 표시될 때까지 대기
    await waitFor(() => {
      expect(screen.getByText('홍길동')).toBeInTheDocument();
    });

    // API가 올바르게 호출되었는지 확인
    expect(fetch).toHaveBeenCalledWith('/api/users/1');
    expect(fetch).toHaveBeenCalledTimes(1);
  });

  test('에러 발생 시 에러 메시지를 표시해야 함', async () => {
    fetch.mockRejectedValueOnce(new Error('Network Error'));

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText('에러: Network Error')).toBeInTheDocument();
    });
  });
});
```

## 좋은 테스트 코드 작성 팁

### 1. AAA 패턴 따르기

```javascript
test('할 일을 추가할 수 있어야 함', () => {
  // Arrange (준비): 테스트에 필요한 환경 설정
  render(<TodoList />);
  const input = screen.getByPlaceholderText('할 일 입력');
  const button = screen.getByText('추가');

  // Act (실행): 테스트할 동작 수행
  fireEvent.change(input, { target: { value: '테스트 작성' } });
  fireEvent.click(button);

  // Assert (검증): 결과 확인
  expect(screen.getByText('테스트 작성')).toBeInTheDocument();
});
```

### 2. 한 가지만 테스트하기

```javascript
// ❌ 나쁜 예: 여러 가지를 한 번에 테스트
test('폼이 동작해야 함', () => {
  render(<SignupForm />);
  // 이메일 검증
  // 비밀번호 검증
  // 제출 동작
  // 에러 처리
});

// ✅ 좋은 예: 각각 분리
test('유효하지 않은 이메일은 에러를 표시해야 함', () => {
  render(<SignupForm />);
  fireEvent.change(screen.getByLabelText('이메일'), {
    target: { value: 'invalid' }
  });
  expect(screen.getByText('올바른 이메일을 입력하세요')).toBeInTheDocument();
});

test('비밀번호가 8자 미만이면 에러를 표시해야 함', () => {
  render(<SignupForm />);
  fireEvent.change(screen.getByLabelText('비밀번호'), {
    target: { value: '123' }
  });
  expect(screen.getByText('비밀번호는 8자 이상이어야 합니다')).toBeInTheDocument();
});
```

### 3. 구현이 아닌 동작 테스트하기

```javascript
// ❌ 나쁜 예: 구현 세부사항 테스트
test('useState를 사용해야 함', () => {
  const { result } = renderHook(() => useState(0));
  expect(result.current).toBeDefined();
});

// ✅ 좋은 예: 사용자 관점에서 테스트
test('버튼을 클릭하면 카운트가 증가해야 함', () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: '+' });
  const count = screen.getByText('0');

  fireEvent.click(button);
  expect(count).toHaveTextContent('1');
});
```

### 4. Mock은 최소화하기

```javascript
// ❌ 나쁜 예: 과도한 Mock
test('사용자 목록을 렌더링해야 함', () => {
  const mockUsers = [{ id: 1, name: 'Test' }];
  const mockFetch = jest.fn().mockResolvedValue(mockUsers);
  const mockUseEffect = jest.fn();
  const mockUseState = jest.fn();
  // ...
});

// ✅ 좋은 예: 필요한 부분만 Mock
test('사용자 목록을 렌더링해야 함', async () => {
  global.fetch = jest.fn().mockResolvedValue({
    json: async () => [{ id: 1, name: 'Test' }]
  });

  render(<UserList />);
  await waitFor(() => {
    expect(screen.getByText('Test')).toBeInTheDocument();
  });
});
```

### 5. 의미있는 테스트 이름 짓기

```javascript
// ❌ 나쁜 예
test('test1', () => {});
test('it works', () => {});
test('button', () => {});

// ✅ 좋은 예
test('빈 이메일 입력 시 에러 메시지를 표시해야 함', () => {});
test('로그인 성공 시 대시보드로 리다이렉트해야 함', () => {});
test('삭제 버튼 클릭 시 확인 다이얼로그가 나타나야 함', () => {});
```

## 테스트 커버리지

### 커버리지 확인

```bash
npm run test:coverage
```

```
------------------------|---------|----------|---------|---------|-------------------
File                    | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
------------------------|---------|----------|---------|---------|-------------------
All files               |   85.71 |    66.67 |      80 |   85.71 |
 components             |   85.71 |    66.67 |      80 |   85.71 |
  Button.jsx            |     100 |      100 |     100 |     100 |
  TodoList.jsx          |   83.33 |       50 |   66.67 |   83.33 | 15,23
  UserProfile.jsx       |      80 |      100 |      75 |      80 | 18
------------------------|---------|----------|---------|---------|-------------------
```

### 커버리지 목표 설정

**package.json에 최소 커버리지 기준 설정:**

```json
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

> **주의**: 100% 커버리지가 좋은 테스트를 의미하지는 않습니다. 중요한 것은 의미있는 테스트를 작성하는 것입니다.
{: .prompt-warning }

## CI/CD 통합

### GitHub Actions 예시

```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test -- --coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/coverage-final.json
        fail_ci_if_error: true
```

## 마치며

테스트 코드는 처음에는 시간이 더 걸리는 것처럼 보일 수 있습니다. 하지만 장기적으로 보면:

- **디버깅 시간 감소**
- **리팩토링 안전성 확보**
- **코드 품질 향상**
- **문서화 효과**
- **개발 속도 증가**

등의 이점을 제공합니다.

**테스트 코드 작성의 핵심 원칙:**

1. **사용자 관점으로 테스트하기** - 구현이 아닌 동작을 테스트
2. **단순하게 유지하기** - 한 번에 한 가지만 테스트
3. **읽기 쉽게 작성하기** - 테스트는 문서 역할도 함
4. **적절한 수준의 테스트** - Unit, Integration, E2E 균형있게
5. **지속적으로 실행하기** - CI/CD 파이프라인에 통합

작은 유틸리티 함수부터 시작해서 점진적으로 테스트 커버리지를 늘려가세요. 완벽한 테스트보다 **의미있는 테스트**가 중요합니다!

## 참고 자료

### 공식 문서
- [Jest 공식 문서](https://jestjs.io/)
- [React Testing Library 공식 문서](https://testing-library.com/react)
- [Testing Library 쿼리 우선순위](https://testing-library.com/docs/queries/about#priority)
- [Playwright 공식 문서](https://playwright.dev/)

### 추천 학습 자료
- [Kent C. Dodds - Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Martin Fowler - Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [JavaScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)

### 도구 및 라이브러리
- [MSW (Mock Service Worker)](https://mswjs.io/) - API Mocking
- [Codecov](https://codecov.io/) - 테스트 커버리지 시각화
- [Testing Playground](https://testing-playground.com/) - 쿼리 연습
