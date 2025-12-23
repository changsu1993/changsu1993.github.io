---
title: "JavaScript WeakMap과 WeakRef 완벽 가이드 - 메모리 효율적 참조 관리"
description: "JavaScript 약한 참조(WeakMap, WeakSet, WeakRef, FinalizationRegistry)를 활용한 메모리 관리 패턴. 캐시, 프라이빗 데이터, React 통합 등 실전 예제 포함."
date: 2025-12-23 09:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, weakmap, weakset, weakref, finalizationregistry, memory-management, garbage-collection, cache, es2021, weak-reference, memory-leak, memoization, react-hooks, private-data]
---

## 개요

JavaScript에서 객체 참조를 관리할 때, 일반적인 참조는 **강한 참조(Strong Reference)**입니다. 강한 참조는 참조가 존재하는 한 객체가 가비지 컬렉션(GC)되지 않도록 보장합니다. 하지만 캐시, 메타데이터 저장, 이벤트 핸들러 관리 등의 시나리오에서는 이러한 동작이 메모리 누수를 유발할 수 있습니다.

**약한 참조(Weak Reference)**는 이 문제를 해결합니다. 약한 참조는 객체가 다른 곳에서 참조되지 않으면 GC가 해당 객체를 수집할 수 있도록 허용합니다. JavaScript는 이를 위해 **WeakMap**, **WeakSet**, **WeakRef**, **FinalizationRegistry**를 제공합니다.

### 학습 목표

- 강한 참조와 약한 참조의 차이점 이해
- WeakMap과 WeakSet의 고급 활용 패턴 습득
- WeakRef(ES2021)의 개념과 사용 사례 파악
- FinalizationRegistry를 활용한 정리 콜백 구현
- React 등 실제 프레임워크에서의 활용법 익히기

### 사전 지식

- JavaScript 기초 문법
- [Map과 Set 기본 개념](/posts/javascript-map-set-guide/)
- [메모리 관리와 가비지 컬렉션의 기본 원리](/posts/javascript-memory-management-garbage-collection-guide/)

---

## 강한 참조 vs 약한 참조

### 강한 참조의 문제점

일반적인 JavaScript 참조는 모두 강한 참조입니다. 변수, 객체 프로퍼티, 배열 요소, Map/Set의 키와 값 모두 강한 참조입니다.

```javascript
// 강한 참조 예시
const cache = new Map();

function processUser(user) {
  // user 객체를 캐시에 저장 (강한 참조)
  const result = expensiveComputation(user);
  cache.set(user, result);
  return result;
}

let user = { id: 1, name: '김철수' };
processUser(user);

// user를 더 이상 사용하지 않아도
user = null;

// cache가 여전히 { id: 1, name: '김철수' } 객체를 참조
// 원래 객체는 GC되지 않음 - 메모리 누수!
console.log(cache.size); // 1
```

이 문제는 캐시가 커질수록 심각해집니다. 더 이상 필요 없는 데이터가 메모리에 계속 남아있게 됩니다.

### 약한 참조의 해결책

약한 참조는 GC의 결정에 영향을 주지 않습니다. 객체에 대한 다른 강한 참조가 없으면, 약한 참조만으로는 객체를 메모리에 유지할 수 없습니다.

```javascript
// 약한 참조 예시
const cache = new WeakMap();

function processUser(user) {
  // user 객체를 WeakMap에 저장 (약한 참조)
  const result = expensiveComputation(user);
  cache.set(user, result);
  return result;
}

let user = { id: 1, name: '김철수' };
processUser(user);

// user를 더 이상 사용하지 않으면
user = null;

// WeakMap의 참조는 약한 참조이므로
// { id: 1, name: '김철수' } 객체는 GC 대상이 됨
// 메모리 누수 방지!
```

---

## WeakMap 심화

WeakMap의 기본 개념은 [Map과 Set 가이드](/posts/javascript-map-set-guide/)에서 다루었습니다. 여기서는 실전에서 활용할 수 있는 고급 패턴에 집중합니다.

### WeakMap과 Map의 핵심 차이

| 특성 | Map | WeakMap |
|------|-----|---------|
| 키 타입 | 모든 값 | 객체만 (Symbol 제외) |
| 참조 방식 | 강한 참조 | 약한 참조 |
| 이터러블 | O (keys, values, entries, forEach) | X |
| size 프로퍼티 | O | X |
| GC와의 관계 | 키가 참조되는 한 유지 | 키에 대한 다른 참조가 없으면 GC 가능 |

WeakMap이 이터러블하지 않은 이유는 GC가 언제든 엔트리를 제거할 수 있기 때문입니다. 순회 중에 내용이 변경될 수 있어 일관성을 보장할 수 없습니다.

### 패턴 1: 객체에 프라이빗 데이터 연결

ES2022 이전에는 클래스의 진정한 private 필드가 없었습니다. WeakMap은 [클로저](/posts/javascript-closure-complete-guide/)와 함께 외부에서 접근할 수 없는 프라이빗 데이터를 구현하는 패턴으로 널리 사용되었습니다.

```javascript
// 모듈 스코프에 WeakMap 선언 (외부 접근 불가)
const privateState = new WeakMap();

class BankAccount {
  constructor(initialBalance, pin) {
    // 인스턴스를 키로 사용하여 프라이빗 데이터 저장
    privateState.set(this, {
      balance: initialBalance,
      pin: pin,
      transactions: []
    });
  }

  getBalance(inputPin) {
    const state = privateState.get(this);
    if (state.pin !== inputPin) {
      throw new Error('잘못된 PIN입니다.');
    }
    return state.balance;
  }

  deposit(amount, inputPin) {
    const state = privateState.get(this);
    if (state.pin !== inputPin) {
      throw new Error('잘못된 PIN입니다.');
    }
    state.balance += amount;
    state.transactions.push({ type: 'deposit', amount, date: new Date() });
  }

  withdraw(amount, inputPin) {
    const state = privateState.get(this);
    if (state.pin !== inputPin) {
      throw new Error('잘못된 PIN입니다.');
    }
    if (state.balance < amount) {
      throw new Error('잔액이 부족합니다.');
    }
    state.balance -= amount;
    state.transactions.push({ type: 'withdraw', amount, date: new Date() });
  }

  getTransactionCount(inputPin) {
    const state = privateState.get(this);
    if (state.pin !== inputPin) {
      throw new Error('잘못된 PIN입니다.');
    }
    return state.transactions.length;
  }
}

const account = new BankAccount(1000, '1234');
account.deposit(500, '1234');
console.log(account.getBalance('1234')); // 1500

// 프라이빗 데이터에 직접 접근 불가
console.log(account.balance); // undefined
console.log(account.pin);     // undefined

// account가 GC되면 privateState의 해당 엔트리도 자동 정리
```

> ES2022부터는 `#` 접두사로 진정한 private 필드를 사용할 수 있습니다. 하지만 레거시 환경 지원이 필요하거나, 클래스 외부에서 프라이빗 데이터를 관리해야 하는 경우 WeakMap 패턴이 여전히 유용합니다.
{: .prompt-tip }

### 패턴 2: DOM 요소에 메타데이터 연결

DOM 요소에 커스텀 데이터를 연결할 때, `data-*` 속성이나 직접 프로퍼티 할당보다 WeakMap이 더 안전합니다.

```javascript
// DOM 요소별 상태 관리
const elementState = new WeakMap();

function initializeElement(element) {
  elementState.set(element, {
    isInitialized: true,
    clickCount: 0,
    lastInteraction: null,
    customHandlers: []
  });
}

function trackClick(element) {
  let state = elementState.get(element);

  if (!state) {
    initializeElement(element);
    state = elementState.get(element);
  }

  state.clickCount++;
  state.lastInteraction = new Date();
}

function addCustomHandler(element, handler) {
  let state = elementState.get(element);

  if (!state) {
    initializeElement(element);
    state = elementState.get(element);
  }

  state.customHandlers.push(handler);
}

function getElementStats(element) {
  const state = elementState.get(element);
  if (!state) return null;

  return {
    clicks: state.clickCount,
    lastInteraction: state.lastInteraction,
    handlerCount: state.customHandlers.length
  };
}

// 사용 예시
document.querySelectorAll('.trackable').forEach(el => {
  initializeElement(el);

  el.addEventListener('click', () => {
    trackClick(el);
    console.log(getElementStats(el));
  });
});

// DOM에서 요소가 제거되고 다른 참조가 없으면
// elementState의 해당 엔트리도 자동으로 GC됨
```

**장점:**
- DOM 요소가 제거되면 관련 메타데이터도 자동 정리
- 요소의 프로퍼티를 오염시키지 않음
- 다른 라이브러리와의 충돌 위험 없음

### 패턴 3: 메모이제이션 캐시

함수의 계산 결과를 캐싱할 때, 입력 객체가 더 이상 필요 없어지면 캐시도 자동으로 정리됩니다.

```javascript
// 객체 기반 메모이제이션
function createMemoizedProcessor() {
  const cache = new WeakMap();

  return function process(obj) {
    // 캐시 확인
    if (cache.has(obj)) {
      console.log('캐시 히트');
      return cache.get(obj);
    }

    // 비용이 큰 연산 수행
    console.log('연산 수행');
    const result = {
      keys: Object.keys(obj),
      values: Object.values(obj),
      hash: JSON.stringify(obj).split('').reduce((a, b) => {
        a = ((a << 5) - a) + b.charCodeAt(0);
        return a & a;
      }, 0),
      processedAt: new Date()
    };

    // 결과 캐싱
    cache.set(obj, result);
    return result;
  };
}

const process = createMemoizedProcessor();

const config = { theme: 'dark', language: 'ko', debug: false };
process(config); // "연산 수행"
process(config); // "캐시 히트"

// config를 더 이상 사용하지 않으면 캐시도 자동 정리
```

### 패턴 4: 순환 참조 안전한 직렬화

객체를 직렬화할 때 순환 참조를 감지하기 위해 WeakSet을 사용합니다.

```javascript
function safeStringify(obj, space = 2) {
  const seen = new WeakSet();

  return JSON.stringify(obj, (key, value) => {
    // 객체가 아니면 그대로 반환
    if (typeof value !== 'object' || value === null) {
      return value;
    }

    // 이미 방문한 객체면 순환 참조
    if (seen.has(value)) {
      return '[Circular Reference]';
    }

    // 방문 기록
    seen.add(value);
    return value;
  }, space);
}

// 순환 참조가 있는 객체
const user = { name: '김철수' };
const team = { name: '개발팀', leader: user };
user.team = team; // 순환 참조!

console.log(safeStringify(user));
// {
//   "name": "김철수",
//   "team": {
//     "name": "개발팀",
//     "leader": "[Circular Reference]"
//   }
// }

// seen WeakSet은 함수 종료 후 GC 가능
```

---

## WeakSet 심화

WeakSet은 객체만 저장할 수 있고, 약한 참조를 사용하는 Set입니다. 주로 객체의 "상태"나 "플래그"를 추적하는 데 사용됩니다.

### 패턴 1: 객체 초기화 추적

```javascript
const initialized = new WeakSet();

class LazyComponent {
  constructor(config) {
    this.config = config;
    // 초기화는 나중에 필요할 때
  }

  ensureInitialized() {
    if (initialized.has(this)) {
      return; // 이미 초기화됨
    }

    console.log('초기화 수행...');
    this.data = this.loadData();
    this.setupListeners();

    initialized.add(this);
  }

  loadData() {
    // 데이터 로드 로직
    return { loaded: true };
  }

  setupListeners() {
    // 이벤트 리스너 설정
  }

  render() {
    this.ensureInitialized();
    console.log('렌더링:', this.data);
  }
}

const comp = new LazyComponent({ id: 1 });
comp.render(); // "초기화 수행..." 후 "렌더링: { loaded: true }"
comp.render(); // "렌더링: { loaded: true }" (초기화 생략)
```

### 패턴 2: 브랜딩 패턴 (객체 유효성 검증)

특정 생성자로 만들어진 객체인지 검증하는 패턴입니다.

```javascript
const validTokens = new WeakSet();

class AuthToken {
  constructor(userId, expiresAt) {
    this.userId = userId;
    this.expiresAt = expiresAt;
    this.createdAt = Date.now();

    // 유효한 토큰으로 등록
    validTokens.add(this);
  }

  static isValid(token) {
    // 1. AuthToken 생성자로 만들어진 객체인지 확인
    if (!validTokens.has(token)) {
      return false;
    }

    // 2. 만료 시간 확인
    if (Date.now() > token.expiresAt) {
      return false;
    }

    return true;
  }

  static revoke(token) {
    // 토큰 무효화
    validTokens.delete(token);
  }
}

const token = new AuthToken('user123', Date.now() + 3600000);
console.log(AuthToken.isValid(token)); // true

// 가짜 토큰 시도
const fakeToken = {
  userId: 'user123',
  expiresAt: Date.now() + 3600000,
  createdAt: Date.now()
};
console.log(AuthToken.isValid(fakeToken)); // false

// 토큰 무효화
AuthToken.revoke(token);
console.log(AuthToken.isValid(token)); // false
```

### 패턴 3: 이벤트 처리 중복 방지

```javascript
const handledEvents = new WeakSet();

function handleOnce(event, handler) {
  // 이미 처리된 이벤트인지 확인
  if (handledEvents.has(event)) {
    console.log('이벤트가 이미 처리되었습니다.');
    return;
  }

  // 이벤트 처리
  handler(event);

  // 처리됨으로 표시
  handledEvents.add(event);
}

// 이벤트 버블링 중 중복 처리 방지
document.addEventListener('click', (event) => {
  handleOnce(event, (e) => {
    console.log('클릭 처리:', e.target);
  });
});

// 같은 이벤트가 다른 핸들러에서 다시 처리되지 않음
```

---

## WeakRef (ES2021)

WeakRef는 ES2021에서 도입된 기능으로, 객체에 대한 약한 참조를 명시적으로 생성할 수 있게 해줍니다. WeakMap/WeakSet과 달리, WeakRef는 약한 참조를 직접 다룰 수 있습니다.

### WeakRef 기본 사용법

```javascript
// 객체 생성
let user = { name: '김철수', data: new Array(10000).fill('data') };

// WeakRef로 약한 참조 생성
const weakRef = new WeakRef(user);

// deref()로 원본 객체 접근
console.log(weakRef.deref()); // { name: '김철수', data: [...] }
console.log(weakRef.deref()?.name); // '김철수'

// 원본 참조 제거
user = null;

// GC가 실행된 후에는 deref()가 undefined 반환 가능
// (GC 타이밍은 예측 불가)
setTimeout(() => {
  const obj = weakRef.deref();
  if (obj) {
    console.log('객체가 아직 존재합니다:', obj.name);
  } else {
    console.log('객체가 GC되었습니다.');
  }
}, 1000);
```

> WeakRef.prototype.deref()는 객체가 GC되었으면 `undefined`를 반환합니다. 따라서 항상 반환값을 확인해야 합니다.
{: .prompt-warning }

### WeakRef vs WeakMap

| 특성 | WeakRef | WeakMap |
|------|---------|---------|
| 용도 | 단일 객체에 대한 약한 참조 | 객체-값 쌍의 약한 참조 컬렉션 |
| 참조 접근 | deref() 메서드 | get() 메서드 |
| 값 연결 | 불가능 | 가능 |
| 사용 시나리오 | 캐시, 옵저버 패턴 | 메타데이터 저장, 프라이빗 데이터 |

### 패턴 1: 메모리 효율적인 캐시

WeakRef를 사용하면 캐시된 객체가 필요 없어졌을 때 자동으로 메모리에서 해제됩니다.

```javascript
class WeakCache {
  constructor() {
    this.cache = new Map(); // key -> WeakRef
  }

  set(key, value) {
    // 값을 WeakRef로 감싸서 저장
    this.cache.set(key, new WeakRef(value));
  }

  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;

    // deref()로 실제 값 가져오기
    const value = ref.deref();

    // GC되었으면 캐시에서 제거
    if (value === undefined) {
      this.cache.delete(key);
      return undefined;
    }

    return value;
  }

  has(key) {
    return this.get(key) !== undefined;
  }

  delete(key) {
    return this.cache.delete(key);
  }

  // 정리: GC된 엔트리 제거
  cleanup() {
    for (const [key, ref] of this.cache) {
      if (ref.deref() === undefined) {
        this.cache.delete(key);
      }
    }
  }
}

// 사용 예시
const imageCache = new WeakCache();

async function loadImage(url) {
  // 캐시 확인
  let image = imageCache.get(url);
  if (image) {
    console.log('캐시에서 이미지 반환');
    return image;
  }

  // 이미지 로드
  console.log('이미지 로드 중...');
  image = await fetchImage(url);

  // 캐시에 저장
  imageCache.set(url, image);
  return image;
}

async function fetchImage(url) {
  // 이미지 로드 시뮬레이션
  return { url, data: new ArrayBuffer(1024 * 1024), loadedAt: Date.now() };
}
```

### 패턴 2: 옵저버 패턴에서 메모리 누수 방지

이벤트 에미터나 옵저버 패턴에서 리스너 객체가 GC되면 자동으로 구독이 해제됩니다.

```javascript
class WeakEventEmitter {
  constructor() {
    this.listeners = new Map(); // eventName -> Set<WeakRef>
  }

  on(eventName, listener) {
    if (!this.listeners.has(eventName)) {
      this.listeners.set(eventName, new Set());
    }

    // 리스너 객체를 WeakRef로 저장
    const listenerObj = { callback: listener };
    this.listeners.get(eventName).add(new WeakRef(listenerObj));

    // 구독 해제 함수 반환
    return listenerObj; // 이 객체에 대한 참조를 유지해야 구독 유지
  }

  emit(eventName, ...args) {
    const refs = this.listeners.get(eventName);
    if (!refs) return;

    for (const ref of refs) {
      const listenerObj = ref.deref();

      if (listenerObj) {
        // 리스너가 아직 존재하면 호출
        listenerObj.callback(...args);
      } else {
        // GC되었으면 Set에서 제거
        refs.delete(ref);
      }
    }
  }

  // 명시적 구독 해제 (옵션)
  off(eventName, listenerObj) {
    const refs = this.listeners.get(eventName);
    if (!refs) return;

    for (const ref of refs) {
      if (ref.deref() === listenerObj) {
        refs.delete(ref);
        return;
      }
    }
  }
}

// 사용 예시
const emitter = new WeakEventEmitter();

function setupComponent() {
  const subscription = emitter.on('data', (data) => {
    console.log('데이터 수신:', data);
  });

  // subscription을 저장해야 구독이 유지됨
  return { subscription };
}

let component = setupComponent();
emitter.emit('data', { value: 1 }); // "데이터 수신: { value: 1 }"

// 컴포넌트 제거
component = null;

// GC 후 리스너가 자동으로 제거됨
// 명시적인 cleanup 호출 필요 없음
```

### 패턴 3: 대용량 객체의 지연 로딩

```javascript
class LazyLoader {
  constructor(loadFn) {
    this.loadFn = loadFn;
    this.weakRef = null;
    this.loading = false;
  }

  async get() {
    // 캐시된 값 확인
    if (this.weakRef) {
      const value = this.weakRef.deref();
      if (value !== undefined) {
        console.log('캐시된 값 반환');
        return value;
      }
    }

    // 로드 중이면 대기
    if (this.loading) {
      return this.loadPromise;
    }

    // 새로 로드
    console.log('새로 로드');
    this.loading = true;
    this.loadPromise = this.loadFn();

    try {
      const value = await this.loadPromise;
      this.weakRef = new WeakRef(value);
      return value;
    } finally {
      this.loading = false;
    }
  }

  // 명시적 캐시 무효화
  invalidate() {
    this.weakRef = null;
  }
}

// 사용 예시
const heavyDataLoader = new LazyLoader(async () => {
  // 대용량 데이터 로드 시뮬레이션
  await new Promise(resolve => setTimeout(resolve, 1000));
  return {
    data: new Array(100000).fill(0).map((_, i) => ({ id: i, value: Math.random() })),
    loadedAt: Date.now()
  };
});

async function processData() {
  const data = await heavyDataLoader.get(); // 처음: "새로 로드"
  console.log('데이터 크기:', data.data.length);

  const data2 = await heavyDataLoader.get(); // "캐시된 값 반환"
  console.log('같은 데이터:', data === data2); // true
}
```

---

## FinalizationRegistry (ES2021)

FinalizationRegistry는 객체가 GC될 때 콜백을 실행할 수 있게 해주는 ES2021 기능입니다. 이를 통해 객체가 정리될 때 관련 리소스를 해제할 수 있습니다.

### 기본 사용법

```javascript
// 정리 콜백을 받는 레지스트리 생성
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`객체가 GC되었습니다. 보유 값: ${heldValue}`);
  // 여기서 관련 리소스 정리
});

let user = { name: '김철수' };

// 객체 등록 (객체, 보유값, 등록해제 토큰)
registry.register(user, 'user-123', user);

// user를 더 이상 참조하지 않으면
user = null;

// GC가 실행되면 콜백 호출
// "객체가 GC되었습니다. 보유 값: user-123"
```

### FinalizationRegistry의 구성 요소

| 파라미터 | 설명 |
|----------|------|
| target | 추적할 객체 |
| heldValue | GC 시 콜백에 전달될 값 (target과 달라야 함) |
| unregisterToken | 등록 해제에 사용할 토큰 (선택) |

> `heldValue`로 target 객체 자체를 전달하면 안 됩니다. 그러면 강한 참조가 생겨 객체가 GC되지 않습니다.
{: .prompt-danger }

### 패턴 1: 외부 리소스 자동 정리

파일 핸들, 네트워크 연결, WebSocket 등 외부 리소스를 관리할 때 유용합니다.

```javascript
// 리소스 ID 관리
let nextResourceId = 0;
const allocatedResources = new Set();

// 정리 레지스트리
const resourceRegistry = new FinalizationRegistry((resourceId) => {
  console.log(`리소스 ${resourceId} 자동 정리`);
  allocatedResources.delete(resourceId);
  // 실제로는 여기서 외부 리소스 해제
  // closeResource(resourceId);
});

class ManagedResource {
  constructor() {
    this.resourceId = nextResourceId++;
    allocatedResources.add(this.resourceId);

    // 이 객체가 GC되면 resourceId로 정리 콜백 호출
    resourceRegistry.register(this, this.resourceId, this);

    console.log(`리소스 ${this.resourceId} 할당됨`);
  }

  use() {
    console.log(`리소스 ${this.resourceId} 사용 중`);
  }

  // 명시적 정리 메서드 (권장)
  dispose() {
    console.log(`리소스 ${this.resourceId} 명시적 정리`);
    allocatedResources.delete(this.resourceId);
    // 레지스트리에서 등록 해제 (콜백 호출 방지)
    resourceRegistry.unregister(this);
  }
}

// 사용 예시
let resource = new ManagedResource(); // "리소스 0 할당됨"
resource.use(); // "리소스 0 사용 중"

// 명시적 정리 (권장)
resource.dispose(); // "리소스 0 명시적 정리"

// 또는 참조만 제거 (자동 정리에 의존)
resource = null;
// GC 후: "리소스 0 자동 정리"
```

### 패턴 2: WeakRef와 함께 사용하는 캐시

```javascript
class SmartCache {
  constructor() {
    this.cache = new Map(); // key -> WeakRef

    // GC 시 캐시에서 엔트리 제거
    this.registry = new FinalizationRegistry((key) => {
      console.log(`캐시 엔트리 '${key}' 자동 정리`);
      this.cache.delete(key);
    });
  }

  set(key, value) {
    // 기존 엔트리가 있으면 등록 해제
    const existingRef = this.cache.get(key);
    if (existingRef) {
      const existing = existingRef.deref();
      if (existing) {
        this.registry.unregister(existing);
      }
    }

    // WeakRef로 저장
    this.cache.set(key, new WeakRef(value));

    // 값이 GC되면 캐시에서 제거하도록 등록
    this.registry.register(value, key, value);
  }

  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;

    const value = ref.deref();
    if (value === undefined) {
      // 이미 GC됨 - FinalizationRegistry가 처리할 것이지만 여기서도 정리
      this.cache.delete(key);
    }

    return value;
  }

  delete(key) {
    const ref = this.cache.get(key);
    if (ref) {
      const value = ref.deref();
      if (value) {
        this.registry.unregister(value);
      }
    }
    return this.cache.delete(key);
  }

  get size() {
    return this.cache.size;
  }
}

// 사용 예시
const cache = new SmartCache();

function createLargeObject(id) {
  return { id, data: new Array(10000).fill(id) };
}

cache.set('item1', createLargeObject(1));
cache.set('item2', createLargeObject(2));

console.log(cache.get('item1')?.id); // 1
console.log(cache.size); // 2

// 시간이 지나 GC가 발생하면 자동으로 정리됨
```

### 패턴 3: 디버깅과 모니터링

개발 환경에서 객체 생명주기를 추적하는 데 유용합니다.

```javascript
// 개발 환경에서만 사용
const DEBUG = process.env.NODE_ENV === 'development';

class ObjectTracker {
  constructor() {
    this.createdCount = 0;
    this.gcCount = 0;
    this.registry = new FinalizationRegistry((info) => {
      this.gcCount++;
      console.log(`[ObjectTracker] GC됨: ${info.type} (ID: ${info.id})`);
      console.log(`  - 생성 시간: ${info.createdAt}`);
      console.log(`  - 생존 시간: ${Date.now() - info.createdAt}ms`);
      console.log(`  - 현재 통계: 생성 ${this.createdCount}, GC ${this.gcCount}`);
    });
  }

  track(obj, type = 'Object') {
    if (!DEBUG) return obj;

    const info = {
      id: ++this.createdCount,
      type,
      createdAt: Date.now()
    };

    this.registry.register(obj, info, obj);
    console.log(`[ObjectTracker] 생성됨: ${type} (ID: ${info.id})`);

    return obj;
  }

  untrack(obj) {
    if (!DEBUG) return;
    this.registry.unregister(obj);
  }

  getStats() {
    return {
      created: this.createdCount,
      garbageCollected: this.gcCount,
      potentiallyAlive: this.createdCount - this.gcCount
    };
  }
}

const tracker = new ObjectTracker();

// 사용 예시
let user = tracker.track({ name: '김철수' }, 'User');
let data = tracker.track(new Array(1000).fill(0), 'LargeArray');

console.log(tracker.getStats());
// { created: 2, garbageCollected: 0, potentiallyAlive: 2 }

user = null;
data = null;

// GC 후
// [ObjectTracker] GC됨: User (ID: 1)
// [ObjectTracker] GC됨: LargeArray (ID: 2)
```

---

## React에서의 활용

### 패턴 1: 컴포넌트 상태와 WeakMap

```tsx
import { useEffect, useRef, useCallback } from 'react';

// 컴포넌트 인스턴스별 메타데이터
const componentMetadata = new WeakMap<object, {
  renderCount: number;
  lastRenderTime: number;
}>();

interface UserCardProps {
  userId: string;
  name: string;
}

function UserCard({ userId, name }: UserCardProps) {
  // ref 객체를 키로 사용
  const metadataKey = useRef({});

  useEffect(() => {
    const key = metadataKey.current;

    // 메타데이터 초기화 또는 업데이트
    const existing = componentMetadata.get(key);
    if (existing) {
      existing.renderCount++;
      existing.lastRenderTime = Date.now();
    } else {
      componentMetadata.set(key, {
        renderCount: 1,
        lastRenderTime: Date.now()
      });
    }

    return () => {
      // 언마운트 시 자동 정리 (WeakMap이므로 명시적 삭제 불필요)
      console.log('컴포넌트 언마운트, 렌더 횟수:',
        componentMetadata.get(key)?.renderCount);
    };
  });

  return (
    <div className="user-card">
      <h3>{name}</h3>
      <p>ID: {userId}</p>
    </div>
  );
}

export default UserCard;
```

### 패턴 2: 이벤트 핸들러 자동 정리

```tsx
import { useEffect, useRef, useCallback } from 'react';

// 요소별 핸들러 추적
const elementHandlers = new WeakMap<Element, Map<string, Function>>();

function useAutoCleanupEvent<T extends Element>(
  eventName: string,
  handler: (event: Event) => void,
  options?: AddEventListenerOptions
) {
  const elementRef = useRef<T>(null);
  const handlerRef = useRef(handler);

  // 최신 핸들러 유지
  useEffect(() => {
    handlerRef.current = handler;
  }, [handler]);

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const wrappedHandler = (event: Event) => {
      handlerRef.current(event);
    };

    // 핸들러 등록
    element.addEventListener(eventName, wrappedHandler, options);

    // WeakMap에 추적
    if (!elementHandlers.has(element)) {
      elementHandlers.set(element, new Map());
    }
    elementHandlers.get(element)!.set(eventName, wrappedHandler);

    return () => {
      element.removeEventListener(eventName, wrappedHandler, options);
      elementHandlers.get(element)?.delete(eventName);
    };
  }, [eventName, options]);

  return elementRef;
}

// 사용 예시
function ClickTracker() {
  const buttonRef = useAutoCleanupEvent<HTMLButtonElement>('click', (e) => {
    console.log('버튼 클릭됨', e);
  });

  return (
    <button ref={buttonRef}>
      클릭하세요
    </button>
  );
}

export default ClickTracker;
```

### 패턴 3: 캐시된 API 응답

```tsx
import { useState, useEffect, useCallback } from 'react';

// API 응답 캐시 (WeakRef 사용)
class ResponseCache {
  private cache = new Map<string, WeakRef<unknown>>();
  private registry = new FinalizationRegistry<string>((key) => {
    this.cache.delete(key);
    console.log(`캐시 엔트리 '${key}' 자동 정리됨`);
  });

  set<T extends object>(key: string, value: T): void {
    const existingRef = this.cache.get(key);
    if (existingRef) {
      const existing = existingRef.deref();
      if (existing) {
        this.registry.unregister(existing);
      }
    }

    this.cache.set(key, new WeakRef(value));
    this.registry.register(value, key, value);
  }

  get<T>(key: string): T | undefined {
    const ref = this.cache.get(key);
    return ref?.deref() as T | undefined;
  }

  invalidate(key: string): void {
    const ref = this.cache.get(key);
    if (ref) {
      const value = ref.deref();
      if (value) {
        this.registry.unregister(value);
      }
    }
    this.cache.delete(key);
  }
}

const apiCache = new ResponseCache();

interface User {
  id: string;
  name: string;
  email: string;
}

function useCachedUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      const cacheKey = `user-${userId}`;

      // 캐시 확인
      const cached = apiCache.get<User>(cacheKey);
      if (cached) {
        setUser(cached);
        setLoading(false);
        return;
      }

      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data: User = await response.json();

        if (!cancelled) {
          // 캐시에 저장
          apiCache.set(cacheKey, data);
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err : new Error('Unknown error'));
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  const invalidate = useCallback(() => {
    apiCache.invalidate(`user-${userId}`);
    setUser(null);
    setLoading(true);
  }, [userId]);

  return { user, loading, error, invalidate };
}

// 컴포넌트에서 사용
function UserProfile({ userId }: { userId: string }) {
  const { user, loading, error, invalidate } = useCachedUser(userId);

  if (loading) return <div>로딩 중...</div>;
  if (error) return <div>에러: {error.message}</div>;
  if (!user) return <div>사용자를 찾을 수 없습니다.</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={invalidate}>새로고침</button>
    </div>
  );
}

export default UserProfile;
```

---

## 주의사항과 한계점

### 1. GC 타이밍의 불확실성

WeakRef와 FinalizationRegistry는 GC 타이밍에 의존합니다. GC가 언제 실행될지는 예측할 수 없습니다.

```javascript
const ref = new WeakRef(someObject);
someObject = null;

// GC가 즉시 실행되지 않을 수 있음
console.log(ref.deref()); // 여전히 객체가 존재할 수 있음

// 심지어 GC를 강제로 호출해도 (개발 환경에서)
// gc(); // Node.js --expose-gc 플래그 필요
// 여전히 객체가 존재할 수 있음
```

### 2. FinalizationRegistry 콜백 시점

콜백이 실행되는 시점은 보장되지 않습니다. 콜백이 아예 실행되지 않을 수도 있습니다.

```javascript
const registry = new FinalizationRegistry(() => {
  // 이 콜백이 언제 실행될지 보장되지 않음
  // 프로그램이 종료되기 전에 실행되지 않을 수도 있음
});

// 중요한 리소스 정리는 FinalizationRegistry에만 의존하지 말 것!
// 항상 명시적인 정리 메서드를 제공하세요.
```

### 3. 성능 고려사항

```javascript
// 나쁜 예: 모든 객체에 WeakRef 사용
function processItems(items) {
  // 불필요한 WeakRef 생성 - 오버헤드만 추가
  return items.map(item => new WeakRef(item));
}

// 좋은 예: 필요한 경우에만 사용
class LargeResourceManager {
  private cache = new Map<string, WeakRef<LargeResource>>();

  get(key: string): LargeResource | undefined {
    return this.cache.get(key)?.deref();
  }
}
```

### 4. 크로스 브라우저 지원

WeakRef와 FinalizationRegistry는 ES2021 기능입니다.

| 기능 | Chrome | Firefox | Safari | Edge | Node.js |
|------|--------|---------|--------|------|---------|
| WeakMap | 36+ | 6+ | 8+ | 12+ | 0.12+ |
| WeakSet | 36+ | 34+ | 9+ | 12+ | 0.12+ |
| WeakRef | 84+ | 79+ | 14.1+ | 84+ | 14.6+ |
| FinalizationRegistry | 84+ | 79+ | 14.1+ | 84+ | 14.6+ |

> 레거시 브라우저 지원이 필요하다면 WeakRef와 FinalizationRegistry 사용을 피하고, WeakMap/WeakSet만 사용하세요.
{: .prompt-warning }

### 5. Symbol 키 제한

ES2023부터 WeakMap과 WeakSet에서 등록되지 않은 Symbol도 키로 사용할 수 있지만, 이는 최신 환경에서만 지원됩니다.

```javascript
// ES2023+ 환경에서만 동작
const weakMap = new WeakMap();
const symbolKey = Symbol('myKey');

// 등록되지 않은 Symbol은 키로 사용 가능 (ES2023+)
weakMap.set(symbolKey, 'value');

// 등록된 Symbol은 여전히 불가
// weakMap.set(Symbol.for('registered'), 'value'); // TypeError
```

---

## 베스트 프랙티스

### 1. 명시적 정리 메서드 제공

```javascript
class ResourceHolder {
  constructor() {
    this.resource = allocateResource();
    registry.register(this, this.resource.id, this);
  }

  // 항상 명시적 정리 메서드 제공
  dispose() {
    releaseResource(this.resource);
    registry.unregister(this);
    this.resource = null;
  }
}

// 사용
const holder = new ResourceHolder();
// 사용 후 명시적 정리 (권장)
holder.dispose();
// FinalizationRegistry는 백업 용도로만 사용
```

### 2. deref() 반환값 항상 확인

```javascript
const ref = new WeakRef(obj);

// 나쁜 예
const name = ref.deref().name; // GC되었으면 에러!

// 좋은 예
const target = ref.deref();
if (target) {
  const name = target.name;
  // 작업 수행
}

// 또는 옵셔널 체이닝 사용
const name = ref.deref()?.name;
```

### 3. 적절한 사용 사례 선택

| 시나리오 | 추천 |
|----------|------|
| 객체에 메타데이터 연결 | WeakMap |
| 객체 집합 추적 (중복 방지) | WeakSet |
| 메모리 효율적 캐시 | WeakRef + FinalizationRegistry |
| 프라이빗 데이터 | WeakMap 또는 # private 필드 |
| 리소스 자동 정리 | FinalizationRegistry (백업용) |

### 4. 디버깅 어려움 인지

WeakRef와 FinalizationRegistry는 디버깅이 어렵습니다. 개발 환경에서 로깅을 추가하세요.

```javascript
class DebugWeakCache {
  constructor(name) {
    this.name = name;
    this.cache = new Map();
    this.registry = new FinalizationRegistry((key) => {
      console.log(`[${this.name}] 캐시 엔트리 GC됨: ${key}`);
      this.cache.delete(key);
    });
  }

  set(key, value) {
    console.log(`[${this.name}] 캐시 설정: ${key}`);
    this.cache.set(key, new WeakRef(value));
    this.registry.register(value, key, value);
  }

  get(key) {
    const ref = this.cache.get(key);
    const value = ref?.deref();
    console.log(`[${this.name}] 캐시 조회: ${key} -> ${value ? '히트' : '미스'}`);
    return value;
  }
}
```

---

> 더 고급 메타프로그래밍 기법에 관심이 있다면 [Proxy와 Reflect 가이드](/posts/javascript-proxy-reflect-guide/)도 참고하세요. Proxy를 WeakMap과 함께 사용하면 더욱 강력한 객체 래핑 패턴을 구현할 수 있습니다.
{: .prompt-info }

## 마무리

JavaScript의 약한 참조 메커니즘은 메모리 효율적인 프로그램을 작성하는 데 필수적인 도구입니다.

**핵심 요약:**

- **WeakMap**: 객체 키에 값을 연결하되, 키 객체가 GC될 때 자동으로 엔트리 제거
- **WeakSet**: 객체 집합을 추적하되, 객체가 GC될 때 자동으로 제거
- **WeakRef**: 단일 객체에 대한 명시적 약한 참조, deref()로 접근
- **FinalizationRegistry**: 객체 GC 시 콜백 실행 (리소스 정리용)

**사용 지침:**

1. 캐시, 메타데이터 저장에는 WeakMap/WeakSet 사용
2. 더 세밀한 제어가 필요하면 WeakRef + FinalizationRegistry 조합
3. 중요한 리소스 정리는 명시적 메서드에 의존, GC 콜백은 백업용
4. GC 타이밍의 불확실성을 항상 고려
5. 레거시 환경 지원이 필요하면 WeakRef/FinalizationRegistry 사용 자제

약한 참조를 적절히 활용하면 메모리 누수 없이 효율적인 JavaScript 애플리케이션을 구축할 수 있습니다.

---

## 참고 자료

- [MDN - WeakMap](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
- [MDN - WeakSet](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
- [MDN - WeakRef](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakRef)
- [MDN - FinalizationRegistry](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry)
- [TC39 - WeakRefs Proposal](https://github.com/tc39/proposal-weakrefs)
- [V8 Blog - Weak references and pointers](https://v8.dev/features/weak-references)
- [JavaScript.info - WeakMap and WeakSet](https://javascript.info/weakmap-weakset)
