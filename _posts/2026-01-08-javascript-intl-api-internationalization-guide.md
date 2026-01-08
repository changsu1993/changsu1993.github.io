---
title: "JavaScript Intl API 완벽 가이드 - 다국어 웹 앱을 위한 날짜/숫자/통화 포맷팅"
description: "JavaScript Intl API로 날짜, 숫자, 통화, 상대 시간을 로케일별로 포맷팅하는 방법을 배웁니다. DateTimeFormat, NumberFormat, RelativeTimeFormat 등 8가지 Intl 생성자의 실전 활용법과 React 통합 예제를 제공합니다."
date: 2026-01-08 09:00:00 +0900
categories: [JavaScript, Web API]
tags: [javascript, intl-api, DateTimeFormat, NumberFormat, RelativeTimeFormat, i18n, localization, 다국어, 날짜포맷, 통화포맷]
---

글로벌 서비스를 개발할 때 가장 먼저 부딪히는 문제 중 하나가 날짜, 숫자, 통화의 지역별 표기 차이입니다. 미국에서는 `12/25/2025`로 표기하는 날짜가 한국에서는 `2025. 12. 25.`로, 독일에서는 `25.12.2025`로 표기됩니다.

이 글에서는 JavaScript의 내장 국제화 API인 **Intl 객체**의 모든 기능을 상세히 다룹니다. 외부 라이브러리 없이 브라우저 내장 기능만으로 강력한 다국어 지원을 구현하는 방법을 배워보세요.

> **사전 지식**: [JavaScript 날짜와 시간 완벽 가이드](/posts/javascript-date-time/)에서 Date 객체 기본 사용법을, [JavaScript Number와 Math 완벽 가이드](/posts/javascript-number-math/)에서 숫자 처리 기초를 먼저 학습하면 이 글을 더 쉽게 이해할 수 있습니다.

## Intl API 개요

**Intl**(Internationalization)은 JavaScript의 국제화 API를 제공하는 네임스페이스 객체입니다. 다국어 웹 애플리케이션을 개발할 때 날짜, 시간, 숫자, 통화 등을 각 로케일(locale)에 맞게 포맷팅하는 기능을 제공합니다.

### 왜 Intl API가 필요한가?

전 세계 사용자를 대상으로 하는 웹 애플리케이션에서는 다음과 같은 문제를 해결해야 합니다:

```javascript
// 날짜 표기: 국가마다 다름
// 미국: 12/25/2025 (MM/DD/YYYY)
// 한국: 2025. 12. 25. (YYYY. MM. DD.)
// 독일: 25.12.2025 (DD.MM.YYYY)

// 숫자 표기: 천 단위 구분자가 다름
// 미국: 1,234,567.89
// 독일: 1.234.567,89
// 프랑스: 1 234 567,89

// 통화 표기: 기호와 위치가 다름
// 미국: $1,234.56
// 한국: ₩1,234
// 유럽: 1.234,56 €
```

Intl API 없이 이러한 포맷팅을 직접 구현하면 코드가 복잡해지고 오류 발생 가능성이 높아집니다.

### Intl API의 주요 생성자

| 생성자 | 용도 | 사용 예시 |
|--------|------|----------|
| `Intl.DateTimeFormat` | 날짜/시간 포맷팅 | "2025년 1월 8일 수요일" |
| `Intl.NumberFormat` | 숫자/통화/단위 포맷팅 | "₩1,234,567" |
| `Intl.RelativeTimeFormat` | 상대 시간 표시 | "3일 전", "2시간 후" |
| `Intl.PluralRules` | 복수형 규칙 선택 | "1 item" vs "2 items" |
| `Intl.Collator` | 문자열 정렬 | 로케일 기반 정렬 |
| `Intl.ListFormat` | 리스트 포맷팅 | "사과, 바나나, 그리고 오렌지" |
| `Intl.Segmenter` | 텍스트 분할 | 단어, 문장, 문자 단위 |
| `Intl.DisplayNames` | 언어/지역명 표시 | "Korean", "대한민국" |

### 로케일(Locale) 이해하기

로케일은 언어와 지역을 식별하는 문자열입니다. BCP 47 형식을 따릅니다:

```javascript
// 기본 형식: 언어[-스크립트][-지역]
"ko"        // 한국어
"ko-KR"     // 한국어 (대한민국)
"en-US"     // 영어 (미국)
"en-GB"     // 영어 (영국)
"zh-Hans"   // 중국어 간체
"zh-Hant"   // 중국어 번체

// 유니코드 확장 포함
"ko-KR-u-ca-buddhist"  // 한국어, 불교 달력 사용
"de-DE-u-co-phonebk"   // 독일어, 전화번호부 정렬
"th-TH-u-nu-thai"      // 태국어, 태국 숫자 체계
```

---

## Intl.DateTimeFormat

`Intl.DateTimeFormat`은 날짜와 시간을 로케일에 맞게 포맷팅합니다.

### 기본 사용법

```javascript
const date = new Date('2025-12-25T15:30:00');

// 기본 포맷 (브라우저 로케일)
console.log(new Intl.DateTimeFormat().format(date));

// 로케일 지정
console.log(new Intl.DateTimeFormat('ko-KR').format(date));
// "2025. 12. 25."

console.log(new Intl.DateTimeFormat('en-US').format(date));
// "12/25/2025"

console.log(new Intl.DateTimeFormat('de-DE').format(date));
// "25.12.2025"

console.log(new Intl.DateTimeFormat('ja-JP').format(date));
// "2025/12/25"
```

### dateStyle과 timeStyle 옵션

빠르게 날짜/시간 스타일을 지정할 때 사용합니다:

```javascript
const date = new Date('2025-12-25T15:30:45');

// dateStyle: "full", "long", "medium", "short"
console.log(new Intl.DateTimeFormat('ko-KR', {
  dateStyle: 'full'
}).format(date));
// "2025년 12월 25일 목요일"

console.log(new Intl.DateTimeFormat('ko-KR', {
  dateStyle: 'long'
}).format(date));
// "2025년 12월 25일"

console.log(new Intl.DateTimeFormat('ko-KR', {
  dateStyle: 'medium'
}).format(date));
// "2025. 12. 25."

console.log(new Intl.DateTimeFormat('ko-KR', {
  dateStyle: 'short'
}).format(date));
// "25. 12. 25."

// timeStyle: "full", "long", "medium", "short"
console.log(new Intl.DateTimeFormat('ko-KR', {
  timeStyle: 'full'
}).format(date));
// "오후 3시 30분 45초 대한민국 표준시"

console.log(new Intl.DateTimeFormat('ko-KR', {
  timeStyle: 'short'
}).format(date));
// "오후 3:30"

// 날짜와 시간 함께
console.log(new Intl.DateTimeFormat('ko-KR', {
  dateStyle: 'full',
  timeStyle: 'long'
}).format(date));
// "2025년 12월 25일 목요일 오후 3시 30분 45초 GMT+9"
```

### 개별 컴포넌트 옵션

더 세밀한 제어가 필요할 때 개별 옵션을 사용합니다:

```javascript
const date = new Date('2025-12-25T15:30:45.123');

// 날짜 컴포넌트
const dateOptions = {
  weekday: 'long',      // "long", "short", "narrow"
  year: 'numeric',      // "numeric", "2-digit"
  month: 'long',        // "numeric", "2-digit", "long", "short", "narrow"
  day: 'numeric',       // "numeric", "2-digit"
};

console.log(new Intl.DateTimeFormat('ko-KR', dateOptions).format(date));
// "2025년 12월 25일 목요일"

console.log(new Intl.DateTimeFormat('en-US', dateOptions).format(date));
// "Thursday, December 25, 2025"

// 시간 컴포넌트
const timeOptions = {
  hour: 'numeric',              // "numeric", "2-digit"
  minute: '2-digit',            // "numeric", "2-digit"
  second: '2-digit',            // "numeric", "2-digit"
  fractionalSecondDigits: 3,    // 1, 2, 3 (밀리초 자릿수)
  hour12: false,                // 24시간 형식
};

console.log(new Intl.DateTimeFormat('ko-KR', timeOptions).format(date));
// "15:30:45.123"

// 12시간 형식
console.log(new Intl.DateTimeFormat('ko-KR', {
  hour: 'numeric',
  minute: '2-digit',
  hour12: true,
  dayPeriod: 'long'   // "오전", "오후" 표시 방식
}).format(date));
// "오후 3:30"
```

### 타임존 처리

```javascript
const date = new Date('2025-12-25T15:30:00Z'); // UTC 시간

// 다양한 타임존으로 표시
const options = {
  dateStyle: 'medium',
  timeStyle: 'long',
  timeZone: 'Asia/Seoul'
};

console.log(new Intl.DateTimeFormat('ko-KR', options).format(date));
// "2025. 12. 26. 오전 12:30:00 GMT+9"

console.log(new Intl.DateTimeFormat('en-US', {
  ...options,
  timeZone: 'America/New_York'
}).format(date));
// "Dec 25, 2025, 10:30:00 AM GMT-5"

console.log(new Intl.DateTimeFormat('en-GB', {
  ...options,
  timeZone: 'Europe/London'
}).format(date));
// "25 Dec 2025, 15:30:00 GMT"

// 타임존 이름 표시
console.log(new Intl.DateTimeFormat('ko-KR', {
  timeZone: 'America/Los_Angeles',
  timeZoneName: 'long',
  hour: 'numeric',
  minute: 'numeric'
}).format(date));
// "오전 7:30 미 태평양 표준시"
```

### 다양한 달력 시스템

```javascript
const date = new Date('2025-12-25');

// 일본 연호
console.log(new Intl.DateTimeFormat('ja-JP-u-ca-japanese', {
  dateStyle: 'full'
}).format(date));
// "令和7年12月25日木曜日"

// 중국 음력
console.log(new Intl.DateTimeFormat('zh-CN-u-ca-chinese', {
  dateStyle: 'full'
}).format(date));
// "2025年冬月初六星期四"

// 불교력 (태국)
console.log(new Intl.DateTimeFormat('th-TH-u-ca-buddhist', {
  dateStyle: 'full'
}).format(date));
// "วันพฤหัสบดีที่ 25 ธันวาคม พ.ศ. 2568"
```

---

## Intl.NumberFormat

`Intl.NumberFormat`은 숫자, 통화, 백분율, 단위를 로케일에 맞게 포맷팅합니다.

### 기본 숫자 포맷팅

```javascript
const number = 1234567.89;

// 로케일별 숫자 표기
console.log(new Intl.NumberFormat('ko-KR').format(number));
// "1,234,567.89"

console.log(new Intl.NumberFormat('de-DE').format(number));
// "1.234.567,89"

console.log(new Intl.NumberFormat('fr-FR').format(number));
// "1 234 567,89"

console.log(new Intl.NumberFormat('en-IN').format(number));
// "12,34,567.89" (인도식 표기)
```

### 통화 포맷팅

```javascript
const price = 1234567.89;

// style: "currency"
console.log(new Intl.NumberFormat('ko-KR', {
  style: 'currency',
  currency: 'KRW'
}).format(price));
// "₩1,234,568"

console.log(new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(price));
// "$1,234,567.89"

console.log(new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(price));
// "￥1,234,568"

console.log(new Intl.NumberFormat('de-DE', {
  style: 'currency',
  currency: 'EUR'
}).format(price));
// "1.234.567,89 €"

// currencyDisplay 옵션
console.log(new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'name'    // "symbol", "narrowSymbol", "code", "name"
}).format(price));
// "1,234,567.89 US dollars"

console.log(new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'code'
}).format(price));
// "USD 1,234,567.89"
```

### 백분율 포맷팅

```javascript
const ratio = 0.8567;

console.log(new Intl.NumberFormat('ko-KR', {
  style: 'percent'
}).format(ratio));
// "86%"

console.log(new Intl.NumberFormat('ko-KR', {
  style: 'percent',
  minimumFractionDigits: 1,
  maximumFractionDigits: 2
}).format(ratio));
// "85.67%"
```

### 단위 포맷팅

```javascript
// style: "unit"
console.log(new Intl.NumberFormat('ko-KR', {
  style: 'unit',
  unit: 'kilometer'
}).format(100));
// "100km"

console.log(new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'mile',
  unitDisplay: 'long'   // "short", "narrow", "long"
}).format(100));
// "100 miles"

// 복합 단위
console.log(new Intl.NumberFormat('ko-KR', {
  style: 'unit',
  unit: 'kilometer-per-hour'
}).format(120));
// "120km/h"

console.log(new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'liter-per-100-kilometer',
  unitDisplay: 'long'
}).format(8.5));
// "8.5 liters per 100 kilometers"

// 데이터 크기
console.log(new Intl.NumberFormat('ko-KR', {
  style: 'unit',
  unit: 'gigabyte'
}).format(256));
// "256GB"

// 온도
console.log(new Intl.NumberFormat('ko-KR', {
  style: 'unit',
  unit: 'celsius'
}).format(25));
// "25°C"
```

### 압축 표기 (Compact Notation)

큰 숫자를 간결하게 표시할 때 유용합니다:

```javascript
// notation: "compact"
console.log(new Intl.NumberFormat('ko-KR', {
  notation: 'compact'
}).format(1234));
// "1.2천"

console.log(new Intl.NumberFormat('ko-KR', {
  notation: 'compact'
}).format(1234567));
// "123만"

console.log(new Intl.NumberFormat('ko-KR', {
  notation: 'compact',
  compactDisplay: 'long'   // "short", "long"
}).format(1234567));
// "123만"

console.log(new Intl.NumberFormat('en-US', {
  notation: 'compact',
  compactDisplay: 'short'
}).format(1234567));
// "1.2M"

console.log(new Intl.NumberFormat('en-US', {
  notation: 'compact',
  compactDisplay: 'long'
}).format(1234567));
// "1.2 million"

// 과학적 표기
console.log(new Intl.NumberFormat('en-US', {
  notation: 'scientific'
}).format(1234567));
// "1.235E6"

// 공학적 표기
console.log(new Intl.NumberFormat('en-US', {
  notation: 'engineering'
}).format(1234567));
// "1.235E6"
```

### 소수점 자릿수 제어

```javascript
const number = 1234.5678;

// 최소/최대 소수점 자릿수
console.log(new Intl.NumberFormat('ko-KR', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 4
}).format(number));
// "1,234.5678"

console.log(new Intl.NumberFormat('ko-KR', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
}).format(number));
// "1,234.57" (반올림)

// 유효 숫자 자릿수
console.log(new Intl.NumberFormat('ko-KR', {
  maximumSignificantDigits: 3
}).format(number));
// "1,230"

console.log(new Intl.NumberFormat('ko-KR', {
  minimumSignificantDigits: 6,
  maximumSignificantDigits: 6
}).format(1234));
// "1,234.00"
```

### 부호 표시

```javascript
// signDisplay 옵션
const formatter = (signDisplay) => new Intl.NumberFormat('ko-KR', {
  signDisplay,
  style: 'currency',
  currency: 'KRW'
});

console.log(formatter('auto').format(1000));     // "₩1,000"
console.log(formatter('auto').format(-1000));    // "-₩1,000"
console.log(formatter('auto').format(0));        // "₩0"

console.log(formatter('always').format(1000));   // "+₩1,000"
console.log(formatter('always').format(-1000));  // "-₩1,000"

console.log(formatter('exceptZero').format(1000));  // "+₩1,000"
console.log(formatter('exceptZero').format(0));     // "₩0"

console.log(formatter('negative').format(1000));    // "₩1,000"
console.log(formatter('negative').format(-1000));   // "-₩1,000"

console.log(formatter('never').format(-1000));      // "₩1,000"
```

---

## Intl.RelativeTimeFormat

`Intl.RelativeTimeFormat`은 "3일 전", "2시간 후"와 같은 상대 시간을 포맷팅합니다.

### 기본 사용법

```javascript
const rtf = new Intl.RelativeTimeFormat('ko-KR');

// 양수: 미래, 음수: 과거
console.log(rtf.format(-1, 'day'));      // "1일 전"
console.log(rtf.format(1, 'day'));       // "1일 후"
console.log(rtf.format(-3, 'hour'));     // "3시간 전"
console.log(rtf.format(2, 'week'));      // "2주 후"
console.log(rtf.format(-1, 'month'));    // "1개월 전"
console.log(rtf.format(1, 'year'));      // "1년 후"
```

### 지원 단위

```javascript
const rtf = new Intl.RelativeTimeFormat('ko-KR');

// 지원되는 모든 단위
console.log(rtf.format(-30, 'second'));   // "30초 전"
console.log(rtf.format(-5, 'minute'));    // "5분 전"
console.log(rtf.format(-2, 'hour'));      // "2시간 전"
console.log(rtf.format(-1, 'day'));       // "1일 전"
console.log(rtf.format(-1, 'week'));      // "1주 전"
console.log(rtf.format(-1, 'month'));     // "1개월 전"
console.log(rtf.format(-1, 'quarter'));   // "1분기 전"
console.log(rtf.format(-1, 'year'));      // "1년 전"
```

### style 옵션

```javascript
// style: "long" (기본값)
const rtfLong = new Intl.RelativeTimeFormat('ko-KR', { style: 'long' });
console.log(rtfLong.format(-3, 'day'));
// "3일 전"

// style: "short"
const rtfShort = new Intl.RelativeTimeFormat('ko-KR', { style: 'short' });
console.log(rtfShort.format(-3, 'day'));
// "3일 전"

// style: "narrow"
const rtfNarrow = new Intl.RelativeTimeFormat('ko-KR', { style: 'narrow' });
console.log(rtfNarrow.format(-3, 'day'));
// "3일 전"

// 영어에서는 차이가 더 명확함
const rtfEnLong = new Intl.RelativeTimeFormat('en-US', { style: 'long' });
console.log(rtfEnLong.format(-3, 'month'));
// "3 months ago"

const rtfEnShort = new Intl.RelativeTimeFormat('en-US', { style: 'short' });
console.log(rtfEnShort.format(-3, 'month'));
// "3 mo. ago"

const rtfEnNarrow = new Intl.RelativeTimeFormat('en-US', { style: 'narrow' });
console.log(rtfEnNarrow.format(-3, 'month'));
// "3 mo. ago"
```

### numeric 옵션

```javascript
// numeric: "always" (기본값) - 항상 숫자 사용
const rtfAlways = new Intl.RelativeTimeFormat('ko-KR', { numeric: 'always' });
console.log(rtfAlways.format(-1, 'day'));   // "1일 전"
console.log(rtfAlways.format(0, 'day'));    // "0일 후"
console.log(rtfAlways.format(1, 'day'));    // "1일 후"

// numeric: "auto" - 가능하면 자연어 사용
const rtfAuto = new Intl.RelativeTimeFormat('ko-KR', { numeric: 'auto' });
console.log(rtfAuto.format(-1, 'day'));     // "어제"
console.log(rtfAuto.format(0, 'day'));      // "오늘"
console.log(rtfAuto.format(1, 'day'));      // "내일"
console.log(rtfAuto.format(-2, 'day'));     // "그저께" 또는 "2일 전"

// 영어 예시
const rtfEnAuto = new Intl.RelativeTimeFormat('en-US', { numeric: 'auto' });
console.log(rtfEnAuto.format(-1, 'day'));   // "yesterday"
console.log(rtfEnAuto.format(0, 'day'));    // "today"
console.log(rtfEnAuto.format(1, 'day'));    // "tomorrow"
console.log(rtfEnAuto.format(-1, 'week'));  // "last week"
console.log(rtfEnAuto.format(1, 'month'));  // "next month"
console.log(rtfEnAuto.format(1, 'year'));   // "next year"
```

### 실용적인 상대 시간 함수

```javascript
function getRelativeTime(date, locale = 'ko-KR') {
  const now = new Date();
  const diffMs = date.getTime() - now.getTime();
  const diffSec = Math.round(diffMs / 1000);
  const diffMin = Math.round(diffSec / 60);
  const diffHour = Math.round(diffMin / 60);
  const diffDay = Math.round(diffHour / 24);
  const diffWeek = Math.round(diffDay / 7);
  const diffMonth = Math.round(diffDay / 30);
  const diffYear = Math.round(diffDay / 365);

  const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });

  if (Math.abs(diffSec) < 60) {
    return rtf.format(diffSec, 'second');
  } else if (Math.abs(diffMin) < 60) {
    return rtf.format(diffMin, 'minute');
  } else if (Math.abs(diffHour) < 24) {
    return rtf.format(diffHour, 'hour');
  } else if (Math.abs(diffDay) < 7) {
    return rtf.format(diffDay, 'day');
  } else if (Math.abs(diffWeek) < 4) {
    return rtf.format(diffWeek, 'week');
  } else if (Math.abs(diffMonth) < 12) {
    return rtf.format(diffMonth, 'month');
  } else {
    return rtf.format(diffYear, 'year');
  }
}

// 사용 예시
const pastDate = new Date(Date.now() - 1000 * 60 * 60 * 3); // 3시간 전
console.log(getRelativeTime(pastDate));
// "3시간 전"

const futureDate = new Date(Date.now() + 1000 * 60 * 60 * 24 * 2); // 2일 후
console.log(getRelativeTime(futureDate));
// "2일 후" 또는 "모레"
```

---

## Intl.PluralRules

`Intl.PluralRules`는 언어별 복수형 규칙을 처리합니다. 각 언어마다 복수형을 처리하는 방식이 다릅니다.

### 복수형 카테고리

Intl.PluralRules는 다음 카테고리 중 하나를 반환합니다:
- `zero`: 0을 위한 형태
- `one`: 1을 위한 형태 (또는 "1"처럼 취급되는 숫자)
- `two`: 2를 위한 형태
- `few`: "소수"를 위한 형태
- `many`: "다수"를 위한 형태
- `other`: 일반적인 형태 (모든 언어에 존재)

```javascript
// 영어: "one"과 "other"만 사용
const prEn = new Intl.PluralRules('en-US');
console.log(prEn.select(0));   // "other"
console.log(prEn.select(1));   // "one"
console.log(prEn.select(2));   // "other"
console.log(prEn.select(5));   // "other"

// 한국어: "other"만 사용 (복수형 구분 없음)
const prKo = new Intl.PluralRules('ko-KR');
console.log(prKo.select(0));   // "other"
console.log(prKo.select(1));   // "other"
console.log(prKo.select(2));   // "other"

// 러시아어: 복잡한 복수형 규칙
const prRu = new Intl.PluralRules('ru');
console.log(prRu.select(1));   // "one"
console.log(prRu.select(2));   // "few"
console.log(prRu.select(5));   // "many"
console.log(prRu.select(21));  // "one"
console.log(prRu.select(22));  // "few"

// 아랍어: 가장 복잡한 복수형 규칙
const prAr = new Intl.PluralRules('ar');
console.log(prAr.select(0));   // "zero"
console.log(prAr.select(1));   // "one"
console.log(prAr.select(2));   // "two"
console.log(prAr.select(3));   // "few"
console.log(prAr.select(11));  // "many"
console.log(prAr.select(100)); // "other"
```

### 서수 (Ordinal) 처리

```javascript
// type: "ordinal" - 순서를 나타내는 숫자 (1st, 2nd, 3rd...)
const prOrdinal = new Intl.PluralRules('en-US', { type: 'ordinal' });

console.log(prOrdinal.select(1));   // "one"    -> 1st
console.log(prOrdinal.select(2));   // "two"    -> 2nd
console.log(prOrdinal.select(3));   // "few"    -> 3rd
console.log(prOrdinal.select(4));   // "other"  -> 4th
console.log(prOrdinal.select(21));  // "one"    -> 21st
console.log(prOrdinal.select(22));  // "two"    -> 22nd
console.log(prOrdinal.select(23));  // "few"    -> 23rd
console.log(prOrdinal.select(24));  // "other"  -> 24th
```

### 실용적인 복수형 처리 함수

```javascript
// 영어 복수형 처리
function pluralize(count, singular, plural) {
  const pr = new Intl.PluralRules('en-US');
  const rule = pr.select(count);

  const forms = {
    one: singular,
    other: plural
  };

  return `${count} ${forms[rule]}`;
}

console.log(pluralize(1, 'item', 'items'));    // "1 item"
console.log(pluralize(5, 'item', 'items'));    // "5 items"
console.log(pluralize(0, 'file', 'files'));    // "0 files"

// 서수 접미사 붙이기
function formatOrdinal(n) {
  const pr = new Intl.PluralRules('en-US', { type: 'ordinal' });
  const suffixes = {
    one: 'st',
    two: 'nd',
    few: 'rd',
    other: 'th'
  };

  return `${n}${suffixes[pr.select(n)]}`;
}

console.log(formatOrdinal(1));    // "1st"
console.log(formatOrdinal(2));    // "2nd"
console.log(formatOrdinal(3));    // "3rd"
console.log(formatOrdinal(4));    // "4th"
console.log(formatOrdinal(11));   // "11th"
console.log(formatOrdinal(21));   // "21st"
console.log(formatOrdinal(22));   // "22nd"
console.log(formatOrdinal(23));   // "23rd"
console.log(formatOrdinal(100));  // "100th"

// 다국어 복수형 처리
function pluralizeMultiLang(count, forms, locale = 'en-US') {
  const pr = new Intl.PluralRules(locale);
  const rule = pr.select(count);

  return `${count} ${forms[rule] || forms.other}`;
}

// 러시아어 예시: "1 яблоко", "2 яблока", "5 яблок"
const russianForms = {
  one: 'яблоко',
  few: 'яблока',
  many: 'яблок',
  other: 'яблок'
};

console.log(pluralizeMultiLang(1, russianForms, 'ru'));   // "1 яблоко"
console.log(pluralizeMultiLang(2, russianForms, 'ru'));   // "2 яблока"
console.log(pluralizeMultiLang(5, russianForms, 'ru'));   // "5 яблок"
console.log(pluralizeMultiLang(21, russianForms, 'ru'));  // "21 яблоко"
```

---

## Intl.Collator

`Intl.Collator`는 문자열을 로케일에 맞게 비교하고 정렬합니다.

### 기본 사용법

```javascript
// 기본 비교
const collator = new Intl.Collator('ko-KR');

console.log(collator.compare('가', '나'));   // -1 (가 < 나)
console.log(collator.compare('나', '가'));   // 1  (나 > 가)
console.log(collator.compare('가', '가'));   // 0  (같음)

// 배열 정렬
const words = ['다람쥐', '가방', '나무', '바다', '사과'];
words.sort(collator.compare);
console.log(words);
// ["가방", "나무", "다람쥐", "바다", "사과"]
```

### 대소문자 처리

```javascript
// sensitivity 옵션
// "base": 기본 문자만 비교 (a = A = á)
// "accent": 기본 문자 + 악센트 비교 (a = A, a ≠ á)
// "case": 기본 문자 + 대소문자 비교 (a ≠ A, a = á)
// "variant": 모두 비교 (a ≠ A ≠ á)

const words = ['apple', 'Apple', 'APPLE', 'banana'];

// 대소문자 무시 정렬
const collatorBase = new Intl.Collator('en-US', { sensitivity: 'base' });
const sorted1 = [...words].sort(collatorBase.compare);
console.log(sorted1);
// ["apple", "Apple", "APPLE", "banana"]

// 대소문자 구분 정렬
const collatorVariant = new Intl.Collator('en-US', { sensitivity: 'variant' });
const sorted2 = [...words].sort(collatorVariant.compare);
console.log(sorted2);
// ["APPLE", "Apple", "apple", "banana"]

// caseFirst 옵션: 대문자/소문자 우선
const collatorUpper = new Intl.Collator('en-US', { caseFirst: 'upper' });
const collatorLower = new Intl.Collator('en-US', { caseFirst: 'lower' });

console.log(['a', 'A', 'b', 'B'].sort(collatorUpper.compare));
// ["A", "a", "B", "b"]

console.log(['a', 'A', 'b', 'B'].sort(collatorLower.compare));
// ["a", "A", "b", "B"]
```

### 숫자 정렬

```javascript
// numeric 옵션: 숫자를 숫자값으로 비교
const files = ['file1', 'file10', 'file2', 'file20', 'file3'];

// 기본 정렬 (문자열 순)
const collatorDefault = new Intl.Collator('en-US');
console.log([...files].sort(collatorDefault.compare));
// ["file1", "file10", "file2", "file20", "file3"]

// 숫자 인식 정렬
const collatorNumeric = new Intl.Collator('en-US', { numeric: true });
console.log([...files].sort(collatorNumeric.compare));
// ["file1", "file2", "file3", "file10", "file20"]
```

### 다양한 로케일 정렬

```javascript
// 독일어: ä를 a로 취급 vs ae로 취급
const germanWords = ['Äpfel', 'Apfel', 'Aepfel', 'Birne'];

// 기본 독일어 정렬
const collatorDe = new Intl.Collator('de-DE');
console.log([...germanWords].sort(collatorDe.compare));

// 전화번호부 정렬 (ä = ae)
const collatorDePhonebook = new Intl.Collator('de-DE-u-co-phonebk');
console.log([...germanWords].sort(collatorDePhonebook.compare));

// 스웨덴어: ä, ö, å는 z 뒤에 옴
const swedishWords = ['zoo', 'äpple', 'örn', 'apple'];
const collatorSv = new Intl.Collator('sv-SE');
console.log([...swedishWords].sort(collatorSv.compare));
// ["apple", "zoo", "äpple", "örn"]
```

### 검색 최적화

```javascript
// usage 옵션: "sort" (정렬용) vs "search" (검색용)
const collatorSearch = new Intl.Collator('en-US', {
  usage: 'search',
  sensitivity: 'base'
});

function search(query, items) {
  return items.filter(item =>
    collatorSearch.compare(
      item.toLowerCase().slice(0, query.length),
      query.toLowerCase()
    ) === 0
  );
}

const items = ['Café', 'cafe', 'CAFE', 'Coffee', 'Tea'];
console.log(search('cafe', items));
// ["Café", "cafe", "CAFE"]
```

---

## Intl.ListFormat

`Intl.ListFormat`은 배열을 자연스러운 문장 형태의 리스트로 포맷팅합니다.

### 기본 사용법

```javascript
const fruits = ['사과', '바나나', '오렌지'];

// 기본 (접속사)
const lfKo = new Intl.ListFormat('ko-KR');
console.log(lfKo.format(fruits));
// "사과, 바나나 및 오렌지"

const lfEn = new Intl.ListFormat('en-US');
console.log(lfEn.format(fruits));
// "사과, 바나나, and 오렌지"
```

### type 옵션

```javascript
const items = ['Apple', 'Banana', 'Orange'];

// type: "conjunction" (기본값) - "and"로 연결
const lfAnd = new Intl.ListFormat('en-US', { type: 'conjunction' });
console.log(lfAnd.format(items));
// "Apple, Banana, and Orange"

// type: "disjunction" - "or"로 연결
const lfOr = new Intl.ListFormat('en-US', { type: 'disjunction' });
console.log(lfOr.format(items));
// "Apple, Banana, or Orange"

// type: "unit" - 단위 나열 (접속사 없음)
const lfUnit = new Intl.ListFormat('en-US', { type: 'unit' });
console.log(lfUnit.format(items));
// "Apple, Banana, Orange"

// 한국어
const lfKoAnd = new Intl.ListFormat('ko-KR', { type: 'conjunction' });
console.log(lfKoAnd.format(['사과', '바나나', '오렌지']));
// "사과, 바나나 및 오렌지"

const lfKoOr = new Intl.ListFormat('ko-KR', { type: 'disjunction' });
console.log(lfKoOr.format(['사과', '바나나', '오렌지']));
// "사과, 바나나 또는 오렌지"
```

### style 옵션

```javascript
const items = ['Motorcycle', 'Bus', 'Car'];

// style: "long" (기본값)
const lfLong = new Intl.ListFormat('en-US', {
  style: 'long',
  type: 'conjunction'
});
console.log(lfLong.format(items));
// "Motorcycle, Bus, and Car"

// style: "short"
const lfShort = new Intl.ListFormat('en-US', {
  style: 'short',
  type: 'conjunction'
});
console.log(lfShort.format(items));
// "Motorcycle, Bus, & Car"

// style: "narrow"
const lfNarrow = new Intl.ListFormat('en-US', {
  style: 'narrow',
  type: 'unit'
});
console.log(lfNarrow.format(items));
// "Motorcycle Bus Car"
```

### 실용적인 예시

```javascript
// 참석자 목록 표시
function formatAttendees(names, locale = 'ko-KR') {
  if (names.length === 0) return '참석자 없음';

  const lf = new Intl.ListFormat(locale, {
    style: 'long',
    type: 'conjunction'
  });

  return `${lf.format(names)}님이 참석합니다.`;
}

console.log(formatAttendees(['김철수', '이영희']));
// "김철수 및 이영희님이 참석합니다."

console.log(formatAttendees(['김철수', '이영희', '박민수']));
// "김철수, 이영희 및 박민수님이 참석합니다."

// 필터 옵션 표시
function formatFilters(filters, locale = 'ko-KR') {
  if (filters.length === 0) return '필터 없음';

  const lf = new Intl.ListFormat(locale, {
    style: 'short',
    type: 'disjunction'
  });

  return `${lf.format(filters)} 중 선택`;
}

console.log(formatFilters(['빨강', '파랑', '초록']));
// "빨강, 파랑 또는 초록 중 선택"
```

---

## Intl.Segmenter

`Intl.Segmenter`는 텍스트를 의미 있는 단위(문자, 단어, 문장)로 분할합니다. 특히 띄어쓰기가 없는 언어(중국어, 일본어, 태국어 등)에서 유용합니다.

### 기본 사용법

```javascript
// granularity 옵션: "grapheme", "word", "sentence"

// 문자(grapheme) 단위 분할
const segmenterChar = new Intl.Segmenter('ko-KR', { granularity: 'grapheme' });
const textKo = '안녕하세요';

for (const segment of segmenterChar.segment(textKo)) {
  console.log(segment.segment);
}
// "안"
// "녕"
// "하"
// "세"
// "요"

// 이모지도 올바르게 분할
const segmenterEmoji = new Intl.Segmenter('en', { granularity: 'grapheme' });
const textEmoji = '👨‍👩‍👧‍👦🇰🇷';

for (const segment of segmenterEmoji.segment(textEmoji)) {
  console.log(segment.segment);
}
// "👨‍👩‍👧‍👦" (가족 이모지 - 하나의 grapheme)
// "🇰🇷" (국기 이모지 - 하나의 grapheme)
```

### 단어 단위 분할

```javascript
// 영어 단어 분할
const segmenterWord = new Intl.Segmenter('en-US', { granularity: 'word' });
const textEn = "Hello, world! How are you?";

const words = [];
for (const segment of segmenterWord.segment(textEn)) {
  if (segment.isWordLike) {
    words.push(segment.segment);
  }
}
console.log(words);
// ["Hello", "world", "How", "are", "you"]

// 일본어 단어 분할 (띄어쓰기 없음)
const segmenterJa = new Intl.Segmenter('ja-JP', { granularity: 'word' });
const textJa = '吾輩は猫である';

const wordsJa = [];
for (const segment of segmenterJa.segment(textJa)) {
  if (segment.isWordLike) {
    wordsJa.push(segment.segment);
  }
}
console.log(wordsJa);
// ["吾輩", "は", "猫", "で", "ある"]

// 중국어 단어 분할
const segmenterZh = new Intl.Segmenter('zh-CN', { granularity: 'word' });
const textZh = '今天天气很好';

const wordsZh = [];
for (const segment of segmenterZh.segment(textZh)) {
  if (segment.isWordLike) {
    wordsZh.push(segment.segment);
  }
}
console.log(wordsZh);
// ["今天", "天气", "很", "好"]
```

### 문장 단위 분할

```javascript
const segmenterSentence = new Intl.Segmenter('ko-KR', { granularity: 'sentence' });
const text = '안녕하세요. 반갑습니다! 오늘 날씨가 좋네요?';

const sentences = [];
for (const segment of segmenterSentence.segment(text)) {
  sentences.push(segment.segment.trim());
}
console.log(sentences);
// ["안녕하세요.", "반갑습니다!", "오늘 날씨가 좋네요?"]
```

### 실용적인 활용

```javascript
// 글자 수 세기 (이모지 포함)
function countGraphemes(text, locale = 'en') {
  const segmenter = new Intl.Segmenter(locale, { granularity: 'grapheme' });
  return [...segmenter.segment(text)].length;
}

console.log(countGraphemes('Hello'));           // 5
console.log(countGraphemes('안녕하세요'));        // 5
console.log(countGraphemes('👨‍👩‍👧‍👦'));               // 1 (가족 이모지)
console.log('👨‍👩‍👧‍👦'.length);                       // 11 (잘못된 방법)

// 단어 수 세기
function countWords(text, locale = 'en') {
  const segmenter = new Intl.Segmenter(locale, { granularity: 'word' });
  return [...segmenter.segment(text)]
    .filter(segment => segment.isWordLike)
    .length;
}

console.log(countWords('Hello world, how are you?'));  // 5
console.log(countWords('吾輩は猫である', 'ja'));         // 5

// 텍스트 하이라이팅을 위한 단어 위치 찾기
function getWordPositions(text, locale = 'en') {
  const segmenter = new Intl.Segmenter(locale, { granularity: 'word' });
  return [...segmenter.segment(text)]
    .filter(segment => segment.isWordLike)
    .map(segment => ({
      word: segment.segment,
      start: segment.index,
      end: segment.index + segment.segment.length
    }));
}

console.log(getWordPositions('Hello world'));
// [
//   { word: 'Hello', start: 0, end: 5 },
//   { word: 'world', start: 6, end: 11 }
// ]
```

---

## Intl.DisplayNames

`Intl.DisplayNames`는 언어, 지역, 스크립트, 통화 등의 표시 이름을 현지화합니다.

### 기본 사용법

```javascript
// 언어 이름
const dnLang = new Intl.DisplayNames('ko-KR', { type: 'language' });
console.log(dnLang.of('en'));      // "영어"
console.log(dnLang.of('ko'));      // "한국어"
console.log(dnLang.of('ja'));      // "일본어"
console.log(dnLang.of('zh-Hans')); // "중국어(간체)"
console.log(dnLang.of('en-US'));   // "영어(미국)"

// 영어로 표시
const dnLangEn = new Intl.DisplayNames('en-US', { type: 'language' });
console.log(dnLangEn.of('ko'));    // "Korean"
console.log(dnLangEn.of('ja'));    // "Japanese"
```

### 지역 이름

```javascript
const dnRegion = new Intl.DisplayNames('ko-KR', { type: 'region' });
console.log(dnRegion.of('US'));    // "미국"
console.log(dnRegion.of('KR'));    // "대한민국"
console.log(dnRegion.of('JP'));    // "일본"
console.log(dnRegion.of('CN'));    // "중국"
console.log(dnRegion.of('DE'));    // "독일"

// 영어로 표시
const dnRegionEn = new Intl.DisplayNames('en-US', { type: 'region' });
console.log(dnRegionEn.of('KR'));  // "South Korea"
```

### 스크립트 이름

```javascript
const dnScript = new Intl.DisplayNames('ko-KR', { type: 'script' });
console.log(dnScript.of('Hang'));  // "한글"
console.log(dnScript.of('Hani'));  // "한자"
console.log(dnScript.of('Latn'));  // "로마자"
console.log(dnScript.of('Cyrl'));  // "키릴 문자"
console.log(dnScript.of('Arab'));  // "아랍 문자"
```

### 통화 이름

```javascript
const dnCurrency = new Intl.DisplayNames('ko-KR', { type: 'currency' });
console.log(dnCurrency.of('KRW')); // "대한민국 원"
console.log(dnCurrency.of('USD')); // "미국 달러"
console.log(dnCurrency.of('EUR')); // "유로"
console.log(dnCurrency.of('JPY')); // "일본 엔화"
console.log(dnCurrency.of('CNY')); // "중국 위안화"

// 영어로 표시
const dnCurrencyEn = new Intl.DisplayNames('en-US', { type: 'currency' });
console.log(dnCurrencyEn.of('KRW')); // "South Korean Won"
```

### 날짜/시간 필드 이름

```javascript
const dnDateTime = new Intl.DisplayNames('ko-KR', { type: 'dateTimeField' });
console.log(dnDateTime.of('year'));      // "년"
console.log(dnDateTime.of('month'));     // "월"
console.log(dnDateTime.of('day'));       // "일"
console.log(dnDateTime.of('hour'));      // "시"
console.log(dnDateTime.of('minute'));    // "분"
console.log(dnDateTime.of('second'));    // "초"
console.log(dnDateTime.of('weekday'));   // "요일"
```

### style 옵션

```javascript
// style: "long", "short", "narrow"
const dnLong = new Intl.DisplayNames('ko-KR', {
  type: 'language',
  style: 'long'
});
console.log(dnLong.of('en-US'));    // "영어(미국)"

const dnShort = new Intl.DisplayNames('ko-KR', {
  type: 'language',
  style: 'short'
});
console.log(dnShort.of('en-US'));   // "영어(미국)"

const dnNarrow = new Intl.DisplayNames('ko-KR', {
  type: 'language',
  style: 'narrow'
});
console.log(dnNarrow.of('en-US'));  // "영어(미국)"
```

---

## 실전 활용 사례

### React에서의 Intl API 활용

```tsx
import { useState, useMemo } from 'react';

// 커스텀 훅: 다국어 포맷터
function useIntlFormatters(locale: string) {
  return useMemo(() => ({
    dateFormatter: new Intl.DateTimeFormat(locale, {
      dateStyle: 'long',
      timeStyle: 'short'
    }),
    numberFormatter: new Intl.NumberFormat(locale),
    currencyFormatter: new Intl.NumberFormat(locale, {
      style: 'currency',
      currency: locale === 'ko-KR' ? 'KRW' :
                locale === 'ja-JP' ? 'JPY' : 'USD'
    }),
    relativeTimeFormatter: new Intl.RelativeTimeFormat(locale, {
      numeric: 'auto'
    }),
    listFormatter: new Intl.ListFormat(locale, {
      style: 'long',
      type: 'conjunction'
    })
  }), [locale]);
}

// 사용 예시
function ProductCard({ product, locale }) {
  const { currencyFormatter, relativeTimeFormatter } = useIntlFormatters(locale);

  const formattedPrice = currencyFormatter.format(product.price);
  const timeAgo = getRelativeTimeFromDate(product.createdAt, relativeTimeFormatter);

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p className="price">{formattedPrice}</p>
      <p className="time">{timeAgo}</p>
    </div>
  );
}

// 상대 시간 계산 유틸리티
function getRelativeTimeFromDate(
  date: Date,
  formatter: Intl.RelativeTimeFormat
): string {
  const now = new Date();
  const diffMs = date.getTime() - now.getTime();
  const diffDays = Math.round(diffMs / (1000 * 60 * 60 * 24));

  if (Math.abs(diffDays) < 1) {
    const diffHours = Math.round(diffMs / (1000 * 60 * 60));
    if (Math.abs(diffHours) < 1) {
      const diffMinutes = Math.round(diffMs / (1000 * 60));
      return formatter.format(diffMinutes, 'minute');
    }
    return formatter.format(diffHours, 'hour');
  }
  return formatter.format(diffDays, 'day');
}
```

### 다국어 지원 유틸리티 클래스

```typescript
class I18nFormatter {
  private locale: string;
  private dateFormatter: Intl.DateTimeFormat;
  private numberFormatter: Intl.NumberFormat;
  private currencyFormatter: Intl.NumberFormat;
  private relativeFormatter: Intl.RelativeTimeFormat;

  constructor(locale: string, currency: string = 'USD') {
    this.locale = locale;

    this.dateFormatter = new Intl.DateTimeFormat(locale, {
      dateStyle: 'long',
      timeStyle: 'medium'
    });

    this.numberFormatter = new Intl.NumberFormat(locale);

    this.currencyFormatter = new Intl.NumberFormat(locale, {
      style: 'currency',
      currency
    });

    this.relativeFormatter = new Intl.RelativeTimeFormat(locale, {
      numeric: 'auto'
    });
  }

  formatDate(date: Date): string {
    return this.dateFormatter.format(date);
  }

  formatNumber(num: number): string {
    return this.numberFormatter.format(num);
  }

  formatCurrency(amount: number): string {
    return this.currencyFormatter.format(amount);
  }

  formatRelativeTime(date: Date): string {
    const now = new Date();
    const diffMs = date.getTime() - now.getTime();
    const diffSec = Math.round(diffMs / 1000);
    const diffMin = Math.round(diffSec / 60);
    const diffHour = Math.round(diffMin / 60);
    const diffDay = Math.round(diffHour / 24);

    if (Math.abs(diffSec) < 60) {
      return this.relativeFormatter.format(diffSec, 'second');
    }
    if (Math.abs(diffMin) < 60) {
      return this.relativeFormatter.format(diffMin, 'minute');
    }
    if (Math.abs(diffHour) < 24) {
      return this.relativeFormatter.format(diffHour, 'hour');
    }
    return this.relativeFormatter.format(diffDay, 'day');
  }

  formatCompactNumber(num: number): string {
    return new Intl.NumberFormat(this.locale, {
      notation: 'compact',
      compactDisplay: 'short'
    }).format(num);
  }
}

// 사용 예시
const koFormatter = new I18nFormatter('ko-KR', 'KRW');
const enFormatter = new I18nFormatter('en-US', 'USD');

console.log(koFormatter.formatCurrency(1234567));
// "₩1,234,567"

console.log(enFormatter.formatCurrency(1234567));
// "$1,234,567.00"

console.log(koFormatter.formatCompactNumber(1234567));
// "123만"

console.log(enFormatter.formatCompactNumber(1234567));
// "1.2M"
```

---

## 성능 고려사항

### Intl 객체 재사용

Intl 객체 생성은 비용이 크므로 재사용하는 것이 좋습니다:

```javascript
// 잘못된 방법: 매번 새 객체 생성
function formatPrices(prices) {
  return prices.map(price =>
    new Intl.NumberFormat('ko-KR', {
      style: 'currency',
      currency: 'KRW'
    }).format(price)
  );
}

// 올바른 방법: 객체 재사용
const currencyFormatter = new Intl.NumberFormat('ko-KR', {
  style: 'currency',
  currency: 'KRW'
});

function formatPricesOptimized(prices) {
  return prices.map(price => currencyFormatter.format(price));
}

// 성능 비교
console.time('without reuse');
for (let i = 0; i < 10000; i++) {
  new Intl.NumberFormat('ko-KR').format(1234567);
}
console.timeEnd('without reuse');
// without reuse: ~500ms

console.time('with reuse');
const formatter = new Intl.NumberFormat('ko-KR');
for (let i = 0; i < 10000; i++) {
  formatter.format(1234567);
}
console.timeEnd('with reuse');
// with reuse: ~20ms
```

### 캐싱 전략

```javascript
// 포맷터 캐시 구현
const formatterCache = new Map();

function getFormatter(locale, options) {
  const key = `${locale}-${JSON.stringify(options)}`;

  if (!formatterCache.has(key)) {
    formatterCache.set(key, new Intl.NumberFormat(locale, options));
  }

  return formatterCache.get(key);
}

// 사용
const formatter1 = getFormatter('ko-KR', { style: 'currency', currency: 'KRW' });
const formatter2 = getFormatter('ko-KR', { style: 'currency', currency: 'KRW' });

console.log(formatter1 === formatter2); // true (같은 인스턴스)
```

### 지연 초기화

```javascript
// 필요할 때만 포맷터 생성
class LazyFormatters {
  private _dateFormatter: Intl.DateTimeFormat | null = null;
  private _numberFormatter: Intl.NumberFormat | null = null;

  constructor(private locale: string) {}

  get dateFormatter(): Intl.DateTimeFormat {
    if (!this._dateFormatter) {
      this._dateFormatter = new Intl.DateTimeFormat(this.locale, {
        dateStyle: 'long'
      });
    }
    return this._dateFormatter;
  }

  get numberFormatter(): Intl.NumberFormat {
    if (!this._numberFormatter) {
      this._numberFormatter = new Intl.NumberFormat(this.locale);
    }
    return this._numberFormatter;
  }
}
```

---

## 브라우저 지원

대부분의 Intl API는 모든 현대 브라우저에서 지원됩니다:

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| DateTimeFormat | 24+ | 29+ | 10+ | 12+ |
| NumberFormat | 24+ | 29+ | 10+ | 12+ |
| RelativeTimeFormat | 71+ | 65+ | 14+ | 79+ |
| PluralRules | 63+ | 58+ | 13+ | 18+ |
| Collator | 24+ | 29+ | 10+ | 12+ |
| ListFormat | 72+ | 78+ | 14.1+ | 79+ |
| Segmenter | 87+ | 125+ | 14.1+ | 87+ |
| DisplayNames | 81+ | 86+ | 14.1+ | 81+ |

`Intl.Segmenter`는 비교적 최신 기능으로, 구형 브라우저 지원이 필요하면 폴리필을 사용해야 합니다.

---

## 마치며

JavaScript의 Intl API는 다국어 웹 애플리케이션 개발에 필수적인 도구입니다:

1. **DateTimeFormat**: 로케일별 날짜/시간 포맷팅
2. **NumberFormat**: 숫자, 통화, 백분율, 단위 포맷팅
3. **RelativeTimeFormat**: "어제", "3일 전" 같은 상대 시간 표시
4. **PluralRules**: 언어별 복수형 규칙 처리
5. **Collator**: 로케일 기반 문자열 정렬
6. **ListFormat**: 자연스러운 리스트 포맷팅
7. **Segmenter**: 단어, 문장, 문자 단위 텍스트 분할
8. **DisplayNames**: 언어, 지역, 통화 이름 표시

성능을 위해 Intl 객체를 재사용하고, 캐싱 전략을 적용하는 것이 중요합니다. 외부 라이브러리 없이 브라우저 내장 API만으로 강력한 국제화 기능을 구현할 수 있습니다.

## 관련 포스트

- [JavaScript Fetch API 완벽 가이드](/posts/javascript-fetch-api-network-guide/) - 네트워크 요청 API
- [JavaScript Web Storage API 가이드](/posts/javascript-web-storage-api-guide/) - 브라우저 저장소 API
- [JavaScript URL과 URLSearchParams 가이드](/posts/javascript-url-urlsearchparams-guide/) - URL 처리 API
- [JavaScript MutationObserver API 가이드](/posts/javascript-mutation-observer-guide/) - DOM 변화 감지 API

## 참고 자료

- [MDN - Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)
- [MDN - Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat)
- [MDN - Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)
- [MDN - Intl.RelativeTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat)
- [MDN - Intl.PluralRules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules)
- [MDN - Intl.Segmenter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter)
- [ECMAScript Internationalization API Specification](https://tc39.es/ecma402/)
