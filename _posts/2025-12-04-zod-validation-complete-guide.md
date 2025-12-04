---
title: "Zod 완벽 가이드: TypeScript 스키마 검증의 모든 것"
description: "TypeScript 런타임 검증 라이브러리 Zod 완벽 가이드. 스키마 정의부터 타입 추론, 커스텀 에러 처리, React Hook Form 통합까지 실전 예제로 안전한 데이터 검증을 마스터하세요."
date: 2025-12-04 19:00:00 +0900
categories: [Frontend, TypeScript]
tags: [zod, typescript, validation, schema, react-hook-form, runtime-validation, type-safety, form-validation, data-validation, type-inference]
---

## 개요

TypeScript는 정적 타입 검사를 통해 개발 시점의 오류를 방지합니다. 하지만 **런타임**에서 외부 데이터(API 응답, 사용자 입력, 환경변수 등)의 타입은 보장되지 않습니다. **Zod**는 이 문제를 해결하는 TypeScript 우선 스키마 검증 라이브러리입니다.

이 글에서는 Zod의 기초부터 고급 패턴까지, 실전에서 바로 활용할 수 있는 예제와 함께 상세히 알아봅니다.

---

## TypeScript만으로는 부족한 이유

### 컴파일 타임 vs 런타임

TypeScript의 타입 시스템은 **컴파일 타임**에만 존재합니다. JavaScript로 변환되면 모든 타입 정보가 사라집니다.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// API에서 받아온 데이터 - 실제로는 타입이 보장되지 않음
const response = await fetch('/api/user');
const user: User = await response.json();

// TypeScript는 이 코드에 문제가 없다고 판단하지만
// 실제 API 응답이 { id: "123", name: null } 이라면?
console.log(user.email.toLowerCase()); // 런타임 에러!
```

### 런타임 검증이 필요한 상황

| 상황 | 설명 | 위험도 |
|------|------|--------|
| API 응답 | 서버가 예상과 다른 형식 반환 | 높음 |
| 사용자 입력 | 폼 데이터, URL 파라미터 | 높음 |
| 환경 변수 | 필수 설정값 누락 | 높음 |
| 외부 라이브러리 | 타입 정의와 실제 동작 불일치 | 중간 |
| localStorage | 저장된 데이터 형식 변경 | 중간 |
| WebSocket | 실시간 메시지 형식 | 높음 |

### Zod가 해결하는 문제

Zod는 **런타임 타입 검증**과 **TypeScript 타입 추론**을 동시에 제공합니다.

```typescript
import { z } from 'zod';

// 스키마 정의 = 런타임 검증 + 타입 정의
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
});

// 스키마에서 TypeScript 타입 추론
type User = z.infer<typeof userSchema>;
// type User = { id: number; name: string; email: string; }

// 런타임에서 안전하게 검증
const result = userSchema.safeParse(apiResponse);
if (result.success) {
  // result.data는 User 타입으로 보장됨
  console.log(result.data.email.toLowerCase());
} else {
  // 검증 실패 시 상세한 에러 정보 제공
  console.error(result.error.issues);
}
```

---

## 설치 및 기본 사용법

### 설치

```bash
# npm
npm install zod

# yarn
yarn add zod

# pnpm
pnpm add zod
```

Zod는 **제로 의존성** 라이브러리로, 번들 크기가 약 **12KB** (minified + gzipped)입니다.

### tsconfig.json 설정

Zod를 최적으로 사용하려면 다음 설정을 권장합니다.

```jsonc
{
  "compilerOptions": {
    "strict": true,           // 필수
    "strictNullChecks": true  // 필수 (strict에 포함)
  }
}
```

### 기본 스키마 정의

```typescript
import { z } from 'zod';

// 기본 스키마 생성
const stringSchema = z.string();
const numberSchema = z.number();
const booleanSchema = z.boolean();

// parse: 검증 성공 시 값 반환, 실패 시 에러 throw
const name = stringSchema.parse('John'); // 'John'
const invalid = stringSchema.parse(123); // ZodError 발생!

// safeParse: 에러를 throw하지 않고 결과 객체 반환
const result = stringSchema.safeParse('John');
if (result.success) {
  console.log(result.data); // 'John'
} else {
  console.log(result.error); // ZodError
}
```

---

## 기본 타입 스키마

### 문자열 (z.string)

```typescript
import { z } from 'zod';

// 기본 문자열
const basicString = z.string();

// 문자열 검증 메서드 체이닝
const emailSchema = z.string()
  .min(1, '이메일은 필수입니다')
  .email('올바른 이메일 형식이 아닙니다');

const usernameSchema = z.string()
  .min(3, '3자 이상 입력하세요')
  .max(20, '20자 이하로 입력하세요')
  .regex(/^[a-zA-Z0-9_]+$/, '영문, 숫자, 밑줄만 사용 가능합니다');

const urlSchema = z.string().url('올바른 URL을 입력하세요');
const uuidSchema = z.string().uuid('올바른 UUID 형식이 아닙니다');

// 문자열 변환
const trimmedSchema = z.string().trim();
const lowercaseSchema = z.string().toLowerCase();
const uppercaseSchema = z.string().toUpperCase();
```

### 문자열 검증 메서드 정리

| 메서드 | 설명 | 예시 |
|--------|------|------|
| `.min(n)` | 최소 길이 | `.min(1, '필수')` |
| `.max(n)` | 최대 길이 | `.max(100)` |
| `.length(n)` | 정확한 길이 | `.length(6)` |
| `.email()` | 이메일 형식 | `.email()` |
| `.url()` | URL 형식 | `.url()` |
| `.uuid()` | UUID 형식 | `.uuid()` |
| `.cuid()` | CUID 형식 | `.cuid()` |
| `.regex(re)` | 정규식 매칭 | `.regex(/^[A-Z]/)` |
| `.includes(s)` | 문자열 포함 | `.includes('@')` |
| `.startsWith(s)` | 시작 문자열 | `.startsWith('http')` |
| `.endsWith(s)` | 끝 문자열 | `.endsWith('.com')` |
| `.datetime()` | ISO 8601 형식 | `.datetime()` |
| `.ip()` | IP 주소 형식 | `.ip()` |
| `.trim()` | 앞뒤 공백 제거 | `.trim()` |
| `.toLowerCase()` | 소문자 변환 | `.toLowerCase()` |
| `.toUpperCase()` | 대문자 변환 | `.toUpperCase()` |

### 숫자 (z.number)

```typescript
import { z } from 'zod';

// 기본 숫자
const numberSchema = z.number();

// 숫자 검증 메서드
const ageSchema = z.number()
  .int('정수만 입력 가능합니다')
  .min(0, '0 이상이어야 합니다')
  .max(120, '120 이하여야 합니다');

const priceSchema = z.number()
  .positive('양수만 가능합니다')
  .multipleOf(100, '100 단위로 입력하세요');

const percentSchema = z.number()
  .min(0)
  .max(100);

// NaN, Infinity 처리
const safeNumber = z.number().finite(); // Infinity 제외
```

### 숫자 검증 메서드 정리

| 메서드 | 설명 |
|--------|------|
| `.int()` | 정수만 |
| `.positive()` | 양수 (> 0) |
| `.nonnegative()` | 0 이상 (>= 0) |
| `.negative()` | 음수 (< 0) |
| `.nonpositive()` | 0 이하 (<= 0) |
| `.min(n)` | 최소값 |
| `.max(n)` | 최대값 |
| `.multipleOf(n)` | n의 배수 |
| `.finite()` | 유한한 숫자 |
| `.safe()` | 안전한 정수 범위 |

### 불리언 (z.boolean)

```typescript
import { z } from 'zod';

const boolSchema = z.boolean();

// 특정 값만 허용
const trueOnly = z.literal(true);

// 체크박스 검증 (약관 동의 등)
const termsSchema = z.literal(true, {
  errorMap: () => ({ message: '약관에 동의해야 합니다' }),
});
```

### 날짜 (z.date)

```typescript
import { z } from 'zod';

const dateSchema = z.date();

// 날짜 범위 검증
const birthDateSchema = z.date()
  .min(new Date('1900-01-01'), '1900년 이후여야 합니다')
  .max(new Date(), '미래 날짜는 선택할 수 없습니다');

// 문자열을 날짜로 변환 (coerce)
const dateFromString = z.coerce.date();
dateFromString.parse('2024-01-15'); // Date 객체로 변환
```

### 열거형 (z.enum)

```typescript
import { z } from 'zod';

// 문자열 열거형
const roleSchema = z.enum(['admin', 'user', 'guest']);
type Role = z.infer<typeof roleSchema>; // 'admin' | 'user' | 'guest'

// 열거형 값 접근
roleSchema.enum.admin; // 'admin'
roleSchema.options; // ['admin', 'user', 'guest']

// TypeScript enum 사용
enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
  Pending = 'PENDING',
}

const statusSchema = z.nativeEnum(Status);
type StatusType = z.infer<typeof statusSchema>; // Status
```

### 리터럴 (z.literal)

```typescript
import { z } from 'zod';

// 특정 값만 허용
const literalSchema = z.literal('hello');
const numberLiteral = z.literal(42);
const boolLiteral = z.literal(true);

// 상태값 정의에 유용
const successStatus = z.literal('success');
const errorStatus = z.literal('error');
```

---

## 객체와 배열 스키마

### 객체 (z.object)

```typescript
import { z } from 'zod';

// 기본 객체 스키마
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().optional(), // 선택적 필드
});

type User = z.infer<typeof userSchema>;
// type User = {
//   id: number;
//   name: string;
//   email: string;
//   age?: number | undefined;
// }

// 중첩 객체
const profileSchema = z.object({
  user: userSchema,
  settings: z.object({
    theme: z.enum(['light', 'dark']),
    notifications: z.boolean(),
  }),
  createdAt: z.date(),
});
```

### 객체 수정 메서드

```typescript
import { z } from 'zod';

const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  password: z.string(),
});

// pick: 특정 필드만 선택
const userPublicSchema = userSchema.pick({
  id: true,
  name: true,
  email: true,
});

// omit: 특정 필드 제외
const userWithoutPassword = userSchema.omit({
  password: true,
});

// partial: 모든 필드를 선택적으로
const updateUserSchema = userSchema.partial();
// 모든 필드가 optional이 됨

// required: 모든 필드를 필수로
const requiredUserSchema = userSchema.partial().required();

// extend: 필드 추가
const adminSchema = userSchema.extend({
  role: z.literal('admin'),
  permissions: z.array(z.string()),
});

// merge: 두 스키마 병합
const timestampSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

const fullUserSchema = userSchema.merge(timestampSchema);
```

### 객체 동작 설정

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string(),
});

// strict: 정의되지 않은 키가 있으면 에러
const strictSchema = schema.strict();
strictSchema.parse({ name: 'John', extra: 'data' }); // ZodError!

// passthrough: 정의되지 않은 키도 유지
const passthroughSchema = schema.passthrough();
passthroughSchema.parse({ name: 'John', extra: 'data' });
// { name: 'John', extra: 'data' }

// strip: 정의되지 않은 키 제거 (기본값)
const stripSchema = schema.strip();
stripSchema.parse({ name: 'John', extra: 'data' });
// { name: 'John' }
```

### 배열 (z.array)

```typescript
import { z } from 'zod';

// 기본 배열
const stringArraySchema = z.array(z.string());

// 배열 검증
const tagsSchema = z.array(z.string())
  .min(1, '최소 1개 태그가 필요합니다')
  .max(10, '최대 10개까지 가능합니다');

// 중첩 배열
const matrixSchema = z.array(z.array(z.number()));

// 객체 배열
const usersSchema = z.array(
  z.object({
    id: z.number(),
    name: z.string(),
  })
);

// 비어있지 않은 배열
const nonEmptyArray = z.array(z.string()).nonempty('배열이 비어있습니다');
type NonEmpty = z.infer<typeof nonEmptyArray>;
// [string, ...string[]]
```

### 튜플 (z.tuple)

```typescript
import { z } from 'zod';

// 고정 길이 배열
const coordinateSchema = z.tuple([z.number(), z.number()]);
type Coordinate = z.infer<typeof coordinateSchema>; // [number, number]

// 다양한 타입의 튜플
const mixedTupleSchema = z.tuple([
  z.string(),  // 이름
  z.number(),  // 나이
  z.boolean(), // 활성 상태
]);

// rest 요소
const variadicSchema = z.tuple([z.string(), z.number()]).rest(z.boolean());
// [string, number, ...boolean[]]
```

### 레코드 (z.record)

```typescript
import { z } from 'zod';

// 동적 키를 가진 객체
const stringRecordSchema = z.record(z.string());
type StringRecord = z.infer<typeof stringRecordSchema>;
// { [k: string]: string }

// 특정 키 타입으로 제한
const userRolesSchema = z.record(
  z.enum(['admin', 'user', 'guest']),
  z.array(z.string())
);
// Record<'admin' | 'user' | 'guest', string[]>

// 실제 사용 예시: 국제화
const translationsSchema = z.record(
  z.enum(['ko', 'en', 'ja']),
  z.string()
);
```

---

## 스키마 조합

### Union (z.union)

여러 스키마 중 하나에 매칭되면 유효합니다.

```typescript
import { z } from 'zod';

// 기본 union
const stringOrNumber = z.union([z.string(), z.number()]);
type StringOrNumber = z.infer<typeof stringOrNumber>; // string | number

stringOrNumber.parse('hello'); // 성공
stringOrNumber.parse(42);      // 성공
stringOrNumber.parse(true);    // 실패

// 복잡한 union
const responseSchema = z.union([
  z.object({
    success: z.literal(true),
    data: z.object({ id: z.number(), name: z.string() }),
  }),
  z.object({
    success: z.literal(false),
    error: z.object({ code: z.string(), message: z.string() }),
  }),
]);
```

### Discriminated Union (z.discriminatedUnion)

공통 판별자(discriminator)로 타입을 구분하는 union입니다. 일반 union보다 **성능이 좋고** 에러 메시지가 명확합니다.

```typescript
import { z } from 'zod';

// 이벤트 타입별 스키마
const eventSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('click'),
    x: z.number(),
    y: z.number(),
  }),
  z.object({
    type: z.literal('keypress'),
    key: z.string(),
    modifiers: z.array(z.string()),
  }),
  z.object({
    type: z.literal('scroll'),
    direction: z.enum(['up', 'down']),
    amount: z.number(),
  }),
]);

type Event = z.infer<typeof eventSchema>;

// 타입 좁히기가 자동으로 동작
function handleEvent(event: Event) {
  switch (event.type) {
    case 'click':
      console.log(`Clicked at (${event.x}, ${event.y})`);
      break;
    case 'keypress':
      console.log(`Key pressed: ${event.key}`);
      break;
    case 'scroll':
      console.log(`Scrolled ${event.direction} by ${event.amount}`);
      break;
  }
}
```

### Intersection (z.intersection)

두 스키마의 교집합입니다. 두 스키마 모두 만족해야 유효합니다.

```typescript
import { z } from 'zod';

const personSchema = z.object({
  name: z.string(),
});

const employeeSchema = z.object({
  employeeId: z.string(),
  department: z.string(),
});

// 두 스키마 결합
const employeePersonSchema = z.intersection(personSchema, employeeSchema);
// 또는
const employeePersonSchema2 = personSchema.and(employeeSchema);

type EmployeePerson = z.infer<typeof employeePersonSchema>;
// { name: string; employeeId: string; department: string; }
```

---

## 변환과 정제

### transform - 값 변환

`transform`은 검증 후 값을 변환합니다.

```typescript
import { z } from 'zod';

// 문자열을 숫자로 변환
const numberFromString = z.string()
  .transform((val) => parseInt(val, 10));

numberFromString.parse('42'); // 42 (number)

// 문자열을 Date로 변환
const dateFromString = z.string()
  .transform((val) => new Date(val));

// 복잡한 변환
const userInputSchema = z.object({
  name: z.string().transform((val) => val.trim()),
  email: z.string().email().transform((val) => val.toLowerCase()),
  tags: z.string().transform((val) => val.split(',').map((t) => t.trim())),
});

userInputSchema.parse({
  name: '  John Doe  ',
  email: 'JOHN@EXAMPLE.COM',
  tags: 'react, typescript, zod',
});
// { name: 'John Doe', email: 'john@example.com', tags: ['react', 'typescript', 'zod'] }
```

### coerce - 타입 강제 변환

`coerce`는 입력값을 자동으로 해당 타입으로 변환합니다.

```typescript
import { z } from 'zod';

// 문자열 "42"를 숫자 42로 변환
const coercedNumber = z.coerce.number();
coercedNumber.parse('42');   // 42
coercedNumber.parse(42);     // 42

// boolean 강제 변환
const coercedBoolean = z.coerce.boolean();
coercedBoolean.parse('true');  // true
coercedBoolean.parse('');      // false
coercedBoolean.parse(1);       // true
coercedBoolean.parse(0);       // false

// Date 강제 변환
const coercedDate = z.coerce.date();
coercedDate.parse('2024-01-15');         // Date 객체
coercedDate.parse(1705276800000);        // Date 객체 (timestamp)

// 폼 입력 처리에 유용
const formSchema = z.object({
  age: z.coerce.number().min(0).max(120),
  isActive: z.coerce.boolean(),
  birthDate: z.coerce.date(),
});
```

### refine - 커스텀 검증

`refine`은 커스텀 검증 로직을 추가합니다.

```typescript
import { z } from 'zod';

// 단순 검증
const evenNumber = z.number().refine(
  (val) => val % 2 === 0,
  { message: '짝수만 입력 가능합니다' }
);

// 비밀번호 강도 검증
const passwordSchema = z.string()
  .min(8, '8자 이상 입력하세요')
  .refine(
    (val) => /[A-Z]/.test(val),
    { message: '대문자를 포함해야 합니다' }
  )
  .refine(
    (val) => /[a-z]/.test(val),
    { message: '소문자를 포함해야 합니다' }
  )
  .refine(
    (val) => /[0-9]/.test(val),
    { message: '숫자를 포함해야 합니다' }
  )
  .refine(
    (val) => /[!@#$%^&*]/.test(val),
    { message: '특수문자를 포함해야 합니다' }
  );

// 객체 레벨 검증 (비밀번호 확인)
const signupSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine(
  (data) => data.password === data.confirmPassword,
  {
    message: '비밀번호가 일치하지 않습니다',
    path: ['confirmPassword'], // 에러가 표시될 필드
  }
);
```

### superRefine - 고급 커스텀 검증

`superRefine`은 여러 에러를 동시에 추가할 수 있습니다.

```typescript
import { z } from 'zod';

const passwordSchema = z.string().superRefine((val, ctx) => {
  if (val.length < 8) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: '8자 이상 입력하세요',
    });
  }

  if (!/[A-Z]/.test(val)) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: '대문자를 포함해야 합니다',
    });
  }

  if (!/[0-9]/.test(val)) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: '숫자를 포함해야 합니다',
    });
  }
});

// 모든 검증 실패 메시지를 한번에 수집
const result = passwordSchema.safeParse('abc');
if (!result.success) {
  console.log(result.error.issues);
  // [
  //   { message: '8자 이상 입력하세요', ... },
  //   { message: '대문자를 포함해야 합니다', ... },
  //   { message: '숫자를 포함해야 합니다', ... },
  // ]
}
```

### 비동기 검증

```typescript
import { z } from 'zod';

// 비동기 refine
const usernameSchema = z.string()
  .min(3)
  .refine(
    async (username) => {
      const response = await fetch(`/api/check-username?username=${username}`);
      const { available } = await response.json();
      return available;
    },
    { message: '이미 사용 중인 사용자명입니다' }
  );

// 비동기 검증 실행 (parseAsync 또는 safeParseAsync 사용)
const result = await usernameSchema.safeParseAsync('newuser');
```

비동기 검증 로직을 테스트하는 방법은 [프론트엔드 테스트 완벽 가이드](/posts/frontend-testing-guide/)에서 확인하세요.

---

## 타입 추론

Zod의 가장 강력한 기능 중 하나는 스키마에서 TypeScript 타입을 자동으로 추론하는 것입니다. TypeScript의 고급 타입 패턴에 대해 더 알아보려면 [TypeScript 고급 타입 패턴 가이드](/posts/typescript-advanced-type-patterns/)를 참고하세요.

### z.infer - 스키마에서 타입 추론

```typescript
import { z } from 'zod';

const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  role: z.enum(['admin', 'user']),
  profile: z.object({
    bio: z.string().optional(),
    website: z.string().url().optional(),
  }).optional(),
  tags: z.array(z.string()),
  createdAt: z.date(),
});

// 스키마에서 타입 추론
type User = z.infer<typeof userSchema>;

// 추론된 타입:
// type User = {
//   id: number;
//   name: string;
//   email: string;
//   role: 'admin' | 'user';
//   profile?: {
//     bio?: string | undefined;
//     website?: string | undefined;
//   } | undefined;
//   tags: string[];
//   createdAt: Date;
// }
```

### z.input vs z.output

`transform`을 사용하면 입력과 출력 타입이 달라질 수 있습니다.

```typescript
import { z } from 'zod';

const schema = z.object({
  age: z.string().transform((val) => parseInt(val, 10)),
  date: z.string().transform((val) => new Date(val)),
});

// 입력 타입 (transform 전)
type Input = z.input<typeof schema>;
// { age: string; date: string; }

// 출력 타입 (transform 후)
type Output = z.output<typeof schema>;
// { age: number; date: Date; }

// z.infer는 z.output과 동일
type Inferred = z.infer<typeof schema>;
// { age: number; date: Date; }
```

### 타입과 스키마 동기화 패턴

```typescript
import { z } from 'zod';

// 1. 스키마 우선 (Schema First) - 권장
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
});
type User = z.infer<typeof userSchema>;

// 2. 타입 우선 (Type First) - 필요한 경우
interface User {
  id: number;
  name: string;
}
const userSchema: z.ZodType<User> = z.object({
  id: z.number(),
  name: z.string(),
});

// 3. 재사용 가능한 스키마 패턴
const idSchema = z.object({ id: z.number() });
const timestampsSchema = z.object({
  createdAt: z.date(),
  updatedAt: z.date(),
});

const userSchema = idSchema.merge(z.object({
  name: z.string(),
  email: z.string().email(),
})).merge(timestampsSchema);
```

---

## 에러 처리

### parse vs safeParse

```typescript
import { z } from 'zod';

const schema = z.string().email();

// parse: 실패 시 throw
try {
  const email = schema.parse('invalid');
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log(error.issues);
  }
}

// safeParse: 실패해도 throw 안함 (권장)
const result = schema.safeParse('invalid');
if (result.success) {
  console.log(result.data); // 검증된 값
} else {
  console.log(result.error.issues); // 에러 정보
}
```

### ZodError 구조

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().min(0),
});

const result = schema.safeParse({
  name: 'J',
  email: 'invalid',
  age: -5,
});

if (!result.success) {
  console.log(result.error.issues);
  // [
  //   {
  //     code: 'too_small',
  //     minimum: 2,
  //     type: 'string',
  //     inclusive: true,
  //     message: 'String must contain at least 2 character(s)',
  //     path: ['name']
  //   },
  //   {
  //     code: 'invalid_string',
  //     validation: 'email',
  //     message: 'Invalid email',
  //     path: ['email']
  //   },
  //   {
  //     code: 'too_small',
  //     minimum: 0,
  //     type: 'number',
  //     inclusive: true,
  //     message: 'Number must be greater than or equal to 0',
  //     path: ['age']
  //   }
  // ]

  // 포맷팅된 에러
  console.log(result.error.format());
  // {
  //   _errors: [],
  //   name: { _errors: ['String must contain at least 2 character(s)'] },
  //   email: { _errors: ['Invalid email'] },
  //   age: { _errors: ['Number must be greater than or equal to 0'] }
  // }

  // 평탄화된 에러
  console.log(result.error.flatten());
  // {
  //   formErrors: [],
  //   fieldErrors: {
  //     name: ['String must contain at least 2 character(s)'],
  //     email: ['Invalid email'],
  //     age: ['Number must be greater than or equal to 0']
  //   }
  // }
}
```

### 커스텀 에러 메시지

```typescript
import { z } from 'zod';

// 방법 1: 메서드에 직접 지정
const schema1 = z.string()
  .min(2, '2자 이상 입력하세요')
  .max(50, '50자 이하로 입력하세요')
  .email('올바른 이메일 형식이 아닙니다');

// 방법 2: 객체 형태로 지정
const schema2 = z.string().min(2, {
  message: '2자 이상 입력하세요',
});

// 방법 3: errorMap으로 전역 설정
const customErrorMap: z.ZodErrorMap = (issue, ctx) => {
  if (issue.code === z.ZodIssueCode.invalid_type) {
    if (issue.expected === 'string') {
      return { message: '문자열을 입력하세요' };
    }
    if (issue.expected === 'number') {
      return { message: '숫자를 입력하세요' };
    }
  }
  if (issue.code === z.ZodIssueCode.too_small) {
    return { message: `최소 ${issue.minimum}자 이상이어야 합니다` };
  }
  return { message: ctx.defaultError };
};

z.setErrorMap(customErrorMap);

// 방법 4: 스키마별 errorMap
const schema3 = z.number({
  errorMap: (issue, ctx) => {
    if (issue.code === z.ZodIssueCode.invalid_type) {
      return { message: '나이는 숫자여야 합니다' };
    }
    return { message: ctx.defaultError };
  },
});
```

### 에러 메시지 한글화 예시

```typescript
import { z } from 'zod';

const koreanErrorMap: z.ZodErrorMap = (issue, ctx) => {
  switch (issue.code) {
    case z.ZodIssueCode.invalid_type:
      if (issue.received === 'undefined') {
        return { message: '필수 항목입니다' };
      }
      return { message: `${issue.expected} 타입이 필요합니다` };

    case z.ZodIssueCode.too_small:
      if (issue.type === 'string') {
        return { message: `${issue.minimum}자 이상 입력하세요` };
      }
      if (issue.type === 'number') {
        return { message: `${issue.minimum} 이상이어야 합니다` };
      }
      if (issue.type === 'array') {
        return { message: `최소 ${issue.minimum}개 이상 선택하세요` };
      }
      break;

    case z.ZodIssueCode.too_big:
      if (issue.type === 'string') {
        return { message: `${issue.maximum}자 이하로 입력하세요` };
      }
      if (issue.type === 'number') {
        return { message: `${issue.maximum} 이하여야 합니다` };
      }
      break;

    case z.ZodIssueCode.invalid_string:
      if (issue.validation === 'email') {
        return { message: '올바른 이메일 형식이 아닙니다' };
      }
      if (issue.validation === 'url') {
        return { message: '올바른 URL 형식이 아닙니다' };
      }
      break;
  }

  return { message: ctx.defaultError };
};

z.setErrorMap(koreanErrorMap);
```

---

## React Hook Form 통합

Zod와 React Hook Form을 함께 사용하면 강력한 타입 안전성과 폼 관리를 동시에 얻을 수 있습니다. 더 자세한 React Hook Form 사용법은 [React Hook Form 완벽 가이드](/posts/react-hook-form-complete-guide/)를 참고하세요.

### 설치

```bash
npm install react-hook-form @hookform/resolvers zod
```

### 기본 사용법

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string()
    .min(1, '이메일을 입력하세요')
    .email('올바른 이메일 형식이 아닙니다'),
  password: z.string()
    .min(8, '비밀번호는 8자 이상이어야 합니다'),
});

type LoginFormData = z.infer<typeof loginSchema>;

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    console.log('로그인 시도:', data);
    // API 호출
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          type="email"
          {...register('email')}
        />
        {errors.email && (
          <span className="text-red-500">{errors.email.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input
          id="password"
          type="password"
          {...register('password')}
        />
        {errors.password && (
          <span className="text-red-500">{errors.password.message}</span>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '로그인 중...' : '로그인'}
      </button>
    </form>
  );
}
```

### 회원가입 폼 예제

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const signupSchema = z.object({
  username: z.string()
    .min(3, '3자 이상 입력하세요')
    .max(20, '20자 이하로 입력하세요')
    .regex(/^[a-zA-Z0-9_]+$/, '영문, 숫자, 밑줄만 사용 가능합니다'),
  email: z.string()
    .min(1, '이메일을 입력하세요')
    .email('올바른 이메일 형식이 아닙니다'),
  password: z.string()
    .min(8, '8자 이상 입력하세요')
    .regex(/[A-Z]/, '대문자를 포함해야 합니다')
    .regex(/[a-z]/, '소문자를 포함해야 합니다')
    .regex(/[0-9]/, '숫자를 포함해야 합니다'),
  confirmPassword: z.string(),
  age: z.coerce
    .number({ invalid_type_error: '나이를 입력하세요' })
    .min(14, '14세 이상만 가입 가능합니다')
    .max(120, '올바른 나이를 입력하세요'),
  terms: z.literal(true, {
    errorMap: () => ({ message: '이용약관에 동의해야 합니다' }),
  }),
}).refine((data) => data.password === data.confirmPassword, {
  message: '비밀번호가 일치하지 않습니다',
  path: ['confirmPassword'],
});

type SignupFormData = z.infer<typeof signupSchema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm<SignupFormData>({
    resolver: zodResolver(signupSchema),
    defaultValues: {
      username: '',
      email: '',
      password: '',
      confirmPassword: '',
    },
  });

  const onSubmit = async (data: SignupFormData) => {
    try {
      const response = await fetch('/api/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const errorData = await response.json();

        if (errorData.code === 'EMAIL_EXISTS') {
          setError('email', {
            type: 'server',
            message: '이미 사용 중인 이메일입니다',
          });
          return;
        }

        if (errorData.code === 'USERNAME_EXISTS') {
          setError('username', {
            type: 'server',
            message: '이미 사용 중인 사용자명입니다',
          });
          return;
        }
      }

      alert('회원가입이 완료되었습니다!');
    } catch (error) {
      setError('root', {
        type: 'server',
        message: '서버 오류가 발생했습니다',
      });
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4 max-w-md">
      {errors.root && (
        <div className="bg-red-100 text-red-700 p-3 rounded">
          {errors.root.message}
        </div>
      )}

      <div>
        <label htmlFor="username">사용자명</label>
        <input id="username" {...register('username')} />
        {errors.username && (
          <span className="text-red-500 text-sm">{errors.username.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">이메일</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && (
          <span className="text-red-500 text-sm">{errors.email.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호</label>
        <input id="password" type="password" {...register('password')} />
        {errors.password && (
          <span className="text-red-500 text-sm">{errors.password.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="confirmPassword">비밀번호 확인</label>
        <input id="confirmPassword" type="password" {...register('confirmPassword')} />
        {errors.confirmPassword && (
          <span className="text-red-500 text-sm">{errors.confirmPassword.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="age">나이</label>
        <input id="age" type="number" {...register('age')} />
        {errors.age && (
          <span className="text-red-500 text-sm">{errors.age.message}</span>
        )}
      </div>

      <div>
        <label className="flex items-center gap-2">
          <input type="checkbox" {...register('terms')} />
          <span>이용약관에 동의합니다</span>
        </label>
        {errors.terms && (
          <span className="text-red-500 text-sm">{errors.terms.message}</span>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white py-2 rounded disabled:opacity-50"
      >
        {isSubmitting ? '처리 중...' : '회원가입'}
      </button>
    </form>
  );
}
```

---

## 실전 패턴

### API 응답 검증

```typescript
import { z } from 'zod';

// API 응답 스키마 정의
const userResponseSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().transform((val) => new Date(val)),
});

const apiResponseSchema = z.object({
  success: z.boolean(),
  data: userResponseSchema.nullable(),
  error: z.object({
    code: z.string(),
    message: z.string(),
  }).nullable(),
});

type ApiResponse = z.infer<typeof apiResponseSchema>;

// 타입 안전한 API 호출 함수
async function fetchUser(id: number): Promise<z.infer<typeof userResponseSchema>> {
  const response = await fetch(`/api/users/${id}`);
  const json = await response.json();

  const result = apiResponseSchema.safeParse(json);

  if (!result.success) {
    throw new Error(`API 응답 형식 오류: ${result.error.message}`);
  }

  if (!result.data.success || !result.data.data) {
    throw new Error(result.data.error?.message || '알 수 없는 오류');
  }

  return result.data.data;
}

// 사용 예시
async function displayUser() {
  try {
    const user = await fetchUser(1);
    console.log(`사용자: ${user.name}`);
    console.log(`가입일: ${user.createdAt.toLocaleDateString()}`);
  } catch (error) {
    console.error('사용자 조회 실패:', error);
  }
}
```

### 환경 변수 검증

```typescript
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1, 'API_KEY는 필수입니다'),
  REDIS_URL: z.string().url().optional(),
  DEBUG: z.coerce.boolean().default(false),
  MAX_CONNECTIONS: z.coerce.number().min(1).max(100).default(10),
});

type Env = z.infer<typeof envSchema>;

// 앱 시작 시 환경변수 검증
function validateEnv(): Env {
  const result = envSchema.safeParse(process.env);

  if (!result.success) {
    console.error('환경변수 검증 실패:');
    result.error.issues.forEach((issue) => {
      console.error(`  - ${issue.path.join('.')}: ${issue.message}`);
    });
    process.exit(1);
  }

  return result.data;
}

export const env = validateEnv();
```

### URL 쿼리 파라미터 검증

```typescript
import { z } from 'zod';

const paginationSchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  sort: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().optional(),
  category: z.string().optional(),
});

type PaginationParams = z.infer<typeof paginationSchema>;

function parseQueryParams(searchParams: URLSearchParams): PaginationParams {
  const params = Object.fromEntries(searchParams.entries());
  return paginationSchema.parse(params);
}

// Next.js App Router 예시
// app/products/page.tsx
export default function ProductsPage({
  searchParams,
}: {
  searchParams: { [key: string]: string | string[] | undefined };
}) {
  const result = paginationSchema.safeParse(searchParams);

  if (!result.success) {
    // 기본값 사용
    return <ProductList page={1} limit={20} sort="desc" />;
  }

  return <ProductList {...result.data} />;
}
```

Next.js App Router의 더 자세한 사용법은 [Next.js 15 App Router 완벽 가이드](/posts/nextjs-15-app-router-guide/)를 참고하세요.

### localStorage 데이터 검증

```typescript
import { z } from 'zod';

const userPreferencesSchema = z.object({
  theme: z.enum(['light', 'dark', 'system']).default('system'),
  language: z.enum(['ko', 'en', 'ja']).default('ko'),
  notifications: z.boolean().default(true),
  fontSize: z.enum(['small', 'medium', 'large']).default('medium'),
});

type UserPreferences = z.infer<typeof userPreferencesSchema>;

const STORAGE_KEY = 'user_preferences';

function loadPreferences(): UserPreferences {
  try {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (!stored) {
      return userPreferencesSchema.parse({});
    }

    const parsed = JSON.parse(stored);
    return userPreferencesSchema.parse(parsed);
  } catch {
    // 파싱 실패 시 기본값 반환
    return userPreferencesSchema.parse({});
  }
}

function savePreferences(preferences: Partial<UserPreferences>): void {
  const current = loadPreferences();
  const updated = { ...current, ...preferences };

  // 저장 전 검증
  const validated = userPreferencesSchema.parse(updated);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(validated));
}
```

### 재사용 가능한 스키마 패턴

```typescript
import { z } from 'zod';

// 공통 스키마 정의
const idSchema = z.object({
  id: z.number().positive(),
});

const timestampsSchema = z.object({
  createdAt: z.coerce.date(),
  updatedAt: z.coerce.date(),
});

const softDeleteSchema = z.object({
  deletedAt: z.coerce.date().nullable(),
});

// 기본 사용자 정보
const userBaseSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

// 완전한 사용자 스키마 조합
const userSchema = idSchema
  .merge(userBaseSchema)
  .merge(timestampsSchema)
  .merge(softDeleteSchema);

type User = z.infer<typeof userSchema>;

// 생성용 스키마 (id, timestamps 제외)
const createUserSchema = userBaseSchema.extend({
  password: z.string().min(8),
});

type CreateUser = z.infer<typeof createUserSchema>;

// 수정용 스키마 (모든 필드 optional)
const updateUserSchema = userBaseSchema.partial();

type UpdateUser = z.infer<typeof updateUserSchema>;

// API 응답 래퍼
function createApiResponseSchema<T extends z.ZodTypeAny>(dataSchema: T) {
  return z.object({
    success: z.literal(true),
    data: dataSchema,
    meta: z.object({
      timestamp: z.string(),
      requestId: z.string(),
    }).optional(),
  });
}

const userResponseSchema = createApiResponseSchema(userSchema);
const usersListResponseSchema = createApiResponseSchema(z.array(userSchema));
```

---

## 성능 고려사항

### 스키마 캐싱

스키마는 함수 외부에서 정의하여 재사용하세요.

```typescript
import { z } from 'zod';

// 좋은 예: 스키마를 모듈 레벨에서 정의
const userSchema = z.object({
  id: z.number(),
  name: z.string(),
});

function validateUser(data: unknown) {
  return userSchema.safeParse(data);
}

// 나쁜 예: 함수 호출마다 스키마 생성
function validateUserBad(data: unknown) {
  const schema = z.object({ // 매번 새로 생성됨
    id: z.number(),
    name: z.string(),
  });
  return schema.safeParse(data);
}
```

### 조기 종료 (Early Termination)

`safeParse`는 모든 에러를 수집합니다. 첫 번째 에러에서 멈추려면 `parse`를 try-catch와 함께 사용하세요.

```typescript
import { z } from 'zod';

const schema = z.object({
  a: z.string(),
  b: z.number(),
  c: z.boolean(),
});

// 모든 에러 수집 (기본)
const result = schema.safeParse({ a: 1, b: 'x', c: 'y' });
// 3개의 에러 모두 포함

// 첫 번째 에러에서 멈춤
try {
  schema.parse({ a: 1, b: 'x', c: 'y' });
} catch (error) {
  if (error instanceof z.ZodError) {
    // 첫 번째 에러만 확인 가능
  }
}
```

### Lazy 스키마 (순환 참조)

자기 참조나 순환 참조가 있는 스키마는 `z.lazy`를 사용합니다.

```typescript
import { z } from 'zod';

// 트리 구조
interface TreeNode {
  value: string;
  children: TreeNode[];
}

const treeNodeSchema: z.ZodType<TreeNode> = z.lazy(() =>
  z.object({
    value: z.string(),
    children: z.array(treeNodeSchema),
  })
);

// 상호 참조
interface User {
  name: string;
  posts: Post[];
}

interface Post {
  title: string;
  author: User;
}

const postSchema: z.ZodType<Post> = z.lazy(() =>
  z.object({
    title: z.string(),
    author: userSchema,
  })
);

const userSchema: z.ZodType<User> = z.lazy(() =>
  z.object({
    name: z.string(),
    posts: z.array(postSchema),
  })
);
```

---

## 마무리

Zod는 TypeScript 프로젝트에서 런타임 타입 안전성을 보장하는 강력한 도구입니다.

### 핵심 포인트 정리

| 개념 | 설명 |
|------|------|
| 스키마 정의 | `z.object()`, `z.array()` 등으로 데이터 구조 정의 |
| 타입 추론 | `z.infer<typeof schema>`로 TypeScript 타입 생성 |
| 검증 | `parse()` (throw) / `safeParse()` (결과 객체) |
| 변환 | `transform()`, `coerce`로 값 변환 |
| 정제 | `refine()`, `superRefine()`으로 커스텀 검증 |
| 조합 | `union()`, `discriminatedUnion()`, `intersection()` |

### 권장 사용 패턴

1. **스키마 우선 설계**: 타입을 먼저 정의하지 말고 스키마를 먼저 정의하세요
2. **safeParse 사용**: `parse` 대신 `safeParse`로 안전하게 에러 처리
3. **스키마 재사용**: 공통 스키마를 모듈화하여 재사용
4. **커스텀 에러 메시지**: 사용자 친화적인 한글 에러 메시지 설정
5. **환경변수 검증**: 앱 시작 시 필수 설정값 검증

### 언제 Zod를 사용해야 할까?

- API 응답 데이터 검증
- 폼 입력값 검증 (React Hook Form과 함께)
- 환경변수 검증
- URL 파라미터 검증
- localStorage/sessionStorage 데이터 검증
- WebSocket 메시지 검증

Zod를 통해 TypeScript의 정적 타입 시스템과 런타임 검증을 완벽하게 연결하여, 안전하고 신뢰할 수 있는 애플리케이션을 구축하세요.

---

## 참고 자료

- [Zod 공식 문서](https://zod.dev/)
- [Zod GitHub](https://github.com/colinhacks/zod)
- [@hookform/resolvers](https://github.com/react-hook-form/resolvers)
- [React Hook Form 완벽 가이드](/posts/react-hook-form-complete-guide/)
