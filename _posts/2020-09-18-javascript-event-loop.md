---
title: JavaScript Event Loop 완벽 가이드 - 비동기 처리의 핵심 메커니즘
date: 2020-09-18 16:43:00 +0900
categories: [개발, JavaScript]
tags: [javascript, event-loop, async, promise]
author: changsu
toc: true
comments: true
description: JavaScript Event Loop의 동작 원리부터 Microtask와 Macrotask의 차이, 실전 활용까지 비동기 처리의 핵심을 완벽하게 이해합니다. Call Stack, Web API, Task Queue의 상호작용을 시각화로 배우고 Promise와 setTimeout의 실행 순서를 예측하는 방법을 익힙니다. 싱글 스레드 환경에서의 비동기 처리 메커니즘을 마스터합니다.
---

## 들어가며

**Event Loop**는 JavaScript의 비동기 처리를 가능하게 하는 핵심 메커니즘입니다.

JavaScript는 싱글 스레드 언어이지만, Event Loop 덕분에 비동기 작업을 효율적으로 처리할 수 있습니다.

> Event Loop를 이해하면 `setTimeout`, `Promise`, `async/await` 같은 비동기 코드의 동작 순서를 정확하게 예측할 수 있습니다.
{: .prompt-info }

## Event Loop란?

**Event Loop**는 Call Stack과 Message Queue의 상태를 지속적으로 체크하여, **Call Stack이 비어있을 때** Message Queue의 첫 번째 콜백을 Call Stack으로 이동시키는 메커니즘입니다.

### Event Loop의 핵심 역할

| 역할 | 설명 |
|------|------|
| **모니터링** | Call Stack과 Queue 상태 지속 감시 |
| **작업 스케줄링** | 적절한 타이밍에 콜백 실행 |
| **동시성 제어** | 싱글 스레드에서 비동기 처리 가능하게 함 |

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout');
}, 0);

console.log('End');

// 출력:
// Start
// End
// Timeout (setTimeout이 0초여도 동기 코드 이후에 실행)
```

> Event Loop의 이러한 반복적인 행동을 **Tick**이라고 부릅니다.
{: .prompt-tip }

## Event Loop 동작 원리

### JavaScript 런타임 구성 요소

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="background: rgba(255,255,255,0.95); padding: 20px; border-radius: 10px;">
    <div style="text-align: center; font-size: 18px; font-weight: bold; color: #2c3e50; margin-bottom: 20px; padding-bottom: 15px; border-bottom: 3px solid #667eea;">
      JavaScript Runtime
    </div>

    <!-- Call Stack & Heap -->
    <div style="display: flex; gap: 15px; justify-content: center; margin-bottom: 20px;">
      <div style="flex: 1; max-width: 200px;">
        <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); color: white; padding: 15px; border-radius: 8px; text-align: center; font-weight: bold; box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);">
          Call Stack
        </div>
      </div>
      <div style="flex: 1; max-width: 200px;">
        <div style="background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%); color: white; padding: 15px; border-radius: 8px; text-align: center; font-weight: bold; box-shadow: 0 4px 10px rgba(231, 76, 60, 0.3);">
          Heap
        </div>
      </div>
    </div>

    <!-- Web APIs -->
    <div style="margin-bottom: 20px;">
      <div style="background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%); color: white; padding: 15px; border-radius: 8px; box-shadow: 0 4px 10px rgba(243, 156, 18, 0.3);">
        <div style="font-weight: bold; margin-bottom: 10px; text-align: center; font-size: 16px;">
          Web APIs
        </div>
        <div style="display: flex; gap: 10px; justify-content: center; flex-wrap: wrap;">
          <div style="background: rgba(255,255,255,0.2); padding: 5px 12px; border-radius: 5px; font-size: 14px;">setTimeout</div>
          <div style="background: rgba(255,255,255,0.2); padding: 5px 12px; border-radius: 5px; font-size: 14px;">DOM Events</div>
          <div style="background: rgba(255,255,255,0.2); padding: 5px 12px; border-radius: 5px; font-size: 14px;">fetch</div>
        </div>
      </div>
    </div>

    <!-- Microtask & Macrotask Queues -->
    <div style="display: flex; gap: 15px; justify-content: center; margin-bottom: 20px;">
      <div style="flex: 1; max-width: 200px;">
        <div style="background: linear-gradient(135deg, #9b59b6 0%, #8e44ad 100%); color: white; padding: 15px; border-radius: 8px; text-align: center; box-shadow: 0 4px 10px rgba(155, 89, 182, 0.3);">
          <div style="font-weight: bold; margin-bottom: 8px;">Microtask Queue</div>
          <div style="font-size: 13px; opacity: 0.9;">Promise</div>
        </div>
      </div>
      <div style="flex: 1; max-width: 200px;">
        <div style="background: linear-gradient(135deg, #16a085 0%, #138d75 100%); color: white; padding: 15px; border-radius: 8px; text-align: center; box-shadow: 0 4px 10px rgba(22, 160, 133, 0.3);">
          <div style="font-weight: bold; margin-bottom: 8px;">Macrotask Queue</div>
          <div style="font-size: 13px; opacity: 0.9;">setTimeout</div>
        </div>
      </div>
    </div>

    <!-- Event Loop -->
    <div style="text-align: center; margin-top: 20px;">
      <div style="display: inline-block; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 12px 30px; border-radius: 25px; font-weight: bold; font-size: 16px; box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);">
        ⚡ Event Loop ⚡
      </div>
      <div style="margin-top: 10px; color: #7f8c8d; font-size: 14px;">
        ↑ 지속적으로 모니터링 및 작업 스케줄링 ↑
      </div>
    </div>
  </div>
</div>

### Event Loop 처리 순서

1. **동기 코드** 실행 (Call Stack)
2. **Microtask Queue** 확인 및 실행 (비어있을 때까지 모두 실행)
3. **렌더링** 수행 (필요한 경우)
4. **Macrotask Queue**에서 하나의 작업 실행
5. 1번으로 돌아가서 반복 (Tick)

## Tick이란?

**Tick**은 Event Loop가 한 사이클을 완료하는 것을 의미합니다.

### Tick의 과정

```javascript
// 첫 번째 Tick
console.log('1');  // 동기 코드 실행

setTimeout(() => {
  console.log('2'); // 다음 Tick에서 실행
}, 0);

Promise.resolve().then(() => {
  console.log('3'); // 현재 Tick의 Microtask에서 실행
});

console.log('4');  // 동기 코드 실행

// 출력: 1, 4, 3, 2
```

**실행 순서 분석:**

| Tick | 단계 | 실행 내용 | 출력 |
|------|------|-----------|------|
| 1 | 동기 코드 | `console.log('1')` | `'1'` |
| 1 | 동기 코드 | `setTimeout` 등록 | - |
| 1 | 동기 코드 | `Promise.resolve().then()` 등록 | - |
| 1 | 동기 코드 | `console.log('4')` | `'4'` |
| 1 | Microtask | Promise 콜백 실행 | `'3'` |
| 2 | Macrotask | setTimeout 콜백 실행 | `'2'` |

> 하나의 Tick 안에서 모든 Microtask가 실행되지만, Macrotask는 하나씩만 실행됩니다.
{: .prompt-warning }

## Microtask vs Macrotask

JavaScript의 비동기 작업은 두 가지 Queue로 분류됩니다.

### Microtask Queue

**Microtask**는 현재 실행 중인 스크립트 직후에 실행되는 작업입니다.

| Microtask 생성 API | 설명 |
|-------------------|------|
| `Promise.then()` | Promise 체이닝 |
| `Promise.catch()` | Promise 에러 처리 |
| `Promise.finally()` | Promise 완료 처리 |
| `queueMicrotask()` | 명시적 Microtask 등록 |
| `MutationObserver` | DOM 변화 감지 |

### Macrotask Queue

**Macrotask**는 다음 Event Loop Tick에서 실행되는 작업입니다.

| Macrotask 생성 API | 설명 |
|-------------------|------|
| `setTimeout()` | 지연 실행 |
| `setInterval()` | 반복 실행 |
| `setImmediate()` | 즉시 실행 (Node.js) |
| `requestAnimationFrame()` | 애니메이션 프레임 |
| `I/O 작업` | 파일 읽기/쓰기 |

### 우선순위 비교

```javascript
setTimeout(() => console.log('setTimeout'), 0);
Promise.resolve().then(() => console.log('Promise'));
queueMicrotask(() => console.log('queueMicrotask'));
console.log('Sync');

// 출력:
// Sync
// Promise
// queueMicrotask
// setTimeout
```

**우선순위:**
```
동기 코드 > Microtask Queue > Macrotask Queue
```

> Microtask는 **현재 Tick**에서 실행되고, Macrotask는 **다음 Tick**에서 실행됩니다.
{: .prompt-tip }

## 실전 예제

### 예제 1: 복잡한 비동기 순서

```javascript
console.log('Script start');

setTimeout(() => {
  console.log('setTimeout 1');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
  })
  .then(() => {
    console.log('Promise 2');
  });

setTimeout(() => {
  console.log('setTimeout 2');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 3');
});

console.log('Script end');
```

**출력 순서:**
```
Script start
Script end
Promise 1
Promise 3
Promise 2
setTimeout 1
setTimeout 2
```

**단계별 분석:**

| Tick | 단계 | 실행 내용 |
|------|------|-----------|
| 1 | 동기 | `Script start` |
| 1 | 동기 | `setTimeout 1` 등록 → Macrotask Queue |
| 1 | 동기 | `Promise 1, 2` 체인 등록 → Microtask Queue |
| 1 | 동기 | `setTimeout 2` 등록 → Macrotask Queue |
| 1 | 동기 | `Promise 3` 등록 → Microtask Queue |
| 1 | 동기 | `Script end` |
| 1 | Microtask | `Promise 1` → 다시 `Promise 2` Microtask 등록 |
| 1 | Microtask | `Promise 3` |
| 1 | Microtask | `Promise 2` |
| 2 | Macrotask | `setTimeout 1` |
| 3 | Macrotask | `setTimeout 2` |

### 예제 2: Microtask의 연쇄 실행

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    return Promise.resolve();
  })
  .then(() => {
    console.log('Promise 2');
  });

console.log('End');
```

**출력:**
```
Start
End
Promise 1
Promise 2
Timeout
```

> Promise 체이닝은 모두 Microtask Queue에서 처리되므로, setTimeout보다 먼저 실행됩니다.
{: .prompt-info }

### 예제 3: 무한 Microtask 주의

```javascript
// ⚠️ 주의: 이 코드는 브라우저를 멈추게 합니다!
function recursiveMicrotask() {
  queueMicrotask(() => {
    console.log('Microtask');
    recursiveMicrotask();  // 무한 재귀
  });
}

// recursiveMicrotask(); // 실행하지 마세요!
```

**문제점:**
- Microtask는 현재 Tick에서 Queue가 빌 때까지 계속 실행됨
- 무한히 Microtask를 생성하면 Event Loop가 다음 Tick으로 넘어가지 못함
- 렌더링이 차단되어 브라우저가 응답하지 않음

> **해결책**: Macrotask(`setTimeout`)를 사용하면 렌더링 기회를 제공합니다.
{: .prompt-warning }

```javascript
// ✅ 개선된 버전
function safeRecursion() {
  setTimeout(() => {
    console.log('Macrotask');
    safeRecursion();
  }, 0);
}
```

## Event Loop와 렌더링

### 렌더링 타이밍

브라우저는 **Macrotask 실행 후, 다음 Macrotask 실행 전**에 렌더링을 수행합니다.

```javascript
const button = document.querySelector('button');
const display = document.querySelector('#display');

button.addEventListener('click', () => {
  display.textContent = 'Loading...';

  // ❌ 동기 작업: 렌더링이 차단됨
  for (let i = 0; i < 1000000000; i++) {
    // 무거운 작업
  }

  display.textContent = 'Done!';
  // 사용자는 'Loading...'을 볼 수 없음!
});
```

**개선 방법:**

```javascript
button.addEventListener('click', () => {
  display.textContent = 'Loading...';

  // ✅ setTimeout으로 렌더링 기회 제공
  setTimeout(() => {
    for (let i = 0; i < 1000000000; i++) {
      // 무거운 작업
    }
    display.textContent = 'Done!';
  }, 0);
  // 사용자는 'Loading...'을 볼 수 있음!
});
```

### requestAnimationFrame의 활용

```javascript
function animateProgress() {
  let progress = 0;

  function updateProgress() {
    progress += 1;
    progressBar.style.width = progress + '%';

    if (progress < 100) {
      requestAnimationFrame(updateProgress);  // 다음 렌더링 직전에 실행
    }
  }

  requestAnimationFrame(updateProgress);
}
```

> `requestAnimationFrame`은 렌더링 직전에 실행되어 부드러운 애니메이션을 보장합니다.
{: .prompt-tip }

## 성능 최적화

### 1. 무거운 작업 분할

```javascript
// ❌ 나쁜 예: UI 차단
function processLargeArray(array) {
  array.forEach(item => {
    heavyProcessing(item);
  });
}

// ✅ 좋은 예: 작업 분할
async function processLargeArrayAsync(array, chunkSize = 100) {
  for (let i = 0; i < array.length; i += chunkSize) {
    const chunk = array.slice(i, i + chunkSize);

    chunk.forEach(item => {
      heavyProcessing(item);
    });

    // 다음 청크 처리 전에 다른 작업 기회 제공
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

### 2. Debouncing과 Throttling

```javascript
// Debounce: 마지막 호출 후 일정 시간 대기
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Throttle: 일정 시간 간격으로 실행
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      func.apply(this, args);
    }
  };
}

// 사용 예
input.addEventListener('input', debounce(handleInput, 300));
window.addEventListener('scroll', throttle(handleScroll, 100));
```

### 3. 우선순위 관리

```javascript
// 높은 우선순위: Microtask 사용
function highPriorityTask() {
  queueMicrotask(() => {
    console.log('High priority');
  });
}

// 낮은 우선순위: setTimeout 사용
function lowPriorityTask() {
  setTimeout(() => {
    console.log('Low priority');
  }, 0);
}

// 즉각적인 피드백이 필요한 경우
button.addEventListener('click', () => {
  highPriorityTask();  // 사용자 피드백
  lowPriorityTask();   // 분석 데이터 전송
});
```

## 디버깅 팁

### Chrome DevTools 활용

```javascript
// Performance 탭에서 Event Loop 모니터링
console.time('Task');
setTimeout(() => {
  console.timeEnd('Task');
}, 0);

// Long Task 감지
if ('PerformanceObserver' in window) {
  const observer = new PerformanceObserver(list => {
    for (const entry of list.getEntries()) {
      console.warn('Long task detected:', entry.duration);
    }
  });

  observer.observe({ entryTypes: ['longtask'] });
}
```

### 실행 순서 시각화

```javascript
function logWithTiming(message) {
  const timestamp = performance.now().toFixed(2);
  console.log(`[${timestamp}ms] ${message}`);
}

logWithTiming('Start');

setTimeout(() => logWithTiming('setTimeout'), 0);
Promise.resolve().then(() => logWithTiming('Promise'));

logWithTiming('End');
```

## 핵심 정리

### Event Loop 처리 순서

```
1. Call Stack 실행 (동기 코드)
   ↓
2. Microtask Queue 비우기 (모든 Microtask 실행)
   ↓
3. 렌더링 (필요한 경우)
   ↓
4. Macrotask Queue에서 하나 실행
   ↓
5. 1번으로 돌아가기 (Tick 반복)
```

### Microtask vs Macrotask 비교

| 구분 | Microtask | Macrotask |
|------|-----------|-----------|
| **실행 타이밍** | 현재 Tick | 다음 Tick |
| **처리 방식** | Queue가 빌 때까지 모두 실행 | Tick당 하나씩 실행 |
| **우선순위** | 높음 | 낮음 |
| **렌더링** | 렌더링 전에 실행 | 렌더링 후에 실행 |
| **예시** | Promise, queueMicrotask | setTimeout, setInterval |

### Best Practices

1. **무거운 작업은 분할**하여 UI 차단 방지
2. **우선순위에 맞는 Queue 선택** (중요한 작업은 Microtask)
3. **무한 Microtask 생성 주의** (렌더링 차단 위험)
4. **Debouncing/Throttling** 활용으로 과도한 실행 방지
5. **Performance API**로 성능 모니터링

> Event Loop를 이해하면 비동기 코드의 실행 순서를 예측하고, 성능 문제를 해결할 수 있습니다.
{: .prompt-info }

## 참고 자료

- [MDN - Event Loop](https://developer.mozilla.org/ko/docs/Web/JavaScript/Event_loop)
- [Jake Archibald - In The Loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
- [JavaScript Visualizer 9000](https://www.jsv9000.app/)
- [Loupe - Call Stack Visualizer](https://latentflip.com/loupe/)
