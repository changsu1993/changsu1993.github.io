---
title: JavaScript Script 로딩 최적화 완벽 가이드 - async vs defer
date: 2020-08-24 22:25:00 +0900
categories: [개발, JavaScript]
tags: [javascript, async, performance, html]
author: changsu
toc: true
comments: true
description: JavaScript 파일 로딩 최적화를 위한 async와 defer 속성 완벽 정리. 각 방식의 동작 원리와 최적의 사용 시나리오를 실전 예제와 함께 알아봅니다.
---

## 들어가며

웹 페이지 성능 최적화에서 JavaScript 파일을 **어떻게, 언제 로딩하는가**는 매우 중요한 문제입니다.

`<script>` 태그의 위치와 `async`, `defer` 속성의 선택에 따라 페이지 로딩 속도와 사용자 경험이 크게 달라질 수 있습니다.

> 잘못된 스크립트 로딩 방식은 렌더링 차단(Render-blocking)을 일으켜 사용자가 빈 화면을 오래 보게 만들 수 있습니다.
{: .prompt-warning }

## HTML 파싱과 JavaScript 실행

브라우저가 HTML 파일을 처리하는 과정을 먼저 이해해야 합니다.

### 브라우저의 HTML 처리 과정

1. **HTML 다운로드** - 서버로부터 HTML 파일 수신
2. **파싱(Parsing)** - HTML을 한 줄씩 읽으며 DOM 트리 구성
3. **CSS 병합** - 스타일 정보를 DOM에 적용
4. **렌더링** - 최종적으로 화면에 표시

> HTML 파싱 중 `<script>` 태그를 만나면 파싱이 **중단(Blocking)**됩니다!
{: .prompt-info }

## Script 로딩 방식 비교

### 1. Head 안에 Script (기본 방식)

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script src="main.js"></script>
  </head>
  <body>
    <div>콘텐츠</div>
  </body>
</html>
```

**동작 순서:**
```
HTML 파싱 시작
    ↓
<script> 태그 발견
    ↓
HTML 파싱 중단 ⏸
    ↓
main.js 다운로드 ⬇️
    ↓
main.js 실행 ▶️
    ↓
HTML 파싱 재개 ▶️
    ↓
페이지 렌더링
```

**문제점:**

| 문제 | 설명 | 영향 |
|------|------|------|
| **렌더링 차단** | JS 다운로드/실행 중 HTML 파싱 중단 | 빈 화면 노출 시간 증가 |
| **느린 초기 로딩** | 큰 JS 파일일 경우 대기 시간 증가 | 사용자 이탈률 증가 |
| **DOM 접근 불가** | Body의 요소가 아직 없음 | querySelector 에러 발생 |

> ❌ **비추천**: 초기 로딩이 느리고 사용자 경험이 나쁩니다.
{: .prompt-danger }

### 2. Body 끝에 Script

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
  </head>
  <body>
    <div>콘텐츠</div>
    <!-- 모든 콘텐츠 후 스크립트 -->
    <script src="main.js"></script>
  </body>
</html>
```

**동작 순서:**
```
HTML 파싱 시작
    ↓
Body 콘텐츠 파싱
    ↓
페이지 렌더링 ✅ (사용자가 콘텐츠 볼 수 있음)
    ↓
<script> 태그 발견
    ↓
main.js 다운로드 ⬇️
    ↓
main.js 실행 ▶️
```

**장단점:**

| 구분 | 내용 |
|------|------|
| ✅ **장점** | HTML 콘텐츠를 빠르게 표시, DOM 요소 접근 가능 |
| ❌ **단점** | JS 의존적인 페이지는 완전한 동작까지 시간 소요 |

> ⚠️ **부분적 해결**: 콘텐츠는 빠르게 보이지만, JS 기능은 늦게 동작합니다.
{: .prompt-warning }

## Async 속성

**`async`**는 스크립트를 비동기적으로 다운로드하고, 다운로드 완료 즉시 실행합니다.

### 기본 문법

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script async src="main.js"></script>
  </head>
  <body>
    <div>콘텐츠</div>
  </body>
</html>
```

> `async`는 Boolean 속성이므로, 선언만으로 `true`가 됩니다.
{: .prompt-tip }

### 동작 원리

```
HTML 파싱 시작
    ↓
<script async> 발견
    ↓
HTML 파싱 계속 ▶️  +  main.js 다운로드 ⬇️ (병렬)
    ↓
main.js 다운로드 완료
    ↓
HTML 파싱 중단 ⏸
    ↓
main.js 실행 ▶️
    ↓
HTML 파싱 재개 ▶️
```

### 장단점

| 구분 | 내용 |
|------|------|
| ✅ **장점** | 다운로드가 HTML 파싱과 병렬로 진행되어 시간 절약 |
| ❌ **단점 1** | 실행 시점이 불확실하여 DOM 조작 시 에러 발생 가능 |
| ❌ **단점 2** | 여러 스크립트 사용 시 실행 순서 보장 안 됨 |

### 실행 순서 문제

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script async src="a.js"></script>
  <script async src="b.js"></script>
  <script async src="c.js"></script>
</head>
```

**문제점:**
- `a.js`, `b.js`, `c.js` 중 **다운로드가 먼저 완료된 순서대로** 실행
- 파일 크기나 네트워크 상태에 따라 실행 순서가 달라짐
- `c.js`가 `a.js`보다 먼저 실행될 수 있음

> ⚠️ **주의**: 스크립트 간 의존성이 있다면 `async` 사용 시 오류 발생!
{: .prompt-warning }

### 적합한 사용 사례

```html
<!-- ✅ 좋은 예: 독립적인 스크립트 -->
<script async src="google-analytics.js"></script>
<script async src="ad-script.js"></script>
<script async src="social-widget.js"></script>
```

| 사용 사례 | 설명 |
|----------|------|
| **Google Analytics** | 페이지와 독립적으로 동작 |
| **광고 스크립트** | 다른 스크립트와 의존성 없음 |
| **소셜 미디어 위젯** | 독립적인 서드파티 스크립트 |

> 💡 **추천**: 다른 코드에 의존하지 않는 독립적인 스크립트에 사용하세요!
{: .prompt-tip }

## Defer 속성

**`defer`**는 스크립트를 비동기로 다운로드하지만, **HTML 파싱 완료 후** 순서대로 실행합니다.

### 기본 문법

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script defer src="main.js"></script>
  </head>
  <body>
    <div>콘텐츠</div>
  </body>
</html>
```

### 동작 원리

```
HTML 파싱 시작
    ↓
<script defer> 발견
    ↓
HTML 파싱 계속 ▶️  +  main.js 다운로드 ⬇️ (병렬)
    ↓
HTML 파싱 완료 ✅
    ↓
DOM 구성 완료
    ↓
main.js 실행 ▶️ (다운로드 완료된 상태)
    ↓
DOMContentLoaded 이벤트 발생
```

### 장점

| 장점 | 설명 |
|------|------|
| ✅ **병렬 다운로드** | HTML 파싱과 동시에 스크립트 다운로드 |
| ✅ **렌더링 우선** | 사용자에게 콘텐츠를 먼저 보여줌 |
| ✅ **DOM 접근 가능** | HTML 파싱 완료 후 실행되므로 모든 요소 접근 가능 |
| ✅ **순서 보장** | 여러 스크립트의 실행 순서 보장 |

### 실행 순서 보장

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script defer src="jquery.js"></script>
  <script defer src="app.js"></script>
  <script defer src="main.js"></script>
</head>
```

**실행 순서:**
1. HTML 파싱과 동시에 `jquery.js`, `app.js`, `main.js` 모두 다운로드
2. HTML 파싱 완료 후
3. **순서대로** `jquery.js` → `app.js` → `main.js` 실행

> ✅ **보장**: 스크립트 간 의존성이 있어도 안전하게 사용 가능!
{: .prompt-tip }

### 적합한 사용 사례

```html
<!-- ✅ 좋은 예: DOM 조작이 필요한 스크립트 -->
<script defer src="jquery.min.js"></script>
<script defer src="app.js"></script>
<script defer src="main.js"></script>
```

| 사용 사례 | 이유 |
|----------|------|
| **DOM 조작 스크립트** | HTML 요소가 모두 로드된 후 실행 |
| **의존성 있는 스크립트** | 실행 순서 보장 필요 |
| **대부분의 애플리케이션** | 가장 안전하고 효율적인 방식 |

> 🌟 **최고의 선택**: 대부분의 경우 `defer`가 최적의 옵션입니다!
{: .prompt-tip }

## 비교표: 로딩 방식별 특징

| 방식 | 다운로드 | 실행 시점 | 파싱 차단 | 순서 보장 | 추천도 |
|------|---------|----------|----------|----------|--------|
| **Head 기본** | 즉시 | 즉시 | ⛔ 차단 | ✅ | ⭐ |
| **Body 끝** | 파싱 후 | 다운로드 후 | ❌ | ✅ | ⭐⭐ |
| **async** | 병렬 | 완료 즉시 | ⚠️ 실행 시 | ❌ | ⭐⭐⭐ |
| **defer** | 병렬 | 파싱 완료 후 | ❌ | ✅ | ⭐⭐⭐⭐⭐ |

## 실전 예제

### 일반적인 웹 애플리케이션

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My App</title>

  <!-- CSS는 head에 (렌더링 차단 방지) -->
  <link rel="stylesheet" href="styles.css">

  <!-- 애플리케이션 스크립트는 defer -->
  <script defer src="vendor/jquery.js"></script>
  <script defer src="app.js"></script>
  <script defer src="main.js"></script>

  <!-- 독립적인 분석 스크립트는 async -->
  <script async src="analytics.js"></script>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

### React/Vue 같은 SPA

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>React App</title>

  <!-- 번들 파일은 defer -->
  <script defer src="runtime.js"></script>
  <script defer src="vendor.bundle.js"></script>
  <script defer src="main.bundle.js"></script>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

### 서드파티 스크립트 혼합

```html
<head>
  <!-- 애플리케이션 코어 (defer, 순서 중요) -->
  <script defer src="app.js"></script>

  <!-- Google Analytics (async, 독립적) -->
  <script async src="https://www.google-analytics.com/analytics.js"></script>

  <!-- 광고 스크립트 (async, 독립적) -->
  <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
</head>
```

## 성능 측정

### Lighthouse로 확인하기

Chrome DevTools의 Lighthouse를 사용하여 스크립트 로딩 최적화 효과를 측정할 수 있습니다.

```html
<!-- ❌ 나쁜 예: head에 일반 script -->
<script src="large-library.js"></script>
<!-- Lighthouse Score: Performance 45 -->

<!-- ✅ 좋은 예: defer 사용 -->
<script defer src="large-library.js"></script>
<!-- Lighthouse Score: Performance 92 -->
```

### Performance API로 측정

```javascript
// DOMContentLoaded 시간 측정
window.addEventListener('DOMContentLoaded', () => {
  const perfData = performance.timing;
  const pageLoadTime = perfData.loadEventEnd - perfData.navigationStart;
  const domReadyTime = perfData.domContentLoadedEventEnd - perfData.navigationStart;

  console.log(`Page Load Time: ${pageLoadTime}ms`);
  console.log(`DOM Ready Time: ${domReadyTime}ms`);
});
```

## Bonus: use strict

### 'use strict'란?

ECMAScript 5에 추가된 **엄격 모드(Strict Mode)**입니다.

```javascript
'use strict';

// 이제 엄격한 규칙이 적용됩니다
x = 10;  // ❌ ReferenceError: x is not defined
```

### 왜 사용해야 하나?

| 이유 | 설명 |
|------|------|
| **에러 조기 발견** | 잠재적 버그를 에러로 변환 |
| **안전한 코드** | 위험한 문법 사용 방지 |
| **성능 향상** | JS 엔진이 최적화하기 쉬움 |

### 사용 방법

```javascript
// 파일 전체에 적용
'use strict';

function myFunction() {
  // 이 함수도 strict mode
}

// 또는 함수별로 적용
function strictFunction() {
  'use strict';
  // 이 함수만 strict mode
}
```

### Strict Mode의 주요 변경사항

```javascript
'use strict';

// 1. 선언되지 않은 변수 사용 금지
x = 10;  // ❌ ReferenceError

// 2. 읽기 전용 프로퍼티 쓰기 금지
const obj = {};
Object.defineProperty(obj, 'x', { value: 42, writable: false });
obj.x = 9;  // ❌ TypeError

// 3. 삭제할 수 없는 프로퍼티 삭제 금지
delete Object.prototype;  // ❌ TypeError

// 4. 중복된 매개변수 금지
function sum(a, a, c) {  // ❌ SyntaxError
  return a + a + c;
}

// 5. with문 사용 금지
with (Math) {  // ❌ SyntaxError
  x = cos(2);
}
```

> 💡 **권장사항**: TypeScript를 사용하지 않는다면 항상 `'use strict';`를 사용하세요!
{: .prompt-tip }

## 핵심 정리

### 스크립트 로딩 Best Practices

1. **기본적으로 `defer` 사용**
   - 대부분의 애플리케이션 스크립트에 적합
   - 성능과 안정성의 균형

2. **독립적인 스크립트는 `async`**
   - Google Analytics, 광고, 소셜 위젯 등
   - 페이지와 독립적으로 동작하는 경우

3. **CSS는 항상 `<head>`에**
   - 렌더링 차단을 피하기 위함
   - Critical CSS 인라인 고려

4. **큰 라이브러리는 번들링/코드 스플리팅**
   - Webpack, Rollup 등 사용
   - 필요한 코드만 로드

### 선택 가이드

```
스크립트가 DOM 조작을 하나요?
├─ Yes → defer 사용
└─ No → 다른 스크립트에 의존하나요?
    ├─ Yes → defer 사용
    └─ No → async 사용 (독립적인 경우)
```

### 성능 체크리스트

- [ ] 불필요한 스크립트 제거
- [ ] 스크립트 번들링 및 압축(minify)
- [ ] `defer` 또는 `async` 속성 사용
- [ ] 서드파티 스크립트는 `async`로 비동기 로드
- [ ] Lighthouse로 성능 측정
- [ ] `'use strict'` 사용 (TypeScript 아닌 경우)

> defer는 대부분의 경우 최고의 선택입니다. 성능과 안정성, 둘 다 잡을 수 있습니다!
{: .prompt-info }

## 참고 자료

- [MDN - The Script element](https://developer.mozilla.org/ko/docs/Web/HTML/Element/script)
- [HTML Standard - Scripting](https://html.spec.whatwg.org/multipage/scripting.html)
- [JavaScript.info - Script async, defer](https://ko.javascript.info/script-async-defer)
- [web.dev - Efficiently load JavaScript](https://web.dev/efficiently-load-third-party-javascript/)
