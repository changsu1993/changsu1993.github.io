---
title: "TypeScript 5.x 고급 타입 패턴 완벽 가이드 - 실무에서 바로 쓰는 타입 안전성 전략"
date: 2025-11-22 10:00:00 +0900
categories: [Frontend, TypeScript]
tags: [typescript, typescript5, generic, utility-types, type-patterns, type-safety, 타입스크립트, 제네릭, 유틸리티타입]
description: "TypeScript 5.x의 새로운 기능부터 제네릭 고급 활용, 커스텀 유틸리티 타입, Branded Types, Exhaustive Check까지. 실무에서 타입 안전성을 높이는 모든 패턴을 예제와 함께 완벽 정리"
---

## 목차
1. [TypeScript 5.x 소개](#typescript-5x-소개)
2. [TypeScript 5.x 새로운 기능들](#typescript-5x-새로운-기능들)
3. [실무에서 자주 쓰는 유틸리티 타입](#실무에서-자주-쓰는-유틸리티-타입)
4. [제네릭 고급 활용법](#제네릭-고급-활용법)
5. [타입 안정성을 높이는 패턴들](#타입-안정성을-높이는-패턴들)
6. [실전 프로젝트 패턴](#실전-프로젝트-패턴)
7. [성능과 컴파일 최적화](#성능과-컴파일-최적화)
8. [FAQ](#faq)

---

## TypeScript 5.x 소개

TypeScript는 JavaScript에 정적 타입을 추가한 언어로, 대규모 애플리케이션 개발에서 필수적인 도구가 되었습니다. TypeScript 5.x 버전은 2023년 3월 5.0 출시를 시작으로 꾸준히 발전해왔으며, 각 버전마다 개발자 경험을 크게 향상시키는 기능들이 추가되었습니다.

### TypeScript 5.0 ~ 5.6 주요 업데이트 히스토리

| 버전 | 출시일 | 핵심 기능 |
|------|--------|-----------|
| 5.0 | 2023.03 | Decorators 정식 지원, const 타입 파라미터 |
| 5.1 | 2023.06 | 암묵적 반환 타입 개선, JSX 타입 확장 |
| 5.2 | 2023.08 | `using` 선언, Decorator Metadata |
| 5.3 | 2023.11 | Import Attributes, `switch(true)` Narrowing |
| 5.4 | 2024.03 | `NoInfer<T>`, 클로저에서 Narrowed Types 보존 |
| 5.5 | 2024.06 | Inferred Type Predicates, 정규식 문법 체크 |
| 5.6 | 2024.09 | Iterator Helper Methods, Nullish/Truthy 체크 강화 |

### 5.3, 5.4, 5.5, 5.6의 핵심 기능 요약

#### TypeScript 5.3 (2023년 11월)
- **Import Attributes**: 모듈 타입 명시적 지정 가능
- **`resolution-mode`**: import 타입에서 해석 모드 지정
- **`switch(true)` narrowing**: 패턴 매칭 스타일 타입 좁히기 지원

#### TypeScript 5.4 (2024년 3월)
- **`NoInfer<T>`**: 제네릭 추론 제어를 위한 유틸리티 타입
- **클로저에서 narrowed types 보존**: 콜백 함수 내 타입 안전성 향상
- **`Object.groupBy`/`Map.groupBy`**: 새로운 그룹화 메서드 타입 지원

#### TypeScript 5.5 (2024년 6월)
- **Inferred Type Predicates**: 타입 가드 자동 추론
- **정규식 문법 체크**: 컴파일 타임에 정규식 오류 감지
- **새로운 Set 메서드 타입**: `union()`, `intersection()`, `difference()` 등

#### TypeScript 5.6 (2024년 9월)
- **Disallowed Nullish and Truthy Checks**: 항상 참/거짓인 조건 검사 오류
- **Iterator Helper Methods**: `map()`, `filter()` 등 이터레이터 헬퍼 지원
- **Arbitrary Module Identifiers**: 모듈 식별자 유연성 향상

### TypeScript의 발전 방향

TypeScript 팀은 다음 방향으로 언어를 발전시키고 있습니다:

1. **타입 추론 강화**: 개발자가 명시적으로 타입을 지정하지 않아도 정확한 타입 추론
2. **개발자 경험 개선**: 에디터 지원, 오류 메시지 개선, 컴파일 속도 향상
3. **ECMAScript 표준 추종**: 새로운 JavaScript 기능의 빠른 지원
4. **타입 안전성 강화**: 런타임 오류를 컴파일 타임에 더 많이 잡아내기

---

## TypeScript 5.x 새로운 기능들

### 5.3: Import Attributes

Import Attributes(이전 Import Assertions)는 모듈을 import할 때 추가 메타데이터를 제공하는 방법입니다.

```typescript
// JSON 파일을 명시적으로 JSON 모듈로 import
import config from './config.json' with { type: 'json' };

// 타입만 import할 때도 사용 가능
import type { Config } from './config.json' with { type: 'json' };

// 동적 import에서도 사용
const data = await import('./data.json', { with: { type: 'json' } });
```

#### 실제 사용 사례

```typescript
// package.json에서 버전 정보 가져오기
import packageJson from '../package.json' with { type: 'json' };

console.log(`App Version: ${packageJson.version}`);

// CSS 모듈 (번들러 지원 필요)
import styles from './styles.css' with { type: 'css' };
```

### 5.3: resolution-mode in import types

import type에서 해석 모드를 명시적으로 지정할 수 있습니다.

```typescript
// Node.js ESM 방식으로 타입 해석
import type { PackageJson } from 'pkg' with { 'resolution-mode': 'import' };

// CommonJS 방식으로 타입 해석
import type { Config } from 'config' with { 'resolution-mode': 'require' };
```

### 5.3: switch(true) narrowing

`switch(true)` 패턴에서 타입 narrowing이 정확하게 동작합니다.

```typescript
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Square {
  kind: 'square';
  sideLength: number;
}

interface Triangle {
  kind: 'triangle';
  base: number;
  height: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape): number {
  // switch(true) 패턴으로 복잡한 조건 처리
  switch (true) {
    case shape.kind === 'circle':
      // shape는 Circle로 narrowing됨
      return Math.PI * shape.radius ** 2;

    case shape.kind === 'square':
      // shape는 Square로 narrowing됨
      return shape.sideLength ** 2;

    case shape.kind === 'triangle':
      // shape는 Triangle로 narrowing됨
      return (shape.base * shape.height) / 2;

    default:
      // 컴파일러가 모든 케이스를 처리했음을 인식
      const _exhaustive: never = shape;
      throw new Error(`Unknown shape: ${_exhaustive}`);
  }
}
```

### 5.4: NoInfer<T> 유틸리티 타입

`NoInfer<T>`는 제네릭 타입 추론을 제어하는 강력한 도구입니다. 특정 위치에서 타입 추론을 방지하고 싶을 때 사용합니다.

```typescript
// NoInfer가 없는 경우
function createFSM<S extends string>(config: {
  initial: S;
  states: Record<S, { onEnter?: () => void }>;
}) {}

// 문제: 'idle' | 'running' | 'typo'로 추론됨
createFSM({
  initial: 'idle',
  states: {
    idle: {},
    running: {},
    typo: {} // 오타지만 타입 추론에 포함됨
  }
});

// NoInfer 사용
function createFSMSafe<S extends string>(config: {
  initial: NoInfer<S>;  // 여기서는 추론하지 않음
  states: Record<S, { onEnter?: () => void }>;
}) {}

// 이제 states의 키만으로 S가 추론되고, initial은 그 타입을 따라야 함
createFSMSafe({
  initial: 'idle',
  states: {
    idle: {},
    running: {}
  }
});

// 오류: 'typo'는 'idle' | 'running'에 없음
createFSMSafe({
  initial: 'typo', // Error!
  states: {
    idle: {},
    running: {}
  }
});
```

#### 실전 활용: 기본값 설정 함수

```typescript
function withDefault<T>(
  value: T | undefined,
  defaultValue: NoInfer<T>  // defaultValue에서 T를 추론하지 않음
): T {
  return value ?? defaultValue;
}

// T는 value의 타입(string)으로 추론됨
const result = withDefault<string>(undefined, 'default');

// 오류: defaultValue가 value의 타입과 일치하지 않음
const wrong = withDefault('hello', 42); // Error: 42는 string이 아님
```

### 5.4: 클로저에서 narrowed types 보존

이전 버전에서는 콜백 함수 내에서 타입 narrowing이 유지되지 않는 경우가 있었습니다. 5.4에서 이 문제가 해결되었습니다.

```typescript
function processData(data: string | number | null) {
  if (data === null) {
    return;
  }

  // 이전: 콜백 내에서 data가 string | number | null로 돌아감
  // 5.4+: data가 string | number로 유지됨
  setTimeout(() => {
    console.log(data.toString()); // 이제 오류 없음!
  }, 1000);

  // 배열 메서드에서도 마찬가지
  [1, 2, 3].forEach(() => {
    if (typeof data === 'string') {
      console.log(data.toUpperCase());
    }
  });
}
```

### 5.4: Object.groupBy / Map.groupBy 타입 지원

ES2024의 새로운 그룹화 메서드에 대한 타입 지원이 추가되었습니다.

```typescript
interface Product {
  id: number;
  name: string;
  category: string;
  price: number;
}

const products: Product[] = [
  { id: 1, name: 'Laptop', category: 'electronics', price: 999 },
  { id: 2, name: 'Phone', category: 'electronics', price: 699 },
  { id: 3, name: 'Shirt', category: 'clothing', price: 29 },
  { id: 4, name: 'Pants', category: 'clothing', price: 49 }
];

// Object.groupBy - 객체로 그룹화
const groupedByCategory = Object.groupBy(products, (product) => product.category);
// 타입: Partial<Record<string, Product[]>>

console.log(groupedByCategory.electronics);
// [{ id: 1, name: 'Laptop', ... }, { id: 2, name: 'Phone', ... }]

// Map.groupBy - Map으로 그룹화
const mapByCategory = Map.groupBy(products, (product) => product.category);
// 타입: Map<string, Product[]>

for (const [category, items] of mapByCategory) {
  console.log(`${category}: ${items.length} items`);
}

// 복잡한 키로 그룹화
const groupedByPriceRange = Object.groupBy(products, (product) => {
  if (product.price < 50) return 'budget';
  if (product.price < 500) return 'mid';
  return 'premium';
});
```

### 5.5: Inferred Type Predicates

TypeScript 5.5는 타입 가드 함수의 반환 타입을 자동으로 추론합니다.

```typescript
// 5.5 이전: 명시적 타입 가드 필요
function isStringOld(value: unknown): value is string {
  return typeof value === 'string';
}

// 5.5+: 타입 가드가 자동으로 추론됨!
function isString(value: unknown) {
  return typeof value === 'string';
}
// 추론된 타입: (value: unknown) => value is string

// 배열 필터링에서 자동으로 타입이 좁혀짐
const mixed: (string | number | null)[] = ['a', 1, null, 'b', 2];

const strings = mixed.filter((item) => typeof item === 'string');
// 5.5+: strings의 타입은 string[]
// 이전: (string | number | null)[]
```

#### 복잡한 타입 가드 자동 추론

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface Admin extends User {
  role: 'admin';
  permissions: string[];
}

// 자동으로 타입 가드로 인식됨
function isAdmin(user: User | Admin) {
  return 'role' in user && user.role === 'admin';
}
// 추론: (user: User | Admin) => user is Admin

const users: (User | Admin)[] = [
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com', role: 'admin', permissions: ['read', 'write'] }
];

const admins = users.filter(isAdmin);
// admins의 타입: Admin[]
```

### 5.5: 정규식 문법 체크

TypeScript 5.5는 정규식 리터럴의 문법을 컴파일 타임에 검사합니다.

```typescript
// 컴파일 오류: 잘못된 정규식
const badRegex = /[a-Z]/;  // Error: Invalid character class range

// 올바른 정규식
const goodRegex = /[a-zA-Z]/;

// 잘못된 그룹 참조
const badGroup = /(\w+) \2/;  // Error: \2 참조하지만 그룹이 1개뿐

// 올바른 그룹 참조
const goodGroup = /(\w+) \1/;

// 잘못된 플래그 조합
const badFlags = /test/uu;  // Error: 중복 플래그
```

### 5.5: 새로운 Set 메서드 타입

ES2025의 새로운 Set 메서드들에 대한 타입이 추가되었습니다.

```typescript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// 합집합
const union = setA.union(setB);
// Set { 1, 2, 3, 4, 5, 6 }

// 교집합
const intersection = setA.intersection(setB);
// Set { 3, 4 }

// 차집합
const difference = setA.difference(setB);
// Set { 1, 2 }

// 대칭 차집합
const symmetricDifference = setA.symmetricDifference(setB);
// Set { 1, 2, 5, 6 }

// 부분집합/상위집합 체크
const isSubset = setA.isSubsetOf(new Set([1, 2, 3, 4, 5]));
const isSuperset = setA.isSupersetOf(new Set([1, 2]));
const isDisjoint = setA.isDisjointFrom(new Set([7, 8, 9]));
```

### 5.6: Disallowed Nullish and Truthy Checks

TypeScript 5.6은 항상 참이거나 거짓인 조건문을 오류로 처리합니다.

```typescript
function example(value: string) {
  // 오류: 이 조건은 항상 참
  if (value !== null) {  // string 타입은 null이 될 수 없음
    console.log(value);
  }

  // 오류: 이 조건은 항상 거짓
  if (value === undefined) {  // string 타입은 undefined가 될 수 없음
    console.log('never');
  }
}

// 실수 방지 예시
function processUser(user: { name: string }) {
  // 5.6 이전: 경고 없음
  // 5.6+: 오류 - user.name은 항상 truthy가 아닐 수 있음 (빈 문자열)
  if (user) {  // 이 조건은 항상 참 (user는 객체)
    console.log(user.name);
  }
}
```

### 5.6: Iterator Helper Methods

ES2025 Iterator Helpers에 대한 타입 지원이 추가되었습니다.

```typescript
// 이터레이터 생성
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
}

// Iterator Helper Methods 사용
const doubled = numbers()
  .map(n => n * 2)
  .filter(n => n > 4)
  .take(2)
  .toArray();
// [6, 8]

// 무한 이터레이터도 안전하게 처리
function* infiniteNumbers() {
  let n = 0;
  while (true) {
    yield n++;
  }
}

const firstTen = infiniteNumbers()
  .take(10)
  .toArray();
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// drop과 take 조합
const middleNumbers = infiniteNumbers()
  .drop(5)
  .take(5)
  .toArray();
// [5, 6, 7, 8, 9]
```

---

## 실무에서 자주 쓰는 유틸리티 타입

### 기본 유틸리티 타입 심화

#### Partial<T>: 모든 속성을 선택적으로

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// 모든 속성이 선택적
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// 실제 사용: 부분 업데이트
function updateUser(id: number, updates: Partial<User>): User {
  const user = getUser(id);
  return { ...user, ...updates };
}

updateUser(1, { name: 'New Name' });  // name만 업데이트
updateUser(1, { email: 'new@email.com', age: 30 });  // email, age 업데이트
```

#### Required<T>: 모든 속성을 필수로

```typescript
interface Config {
  host?: string;
  port?: number;
  ssl?: boolean;
}

// 모든 속성이 필수
type RequiredConfig = Required<Config>;
// { host: string; port: number; ssl: boolean; }

// 실제 사용: 기본값 적용 후 타입
function createServer(config: Config): RequiredConfig {
  return {
    host: config.host ?? 'localhost',
    port: config.port ?? 3000,
    ssl: config.ssl ?? false
  };
}
```

#### Pick<T, K>: 특정 속성만 선택

```typescript
interface Article {
  id: number;
  title: string;
  content: string;
  author: string;
  createdAt: Date;
  updatedAt: Date;
}

// 목록에 필요한 속성만 선택
type ArticleListItem = Pick<Article, 'id' | 'title' | 'author' | 'createdAt'>;

// API 응답 타입 정의
async function getArticleList(): Promise<ArticleListItem[]> {
  const response = await fetch('/api/articles');
  return response.json();
}
```

#### Omit<T, K>: 특정 속성 제외

```typescript
interface FullProduct {
  id: number;
  name: string;
  price: number;
  description: string;
  internalCode: string;  // 내부용
  costPrice: number;     // 내부용
}

// 외부에 노출할 타입 (내부 정보 제외)
type PublicProduct = Omit<FullProduct, 'internalCode' | 'costPrice'>;

// 생성용 타입 (id 제외)
type CreateProductDTO = Omit<FullProduct, 'id'>;
```

#### Record<K, T>: 키-값 맵 타입

```typescript
// 문자열 키, 특정 값 타입
type StatusMessages = Record<'success' | 'error' | 'loading', string>;

const messages: StatusMessages = {
  success: '성공적으로 처리되었습니다.',
  error: '오류가 발생했습니다.',
  loading: '처리 중입니다...'
};

// 객체를 딕셔너리로 사용
type UserMap = Record<string, User>;

const users: UserMap = {
  'user-1': { id: 1, name: 'John', email: 'john@test.com', age: 30 },
  'user-2': { id: 2, name: 'Jane', email: 'jane@test.com', age: 25 }
};

// 열거형 키와 함께 사용
enum Permission {
  Read = 'READ',
  Write = 'WRITE',
  Delete = 'DELETE'
}

type PermissionDescriptions = Record<Permission, string>;

const descriptions: PermissionDescriptions = {
  [Permission.Read]: '데이터를 읽을 수 있습니다.',
  [Permission.Write]: '데이터를 수정할 수 있습니다.',
  [Permission.Delete]: '데이터를 삭제할 수 있습니다.'
};
```

### 고급 유틸리티 타입

#### ReturnType<T>: 함수 반환 타입 추출

```typescript
function fetchUser(id: number) {
  return {
    id,
    name: 'John',
    email: 'john@test.com',
    createdAt: new Date()
  };
}

// 함수의 반환 타입 추출
type User = ReturnType<typeof fetchUser>;
// { id: number; name: string; email: string; createdAt: Date; }

// 비동기 함수의 경우
async function fetchUserAsync(id: number) {
  const response = await fetch(`/api/users/${id}`);
  return response.json() as Promise<{ id: number; name: string }>;
}

type AsyncUser = Awaited<ReturnType<typeof fetchUserAsync>>;
// { id: number; name: string; }
```

#### Parameters<T>: 함수 매개변수 타입 추출

```typescript
function createUser(name: string, email: string, age?: number) {
  return { name, email, age };
}

// 함수의 매개변수 타입을 튜플로 추출
type CreateUserParams = Parameters<typeof createUser>;
// [name: string, email: string, age?: number]

// 특정 매개변수만 추출
type FirstParam = Parameters<typeof createUser>[0];  // string
type SecondParam = Parameters<typeof createUser>[1]; // string

// 래퍼 함수 만들기
function loggedCreateUser(...args: Parameters<typeof createUser>) {
  console.log('Creating user with:', args);
  return createUser(...args);
}
```

#### Awaited<T>: Promise 해제 타입

```typescript
type PromiseString = Promise<string>;
type ResolvedString = Awaited<PromiseString>;  // string

// 중첩된 Promise도 처리
type NestedPromise = Promise<Promise<Promise<number>>>;
type ResolvedNumber = Awaited<NestedPromise>;  // number

// 실제 활용: API 응답 타입
async function fetchData(): Promise<{ data: string[] }> {
  return { data: ['a', 'b', 'c'] };
}

type FetchResult = Awaited<ReturnType<typeof fetchData>>;
// { data: string[] }
```

#### NoInfer<T>: 타입 추론 제어 (5.4+)

```typescript
// 이벤트 시스템 예제
function on<T extends string>(
  event: T,
  callback: (data: NoInfer<T>) => void
) {}

// T는 첫 번째 인자에서만 추론됨
on('click', (data) => {
  // data의 타입은 'click'
});

// 상태 머신 예제
function createStateMachine<State extends string>(config: {
  initial: NoInfer<State>;
  states: Record<State, { on?: Record<string, State> }>;
}) {}

createStateMachine({
  initial: 'idle',  // 'states'의 키에서 추론된 타입이어야 함
  states: {
    idle: { on: { START: 'running' } },
    running: { on: { STOP: 'idle' } }
  }
});
```

### 조건부 타입

#### Exclude<T, U>: T에서 U 제외

```typescript
type AllTypes = string | number | boolean | null | undefined;
type PrimitiveTypes = Exclude<AllTypes, null | undefined>;
// string | number | boolean

// 실제 활용: 특정 상태 제외
type Status = 'pending' | 'approved' | 'rejected' | 'cancelled';
type ActiveStatus = Exclude<Status, 'cancelled'>;
// 'pending' | 'approved' | 'rejected'
```

#### Extract<T, U>: T에서 U만 추출

```typescript
type AllTypes = string | number | boolean | (() => void);
type Functions = Extract<AllTypes, Function>;
// () => void

// 실제 활용: 특정 패턴의 키만 추출
type Keys = 'onClick' | 'onSubmit' | 'className' | 'style';
type EventKeys = Extract<Keys, `on${string}`>;
// 'onClick' | 'onSubmit'
```

#### NonNullable<T>: null과 undefined 제거

```typescript
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// string

// 실제 활용: API 응답 정제
interface ApiResponse {
  data?: {
    user?: {
      name: string;
      email?: string | null;
    } | null;
  } | null;
}

function processUser(response: ApiResponse) {
  const user = response.data?.user;
  if (user) {
    // user의 타입: { name: string; email?: string | null }
    const name: string = user.name;
    const email: NonNullable<typeof user.email> | undefined =
      user.email ?? undefined;
  }
}
```

### 문자열 조작 타입

```typescript
// 대문자 변환
type Uppercase<T extends string> = ...
type Upper = Uppercase<'hello'>;  // 'HELLO'

// 소문자 변환
type Lowercase<T extends string> = ...
type Lower = Lowercase<'HELLO'>;  // 'hello'

// 첫 글자만 대문자
type Capitalize<T extends string> = ...
type Cap = Capitalize<'hello'>;  // 'Hello'

// 첫 글자만 소문자
type Uncapitalize<T extends string> = ...
type Uncap = Uncapitalize<'HELLO'>;  // 'hELLO'

// 실제 활용: 이벤트 핸들러 타입 생성
type EventName = 'click' | 'change' | 'submit';
type EventHandler<T extends string> = `on${Capitalize<T>}`;
type Handlers = EventHandler<EventName>;
// 'onClick' | 'onChange' | 'onSubmit'

// 실제 활용: API 엔드포인트 타입
type Resource = 'user' | 'post' | 'comment';
type Endpoint = `/${Lowercase<Resource>}s`;
// '/users' | '/posts' | '/comments'
```

### 커스텀 유틸리티 타입 만들기 (실전 예제)

#### 1. DeepPartial: 중첩 객체도 모두 선택적으로

```typescript
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

// 사용 예시
interface DeepConfig {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    url: string;
    poolSize: number;
  };
}

type PartialConfig = DeepPartial<DeepConfig>;
// 모든 중첩 속성이 선택적

const partialConfig: PartialConfig = {
  server: {
    ssl: {
      enabled: true
      // cert는 선택적
    }
    // host, port는 선택적
  }
  // database는 선택적
};
```

#### 2. DeepReadonly: 중첩 객체도 모두 읽기 전용

```typescript
type DeepReadonly<T> = T extends (infer U)[]
  ? ReadonlyArray<DeepReadonly<U>>
  : T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

// 사용 예시
interface AppState {
  user: {
    name: string;
    preferences: {
      theme: 'light' | 'dark';
      notifications: boolean;
    };
  };
  posts: { id: number; title: string }[];
}

type ImmutableState = DeepReadonly<AppState>;

const state: ImmutableState = {
  user: {
    name: 'John',
    preferences: { theme: 'dark', notifications: true }
  },
  posts: [{ id: 1, title: 'Hello' }]
};

// state.user.name = 'Jane';  // 오류!
// state.posts.push({ id: 2, title: 'World' });  // 오류!
```

#### 3. RequiredKeys / OptionalKeys: 필수/선택 키 추출

```typescript
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// 사용 예시
interface User {
  id: number;
  name: string;
  email?: string;
  age?: number;
}

type Required = RequiredKeys<User>;  // 'id' | 'name'
type Optional = OptionalKeys<User>;  // 'email' | 'age'
```

#### 4. Mutable: Readonly 제거

```typescript
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// 중첩된 경우
type DeepMutable<T> = T extends object
  ? { -readonly [P in keyof T]: DeepMutable<T[P]> }
  : T;

// 사용 예시
interface ReadonlyUser {
  readonly id: number;
  readonly name: string;
}

type MutableUser = Mutable<ReadonlyUser>;
// { id: number; name: string; }
```

#### 5. PickByType: 특정 타입의 속성만 선택

```typescript
type PickByType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

// 사용 예시
interface Mixed {
  id: number;
  name: string;
  active: boolean;
  count: number;
  description: string;
}

type StringProps = PickByType<Mixed, string>;
// { name: string; description: string; }

type NumberProps = PickByType<Mixed, number>;
// { id: number; count: number; }
```

#### 6. OmitByType: 특정 타입의 속성 제외

```typescript
type OmitByType<T, V> = {
  [K in keyof T as T[K] extends V ? never : K]: T[K];
};

// 사용 예시
type WithoutStrings = OmitByType<Mixed, string>;
// { id: number; active: boolean; count: number; }
```

#### 7. UnionToIntersection: 유니온을 교차 타입으로

```typescript
type UnionToIntersection<U> = (
  U extends any ? (x: U) => void : never
) extends (x: infer I) => void
  ? I
  : never;

// 사용 예시
type Union = { a: string } | { b: number } | { c: boolean };
type Intersection = UnionToIntersection<Union>;
// { a: string } & { b: number } & { c: boolean }
```

---

## 제네릭 고급 활용법

### 제네릭 기본 복습과 심화

제네릭은 타입을 매개변수화하여 재사용 가능한 컴포넌트를 만드는 핵심 기능입니다.

```typescript
// 기본 제네릭 함수
function identity<T>(value: T): T {
  return value;
}

// 타입 추론
const str = identity('hello');  // string으로 추론
const num = identity(42);       // number로 추론

// 명시적 타입 지정
const explicit = identity<string>('hello');

// 제네릭 인터페이스
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

// 제네릭 클래스
class Box<T> {
  constructor(private value: T) {}

  get(): T {
    return this.value;
  }

  set(value: T): void {
    this.value = value;
  }
}
```

### 제네릭 제약조건 (extends)

```typescript
// 특정 속성을 가진 타입으로 제한
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(value: T): T {
  console.log(value.length);
  return value;
}

logLength('hello');      // OK: string has length
logLength([1, 2, 3]);    // OK: array has length
logLength({ length: 5 }); // OK: object has length
// logLength(42);        // Error: number doesn't have length

// 객체 키 제약
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'John', age: 30 };
const name = getProperty(user, 'name');  // string
const age = getProperty(user, 'age');    // number
// getProperty(user, 'email');  // Error: 'email' is not a key of user
```

### 다중 제네릭과 기본값

```typescript
// 다중 제네릭
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: 'John' }, { age: 30 });
// { name: string; age: number }

// 제네릭 기본값
interface ApiResponse<T = unknown, E = Error> {
  data: T | null;
  error: E | null;
  status: number;
}

// 기본값 사용
const response1: ApiResponse = { data: null, error: null, status: 200 };

// 명시적 지정
const response2: ApiResponse<User> = {
  data: { id: 1, name: 'John', email: 'john@test.com', age: 30 },
  error: null,
  status: 200
};

// 에러 타입도 지정
interface CustomError {
  code: string;
  message: string;
}

const response3: ApiResponse<User, CustomError> = {
  data: null,
  error: { code: 'NOT_FOUND', message: 'User not found' },
  status: 404
};
```

### 조건부 타입과 제네릭

```typescript
// 기본 조건부 타입
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// 조건부 타입으로 타입 변환
type Flatten<T> = T extends Array<infer U> ? U : T;

type Str = Flatten<string[]>;     // string
type Num = Flatten<number>;       // number
type Mixed = Flatten<(string | number)[]>;  // string | number

// 실제 활용: 응답 타입 추출
type ApiEndpoints = {
  '/users': User[];
  '/users/:id': User;
  '/posts': Post[];
  '/posts/:id': Post;
};

type GetResponse<T extends keyof ApiEndpoints> = ApiEndpoints[T];

type UsersResponse = GetResponse<'/users'>;  // User[]
type UserResponse = GetResponse<'/users/:id'>;  // User
```

### infer 키워드 마스터하기

`infer`는 조건부 타입 내에서 타입을 추론하고 캡처하는 강력한 도구입니다.

```typescript
// 함수 반환 타입 추출 (ReturnType 구현)
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// 함수 매개변수 타입 추출 (Parameters 구현)
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

// 첫 번째 매개변수만 추출
type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any
  ? F
  : never;

function greet(name: string, age: number) {
  return `Hello, ${name}`;
}

type FirstArgType = FirstArg<typeof greet>;  // string

// Promise 내부 타입 추출 (Awaited 구현)
type UnwrapPromise<T> = T extends Promise<infer U>
  ? UnwrapPromise<U>  // 재귀적으로 중첩 Promise 해제
  : T;

type Resolved = UnwrapPromise<Promise<Promise<string>>>;  // string

// 배열 요소 타입 추출
type ArrayElement<T> = T extends (infer E)[] ? E : T;

type Element = ArrayElement<string[]>;  // string

// 객체의 특정 메서드 반환 타입 추출
type MethodReturnType<T, M extends keyof T> = T[M] extends (...args: any[]) => infer R
  ? R
  : never;

interface UserService {
  getUser(id: number): Promise<User>;
  createUser(data: Partial<User>): Promise<User>;
  deleteUser(id: number): Promise<boolean>;
}

type GetUserReturn = MethodReturnType<UserService, 'getUser'>;
// Promise<User>
```

### Mapped Types와 제네릭 결합

```typescript
// 모든 속성을 특정 타입으로 변환
type AllStrings<T> = {
  [K in keyof T]: string;
};

interface User {
  id: number;
  name: string;
  active: boolean;
}

type UserStrings = AllStrings<User>;
// { id: string; name: string; active: string; }

// 속성 이름 변환
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getId: () => number; getName: () => string; getActive: () => boolean; }

// Setters 생성
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type UserSetters = Setters<User>;
// { setId: (value: number) => void; setName: (value: string) => void; ... }

// 특정 속성만 선택적으로 만들기
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = PartialBy<User, 'id'>;
// { name: string; active: boolean; id?: number; }

// 특정 속성만 필수로 만들기
type RequiredBy<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;
```

### 재귀적 타입 정의

```typescript
// JSON 타입 정의
type JsonPrimitive = string | number | boolean | null;
type JsonArray = JsonValue[];
type JsonObject = { [key: string]: JsonValue };
type JsonValue = JsonPrimitive | JsonArray | JsonObject;

// 사용 예시
const jsonData: JsonValue = {
  name: 'John',
  age: 30,
  tags: ['developer', 'typescript'],
  address: {
    city: 'Seoul',
    zip: null
  }
};

// 중첩 경로 타입
type Path<T, K extends keyof T = keyof T> = K extends string
  ? T[K] extends Record<string, any>
    ? K | `${K}.${Path<T[K]>}`
    : K
  : never;

interface NestedObject {
  a: {
    b: {
      c: string;
    };
    d: number;
  };
  e: boolean;
}

type Paths = Path<NestedObject>;
// 'a' | 'e' | 'a.b' | 'a.d' | 'a.b.c'
```

### 실전 패턴: API 응답 타입 추론

```typescript
// API 엔드포인트 정의
interface ApiDefinition {
  '/users': {
    GET: { response: User[] };
    POST: { body: CreateUserDTO; response: User };
  };
  '/users/:id': {
    GET: { response: User };
    PUT: { body: UpdateUserDTO; response: User };
    DELETE: { response: void };
  };
}

// 응답 타입 추출
type ApiResponse<
  Path extends keyof ApiDefinition,
  Method extends keyof ApiDefinition[Path]
> = ApiDefinition[Path][Method] extends { response: infer R } ? R : never;

// 요청 바디 타입 추출
type ApiBody<
  Path extends keyof ApiDefinition,
  Method extends keyof ApiDefinition[Path]
> = ApiDefinition[Path][Method] extends { body: infer B } ? B : never;

// 사용 예시
type UsersGetResponse = ApiResponse<'/users', 'GET'>;  // User[]
type UserPostBody = ApiBody<'/users', 'POST'>;  // CreateUserDTO

// 타입 안전한 fetch 래퍼
async function api<
  P extends keyof ApiDefinition,
  M extends keyof ApiDefinition[P]
>(
  path: P,
  method: M,
  body?: ApiBody<P, M>
): Promise<ApiResponse<P, M>> {
  const response = await fetch(path as string, {
    method: method as string,
    body: body ? JSON.stringify(body) : undefined
  });
  return response.json();
}
```

### 실전 패턴: 폼 유효성 검사 타입

```typescript
// 검증 규칙 정의
type ValidationRule<T> = {
  validate: (value: T) => boolean;
  message: string;
};

// 필드별 검증 스키마
type ValidationSchema<T> = {
  [K in keyof T]?: ValidationRule<T[K]>[];
};

// 검증 결과
type ValidationResult<T> = {
  [K in keyof T]?: string[];
};

// 폼 데이터 타입
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

// 검증 스키마 정의
const loginSchema: ValidationSchema<LoginForm> = {
  email: [
    {
      validate: (v) => v.length > 0,
      message: '이메일을 입력해주세요.'
    },
    {
      validate: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
      message: '올바른 이메일 형식이 아닙니다.'
    }
  ],
  password: [
    {
      validate: (v) => v.length >= 8,
      message: '비밀번호는 8자 이상이어야 합니다.'
    }
  ]
};

// 검증 함수
function validate<T extends Record<string, any>>(
  data: T,
  schema: ValidationSchema<T>
): ValidationResult<T> {
  const result: ValidationResult<T> = {};

  for (const key in schema) {
    const rules = schema[key];
    if (!rules) continue;

    const errors: string[] = [];
    for (const rule of rules) {
      if (!rule.validate(data[key])) {
        errors.push(rule.message);
      }
    }

    if (errors.length > 0) {
      result[key] = errors;
    }
  }

  return result;
}
```

---

## 타입 안정성을 높이는 패턴들

### Branded Types (Nominal Typing)

TypeScript는 구조적 타입 시스템을 사용하므로, 같은 구조의 타입은 호환됩니다. Branded Types를 사용하면 구조는 같지만 의미가 다른 타입을 구분할 수 있습니다.

```typescript
// 브랜드 타입 정의
declare const brand: unique symbol;

type Brand<T, B> = T & { [brand]: B };

// 특정 도메인 타입 정의
type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;
type Email = Brand<string, 'Email'>;

// 생성 함수 (유효성 검사 포함)
function createUserId(id: string): UserId {
  if (!id.startsWith('user_')) {
    throw new Error('Invalid user ID format');
  }
  return id as UserId;
}

function createEmail(email: string): Email {
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    throw new Error('Invalid email format');
  }
  return email as Email;
}

// 사용 예시
function getUser(id: UserId): User {
  // ...
  return {} as User;
}

function getPost(id: PostId): Post {
  // ...
  return {} as Post;
}

const userId = createUserId('user_123');
const postId = 'post_456' as PostId;

getUser(userId);  // OK
// getUser(postId);  // Error: PostId는 UserId에 할당할 수 없음
// getUser('user_123');  // Error: string은 UserId에 할당할 수 없음
```

#### 실전 예제: 금액 타입

```typescript
type KRW = Brand<number, 'KRW'>;
type USD = Brand<number, 'USD'>;

function krw(amount: number): KRW {
  return amount as KRW;
}

function usd(amount: number): USD {
  return amount as USD;
}

function addKRW(a: KRW, b: KRW): KRW {
  return (a + b) as KRW;
}

const price1 = krw(10000);
const price2 = krw(5000);
const total = addKRW(price1, price2);  // OK

const dollars = usd(100);
// addKRW(price1, dollars);  // Error: USD는 KRW에 할당할 수 없음
```

### Exhaustive Check

switch문에서 모든 케이스를 처리했는지 컴파일 타임에 확인합니다.

```typescript
type Status = 'pending' | 'approved' | 'rejected';

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

function getStatusMessage(status: Status): string {
  switch (status) {
    case 'pending':
      return '검토 중입니다.';
    case 'approved':
      return '승인되었습니다.';
    case 'rejected':
      return '거절되었습니다.';
    default:
      // 모든 케이스를 처리하면 여기에 도달하지 않음
      return assertNever(status);
  }
}

// 새로운 상태가 추가되면 컴파일 오류 발생
type NewStatus = 'pending' | 'approved' | 'rejected' | 'cancelled';

function getNewStatusMessage(status: NewStatus): string {
  switch (status) {
    case 'pending':
      return '검토 중입니다.';
    case 'approved':
      return '승인되었습니다.';
    case 'rejected':
      return '거절되었습니다.';
    // 'cancelled' 케이스가 없으면 컴파일 오류!
    // case 'cancelled':
    //   return '취소되었습니다.';
    default:
      return assertNever(status);
      // Error: 'cancelled' is not assignable to 'never'
  }
}
```

### Type Guards

#### 기본 타입 가드

```typescript
// typeof 가드
function processValue(value: string | number) {
  if (typeof value === 'string') {
    // value는 string
    console.log(value.toUpperCase());
  } else {
    // value는 number
    console.log(value.toFixed(2));
  }
}

// instanceof 가드
class Dog {
  bark() { console.log('Woof!'); }
}

class Cat {
  meow() { console.log('Meow!'); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// in 가드
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(animal: Bird | Fish) {
  if ('fly' in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

#### 사용자 정의 타입 가드 (is 키워드)

```typescript
interface User {
  type: 'user';
  name: string;
  email: string;
}

interface Admin {
  type: 'admin';
  name: string;
  email: string;
  permissions: string[];
}

// 타입 가드 함수
function isAdmin(person: User | Admin): person is Admin {
  return person.type === 'admin';
}

function processAccess(person: User | Admin) {
  if (isAdmin(person)) {
    // person은 Admin
    console.log(`Admin permissions: ${person.permissions.join(', ')}`);
  } else {
    // person은 User
    console.log(`Regular user: ${person.name}`);
  }
}

// 배열 필터링에서 타입 가드 활용
const people: (User | Admin)[] = [
  { type: 'user', name: 'John', email: 'john@test.com' },
  { type: 'admin', name: 'Jane', email: 'jane@test.com', permissions: ['read', 'write'] }
];

const admins = people.filter(isAdmin);  // Admin[]
```

#### assertion 함수

```typescript
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function processInput(input: unknown) {
  assertIsString(input);
  // 이후로 input은 string
  console.log(input.toUpperCase());
}

// null/undefined 검사
function assertDefined<T>(value: T): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error('Value is null or undefined');
  }
}

function process(data: string | null | undefined) {
  assertDefined(data);
  // data는 string
  console.log(data.length);
}
```

### Discriminated Unions (태그된 유니온)

```typescript
// 공통 판별자 속성을 가진 타입들
interface LoadingState {
  status: 'loading';
}

interface SuccessState<T> {
  status: 'success';
  data: T;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// 타입 안전한 상태 처리
function renderData<T>(state: AsyncState<T>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    case 'success':
      return `Data: ${JSON.stringify(state.data)}`;
    case 'error':
      return `Error: ${state.error}`;
  }
}

// 실전 예제: Redux 액션 타입
interface FetchUserRequest {
  type: 'FETCH_USER_REQUEST';
}

interface FetchUserSuccess {
  type: 'FETCH_USER_SUCCESS';
  payload: User;
}

interface FetchUserFailure {
  type: 'FETCH_USER_FAILURE';
  error: string;
}

type UserAction = FetchUserRequest | FetchUserSuccess | FetchUserFailure;

function userReducer(state: User | null, action: UserAction): User | null {
  switch (action.type) {
    case 'FETCH_USER_REQUEST':
      return null;
    case 'FETCH_USER_SUCCESS':
      return action.payload;
    case 'FETCH_USER_FAILURE':
      console.error(action.error);
      return null;
  }
}
```

### Const Assertions (as const)

```typescript
// as const 없이
const config1 = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};
// 타입: { apiUrl: string; timeout: number; retries: number; }

// as const 사용
const config2 = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
} as const;
// 타입: { readonly apiUrl: 'https://api.example.com'; readonly timeout: 5000; readonly retries: 3; }

// 배열에서의 차이
const colors1 = ['red', 'green', 'blue'];
// 타입: string[]

const colors2 = ['red', 'green', 'blue'] as const;
// 타입: readonly ['red', 'green', 'blue']

// 튜플 타입 생성
const point = [10, 20] as const;  // readonly [10, 20]

// 열거형 대안으로 사용
const Status = {
  Pending: 'PENDING',
  Approved: 'APPROVED',
  Rejected: 'REJECTED'
} as const;

type StatusType = typeof Status[keyof typeof Status];
// 'PENDING' | 'APPROVED' | 'REJECTED'
```

### Template Literal Types

```typescript
// 기본 템플릿 리터럴 타입
type Greeting = `Hello, ${string}!`;

const greet1: Greeting = 'Hello, World!';  // OK
const greet2: Greeting = 'Hello, TypeScript!';  // OK
// const greet3: Greeting = 'Hi, World!';  // Error

// 조합 타입
type Color = 'red' | 'green' | 'blue';
type Size = 'small' | 'medium' | 'large';

type ClassName = `${Size}-${Color}`;
// 'small-red' | 'small-green' | 'small-blue' | 'medium-red' | ...

// 이벤트 핸들러 타입
type ElementEvents = 'click' | 'focus' | 'blur' | 'change';
type EventHandler = `on${Capitalize<ElementEvents>}`;
// 'onClick' | 'onFocus' | 'onBlur' | 'onChange'

// CSS 속성 타입
type CSSUnit = 'px' | 'em' | 'rem' | '%';
type CSSValue = `${number}${CSSUnit}`;

const width: CSSValue = '100px';  // OK
const height: CSSValue = '50%';   // OK
// const invalid: CSSValue = '100';  // Error

// API 경로 타입
type ApiVersion = 'v1' | 'v2';
type Resource = 'users' | 'posts' | 'comments';
type ApiPath = `/api/${ApiVersion}/${Resource}`;
// '/api/v1/users' | '/api/v1/posts' | '/api/v1/comments' | ...

// 경로 매개변수 추출
type ExtractParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<Rest>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractParams<'/users/:userId/posts/:postId'>;
// 'userId' | 'postId'
```

### Strict Null Checks 패턴

```typescript
// 옵셔널 체이닝과 널 병합
interface UserProfile {
  name: string;
  address?: {
    city?: string;
    country?: string;
  };
  contacts?: {
    email?: string;
    phone?: string;
  }[];
}

function getCity(user: UserProfile): string {
  // 옵셔널 체이닝 + 널 병합
  return user.address?.city ?? 'Unknown';
}

function getFirstEmail(user: UserProfile): string | undefined {
  // 배열의 첫 요소에 안전하게 접근
  return user.contacts?.[0]?.email;
}

// Non-null assertion (!)은 가능한 피하기
function processUser(user: UserProfile | null) {
  // ❌ 위험: user가 null이면 런타임 오류
  // console.log(user!.name);

  // ✅ 안전: 먼저 검사
  if (user) {
    console.log(user.name);
  }
}

// 널 체크 유틸리티 함수
function isNotNullish<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}

const values = [1, null, 2, undefined, 3];
const numbers = values.filter(isNotNullish);  // number[]
```

### 타입 단언 최소화

```typescript
// ❌ 나쁜 예: 과도한 타입 단언
function processData(data: unknown) {
  const user = data as User;
  console.log(user.name);
}

// ✅ 좋은 예: 타입 가드 사용
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'name' in data &&
    'email' in data &&
    typeof (data as User).name === 'string' &&
    typeof (data as User).email === 'string'
  );
}

function processDataSafe(data: unknown) {
  if (isUser(data)) {
    console.log(data.name);  // 안전!
  } else {
    console.log('Invalid user data');
  }
}

// ✅ Zod 같은 런타임 검증 라이브러리 활용
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().optional()
});

type UserFromSchema = z.infer<typeof UserSchema>;

function processWithZod(data: unknown) {
  const result = UserSchema.safeParse(data);
  if (result.success) {
    const user = result.data;  // UserFromSchema 타입
    console.log(user.name);
  } else {
    console.log('Validation errors:', result.error);
  }
}
```

---

## 실전 프로젝트 패턴

### API 타입 안전하게 관리하기 (fetch wrapper)

```typescript
// API 응답 타입 정의
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface ApiError {
  code: string;
  message: string;
  details?: Record<string, string[]>;
}

// API 엔드포인트 정의
interface ApiEndpoints {
  'GET /users': {
    response: User[];
    query?: { page?: number; limit?: number };
  };
  'GET /users/:id': {
    response: User;
    params: { id: string };
  };
  'POST /users': {
    response: User;
    body: CreateUserDTO;
  };
  'PUT /users/:id': {
    response: User;
    params: { id: string };
    body: UpdateUserDTO;
  };
  'DELETE /users/:id': {
    response: void;
    params: { id: string };
  };
}

// 엔드포인트에서 HTTP 메서드와 경로 추출
type ExtractMethod<T extends string> = T extends `${infer M} ${string}` ? M : never;
type ExtractPath<T extends string> = T extends `${string} ${infer P}` ? P : never;

// 타입 안전한 fetch 클라이언트
class ApiClient {
  constructor(private baseUrl: string) {}

  async request<K extends keyof ApiEndpoints>(
    endpoint: K,
    options?: {
      params?: ApiEndpoints[K] extends { params: infer P } ? P : never;
      query?: ApiEndpoints[K] extends { query: infer Q } ? Q : never;
      body?: ApiEndpoints[K] extends { body: infer B } ? B : never;
    }
  ): Promise<ApiResponse<ApiEndpoints[K]['response']>> {
    const [method, pathTemplate] = (endpoint as string).split(' ');

    // 경로 파라미터 치환
    let path = pathTemplate;
    if (options?.params) {
      for (const [key, value] of Object.entries(options.params)) {
        path = path.replace(`:${key}`, String(value));
      }
    }

    // 쿼리 스트링 생성
    const queryString = options?.query
      ? '?' + new URLSearchParams(options.query as Record<string, string>).toString()
      : '';

    const response = await fetch(`${this.baseUrl}${path}${queryString}`, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: options?.body ? JSON.stringify(options.body) : undefined
    });

    return response.json();
  }
}

// 사용 예시
const api = new ApiClient('https://api.example.com');

// 타입 안전한 API 호출
async function example() {
  // GET /users
  const users = await api.request('GET /users', {
    query: { page: 1, limit: 10 }
  });
  // users.data는 User[]

  // GET /users/:id
  const user = await api.request('GET /users/:id', {
    params: { id: '123' }
  });
  // user.data는 User

  // POST /users
  const newUser = await api.request('POST /users', {
    body: { name: 'John', email: 'john@test.com' }
  });
  // newUser.data는 User
}
```

### React Props 타입 패턴

```typescript
import { ComponentProps, PropsWithChildren, ReactNode, HTMLAttributes } from 'react';

// 기본 HTML 요소 Props 확장
interface ButtonProps extends ComponentProps<'button'> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}

function Button({ variant = 'primary', size = 'md', loading, children, ...props }: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
}

// PropsWithChildren 활용
interface CardProps {
  title: string;
  footer?: ReactNode;
}

function Card({ title, footer, children }: PropsWithChildren<CardProps>) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// 다형성 컴포넌트 (as prop 패턴)
type PolymorphicProps<E extends React.ElementType, P = {}> = P &
  Omit<ComponentProps<E>, keyof P | 'as'> & {
    as?: E;
  };

interface TextOwnProps {
  color?: 'primary' | 'secondary' | 'muted';
  size?: 'sm' | 'md' | 'lg';
}

type TextProps<E extends React.ElementType = 'span'> = PolymorphicProps<E, TextOwnProps>;

function Text<E extends React.ElementType = 'span'>({
  as,
  color = 'primary',
  size = 'md',
  ...props
}: TextProps<E>) {
  const Component = as || 'span';
  return <Component className={`text-${color} text-${size}`} {...props} />;
}

// 사용 예시
function Example() {
  return (
    <>
      <Text>Default span</Text>
      <Text as="p" color="muted">Paragraph</Text>
      <Text as="h1" size="lg">Heading</Text>
      <Text as="a" href="/about">Link</Text>
    </>
  );
}
```

### 상태 관리 타입 (Redux, Zustand)

#### Redux Toolkit 타입

```typescript
import { createSlice, PayloadAction, configureStore } from '@reduxjs/toolkit';

// 상태 타입
interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null
};

// 슬라이스 생성
const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
      state.error = null;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.user = null;
    },
    clearUser: (state) => {
      state.user = null;
      state.error = null;
    }
  }
});

// 스토어 생성
const store = configureStore({
  reducer: {
    user: userSlice.reducer
  }
});

// 타입 추출
type RootState = ReturnType<typeof store.getState>;
type AppDispatch = typeof store.dispatch;

// 타입이 적용된 훅
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';

const useAppDispatch = () => useDispatch<AppDispatch>();
const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// 사용 예시
function UserComponent() {
  const dispatch = useAppDispatch();
  const user = useAppSelector((state) => state.user.user);
  const loading = useAppSelector((state) => state.user.loading);

  return (
    <div>
      {loading && <div>Loading...</div>}
      {user && <div>{user.name}</div>}
    </div>
  );
}
```

#### Zustand 타입

```typescript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// 상태 타입
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

// 스토어 생성
const useCartStore = create<CartState>()(
  devtools(
    persist(
      (set, get) => ({
        items: [],
        total: 0,

        addItem: (item) =>
          set((state) => {
            const existingItem = state.items.find((i) => i.id === item.id);

            if (existingItem) {
              return {
                items: state.items.map((i) =>
                  i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
                ),
                total: state.total + item.price
              };
            }

            return {
              items: [...state.items, { ...item, quantity: 1 }],
              total: state.total + item.price
            };
          }),

        removeItem: (id) =>
          set((state) => {
            const item = state.items.find((i) => i.id === id);
            if (!item) return state;

            return {
              items: state.items.filter((i) => i.id !== id),
              total: state.total - item.price * item.quantity
            };
          }),

        updateQuantity: (id, quantity) =>
          set((state) => {
            const item = state.items.find((i) => i.id === id);
            if (!item) return state;

            const quantityDiff = quantity - item.quantity;
            return {
              items: state.items.map((i) =>
                i.id === id ? { ...i, quantity } : i
              ),
              total: state.total + item.price * quantityDiff
            };
          }),

        clearCart: () => set({ items: [], total: 0 })
      }),
      { name: 'cart-storage' }
    )
  )
);

// 선택자 (Selector) 타입
const selectItems = (state: CartState) => state.items;
const selectTotal = (state: CartState) => state.total;
const selectItemCount = (state: CartState) =>
  state.items.reduce((sum, item) => sum + item.quantity, 0);

// 사용 예시
function CartComponent() {
  const items = useCartStore(selectItems);
  const total = useCartStore(selectTotal);
  const addItem = useCartStore((state) => state.addItem);

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>
          {item.name} x {item.quantity}
        </div>
      ))}
      <div>Total: {total}</div>
    </div>
  );
}
```

### 폼 라이브러리 타입 (React Hook Form + Zod)

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Zod 스키마 정의
const signUpSchema = z.object({
  email: z.string().email('올바른 이메일 형식이 아닙니다.'),
  password: z
    .string()
    .min(8, '비밀번호는 8자 이상이어야 합니다.')
    .regex(/[A-Z]/, '대문자를 포함해야 합니다.')
    .regex(/[0-9]/, '숫자를 포함해야 합니다.'),
  confirmPassword: z.string(),
  name: z.string().min(2, '이름은 2자 이상이어야 합니다.'),
  age: z.number().min(18, '18세 이상이어야 합니다.').optional(),
  terms: z.literal(true, {
    errorMap: () => ({ message: '약관에 동의해야 합니다.' })
  })
}).refine((data) => data.password === data.confirmPassword, {
  message: '비밀번호가 일치하지 않습니다.',
  path: ['confirmPassword']
});

// Zod 스키마에서 타입 추론
type SignUpFormData = z.infer<typeof signUpSchema>;

// 폼 컴포넌트
function SignUpForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm<SignUpFormData>({
    resolver: zodResolver(signUpSchema),
    defaultValues: {
      email: '',
      password: '',
      confirmPassword: '',
      name: '',
      terms: false
    }
  });

  const onSubmit = async (data: SignUpFormData) => {
    console.log('Form data:', data);
    // API 호출
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="이메일" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>

      <div>
        <input {...register('password')} type="password" placeholder="비밀번호" />
        {errors.password && <span>{errors.password.message}</span>}
      </div>

      <div>
        <input
          {...register('confirmPassword')}
          type="password"
          placeholder="비밀번호 확인"
        />
        {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}
      </div>

      <div>
        <input {...register('name')} placeholder="이름" />
        {errors.name && <span>{errors.name.message}</span>}
      </div>

      <div>
        <input
          {...register('age', { valueAsNumber: true })}
          type="number"
          placeholder="나이 (선택)"
        />
        {errors.age && <span>{errors.age.message}</span>}
      </div>

      <div>
        <label>
          <input {...register('terms')} type="checkbox" />
          약관에 동의합니다.
        </label>
        {errors.terms && <span>{errors.terms.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '처리 중...' : '회원가입'}
      </button>
    </form>
  );
}
```

### 환경 변수 타입 안전하게 사용하기

```typescript
// env.ts
import { z } from 'zod';

// 환경 변수 스키마 정의
const envSchema = z.object({
  // 필수 환경 변수
  NODE_ENV: z.enum(['development', 'production', 'test']),
  API_URL: z.string().url(),
  DATABASE_URL: z.string(),

  // 선택적 환경 변수
  PORT: z.string().transform(Number).default('3000'),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),

  // API 키
  API_KEY: z.string().min(1),
  SECRET_KEY: z.string().min(32)
});

// 환경 변수 타입
type Env = z.infer<typeof envSchema>;

// 환경 변수 검증 및 파싱
function validateEnv(): Env {
  const result = envSchema.safeParse(process.env);

  if (!result.success) {
    console.error('Invalid environment variables:');
    console.error(result.error.format());
    process.exit(1);
  }

  return result.data;
}

// 환경 변수 내보내기
export const env = validateEnv();

// 사용 예시
// import { env } from './env';
// console.log(env.API_URL);  // 타입 안전!
// console.log(env.PORT);     // number 타입

// Vite 환경에서의 타입 정의
// vite-env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
  readonly VITE_ANALYTICS_ID?: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

// Next.js 환경에서의 타입 정의 (next.config.js와 함께 사용)
declare namespace NodeJS {
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
    NEXT_PUBLIC_API_URL: string;
    DATABASE_URL: string;
    SECRET_KEY: string;
  }
}
```

---

## 성능과 컴파일 최적화

### satisfies 연산자 활용

`satisfies` 연산자는 TypeScript 4.9에서 도입되었으며, 타입 체크와 타입 추론의 균형을 맞춥니다.

```typescript
// as const와 satisfies 비교
type Colors = Record<string, string>;

// as만 사용: 타입이 Colors로 변환됨
const colors1 = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
} as Colors;

colors1.red;  // 타입: string (구체적인 값을 잃음)

// satisfies 사용: 타입 검사 + 원래 타입 유지
const colors2 = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
} satisfies Colors;

colors2.red;  // 타입: '#ff0000' (리터럴 타입 유지)
// colors2.purple;  // Error: 'purple' 속성이 없음

// as const + satisfies 조합
const colors3 = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
} as const satisfies Colors;

// 실전 예제: 라우트 설정
type Route = {
  path: string;
  component: React.ComponentType;
  exact?: boolean;
};

const routes = [
  { path: '/', component: HomePage, exact: true },
  { path: '/about', component: AboutPage },
  { path: '/contact', component: ContactPage }
] satisfies Route[];

// routes는 여전히 구체적인 타입을 유지하면서 Route[] 제약을 만족
```

### const type parameters (5.0+)

```typescript
// const 타입 파라미터 없이
function getValues<T extends readonly string[]>(values: T) {
  return values;
}

const values1 = getValues(['a', 'b', 'c']);
// 타입: string[]

// const 타입 파라미터 사용
function getConstValues<const T extends readonly string[]>(values: T) {
  return values;
}

const values2 = getConstValues(['a', 'b', 'c']);
// 타입: readonly ['a', 'b', 'c']

// 실전 예제: 상태 머신
function createMachine<const T extends readonly string[]>(states: T) {
  type State = T[number];

  return {
    states,
    transition(from: State, to: State) {
      console.log(`${from} -> ${to}`);
    }
  };
}

const machine = createMachine(['idle', 'loading', 'success', 'error']);

machine.transition('idle', 'loading');  // OK
// machine.transition('idle', 'unknown');  // Error: 'unknown'이 상태에 없음
```

### 타입 복잡도 줄이기

```typescript
// ❌ 복잡한 조건부 타입 체이닝
type ComplexType<T> = T extends string
  ? T extends `${infer A}.${infer B}`
    ? A extends keyof SomeMap
      ? SomeMap[A] extends { [key: string]: infer V }
        ? V extends string
          ? V
          : never
        : never
      : never
    : T
  : never;

// ✅ 단계별로 분리
type ExtractDomain<T extends string> = T extends `${infer A}.${string}` ? A : T;
type GetMapValue<K extends string> = K extends keyof SomeMap ? SomeMap[K] : never;
type ExtractStringValue<T> = T extends { [key: string]: infer V }
  ? V extends string
    ? V
    : never
  : never;

type SimplifiedType<T extends string> = ExtractStringValue<
  GetMapValue<ExtractDomain<T>>
>;

// 재귀 깊이 제한
type RecursiveType<T, Depth extends number[] = []> =
  Depth['length'] extends 10  // 최대 10단계
    ? T
    : T extends object
    ? { [K in keyof T]: RecursiveType<T[K], [...Depth, 0]> }
    : T;
```

### tsconfig 최적화 옵션

```json
{
  "compilerOptions": {
    // 타입 안전성 최대화
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noPropertyAccessFromIndexSignature": true,

    // 성능 최적화
    "skipLibCheck": true,
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",

    // 모듈 해석 최적화
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,

    // 출력 최적화
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,

    // 최신 기능 활성화
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],

    // 경로 별칭
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

#### 주요 옵션 설명

| 옵션 | 설명 |
|------|------|
| `noUncheckedIndexedAccess` | 인덱스 접근 시 undefined 가능성 체크 |
| `exactOptionalPropertyTypes` | 선택적 속성에 undefined 명시적 할당 필요 |
| `skipLibCheck` | node_modules 타입 체크 스킵 (빌드 속도 향상) |
| `incremental` | 증분 컴파일 활성화 |
| `isolatedModules` | 각 파일을 독립적으로 트랜스파일 가능하게 함 |

---

## FAQ

### Q1: `any`와 `unknown`의 차이점은 무엇인가요?

**A:** `any`는 타입 검사를 완전히 비활성화하고, `unknown`은 타입 안전한 any입니다.

```typescript
// any: 모든 연산 허용 (위험)
function processAny(value: any) {
  value.foo();        // OK (런타임 오류 가능)
  value.bar.baz();    // OK (런타임 오류 가능)
  value + 1;          // OK
}

// unknown: 타입 체크 필요
function processUnknown(value: unknown) {
  // value.foo();     // Error: Object is of type 'unknown'

  // 타입 검사 후 안전하게 사용
  if (typeof value === 'string') {
    value.toUpperCase();  // OK
  }

  if (typeof value === 'object' && value !== null && 'foo' in value) {
    (value as { foo: () => void }).foo();  // OK
  }
}
```

**권장사항:** `any` 대신 `unknown`을 사용하고, 타입 가드로 좁히세요.

### Q2: 제네릭 타입 파라미터의 네이밍 컨벤션은?

**A:** 일반적인 컨벤션:

```typescript
// 단일 문자 (간단한 경우)
function identity<T>(value: T): T { return value; }
function map<K, V>(key: K, value: V) {}

// 의미있는 이름 (복잡한 경우)
interface Repository<Entity, Id = string> {
  findById(id: Id): Promise<Entity | null>;
  save(entity: Entity): Promise<Entity>;
}

// 일반적인 규칙
// T - Type (일반적인 타입)
// K - Key
// V - Value
// E - Element (배열 요소)
// R - Return (반환 타입)
// P - Props (React)
// S - State
```

### Q3: `type`과 `interface`는 언제 사용해야 하나요?

**A:** 일반적인 가이드라인:

```typescript
// interface: 객체 타입, 확장 가능성이 필요한 경우
interface User {
  id: number;
  name: string;
}

// 선언 병합 가능 (라이브러리 타입 확장에 유용)
interface User {
  email: string;  // 자동으로 병합됨
}

// type: 유니온, 튜플, 복잡한 타입 조합
type ID = string | number;
type Point = [number, number];
type EventHandler = (event: Event) => void;

// 조건부 타입, 매핑된 타입
type Readonly<T> = { readonly [K in keyof T]: T[K] };
```

**실용적 조언:** 팀 내 일관성이 가장 중요합니다. 하나를 선택하고 일관되게 사용하세요.

### Q4: 타입 단언(as)은 언제 사용해야 하나요?

**A:** 가능한 피하되, 다음 상황에서는 사용할 수 있습니다:

```typescript
// 1. DOM API 사용 시
const input = document.getElementById('input') as HTMLInputElement;

// 2. 외부 라이브러리 타입이 부정확할 때
const data = externalLib.getData() as ExpectedType;

// 3. 테스트에서 부분 모킹
const mockUser = { name: 'Test' } as User;

// ❌ 피해야 할 사용
const data = JSON.parse(json) as User;  // 런타임 검증 없이 위험

// ✅ 대안: 런타임 검증
function isUser(data: unknown): data is User {
  return typeof data === 'object' && data !== null && 'name' in data;
}

const parsed = JSON.parse(json);
if (isUser(parsed)) {
  // parsed는 User 타입
}
```

### Q5: Discriminated Union과 일반 Union의 차이점은?

**A:** Discriminated Union은 공통 리터럴 속성(판별자)을 가집니다.

```typescript
// 일반 Union: 타입 좁히기 어려움
type Shape = Circle | Square;

function getArea1(shape: Shape) {
  // 어떤 타입인지 판별하기 어려움
  if ('radius' in shape) {
    return Math.PI * shape.radius ** 2;
  }
  return shape.sideLength ** 2;
}

// Discriminated Union: 판별자로 명확한 분기
interface Circle {
  kind: 'circle';  // 판별자
  radius: number;
}

interface Square {
  kind: 'square';  // 판별자
  sideLength: number;
}

type ShapeDiscriminated = Circle | Square;

function getArea2(shape: ShapeDiscriminated) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;  // shape는 Circle
    case 'square':
      return shape.sideLength ** 2;  // shape는 Square
  }
}
```

### Q6: infer 키워드는 정확히 어떻게 동작하나요?

**A:** `infer`는 조건부 타입에서 타입을 추출하고 이름을 붙입니다.

```typescript
// 기본 패턴: T extends Pattern<infer U> ? U : Default
type ElementType<T> = T extends (infer U)[] ? U : never;

type A = ElementType<string[]>;  // string
type B = ElementType<number[]>;  // number
type C = ElementType<string>;    // never

// 함수 타입에서 추출
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type ParamType<T> = T extends (arg: infer P) => any ? P : never;

// 여러 위치에서 추론
type FirstAndLast<T> = T extends [infer First, ...any[], infer Last]
  ? [First, Last]
  : never;

type FL = FirstAndLast<[1, 2, 3, 4, 5]>;  // [1, 5]

// 동일 타입 변수의 여러 추론 (Union으로 합쳐짐)
type Flatten<T> = T extends { a: infer U; b: infer U } ? U : never;

type F = Flatten<{ a: string; b: number }>;  // string | number
```

### Q7: TypeScript 5.x로 업그레이드할 때 주의할 점은?

**A:** 주요 주의사항:

1. **Breaking Changes 확인**
```typescript
// 5.0: lib.d.ts 변경으로 일부 타입이 변경됨
// 예: PromiseConstructor.resolve의 반환 타입

// 5.4: NoInfer 동작 이해
// 기존에 작동하던 타입 추론이 달라질 수 있음

// 5.6: Nullish/Truthy 체크 강화
// 항상 true/false인 조건문이 오류로 처리됨
```

2. **tsconfig 업데이트**
```json
{
  "compilerOptions": {
    // 새로운 모듈 해석 모드
    "moduleResolution": "bundler",

    // 새로운 타겟
    "target": "ES2022",

    // 새로운 lib
    "lib": ["ES2023"]
  }
}
```

3. **의존성 호환성 확인**
```bash
# TypeScript 버전 확인
npx tsc --version

# 의존성 타입 호환성 체크
npm ls @types/react
```

---

## 마치며

TypeScript 5.x는 강력한 타입 시스템과 개발자 경험 개선을 통해 더욱 안전하고 생산적인 개발 환경을 제공합니다. 이 가이드에서 다룬 패턴들을 실무에 적용하면:

1. **컴파일 타임 오류 감지**: 런타임 오류를 크게 줄일 수 있습니다
2. **코드 자동 완성**: IDE의 강력한 지원을 받을 수 있습니다
3. **리팩토링 안전성**: 대규모 코드 변경도 자신있게 수행할 수 있습니다
4. **문서화**: 타입 자체가 코드의 의도를 설명합니다

### 다음 단계 추천

1. 프로젝트에 `strict` 모드 활성화
2. 커스텀 유틸리티 타입 라이브러리 구축
3. Zod 같은 런타임 검증 라이브러리와 TypeScript 연동
4. 타입 테스트 도구(tsd, expect-type) 도입

---

## 참고 자료

### 공식 문서
- [TypeScript 공식 문서](https://www.typescriptlang.org/docs/)
- [TypeScript 5.0 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html)
- [TypeScript 5.3 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-3.html)
- [TypeScript 5.4 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html)
- [TypeScript 5.5 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html)
- [TypeScript 5.6 릴리스 노트](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-6.html)

### 유용한 리소스
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Type Challenges](https://github.com/type-challenges/type-challenges)
- [Zod Documentation](https://zod.dev/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
