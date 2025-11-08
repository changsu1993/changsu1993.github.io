---
title: JavaScript 날짜와 시간 완벽 가이드 - Date 객체 마스터하기
description: JavaScript의 Date 객체를 사용한 날짜와 시간 처리 방법을 상세히 알아봅니다. Date 생성, 포맷팅, 타임스탬프, 날짜 계산, moment.js 대안까지 실전에서 바로 활용할 수 있는 가이드를 제공합니다.
author: changsu
date: 2020-06-22 00:18:34 +0900
categories: [Programming, JavaScript]
tags: [javascript, date, time, timestamp, datetime, date-formatting, timezone, date-fns]
---

## JavaScript Date 객체

**Date 객체**는 JavaScript에서 날짜와 시간을 다루기 위한 내장 객체입니다. 웹 개발에서 날짜와 시간은 필수적인 요소입니다.

### Date가 필요한 경우

```javascript
// 실전 예시
// - 회원가입 날짜 기록
// - 게시글 작성 시간 표시
// - 이벤트 시작/종료 시간 관리
// - 현재 시간 표시
// - 날짜 간격 계산 (D-day)
// - 로그 타임스탬프
```

## Date 객체 생성

### 현재 날짜와 시간

```javascript
let now = new Date();
console.log(now);
// Mon Jun 22 2020 00:18:34 GMT+0900 (한국 표준시)

console.log(typeof now);  // "object"
```

### 특정 날짜와 시간

```javascript
// 문자열로 생성 (다양한 형식 지원)
let date1 = new Date('December 17, 2019 03:24:00');
let date2 = new Date('2019-12-17T03:24:00');
let date3 = new Date('2019-12-17');

// 숫자로 생성 (년, 월, 일, 시, 분, 초, 밀리초)
// ⚠️ 월은 0부터 시작 (0 = 1월, 11 = 12월)
let date4 = new Date(2019, 11, 17);           // 2019년 12월 17일
let date5 = new Date(2019, 11, 17, 15, 30);   // 2019년 12월 17일 15:30
let date6 = new Date(2019, 11, 17, 15, 30, 45, 500);  // 밀리초까지

console.log(date4);  // Tue Dec 17 2019 00:00:00 GMT+0900

// 타임스탬프로 생성 (밀리초)
let date7 = new Date(1592752530566);
console.log(date7);  // Mon Jun 22 2020 00:15:30 GMT+0900
```

### 월 인덱스 주의사항

```javascript
// ❌ 잘못된 예 (12월을 12로 작성)
let wrongDate = new Date(2019, 12, 17);  // 2020년 1월 17일

// ✅ 올바른 예 (12월을 11로 작성)
let correctDate = new Date(2019, 11, 17);  // 2019년 12월 17일

// 월 매핑
// 0: 1월, 1: 2월, 2: 3월, ... 11: 12월
```

## Date 메서드

### Get 메서드 (날짜/시간 가져오기)

```javascript
let now = new Date();

// 연도
console.log(now.getFullYear());  // 2020

// 월 (0-11, 1월 = 0)
console.log(now.getMonth());     // 5 (6월)
console.log(now.getMonth() + 1); // 6 (실제 월)

// 일
console.log(now.getDate());      // 22

// 요일 (0-6, 일요일 = 0)
console.log(now.getDay());       // 1 (월요일)

// 시간
console.log(now.getHours());     // 0

// 분
console.log(now.getMinutes());   // 18

// 초
console.log(now.getSeconds());   // 34

// 밀리초
console.log(now.getMilliseconds());  // 123
```

### 요일 한글로 변환

```javascript
function getDayName(date) {
  const days = ['일요일', '월요일', '화요일', '수요일', '목요일', '금요일', '토요일'];
  return days[date.getDay()];
}

let now = new Date();
console.log(getDayName(now));  // "월요일"
```

### Set 메서드 (날짜/시간 설정)

```javascript
let date = new Date();

// 연도 설정
date.setFullYear(2025);

// 월 설정 (0-11)
date.setMonth(11);  // 12월

// 일 설정
date.setDate(25);

// 시간 설정
date.setHours(15);
date.setMinutes(30);
date.setSeconds(0);

console.log(date);  // Thu Dec 25 2025 15:30:00 GMT+0900

// 여러 값 동시 설정
date.setFullYear(2025, 11, 25);  // 2025년 12월 25일
```

## 타임스탬프 (Timestamp)

### getTime() - 밀리초 반환

```javascript
let now = new Date();
let timestamp = now.getTime();

console.log(timestamp);  // 1592752530566 (밀리초)

// 1970년 1월 1일 00:00:00 UTC 이후 경과한 밀리초
```

### Date.now() - 현재 타임스탬프

```javascript
// 현재 타임스탬프 (더 빠름)
let timestamp1 = Date.now();
let timestamp2 = new Date().getTime();

console.log(timestamp1);  // 1592752530566
console.log(timestamp2);  // 1592752530566

// 성능 측정에 유용
let start = Date.now();
// 어떤 작업...
let end = Date.now();
console.log(`실행 시간: ${end - start}ms`);
```

### 타임스탬프 비교

```javascript
let date1 = new Date('2020-01-01');
let date2 = new Date('2020-12-31');

console.log(date1.getTime() < date2.getTime());  // true (date1이 더 과거)

// 날짜 차이 계산 (일 단위)
let diffMs = date2.getTime() - date1.getTime();
let diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
console.log(`${diffDays}일 차이`);  // 365일 차이
```

## 날짜 포맷팅

### toLocaleString() 계열

```javascript
let now = new Date();

// 로케일 날짜 문자열
console.log(now.toLocaleDateString());
// "2020. 6. 22."

console.log(now.toLocaleDateString('en-US'));
// "6/22/2020"

console.log(now.toLocaleDateString('ko-KR', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  weekday: 'long'
}));
// "2020년 6월 22일 월요일"

// 로케일 시간 문자열
console.log(now.toLocaleTimeString());
// "오전 12:18:34"

console.log(now.toLocaleTimeString('en-US'));
// "12:18:34 AM"

// 날짜와 시간 모두
console.log(now.toLocaleString('ko-KR'));
// "2020. 6. 22. 오전 12:18:34"
```

### 커스텀 포맷팅

```javascript
function formatDate(date, format = 'YYYY-MM-DD') {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');

  return format
    .replace('YYYY', year)
    .replace('MM', month)
    .replace('DD', day)
    .replace('HH', hours)
    .replace('mm', minutes)
    .replace('ss', seconds);
}

let now = new Date();
console.log(formatDate(now));                          // "2020-06-22"
console.log(formatDate(now, 'YYYY년 MM월 DD일'));       // "2020년 06월 22일"
console.log(formatDate(now, 'YYYY-MM-DD HH:mm:ss'));   // "2020-06-22 00:18:34"
```

### 상대 시간 표시

```javascript
function getRelativeTime(date) {
  const now = new Date();
  const diffMs = now.getTime() - date.getTime();
  const diffSec = Math.floor(diffMs / 1000);
  const diffMin = Math.floor(diffSec / 60);
  const diffHour = Math.floor(diffMin / 60);
  const diffDay = Math.floor(diffHour / 24);

  if (diffSec < 60) {
    return '방금 전';
  } else if (diffMin < 60) {
    return `${diffMin}분 전`;
  } else if (diffHour < 24) {
    return `${diffHour}시간 전`;
  } else if (diffDay < 7) {
    return `${diffDay}일 전`;
  } else {
    return formatDate(date, 'YYYY-MM-DD');
  }
}

let posted = new Date(Date.now() - 1000 * 60 * 30);  // 30분 전
console.log(getRelativeTime(posted));  // "30분 전"

let yesterday = new Date(Date.now() - 1000 * 60 * 60 * 25);  // 25시간 전
console.log(getRelativeTime(yesterday));  // "1일 전"
```

## 날짜 계산

### 날짜 더하기/빼기

```javascript
let now = new Date();

// 7일 후
let after7Days = new Date(now);
after7Days.setDate(now.getDate() + 7);
console.log(formatDate(after7Days));

// 3개월 후
let after3Months = new Date(now);
after3Months.setMonth(now.getMonth() + 3);
console.log(formatDate(after3Months));

// 1년 전
let before1Year = new Date(now);
before1Year.setFullYear(now.getFullYear() - 1);
console.log(formatDate(before1Year));

// 시간 더하기 (밀리초 사용)
let after2Hours = new Date(now.getTime() + 2 * 60 * 60 * 1000);
console.log(formatDate(after2Hours, 'YYYY-MM-DD HH:mm'));
```

### D-Day 계산

```javascript
function getDDay(targetDate) {
  const now = new Date();
  const target = new Date(targetDate);

  // 시간 제거 (날짜만 비교)
  now.setHours(0, 0, 0, 0);
  target.setHours(0, 0, 0, 0);

  const diffMs = target.getTime() - now.getTime();
  const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));

  if (diffDays > 0) {
    return `D-${diffDays}`;
  } else if (diffDays === 0) {
    return 'D-Day';
  } else {
    return `D+${Math.abs(diffDays)}`;
  }
}

console.log(getDDay('2024-12-25'));  // "D-550" (예시)
console.log(getDDay(new Date()));    // "D-Day"
```

### 두 날짜 사이의 기간

```javascript
function getDateDiff(startDate, endDate) {
  const start = new Date(startDate);
  const end = new Date(endDate);

  const diffMs = Math.abs(end.getTime() - start.getTime());

  const days = Math.floor(diffMs / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diffMs % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60));

  return {
    days,
    hours,
    minutes,
    totalDays: diffMs / (1000 * 60 * 60 * 24),
    totalHours: diffMs / (1000 * 60 * 60),
    totalMinutes: diffMs / (1000 * 60)
  };
}

let start = new Date('2020-01-01');
let end = new Date('2020-12-31');
let diff = getDateDiff(start, end);

console.log(`${diff.days}일 ${diff.hours}시간 ${diff.minutes}분`);
// "365일 0시간 0분"
```

## 타임존 (Timezone)

### GMT/UTC

```javascript
let now = new Date();

// 로컬 시간
console.log(now.toString());
// "Mon Jun 22 2020 00:18:34 GMT+0900 (한국 표준시)"

// UTC 시간
console.log(now.toUTCString());
// "Sun, 21 Jun 2020 15:18:34 GMT"

// ISO 형식 (UTC)
console.log(now.toISOString());
// "2020-06-21T15:18:34.566Z"

// 타임존 오프셋 (분 단위)
console.log(now.getTimezoneOffset());
// -540 (UTC+9는 -540분)
```

### UTC 메서드

```javascript
let now = new Date();

// UTC 기준 값 가져오기
console.log(now.getUTCFullYear());   // 2020
console.log(now.getUTCMonth());      // 5
console.log(now.getUTCDate());       // 21
console.log(now.getUTCHours());      // 15 (한국 시간 00시)

// UTC로 Date 생성
let utcDate = new Date(Date.UTC(2020, 5, 22, 0, 0, 0));
console.log(utcDate.toISOString());
// "2020-06-22T00:00:00.000Z"
```

## 실전 예제

### 1. 나이 계산

```javascript
function calculateAge(birthDate) {
  const birth = new Date(birthDate);
  const now = new Date();

  let age = now.getFullYear() - birth.getFullYear();
  const monthDiff = now.getMonth() - birth.getMonth();

  // 생일이 지나지 않았으면 -1
  if (monthDiff < 0 || (monthDiff === 0 && now.getDate() < birth.getDate())) {
    age--;
  }

  return age;
}

console.log(calculateAge('1990-05-15'));  // 30 (2020년 기준)
console.log(calculateAge('2000-12-25'));  // 19
```

### 2. 영업일 계산

```javascript
function addBusinessDays(startDate, days) {
  let date = new Date(startDate);
  let addedDays = 0;

  while (addedDays < days) {
    date.setDate(date.getDate() + 1);

    // 주말 제외 (0 = 일요일, 6 = 토요일)
    if (date.getDay() !== 0 && date.getDay() !== 6) {
      addedDays++;
    }
  }

  return date;
}

let today = new Date('2020-06-22');  // 월요일
let after5BusinessDays = addBusinessDays(today, 5);
console.log(formatDate(after5BusinessDays));  // "2020-06-29" (월요일)
```

### 3. 월의 마지막 날

```javascript
function getLastDayOfMonth(year, month) {
  // 다음 달 0일 = 이번 달 마지막 날
  return new Date(year, month + 1, 0).getDate();
}

console.log(getLastDayOfMonth(2020, 1));   // 29 (2월, 윤년)
console.log(getLastDayOfMonth(2020, 3));   // 30 (4월)
console.log(getLastDayOfMonth(2020, 11));  // 31 (12월)

// 윤년 판별
function isLeapYear(year) {
  return getLastDayOfMonth(year, 1) === 29;
}

console.log(isLeapYear(2020));  // true
console.log(isLeapYear(2021));  // false
```

### 4. 디지털 시계

```javascript
function updateClock() {
  const now = new Date();
  const hours = String(now.getHours()).padStart(2, '0');
  const minutes = String(now.getMinutes()).padStart(2, '0');
  const seconds = String(now.getSeconds()).padStart(2, '0');

  console.log(`${hours}:${minutes}:${seconds}`);
}

// 1초마다 업데이트
setInterval(updateClock, 1000);
```

### 5. 이벤트 타이머

```javascript
function getTimeUntilEvent(eventDate) {
  const now = new Date();
  const event = new Date(eventDate);
  const diff = event.getTime() - now.getTime();

  if (diff < 0) {
    return '이벤트가 종료되었습니다';
  }

  const days = Math.floor(diff / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((diff % (1000 * 60)) / 1000);

  return `${days}일 ${hours}시간 ${minutes}분 ${seconds}초 남음`;
}

console.log(getTimeUntilEvent('2024-12-31 23:59:59'));
```

## 날짜 라이브러리

JavaScript의 Date 객체는 한계가 있어 실무에서는 라이브러리를 많이 사용합니다.

### date-fns (권장)

```javascript
// npm install date-fns

import { format, addDays, differenceInDays } from 'date-fns';
import { ko } from 'date-fns/locale';

// 포맷팅
let now = new Date();
console.log(format(now, 'yyyy년 MM월 dd일'));
// "2020년 06월 22일"

console.log(format(now, 'PPP', { locale: ko }));
// "2020년 6월 22일"

// 날짜 계산
let after7Days = addDays(now, 7);
let diff = differenceInDays(new Date('2020-12-31'), now);
```

### Day.js (경량)

```javascript
// npm install dayjs

import dayjs from 'dayjs';

// 포맷팅
console.log(dayjs().format('YYYY-MM-DD HH:mm:ss'));
// "2020-06-22 00:18:34"

// 날짜 계산
console.log(dayjs().add(7, 'day').format('YYYY-MM-DD'));
// "2020-06-29"

// 상대 시간
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.extend(relativeTime);

console.log(dayjs('2020-06-21').fromNow());
// "a day ago"
```

### Luxon (현대적)

```javascript
// npm install luxon

import { DateTime } from 'luxon';

// 현재 시간
let now = DateTime.now();

// 포맷팅
console.log(now.toFormat('yyyy-MM-dd HH:mm:ss'));
// "2020-06-22 00:18:34"

// 타임존 변환
let nyTime = now.setZone('America/New_York');
console.log(nyTime.toFormat('HH:mm'));
```

## 참고 자료

- [MDN - Date](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [MDN - Intl.DateTimeFormat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat)
- [date-fns](https://date-fns.org/)
- [Day.js](https://day.js.org/)
- [Luxon](https://moment.github.io/luxon/)
- [JavaScript.info - Date and time](https://javascript.info/date)

## 마치며

JavaScript의 Date 객체는 날짜와 시간을 다루는 기본 도구입니다:

1. **Date 생성**: `new Date()`, 문자열, 숫자, 타임스탬프
2. **Get/Set 메서드**: 날짜 요소 가져오기/설정하기
3. **타임스탬프**: `getTime()`, `Date.now()`로 밀리초 값 활용
4. **포맷팅**: `toLocaleString()`, 커스텀 포맷 함수
5. **날짜 계산**: 더하기/빼기, D-Day, 기간 계산
6. **라이브러리**: date-fns, Day.js, Luxon 활용

복잡한 날짜 처리가 필요한 경우 date-fns나 Day.js 같은 라이브러리 사용을 권장합니다.