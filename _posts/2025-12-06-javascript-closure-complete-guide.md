---
title: "JavaScript 클로저(Closure) 완벽 가이드 - 개념부터 실전 활용까지"
description: "JavaScript 클로저(Closure)의 동작 원리와 실전 활용법을 완벽 설명합니다. 렉시컬 스코프, 데이터 은닉, 커링, 메모이제이션 패턴과 React Hooks의 stale closure 문제 해결법까지 코드 예제와 함께 다룹니다."
date: 2025-12-06 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, closure, 클로저, lexical-scope, react-hooks, stale-closure, memoization, currying]
---

## 개요

**클로저(Closure)**는 JavaScript에서 가장 강력하면서도 많은 개발자들이 혼란스러워하는 개념 중 하나입니다. 클로저를 이해하면 데이터 은닉, 상태 유지, 함수형 프로그래밍 패턴 등 고급 JavaScript 기법을 자유롭게 활용할 수 있습니다.

이 글에서는 클로저의 정확한 정의와 동작 원리를 설명하고, 실무에서 활용할 수 있는 다양한 패턴을 코드 예제와 함께 다룹니다. 특히 React Hooks에서 발생하는 stale closure 문제와 해결법까지 깊이 있게 살펴보겠습니다. 클로저와 밀접한 관련이 있는 콜백 패턴은 [JavaScript Callback 완벽 가이드](/posts/javascript-callback/)를 참고하세요.

### 학습 목표

- 클로저의 정확한 정의와 생성 조건 이해
- 렉시컬 스코프와 클로저의 관계 파악
- 실전에서 활용 가능한 클로저 패턴 습득
- 클로저 관련 메모리 이슈 및 해결 방법 학습
- React Hooks의 stale closure 문제 이해 및 대응

### 사전 지식

- JavaScript 함수 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- Scope 개념 ([JavaScript Scope 완벽 가이드](/posts/javascript-scope/) 참고)
- ES6+ 문법 기초

---

## 클로저란 무엇인가?

### 공식적인 정의

MDN에서는 클로저를 다음과 같이 정의합니다:

> 클로저는 함수와 그 함수가 선언된 렉시컬 환경(Lexical Environment)의 조합이다.

이 정의를 쉽게 풀어보면, **클로저는 함수가 자신이 생성될 때의 환경(스코프)을 기억하고, 그 환경 밖에서 호출되더라도 해당 환경에 접근할 수 있는 현상**입니다.

### 가장 간단한 클로저 예제

```javascript
function outer() {
  const message = '안녕하세요'; // outer 함수의 지역 변수

  function inner() {
    console.log(message); // outer의 변수에 접근
  }

  return inner; // 내부 함수를 반환
}

const greeting = outer(); // outer 실행 후 inner 함수 반환
greeting(); // '안녕하세요' - outer는 이미 종료되었지만 message에 접근 가능!
```

**무슨 일이 일어났는가?**

1. `outer()` 함수가 호출되어 `message` 변수가 생성됩니다
2. `inner` 함수가 정의되고 반환됩니다
3. `outer()` 함수의 실행이 종료됩니다
4. 일반적으로 함수가 종료되면 지역 변수는 메모리에서 해제됩니다
5. 하지만 `greeting()`을 호출하면 여전히 `message`에 접근할 수 있습니다

이것이 클로저입니다. `inner` 함수가 자신이 생성될 때의 렉시컬 환경을 **기억**하고 있기 때문입니다.

---

## 렉시컬 스코프와 클로저

### 렉시컬 스코프란?

**렉시컬 스코프(Lexical Scope)**는 함수의 스코프가 **함수가 정의된 위치**에 의해 결정되는 것을 의미합니다. 함수가 어디서 호출되었는지는 중요하지 않습니다.

```javascript
const globalVar = '전역 변수';

function outer() {
  const outerVar = '외부 변수';

  function inner() {
    const innerVar = '내부 변수';

    // inner는 자신의 스코프, outer의 스코프, 전역 스코프에 모두 접근 가능
    console.log(innerVar);  // '내부 변수'
    console.log(outerVar);  // '외부 변수'
    console.log(globalVar); // '전역 변수'
  }

  return inner;
}

const fn = outer();
fn(); // 어디서 호출되든 렉시컬 스코프 기준으로 변수를 찾음
```

### 스코프 체인과 클로저

JavaScript 엔진은 변수를 찾을 때 **스코프 체인**을 따라 올라갑니다:

```javascript
function createMultiplier(multiplier) {
  // multiplier는 createMultiplier의 스코프에 존재

  return function(number) {
    // number는 반환된 함수의 스코프에 존재
    // multiplier는 외부 스코프에서 찾음 (클로저)
    return number * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

**스코프 체인 구조:**

```
전역 스코프
└── createMultiplier 스코프 (multiplier = 2 or 3)
    └── 반환된 함수 스코프 (number = 5)
```

각 `double`과 `triple` 함수는 서로 다른 클로저를 가지고 있으며, 각각 자신만의 `multiplier` 값을 기억합니다.

---

## 클로저가 생성되는 조건

클로저가 생성되려면 다음 조건이 충족되어야 합니다:

### 1. 내부 함수가 외부 함수의 변수를 참조

```javascript
function outer() {
  const x = 10;

  function inner() {
    console.log(x); // 외부 변수 참조 - 클로저 생성
  }

  return inner;
}
```

### 2. 내부 함수가 외부 함수의 스코프 밖에서 실행

```javascript
function outer() {
  const x = 10;

  function inner() {
    console.log(x);
  }

  return inner; // 함수를 반환하여 외부에서 실행 가능하게 함
}

const closureFn = outer();
closureFn(); // outer의 스코프 밖에서 실행
```

### 클로저가 아닌 경우

```javascript
function notClosure() {
  const x = 10;

  function inner() {
    console.log(x);
  }

  inner(); // 내부에서 바로 실행 - 기술적으로 클로저지만 실용적 의미 없음
}

notClosure();
```

> 엄밀히 말하면 JavaScript의 모든 함수는 자신이 정의된 스코프를 기억하므로 클로저입니다. 하지만 실무에서 "클로저"라고 할 때는 주로 함수가 자신의 렉시컬 스코프 밖에서 실행되면서 외부 변수에 접근하는 경우를 말합니다.
{: .prompt-info }

---

## 클로저의 실전 활용 패턴

### 1. 데이터 은닉 (Private 변수)

JavaScript에는 전통적으로 private 키워드가 없었습니다. 클로저를 사용하면 외부에서 직접 접근할 수 없는 private 변수를 구현할 수 있습니다.

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private 변수

  return {
    deposit(amount) {
      if (amount > 0) {
        balance += amount;
        console.log(`${amount}원 입금. 잔액: ${balance}원`);
      }
    },
    withdraw(amount) {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        console.log(`${amount}원 출금. 잔액: ${balance}원`);
        return amount;
      }
      console.log('출금 실패: 잔액 부족');
      return 0;
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount(10000);

account.deposit(5000);    // 5000원 입금. 잔액: 15000원
account.withdraw(3000);   // 3000원 출금. 잔액: 12000원
console.log(account.getBalance()); // 12000

// balance에 직접 접근 불가능!
console.log(account.balance); // undefined
```

**장점:**
- 외부에서 `balance`를 임의로 변경할 수 없음
- 반드시 제공된 메서드를 통해서만 상태 변경 가능
- 데이터 무결성 보장

### 2. 팩토리 함수와 모듈 패턴

여러 인스턴스를 생성하는 팩토리 함수:

```javascript
function createCounter(initialValue = 0, step = 1) {
  let count = initialValue;

  return {
    increment() {
      count += step;
      return count;
    },
    decrement() {
      count -= step;
      return count;
    },
    reset() {
      count = initialValue;
      return count;
    },
    getValue() {
      return count;
    }
  };
}

const counter1 = createCounter(0, 1);
const counter2 = createCounter(100, 5);

console.log(counter1.increment()); // 1
console.log(counter1.increment()); // 2
console.log(counter2.increment()); // 105
console.log(counter2.increment()); // 110

// 각 카운터는 독립적인 상태를 가짐
console.log(counter1.getValue()); // 2
console.log(counter2.getValue()); // 110
```

**IIFE(즉시 실행 함수)를 활용한 모듈 패턴:**

```javascript
const Calculator = (function() {
  // private 변수와 함수
  let history = [];

  function addToHistory(operation, result) {
    history.push({ operation, result, timestamp: new Date() });
  }

  // public API
  return {
    add(a, b) {
      const result = a + b;
      addToHistory(`${a} + ${b}`, result);
      return result;
    },
    subtract(a, b) {
      const result = a - b;
      addToHistory(`${a} - ${b}`, result);
      return result;
    },
    multiply(a, b) {
      const result = a * b;
      addToHistory(`${a} * ${b}`, result);
      return result;
    },
    getHistory() {
      return [...history]; // 복사본 반환으로 원본 보호
    },
    clearHistory() {
      history = [];
    }
  };
})();

Calculator.add(5, 3);      // 8
Calculator.multiply(4, 2); // 8
console.log(Calculator.getHistory());
// [{ operation: '5 + 3', result: 8, ... }, { operation: '4 * 2', result: 8, ... }]

// history에 직접 접근 불가
console.log(Calculator.history); // undefined
```

### 3. 커링(Currying)과 부분 적용

**커링**은 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수들의 체인으로 변환하는 기법입니다.

```javascript
// 일반 함수
function add(a, b, c) {
  return a + b + c;
}

// 커링된 버전
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

// 사용
console.log(add(1, 2, 3));           // 6
console.log(curriedAdd(1)(2)(3));    // 6

// 화살표 함수로 더 간결하게
const curriedAddArrow = a => b => c => a + b + c;
console.log(curriedAddArrow(1)(2)(3)); // 6
```

**부분 적용 예시:**

```javascript
// 범용 커링 함수
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

// 사용 예시
function formatMessage(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const curriedFormat = curry(formatMessage);

// 부분 적용
const greetHello = curriedFormat('안녕하세요');
const greetHelloFormal = greetHello('님');

console.log(greetHelloFormal('!'));  // '안녕하세요, 님!'
console.log(curriedFormat('Hello')('World')('!')); // 'Hello, World!'
```

**실무 활용 - API 요청 설정:**

```javascript
const createApiRequest = baseUrl => endpoint => options => {
  return fetch(`${baseUrl}${endpoint}`, {
    headers: {
      'Content-Type': 'application/json',
    },
    ...options
  });
};

// 기본 API 설정
const api = createApiRequest('https://api.example.com');

// 엔드포인트별 함수 생성
const getUsers = api('/users');
const getPosts = api('/posts');

// 사용
getUsers({ method: 'GET' }).then(res => res.json());
getPosts({ method: 'GET' }).then(res => res.json());
```

### 4. 메모이제이션(Memoization)

메모이제이션은 함수의 결과를 캐싱하여 동일한 입력에 대해 재계산을 방지하는 최적화 기법입니다.

```javascript
function memoize(fn) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('캐시에서 반환:', key);
      return cache.get(key);
    }

    console.log('새로 계산:', key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// 피보나치 (비효율적인 재귀 버전)
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// 메모이제이션 적용
const memoizedFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return memoizedFib(n - 1) + memoizedFib(n - 2);
});

console.log(memoizedFib(10)); // 55
console.log(memoizedFib(10)); // 캐시에서 즉시 반환

// 성능 비교
console.time('일반');
fibonacci(35);
console.timeEnd('일반'); // 수백 ms

console.time('메모이제이션');
memoizedFib(35);
console.timeEnd('메모이제이션'); // 수 ms
```

**실무 활용 - 복잡한 계산 캐싱:**

```javascript
const memoizedExpensiveCalculation = memoize(function(data, options) {
  // 복잡한 데이터 처리
  console.log('복잡한 계산 수행 중...');

  return data
    .filter(item => item.active)
    .map(item => ({
      ...item,
      calculated: item.value * options.multiplier
    }))
    .sort((a, b) => b.calculated - a.calculated);
});

const data = [
  { id: 1, value: 100, active: true },
  { id: 2, value: 200, active: true },
  { id: 3, value: 150, active: false }
];

// 첫 번째 호출 - 계산 수행
const result1 = memoizedExpensiveCalculation(data, { multiplier: 2 });

// 두 번째 호출 - 캐시에서 반환
const result2 = memoizedExpensiveCalculation(data, { multiplier: 2 });
```

### 5. 이벤트 핸들러에서의 클로저

```javascript
function setupButtons() {
  const buttons = document.querySelectorAll('.action-button');

  buttons.forEach((button, index) => {
    // 클로저를 통해 각 버튼이 자신의 index를 기억
    button.addEventListener('click', function() {
      console.log(`버튼 ${index + 1} 클릭됨`);
      handleButtonClick(index);
    });
  });
}

function handleButtonClick(buttonIndex) {
  console.log(`버튼 ${buttonIndex + 1} 처리 중...`);
}
```

**상태를 기억하는 이벤트 핸들러:**

```javascript
function createToggleHandler(element, onClass, offClass) {
  let isOn = false;

  return function() {
    isOn = !isOn;

    if (isOn) {
      element.classList.remove(offClass);
      element.classList.add(onClass);
    } else {
      element.classList.remove(onClass);
      element.classList.add(offClass);
    }

    return isOn;
  };
}

const button = document.querySelector('#toggleBtn');
const handleToggle = createToggleHandler(button, 'active', 'inactive');

button.addEventListener('click', () => {
  const state = handleToggle();
  console.log(`현재 상태: ${state ? 'ON' : 'OFF'}`);
});
```

---

## 반복문에서의 클로저 문제

### 전통적인 문제 상황 (var 사용 시)

```javascript
// 문제가 있는 코드
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}

// 예상: 0, 1, 2
// 실제: 3, 3, 3
```

**왜 이런 일이 발생하는가?**

1. `var`는 함수 스코프이므로 루프 전체에서 하나의 `i` 변수만 존재
2. `setTimeout`의 콜백은 비동기로 실행
3. 콜백이 실행될 때 루프는 이미 종료되어 `i`는 3
4. 모든 콜백이 같은 `i`를 참조하므로 모두 3 출력

### 해결 방법 1: IIFE 사용

```javascript
for (var i = 0; i < 3; i++) {
  (function(capturedI) {
    setTimeout(function() {
      console.log(capturedI);
    }, 1000);
  })(i);
}

// 출력: 0, 1, 2
```

IIFE가 즉시 실행되면서 현재 `i` 값을 `capturedI`로 복사하여 클로저에 캡처합니다.

### 해결 방법 2: let 사용 (권장)

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}

// 출력: 0, 1, 2
```

`let`은 블록 스코프이므로 각 반복마다 새로운 `i` 바인딩이 생성됩니다.

### 해결 방법 3: forEach 사용

```javascript
[0, 1, 2].forEach(function(i) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
});

// 출력: 0, 1, 2
```

### 실제 활용 예시

```javascript
// 버튼에 이벤트 핸들러 등록 - 잘못된 방법
function setupButtonsWrong() {
  const buttons = document.querySelectorAll('button');

  for (var i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
      alert('버튼 ' + i + ' 클릭');  // 항상 마지막 인덱스 출력
    });
  }
}

// 올바른 방법 1: let 사용
function setupButtonsCorrect1() {
  const buttons = document.querySelectorAll('button');

  for (let i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
      alert('버튼 ' + i + ' 클릭');  // 올바른 인덱스 출력
    });
  }
}

// 올바른 방법 2: forEach 사용
function setupButtonsCorrect2() {
  const buttons = document.querySelectorAll('button');

  buttons.forEach((button, i) => {
    button.addEventListener('click', () => {
      alert('버튼 ' + i + ' 클릭');
    });
  });
}
```

> `var` 대신 `let`이나 `const`를 사용하는 것이 현대 JavaScript의 모범 사례입니다. 이는 클로저 관련 버그를 예방하고 코드의 의도를 명확하게 합니다.
{: .prompt-tip }

---

## 클로저와 메모리

### 클로저의 메모리 유지

클로저는 외부 함수의 변수를 참조하기 때문에, 해당 변수는 가비지 컬렉션되지 않고 메모리에 유지됩니다.

```javascript
function createHeavyObject() {
  const largeData = new Array(1000000).fill('data'); // 큰 배열

  return function() {
    console.log(largeData.length); // largeData 참조
  };
}

const closure = createHeavyObject();
// largeData는 closure가 존재하는 한 메모리에 유지됨
```

### 메모리 누수 방지

**1. 필요 없는 참조 해제:**

```javascript
function createHandler() {
  const element = document.getElementById('target');
  const data = fetchLargeData();

  element.addEventListener('click', function handler() {
    console.log(data.summary); // data 전체가 아닌 필요한 것만 참조
  });

  // 이벤트 제거 시 클로저도 해제
  return function cleanup() {
    element.removeEventListener('click', handler);
  };
}

const cleanup = createHandler();
// 나중에 정리
cleanup();
```

**2. 필요한 데이터만 캡처:**

```javascript
// 나쁜 예 - 전체 객체 캡처
function badExample() {
  const hugeObject = {
    data: new Array(1000000).fill('x'),
    id: 123,
    name: 'example'
  };

  return function() {
    console.log(hugeObject.id); // id만 필요하지만 전체 객체 참조
  };
}

// 좋은 예 - 필요한 것만 캡처
function goodExample() {
  const hugeObject = {
    data: new Array(1000000).fill('x'),
    id: 123,
    name: 'example'
  };

  const { id } = hugeObject; // 필요한 것만 추출

  return function() {
    console.log(id); // id만 참조, hugeObject는 가비지 컬렉션 가능
  };
}
```

**3. WeakMap을 활용한 메모리 관리:**

```javascript
const cache = new WeakMap();

function memoizeWithWeakMap(fn) {
  return function(obj) {
    if (cache.has(obj)) {
      return cache.get(obj);
    }

    const result = fn(obj);
    cache.set(obj, result);
    return result;
  };
}

const expensiveOperation = memoizeWithWeakMap((obj) => {
  console.log('계산 중...');
  return obj.value * 2;
});

let obj = { value: 21 };
console.log(expensiveOperation(obj)); // 계산 중... 42
console.log(expensiveOperation(obj)); // 42 (캐시)

obj = null; // 객체 참조 해제하면 WeakMap 엔트리도 자동 제거
```

---

## React Hooks와 클로저: Stale Closure 문제

> React Hooks 성능 최적화에 대한 자세한 내용은 [React Hooks 성능 최적화 가이드](/posts/react-hooks-performance-optimization/)를 참고하세요.
{: .prompt-tip }

React Hooks는 클로저를 활용하여 컴포넌트 상태를 관리합니다. 하지만 이로 인해 **stale closure(오래된 클로저)** 문제가 발생할 수 있습니다.

### Stale Closure란?

Stale closure는 클로저가 오래된(이전 렌더링의) 값을 참조하는 현상입니다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // 이 클로저는 count의 초기값(0)만 기억함!
      console.log('현재 count:', count);
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // 빈 의존성 배열

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>증가</button>
    </div>
  );
}
```

**문제점:**
- 버튼을 클릭해도 콘솔에는 항상 0이 출력됨
- `useEffect`의 콜백이 최초 렌더링 시의 `count`(0)를 클로저로 캡처
- 의존성 배열이 비어있어 effect가 재실행되지 않음

### 해결 방법 1: 의존성 배열에 값 추가

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('현재 count:', count);
    }, 1000);

    return () => clearInterval(intervalId);
  }, [count]); // count를 의존성에 추가

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>증가</button>
    </div>
  );
}
```

> 이 방법은 `count`가 변경될 때마다 interval이 재설정됩니다. 정확한 타이밍이 중요한 경우 다른 방법을 고려하세요.
{: .prompt-warning }

### 해결 방법 2: useRef 활용

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  // count가 변경될 때마다 ref 업데이트
  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // ref.current는 항상 최신 값
      console.log('현재 count:', countRef.current);
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // 의존성 배열 비어있어도 OK

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>증가</button>
    </div>
  );
}
```

### 해결 방법 3: 함수형 업데이트 사용

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // 함수형 업데이트로 최신 상태 기반으로 업데이트
      setCount(prevCount => {
        console.log('현재 count:', prevCount);
        return prevCount + 1;
      });
    }, 1000);

    return () => clearInterval(intervalId);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
    </div>
  );
}
```

### 실전 예시: 이벤트 핸들러의 Stale Closure

```tsx
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // 문제가 있는 코드
  const handleSearchBad = useCallback(() => {
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, []); // query가 의존성에 없음 - stale closure!

  // 올바른 코드
  const handleSearchGood = useCallback(() => {
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, [query]); // query를 의존성에 포함

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button onClick={handleSearchGood}>검색</button>
    </div>
  );
}
```

### 커스텀 훅에서의 Stale Closure 방지

```tsx
// useLatest: 항상 최신 값을 참조하는 ref 반환
function useLatest<T>(value: T) {
  const ref = useRef(value);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref;
}

// 사용 예시
function ChatRoom({ roomId, onMessage }) {
  const latestOnMessage = useLatest(onMessage);

  useEffect(() => {
    const connection = createConnection(roomId);

    connection.on('message', (msg) => {
      // 항상 최신 onMessage 콜백 사용
      latestOnMessage.current(msg);
    });

    return () => connection.disconnect();
  }, [roomId]); // onMessage는 의존성에서 제외 가능
}
```

---

## 면접 단골 질문과 답변

### Q1. 클로저란 무엇인가요?

**A:** 클로저는 함수와 그 함수가 선언된 렉시컬 환경의 조합입니다. 내부 함수가 외부 함수의 변수에 접근할 수 있으며, 외부 함수가 종료된 후에도 해당 변수를 기억하고 접근할 수 있습니다.

```javascript
function outer() {
  const secret = '비밀';
  return function inner() {
    return secret;
  };
}

const getSecret = outer();
console.log(getSecret()); // '비밀'
```

### Q2. 클로저는 언제 사용하나요?

**A:** 주요 사용 사례:
1. **데이터 은닉**: private 변수 구현
2. **상태 유지**: 함수 호출 간 상태 보존
3. **팩토리 함수**: 설정이 다른 함수 생성
4. **콜백과 이벤트 핸들러**: 컨텍스트 정보 유지
5. **커링과 부분 적용**: 함수형 프로그래밍 패턴
6. **메모이제이션**: 계산 결과 캐싱

### Q3. 클로저의 단점은 무엇인가요?

**A:**
1. **메모리 사용**: 외부 변수가 가비지 컬렉션되지 않아 메모리 사용량 증가
2. **성능**: 스코프 체인 탐색으로 인한 약간의 성능 오버헤드
3. **디버깅 어려움**: 변수의 실제 값을 추적하기 어려울 수 있음
4. **의도치 않은 참조**: 잘못 사용하면 예상치 못한 동작 발생

**해결책:**
- 필요 없어진 참조는 명시적으로 해제
- 필요한 값만 클로저에 캡처
- 코드 리뷰와 테스트로 의도 검증

### Q4. 다음 코드의 출력 결과와 이유를 설명하세요.

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**A:** 출력은 `3, 3, 3`입니다.

**이유:**
1. `var`는 함수 스코프이므로 전체 루프에서 하나의 `i`만 존재
2. `setTimeout` 콜백은 이벤트 루프에 의해 나중에 실행
3. 콜백 실행 시점에 루프는 이미 종료되어 `i`는 3
4. 세 콜백 모두 같은 `i`를 참조

**해결:**
```javascript
// 방법 1: let 사용
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// 방법 2: IIFE
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 0))(i);
}
```

### Q5. 클로저를 사용해 카운터를 구현하세요.

**A:**
```javascript
function createCounter() {
  let count = 0;

  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; },
    reset() { count = 0; return count; }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount());  // 1
```

### Q6. React에서 stale closure 문제란 무엇이고 어떻게 해결하나요?

**A:** Stale closure는 React 컴포넌트에서 클로저가 이전 렌더링의 상태값을 캡처하여 최신 상태에 접근하지 못하는 문제입니다.

**해결 방법:**
1. `useEffect`/`useCallback` 의존성 배열에 필요한 값 추가
2. `useRef`로 최신 값 참조
3. 함수형 업데이트 사용 (`setState(prev => ...)`)

```tsx
// 문제
useEffect(() => {
  console.log(count); // 항상 초기값
}, []);

// 해결 1: 의존성 추가
useEffect(() => {
  console.log(count);
}, [count]);

// 해결 2: useRef 사용
const countRef = useRef(count);
useEffect(() => { countRef.current = count; }, [count]);
```

---

## 핵심 정리

### 클로저의 핵심 개념

| 개념 | 설명 |
|------|------|
| **정의** | 함수와 렉시컬 환경의 조합 |
| **생성 조건** | 내부 함수가 외부 변수를 참조하고 외부에서 실행될 때 |
| **렉시컬 스코프** | 함수 정의 위치에 따라 스코프 결정 |
| **메모리** | 참조된 변수는 가비지 컬렉션되지 않음 |

### 주요 활용 패턴

| 패턴 | 용도 | 예시 |
|------|------|------|
| **데이터 은닉** | Private 변수 구현 | 은행 계좌, 카운터 |
| **팩토리 함수** | 설정이 다른 함수 생성 | 이벤트 핸들러 생성기 |
| **커링** | 함수 부분 적용 | API 요청 설정 |
| **메모이제이션** | 계산 결과 캐싱 | 피보나치, 복잡한 연산 |
| **모듈 패턴** | 캡슐화된 모듈 생성 | Calculator 모듈 |

### 주의사항

1. **var vs let**: 반복문에서는 `let` 사용 권장
2. **메모리 관리**: 불필요한 참조는 해제
3. **React Hooks**: stale closure 문제 인지 및 해결
4. **디버깅**: 클로저 변수 추적 어려움 인지

> 클로저는 JavaScript의 강력한 기능이지만, 올바르게 이해하고 사용해야 합니다. 데이터 은닉, 상태 관리, 함수형 프로그래밍 패턴에 활용하되, 메모리와 성능에 미치는 영향을 항상 고려하세요.
{: .prompt-info }

---

## 참고 자료

- [MDN - Closures](https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures)
- [JavaScript.info - 클로저](https://ko.javascript.info/closure)
- [You Don't Know JS - Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/README.md)
- [React 공식 문서 - Hooks FAQ](https://react.dev/reference/react/useEffect#my-effect-runs-twice-when-the-component-mounts)
- [Kent C. Dodds - React Hooks Pitfalls](https://kentcdodds.com/blog/react-hooks-pitfalls)
