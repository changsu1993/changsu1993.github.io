---
title: JavaScript 타이머와 애니메이션 완벽 가이드 - setTimeout, setInterval, requestAnimationFrame
description: JavaScript의 타이머 API를 마스터하세요. setTimeout, setInterval, requestAnimationFrame의 차이점, Debounce/Throttle 패턴, 실무 활용법까지 완벽 정리합니다.
author: cotes
date: 2020-09-21 17:57:41 +0900
categories: [JavaScript, Asynchronous]
tags: [javascript, settimeout, setinterval, requestanimationframe, debounce, throttle, timer, animation]
---

## 타이머 API 개요

JavaScript는 함수를 **지연 실행**하거나 **주기적으로 실행**하기 위한 세 가지 주요 API를 제공합니다:

1. **setTimeout**: 일정 시간 후 **한 번** 실행
2. **setInterval**: 일정 간격으로 **반복** 실행
3. **requestAnimationFrame**: **브라우저 렌더링 주기**에 맞춰 실행 (애니메이션 최적화)

## setTimeout - 지연 실행

### 기본 문법

```javascript
let timerId = setTimeout(function|code, delay, arg1, arg2, ...);
```

**파라미터:**
- `function|code`: 실행할 함수 (문자열도 가능하지만 비권장)
- `delay`: 지연 시간 (ms 단위, 기본값 0)
- `arg1, arg2, ...`: 함수에 전달할 인자

### 기본 사용 예제

```javascript
// 1초 후 실행
function greet() {
  console.log('Hello!');
}

setTimeout(greet, 1000);

// 인자 전달
function greet(name, message) {
  console.log(`${name}, ${message}`);
}

setTimeout(greet, 1000, 'Alice', 'Welcome!');
// 1초 후: "Alice, Welcome!"
```

### 화살표 함수 사용

```javascript
// ✅ 권장: 화살표 함수
setTimeout(() => console.log('Hello'), 1000);

// ⚠️ 비권장: 문자열 (eval처럼 동작)
setTimeout("console.log('Hello')", 1000);
```

### 주의: 함수 호출 vs 함수 참조

```javascript
function greet() {
  return 'Hello';
}

// ❌ 잘못된 사용 - 즉시 실행됨
setTimeout(greet(), 1000);
// greet()의 반환값(undefined)이 setTimeout에 전달됨

// ✅ 올바른 사용 - 함수 참조 전달
setTimeout(greet, 1000);

// ✅ 인자가 필요하면 화살표 함수로 감싸기
setTimeout(() => greet(), 1000);
```

### clearTimeout - 타이머 취소

```javascript
const timerId = setTimeout(() => {
  console.log('실행되지 않음');
}, 1000);

// 타이머 취소
clearTimeout(timerId);

// 실용 예제: 사용자 동작 취소
let autosaveTimer;

function scheduleAutosave() {
  // 이전 타이머가 있으면 취소
  if (autosaveTimer) {
    clearTimeout(autosaveTimer);
  }

  // 새 타이머 설정
  autosaveTimer = setTimeout(() => {
    console.log('자동 저장 실행');
    // 저장 로직...
  }, 3000);
}

// 사용자가 타이핑할 때마다 호출
document.addEventListener('input', scheduleAutosave);
```

## setInterval - 주기적 실행

### 기본 문법

```javascript
let timerId = setInterval(function|code, delay, arg1, arg2, ...);
```

### 기본 사용 예제

```javascript
// 3초마다 실행
let count = 0;
const timerId = setInterval(() => {
  count++;
  console.log(`${count}초 경과`);
}, 1000);

// 5초 후 정지
setTimeout(() => {
  clearInterval(timerId);
  console.log('타이머 중지');
}, 5000);
```

### clearInterval - 반복 중지

```javascript
// 조건부 중지
let count = 0;
const timerId = setInterval(() => {
  count++;
  console.log(count);

  if (count >= 10) {
    clearInterval(timerId);
    console.log('완료!');
  }
}, 1000);
```

## setInterval vs 중첩 setTimeout

### 실행 간격의 차이

**setInterval**은 함수 실행 시간을 포함한 간격이고, **중첩 setTimeout**은 함수 실행 후의 간격입니다.

```javascript
// setInterval - 함수 실행 시간 포함
let i = 1;
setInterval(function() {
  heavyTask(i); // 100ms 걸림
}, 1000);
// 실제 간격: 900ms (1000 - 100)

// 중첩 setTimeout - 함수 실행 후 간격
let i = 1;
setTimeout(function run() {
  heavyTask(i); // 100ms 걸림
  setTimeout(run, 1000);
}, 1000);
// 실제 간격: 1000ms (보장됨)
```

### 시각적 비교

```
setInterval(func, 100):
|func(100ms)| 간격 없음 |func(100ms)| 간격 없음 |func(100ms)|
← 100ms →                 ← 100ms →

중첩 setTimeout(func, 100):
|func(100ms)| ← 100ms → |func(100ms)| ← 100ms → |func(100ms)|
```

### 중첩 setTimeout의 장점

#### 1. 유연한 간격 조정

```javascript
let delay = 5000;

let timerId = setTimeout(function request() {
  // API 요청
  fetch('/api/data')
    .then(response => response.json())
    .then(data => {
      console.log('데이터 받음:', data);
      delay = 5000; // 성공 시 간격 초기화
    })
    .catch(error => {
      console.error('에러 발생:', error);
      delay *= 2; // 실패 시 간격을 2배로 증가 (백오프)
      console.log(`다음 요청까지 ${delay}ms 대기`);
    })
    .finally(() => {
      // 다음 요청 스케줄링
      timerId = setTimeout(request, delay);
    });
}, delay);
```

#### 2. 실행 시간 측정 및 조정

```javascript
let timerId = setTimeout(function tick() {
  const start = performance.now();

  // CPU 집약적 작업
  performHeavyTask();

  const elapsed = performance.now() - start;
  console.log(`실행 시간: ${elapsed}ms`);

  // 실행 시간에 따라 다음 간격 조정
  const nextDelay = elapsed > 100 ? 2000 : 1000;

  timerId = setTimeout(tick, nextDelay);
}, 1000);
```

## setTimeout(func, 0) - 즉시 실행?

### 동작 원리

`setTimeout(func, 0)`은 **즉시 실행되지 않습니다**. 현재 코드 실행이 완료된 후 **이벤트 루프의 다음 틱**에 실행됩니다.

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

console.log('3');

// 출력 순서:
// 1
// 3
// 2
```

### 활용 사례: UI 블로킹 방지

```javascript
// ❌ UI가 멈춤
function processLargeArray(arr) {
  arr.forEach(item => {
    // 무거운 작업
    heavyProcess(item);
  });
  console.log('완료');
}

// ✅ UI 반응성 유지
function processLargeArrayAsync(arr, callback) {
  let index = 0;

  function processChunk() {
    const chunkSize = 100;
    const end = Math.min(index + chunkSize, arr.length);

    for (let i = index; i < end; i++) {
      heavyProcess(arr[i]);
    }

    index = end;

    if (index < arr.length) {
      // 다음 청크를 다음 틱에 처리
      setTimeout(processChunk, 0);
    } else {
      callback();
    }
  }

  processChunk();
}

processLargeArrayAsync(largeArray, () => {
  console.log('완료');
});
```

## requestAnimationFrame - 애니메이션 최적화

### 왜 setTimeout/setInterval이 아닌가?

```javascript
// ❌ setInterval로 애니메이션 (비효율적)
let position = 0;
setInterval(() => {
  position += 5;
  element.style.left = position + 'px';
}, 16); // 약 60fps

// 문제점:
// 1. 브라우저 렌더링 주기와 맞지 않을 수 있음
// 2. 백그라운드 탭에서도 계속 실행 (배터리 낭비)
// 3. 프레임 드롭 가능성
```

### requestAnimationFrame 장점

1. **브라우저 렌더링 주기에 동기화** (보통 60fps)
2. **백그라운드 탭에서 자동 중지** (성능 최적화)
3. **프레임 드롭 최소화**
4. **다수의 애니메이션을 하나의 리페인트로 통합**

### 기본 사용법

```javascript
function animate() {
  // 애니메이션 로직
  console.log('프레임 렌더링');

  // 다음 프레임 요청
  requestAnimationFrame(animate);
}

// 애니메이션 시작
requestAnimationFrame(animate);
```

### 실용 예제: 요소 이동 애니메이션

```javascript
const box = document.getElementById('box');
let position = 0;
let animationId;

function moveBox() {
  position += 2;
  box.style.left = position + 'px';

  // 500px까지 이동
  if (position < 500) {
    animationId = requestAnimationFrame(moveBox);
  }
}

// 시작
animationId = requestAnimationFrame(moveBox);

// 중지 (필요 시)
// cancelAnimationFrame(animationId);
```

### 시간 기반 애니메이션 (프레임 독립적)

```javascript
const box = document.getElementById('box');
let startTime = null;
const duration = 2000; // 2초
const startPosition = 0;
const endPosition = 500;

function animate(currentTime) {
  if (!startTime) startTime = currentTime;

  const elapsed = currentTime - startTime;
  const progress = Math.min(elapsed / duration, 1); // 0 ~ 1

  // Easing 함수 적용
  const easedProgress = easeInOutQuad(progress);

  const currentPosition = startPosition +
    (endPosition - startPosition) * easedProgress;

  box.style.left = currentPosition + 'px';

  if (progress < 1) {
    requestAnimationFrame(animate);
  } else {
    console.log('애니메이션 완료');
  }
}

// Easing 함수
function easeInOutQuad(t) {
  return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
}

requestAnimationFrame(animate);
```

### cancelAnimationFrame - 애니메이션 중지

```javascript
let animationId;

function animate() {
  // 애니메이션 로직
  animationId = requestAnimationFrame(animate);
}

// 시작
animationId = requestAnimationFrame(animate);

// 중지
document.getElementById('stopBtn').addEventListener('click', () => {
  cancelAnimationFrame(animationId);
  console.log('애니메이션 중지');
});
```

## 실무 패턴

### 1. Debounce (디바운스)

마지막 호출 후 일정 시간이 지나야 실행됩니다.

```javascript
function debounce(func, delay) {
  let timeoutId;

  return function(...args) {
    // 이전 타이머 취소
    clearTimeout(timeoutId);

    // 새 타이머 설정
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// 사용 예: 검색 자동완성
const searchInput = document.getElementById('search');

const performSearch = debounce((query) => {
  console.log('검색 실행:', query);
  // API 호출...
}, 300);

searchInput.addEventListener('input', (e) => {
  performSearch(e.target.value);
});
```

**사용 사례:**
- 검색 자동완성
- 폼 검증
- 윈도우 리사이즈 이벤트
- 자동 저장

### 2. Throttle (쓰로틀)

일정 시간 간격으로만 실행됩니다.

```javascript
function throttle(func, limit) {
  let inThrottle;

  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;

      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// 사용 예: 스크롤 이벤트
const handleScroll = throttle(() => {
  console.log('스크롤 위치:', window.scrollY);
  // 무한 스크롤 로직...
}, 200);

window.addEventListener('scroll', handleScroll);
```

**사용 사례:**
- 무한 스크롤
- 스크롤 이벤트 처리
- 게임 입력 처리
- 버튼 연타 방지

### Debounce vs Throttle 비교

```javascript
// 사용자가 1초 동안 10번 입력한 경우

// Debounce: 마지막 입력 후 300ms 뒤 1번만 실행
// ||||||||||| ---- [실행]

// Throttle: 200ms마다 최대 1번씩 실행
// |||||||||||
// [실행] -- [실행] -- [실행] -- [실행] -- [실행]
```

### 3. 폴링 (Polling)

주기적으로 서버에 데이터를 요청합니다.

```javascript
class Poller {
  constructor(fetchFunction, interval = 5000) {
    this.fetchFunction = fetchFunction;
    this.interval = interval;
    this.timerId = null;
    this.isRunning = false;
  }

  start() {
    if (this.isRunning) return;

    this.isRunning = true;
    this.poll();
  }

  poll() {
    this.fetchFunction()
      .then(data => {
        console.log('데이터 받음:', data);
      })
      .catch(error => {
        console.error('에러:', error);
      })
      .finally(() => {
        if (this.isRunning) {
          this.timerId = setTimeout(() => this.poll(), this.interval);
        }
      });
  }

  stop() {
    this.isRunning = false;
    if (this.timerId) {
      clearTimeout(this.timerId);
      this.timerId = null;
    }
  }
}

// 사용
const poller = new Poller(
  () => fetch('/api/notifications').then(res => res.json()),
  10000 // 10초마다
);

poller.start();

// 중지
// poller.stop();
```

### 4. 리트라이 (Retry) with 백오프

```javascript
function retryWithBackoff(fn, maxRetries = 3, initialDelay = 1000) {
  return new Promise((resolve, reject) => {
    let retries = 0;

    function attempt() {
      fn()
        .then(resolve)
        .catch(error => {
          retries++;

          if (retries >= maxRetries) {
            reject(new Error(`최대 재시도 횟수 초과: ${error.message}`));
            return;
          }

          const delay = initialDelay * Math.pow(2, retries - 1);
          console.log(`재시도 ${retries}/${maxRetries} (${delay}ms 후)`);

          setTimeout(attempt, delay);
        });
    }

    attempt();
  });
}

// 사용
retryWithBackoff(
  () => fetch('/api/data').then(res => {
    if (!res.ok) throw new Error('API 에러');
    return res.json();
  }),
  3,
  1000
)
  .then(data => console.log('성공:', data))
  .catch(error => console.error('실패:', error));

// 재시도 간격: 1초 → 2초 → 4초
```

### 5. 타임아웃 (Timeout)

```javascript
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

// 사용
withTimeout(fetch('/api/slow-endpoint'), 5000)
  .then(response => response.json())
  .then(data => console.log('데이터:', data))
  .catch(error => {
    if (error.message === 'Timeout') {
      console.error('요청 시간 초과');
    } else {
      console.error('에러:', error);
    }
  });
```

### 6. 카운트다운 타이머

```javascript
class CountdownTimer {
  constructor(duration, onTick, onComplete) {
    this.duration = duration;
    this.remaining = duration;
    this.onTick = onTick;
    this.onComplete = onComplete;
    this.timerId = null;
    this.isRunning = false;
  }

  start() {
    if (this.isRunning) return;

    this.isRunning = true;
    this.tick();
  }

  tick() {
    this.onTick(this.remaining);

    if (this.remaining <= 0) {
      this.stop();
      this.onComplete();
      return;
    }

    this.remaining--;
    this.timerId = setTimeout(() => this.tick(), 1000);
  }

  pause() {
    this.isRunning = false;
    if (this.timerId) {
      clearTimeout(this.timerId);
    }
  }

  reset() {
    this.pause();
    this.remaining = this.duration;
  }

  stop() {
    this.pause();
    this.remaining = 0;
  }
}

// 사용: 60초 카운트다운
const timer = new CountdownTimer(
  60,
  (remaining) => {
    console.log(`남은 시간: ${remaining}초`);
    document.getElementById('timer').textContent = remaining;
  },
  () => {
    console.log('시간 종료!');
    alert('시간이 다 되었습니다!');
  }
);

timer.start();
```

## 가비지 컬렉션 고려사항

### 메모리 누수 위험

```javascript
// ❌ 메모리 누수 위험
class Component {
  constructor() {
    this.data = new Array(1000000); // 큰 데이터

    setInterval(() => {
      console.log('Component 살아있음');
    }, 1000);
  }
}

const component = new Component();
// component를 더 이상 사용하지 않아도
// setInterval이 계속 실행되고 메모리에 남아있음
```

### 올바른 정리

```javascript
// ✅ 올바른 정리
class Component {
  constructor() {
    this.data = new Array(1000000);
    this.timerId = null;
  }

  start() {
    this.timerId = setInterval(() => {
      console.log('Component 살아있음');
    }, 1000);
  }

  destroy() {
    if (this.timerId) {
      clearInterval(this.timerId);
      this.timerId = null;
    }
    this.data = null;
  }
}

const component = new Component();
component.start();

// 컴포넌트가 필요없을 때
component.destroy();
```

## 브라우저 제약사항

### 최소 지연 시간 (4ms)

HTML5 표준에 따르면, **5번째 중첩 타이머부터는 최소 4ms 지연**이 강제됩니다.

```javascript
let start = Date.now();
let times = [];

setTimeout(function run() {
  times.push(Date.now() - start);

  if (times.length < 10) {
    setTimeout(run, 0);
  } else {
    console.log(times);
    // [1, 2, 3, 4, 5, 9, 13, 17, 21, 25]
    // 5번째부터 약 4ms씩 증가
  }
}, 0);
```

### 백그라운드 탭 쓰로틀링

브라우저는 백그라운드 탭의 타이머를 **최소 1초**로 쓰로틀링합니다.

```javascript
// 활성 탭: 100ms마다 실행
// 백그라운드 탭: 1000ms마다만 실행
setInterval(() => {
  console.log('실행');
}, 100);
```

## 성능 비교

### setTimeout vs requestAnimationFrame

```javascript
// 성능 테스트
console.time('setTimeout');
let count1 = 0;
const timer1 = setInterval(() => {
  count1++;
  if (count1 >= 60) {
    clearInterval(timer1);
    console.timeEnd('setTimeout');
  }
}, 16); // 약 60fps 시도

console.time('requestAnimationFrame');
let count2 = 0;
function loop() {
  count2++;
  if (count2 < 60) {
    requestAnimationFrame(loop);
  } else {
    console.timeEnd('requestAnimationFrame');
  }
}
requestAnimationFrame(loop);

// requestAnimationFrame이 더 정확하고 효율적
```

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 타이머는 항상 정리
const timerId = setTimeout(() => {}, 1000);
// 사용 후 정리
clearTimeout(timerId);

// 2. 애니메이션은 requestAnimationFrame 사용
requestAnimationFrame(animate);  // ✅
setInterval(animate, 16);        // ❌

// 3. Debounce/Throttle 활용
const search = debounce(query => {}, 300);  // ✅
searchInput.addEventListener('input', e => {
  fetchAPI(e.target.value);  // ❌ 매번 호출
});

// 4. 컴포넌트 정리
class Component {
  destroy() {
    clearTimeout(this.timerId);
    clearInterval(this.intervalId);
    cancelAnimationFrame(this.animationId);
  }
}

// 5. 에러 처리
setTimeout(() => {
  try {
    riskyOperation();
  } catch (error) {
    console.error('타이머 에러:', error);
  }
}, 1000);
```

### ❌ 피해야 할 사항

```javascript
// 1. 문자열 평가
setTimeout("console.log('bad')", 1000);  // ❌
setTimeout(() => console.log('good'), 1000);  // ✅

// 2. 함수 즉시 실행
setTimeout(myFunction(), 1000);  // ❌
setTimeout(myFunction, 1000);    // ✅

// 3. setInterval로 애니메이션
setInterval(animate, 16);           // ❌
requestAnimationFrame(animate);     // ✅

// 4. 타이머 정리 안 함
setInterval(() => {}, 1000);  // ❌ 메모리 누수
const id = setInterval(() => {}, 1000);
clearInterval(id);  // ✅
```

## 비교 정리표

| 기능 | setTimeout | setInterval | requestAnimationFrame |
|------|-----------|------------|----------------------|
| **용도** | 지연 실행 | 주기적 실행 | 애니메이션 |
| **반복** | 1회 | 무한 | 무한 |
| **간격 보장** | N/A | ❌ | ✅ |
| **백그라운드 중지** | ❌ | ❌ | ✅ |
| **프레임 동기화** | ❌ | ❌ | ✅ |
| **최소 지연** | 4ms (중첩 시) | 4ms | ~16ms (60fps) |
| **정리 함수** | clearTimeout | clearInterval | cancelAnimationFrame |

## 핵심 정리

### 언제 무엇을 사용할까?

1. **setTimeout**: 일회성 지연 실행
   - 자동 저장, 알림 표시, 지연된 API 호출

2. **중첩 setTimeout**: 유연한 주기적 실행
   - 폴링, 백오프 재시도, 조건부 간격 조정

3. **setInterval**: 고정 간격 반복 (권장 X)
   - 정확한 간격이 중요하지 않은 경우만

4. **requestAnimationFrame**: 부드러운 애니메이션
   - CSS 애니메이션, 캔버스 애니메이션, 스크롤 효과

### 실무 패턴 요약

- **Debounce**: 마지막 호출 후 실행 (검색 자동완성)
- **Throttle**: 일정 간격으로만 실행 (스크롤 이벤트)
- **Polling**: 주기적 데이터 요청 (알림 확인)
- **Retry**: 실패 시 재시도 (API 요청)
- **Timeout**: 시간 제한 (느린 API 차단)

> **결론**: 타이머는 강력하지만 남용하면 성능 저하와 메모리 누수를 초래합니다. 적절한 API를 선택하고, 항상 정리하는 습관을 들이세요!
{: .prompt-tip }

## 참고 자료

- [MDN - setTimeout](https://developer.mozilla.org/ko/docs/Web/API/setTimeout)
- [MDN - setInterval](https://developer.mozilla.org/ko/docs/Web/API/setInterval)
- [MDN - requestAnimationFrame](https://developer.mozilla.org/ko/docs/Web/API/window/requestAnimationFrame)
- [JavaScript.info - Scheduling](https://ko.javascript.info/settimeout-setinterval)
- [HTML5 Timers Specification](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html)
