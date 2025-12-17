---
title: "JavaScript Proxy와 Reflect 완벽 가이드 - 메타프로그래밍의 핵심"
description: "JavaScript Proxy와 Reflect API로 객체 동작을 가로채고 커스터마이징하는 메타프로그래밍 완벽 가이드. 유효성 검사, Vue 3 반응성 시스템, 로깅 등 실전 활용법."
date: 2025-12-17 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, proxy, reflect, metaprogramming, es6, vue-reactivity, mobx, immer, validation, observer-pattern, design-pattern, state-management, trap, handler]
---

## 개요

JavaScript에서 객체의 기본 동작을 가로채고 재정의할 수 있다면 어떨까요? 속성 접근, 할당, 함수 호출 등 객체의 거의 모든 동작을 커스터마이징할 수 있습니다. 이것이 바로 **Proxy**와 **Reflect**가 제공하는 메타프로그래밍 능력입니다.

**메타프로그래밍(Metaprogramming)**이란 프로그램이 자기 자신을 읽거나, 분석하거나, 수정할 수 있는 프로그래밍 기법입니다. Proxy는 ES6에서 도입되어 Vue 3의 반응성 시스템, MobX의 상태 관리, Immer의 불변성 처리 등 현대 프레임워크의 핵심 기반 기술로 자리잡았습니다.

### 학습 목표

- Proxy와 Reflect의 기본 개념과 동작 원리 이해
- 핵심 트랩(get, set, has, deleteProperty, apply 등) 활용법 습득
- 실전 활용 사례: 유효성 검사, 반응성 시스템, 로깅, 불변 객체 등
- 성능 고려사항과 베스트 프랙티스 파악

### 사전 지식

- JavaScript 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- 객체와 프로퍼티 ([JavaScript 객체 가이드](/posts/javascript-objects/) 참고)
- ES6+ 문법 (화살표 함수, 구조 분해 할당)

---

## Proxy란 무엇인가?

**Proxy**는 다른 객체(target)를 감싸서 해당 객체에 대한 기본 작업(속성 읽기, 쓰기, 열거 등)을 가로채는 객체입니다. 가로챈 작업에 대해 원하는 로직을 추가하거나 기본 동작을 변경할 수 있습니다.

### 기본 구조

```javascript
const proxy = new Proxy(target, handler);
```

- **target**: 감싸려는 원본 객체 (배열, 함수 포함 모든 객체)
- **handler**: 가로채려는 작업과 그 처리 방법을 정의하는 객체 (트랩 메서드 포함)

### 첫 번째 Proxy 예제

```javascript
const user = {
  name: '김철수',
  age: 30
};

// handler가 비어있으면 모든 작업이 target으로 그대로 전달됨
const proxy = new Proxy(user, {});

console.log(proxy.name); // '김철수' - 원본 객체처럼 동작
proxy.age = 31;
console.log(user.age);   // 31 - 원본 객체에 반영됨
```

빈 handler를 사용하면 proxy는 원본 객체의 투명한 래퍼 역할을 합니다. 이제 handler에 **트랩(trap)**을 추가하여 동작을 가로채보겠습니다.

### 트랩이란?

트랩(trap)은 객체의 내부 메서드 호출을 가로채는 handler의 메서드입니다. JavaScript 명세에서 정의한 내부 메서드들에 대응하는 트랩을 설정할 수 있습니다.

```javascript
const handler = {
  // 속성 읽기 가로채기
  get(target, property, receiver) {
    console.log(`'${property}' 속성을 읽습니다.`);
    return target[property];
  },

  // 속성 쓰기 가로채기
  set(target, property, value, receiver) {
    console.log(`'${property}' 속성에 ${value}를 씁니다.`);
    target[property] = value;
    return true; // 성공 표시
  }
};

const proxy = new Proxy(user, handler);

proxy.name;        // 콘솔: "'name' 속성을 읽습니다."
proxy.email = 'test@example.com'; // 콘솔: "'email' 속성에 test@example.com를 씁니다."
```

---

## Reflect란 무엇인가?

**Reflect**는 JavaScript의 내부 메서드들을 직접 호출할 수 있게 해주는 내장 객체입니다. Proxy의 모든 트랩에 대응하는 정적 메서드를 제공합니다.

### Reflect를 사용하는 이유

**1. 깔끔한 기본 동작 위임**

```javascript
// Reflect 없이
const handler = {
  get(target, property) {
    // 직접 접근 - 특정 상황에서 문제 발생 가능
    return target[property];
  }
};

// Reflect 사용
const handlerWithReflect = {
  get(target, property, receiver) {
    // 올바른 this 바인딩 보장
    return Reflect.get(target, property, receiver);
  }
};
```

**2. 반환값으로 성공/실패 판단**

```javascript
// Object 메서드는 실패 시 예외 발생
try {
  Object.defineProperty(obj, 'prop', descriptor);
  console.log('성공');
} catch (e) {
  console.log('실패');
}

// Reflect는 boolean 반환
if (Reflect.defineProperty(obj, 'prop', descriptor)) {
  console.log('성공');
} else {
  console.log('실패');
}
```

**3. 함수형 스타일**

```javascript
// 명령형
'prop' in obj;
delete obj.prop;

// 함수형
Reflect.has(obj, 'prop');
Reflect.deleteProperty(obj, 'prop');
```

### Reflect의 주요 메서드

| 메서드 | 설명 | 대응하는 연산 |
|-------|------|-------------|
| `Reflect.get(target, prop, receiver)` | 속성 읽기 | `target[prop]` |
| `Reflect.set(target, prop, value, receiver)` | 속성 쓰기 | `target[prop] = value` |
| `Reflect.has(target, prop)` | 속성 존재 확인 | `prop in target` |
| `Reflect.deleteProperty(target, prop)` | 속성 삭제 | `delete target[prop]` |
| `Reflect.ownKeys(target)` | 모든 키 반환 | `Object.keys` + Symbol 키 |
| `Reflect.apply(func, thisArg, args)` | 함수 호출 | `func.apply(thisArg, args)` |
| `Reflect.construct(Class, args)` | 생성자 호출 | `new Class(...args)` |

---

## 핵심 트랩 상세 설명

### 1. get 트랩: 속성 읽기 가로채기

`get` 트랩은 속성에 접근할 때 호출됩니다.

```javascript
const handler = {
  get(target, property, receiver) {
    // target: 원본 객체
    // property: 접근하려는 속성명 (string 또는 Symbol)
    // receiver: proxy 자신 또는 proxy를 상속받은 객체

    console.log(`Getting ${property}`);
    return Reflect.get(target, property, receiver);
  }
};
```

**활용 예제: 기본값 제공**

```javascript
const withDefaults = (target, defaults) => {
  return new Proxy(target, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);
      return value !== undefined ? value : defaults[property];
    }
  });
};

const config = withDefaults(
  { theme: 'dark' },
  { theme: 'light', language: 'ko', timeout: 3000 }
);

console.log(config.theme);    // 'dark' - 원본 값
console.log(config.language); // 'ko' - 기본값
console.log(config.timeout);  // 3000 - 기본값
```

### 2. set 트랩: 속성 쓰기 가로채기

`set` 트랩은 속성에 값을 할당할 때 호출됩니다. 반드시 `true` 또는 `false`를 반환해야 합니다.

```javascript
const handler = {
  set(target, property, value, receiver) {
    // value: 할당하려는 값
    // 반환값: true(성공) 또는 false(실패, strict mode에서 TypeError)

    console.log(`Setting ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  }
};
```

**활용 예제: 타입 검증**

```javascript
const typed = (target, schema) => {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const expectedType = schema[property];

      if (expectedType && typeof value !== expectedType) {
        throw new TypeError(
          `'${property}'는 ${expectedType} 타입이어야 합니다. ` +
          `받은 값: ${typeof value}`
        );
      }

      return Reflect.set(target, property, value, receiver);
    }
  });
};

const user = typed({}, {
  name: 'string',
  age: 'number',
  isActive: 'boolean'
});

user.name = '김철수';  // OK
user.age = 30;         // OK
user.age = '30';       // TypeError: 'age'는 number 타입이어야 합니다.
```

### 3. has 트랩: in 연산자 가로채기

`has` 트랩은 `in` 연산자 사용 시 호출됩니다.

```javascript
const handler = {
  has(target, property) {
    console.log(`Checking if '${property}' exists`);
    return Reflect.has(target, property);
  }
};

const obj = new Proxy({ a: 1 }, handler);
'a' in obj; // 콘솔: "Checking if 'a' exists" → true
```

**활용 예제: Private 속성 숨기기**

```javascript
const hidePrivate = (target) => {
  return new Proxy(target, {
    has(target, property) {
      // _로 시작하는 속성은 없는 것처럼 처리
      if (typeof property === 'string' && property.startsWith('_')) {
        return false;
      }
      return Reflect.has(target, property);
    },

    get(target, property, receiver) {
      if (typeof property === 'string' && property.startsWith('_')) {
        return undefined;
      }
      return Reflect.get(target, property, receiver);
    },

    ownKeys(target) {
      return Reflect.ownKeys(target).filter(
        key => typeof key !== 'string' || !key.startsWith('_')
      );
    }
  });
};

const obj = hidePrivate({
  name: '공개',
  _secret: '비밀',
  _password: '1234'
});

console.log('name' in obj);    // true
console.log('_secret' in obj); // false
console.log(obj._secret);      // undefined
console.log(Object.keys(obj)); // ['name']
```

### 4. deleteProperty 트랩: 속성 삭제 가로채기

`deleteProperty` 트랩은 `delete` 연산자 사용 시 호출됩니다.

```javascript
const handler = {
  deleteProperty(target, property) {
    console.log(`Deleting ${property}`);
    return Reflect.deleteProperty(target, property);
  }
};
```

**활용 예제: 삭제 불가 속성**

```javascript
const protectProperties = (target, protectedKeys) => {
  return new Proxy(target, {
    deleteProperty(target, property) {
      if (protectedKeys.includes(property)) {
        console.warn(`'${property}'는 삭제할 수 없습니다.`);
        return false;
      }
      return Reflect.deleteProperty(target, property);
    }
  });
};

const config = protectProperties(
  { apiKey: 'abc123', debug: true, version: '1.0' },
  ['apiKey', 'version']
);

delete config.debug;   // true - 삭제됨
delete config.apiKey;  // false - 경고 출력, 삭제 안 됨
```

### 5. apply 트랩: 함수 호출 가로채기

`apply` 트랩은 proxy 대상이 함수일 때 함수 호출을 가로챕니다.

```javascript
const handler = {
  apply(target, thisArg, argumentsList) {
    // target: 원본 함수
    // thisArg: 호출 시 this
    // argumentsList: 인자 배열

    console.log(`Called with args: ${argumentsList}`);
    return Reflect.apply(target, thisArg, argumentsList);
  }
};
```

**활용 예제: 함수 실행 시간 측정**

```javascript
const measureTime = (fn, label = fn.name || 'Anonymous') => {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const start = performance.now();
      const result = Reflect.apply(target, thisArg, args);
      const end = performance.now();

      console.log(`[${label}] 실행 시간: ${(end - start).toFixed(2)}ms`);
      return result;
    }
  });
};

const slowFunction = (n) => {
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
};

const measuredFn = measureTime(slowFunction, 'slowFunction');
measuredFn(10000000); // 콘솔: "[slowFunction] 실행 시간: 12.34ms"
```

### 6. construct 트랩: new 연산자 가로채기

`construct` 트랩은 `new` 연산자로 인스턴스 생성 시 호출됩니다.

```javascript
const handler = {
  construct(target, args, newTarget) {
    // target: 원본 생성자
    // args: 생성자에 전달된 인자
    // newTarget: new로 호출된 생성자 (상속 시 유용)

    console.log(`Creating instance with args: ${args}`);
    return Reflect.construct(target, args, newTarget);
  }
};
```

**활용 예제: 싱글톤 패턴**

```javascript
const singleton = (Class) => {
  let instance = null;

  return new Proxy(Class, {
    construct(target, args) {
      if (!instance) {
        instance = Reflect.construct(target, args);
      }
      return instance;
    }
  });
};

class Database {
  constructor(connectionString) {
    this.connection = connectionString;
    console.log('Database 연결 생성');
  }
}

const SingletonDB = singleton(Database);

const db1 = new SingletonDB('mongodb://localhost');
const db2 = new SingletonDB('mysql://localhost'); // 새 인스턴스 생성 안 됨

console.log(db1 === db2); // true
```

### 7. 기타 유용한 트랩들

```javascript
const handler = {
  // Object.getOwnPropertyDescriptor 가로채기
  getOwnPropertyDescriptor(target, property) {
    return Reflect.getOwnPropertyDescriptor(target, property);
  },

  // Object.defineProperty 가로채기
  defineProperty(target, property, descriptor) {
    return Reflect.defineProperty(target, property, descriptor);
  },

  // Object.getPrototypeOf 가로채기
  getPrototypeOf(target) {
    return Reflect.getPrototypeOf(target);
  },

  // Object.setPrototypeOf 가로채기
  setPrototypeOf(target, prototype) {
    return Reflect.setPrototypeOf(target, prototype);
  },

  // Object.keys, for...in 등에서 호출
  ownKeys(target) {
    return Reflect.ownKeys(target);
  },

  // Object.isExtensible 가로채기
  isExtensible(target) {
    return Reflect.isExtensible(target);
  },

  // Object.preventExtensions 가로채기
  preventExtensions(target) {
    return Reflect.preventExtensions(target);
  }
};
```

---

## 실전 활용 사례

### 1. 유효성 검사 (Validation)

복잡한 유효성 검사 로직을 객체에 내장할 수 있습니다.

```javascript
const createValidator = (schema) => {
  return {
    set(target, property, value, receiver) {
      const validator = schema[property];

      if (validator) {
        const { type, required, min, max, pattern, custom, message } = validator;

        // 필수 값 검사
        if (required && (value === undefined || value === null || value === '')) {
          throw new Error(message || `'${property}'는 필수 값입니다.`);
        }

        // 타입 검사
        if (type && value !== undefined && value !== null) {
          const actualType = Array.isArray(value) ? 'array' : typeof value;
          if (actualType !== type) {
            throw new TypeError(
              `'${property}'의 타입이 올바르지 않습니다. ` +
              `예상: ${type}, 실제: ${actualType}`
            );
          }
        }

        // 숫자 범위 검사
        if (typeof value === 'number') {
          if (min !== undefined && value < min) {
            throw new RangeError(`'${property}'는 ${min} 이상이어야 합니다.`);
          }
          if (max !== undefined && value > max) {
            throw new RangeError(`'${property}'는 ${max} 이하여야 합니다.`);
          }
        }

        // 문자열 패턴 검사
        if (typeof value === 'string' && pattern) {
          if (!pattern.test(value)) {
            throw new Error(message || `'${property}'의 형식이 올바르지 않습니다.`);
          }
        }

        // 커스텀 검증
        if (custom && !custom(value)) {
          throw new Error(message || `'${property}'의 값이 유효하지 않습니다.`);
        }
      }

      return Reflect.set(target, property, value, receiver);
    }
  };
};

// 사용 예시
const userSchema = {
  name: {
    type: 'string',
    required: true,
    message: '이름을 입력해주세요.'
  },
  email: {
    type: 'string',
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    message: '올바른 이메일 형식이 아닙니다.'
  },
  age: {
    type: 'number',
    min: 0,
    max: 150
  },
  password: {
    type: 'string',
    custom: (value) => value.length >= 8,
    message: '비밀번호는 8자 이상이어야 합니다.'
  }
};

const user = new Proxy({}, createValidator(userSchema));

user.name = '김철수';     // OK
user.email = 'test@example.com'; // OK
user.age = 30;            // OK

// user.email = 'invalid';  // Error: 올바른 이메일 형식이 아닙니다.
// user.age = -1;           // RangeError: 'age'는 0 이상이어야 합니다.
// user.password = '1234';  // Error: 비밀번호는 8자 이상이어야 합니다.
```

### 2. 반응성 시스템 구현 (Vue 3 스타일)

Vue 3의 반응성 시스템의 핵심 원리를 이해해봅시다.

```javascript
// 현재 실행 중인 effect를 추적
let activeEffect = null;

// 의존성 저장소: Map<target, Map<key, Set<effect>>>
const targetMap = new WeakMap();

// 의존성 추적
function track(target, key) {
  if (!activeEffect) return;

  let depsMap = targetMap.get(target);
  if (!depsMap) {
    depsMap = new Map();
    targetMap.set(target, depsMap);
  }

  let dep = depsMap.get(key);
  if (!dep) {
    dep = new Set();
    depsMap.set(key, dep);
  }

  dep.add(activeEffect);
}

// 의존성 트리거
function trigger(target, key) {
  const depsMap = targetMap.get(target);
  if (!depsMap) return;

  const dep = depsMap.get(key);
  if (dep) {
    dep.forEach(effect => effect());
  }
}

// reactive 함수: 객체를 반응형으로 만듦
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      track(target, key);
      const result = Reflect.get(target, key, receiver);

      // 중첩 객체도 반응형으로
      if (typeof result === 'object' && result !== null) {
        return reactive(result);
      }
      return result;
    },

    set(target, key, value, receiver) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value, receiver);

      // 값이 실제로 변경된 경우에만 트리거
      if (oldValue !== value) {
        trigger(target, key);
      }
      return result;
    }
  });
}

// effect 함수: 반응형 의존성을 추적하고 자동 재실행
function effect(fn) {
  const effectFn = () => {
    activeEffect = effectFn;
    fn();
    activeEffect = null;
  };

  effectFn();
  return effectFn;
}

// computed 함수: 계산된 값
function computed(getter) {
  let cachedValue;
  let dirty = true;

  const effectFn = effect(() => {
    if (dirty) {
      cachedValue = getter();
      dirty = false;
    }
  });

  return {
    get value() {
      if (dirty) {
        cachedValue = getter();
        dirty = false;
      }
      return cachedValue;
    }
  };
}

// 사용 예시
const state = reactive({
  firstName: '철수',
  lastName: '김',
  count: 0
});

// state의 속성이 변경되면 자동으로 다시 실행됨
effect(() => {
  console.log(`이름: ${state.lastName}${state.firstName}`);
});

// 콘솔: "이름: 김철수" (초기 실행)

state.firstName = '영희';
// 콘솔: "이름: 김영희" (자동 재실행)

state.lastName = '박';
// 콘솔: "이름: 박영희" (자동 재실행)

// count 변경은 effect에 영향 없음 (의존성이 없으므로)
state.count = 1; // 아무 출력 없음
```

### 3. 로깅 및 디버깅

객체의 모든 작업을 로깅하여 디버깅에 활용합니다.

```javascript
const createLogger = (target, options = {}) => {
  const {
    name = 'Object',
    logGet = true,
    logSet = true,
    logDelete = true,
    logCall = true,
    formatter = console.log
  } = options;

  return new Proxy(target, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);

      if (logGet && typeof property === 'string') {
        formatter(`[${name}] GET ${property} →`, value);
      }

      // 메서드인 경우 호출도 로깅
      if (typeof value === 'function' && logCall) {
        return new Proxy(value, {
          apply(fn, thisArg, args) {
            formatter(`[${name}] CALL ${property}(`, args, ')');
            const result = Reflect.apply(fn, thisArg, args);
            formatter(`[${name}] RETURN ${property} →`, result);
            return result;
          }
        });
      }

      return value;
    },

    set(target, property, value, receiver) {
      const oldValue = target[property];

      if (logSet && typeof property === 'string') {
        formatter(`[${name}] SET ${property}:`, oldValue, '→', value);
      }

      return Reflect.set(target, property, value, receiver);
    },

    deleteProperty(target, property) {
      if (logDelete && typeof property === 'string') {
        formatter(`[${name}] DELETE ${property}`);
      }

      return Reflect.deleteProperty(target, property);
    }
  });
};

// 사용 예시
const user = createLogger({
  name: '김철수',
  age: 30,
  greet() {
    return `안녕하세요, ${this.name}입니다!`;
  }
}, { name: 'User' });

user.name;           // [User] GET name → 김철수
user.age = 31;       // [User] SET age: 30 → 31
user.greet();        // [User] CALL greet( [] )
                     // [User] GET name → 김철수
                     // [User] RETURN greet → 안녕하세요, 김철수입니다!
delete user.age;     // [User] DELETE age
```

### 4. 불변 객체 (Immutable Object)

객체의 모든 수정 작업을 차단하여 불변성을 보장합니다.

```javascript
const deepFreeze = (obj) => {
  return new Proxy(obj, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);

      // 중첩 객체도 불변으로 만듦
      if (typeof value === 'object' && value !== null) {
        return deepFreeze(value);
      }
      return value;
    },

    set(target, property, value) {
      throw new Error(
        `Cannot modify property '${String(property)}': object is immutable`
      );
    },

    deleteProperty(target, property) {
      throw new Error(
        `Cannot delete property '${String(property)}': object is immutable`
      );
    },

    defineProperty(target, property, descriptor) {
      throw new Error(
        `Cannot define property '${String(property)}': object is immutable`
      );
    },

    setPrototypeOf(target, prototype) {
      throw new Error('Cannot change prototype: object is immutable');
    }
  });
};

const config = deepFreeze({
  api: {
    baseUrl: 'https://api.example.com',
    timeout: 5000
  },
  features: {
    darkMode: true
  }
});

console.log(config.api.baseUrl);  // 'https://api.example.com' - 읽기 가능

// config.api.timeout = 10000;    // Error: Cannot modify property 'timeout'
// delete config.features;        // Error: Cannot delete property 'features'
```

### 5. 음수 인덱스 지원 배열

Python처럼 음수 인덱스로 배열 끝에서부터 접근하는 기능을 추가합니다.

```javascript
const createNegativeArray = (arr) => {
  return new Proxy(arr, {
    get(target, property, receiver) {
      const index = Number(property);

      // 숫자 인덱스이고 음수인 경우
      if (Number.isInteger(index) && index < 0) {
        const actualIndex = target.length + index;
        return Reflect.get(target, String(actualIndex), receiver);
      }

      return Reflect.get(target, property, receiver);
    },

    set(target, property, value, receiver) {
      const index = Number(property);

      if (Number.isInteger(index) && index < 0) {
        const actualIndex = target.length + index;
        return Reflect.set(target, String(actualIndex), value, receiver);
      }

      return Reflect.set(target, property, value, receiver);
    }
  });
};

const arr = createNegativeArray([1, 2, 3, 4, 5]);

console.log(arr[-1]);  // 5 (마지막 요소)
console.log(arr[-2]);  // 4 (뒤에서 두 번째)
console.log(arr[-5]);  // 1 (첫 번째)

arr[-1] = 10;
console.log(arr);      // [1, 2, 3, 4, 10]
```

### 6. Observable 패턴

상태 변경을 감지하고 구독자에게 알리는 패턴입니다.

```javascript
const createObservable = (initialState) => {
  const subscribers = new Map();
  let state = { ...initialState };

  const proxy = new Proxy(state, {
    set(target, property, value, receiver) {
      const oldValue = target[property];
      const result = Reflect.set(target, property, value, receiver);

      if (oldValue !== value) {
        // 해당 속성의 구독자들에게 알림
        const propSubscribers = subscribers.get(property);
        if (propSubscribers) {
          propSubscribers.forEach(callback => {
            callback(value, oldValue, property);
          });
        }

        // 전체 구독자들에게도 알림
        const allSubscribers = subscribers.get('*');
        if (allSubscribers) {
          allSubscribers.forEach(callback => {
            callback({ ...target }, property, value, oldValue);
          });
        }
      }

      return result;
    }
  });

  // 구독 API
  proxy.$subscribe = (propertyOrCallback, callback) => {
    const key = typeof propertyOrCallback === 'function' ? '*' : propertyOrCallback;
    const fn = typeof propertyOrCallback === 'function' ? propertyOrCallback : callback;

    if (!subscribers.has(key)) {
      subscribers.set(key, new Set());
    }
    subscribers.get(key).add(fn);

    // unsubscribe 함수 반환
    return () => {
      subscribers.get(key).delete(fn);
    };
  };

  return proxy;
};

// 사용 예시
const store = createObservable({
  count: 0,
  user: null
});

// 특정 속성 구독
const unsubCount = store.$subscribe('count', (newVal, oldVal) => {
  console.log(`count 변경: ${oldVal} → ${newVal}`);
});

// 모든 변경 구독
store.$subscribe((state, prop, newVal, oldVal) => {
  console.log(`[전체] ${prop} 변경:`, oldVal, '→', newVal);
});

store.count = 1;
// 콘솔: "count 변경: 0 → 1"
// 콘솔: "[전체] count 변경: 0 → 1"

store.user = { name: '김철수' };
// 콘솔: "[전체] user 변경: null → { name: '김철수' }"

unsubCount(); // count 구독 해제
store.count = 2;
// 콘솔: "[전체] count 변경: 1 → 2" (count 개별 구독은 없음)
```

### 7. API 클라이언트 자동 생성

속성 접근만으로 API 호출을 자동으로 처리하는 클라이언트입니다.

```javascript
const createApiClient = (baseUrl) => {
  const buildPath = (segments) => segments.join('/');

  const createProxy = (segments = []) => {
    return new Proxy(() => {}, {
      get(target, property) {
        // HTTP 메서드인 경우
        if (['get', 'post', 'put', 'patch', 'delete'].includes(property)) {
          return async (data, options = {}) => {
            const url = `${baseUrl}/${buildPath(segments)}`;
            const config = {
              method: property.toUpperCase(),
              headers: {
                'Content-Type': 'application/json',
                ...options.headers
              },
              ...options
            };

            if (data && property !== 'get') {
              config.body = JSON.stringify(data);
            }

            const response = await fetch(url, config);

            if (!response.ok) {
              throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }

            return response.json();
          };
        }

        // 그 외에는 경로 세그먼트 추가
        return createProxy([...segments, property]);
      },

      // 함수로 호출하면 동적 세그먼트 추가
      apply(target, thisArg, args) {
        const [segment] = args;
        return createProxy([...segments, segment]);
      }
    });
  };

  return createProxy();
};

// 사용 예시
const api = createApiClient('https://api.example.com');

// GET /users
// api.users.get();

// GET /users/123
// api.users(123).get();

// POST /users
// api.users.post({ name: '김철수', email: 'kim@example.com' });

// PUT /users/123/profile
// api.users(123).profile.put({ bio: '개발자입니다' });

// DELETE /posts/456/comments/789
// api.posts(456).comments(789).delete();
```

---

## 성능 고려사항

### Proxy의 성능 영향

Proxy는 강력하지만 성능 오버헤드가 있습니다. 벤치마크 결과를 살펴봅시다.

```javascript
// 성능 측정 유틸리티
function benchmark(name, fn, iterations = 1000000) {
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    fn();
  }
  const end = performance.now();
  console.log(`${name}: ${(end - start).toFixed(2)}ms (${iterations} iterations)`);
}

const target = { value: 0 };
const proxy = new Proxy(target, {
  get(t, p) { return Reflect.get(t, p); },
  set(t, p, v) { return Reflect.set(t, p, v); }
});

// 읽기 성능 비교
benchmark('Direct read', () => target.value);
benchmark('Proxy read', () => proxy.value);

// 쓰기 성능 비교
benchmark('Direct write', () => { target.value = 1; });
benchmark('Proxy write', () => { proxy.value = 1; });
```

**일반적인 결과:**

| 작업 | 직접 접근 | Proxy 경유 | 오버헤드 |
|-----|----------|-----------|---------|
| 읽기 | ~10ms | ~50ms | 약 5배 |
| 쓰기 | ~15ms | ~60ms | 약 4배 |

### 성능 최적화 전략

**1. 불필요한 Proxy 중첩 피하기**

```javascript
// 비효율적: 매번 새 Proxy 생성
const inefficient = {
  get(target, property, receiver) {
    const value = Reflect.get(target, property, receiver);
    if (typeof value === 'object') {
      return new Proxy(value, this); // 매번 새 Proxy!
    }
    return value;
  }
};

// 효율적: Proxy 캐싱
const proxyCache = new WeakMap();

const efficient = {
  get(target, property, receiver) {
    const value = Reflect.get(target, property, receiver);

    if (typeof value === 'object' && value !== null) {
      if (!proxyCache.has(value)) {
        proxyCache.set(value, new Proxy(value, efficient));
      }
      return proxyCache.get(value);
    }
    return value;
  }
};
```

**2. 핫 패스에서 Proxy 사용 최소화**

```javascript
// 루프 내에서 Proxy 접근 최소화
const data = new Proxy(largeArray, handler);

// 비효율적
for (let i = 0; i < 10000; i++) {
  process(data[i]); // 매번 Proxy 통과
}

// 효율적
const snapshot = [...data]; // 한 번만 Proxy 통과
for (let i = 0; i < 10000; i++) {
  process(snapshot[i]); // 직접 접근
}
```

**3. 필요한 트랩만 정의**

```javascript
// 비효율적: 모든 트랩 정의
const fullHandler = {
  get(t, p) { return Reflect.get(t, p); },
  set(t, p, v) { return Reflect.set(t, p, v); },
  has(t, p) { return Reflect.has(t, p); },
  deleteProperty(t, p) { return Reflect.deleteProperty(t, p); },
  // ... 모든 트랩
};

// 효율적: 필요한 트랩만
const minimalHandler = {
  set(target, property, value) {
    console.log(`Setting ${property}`);
    return Reflect.set(target, property, value);
  }
  // 다른 작업은 자동으로 target으로 전달됨
};
```

---

## 실제 프레임워크에서의 활용

### Vue 3의 반응성 시스템

Vue 3는 Proxy를 사용하여 반응성 시스템을 구현합니다.

```javascript
// Vue 3의 reactive 함수 (단순화된 버전)
import { reactive, effect } from 'vue';

const state = reactive({
  count: 0,
  user: { name: '김철수' }
});

// effect 내에서 state 접근 시 자동으로 의존성 추적
effect(() => {
  console.log(`Count: ${state.count}`);
});

state.count++; // 자동으로 effect 재실행
```

Vue 2는 `Object.defineProperty`를 사용했지만, 여러 한계가 있었습니다:

| 기능 | Vue 2 (defineProperty) | Vue 3 (Proxy) |
|-----|----------------------|---------------|
| 새 속성 추가 감지 | 불가 (`Vue.set` 필요) | 가능 |
| 속성 삭제 감지 | 불가 (`Vue.delete` 필요) | 가능 |
| 배열 인덱스 변경 | 불가 | 가능 |
| Map, Set 지원 | 불가 | 가능 |
| 초기화 비용 | 모든 속성 순회 필요 | 지연 초기화 |

### MobX의 Observable

MobX도 Proxy를 활용하여 상태 관리를 수행합니다.

```javascript
import { makeAutoObservable, autorun } from 'mobx';

class TodoStore {
  todos = [];

  constructor() {
    makeAutoObservable(this); // Proxy로 감싸짐
  }

  addTodo(text) {
    this.todos.push({ text, done: false });
  }

  get completedCount() {
    return this.todos.filter(t => t.done).length;
  }
}

const store = new TodoStore();

autorun(() => {
  console.log(`완료: ${store.completedCount}`);
});

store.addTodo('공부하기'); // 자동 반응
```

### Immer의 불변성 관리

Immer는 Proxy를 사용하여 불변성을 쉽게 관리합니다.

```javascript
import { produce } from 'immer';

const baseState = {
  users: [
    { id: 1, name: '김철수', age: 30 },
    { id: 2, name: '이영희', age: 25 }
  ]
};

// produce 내에서는 마치 가변 객체처럼 수정 가능
const nextState = produce(baseState, draft => {
  draft.users[0].age = 31;
  draft.users.push({ id: 3, name: '박민수', age: 28 });
});

// 실제로는 불변 업데이트가 적용됨
console.log(baseState === nextState); // false
console.log(baseState.users[0] === nextState.users[0]); // false (수정됨)
console.log(baseState.users[1] === nextState.users[1]); // true (수정 안 됨)
```

---

## 주의사항과 제한사항

### 1. Proxy는 === 비교에서 원본과 다름

```javascript
const target = {};
const proxy = new Proxy(target, {});

console.log(proxy === target); // false
console.log(proxy == target);  // false

// Map, Set에서 주의
const map = new Map();
map.set(target, 'value');
console.log(map.get(proxy)); // undefined!
```

### 2. 일부 객체는 Proxy로 감쌀 수 없음

```javascript
// Date 객체의 내부 슬롯 접근 문제
const date = new Date();
const proxyDate = new Proxy(date, {});

// proxyDate.getTime(); // TypeError!

// 해결: bind를 사용하여 메서드 래핑
const safeProxyDate = new Proxy(date, {
  get(target, property, receiver) {
    const value = Reflect.get(target, property);
    if (typeof value === 'function') {
      return value.bind(target);
    }
    return value;
  }
});

console.log(safeProxyDate.getTime()); // 정상 동작
```

### 3. 내부 슬롯을 가진 내장 객체들

다음 객체들은 Proxy 사용 시 주의가 필요합니다:

- `Date`
- `Map`, `Set`, `WeakMap`, `WeakSet`
- `Array` (일부 메서드)
- `Promise`
- `RegExp`
- `TypedArray`

### 4. 취소 가능한 Proxy

필요 시 Proxy를 무효화할 수 있습니다.

```javascript
const { proxy, revoke } = Proxy.revocable({ name: '김철수' }, {
  get(target, property) {
    console.log(`Accessing ${property}`);
    return Reflect.get(target, property);
  }
});

console.log(proxy.name); // '김철수'

revoke(); // Proxy 무효화

// proxy.name; // TypeError: Cannot perform 'get' on a proxy that has been revoked
```

---

## 베스트 프랙티스

### 1. 항상 Reflect와 함께 사용

```javascript
// 권장
const handler = {
  get(target, property, receiver) {
    return Reflect.get(target, property, receiver);
  },
  set(target, property, value, receiver) {
    return Reflect.set(target, property, value, receiver);
  }
};
```

### 2. 불변성 규칙 준수

Proxy 트랩에는 불변성(invariant) 규칙이 있습니다. 이를 위반하면 TypeError가 발생합니다.

```javascript
const frozen = Object.freeze({ value: 1 });
const proxy = new Proxy(frozen, {
  get() {
    return 999; // 다른 값 반환 시도
  }
});

// proxy.value; // TypeError - frozen 객체의 속성 값과 다른 값 반환 불가
```

### 3. receiver 파라미터 활용

```javascript
const parent = {
  name: 'parent',
  get greeting() {
    return `Hello, I'm ${this.name}`;
  }
};

const child = Object.create(
  new Proxy(parent, {
    get(target, property, receiver) {
      // receiver 사용으로 올바른 this 바인딩
      return Reflect.get(target, property, receiver);
    }
  })
);

child.name = 'child';
console.log(child.greeting); // "Hello, I'm child" (receiver 덕분)
```

### 4. 타입 안전성을 위한 TypeScript 활용

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

function createValidatedProxy<T extends object>(
  target: T,
  validator: { [K in keyof T]?: (value: T[K]) => boolean }
): T {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const key = property as keyof T;
      const validate = validator[key];

      if (validate && !validate(value)) {
        throw new Error(`Invalid value for ${String(property)}`);
      }

      return Reflect.set(target, property, value, receiver);
    }
  });
}

const user = createValidatedProxy<User>(
  { name: '', age: 0, email: '' },
  {
    name: (v) => v.length > 0,
    age: (v) => v >= 0 && v <= 150,
    email: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)
  }
);
```

---

> **관련 글**: Proxy와 함께 JavaScript의 고급 개념을 더 알아보세요.
> - 클로저의 동작 원리: [JavaScript 클로저 완벽 가이드](/posts/javascript-closure-complete-guide/)
> - 프로토타입 체인: [JavaScript 프로토타입 체인 가이드](/posts/javascript-prototype-chain-guide/)
> - this 바인딩: [JavaScript this 바인딩 가이드](/posts/javascript-this-binding-guide/)
{: .prompt-tip }

## 핵심 정리

### Proxy의 핵심 개념

| 개념 | 설명 |
|------|------|
| **Proxy** | 객체의 기본 동작을 가로채는 래퍼 객체 |
| **Target** | Proxy로 감싸는 원본 객체 |
| **Handler** | 트랩 메서드를 포함하는 설정 객체 |
| **Trap** | 특정 동작을 가로채는 핸들러 메서드 |
| **Reflect** | 트랩에 대응하는 기본 동작을 제공하는 내장 객체 |

### 주요 트랩 요약

| 트랩 | 가로채는 동작 | 반환값 |
|-----|-------------|-------|
| `get` | 속성 읽기 | 속성 값 |
| `set` | 속성 쓰기 | boolean |
| `has` | `in` 연산자 | boolean |
| `deleteProperty` | `delete` 연산자 | boolean |
| `apply` | 함수 호출 | 함수 반환값 |
| `construct` | `new` 연산자 | 새 객체 |
| `ownKeys` | `Object.keys` 등 | 키 배열 |

### 활용 사례 요약

| 사례 | 설명 |
|------|------|
| **유효성 검사** | 속성 값 할당 시 자동 검증 |
| **반응성 시스템** | 속성 변경 감지 및 자동 업데이트 |
| **로깅/디버깅** | 객체 접근 및 수정 추적 |
| **불변 객체** | 모든 수정 작업 차단 |
| **Observable** | 상태 변경 구독 패턴 |
| **API 클라이언트** | 동적 메서드 생성 |

### 성능 고려사항

- Proxy는 직접 접근 대비 약 4-5배 느림
- Proxy 캐싱으로 중복 생성 방지
- 핫 패스에서는 Proxy 사용 최소화
- 필요한 트랩만 정의

> Proxy와 Reflect는 JavaScript 메타프로그래밍의 핵심 도구입니다. Vue 3, MobX, Immer 등 현대 프레임워크의 기반 기술을 이해하고 활용하면 더 강력하고 유연한 코드를 작성할 수 있습니다.
{: .prompt-info }

---

## 참고 자료

- [MDN - Proxy](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [MDN - Reflect](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
- [ECMAScript Specification - Proxy Objects](https://tc39.es/ecma262/#sec-proxy-objects)
- [JavaScript.info - Proxy와 Reflect](https://javascript.info/proxy)
- [Vue 3 Reactivity in Depth](https://vuejs.org/guide/extras/reactivity-in-depth.html)
- [Immer Documentation](https://immerjs.github.io/immer/)
- [MobX Documentation](https://mobx.js.org/README.html)
