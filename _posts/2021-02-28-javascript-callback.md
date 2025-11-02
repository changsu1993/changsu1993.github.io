---
title: JavaScript Callback 완벽 가이드 - 비동기 프로그래밍의 시작
date: 2021-02-28 18:13:00 +0900
categories: [개발, JavaScript]
tags: [javascript, callback, 비동기, async, 동기, synchronous, asynchronous, callback-hell, 콜백지옥]
author: changsu
toc: true
comments: true
description: JavaScript Callback 함수의 모든 것. 동기/비동기 콜백의 차이부터 콜백 지옥 해결법까지, 비동기 프로그래밍의 기초를 완벽하게 이해합니다.
---

## 들어가며

**Callback(콜백)**은 JavaScript 비동기 프로그래밍의 **가장 기초적인 패턴**입니다.

`setTimeout`, 이벤트 리스너, AJAX 요청 등 모든 비동기 작업은 콜백으로 시작되었습니다. 콜백을 이해하지 못하면 Promise, async/await도 제대로 이해할 수 없습니다.

> JavaScript는 싱글 스레드지만 콜백 덕분에 비동기 작업을 처리할 수 있습니다!
{: .prompt-info }

## 동기 vs 비동기

### 동기(Synchronous)

**동기**는 코드가 **작성된 순서대로** 한 줄씩 실행되는 방식입니다.

```javascript
console.log('1');
console.log('2');
console.log('3');

// 출력:
// 1
// 2
// 3
```

**동작 방식:**
```
코드 1 실행 → 완료 대기 → 코드 2 실행 → 완료 대기 → 코드 3 실행
```

#### 동기의 문제점

```javascript
console.log('작업 시작');

// 5초 걸리는 무거운 작업 (예시)
for (let i = 0; i < 5000000000; i++) {
  // 무거운 연산...
}

console.log('작업 완료');
console.log('다음 작업');

// 문제: 5초 동안 브라우저가 멈춤 (UI 차단)
```

> ⚠️ **문제**: 무거운 작업이 끝날 때까지 다음 코드가 실행되지 않습니다!
{: .prompt-danger }

### 비동기(Asynchronous)

**비동기**는 작업을 시작하고 **완료를 기다리지 않고** 다음 코드를 실행하는 방식입니다.

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 1000);

console.log('3');

// 출력:
// 1
// 3
// 2 (1초 후)
```

**동작 방식:**
```
코드 1 실행 → setTimeout 등록 → 코드 3 실행 → (1초 후) 콜백 실행
```

### 비교표

| 구분 | 동기 | 비동기 |
|------|------|--------|
| **실행 순서** | 코드 작성 순서대로 | 완료 시점이 다를 수 있음 |
| **대기** | 작업 완료까지 대기 | 대기하지 않음 |
| **UI 차단** | 차단 가능 | 차단 없음 |
| **예시** | 일반 코드, 반복문 | setTimeout, fetch, 이벤트 |

## 콜백 함수란?

**콜백 함수**는 **다른 함수의 인자로 전달되는 함수**입니다.

### 기본 개념

```javascript
// greet는 콜백 함수
function greet(name) {
  console.log(`Hello, ${name}!`);
}

// processUserInput은 콜백을 받는 함수
function processUserInput(callback) {
  const name = 'Alice';
  callback(name);  // 콜백 실행
}

processUserInput(greet);  // 'Hello, Alice!'
```

### 인라인 콜백

```javascript
processUserInput(function(name) {
  console.log(`Hi, ${name}!`);
});

// 또는 화살표 함수
processUserInput(name => {
  console.log(`Hey, ${name}!`);
});
```

> 💡 **핵심**: 콜백은 함수를 **값처럼** 전달하여 나중에 실행하는 패턴입니다!
{: .prompt-tip }

## 콜백의 두 가지 유형

### 1. Synchronous Callback (동기 콜백)

**즉시 실행**되는 콜백입니다.

```javascript
// 배열 메서드들은 동기 콜백 사용
const numbers = [1, 2, 3, 4, 5];

console.log('시작');

numbers.forEach(num => {
  console.log(num);
});

console.log('끝');

// 출력:
// 시작
// 1
// 2
// 3
// 4
// 5
// 끝
```

#### 다른 예시

```javascript
// map, filter, reduce도 동기 콜백
const doubled = numbers.map(num => num * 2);
const evens = numbers.filter(num => num % 2 === 0);
const sum = numbers.reduce((acc, num) => acc + num, 0);

console.log(doubled);  // [2, 4, 6, 8, 10]
console.log(evens);    // [2, 4]
console.log(sum);      // 15
```

### 2. Asynchronous Callback (비동기 콜백)

**나중에 실행**되는 콜백입니다.

```javascript
console.log('시작');

setTimeout(() => {
  console.log('타임아웃 콜백');
}, 1000);

console.log('끝');

// 출력:
// 시작
// 끝
// 타임아웃 콜백 (1초 후)
```

#### 다양한 비동기 콜백 예시

```javascript
// 1. 타이머
setTimeout(() => {
  console.log('1초 후 실행');
}, 1000);

setInterval(() => {
  console.log('1초마다 실행');
}, 1000);

// 2. 이벤트 리스너
button.addEventListener('click', () => {
  console.log('버튼 클릭됨');
});

// 3. AJAX (fetch 이전 방식)
$.ajax({
  url: '/api/users',
  success: function(data) {
    console.log('데이터 받음:', data);
  },
  error: function(error) {
    console.error('에러 발생:', error);
  }
});
```

### 비교표

| 구분 | Synchronous Callback | Asynchronous Callback |
|------|---------------------|----------------------|
| **실행 시점** | 즉시 실행 | 나중에 실행 |
| **코드 흐름** | 순차적 | 비순차적 |
| **예시** | forEach, map, filter | setTimeout, 이벤트, AJAX |

## 실전 활용 예제

### 1. 배열 처리

```javascript
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 17 }
];

// 성인 사용자의 이름만 추출
function getAdultNames(users, callback) {
  const adults = users.filter(user => user.age >= 18);
  const names = adults.map(user => user.name);
  callback(names);
}

getAdultNames(users, (names) => {
  console.log('성인 사용자:', names);
  // ['Alice', 'Bob']
});
```

### 2. 데이터 로딩

```javascript
function loadData(callback) {
  console.log('데이터 로딩 시작...');

  setTimeout(() => {
    const data = { id: 1, name: 'Sample Data' };
    callback(data);
  }, 2000);
}

loadData((data) => {
  console.log('데이터 로드 완료:', data);
});

console.log('다른 작업 계속...');

// 출력:
// 데이터 로딩 시작...
// 다른 작업 계속...
// (2초 후) 데이터 로드 완료: { id: 1, name: 'Sample Data' }
```

### 3. 순차 실행

```javascript
function step1(callback) {
  setTimeout(() => {
    console.log('Step 1 완료');
    callback();
  }, 1000);
}

function step2(callback) {
  setTimeout(() => {
    console.log('Step 2 완료');
    callback();
  }, 1000);
}

function step3(callback) {
  setTimeout(() => {
    console.log('Step 3 완료');
    callback();
  }, 1000);
}

// 순차 실행
step1(() => {
  step2(() => {
    step3(() => {
      console.log('모든 작업 완료!');
    });
  });
});
```

## 콜백 지옥 (Callback Hell)

여러 비동기 작업을 순차적으로 실행할 때 **콜백 안에 콜백을 계속 중첩**하면 코드가 복잡해집니다.

### 콜백 지옥의 예

```javascript
// 사용자 정보 → 포스트 → 댓글 순서로 가져오기
getUser(userId, (user) => {
  getUserPosts(user.id, (posts) => {
    getPostComments(posts[0].id, (comments) => {
      getCommentAuthor(comments[0].authorId, (author) => {
        console.log('댓글 작성자:', author);
        // 😱 4단계 중첩!
      });
    });
  });
});
```

### 실제 예시: 로그인 → 데이터 로드 → 처리

```javascript
function login(email, password, successCallback, errorCallback) {
  setTimeout(() => {
    if (email && password) {
      successCallback({ userId: 1, email: email });
    } else {
      errorCallback(new Error('로그인 실패'));
    }
  }, 1000);
}

function getUserData(userId, successCallback, errorCallback) {
  setTimeout(() => {
    if (userId) {
      successCallback({ name: 'Alice', roles: ['admin'] });
    } else {
      errorCallback(new Error('사용자 데이터 로드 실패'));
    }
  }, 1000);
}

function getPurchaseHistory(userId, successCallback, errorCallback) {
  setTimeout(() => {
    if (userId) {
      successCallback([{ item: 'Book', price: 20 }]);
    } else {
      errorCallback(new Error('구매 내역 로드 실패'));
    }
  }, 1000);
}

// ❌ 콜백 지옥!
login(
  'alice@example.com',
  'password123',
  (user) => {
    console.log('로그인 성공:', user);
    getUserData(
      user.userId,
      (userData) => {
        console.log('사용자 데이터:', userData);
        getPurchaseHistory(
          user.userId,
          (purchases) => {
            console.log('구매 내역:', purchases);
            // 더 깊은 중첩 계속...
          },
          (error) => {
            console.error('구매 내역 에러:', error);
          }
        );
      },
      (error) => {
        console.error('사용자 데이터 에러:', error);
      }
    );
  },
  (error) => {
    console.error('로그인 에러:', error);
  }
);
```

### 콜백 지옥의 문제점

| 문제 | 설명 |
|------|------|
| **가독성 저하** | 코드가 오른쪽으로 계속 들여쓰기됨 |
| **유지보수 어려움** | 수정 시 전체 구조 파악 필요 |
| **에러 처리 복잡** | 각 단계마다 에러 핸들러 필요 |
| **디버깅 어려움** | 어디서 에러 발생했는지 추적 힘듦 |

> ⚠️ **Pyramid of Doom**: 콜백 지옥을 "파멸의 피라미드"라고도 부릅니다!
{: .prompt-warning }

## 콜백 지옥 해결 방법

### 1. 함수 분리

```javascript
// ✅ 개선: 함수를 분리하여 평탄하게
function onLoginSuccess(user) {
  console.log('로그인 성공:', user);
  getUserData(user.userId, onGetUserDataSuccess, onError);
}

function onGetUserDataSuccess(userData) {
  console.log('사용자 데이터:', userData);
  // 다음 작업...
}

function onError(error) {
  console.error('에러 발생:', error);
}

login('alice@example.com', 'password123', onLoginSuccess, onError);
```

### 2. Promise 사용 (권장)

```javascript
// ✅ Promise로 개선
function loginPromise(email, password) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (email && password) {
        resolve({ userId: 1, email: email });
      } else {
        reject(new Error('로그인 실패'));
      }
    }, 1000);
  });
}

function getUserDataPromise(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId) {
        resolve({ name: 'Alice', roles: ['admin'] });
      } else {
        reject(new Error('사용자 데이터 로드 실패'));
      }
    }, 1000);
  });
}

// Promise 체이닝
loginPromise('alice@example.com', 'password123')
  .then(user => {
    console.log('로그인 성공:', user);
    return getUserDataPromise(user.userId);
  })
  .then(userData => {
    console.log('사용자 데이터:', userData);
  })
  .catch(error => {
    console.error('에러:', error);
  });
```

### 3. async/await 사용 (최고 권장)

```javascript
// ✅ async/await로 개선 (가장 깔끔!)
async function processUserLogin() {
  try {
    const user = await loginPromise('alice@example.com', 'password123');
    console.log('로그인 성공:', user);

    const userData = await getUserDataPromise(user.userId);
    console.log('사용자 데이터:', userData);

    const purchases = await getPurchaseHistoryPromise(user.userId);
    console.log('구매 내역:', purchases);
  } catch (error) {
    console.error('에러:', error);
  }
}

processUserLogin();
```

## 에러 처리 패턴

### Node.js 스타일: Error-first Callback

```javascript
function readFile(filename, callback) {
  // 비동기 작업 시뮬레이션
  setTimeout(() => {
    const error = null;  // 에러가 없으면 null
    const data = 'File content';

    // 첫 번째 인자는 에러, 두 번째는 데이터
    callback(error, data);
  }, 1000);
}

// 사용
readFile('test.txt', (error, data) => {
  if (error) {
    console.error('에러 발생:', error);
    return;
  }
  console.log('데이터:', data);
});
```

### 성공/실패 콜백 분리

```javascript
function fetchData(successCallback, errorCallback) {
  setTimeout(() => {
    const success = Math.random() > 0.5;

    if (success) {
      successCallback({ data: 'Success!' });
    } else {
      errorCallback(new Error('Failed!'));
    }
  }, 1000);
}

// 사용
fetchData(
  (data) => console.log('성공:', data),
  (error) => console.error('실패:', error)
);
```

## 실전 패턴

### 1. 병렬 실행 후 결과 수집

```javascript
function parallelTasks(tasks, callback) {
  let completed = 0;
  const results = [];

  tasks.forEach((task, index) => {
    task((result) => {
      results[index] = result;
      completed++;

      if (completed === tasks.length) {
        callback(results);
      }
    });
  });
}

// 사용 예
const tasks = [
  (cb) => setTimeout(() => cb('Task 1'), 1000),
  (cb) => setTimeout(() => cb('Task 2'), 500),
  (cb) => setTimeout(() => cb('Task 3'), 1500)
];

parallelTasks(tasks, (results) => {
  console.log('모든 작업 완료:', results);
  // ['Task 1', 'Task 2', 'Task 3']
});
```

### 2. 순차 실행 헬퍼

```javascript
function series(tasks, finalCallback) {
  let index = 0;
  const results = [];

  function next() {
    if (index >= tasks.length) {
      finalCallback(results);
      return;
    }

    tasks[index]((result) => {
      results.push(result);
      index++;
      next();
    });
  }

  next();
}

// 사용 예
const sequentialTasks = [
  (cb) => setTimeout(() => { console.log('Step 1'); cb('Result 1'); }, 1000),
  (cb) => setTimeout(() => { console.log('Step 2'); cb('Result 2'); }, 1000),
  (cb) => setTimeout(() => { console.log('Step 3'); cb('Result 3'); }, 1000)
];

series(sequentialTasks, (results) => {
  console.log('순차 작업 완료:', results);
});
```

### 3. 재시도 로직

```javascript
function retry(task, maxAttempts, callback) {
  let attempts = 0;

  function attempt() {
    attempts++;
    task(
      (result) => {
        callback(null, result);
      },
      (error) => {
        if (attempts < maxAttempts) {
          console.log(`재시도 ${attempts}/${maxAttempts}...`);
          setTimeout(attempt, 1000);
        } else {
          callback(error);
        }
      }
    );
  }

  attempt();
}

// 사용 예
function unreliableTask(success, failure) {
  const shouldSucceed = Math.random() > 0.7;
  setTimeout(() => {
    if (shouldSucceed) {
      success('Success!');
    } else {
      failure(new Error('Failed'));
    }
  }, 500);
}

retry(unreliableTask, 3, (error, result) => {
  if (error) {
    console.error('최종 실패:', error);
  } else {
    console.log('성공:', result);
  }
});
```

## 콜백 vs Promise vs async/await

```javascript
// 1. 콜백
fetchUser(userId, (error, user) => {
  if (error) {
    handleError(error);
  } else {
    fetchPosts(user.id, (error, posts) => {
      if (error) {
        handleError(error);
      } else {
        console.log(posts);
      }
    });
  }
});

// 2. Promise
fetchUserPromise(userId)
  .then(user => fetchPostsPromise(user.id))
  .then(posts => console.log(posts))
  .catch(error => handleError(error));

// 3. async/await
async function getUserPosts(userId) {
  try {
    const user = await fetchUserPromise(userId);
    const posts = await fetchPostsPromise(user.id);
    console.log(posts);
  } catch (error) {
    handleError(error);
  }
}
```

| 패턴 | 가독성 | 에러 처리 | 학습 곡선 | 권장도 |
|------|--------|----------|----------|--------|
| **Callback** | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Promise** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **async/await** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |

## 핵심 정리

### 콜백의 종류

1. **Synchronous Callback**: 즉시 실행 (forEach, map, filter)
2. **Asynchronous Callback**: 나중에 실행 (setTimeout, 이벤트, AJAX)

### 콜백 지옥 해결

1. **함수 분리**: 중첩 줄이기
2. **Promise**: 체이닝으로 평탄하게
3. **async/await**: 동기 코드처럼 작성

### Best Practices

```javascript
// ✅ 좋은 예
function fetchData(callback) {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => callback(null, data))
    .catch(error => callback(error));
}

// ❌ 나쁜 예: 콜백 지옥
fetchA((a) => {
  fetchB(a, (b) => {
    fetchC(b, (c) => {
      // 너무 깊은 중첩...
    });
  });
});
```

### 마이그레이션 가이드

```
레거시 콜백 코드가 있다면
├─ 새 코드는 Promise/async-await 사용
├─ 기존 콜백 → Promise 래퍼 작성
└─ 점진적으로 리팩토링
```

> 콜백은 JavaScript 비동기의 기초입니다. 콜백을 이해해야 Promise와 async/await도 완벽하게 이해할 수 있습니다!
{: .prompt-info }

## 참고 자료

- [MDN - Callback function](https://developer.mozilla.org/ko/docs/Glossary/Callback_function)
- [JavaScript.info - Callbacks](https://ko.javascript.info/callbacks)
- [Node.js - Error-first callbacks](https://nodejs.org/en/knowledge/errors/what-are-the-error-conventions/)
- [CallbackHell.com](https://callbackhell.com/)
