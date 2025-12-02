---
title: "React Hook Form 완벽 가이드: 폼 상태 관리와 유효성 검사"
description: "React Hook Form의 핵심 개념부터 고급 패턴까지. 성능 최적화, Zod 통합, 복잡한 폼 처리 방법을 실전 예제와 함께 알아봅니다."
date: 2025-12-02 09:00:00 +0900
categories: [Frontend, React]
tags: [react, react-hook-form, form, validation, zod, typescript]
---

## 개요

React에서 폼을 다루는 것은 생각보다 복잡합니다. 입력 값 관리, 유효성 검사, 에러 처리, 제출 상태 관리 등 고려해야 할 사항이 많습니다. **React Hook Form**은 이러한 복잡성을 해결하면서도 뛰어난 성능을 제공하는 폼 라이브러리입니다.

이 글에서는 React Hook Form의 기초부터 고급 패턴까지 실전 예제와 함께 상세히 알아봅니다.

---

## React 폼 관리의 어려움

### Controlled vs Uncontrolled 컴포넌트

React에서 폼 입력을 처리하는 방식은 크게 두 가지입니다.

**Controlled 컴포넌트:**

```tsx
import { useState } from 'react';

function ControlledForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({ name, email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="이름"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="이메일"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="비밀번호"
      />
      <button type="submit">제출</button>
    </form>
  );
}
```

**문제점:**
- 입력 필드마다 state와 onChange 핸들러 필요
- 매 입력마다 컴포넌트 전체 리렌더링
- 필드가 많아지면 보일러플레이트 코드 급증

**Uncontrolled 컴포넌트:**

```tsx
import { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef<HTMLInputElement>(null);
  const emailRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({
      name: nameRef.current?.value,
      email: emailRef.current?.value,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} placeholder="이름" />
      <input ref={emailRef} placeholder="이메일" />
      <button type="submit">제출</button>
    </form>
  );
}
```

**문제점:**
- 유효성 검사 구현이 복잡
- 실시간 값 추적이 어려움
- 폼 상태 관리가 불편

### 일반적인 폼 요구사항

실제 프로젝트에서는 다음과 같은 기능들이 필요합니다:

| 요구사항 | 설명 |
|----------|------|
| 유효성 검사 | 필수 입력, 형식 검증, 조건부 검증 |
| 에러 메시지 | 사용자 친화적인 오류 표시 |
| 제출 상태 | 로딩, 성공, 실패 상태 관리 |
| 동적 필드 | 필드 추가/삭제 |
| 성능 | 불필요한 리렌더링 방지 |

React Hook Form은 이 모든 요구사항을 효율적으로 해결합니다.

---

## React Hook Form 소개

### 주요 특징

**1. 최소한의 리렌더링**

React Hook Form은 **uncontrolled 컴포넌트** 방식을 기반으로 합니다. 입력 값이 변경되어도 전체 폼이 리렌더링되지 않습니다.

**2. 간결한 API**

```tsx
const { register, handleSubmit } = useForm();
```

단 두 개의 함수로 기본적인 폼 기능을 구현할 수 있습니다.

**3. 뛰어난 TypeScript 지원**

제네릭을 통해 폼 데이터 타입을 완벽하게 추론합니다.

**4. 작은 번들 크기**

의존성 없이 약 **8.6KB** (gzip 기준)의 작은 크기를 유지합니다.

### 다른 라이브러리와 비교

| 특성 | React Hook Form | Formik | Redux Form |
|------|-----------------|--------|------------|
| 번들 크기 | ~8.6KB | ~12.7KB | ~26.4KB |
| 리렌더링 | 최소화 | 매 입력 | 매 입력 |
| 방식 | Uncontrolled | Controlled | Controlled |
| 학습 곡선 | 낮음 | 중간 | 높음 |
| TypeScript | 우수 | 좋음 | 보통 |

---

## 설치 및 기본 사용법

### 설치

```bash
# npm
npm install react-hook-form

# yarn
yarn add react-hook-form

# pnpm
pnpm add react-hook-form
```

### 기본 구조

```tsx
import { useForm } from 'react-hook-form';

interface FormData {
  firstName: string;
  lastName: string;
  email: string;
}

function BasicForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>();

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('firstName')} placeholder="이름" />
      <input {...register('lastName')} placeholder="성" />
      <input {...register('email')} placeholder="이메일" />
      <button type="submit">제출</button>
    </form>
  );
}
```

### useForm 훅 이해하기

`useForm`은 React Hook Form의 핵심 훅입니다.

```tsx
const {
  register,      // 입력 필드 등록
  handleSubmit,  // 폼 제출 처리
  formState,     // 폼 상태 (errors, isSubmitting 등)
  watch,         // 값 구독 (리렌더링 발생)
  getValues,     // 값 가져오기 (리렌더링 없음)
  setValue,      // 값 설정
  reset,         // 폼 초기화
  trigger,       // 수동 유효성 검사
  control,       // Controller용 객체
} = useForm<FormData>({
  mode: 'onSubmit',           // 유효성 검사 시점
  reValidateMode: 'onChange', // 재검사 시점
  defaultValues: {            // 기본값
    firstName: '',
    lastName: '',
    email: '',
  },
});
```

### mode 옵션

유효성 검사가 실행되는 시점을 설정합니다.

| mode | 설명 |
|------|------|
| `onSubmit` | 제출 시에만 검사 (기본값) |
| `onBlur` | 포커스 해제 시 검사 |
| `onChange` | 값 변경 시마다 검사 |
| `onTouched` | 첫 blur 후 onChange로 전환 |
| `all` | blur와 change 모두에서 검사 |

```tsx
const { register, handleSubmit } = useForm({
  mode: 'onBlur', // 포커스를 벗어날 때 검사
});
```

### register 함수

`register`는 입력 필드를 React Hook Form에 등록합니다.

```tsx
// 기본 사용
<input {...register('username')} />

// register가 반환하는 객체
{
  name: 'username',
  ref: (element) => { /* ref 콜백 */ },
  onChange: (e) => { /* change 핸들러 */ },
  onBlur: (e) => { /* blur 핸들러 */ },
}
```

### handleSubmit 함수

폼 제출을 처리하며, 유효성 검사를 통과한 경우에만 콜백을 실행합니다.

```tsx
const onSubmit = (data: FormData) => {
  // 유효성 검사 통과 시 실행
  console.log('성공:', data);
};

const onError = (errors: FieldErrors<FormData>) => {
  // 유효성 검사 실패 시 실행
  console.log('에러:', errors);
};

<form onSubmit={handleSubmit(onSubmit, onError)}>
```

---

## 유효성 검사

### 기본 Validation 규칙

`register` 함수의 두 번째 인자로 유효성 규칙을 전달합니다.

```tsx
interface FormData {
  username: string;
  email: string;
  password: string;
  age: number;
  website: string;
}

function ValidationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>();

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {/* 필수 입력 */}
      <input
        {...register('username', {
          required: '사용자명은 필수입니다',
        })}
      />
      {errors.username && <span>{errors.username.message}</span>}

      {/* 이메일 형식 */}
      <input
        {...register('email', {
          required: '이메일은 필수입니다',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: '올바른 이메일 형식이 아닙니다',
          },
        })}
      />
      {errors.email && <span>{errors.email.message}</span>}

      {/* 최소/최대 길이 */}
      <input
        type="password"
        {...register('password', {
          required: '비밀번호는 필수입니다',
          minLength: {
            value: 8,
            message: '비밀번호는 8자 이상이어야 합니다',
          },
          maxLength: {
            value: 20,
            message: '비밀번호는 20자 이하여야 합니다',
          },
        })}
      />
      {errors.password && <span>{errors.password.message}</span>}

      {/* 숫자 범위 */}
      <input
        type="number"
        {...register('age', {
          required: '나이는 필수입니다',
          min: {
            value: 18,
            message: '18세 이상이어야 합니다',
          },
          max: {
            value: 99,
            message: '99세 이하여야 합니다',
          },
        })}
      />
      {errors.age && <span>{errors.age.message}</span>}

      <button type="submit">제출</button>
    </form>
  );
}
```

### 유효성 규칙 정리

| 규칙 | 설명 | 예시 |
|------|------|------|
| `required` | 필수 입력 | `required: '필수 항목입니다'` |
| `minLength` | 최소 길이 | `minLength: { value: 3, message: '3자 이상' }` |
| `maxLength` | 최대 길이 | `maxLength: { value: 20, message: '20자 이하' }` |
| `min` | 최소 값 | `min: { value: 0, message: '0 이상' }` |
| `max` | 최대 값 | `max: { value: 100, message: '100 이하' }` |
| `pattern` | 정규식 패턴 | `pattern: { value: /regex/, message: '형식 오류' }` |
| `validate` | 커스텀 검증 | `validate: (value) => value === 'test' \|\| '오류'` |

### 커스텀 Validation

`validate` 옵션으로 복잡한 검증 로직을 구현할 수 있습니다.

```tsx
interface FormData {
  password: string;
  confirmPassword: string;
  username: string;
}

function CustomValidationForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors },
  } = useForm<FormData>();

  const password = watch('password');

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input
        type="password"
        {...register('password', {
          required: '비밀번호를 입력하세요',
          validate: {
            // 여러 검증 규칙을 객체로 전달
            hasUpperCase: (value) =>
              /[A-Z]/.test(value) || '대문자를 포함해야 합니다',
            hasLowerCase: (value) =>
              /[a-z]/.test(value) || '소문자를 포함해야 합니다',
            hasNumber: (value) =>
              /[0-9]/.test(value) || '숫자를 포함해야 합니다',
            hasSpecialChar: (value) =>
              /[!@#$%^&*]/.test(value) || '특수문자를 포함해야 합니다',
          },
        })}
        placeholder="비밀번호"
      />
      {errors.password && <span>{errors.password.message}</span>}

      <input
        type="password"
        {...register('confirmPassword', {
          required: '비밀번호 확인을 입력하세요',
          validate: (value) =>
            value === password || '비밀번호가 일치하지 않습니다',
        })}
        placeholder="비밀번호 확인"
      />
      {errors.confirmPassword && (
        <span>{errors.confirmPassword.message}</span>
      )}

      {/* 비동기 검증 */}
      <input
        {...register('username', {
          required: '사용자명을 입력하세요',
          validate: async (value) => {
            // API 호출로 중복 확인
            const response = await fetch(
              `/api/check-username?username=${value}`
            );
            const { available } = await response.json();
            return available || '이미 사용 중인 사용자명입니다';
          },
        })}
        placeholder="사용자명"
      />
      {errors.username && <span>{errors.username.message}</span>}

      <button type="submit">제출</button>
    </form>
  );
}
```

### 에러 메시지 컴포넌트

에러 표시를 위한 재사용 가능한 컴포넌트를 만들 수 있습니다.

```tsx
import { FieldError } from 'react-hook-form';

interface ErrorMessageProps {
  error?: FieldError;
  className?: string;
}

function ErrorMessage({ error, className = '' }: ErrorMessageProps) {
  if (!error) return null;

  return (
    <span className={`text-red-500 text-sm ${className}`}>
      {error.message}
    </span>
  );
}

// 사용 예시
function Form() {
  const { register, formState: { errors } } = useForm();

  return (
    <div>
      <input {...register('email', { required: '이메일 필수' })} />
      <ErrorMessage error={errors.email} />
    </div>
  );
}
```

---

## Zod 통합

**Zod**는 TypeScript 우선 스키마 선언 및 유효성 검사 라이브러리입니다. React Hook Form과 함께 사용하면 강력한 타입 안전성을 얻을 수 있습니다.

### 설치

```bash
npm install zod @hookform/resolvers
```

### 기본 사용법

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Zod 스키마 정의
const schema = z.object({
  username: z
    .string()
    .min(3, '사용자명은 3자 이상이어야 합니다')
    .max(20, '사용자명은 20자 이하여야 합니다'),
  email: z
    .string()
    .email('올바른 이메일 형식이 아닙니다'),
  password: z
    .string()
    .min(8, '비밀번호는 8자 이상이어야 합니다')
    .regex(/[A-Z]/, '대문자를 포함해야 합니다')
    .regex(/[0-9]/, '숫자를 포함해야 합니다'),
});

// 스키마에서 타입 추론
type FormData = z.infer<typeof schema>;

function ZodForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('username')} placeholder="사용자명" />
        {errors.username && <span>{errors.username.message}</span>}
      </div>

      <div>
        <input {...register('email')} placeholder="이메일" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>

      <div>
        <input
          type="password"
          {...register('password')}
          placeholder="비밀번호"
        />
        {errors.password && <span>{errors.password.message}</span>}
      </div>

      <button type="submit">제출</button>
    </form>
  );
}
```

### 고급 Zod 스키마

```tsx
import { z } from 'zod';

// 비밀번호 확인이 포함된 스키마
const signupSchema = z
  .object({
    username: z
      .string()
      .min(3, '3자 이상 입력하세요')
      .regex(/^[a-zA-Z0-9_]+$/, '영문, 숫자, 밑줄만 사용 가능합니다'),
    email: z.string().email('올바른 이메일을 입력하세요'),
    password: z
      .string()
      .min(8, '8자 이상 입력하세요')
      .regex(/[A-Z]/, '대문자 포함 필수')
      .regex(/[a-z]/, '소문자 포함 필수')
      .regex(/[0-9]/, '숫자 포함 필수')
      .regex(/[^A-Za-z0-9]/, '특수문자 포함 필수'),
    confirmPassword: z.string(),
    age: z.coerce // 문자열을 숫자로 변환
      .number()
      .min(18, '18세 이상이어야 합니다')
      .max(120, '올바른 나이를 입력하세요'),
    terms: z.literal(true, {
      errorMap: () => ({ message: '약관에 동의해야 합니다' }),
    }),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: '비밀번호가 일치하지 않습니다',
    path: ['confirmPassword'], // 에러가 표시될 필드
  });

type SignupFormData = z.infer<typeof signupSchema>;
```

### 조건부 유효성 검사

```tsx
const orderSchema = z
  .object({
    deliveryType: z.enum(['pickup', 'delivery']),
    address: z.string().optional(),
    zipCode: z.string().optional(),
  })
  .refine(
    (data) => {
      if (data.deliveryType === 'delivery') {
        return data.address && data.address.length > 0;
      }
      return true;
    },
    {
      message: '배송 선택 시 주소는 필수입니다',
      path: ['address'],
    }
  )
  .refine(
    (data) => {
      if (data.deliveryType === 'delivery') {
        return data.zipCode && /^\d{5}$/.test(data.zipCode);
      }
      return true;
    },
    {
      message: '올바른 우편번호를 입력하세요',
      path: ['zipCode'],
    }
  );
```

### Zod와 함께 사용하는 완전한 예제

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const profileSchema = z.object({
  name: z.string().min(2, '이름은 2자 이상이어야 합니다'),
  bio: z.string().max(200, '소개는 200자 이하여야 합니다').optional(),
  website: z
    .string()
    .url('올바른 URL을 입력하세요')
    .optional()
    .or(z.literal('')), // 빈 문자열 허용
  birthDate: z.coerce.date().max(new Date(), '미래 날짜는 선택할 수 없습니다'),
  role: z.enum(['user', 'admin', 'moderator'], {
    errorMap: () => ({ message: '역할을 선택하세요' }),
  }),
});

type ProfileFormData = z.infer<typeof profileSchema>;

function ProfileForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<ProfileFormData>({
    resolver: zodResolver(profileSchema),
    defaultValues: {
      name: '',
      bio: '',
      website: '',
      role: 'user',
    },
  });

  const onSubmit = async (data: ProfileFormData) => {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    console.log('제출된 데이터:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label htmlFor="name">이름</label>
        <input id="name" {...register('name')} />
        {errors.name && (
          <p className="text-red-500">{errors.name.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="bio">소개</label>
        <textarea id="bio" {...register('bio')} />
        {errors.bio && (
          <p className="text-red-500">{errors.bio.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="website">웹사이트</label>
        <input id="website" {...register('website')} />
        {errors.website && (
          <p className="text-red-500">{errors.website.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="birthDate">생년월일</label>
        <input id="birthDate" type="date" {...register('birthDate')} />
        {errors.birthDate && (
          <p className="text-red-500">{errors.birthDate.message}</p>
        )}
      </div>

      <div>
        <label htmlFor="role">역할</label>
        <select id="role" {...register('role')}>
          <option value="">선택하세요</option>
          <option value="user">사용자</option>
          <option value="admin">관리자</option>
          <option value="moderator">모더레이터</option>
        </select>
        {errors.role && (
          <p className="text-red-500">{errors.role.message}</p>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '저장 중...' : '저장'}
      </button>
    </form>
  );
}
```

---

## 고급 패턴

### useFieldArray - 동적 필드 관리

`useFieldArray`는 동적으로 필드를 추가/삭제할 수 있게 해줍니다.

```tsx
import { useForm, useFieldArray } from 'react-hook-form';

interface FormData {
  users: {
    name: string;
    email: string;
  }[];
}

function DynamicFieldsForm() {
  const { register, control, handleSubmit } = useForm<FormData>({
    defaultValues: {
      users: [{ name: '', email: '' }],
    },
  });

  const { fields, append, remove, move } = useFieldArray({
    control,
    name: 'users',
  });

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {fields.map((field, index) => (
        <div key={field.id} className="flex gap-2 mb-2">
          <input
            {...register(`users.${index}.name` as const)}
            placeholder="이름"
          />
          <input
            {...register(`users.${index}.email` as const)}
            placeholder="이메일"
          />
          <button type="button" onClick={() => remove(index)}>
            삭제
          </button>
          {index > 0 && (
            <button type="button" onClick={() => move(index, index - 1)}>
              위로
            </button>
          )}
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ name: '', email: '' })}
      >
        사용자 추가
      </button>

      <button type="submit">제출</button>
    </form>
  );
}
```

### useFieldArray 메서드 정리

| 메서드 | 설명 |
|--------|------|
| `append(obj)` | 배열 끝에 항목 추가 |
| `prepend(obj)` | 배열 앞에 항목 추가 |
| `insert(index, obj)` | 특정 위치에 항목 삽입 |
| `remove(index)` | 특정 위치 항목 삭제 |
| `swap(from, to)` | 두 항목 위치 교환 |
| `move(from, to)` | 항목을 다른 위치로 이동 |
| `update(index, obj)` | 특정 위치 항목 업데이트 |
| `replace(arr)` | 전체 배열 교체 |

### useWatch - 값 구독

`useWatch`는 특정 필드의 값을 구독합니다. `watch`와 달리 독립적인 리렌더링을 발생시킵니다.

```tsx
import { useForm, useWatch } from 'react-hook-form';

interface FormData {
  firstName: string;
  lastName: string;
}

function WatchExample() {
  const { register, control } = useForm<FormData>();

  return (
    <form>
      <input {...register('firstName')} />
      <input {...register('lastName')} />
      <Preview control={control} />
    </form>
  );
}

// 별도 컴포넌트에서 값 구독 - 이 컴포넌트만 리렌더링됨
function Preview({ control }: { control: any }) {
  const [firstName, lastName] = useWatch({
    control,
    name: ['firstName', 'lastName'],
  });

  return (
    <p>
      미리보기: {firstName} {lastName}
    </p>
  );
}
```

### useFormContext - 컨텍스트 공유

`FormProvider`와 `useFormContext`를 사용하면 중첩된 컴포넌트에서 폼 메서드에 접근할 수 있습니다.

```tsx
import {
  useForm,
  FormProvider,
  useFormContext,
} from 'react-hook-form';

interface FormData {
  email: string;
  password: string;
  profile: {
    name: string;
    bio: string;
  };
}

// 부모 컴포넌트
function ParentForm() {
  const methods = useForm<FormData>();

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <EmailInput />
        <PasswordInput />
        <ProfileSection />
        <button type="submit">제출</button>
      </form>
    </FormProvider>
  );
}

// 자식 컴포넌트 - props 없이 폼 메서드 사용
function EmailInput() {
  const { register, formState: { errors } } = useFormContext<FormData>();

  return (
    <div>
      <input {...register('email')} placeholder="이메일" />
      {errors.email && <span>{errors.email.message}</span>}
    </div>
  );
}

function PasswordInput() {
  const { register } = useFormContext<FormData>();

  return <input type="password" {...register('password')} />;
}

function ProfileSection() {
  const { register } = useFormContext<FormData>();

  return (
    <div>
      <input {...register('profile.name')} placeholder="이름" />
      <textarea {...register('profile.bio')} placeholder="소개" />
    </div>
  );
}
```

### Controller - UI 라이브러리 통합

외부 UI 라이브러리(MUI, Ant Design 등)와 통합할 때 `Controller`를 사용합니다.

{% raw %}
```tsx
import { useForm, Controller } from 'react-hook-form';
import Select from 'react-select';
import DatePicker from 'react-datepicker';

interface FormData {
  category: { value: string; label: string } | null;
  startDate: Date | null;
  rating: number;
}

function ControllerExample() {
  const { control, handleSubmit } = useForm<FormData>({
    defaultValues: {
      category: null,
      startDate: null,
      rating: 0,
    },
  });

  const categoryOptions = [
    { value: 'tech', label: '기술' },
    { value: 'design', label: '디자인' },
    { value: 'business', label: '비즈니스' },
  ];

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {/* react-select 통합 */}
      <Controller
        name="category"
        control={control}
        rules={{ required: '카테고리를 선택하세요' }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <Select
              {...field}
              options={categoryOptions}
              placeholder="카테고리 선택"
            />
            {error && <span>{error.message}</span>}
          </div>
        )}
      />

      {/* DatePicker 통합 */}
      <Controller
        name="startDate"
        control={control}
        rules={{ required: '날짜를 선택하세요' }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <DatePicker
              selected={field.value}
              onChange={field.onChange}
              dateFormat="yyyy-MM-dd"
              placeholderText="날짜 선택"
            />
            {error && <span>{error.message}</span>}
          </div>
        )}
      />

      {/* 커스텀 Rating 컴포넌트 */}
      <Controller
        name="rating"
        control={control}
        rules={{
          validate: (value) => value > 0 || '평점을 선택하세요',
        }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <div className="flex gap-1">
              {[1, 2, 3, 4, 5].map((star) => (
                <button
                  key={star}
                  type="button"
                  onClick={() => field.onChange(star)}
                  className={star <= field.value ? 'text-yellow-400' : ''}
                >
                  ★
                </button>
              ))}
            </div>
            {error && <span>{error.message}</span>}
          </div>
        )}
      />

      <button type="submit">제출</button>
    </form>
  );
}
```
{% endraw %}

---

## 성능 최적화

### 리렌더링 최소화 원리

React Hook Form이 빠른 이유는 **uncontrolled 컴포넌트** 방식을 사용하기 때문입니다.

```tsx
// Controlled 방식 - 매 입력마다 리렌더링
function ControlledInput() {
  const [value, setValue] = useState('');
  console.log('렌더링!'); // 매번 출력

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// React Hook Form - 리렌더링 없음
function RHFInput() {
  const { register } = useForm();
  console.log('렌더링!'); // 최초 1회만 출력

  return <input {...register('name')} />;
}
```

### formState 구독 최적화

`formState`를 구조 분해하면 해당 상태가 변경될 때만 리렌더링됩니다.

```tsx
// 모든 formState 변경에 리렌더링 (비권장)
const { formState } = useForm();
console.log(formState.errors); // 전체 formState 구독

// 필요한 것만 구독 (권장)
const { formState: { errors, isSubmitting } } = useForm();
// errors나 isSubmitting이 변경될 때만 리렌더링
```

### 조건부 렌더링 최적화

에러 메시지 컴포넌트를 분리하면 불필요한 리렌더링을 방지할 수 있습니다.

```tsx
import { useFormState } from 'react-hook-form';

// 에러 상태만 구독하는 독립적인 컴포넌트
function ErrorDisplay({ name, control }: { name: string; control: any }) {
  const { errors } = useFormState({ control, name });
  const error = errors[name];

  if (!error) return null;
  return <span className="text-red-500">{error.message as string}</span>;
}

function OptimizedForm() {
  const { register, control, handleSubmit } = useForm();

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input {...register('email', { required: '필수' })} />
      <ErrorDisplay name="email" control={control} />

      <input {...register('password', { required: '필수' })} />
      <ErrorDisplay name="password" control={control} />

      <button type="submit">제출</button>
    </form>
  );
}
```

### DevTools 사용법

React Hook Form DevTools를 사용하면 폼 상태를 실시간으로 디버깅할 수 있습니다.

```bash
npm install -D @hookform/devtools
```

```tsx
import { useForm } from 'react-hook-form';
import { DevTool } from '@hookform/devtools';

function FormWithDevTools() {
  const { register, control, handleSubmit } = useForm();

  return (
    <>
      <form onSubmit={handleSubmit((data) => console.log(data))}>
        <input {...register('firstName')} />
        <input {...register('lastName')} />
        <button type="submit">제출</button>
      </form>

      {/* 개발 환경에서만 렌더링 */}
      {process.env.NODE_ENV === 'development' && (
        <DevTool control={control} />
      )}
    </>
  );
}
```

---

## 실전 예제

### 회원가입 폼

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const signupSchema = z
  .object({
    email: z.string().email('올바른 이메일을 입력하세요'),
    password: z
      .string()
      .min(8, '비밀번호는 8자 이상이어야 합니다')
      .regex(/[A-Z]/, '대문자를 포함해야 합니다')
      .regex(/[0-9]/, '숫자를 포함해야 합니다'),
    confirmPassword: z.string(),
    name: z.string().min(2, '이름은 2자 이상이어야 합니다'),
    phone: z
      .string()
      .regex(/^01[0-9]-\d{3,4}-\d{4}$/, '올바른 전화번호 형식이 아닙니다'),
    birthYear: z.coerce
      .number()
      .min(1900, '올바른 연도를 입력하세요')
      .max(new Date().getFullYear(), '올바른 연도를 입력하세요'),
    gender: z.enum(['male', 'female', 'other'], {
      errorMap: () => ({ message: '성별을 선택하세요' }),
    }),
    terms: z.literal(true, {
      errorMap: () => ({ message: '이용약관에 동의해주세요' }),
    }),
    marketing: z.boolean().optional(),
  })
  .refine((data) => data.password === data.confirmPassword, {
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
      marketing: false,
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

        // 서버 에러를 폼 에러로 설정
        if (errorData.field) {
          setError(errorData.field, {
            type: 'server',
            message: errorData.message,
          });
        }
        return;
      }

      alert('회원가입이 완료되었습니다!');
    } catch (error) {
      setError('root', {
        type: 'server',
        message: '서버 오류가 발생했습니다. 다시 시도해주세요.',
      });
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="max-w-md mx-auto">
      {errors.root && (
        <div className="bg-red-100 p-3 rounded mb-4">
          {errors.root.message}
        </div>
      )}

      <div className="mb-4">
        <label className="block mb-1">이메일</label>
        <input
          type="email"
          {...register('email')}
          className="w-full border p-2 rounded"
        />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">비밀번호</label>
        <input
          type="password"
          {...register('password')}
          className="w-full border p-2 rounded"
        />
        {errors.password && (
          <p className="text-red-500 text-sm">{errors.password.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">비밀번호 확인</label>
        <input
          type="password"
          {...register('confirmPassword')}
          className="w-full border p-2 rounded"
        />
        {errors.confirmPassword && (
          <p className="text-red-500 text-sm">
            {errors.confirmPassword.message}
          </p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">이름</label>
        <input
          {...register('name')}
          className="w-full border p-2 rounded"
        />
        {errors.name && (
          <p className="text-red-500 text-sm">{errors.name.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">전화번호</label>
        <input
          {...register('phone')}
          placeholder="010-1234-5678"
          className="w-full border p-2 rounded"
        />
        {errors.phone && (
          <p className="text-red-500 text-sm">{errors.phone.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">출생연도</label>
        <input
          type="number"
          {...register('birthYear')}
          className="w-full border p-2 rounded"
        />
        {errors.birthYear && (
          <p className="text-red-500 text-sm">{errors.birthYear.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">성별</label>
        <select {...register('gender')} className="w-full border p-2 rounded">
          <option value="">선택하세요</option>
          <option value="male">남성</option>
          <option value="female">여성</option>
          <option value="other">기타</option>
        </select>
        {errors.gender && (
          <p className="text-red-500 text-sm">{errors.gender.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="flex items-center gap-2">
          <input type="checkbox" {...register('terms')} />
          <span>이용약관에 동의합니다 (필수)</span>
        </label>
        {errors.terms && (
          <p className="text-red-500 text-sm">{errors.terms.message}</p>
        )}
      </div>

      <div className="mb-6">
        <label className="flex items-center gap-2">
          <input type="checkbox" {...register('marketing')} />
          <span>마케팅 정보 수신에 동의합니다 (선택)</span>
        </label>
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white p-3 rounded disabled:opacity-50"
      >
        {isSubmitting ? '처리 중...' : '회원가입'}
      </button>
    </form>
  );
}
```

### 다단계 폼 (Multi-step Form)

```tsx
import { useState } from 'react';
import { useForm, FormProvider, useFormContext } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 각 단계별 스키마
const step1Schema = z.object({
  email: z.string().email('올바른 이메일을 입력하세요'),
  password: z.string().min(8, '8자 이상 입력하세요'),
});

const step2Schema = z.object({
  name: z.string().min(2, '2자 이상 입력하세요'),
  phone: z.string().min(10, '올바른 전화번호를 입력하세요'),
});

const step3Schema = z.object({
  address: z.string().min(5, '주소를 입력하세요'),
  zipCode: z.string().regex(/^\d{5}$/, '5자리 우편번호를 입력하세요'),
});

// 전체 스키마
const fullSchema = step1Schema.merge(step2Schema).merge(step3Schema);
type FormData = z.infer<typeof fullSchema>;

const schemas = [step1Schema, step2Schema, step3Schema];

function MultiStepForm() {
  const [step, setStep] = useState(0);

  const methods = useForm<FormData>({
    resolver: zodResolver(fullSchema),
    mode: 'onChange',
  });

  const onSubmit = async (data: FormData) => {
    console.log('최종 데이터:', data);
    alert('폼 제출 완료!');
  };

  const nextStep = async () => {
    const currentSchema = schemas[step];
    const fields = Object.keys(currentSchema.shape) as (keyof FormData)[];

    // 현재 단계 필드만 검증
    const isValid = await methods.trigger(fields);

    if (isValid) {
      setStep((prev) => prev + 1);
    }
  };

  const prevStep = () => {
    setStep((prev) => prev - 1);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)} className="max-w-md mx-auto">
        {/* 진행 표시 */}
        <div className="flex justify-between mb-8">
          {['계정', '개인정보', '주소'].map((label, index) => (
            <div
              key={label}
              className={`flex-1 text-center ${
                index <= step ? 'text-blue-500' : 'text-gray-400'
              }`}
            >
              <div
                className={`w-8 h-8 mx-auto rounded-full flex items-center justify-center ${
                  index <= step ? 'bg-blue-500 text-white' : 'bg-gray-200'
                }`}
              >
                {index + 1}
              </div>
              <span className="text-sm">{label}</span>
            </div>
          ))}
        </div>

        {/* 단계별 컨텐츠 */}
        {step === 0 && <Step1 />}
        {step === 1 && <Step2 />}
        {step === 2 && <Step3 />}

        {/* 네비게이션 버튼 */}
        <div className="flex gap-4 mt-6">
          {step > 0 && (
            <button
              type="button"
              onClick={prevStep}
              className="flex-1 border p-2 rounded"
            >
              이전
            </button>
          )}

          {step < 2 ? (
            <button
              type="button"
              onClick={nextStep}
              className="flex-1 bg-blue-500 text-white p-2 rounded"
            >
              다음
            </button>
          ) : (
            <button
              type="submit"
              className="flex-1 bg-green-500 text-white p-2 rounded"
            >
              완료
            </button>
          )}
        </div>
      </form>
    </FormProvider>
  );
}

function Step1() {
  const { register, formState: { errors } } = useFormContext<FormData>();

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">계정 정보</h2>
      <div>
        <label className="block mb-1">이메일</label>
        <input
          type="email"
          {...register('email')}
          className="w-full border p-2 rounded"
        />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>
      <div>
        <label className="block mb-1">비밀번호</label>
        <input
          type="password"
          {...register('password')}
          className="w-full border p-2 rounded"
        />
        {errors.password && (
          <p className="text-red-500 text-sm">{errors.password.message}</p>
        )}
      </div>
    </div>
  );
}

function Step2() {
  const { register, formState: { errors } } = useFormContext<FormData>();

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">개인 정보</h2>
      <div>
        <label className="block mb-1">이름</label>
        <input
          {...register('name')}
          className="w-full border p-2 rounded"
        />
        {errors.name && (
          <p className="text-red-500 text-sm">{errors.name.message}</p>
        )}
      </div>
      <div>
        <label className="block mb-1">전화번호</label>
        <input
          {...register('phone')}
          className="w-full border p-2 rounded"
        />
        {errors.phone && (
          <p className="text-red-500 text-sm">{errors.phone.message}</p>
        )}
      </div>
    </div>
  );
}

function Step3() {
  const { register, formState: { errors } } = useFormContext<FormData>();

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">주소 정보</h2>
      <div>
        <label className="block mb-1">주소</label>
        <input
          {...register('address')}
          className="w-full border p-2 rounded"
        />
        {errors.address && (
          <p className="text-red-500 text-sm">{errors.address.message}</p>
        )}
      </div>
      <div>
        <label className="block mb-1">우편번호</label>
        <input
          {...register('zipCode')}
          className="w-full border p-2 rounded"
        />
        {errors.zipCode && (
          <p className="text-red-500 text-sm">{errors.zipCode.message}</p>
        )}
      </div>
    </div>
  );
}
```

### 파일 업로드 폼

```tsx
import { useForm, Controller } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ACCEPTED_IMAGE_TYPES = ['image/jpeg', 'image/png', 'image/webp'];

const schema = z.object({
  title: z.string().min(1, '제목을 입력하세요'),
  description: z.string().optional(),
  image: z
    .custom<FileList>()
    .refine((files) => files?.length === 1, '이미지를 선택하세요')
    .refine(
      (files) => files?.[0]?.size <= MAX_FILE_SIZE,
      '파일 크기는 5MB 이하여야 합니다'
    )
    .refine(
      (files) => ACCEPTED_IMAGE_TYPES.includes(files?.[0]?.type),
      'JPG, PNG, WebP 형식만 지원합니다'
    ),
  documents: z
    .custom<FileList>()
    .refine((files) => files?.length <= 5, '최대 5개 파일까지 업로드 가능합니다')
    .optional(),
});

type FormData = z.infer<typeof schema>;

function FileUploadForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const imageFile = watch('image');
  const previewUrl = imageFile?.[0]
    ? URL.createObjectURL(imageFile[0])
    : null;

  const onSubmit = async (data: FormData) => {
    const formData = new FormData();
    formData.append('title', data.title);
    if (data.description) {
      formData.append('description', data.description);
    }
    formData.append('image', data.image[0]);

    if (data.documents) {
      Array.from(data.documents).forEach((file) => {
        formData.append('documents', file);
      });
    }

    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData,
    });

    if (response.ok) {
      alert('업로드 완료!');
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="max-w-md mx-auto">
      <div className="mb-4">
        <label className="block mb-1">제목</label>
        <input
          {...register('title')}
          className="w-full border p-2 rounded"
        />
        {errors.title && (
          <p className="text-red-500 text-sm">{errors.title.message}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">설명</label>
        <textarea
          {...register('description')}
          className="w-full border p-2 rounded"
        />
      </div>

      <div className="mb-4">
        <label className="block mb-1">대표 이미지</label>
        <input
          type="file"
          accept="image/jpeg,image/png,image/webp"
          {...register('image')}
          className="w-full"
        />
        {errors.image && (
          <p className="text-red-500 text-sm">{errors.image.message as string}</p>
        )}
        {previewUrl && (
          <img
            src={previewUrl}
            alt="미리보기"
            className="mt-2 max-w-full h-40 object-cover rounded"
          />
        )}
      </div>

      <div className="mb-4">
        <label className="block mb-1">첨부 문서 (최대 5개)</label>
        <input
          type="file"
          multiple
          {...register('documents')}
          className="w-full"
        />
        {errors.documents && (
          <p className="text-red-500 text-sm">
            {errors.documents.message as string}
          </p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white p-2 rounded disabled:opacity-50"
      >
        {isSubmitting ? '업로드 중...' : '업로드'}
      </button>
    </form>
  );
}
```

---

## 자주 묻는 질문 (FAQ)

### Q1. Formik과 React Hook Form 중 무엇을 선택해야 하나요?

**A:** 대부분의 경우 **React Hook Form**을 권장합니다.

| 상황 | 추천 |
|------|------|
| 새 프로젝트 | React Hook Form |
| 성능이 중요한 경우 | React Hook Form |
| 기존 Formik 프로젝트 | 그대로 유지 |
| 복잡한 폼 로직 | 둘 다 가능 |

React Hook Form이 더 작은 번들 크기와 뛰어난 성능을 제공합니다.

---

### Q2. `watch`와 `getValues`의 차이는 무엇인가요?

**A:** 리렌더링 여부가 다릅니다.

```tsx
// watch - 값이 변경되면 컴포넌트 리렌더링
const name = watch('name');

// getValues - 리렌더링 없이 값만 가져옴
const handleClick = () => {
  const name = getValues('name');
  console.log(name);
};
```

**사용 가이드:**
- 실시간 UI 반영이 필요하면 `watch`
- 이벤트 핸들러에서 값만 필요하면 `getValues`

---

### Q3. 서버 사이드 에러를 어떻게 처리하나요?

**A:** `setError`를 사용합니다.

```tsx
const { setError, clearErrors } = useForm();

const onSubmit = async (data) => {
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      const error = await response.json();

      // 특정 필드 에러
      setError('email', {
        type: 'server',
        message: error.message,
      });

      // 전체 폼 에러
      setError('root.serverError', {
        type: 'server',
        message: '서버 오류가 발생했습니다',
      });
    }
  } catch (e) {
    setError('root.serverError', {
      type: 'network',
      message: '네트워크 오류가 발생했습니다',
    });
  }
};
```

---

### Q4. 중첩된 객체 필드는 어떻게 다루나요?

**A:** 점 표기법을 사용합니다.

```tsx
interface FormData {
  user: {
    profile: {
      firstName: string;
      lastName: string;
    };
    settings: {
      notifications: boolean;
    };
  };
}

function NestedForm() {
  const { register } = useForm<FormData>();

  return (
    <form>
      <input {...register('user.profile.firstName')} />
      <input {...register('user.profile.lastName')} />
      <input
        type="checkbox"
        {...register('user.settings.notifications')}
      />
    </form>
  );
}
```

---

### Q5. 폼 초기값을 비동기로 설정하려면 어떻게 하나요?

**A:** `reset` 또는 `values` 옵션을 사용합니다.

```tsx
// 방법 1: useEffect + reset
function AsyncDefaultValues() {
  const { register, reset } = useForm();

  useEffect(() => {
    async function fetchData() {
      const response = await fetch('/api/user');
      const data = await response.json();
      reset(data); // 폼 초기화
    }
    fetchData();
  }, [reset]);

  return <form>...</form>;
}

// 방법 2: values prop (React Hook Form v7.12+)
function AsyncDefaultValues() {
  const [userData, setUserData] = useState(null);
  const { register } = useForm({
    values: userData, // 값이 변경되면 자동으로 폼 업데이트
  });

  useEffect(() => {
    fetch('/api/user')
      .then((res) => res.json())
      .then(setUserData);
  }, []);

  return <form>...</form>;
}
```

---

### Q6. 배열 필드의 유효성 검사는 어떻게 하나요?

**A:** Zod와 `useFieldArray`를 함께 사용합니다.

```tsx
const schema = z.object({
  items: z
    .array(
      z.object({
        name: z.string().min(1, '이름 필수'),
        quantity: z.number().min(1, '1개 이상'),
      })
    )
    .min(1, '최소 1개 항목 필요')
    .max(10, '최대 10개까지'),
});

function ArrayValidationForm() {
  const { control, register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      items: [{ name: '', quantity: 1 }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'items',
  });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {/* 배열 전체 에러 */}
      {errors.items?.root && (
        <p className="text-red-500">{errors.items.root.message}</p>
      )}

      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`items.${index}.name`)} />
          {errors.items?.[index]?.name && (
            <span>{errors.items[index].name.message}</span>
          )}

          <input
            type="number"
            {...register(`items.${index}.quantity`, {
              valueAsNumber: true,
            })}
          />
          {errors.items?.[index]?.quantity && (
            <span>{errors.items[index].quantity.message}</span>
          )}

          <button type="button" onClick={() => remove(index)}>
            삭제
          </button>
        </div>
      ))}

      <button type="button" onClick={() => append({ name: '', quantity: 1 })}>
        추가
      </button>
      <button type="submit">제출</button>
    </form>
  );
}
```

---

### Q7. disabled 상태인 필드 값은 어떻게 처리되나요?

**A:** 기본적으로 제출 데이터에서 제외됩니다.

```tsx
// disabled 필드는 제출 데이터에 포함되지 않음
<input {...register('name')} disabled />

// 값을 유지하면서 편집 불가 상태로 만들려면 readOnly 사용
<input {...register('name')} readOnly />

// 또는 shouldUnregister: false 설정
const { register } = useForm({
  shouldUnregister: false,
});
```

---

### Q8. 실시간 유효성 검사와 제출 시 검사를 함께 사용하려면?

**A:** `mode: 'onTouched'`가 좋은 선택입니다.

```tsx
const { register, handleSubmit } = useForm({
  mode: 'onTouched', // 첫 blur 후부터 onChange로 검사
  reValidateMode: 'onChange', // 에러 발생 후 재검사 시점
});
```

| mode | 동작 |
|------|------|
| `onSubmit` | 제출 시에만 검사 |
| `onBlur` | 포커스 해제 시 검사 |
| `onChange` | 매 입력마다 검사 |
| `onTouched` | 첫 blur 후 onChange로 전환 (권장) |
| `all` | blur + change 모두 검사 |

---

## 마무리

React Hook Form은 React에서 폼을 다루는 가장 효율적인 방법 중 하나입니다.

### 핵심 포인트 정리

| 개념 | 설명 |
|------|------|
| `useForm` | 폼 상태 관리의 핵심 훅 |
| `register` | 입력 필드 등록 |
| `handleSubmit` | 폼 제출 + 유효성 검사 |
| `formState` | 에러, 제출 상태 등 |
| `useFieldArray` | 동적 필드 관리 |
| `Controller` | 외부 UI 컴포넌트 통합 |
| Zod | 스키마 기반 유효성 검사 |

### 권장 사항

1. **새 프로젝트**에서는 React Hook Form + Zod 조합 사용
2. **mode**는 `onTouched`로 설정하여 UX 개선
3. **formState**는 필요한 것만 구조 분해하여 성능 최적화
4. 복잡한 폼은 **FormProvider**로 컴포넌트 분리
5. **DevTools**를 활용하여 디버깅

React Hook Form을 통해 복잡한 폼 로직을 간결하게 관리하고, 뛰어난 사용자 경험을 제공하세요.

---

## 참고 자료

- [React Hook Form 공식 문서](https://react-hook-form.com/)
- [Zod 공식 문서](https://zod.dev/)
- [@hookform/resolvers](https://github.com/react-hook-form/resolvers)
- [React Hook Form DevTools](https://github.com/react-hook-form/devtools)
