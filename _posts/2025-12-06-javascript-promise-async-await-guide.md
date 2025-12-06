---
title: "JavaScript Promise와 async/await 완벽 가이드 - 비동기 프로그래밍 마스터하기"
description: "JavaScript Promise와 async/await 완벽 가이드. 비동기 프로그래밍의 핵심 개념부터 에러 핸들링, Promise.all/race/allSettled 등 고급 패턴, 안티패턴, 실전 예제까지 코드와 함께 설명합니다."
date: 2025-12-06 09:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, promise, async-await, 비동기, error-handling, es2017, 콜백지옥]
---

## 들어가며

**Promise**와 **async/await**는 JavaScript 비동기 프로그래밍의 핵심입니다.

콜백 지옥을 해결하고, 비동기 코드를 동기 코드처럼 읽기 쉽게 작성할 수 있게 해주는 강력한 도구입니다. 이 글에서는 Promise의 개념부터 async/await의 실전 활용, 그리고 다양한 고급 패턴까지 깊이 있게 다룹니다.

> 이 글은 [콜백](/posts/javascript-callback/), [Event Loop](/posts/javascript-event-loop/), [Call Stack](/posts/javascript-call-stack/)에 대한 기본 지식이 있다고 가정합니다. 해당 개념이 생소하다면 먼저 읽어보시는 것을 권장합니다.
{: .prompt-info }

## Promise란?

**Promise**는 비동기 작업의 **최종 완료 또는 실패**를 나타내는 객체입니다.

### Promise의 세 가지 상태

| 상태 | 설명 | 전환 |
|------|------|------|
| **pending** | 대기 중 (초기 상태) | - |
| **fulfilled** | 작업 성공 | `resolve()` 호출 시 |
| **rejected** | 작업 실패 | `reject()` 호출 시 |

```javascript
// Promise의 세 가지 상태
const pendingPromise = new Promise(() => {});
console.log(pendingPromise); // Promise { <pending> }

const fulfilledPromise = Promise.resolve('성공');
console.log(fulfilledPromise); // Promise { '성공' }

const rejectedPromise = Promise.reject('실패');
console.log(rejectedPromise); // Promise { <rejected> '실패' }
```

> Promise는 한 번 fulfilled나 rejected 상태가 되면 **다시 변경되지 않습니다** (settled 상태).
{: .prompt-tip }

### Promise 생성하기

```javascript
const myPromise = new Promise((resolve, reject) => {
  // 비동기 작업 수행
  const success = true;

  if (success) {
    resolve('작업 성공!');  // fulfilled 상태로 전환
  } else {
    reject(new Error('작업 실패!'));  // rejected 상태로 전환
  }
});
```

### 실전 예제: 타이머 Promise

```javascript
function delay(ms) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(`${ms}ms 후 완료`);
    }, ms);
  });
}

delay(1000).then(result => {
  console.log(result); // '1000ms 후 완료'
});
```

### 실전 예제: API 호출 래핑

```javascript
function fetchUser(userId) {
  return new Promise((resolve, reject) => {
    // 실제 API 호출 시뮬레이션
    setTimeout(() => {
      if (userId > 0) {
        resolve({
          id: userId,
          name: 'Alice',
          email: 'alice@example.com'
        });
      } else {
        reject(new Error('유효하지 않은 사용자 ID'));
      }
    }, 1000);
  });
}

// 사용
fetchUser(1)
  .then(user => console.log('사용자:', user))
  .catch(error => console.error('에러:', error.message));
```

## Promise 체이닝

Promise의 가장 강력한 기능 중 하나는 **체이닝(chaining)**입니다. `.then()`, `.catch()`, `.finally()`를 연결하여 비동기 작업을 순차적으로 처리할 수 있습니다.

### .then() - 성공 처리

```javascript
fetchUser(1)
  .then(user => {
    console.log('1단계:', user.name);
    return user.id;  // 다음 .then()으로 전달
  })
  .then(userId => {
    console.log('2단계: 사용자 ID는', userId);
    return fetchPosts(userId);  // Promise 반환
  })
  .then(posts => {
    console.log('3단계: 게시물 수', posts.length);
  });
```

### .then()의 반환값

| 반환값 | 다음 .then()에서 받는 값 |
|--------|-------------------------|
| 일반 값 | 그 값 그대로 |
| Promise | Promise가 resolve된 값 |
| 아무것도 안 반환 | `undefined` |

```javascript
Promise.resolve(1)
  .then(value => value + 1)           // 2
  .then(value => Promise.resolve(value * 2))  // 4
  .then(value => { console.log(value); })     // undefined
  .then(value => console.log(value)); // undefined 출력
```

### .catch() - 에러 처리

`.catch()`는 Promise 체인 어디서든 발생한 에러를 잡습니다.

```javascript
fetchUser(-1)
  .then(user => {
    console.log('사용자:', user);
    return fetchPosts(user.id);
  })
  .then(posts => {
    console.log('게시물:', posts);
  })
  .catch(error => {
    // 위 체인 어디서든 에러가 발생하면 여기서 처리
    console.error('에러 발생:', error.message);
  });
```

### .finally() - 항상 실행

`.finally()`는 성공/실패와 관계없이 **항상 실행**됩니다.

```javascript
let isLoading = true;

fetchUser(1)
  .then(user => {
    console.log('사용자:', user);
  })
  .catch(error => {
    console.error('에러:', error);
  })
  .finally(() => {
    isLoading = false;  // 로딩 상태 해제
    console.log('작업 완료 (성공/실패 무관)');
  });
```

### 체이닝 실전 패턴

```javascript
function processOrder(orderId) {
  return validateOrder(orderId)
    .then(order => {
      console.log('주문 검증 완료');
      return checkInventory(order);
    })
    .then(inventory => {
      console.log('재고 확인 완료');
      return processPayment(inventory);
    })
    .then(payment => {
      console.log('결제 처리 완료');
      return shipOrder(payment);
    })
    .then(shipment => {
      console.log('배송 시작');
      return shipment;
    })
    .catch(error => {
      console.error('주문 처리 실패:', error.message);
      throw error;  // 에러 다시 던지기 (상위에서 처리 가능)
    })
    .finally(() => {
      console.log('주문 프로세스 종료');
    });
}
```

## async/await 문법

**async/await**는 ES2017(ES8)에서 도입된 문법으로, Promise를 더 직관적으로 사용할 수 있게 해줍니다.

### async 함수

`async` 키워드는 함수가 항상 Promise를 반환하도록 만듭니다.

```javascript
// 일반 함수 vs async 함수
function normal() {
  return 'hello';
}
console.log(normal()); // 'hello'

async function asyncFunc() {
  return 'hello';
}
console.log(asyncFunc()); // Promise { 'hello' }

// async 함수의 반환값은 자동으로 Promise.resolve()로 래핑
asyncFunc().then(value => console.log(value)); // 'hello'
```

### await 키워드

`await`는 **async 함수 내에서만** 사용 가능하며, Promise가 settle될 때까지 실행을 일시 중지합니다.

```javascript
async function getUserData() {
  console.log('1. 사용자 정보 요청 시작');

  const user = await fetchUser(1);  // Promise가 resolve될 때까지 대기
  console.log('2. 사용자:', user.name);

  const posts = await fetchPosts(user.id);  // 다시 대기
  console.log('3. 게시물 수:', posts.length);

  return { user, posts };
}

getUserData().then(data => {
  console.log('4. 최종 데이터:', data);
});
```

### 콜백 vs Promise vs async/await 비교

```javascript
// 1. 콜백 방식
getUser(userId, (error, user) => {
  if (error) {
    handleError(error);
    return;
  }
  getPosts(user.id, (error, posts) => {
    if (error) {
      handleError(error);
      return;
    }
    getComments(posts[0].id, (error, comments) => {
      if (error) {
        handleError(error);
        return;
      }
      console.log(comments);
    });
  });
});

// 2. Promise 체이닝
getUser(userId)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(error => handleError(error));

// 3. async/await
async function fetchData() {
  try {
    const user = await getUser(userId);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    console.log(comments);
  } catch (error) {
    handleError(error);
  }
}
```

> async/await는 Promise의 **문법적 설탕(syntactic sugar)**입니다. 내부적으로는 여전히 Promise를 사용합니다.
{: .prompt-info }

## 에러 핸들링

### try/catch 사용

async/await에서는 `try/catch`로 에러를 처리합니다.

```javascript
async function fetchUserSafely(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('에러 발생:', error.message);
    // 기본값 반환 또는 에러 다시 던지기
    return { user: null, posts: [] };
  }
}
```

### 개별 에러 처리

```javascript
async function fetchAllData(userId) {
  let user = null;
  let posts = [];
  let comments = [];

  // 각 작업별 개별 에러 처리
  try {
    user = await fetchUser(userId);
  } catch (error) {
    console.error('사용자 조회 실패:', error.message);
    return null;  // 사용자 없으면 중단
  }

  try {
    posts = await fetchPosts(user.id);
  } catch (error) {
    console.error('게시물 조회 실패, 빈 배열 사용');
    // posts는 이미 빈 배열이므로 계속 진행
  }

  try {
    if (posts.length > 0) {
      comments = await fetchComments(posts[0].id);
    }
  } catch (error) {
    console.error('댓글 조회 실패, 빈 배열 사용');
  }

  return { user, posts, comments };
}
```

### 에러 래퍼 유틸리티

```javascript
// 에러를 [error, data] 튜플로 반환하는 유틸리티
async function to(promise) {
  try {
    const data = await promise;
    return [null, data];
  } catch (error) {
    return [error, null];
  }
}

// 사용 예시
async function fetchData() {
  const [userError, user] = await to(fetchUser(1));
  if (userError) {
    console.error('사용자 조회 실패:', userError.message);
    return;
  }

  const [postsError, posts] = await to(fetchPosts(user.id));
  if (postsError) {
    console.error('게시물 조회 실패:', postsError.message);
    return;
  }

  console.log('성공:', { user, posts });
}
```

### Promise 자체에서 catch 사용

```javascript
async function fetchWithFallback() {
  // await와 .catch() 조합
  const user = await fetchUser(1).catch(() => null);

  if (!user) {
    console.log('사용자를 찾을 수 없어 기본값 사용');
    return { name: 'Guest' };
  }

  return user;
}
```

## Promise 정적 메서드

### Promise.all() - 병렬 실행

모든 Promise가 fulfilled되면 결과 배열을 반환합니다. **하나라도 rejected되면 전체가 rejected**됩니다.

```javascript
const userPromise = fetchUser(1);
const postsPromise = fetchPosts(1);
const commentsPromise = fetchComments(1);

// 세 요청을 병렬로 실행
Promise.all([userPromise, postsPromise, commentsPromise])
  .then(([user, posts, comments]) => {
    console.log('사용자:', user);
    console.log('게시물:', posts);
    console.log('댓글:', comments);
  })
  .catch(error => {
    console.error('하나라도 실패하면 여기로:', error);
  });
```

### async/await와 Promise.all

```javascript
async function fetchAllInParallel() {
  try {
    // 병렬 실행
    const [user, posts, comments] = await Promise.all([
      fetchUser(1),
      fetchPosts(1),
      fetchComments(1)
    ]);

    return { user, posts, comments };
  } catch (error) {
    console.error('에러:', error);
    throw error;
  }
}
```

### 순차 실행 vs 병렬 실행 성능 비교

```javascript
// 순차 실행: 3초 소요 (1초 + 1초 + 1초)
async function sequential() {
  console.time('sequential');
  const a = await delay(1000);
  const b = await delay(1000);
  const c = await delay(1000);
  console.timeEnd('sequential'); // sequential: ~3000ms
}

// 병렬 실행: 1초 소요 (동시에 실행)
async function parallel() {
  console.time('parallel');
  const [a, b, c] = await Promise.all([
    delay(1000),
    delay(1000),
    delay(1000)
  ]);
  console.timeEnd('parallel'); // parallel: ~1000ms
}
```

> 서로 의존성이 없는 비동기 작업은 `Promise.all()`로 병렬 실행하여 성능을 최적화하세요.
{: .prompt-tip }

### Promise.race() - 가장 빠른 것

가장 먼저 settled된 Promise의 결과를 반환합니다.

```javascript
const fast = new Promise(resolve => setTimeout(() => resolve('fast'), 100));
const slow = new Promise(resolve => setTimeout(() => resolve('slow'), 500));

Promise.race([fast, slow])
  .then(result => console.log(result)); // 'fast'
```

### 타임아웃 구현

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), ms);
  });

  return Promise.race([promise, timeout]);
}

// 사용 예시
async function fetchWithTimeout() {
  try {
    const data = await withTimeout(fetchUser(1), 3000);
    console.log('데이터:', data);
  } catch (error) {
    if (error.message === 'Timeout') {
      console.error('요청 시간 초과');
    } else {
      console.error('기타 에러:', error);
    }
  }
}
```

### Promise.allSettled() - 모든 결과 수집

모든 Promise가 settled될 때까지 기다리고, 각각의 결과를 객체로 반환합니다. **실패해도 다른 결과에 영향 없음**.

```javascript
const promises = [
  Promise.resolve('성공 1'),
  Promise.reject(new Error('실패')),
  Promise.resolve('성공 2')
];

Promise.allSettled(promises).then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: '성공 1' },
  //   { status: 'rejected', reason: Error: 실패 },
  //   { status: 'fulfilled', value: '성공 2' }
  // ]
});
```

### 실전 활용: 부분 실패 허용

```javascript
async function fetchMultipleAPIs(urls) {
  const promises = urls.map(url =>
    fetch(url).then(res => res.json())
  );

  const results = await Promise.allSettled(promises);

  // 성공한 것만 필터링
  const successfulData = results
    .filter(result => result.status === 'fulfilled')
    .map(result => result.value);

  // 실패한 것 로깅
  const failures = results
    .filter(result => result.status === 'rejected')
    .map(result => result.reason);

  if (failures.length > 0) {
    console.warn('일부 요청 실패:', failures);
  }

  return successfulData;
}
```

### Promise.any() - 첫 번째 성공

첫 번째로 fulfilled된 Promise의 결과를 반환합니다. **모두 rejected되면 AggregateError**를 던집니다.

```javascript
const promises = [
  Promise.reject(new Error('실패 1')),
  new Promise(resolve => setTimeout(() => resolve('성공!'), 100)),
  Promise.reject(new Error('실패 2'))
];

Promise.any(promises)
  .then(result => console.log(result))  // '성공!'
  .catch(error => console.error('모두 실패:', error));
```

### 가장 빠른 성공 응답 사용

```javascript
async function fetchFromMirrors(mirrors) {
  try {
    // 여러 미러 서버 중 가장 먼저 성공하는 것 사용
    const data = await Promise.any(
      mirrors.map(url => fetch(url).then(res => res.json()))
    );
    return data;
  } catch (error) {
    // AggregateError: 모든 Promise가 rejected된 경우
    console.error('모든 미러 서버 실패');
    throw error;
  }
}
```

### 정적 메서드 비교표

| 메서드 | 반환 조건 | 실패 처리 | 사용 사례 |
|--------|----------|----------|----------|
| `Promise.all()` | 모두 성공 | 하나라도 실패하면 전체 실패 | 모든 데이터가 필요할 때 |
| `Promise.race()` | 첫 번째 settled | 첫 번째가 실패면 실패 | 타임아웃, 가장 빠른 응답 |
| `Promise.allSettled()` | 모두 settled | 개별 결과로 확인 | 부분 실패 허용 |
| `Promise.any()` | 첫 번째 성공 | 모두 실패해야 실패 | 미러 서버, 폴백 |

## 실전 예제

### API 호출과 로딩 상태 관리

```javascript
async function fetchDataWithLoading() {
  const loadingIndicator = document.getElementById('loading');
  const errorMessage = document.getElementById('error');
  const dataContainer = document.getElementById('data');

  try {
    loadingIndicator.style.display = 'block';
    errorMessage.style.display = 'none';

    const response = await fetch('https://api.example.com/data');

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    dataContainer.innerHTML = JSON.stringify(data, null, 2);

  } catch (error) {
    errorMessage.textContent = `에러: ${error.message}`;
    errorMessage.style.display = 'block';

  } finally {
    loadingIndicator.style.display = 'none';
  }
}
```

### 재시도 로직 구현

```javascript
async function fetchWithRetry(url, maxRetries = 3, delay = 1000) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`시도 ${attempt}/${maxRetries}...`);
      const response = await fetch(url);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      return await response.json();

    } catch (error) {
      lastError = error;
      console.warn(`시도 ${attempt} 실패:`, error.message);

      if (attempt < maxRetries) {
        // 지수 백오프: 1초, 2초, 4초...
        const waitTime = delay * Math.pow(2, attempt - 1);
        console.log(`${waitTime}ms 후 재시도...`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }

  throw new Error(`${maxRetries}번 시도 후 실패: ${lastError.message}`);
}
```

### 동시 요청 제한 (Concurrency Limit)

```javascript
async function fetchWithLimit(urls, limit = 3) {
  const results = [];
  const executing = [];

  for (const url of urls) {
    // Promise 생성 (즉시 실행됨)
    const promise = fetch(url)
      .then(res => res.json())
      .then(data => {
        // 완료되면 executing 배열에서 제거
        executing.splice(executing.indexOf(promise), 1);
        return data;
      });

    results.push(promise);
    executing.push(promise);

    // 동시 실행 개수가 limit에 도달하면 하나가 완료될 때까지 대기
    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }

  // 모든 요청 완료 대기
  return Promise.all(results);
}

// 사용 예시: 100개 URL을 한 번에 3개씩 처리
const urls = Array.from({ length: 100 }, (_, i) =>
  `https://api.example.com/item/${i}`
);

fetchWithLimit(urls, 3)
  .then(results => console.log('모든 데이터:', results));
```

### 순차적 API 호출 (for...of)

```javascript
async function processItemsSequentially(items) {
  const results = [];

  for (const item of items) {
    // 각 항목을 순서대로 처리
    const result = await processItem(item);
    results.push(result);
    console.log(`${item.id} 처리 완료`);
  }

  return results;
}

// 주의: forEach는 await를 기다리지 않습니다!
// items.forEach(async (item) => {
//   await processItem(item);  // 병렬로 실행됨!
// });
```

### 캐싱과 Promise

```javascript
const cache = new Map();

async function fetchWithCache(url, ttl = 60000) {
  const cached = cache.get(url);

  if (cached && Date.now() - cached.timestamp < ttl) {
    console.log('캐시에서 반환:', url);
    return cached.data;
  }

  console.log('새로 요청:', url);
  const response = await fetch(url);
  const data = await response.json();

  cache.set(url, {
    data,
    timestamp: Date.now()
  });

  return data;
}

// 중복 요청 방지 (진행 중인 Promise 재사용)
const pendingRequests = new Map();

async function fetchWithDedupe(url) {
  // 이미 진행 중인 요청이 있으면 해당 Promise 반환
  if (pendingRequests.has(url)) {
    console.log('기존 요청 재사용:', url);
    return pendingRequests.get(url);
  }

  const promise = fetch(url)
    .then(res => res.json())
    .finally(() => {
      // 완료되면 Map에서 제거
      pendingRequests.delete(url);
    });

  pendingRequests.set(url, promise);
  return promise;
}
```

## 안티패턴과 Best Practices

### 안티패턴 1: 불필요한 Promise 래핑

```javascript
// 나쁜 예: 이미 Promise인 것을 다시 감싸기
async function fetchData() {
  return new Promise((resolve, reject) => {
    fetch('/api/data')
      .then(res => res.json())
      .then(data => resolve(data))
      .catch(err => reject(err));
  });
}

// 좋은 예: 직접 반환
async function fetchData() {
  const response = await fetch('/api/data');
  return response.json();
}
```

### 안티패턴 2: await 누락

```javascript
// 나쁜 예: await 없이 Promise 반환
async function getData() {
  const data = fetchData();  // await 누락!
  console.log(data);  // Promise { <pending> }
  return data;
}

// 좋은 예: await 사용
async function getData() {
  const data = await fetchData();
  console.log(data);  // 실제 데이터
  return data;
}
```

### 안티패턴 3: 순차 실행을 해야 할 때 병렬로 실행

```javascript
// 나쁜 예: 이전 결과가 필요한데 병렬 실행
async function processOrder() {
  const [user, order] = await Promise.all([
    getUser(),
    getOrder(userId)  // userId가 아직 없음!
  ]);
}

// 좋은 예: 의존성 있는 작업은 순차 실행
async function processOrder() {
  const user = await getUser();
  const order = await getOrder(user.id);

  // 이후 작업은 병렬 가능
  const [invoice, shipping] = await Promise.all([
    createInvoice(order),
    arrangeShipping(order)
  ]);
}
```

### 안티패턴 4: 에러 무시

```javascript
// 나쁜 예: 에러를 삼켜버림
async function fetchData() {
  try {
    return await fetch('/api/data');
  } catch (error) {
    // 에러 무시 - 절대 하지 마세요!
  }
}

// 좋은 예: 에러 처리 또는 전파
async function fetchData() {
  try {
    return await fetch('/api/data');
  } catch (error) {
    console.error('API 호출 실패:', error);
    throw error;  // 또는 기본값 반환
  }
}
```

### 안티패턴 5: forEach와 async/await

```javascript
// 나쁜 예: forEach는 await를 기다리지 않음
async function processItems(items) {
  items.forEach(async (item) => {
    await processItem(item);  // 병렬 실행됨!
  });
  console.log('완료');  // 처리가 끝나기 전에 실행됨
}

// 좋은 예: for...of 사용
async function processItems(items) {
  for (const item of items) {
    await processItem(item);  // 순차 실행
  }
  console.log('완료');  // 모든 처리 후 실행
}

// 또는 병렬 처리가 필요하면 Promise.all
async function processItems(items) {
  await Promise.all(items.map(item => processItem(item)));
  console.log('완료');
}
```

### Best Practices 정리

| 규칙 | 설명 |
|------|------|
| 불필요한 Promise 래핑 피하기 | async 함수는 이미 Promise를 반환 |
| await 잊지 않기 | Promise를 그냥 사용하면 pending 상태 |
| 의존성 파악하기 | 독립적인 작업만 병렬 처리 |
| 에러 항상 처리하기 | try/catch 또는 .catch() 사용 |
| forEach 대신 for...of | 비동기 순차 처리 시 |
| Promise.all 적극 활용 | 독립적인 비동기 작업 병렬화 |

## Top-level await

ES2022부터 모듈의 최상위 레벨에서 `await`를 사용할 수 있습니다.

```javascript
// config.js (ES 모듈)
const response = await fetch('/api/config');
export const config = await response.json();

// main.js
import { config } from './config.js';
// config가 로드된 후에 이 코드가 실행됨
console.log(config);
```

> Top-level await는 ES 모듈(`type="module"`)에서만 사용 가능합니다.
{: .prompt-warning }

## 핵심 정리

### Promise 상태

```
pending (대기)
    ↓ resolve()        ↓ reject()
fulfilled (이행)    rejected (거부)
    └──────────┬──────────┘
            settled (완료)
```

### async/await 핵심

1. **async 함수**는 항상 Promise를 반환
2. **await**는 Promise가 settle될 때까지 대기
3. **try/catch**로 에러 처리
4. 순차 vs 병렬 실행 구분하기

### Promise 정적 메서드

```javascript
Promise.all([p1, p2])       // 모두 성공해야 성공
Promise.race([p1, p2])      // 가장 빠른 것 (성공/실패 무관)
Promise.allSettled([p1, p2]) // 모든 결과 수집
Promise.any([p1, p2])       // 첫 번째 성공
```

### 성능 최적화

- 독립적인 작업은 `Promise.all()`로 병렬 처리
- 의존성 있는 작업은 순차 처리
- 동시 요청 수 제한으로 서버 부하 관리
- 캐싱과 중복 요청 방지

> Promise와 async/await를 마스터하면 복잡한 비동기 로직도 깔끔하게 작성할 수 있습니다!
{: .prompt-info }

## 관련 포스트

- [JavaScript Callback 완벽 가이드](/posts/javascript-callback/) - 비동기 프로그래밍의 기초
- [JavaScript Event Loop 완벽 가이드](/posts/javascript-event-loop/) - 비동기 처리의 핵심 메커니즘
- [JavaScript Call Stack 완벽 가이드](/posts/javascript-call-stack/) - 실행 컨텍스트와 비동기 처리

## 참고 자료

- [MDN - Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN - async function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN - await](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/await)
- [JavaScript.info - Promise](https://ko.javascript.info/promise-basics)
- [JavaScript.info - async/await](https://ko.javascript.info/async-await)
- [ECMAScript Specification - Promise](https://tc39.es/ecma262/#sec-promise-objects)
