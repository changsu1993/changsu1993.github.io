---
title: "JavaScript Symbol 완벽 가이드 - 고유 식별자와 메타프로그래밍"
description: "JavaScript Symbol의 개념부터 Well-known Symbols, Symbol.for() 활용까지. 고유 식별자 생성, 프로퍼티 충돌 방지, 메타프로그래밍 핵심을 마스터합니다."
date: 2025-12-20 10:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, symbol, es6, well-known-symbols, iterator, metaprogramming]
---

## 개요

JavaScript에서 객체의 프로퍼티 키는 전통적으로 문자열만 사용할 수 있었습니다. 하지만 라이브러리 간 프로퍼티 이름 충돌, 진정한 프라이빗 프로퍼티의 부재, 내장 객체 동작 커스터마이징의 어려움 등 여러 한계가 있었습니다.

ES6에서 도입된 **Symbol**은 이러한 문제를 해결하기 위한 7번째 원시 타입입니다. Symbol은 생성될 때마다 고유하고 변경 불가능한 값을 만들어내며, 객체 프로퍼티의 키로 사용할 수 있습니다. 특히 **Well-known Symbols**을 통해 JavaScript 엔진의 내부 동작을 커스터마이징하는 메타프로그래밍이 가능해졌습니다.

### 학습 목표

- Symbol의 기본 개념과 특성(고유성, 불변성) 이해
- Symbol.for()와 Symbol()의 차이점 파악
- Well-known Symbols을 활용한 메타프로그래밍 습득
- 프라이빗 프로퍼티, 프로퍼티 충돌 방지 등 실전 활용법 마스터
- Symbol vs WeakMap 비교를 통한 적절한 사용 시나리오 파악

### 사전 지식

- JavaScript 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- 객체와 프로토타입 ([JavaScript 프로토타입 체인 가이드](/posts/javascript-prototype-chain-guide/) 참고)
- ES6+ 문법 (클래스, 이터레이터 기본 개념)

---

## Symbol 기본 개념

### 7번째 원시 타입

JavaScript에는 7가지 원시 타입이 있습니다: `undefined`, `null`, `boolean`, `number`, `string`, `bigint`, 그리고 ES6에서 추가된 `symbol`입니다.

```javascript
// Symbol 생성
const sym1 = Symbol();
const sym2 = Symbol();

console.log(typeof sym1); // "symbol"
console.log(sym1 === sym2); // false - 모든 Symbol은 고유함

// 설명(description) 추가 가능
const sym3 = Symbol('mySymbol');
console.log(sym3.description); // "mySymbol" (ES2019+)
console.log(sym3.toString()); // "Symbol(mySymbol)"
```

Symbol의 가장 중요한 특성은 **고유성(uniqueness)**입니다. 같은 설명을 가진 Symbol이라도 서로 다릅니다.

```javascript
const sym1 = Symbol('id');
const sym2 = Symbol('id');

console.log(sym1 === sym2); // false
console.log(sym1 == sym2); // false (동등 비교도 false)
```

### Symbol 생성 방법

#### Symbol() - 로컬 Symbol 생성

```javascript
// 기본 생성
const localSymbol = Symbol();

// 설명과 함께 생성 (디버깅 용도)
const userId = Symbol('user id');
const secretKey = Symbol('secret key');

// Symbol은 new 키워드로 생성할 수 없음
// const sym = new Symbol(); // TypeError!
```

#### Symbol.for() - 전역 Symbol 레지스트리

`Symbol.for()`는 전역 Symbol 레지스트리에서 Symbol을 검색하거나 생성합니다. 같은 키로 호출하면 동일한 Symbol을 반환합니다.

```javascript
// 전역 레지스트리에 Symbol 생성
const globalSym1 = Symbol.for('app.id');
const globalSym2 = Symbol.for('app.id');

console.log(globalSym1 === globalSym2); // true - 같은 Symbol!

// 로컬 Symbol과의 차이
const localSym = Symbol('app.id');
console.log(globalSym1 === localSym); // false
```

#### Symbol.keyFor() - 전역 Symbol의 키 조회

```javascript
const globalSym = Symbol.for('myKey');
const localSym = Symbol('myKey');

console.log(Symbol.keyFor(globalSym)); // "myKey"
console.log(Symbol.keyFor(localSym)); // undefined (로컬 Symbol)
```

전역 Symbol은 모듈 간, 심지어 다른 realm(iframe 등) 간에도 공유됩니다.

```javascript
// 모듈 A
export const PLUGIN_ID = Symbol.for('myLibrary.pluginId');

// 모듈 B (같은 Symbol 참조)
import { PLUGIN_ID } from './moduleA';
const sameSymbol = Symbol.for('myLibrary.pluginId');
console.log(PLUGIN_ID === sameSymbol); // true
```

---

## Symbol의 특징

### 타입 변환

Symbol은 다른 원시 타입과 달리 암시적 타입 변환이 제한됩니다.

```javascript
const sym = Symbol('test');

// 문자열 변환
console.log(String(sym)); // "Symbol(test)"
console.log(sym.toString()); // "Symbol(test)"

// 암시적 문자열 변환은 에러 발생
// console.log('Symbol: ' + sym); // TypeError!
// console.log(`Symbol: ${sym}`); // TypeError!

// 숫자 변환 불가
// console.log(+sym); // TypeError!
// console.log(sym * 2); // TypeError!

// 불리언 변환은 가능
console.log(Boolean(sym)); // true
console.log(!sym); // false

if (sym) {
  console.log('Symbol은 truthy'); // 출력됨
}
```

### 열거 불가능 (Non-enumerable)

Symbol 키 프로퍼티는 일반적인 열거 메서드에서 제외됩니다.

```javascript
const id = Symbol('id');
const secret = Symbol('secret');

const user = {
  name: '김철수',
  age: 30,
  [id]: 'user_123',
  [secret]: 'password123'
};

// for...in에서 Symbol 키는 제외
for (const key in user) {
  console.log(key); // "name", "age" 만 출력
}

// Object.keys()에서도 제외
console.log(Object.keys(user)); // ["name", "age"]

// Object.values(), Object.entries()도 마찬가지
console.log(Object.entries(user)); // [["name", "김철수"], ["age", 30]]

// Symbol 프로퍼티 조회 방법
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id), Symbol(secret)]

// Reflect.ownKeys()는 모든 키(문자열 + Symbol) 반환
console.log(Reflect.ownKeys(user)); // ["name", "age", Symbol(id), Symbol(secret)]
```

### JSON 직렬화

Symbol 키 프로퍼티는 JSON 직렬화에서 완전히 무시됩니다.

```javascript
const metadata = Symbol('metadata');

const data = {
  title: '블로그 포스트',
  content: 'Symbol에 대한 내용...',
  [metadata]: { createdAt: new Date(), version: 1 }
};

console.log(JSON.stringify(data));
// {"title":"블로그 포스트","content":"Symbol에 대한 내용..."}
// Symbol 키 프로퍼티는 포함되지 않음

// 직렬화가 필요한 경우 별도 처리 필요
function customStringify(obj) {
  const symbolKeys = Object.getOwnPropertySymbols(obj);
  const combined = { ...obj };

  symbolKeys.forEach(sym => {
    combined[`__symbol_${sym.description}`] = obj[sym];
  });

  return JSON.stringify(combined);
}
```

---

## Well-known Symbols

Well-known Symbols는 JavaScript 엔진의 내부 동작을 커스터마이징할 수 있는 특수한 Symbol들입니다. 이들을 통해 메타프로그래밍이 가능해집니다.

### Symbol.iterator - 이터레이터 프로토콜

`Symbol.iterator`는 객체를 이터러블(반복 가능)하게 만드는 메서드를 정의합니다. 이터레이터에 대한 자세한 내용은 [JavaScript 제너레이터와 이터레이터 가이드](/posts/javascript-generator-iterator-guide/)를 참고하세요.

```javascript
// 커스텀 이터러블 객체
const range = {
  start: 1,
  end: 5,

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;

    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

// for...of 사용 가능
for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

// 스프레드 연산자 사용 가능
console.log([...range]); // [1, 2, 3, 4, 5]

// Array.from() 사용 가능
console.log(Array.from(range)); // [1, 2, 3, 4, 5]
```

제너레이터를 활용한 더 간결한 구현:

```javascript
const range = {
  start: 1,
  end: 5,

  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) {
      yield i;
    }
  }
};

console.log([...range]); // [1, 2, 3, 4, 5]
```

### Symbol.toStringTag - 객체 타입 태그

`Symbol.toStringTag`는 `Object.prototype.toString()`의 결과를 커스터마이징합니다.

```javascript
class CustomCollection {
  get [Symbol.toStringTag]() {
    return 'CustomCollection';
  }
}

const collection = new CustomCollection();
console.log(Object.prototype.toString.call(collection));
// "[object CustomCollection]"

// 내장 객체들도 이를 사용
console.log(Object.prototype.toString.call(new Map()));
// "[object Map]"
console.log(Object.prototype.toString.call(new Set()));
// "[object Set]"
console.log(Object.prototype.toString.call(Promise.resolve()));
// "[object Promise]"
```

실용적인 활용 예시:

```javascript
class Logger {
  constructor(name) {
    this.name = name;
  }

  get [Symbol.toStringTag]() {
    return `Logger:${this.name}`;
  }
}

const appLogger = new Logger('App');
console.log(`${Object.prototype.toString.call(appLogger)}`);
// "[object Logger:App]"

// 타입 체크 유틸리티
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType(appLogger)); // "Logger:App"
console.log(getType(new Map())); // "Map"
console.log(getType([])); // "Array"
```

### Symbol.hasInstance - instanceof 커스터마이징

`Symbol.hasInstance`는 `instanceof` 연산자의 동작을 커스터마이징합니다.

```javascript
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([] instanceof MyArray); // true
console.log([1, 2, 3] instanceof MyArray); // true
console.log({} instanceof MyArray); // false
console.log('string' instanceof MyArray); // false
```

덕 타이핑(Duck Typing) 구현:

```javascript
class Iterable {
  static [Symbol.hasInstance](instance) {
    return instance != null &&
           typeof instance[Symbol.iterator] === 'function';
  }
}

console.log([1, 2, 3] instanceof Iterable); // true
console.log('hello' instanceof Iterable); // true
console.log(new Map() instanceof Iterable); // true
console.log({ a: 1 } instanceof Iterable); // false

// 실용적 활용
function processIterable(data) {
  if (!(data instanceof Iterable)) {
    throw new TypeError('Iterable expected');
  }
  return [...data];
}
```

### Symbol.toPrimitive - 타입 변환 제어

`Symbol.toPrimitive`는 객체가 원시 값으로 변환될 때의 동작을 정의합니다.

```javascript
const money = {
  amount: 1000,
  currency: 'KRW',

  [Symbol.toPrimitive](hint) {
    console.log(`Hint: ${hint}`);

    switch (hint) {
      case 'number':
        return this.amount;
      case 'string':
        return `${this.amount} ${this.currency}`;
      case 'default':
        return this.amount;
    }
  }
};

// 숫자 컨텍스트 (hint: "number")
console.log(+money); // 1000
console.log(money * 2); // 2000

// 문자열 컨텍스트 (hint: "string")
console.log(String(money)); // "1000 KRW"
console.log(`${money}`); // "1000 KRW"

// 기본 컨텍스트 (hint: "default")
console.log(money + 500); // 1500
console.log(money == 1000); // true
```

날짜 객체 예시:

```javascript
class CustomDate {
  constructor(date) {
    this.date = new Date(date);
  }

  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case 'number':
        return this.date.getTime();
      case 'string':
        return this.date.toLocaleDateString('ko-KR');
      default:
        return this.date.toISOString();
    }
  }
}

const today = new CustomDate('2025-12-20');

console.log(+today); // 1734652800000 (타임스탬프)
console.log(String(today)); // "2025. 12. 20."
console.log(today + ''); // "2025-12-20T00:00:00.000Z"
```

### Symbol.species - 파생 객체 생성자 지정

`Symbol.species`는 파생 객체를 생성할 때 사용할 생성자를 지정합니다.

```javascript
class MyArray extends Array {
  // map, filter 등이 MyArray가 아닌 Array를 반환하도록 설정
  static get [Symbol.species]() {
    return Array;
  }
}

const myArr = new MyArray(1, 2, 3);
console.log(myArr instanceof MyArray); // true

const mapped = myArr.map(x => x * 2);
console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array); // true
```

Promise에서의 활용:

```javascript
class TrackedPromise extends Promise {
  static get [Symbol.species]() {
    return Promise; // then()이 일반 Promise 반환
  }
}

const tracked = new TrackedPromise(resolve => resolve(1));
const chained = tracked.then(x => x + 1);

console.log(tracked instanceof TrackedPromise); // true
console.log(chained instanceof TrackedPromise); // false
console.log(chained instanceof Promise); // true
```

### 기타 Well-known Symbols

```javascript
// Symbol.isConcatSpreadable - 배열 병합 동작 제어
const arr = [1, 2, 3];
arr[Symbol.isConcatSpreadable] = false;
console.log([0].concat(arr)); // [0, [1, 2, 3]]

// Symbol.match, Symbol.replace, Symbol.search, Symbol.split
// 문자열 메서드의 동작 커스터마이징
class CaseInsensitiveMatcher {
  constructor(pattern) {
    this.pattern = pattern.toLowerCase();
  }

  [Symbol.match](string) {
    const index = string.toLowerCase().indexOf(this.pattern);
    if (index === -1) return null;
    return [string.slice(index, index + this.pattern.length)];
  }
}

console.log('Hello World'.match(new CaseInsensitiveMatcher('WORLD')));
// ["World"]

// Symbol.unscopables - with 문에서 제외할 프로퍼티 지정 (거의 사용 안 함)
```

---

## 실전 활용 사례

### 프라이빗 프로퍼티 구현

Symbol을 사용하여 "준 프라이빗" 프로퍼티를 구현할 수 있습니다.

```javascript
// 모듈 스코프에서 Symbol 정의
const _balance = Symbol('balance');
const _transactions = Symbol('transactions');

class BankAccount {
  constructor(initialBalance) {
    this[_balance] = initialBalance;
    this[_transactions] = [];
  }

  deposit(amount) {
    this[_balance] += amount;
    this[_transactions].push({ type: 'deposit', amount, date: new Date() });
  }

  withdraw(amount) {
    if (amount > this[_balance]) {
      throw new Error('잔액 부족');
    }
    this[_balance] -= amount;
    this[_transactions].push({ type: 'withdraw', amount, date: new Date() });
  }

  getBalance() {
    return this[_balance];
  }

  getTransactionHistory() {
    return [...this[_transactions]]; // 복사본 반환
  }
}

const account = new BankAccount(1000);
account.deposit(500);

console.log(account.getBalance()); // 1500
console.log(Object.keys(account)); // [] - Symbol 프로퍼티는 숨겨짐

// 하지만 완전히 프라이빗하지 않음
console.log(Object.getOwnPropertySymbols(account)); // [Symbol(balance), Symbol(transactions)]
```

### 객체 프로퍼티 충돌 방지

라이브러리에서 객체에 메타데이터를 추가할 때 Symbol을 사용하면 사용자 프로퍼티와 충돌을 방지할 수 있습니다.

```javascript
// 라이브러리 A
const LIBRARY_A_DATA = Symbol('libraryA.data');

function addLibraryAFeature(obj) {
  obj[LIBRARY_A_DATA] = {
    version: '1.0',
    initialized: true
  };
}

// 라이브러리 B (같은 설명을 사용해도 충돌 없음)
const LIBRARY_B_DATA = Symbol('libraryB.data');

function addLibraryBFeature(obj) {
  obj[LIBRARY_B_DATA] = {
    config: {},
    active: true
  };
}

// 사용자 코드
const myObject = {
  name: '내 객체',
  data: '사용자 데이터' // 문자열 'data' 키
};

addLibraryAFeature(myObject);
addLibraryBFeature(myObject);

// 모든 데이터가 충돌 없이 공존
console.log(myObject.data); // "사용자 데이터"
console.log(myObject[LIBRARY_A_DATA]); // { version: '1.0', initialized: true }
console.log(myObject[LIBRARY_B_DATA]); // { config: {}, active: true }
```

### 커스텀 이터레이터 구현

```javascript
// 페이지네이션 이터레이터
class Paginator {
  constructor(items, pageSize = 10) {
    this.items = items;
    this.pageSize = pageSize;
  }

  *[Symbol.iterator]() {
    for (let i = 0; i < this.items.length; i += this.pageSize) {
      yield this.items.slice(i, i + this.pageSize);
    }
  }

  // 역방향 이터레이션
  *reversed() {
    for (let i = this.items.length; i > 0; i -= this.pageSize) {
      const start = Math.max(0, i - this.pageSize);
      yield this.items.slice(start, i);
    }
  }
}

const items = Array.from({ length: 25 }, (_, i) => `Item ${i + 1}`);
const paginator = new Paginator(items, 10);

for (const page of paginator) {
  console.log(`페이지: ${page.length}개 항목`);
}
// 페이지: 10개 항목
// 페이지: 10개 항목
// 페이지: 5개 항목

// 이중 연결 리스트 이터레이터
class DoublyLinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
  }

  append(value) {
    const node = { value, prev: this.tail, next: null };
    if (this.tail) {
      this.tail.next = node;
    } else {
      this.head = node;
    }
    this.tail = node;
    return this;
  }

  *[Symbol.iterator]() {
    let current = this.head;
    while (current) {
      yield current.value;
      current = current.next;
    }
  }

  *reverse() {
    let current = this.tail;
    while (current) {
      yield current.value;
      current = current.prev;
    }
  }
}

const list = new DoublyLinkedList();
list.append(1).append(2).append(3);

console.log([...list]); // [1, 2, 3]
console.log([...list.reverse()]); // [3, 2, 1]
```

### 프레임워크/라이브러리에서의 활용

```javascript
// React 스타일 컴포넌트 타입 식별
const REACT_ELEMENT_TYPE = Symbol.for('react.element');
const REACT_FRAGMENT_TYPE = Symbol.for('react.fragment');

function createElement(type, props, ...children) {
  return {
    $$typeof: REACT_ELEMENT_TYPE,
    type,
    props: { ...props, children },
    key: props?.key || null
  };
}

function isValidElement(object) {
  return (
    typeof object === 'object' &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}

// Redux 액션 타입
const ActionTypes = {
  ADD_TODO: Symbol('ADD_TODO'),
  REMOVE_TODO: Symbol('REMOVE_TODO'),
  TOGGLE_TODO: Symbol('TOGGLE_TODO')
};

// Symbol을 키로 사용하면 실수로 문자열과 충돌하지 않음
function todoReducer(state = [], action) {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      return [...state, action.payload];
    case ActionTypes.REMOVE_TODO:
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}
```

---

## Symbol vs WeakMap - 프라이빗 데이터 비교

프라이빗 데이터를 구현하는 두 가지 주요 방법을 비교해봅시다. Map과 WeakMap에 대한 자세한 내용은 [JavaScript Map과 Set 완벽 가이드](/posts/javascript-map-set-guide/)를 참고하세요.

### Symbol 방식

```javascript
const _name = Symbol('name');
const _age = Symbol('age');

class Person {
  constructor(name, age) {
    this[_name] = name;
    this[_age] = age;
  }

  greet() {
    return `안녕하세요, ${this[_name]}입니다.`;
  }
}

const person = new Person('김철수', 30);
```

### WeakMap 방식

```javascript
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age });
  }

  greet() {
    const { name } = privateData.get(this);
    return `안녕하세요, ${name}입니다.`;
  }
}

const person = new Person('김철수', 30);
```

### 비교 분석

| 특성 | Symbol | WeakMap |
|------|--------|---------|
| 프라이빗 수준 | 준 프라이빗 (발견 가능) | 진정한 프라이빗 |
| 접근 방법 | `Object.getOwnPropertySymbols()` | 불가능 |
| 성능 | 직접 접근 (빠름) | 해시 조회 (약간 느림) |
| 메모리 | 인스턴스에 저장 | 별도 WeakMap에 저장 |
| 상속 | 가능 | 복잡함 |
| 디버깅 | 쉬움 (프로퍼티 보임) | 어려움 |
| JSON 직렬화 | 자동 제외 | 별도 처리 필요 |

### 현대적 대안: Private Class Fields

ES2022부터 `#` 접두사를 사용한 진정한 프라이빗 필드가 지원됩니다.

```javascript
class Person {
  #name;
  #age;

  constructor(name, age) {
    this.#name = name;
    this.#age = age;
  }

  greet() {
    return `안녕하세요, ${this.#name}입니다.`;
  }
}

const person = new Person('김철수', 30);
// person.#name; // SyntaxError: Private field
```

**선택 가이드:**
- **Symbol**: 열거에서 숨기고 싶지만 리플렉션 접근은 허용할 때
- **WeakMap**: 완전한 프라이빗이 필요하고 레거시 환경 지원 시
- **Private Fields (#)**: 모던 환경에서 진정한 프라이빗 필드가 필요할 때

---

## 브라우저 호환성 및 폴리필

### 브라우저 지원 현황

Symbol은 ES6(ES2015)에서 도입되어 모든 현대 브라우저에서 지원됩니다.

| 기능 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| Symbol (기본) | 38+ | 36+ | 9+ | 12+ |
| Symbol.iterator | 38+ | 36+ | 9+ | 12+ |
| Symbol.for/keyFor | 40+ | 36+ | 9+ | 12+ |
| Well-known Symbols | 47+ | 36+ | 9+ | 12+ |
| Symbol.description | 70+ | 63+ | 12.1+ | 79+ |

### 폴리필 고려사항

Symbol은 완전한 폴리필이 불가능합니다. Symbol의 핵심 기능인 "고유성"을 다른 원시 타입으로 구현할 수 없기 때문입니다.

```javascript
// 부분적 폴리필 (완전하지 않음)
if (!window.Symbol) {
  window.Symbol = (function() {
    let counter = 0;

    function Symbol(description) {
      if (this instanceof Symbol) {
        throw new TypeError('Symbol is not a constructor');
      }

      const sym = Object.create(Symbol.prototype);
      sym._description = description;
      sym._id = `@@symbol_${++counter}_${Math.random().toString(36).slice(2)}`;
      return sym;
    }

    Symbol.prototype.toString = function() {
      return `Symbol(${this._description || ''})`;
    };

    // Symbol.for, Symbol.keyFor 등 추가 구현 필요...

    return Symbol;
  })();
}
```

레거시 환경 지원이 필요하다면 **core-js** 폴리필을 사용하되, 완전한 기능은 기대하지 마세요.

```bash
npm install core-js
```

```javascript
import 'core-js/features/symbol';
```

---

## 실전 예제: 완전한 구현

### 플러그인 시스템 구현

```javascript
// 플러그인 시스템의 핵심 Symbol들
const PLUGIN_INIT = Symbol('plugin.init');
const PLUGIN_DESTROY = Symbol('plugin.destroy');
const PLUGIN_META = Symbol('plugin.meta');

class PluginManager {
  #plugins = new Map();

  register(Plugin) {
    const instance = new Plugin();
    const meta = instance[PLUGIN_META] || { name: Plugin.name, version: '1.0.0' };

    if (typeof instance[PLUGIN_INIT] === 'function') {
      instance[PLUGIN_INIT]();
    }

    this.#plugins.set(meta.name, { instance, meta });
    console.log(`플러그인 등록: ${meta.name} v${meta.version}`);

    return this;
  }

  unregister(name) {
    const plugin = this.#plugins.get(name);
    if (plugin && typeof plugin.instance[PLUGIN_DESTROY] === 'function') {
      plugin.instance[PLUGIN_DESTROY]();
    }
    this.#plugins.delete(name);
  }

  *[Symbol.iterator]() {
    yield* this.#plugins.values();
  }
}

// 플러그인 예시
class LoggerPlugin {
  get [PLUGIN_META]() {
    return { name: 'Logger', version: '2.0.0' };
  }

  [PLUGIN_INIT]() {
    console.log('Logger 플러그인 초기화됨');
    this.startTime = Date.now();
  }

  [PLUGIN_DESTROY]() {
    console.log(`Logger 플러그인 종료 (실행시간: ${Date.now() - this.startTime}ms)`);
  }

  log(message) {
    console.log(`[LOG] ${message}`);
  }
}

const manager = new PluginManager();
manager.register(LoggerPlugin);

for (const { meta } of manager) {
  console.log(`활성 플러그인: ${meta.name}`);
}
```

---

## 정리

Symbol은 JavaScript의 7번째 원시 타입으로, 고유하고 변경 불가능한 값을 생성합니다. 주요 특징과 활용법을 정리하면 다음과 같습니다.

**핵심 개념:**
- `Symbol()`은 항상 고유한 값을 생성 (같은 설명이라도 다름)
- `Symbol.for()`는 전역 레지스트리에서 Symbol을 공유
- Symbol 프로퍼티는 열거에서 제외되고 JSON 직렬화에서 무시됨

**Well-known Symbols:**
- `Symbol.iterator`: 객체를 이터러블하게 만듦
- `Symbol.toStringTag`: 객체 타입 문자열 커스터마이징
- `Symbol.hasInstance`: instanceof 동작 커스터마이징
- `Symbol.toPrimitive`: 타입 변환 동작 제어
- `Symbol.species`: 파생 객체 생성자 지정

**실전 활용:**
- 프라이빗 프로퍼티 구현 (준 프라이빗)
- 라이브러리 간 프로퍼티 충돌 방지
- 커스텀 이터레이터 구현
- 메타프로그래밍 및 내장 동작 커스터마이징

Symbol을 이해하면 JavaScript의 고급 패턴과 라이브러리 내부 구조를 더 깊이 이해할 수 있습니다. Proxy와 Reflect와 함께 사용하면 더욱 강력한 메타프로그래밍이 가능합니다. 자세한 내용은 [JavaScript Proxy와 Reflect 완벽 가이드](/posts/javascript-proxy-reflect-guide/)를 참고하세요.

---

## 참고 자료

- [MDN - Symbol](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- [MDN - Well-known Symbols](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol#well-known_symbols)
- [ECMAScript 명세 - Symbol](https://tc39.es/ecma262/#sec-symbol-objects)
- [JavaScript Map과 Set 완벽 가이드](/posts/javascript-map-set-guide/)
- [JavaScript Proxy와 Reflect 완벽 가이드](/posts/javascript-proxy-reflect-guide/)
- [JavaScript 제너레이터와 이터레이터 가이드](/posts/javascript-generator-iterator-guide/)
- [JavaScript 프로토타입 체인 가이드](/posts/javascript-prototype-chain-guide/)
