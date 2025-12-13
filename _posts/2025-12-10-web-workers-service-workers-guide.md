---
title: "Web Workers와 Service Workers 완벽 가이드 - JavaScript 멀티스레딩과 오프라인 지원"
description: "JavaScript 싱글 스레드 한계를 극복하는 Web Workers와 Service Workers 완벽 가이드. 5가지 캐싱 전략, Worker Pool 패턴, Workbox 설정부터 React/Next.js 실전 예제까지 상세히 다룹니다."
date: 2025-12-10 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [web-workers, service-workers, javascript, pwa, offline, caching, workbox, multithreading, performance, background-tasks, cache-strategies, worker-pool]
---

## 개요

JavaScript는 **싱글 스레드** 언어입니다. [Call Stack](/posts/javascript-call-stack/)에서 코드를 순차적으로 실행하며, [Event Loop](/posts/javascript-event-loop/)를 통해 비동기 작업을 처리하지만, 복잡한 계산이나 대용량 데이터 처리는 여전히 메인 스레드를 블로킹하여 UI가 멈추는 현상을 유발합니다.

**Web Workers**와 **Service Workers**는 이 한계를 극복하는 브라우저 API입니다:

| Worker 유형 | 역할 | 주요 사용 사례 |
|-------------|------|----------------|
| **Web Workers** | 백그라운드 스레드에서 연산 처리 | 대용량 데이터 처리, 이미지 처리, 암호화 |
| **Service Workers** | 네트워크 요청 가로채기, 캐싱 | 오프라인 지원, 푸시 알림, 백그라운드 동기화 |

이 가이드에서는 다음을 학습합니다:

- **Part 1**: Web Workers로 메인 스레드 블로킹 해결
- **Part 2**: Service Workers로 오프라인 지원 구현
- **Part 3**: Workbox를 활용한 실전 PWA 개발

> 이 글은 [Event Loop 완벽 가이드](/posts/javascript-event-loop/)와 [웹 성능 최적화 가이드](/posts/web-performance-optimization-guide/)를 먼저 읽으시면 더 깊이 이해할 수 있습니다.
{: .prompt-info }

---

## Part 1: Web Workers - JavaScript 멀티스레딩

### 1.1 Web Workers가 필요한 이유

JavaScript는 싱글 스레드이므로, 무거운 연산이 실행되면 UI가 멈춥니다:

```javascript
// 메인 스레드에서 무거운 연산 실행
function calculatePrimes(max) {
  const primes = [];
  for (let i = 2; i <= max; i++) {
    let isPrime = true;
    for (let j = 2; j < i; j++) {
      if (i % j === 0) {
        isPrime = false;
        break;
      }
    }
    if (isPrime) primes.push(i);
  }
  return primes;
}

// 이 함수 실행 중에는 버튼 클릭, 스크롤 등 모든 UI가 멈춤
const primes = calculatePrimes(1000000);
```

> 위 코드를 실행하면 브라우저가 몇 초간 응답하지 않습니다. 이를 **"메인 스레드 블로킹"**이라고 합니다.
{: .prompt-warning }

**Web Workers의 해결책:**

```javascript
// 무거운 연산을 별도 스레드에서 실행
const worker = new Worker('prime-worker.js');

worker.postMessage({ max: 1000000 });

worker.onmessage = (event) => {
  console.log('소수 계산 완료:', event.data.length);
};

// 연산 중에도 UI는 정상 동작
console.log('UI가 멈추지 않습니다!');
```

### 1.2 Web Worker 기본 사용법

#### Worker 생성

```javascript
// main.js
const worker = new Worker('worker.js');

// 메시지 전송
worker.postMessage({ type: 'START', data: [1, 2, 3, 4, 5] });

// 결과 수신
worker.onmessage = (event) => {
  console.log('Worker 결과:', event.data);
};

// 에러 처리
worker.onerror = (error) => {
  console.error('Worker 에러:', error.message);
};

// Worker 종료
// worker.terminate();
```

```javascript
// worker.js
self.onmessage = (event) => {
  const { type, data } = event.data;

  if (type === 'START') {
    // 무거운 연산 수행
    const result = data.map(x => x * x);

    // 결과 전송
    self.postMessage({ result });
  }
};
```

#### postMessage와 onmessage의 데이터 흐름

| 순서 | Main Thread | 방향 | Worker Thread |
|:----:|-------------|:----:|---------------|
| 1 | `postMessage({ data })` | → | 메시지 수신 |
| 2 | (대기) | | `onmessage` 실행 |
| 3 | (대기) | | 연산 처리 |
| 4 | 메시지 수신 | ← | `postMessage({ result })` |
| 5 | `onmessage` 실행 | | (완료) |

### 1.3 Dedicated Workers vs Shared Workers

#### Dedicated Worker (전용 워커)

하나의 스크립트에서만 사용하는 워커입니다:

```javascript
// 하나의 페이지에서만 사용
const dedicatedWorker = new Worker('worker.js');
```

#### Shared Worker (공유 워커)

여러 탭, 창, iframe에서 공유하는 워커입니다:

```javascript
// shared-worker.js
const connections = [];

self.onconnect = (event) => {
  const port = event.ports[0];
  connections.push(port);

  port.onmessage = (e) => {
    // 모든 연결된 클라이언트에게 브로드캐스트
    connections.forEach(conn => {
      conn.postMessage(`브로드캐스트: ${e.data}`);
    });
  };

  port.start();
};
```

```javascript
// main.js (여러 탭에서 동일한 worker 공유)
const sharedWorker = new SharedWorker('shared-worker.js');

sharedWorker.port.onmessage = (event) => {
  console.log('메시지 수신:', event.data);
};

sharedWorker.port.start();
sharedWorker.port.postMessage('안녕하세요!');
```

**Shared Worker 사용 사례:**
- 여러 탭 간 실시간 데이터 동기화
- WebSocket 연결 공유
- 공유 캐시 관리

### 1.4 Worker에서 사용 가능한 API

Web Worker는 메인 스레드와 분리된 환경이므로, 일부 API만 사용할 수 있습니다:

| 사용 가능 | 사용 불가 |
|----------|----------|
| `fetch()` | `document` |
| `XMLHttpRequest` | `window` |
| `WebSocket` | `DOM API` |
| `IndexedDB` | `localStorage` |
| `setTimeout/setInterval` | `alert/confirm` |
| `crypto` | `부모 window 참조` |
| `TextEncoder/Decoder` | |

```javascript
// worker.js - 사용 가능한 API 예시
self.onmessage = async (event) => {
  // fetch 사용 가능
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();

  // crypto 사용 가능
  const hashBuffer = await crypto.subtle.digest(
    'SHA-256',
    new TextEncoder().encode('hello')
  );

  // IndexedDB 사용 가능
  const db = await openDB('myDB', 1);

  self.postMessage({ data, hash: hashBuffer });
};
```

### 1.5 Transferable Objects로 성능 최적화

기본적으로 `postMessage`는 데이터를 **복사**합니다. 대용량 데이터의 경우 복사 비용이 큽니다.

**Transferable Objects**를 사용하면 데이터 소유권을 **전송**하여 복사 비용을 제거합니다:

```javascript
// main.js - 데이터 복사 (느림)
const largeArray = new Float32Array(1000000);
worker.postMessage({ data: largeArray }); // 복사됨
console.log(largeArray.length); // 1000000 (원본 유지)

// main.js - 데이터 전송 (빠름)
const largeBuffer = new ArrayBuffer(4000000); // 4MB
worker.postMessage(largeBuffer, [largeBuffer]); // 소유권 전송
console.log(largeBuffer.byteLength); // 0 (전송됨, 더 이상 접근 불가)
```

**Transferable Objects 종류:**
- `ArrayBuffer`
- `MessagePort`
- `ImageBitmap`
- `OffscreenCanvas`

#### 성능 비교

```javascript
// 벤치마크: 100MB 데이터 전송
const size = 100 * 1024 * 1024; // 100MB

// 복사 방식
console.time('복사');
const copiedData = new Uint8Array(size);
worker.postMessage(copiedData);
console.timeEnd('복사'); // 약 150-300ms

// 전송 방식
console.time('전송');
const transferredData = new Uint8Array(size);
worker.postMessage(transferredData.buffer, [transferredData.buffer]);
console.timeEnd('전송'); // 약 1-5ms
```

### 1.6 실전 예제: 이미지 처리

#### 이미지 블러 처리 Worker

```javascript
// image-worker.js
self.onmessage = (event) => {
  const { imageData, radius } = event.data;
  const blurredData = applyBoxBlur(imageData, radius);

  // ImageData의 버퍼를 전송
  self.postMessage(
    { imageData: blurredData },
    [blurredData.data.buffer]
  );
};

function applyBoxBlur(imageData, radius) {
  const { data, width, height } = imageData;
  const output = new Uint8ClampedArray(data.length);

  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      let r = 0, g = 0, b = 0, a = 0, count = 0;

      // 주변 픽셀 평균 계산
      for (let dy = -radius; dy <= radius; dy++) {
        for (let dx = -radius; dx <= radius; dx++) {
          const nx = Math.min(width - 1, Math.max(0, x + dx));
          const ny = Math.min(height - 1, Math.max(0, y + dy));
          const i = (ny * width + nx) * 4;

          r += data[i];
          g += data[i + 1];
          b += data[i + 2];
          a += data[i + 3];
          count++;
        }
      }

      const i = (y * width + x) * 4;
      output[i] = r / count;
      output[i + 1] = g / count;
      output[i + 2] = b / count;
      output[i + 3] = a / count;
    }
  }

  return new ImageData(output, width, height);
}
```

```tsx
// ImageBlur.tsx
import { useRef, useState, useCallback } from 'react';

export function ImageBlur() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const workerRef = useRef<Worker | null>(null);

  const processImage = useCallback(async (file: File) => {
    const canvas = canvasRef.current;
    if (!canvas) return;

    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    setIsProcessing(true);

    // 이미지 로드
    const img = new Image();
    img.src = URL.createObjectURL(file);
    await img.decode();

    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

    // Worker 생성 및 처리
    if (!workerRef.current) {
      workerRef.current = new Worker(
        new URL('./image-worker.js', import.meta.url)
      );
    }

    workerRef.current.onmessage = (event) => {
      ctx.putImageData(event.data.imageData, 0, 0);
      setIsProcessing(false);
    };

    workerRef.current.postMessage(
      { imageData, radius: 5 },
      [imageData.data.buffer]
    );
  }, []);

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={(e) => {
          const file = e.target.files?.[0];
          if (file) processImage(file);
        }}
        disabled={isProcessing}
      />
      {isProcessing && <p>이미지 처리 중...</p>}
      <canvas ref={canvasRef} />
    </div>
  );
}
```

### 1.7 실전 예제: 대용량 JSON 파싱

```javascript
// json-worker.js
self.onmessage = async (event) => {
  const { url } = event.data;

  try {
    self.postMessage({ status: 'fetching' });

    const response = await fetch(url);
    const text = await response.text();

    self.postMessage({ status: 'parsing' });

    const data = JSON.parse(text);

    self.postMessage({ status: 'processing' });

    // 데이터 가공 (예: 정렬, 필터링)
    const processed = data
      .filter(item => item.active)
      .sort((a, b) => b.score - a.score)
      .slice(0, 100);

    self.postMessage({ status: 'complete', data: processed });
  } catch (error) {
    self.postMessage({ status: 'error', message: error.message });
  }
};
```

```tsx
// DataLoader.tsx
import { useState, useEffect } from 'react';

interface LoadingState {
  status: 'idle' | 'fetching' | 'parsing' | 'processing' | 'complete' | 'error';
  data?: any[];
  message?: string;
}

export function DataLoader({ url }: { url: string }) {
  const [state, setState] = useState<LoadingState>({ status: 'idle' });

  useEffect(() => {
    const worker = new Worker(
      new URL('./json-worker.js', import.meta.url)
    );

    worker.onmessage = (event) => {
      setState(event.data);
    };

    worker.postMessage({ url });

    return () => worker.terminate();
  }, [url]);

  const statusMessages = {
    idle: '대기 중...',
    fetching: '데이터 다운로드 중...',
    parsing: 'JSON 파싱 중...',
    processing: '데이터 처리 중...',
    complete: '완료!',
    error: `에러: ${state.message}`,
  };

  return (
    <div>
      <p>{statusMessages[state.status]}</p>
      {state.status === 'complete' && (
        <ul>
          {state.data?.map((item, index) => (
            <li key={index}>{item.name}: {item.score}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 1.8 Worker Pool 패턴

여러 작업을 병렬로 처리할 때 Worker Pool을 사용합니다:

```javascript
// worker-pool.js
class WorkerPool {
  constructor(workerScript, poolSize = navigator.hardwareConcurrency || 4) {
    this.workers = [];
    this.taskQueue = [];
    this.availableWorkers = [];

    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerScript);
      worker.onmessage = (event) => this.handleWorkerMessage(worker, event);
      this.workers.push(worker);
      this.availableWorkers.push(worker);
    }
  }

  handleWorkerMessage(worker, event) {
    const { resolve, reject } = worker.currentTask;

    if (event.data.error) {
      reject(new Error(event.data.error));
    } else {
      resolve(event.data);
    }

    worker.currentTask = null;
    this.availableWorkers.push(worker);
    this.processNextTask();
  }

  processNextTask() {
    if (this.taskQueue.length === 0 || this.availableWorkers.length === 0) {
      return;
    }

    const task = this.taskQueue.shift();
    const worker = this.availableWorkers.shift();

    worker.currentTask = task;
    worker.postMessage(task.data);
  }

  exec(data) {
    return new Promise((resolve, reject) => {
      this.taskQueue.push({ data, resolve, reject });
      this.processNextTask();
    });
  }

  terminate() {
    this.workers.forEach(worker => worker.terminate());
  }
}

// 사용 예시
const pool = new WorkerPool('./compute-worker.js', 4);

// 여러 작업 병렬 실행
const tasks = Array.from({ length: 100 }, (_, i) => i);
const results = await Promise.all(
  tasks.map(task => pool.exec({ value: task }))
);

pool.terminate();
```

---

## Part 2: Service Workers - 오프라인 지원과 캐싱

### 2.1 Service Workers란?

**Service Worker**는 브라우저와 네트워크 사이에서 동작하는 프록시 서버입니다.

```
브라우저  <-->  Service Worker  <-->  네트워크
                    |
                  캐시
```

**주요 기능:**
- 네트워크 요청 가로채기 및 수정
- 오프라인 지원
- 푸시 알림
- 백그라운드 동기화
- 리소스 캐싱

> Service Worker는 보안을 위해 **HTTPS**에서만 동작합니다 (localhost 제외).
{: .prompt-warning }

### 2.2 Service Worker 라이프사이클

Service Worker는 명확한 라이프사이클을 가집니다:

```
등록(Register) → 설치(Install) → 대기(Waiting) → 활성화(Activate) → 실행(Running)
```

#### 1. 등록 (Register)

```javascript
// main.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/', // 제어 범위
      });

      console.log('SW 등록 성공:', registration.scope);

      // 업데이트 감지
      registration.addEventListener('updatefound', () => {
        const newWorker = registration.installing;
        console.log('새 SW 발견:', newWorker);
      });
    } catch (error) {
      console.error('SW 등록 실패:', error);
    }
  });
}
```

#### 2. 설치 (Install)

```javascript
// sw.js
const CACHE_NAME = 'my-app-v1';
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/styles.css',
  '/app.js',
  '/images/logo.png',
];

self.addEventListener('install', (event) => {
  console.log('SW 설치 중...');

  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('정적 자산 캐싱...');
        return cache.addAll(STATIC_ASSETS);
      })
      .then(() => {
        // 대기 상태 건너뛰기 (선택적)
        return self.skipWaiting();
      })
  );
});
```

#### 3. 활성화 (Activate)

```javascript
// sw.js
self.addEventListener('activate', (event) => {
  console.log('SW 활성화...');

  event.waitUntil(
    // 이전 버전 캐시 삭제
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames
          .filter(name => name !== CACHE_NAME)
          .map(name => {
            console.log('이전 캐시 삭제:', name);
            return caches.delete(name);
          })
      );
    }).then(() => {
      // 모든 클라이언트 즉시 제어
      return self.clients.claim();
    })
  );
});
```

### 2.3 Fetch 이벤트 가로채기

Service Worker의 핵심은 **fetch 이벤트**를 가로채는 것입니다:

```javascript
// sw.js
self.addEventListener('fetch', (event) => {
  const { request } = event;

  // 네비게이션 요청 처리
  if (request.mode === 'navigate') {
    event.respondWith(
      fetch(request).catch(() => caches.match('/offline.html'))
    );
    return;
  }

  // 이미지 요청 처리
  if (request.destination === 'image') {
    event.respondWith(
      caches.match(request).then(cached => {
        return cached || fetch(request).then(response => {
          // 응답 캐싱
          const clone = response.clone();
          caches.open('images-cache').then(cache => {
            cache.put(request, clone);
          });
          return response;
        });
      })
    );
    return;
  }

  // 기본 처리
  event.respondWith(
    caches.match(request).then(cached => {
      return cached || fetch(request);
    })
  );
});
```

### 2.4 캐싱 전략

#### 1. Cache First (캐시 우선)

캐시에서 먼저 찾고, 없으면 네트워크에서 가져옵니다. **정적 자산**에 적합합니다.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      if (cached) {
        return cached;
      }
      return fetch(event.request).then(response => {
        // 응답 캐싱
        const clone = response.clone();
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, clone);
        });
        return response;
      });
    })
  );
});
```

```
요청 → 캐시 확인 → [있음] → 캐시 응답 반환
              ↓
           [없음]
              ↓
        네트워크 요청 → 응답 캐싱 → 응답 반환
```

#### 2. Network First (네트워크 우선)

네트워크에서 먼저 가져오고, 실패하면 캐시를 사용합니다. **API 응답**, **자주 변경되는 데이터**에 적합합니다.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        // 성공 시 캐시 업데이트
        const clone = response.clone();
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, clone);
        });
        return response;
      })
      .catch(() => {
        // 실패 시 캐시 사용
        return caches.match(event.request);
      })
  );
});
```

```
요청 → 네트워크 요청 → [성공] → 캐시 업데이트 → 응답 반환
                  ↓
               [실패]
                  ↓
           캐시 확인 → 캐시 응답 반환
```

#### 3. Stale While Revalidate

캐시된 응답을 먼저 반환하고, 백그라운드에서 네트워크로 캐시를 업데이트합니다. **최신성보다 속도가 중요한 경우**에 적합합니다.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open(CACHE_NAME).then(cache => {
      return cache.match(event.request).then(cached => {
        const fetchPromise = fetch(event.request).then(response => {
          cache.put(event.request, response.clone());
          return response;
        });

        // 캐시된 응답이 있으면 먼저 반환, 없으면 네트워크 대기
        return cached || fetchPromise;
      });
    })
  );
});
```

```
요청 → 캐시 확인 → [있음] → 캐시 응답 즉시 반환
              ↓              ↓ (백그라운드)
           [없음]      네트워크 요청 → 캐시 업데이트
              ↓
        네트워크 요청 대기 → 응답 반환
```

#### 4. Cache Only

네트워크 요청 없이 캐시만 사용합니다. **빌드 시 캐싱된 자산**에 적합합니다.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(caches.match(event.request));
});
```

#### 5. Network Only

캐시를 사용하지 않고 항상 네트워크에서 가져옵니다. **실시간 데이터**, **인증 요청**에 적합합니다.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(fetch(event.request));
});
```

### 2.5 전략별 사용 가이드

```javascript
// sw.js - 요청 유형별 전략 적용
const CACHE_NAME = 'my-app-v1';

self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // 정적 자산: Cache First
  if (isStaticAsset(url)) {
    event.respondWith(cacheFirst(request));
    return;
  }

  // API 요청: Network First
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request));
    return;
  }

  // 이미지: Stale While Revalidate
  if (request.destination === 'image') {
    event.respondWith(staleWhileRevalidate(request));
    return;
  }

  // HTML: Network First with Offline Fallback
  if (request.mode === 'navigate') {
    event.respondWith(
      networkFirst(request).catch(() => caches.match('/offline.html'))
    );
    return;
  }

  // 기본: Cache First
  event.respondWith(cacheFirst(request));
});

function isStaticAsset(url) {
  return /\.(js|css|woff2?|ttf|eot)$/.test(url.pathname);
}

async function cacheFirst(request) {
  const cached = await caches.match(request);
  if (cached) return cached;

  const response = await fetch(request);
  const cache = await caches.open(CACHE_NAME);
  cache.put(request, response.clone());
  return response;
}

async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, response.clone());
    return response;
  } catch (error) {
    return caches.match(request);
  }
}

async function staleWhileRevalidate(request) {
  const cache = await caches.open(CACHE_NAME);
  const cached = await cache.match(request);

  const fetchPromise = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });

  return cached || fetchPromise;
}
```

### 2.6 오프라인 페이지 구현

```html
<!-- offline.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>오프라인</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      margin: 0;
      background: #f5f5f5;
      text-align: center;
      padding: 20px;
    }
    .icon {
      font-size: 64px;
      margin-bottom: 20px;
    }
    h1 {
      color: #333;
      margin-bottom: 10px;
    }
    p {
      color: #666;
      margin-bottom: 20px;
    }
    button {
      padding: 12px 24px;
      font-size: 16px;
      background: #4a90d9;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    button:hover {
      background: #357abd;
    }
  </style>
</head>
<body>
  <div class="icon">&#128268;</div>
  <h1>오프라인 상태입니다</h1>
  <p>인터넷 연결을 확인하고 다시 시도해 주세요.</p>
  <button onclick="location.reload()">다시 시도</button>

  <script>
    // 온라인 상태 감지 시 자동 새로고침
    window.addEventListener('online', () => {
      location.reload();
    });
  </script>
</body>
</html>
```

### 2.7 Push Notifications 기초

```javascript
// main.js - 푸시 알림 권한 요청
async function subscribeToPush() {
  const permission = await Notification.requestPermission();

  if (permission !== 'granted') {
    console.log('알림 권한 거부됨');
    return;
  }

  const registration = await navigator.serviceWorker.ready;

  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
  });

  // 서버에 구독 정보 전송
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription),
  });
}

function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}
```

```javascript
// sw.js - 푸시 이벤트 처리
self.addEventListener('push', (event) => {
  const data = event.data?.json() ?? {
    title: '새 알림',
    body: '새로운 알림이 있습니다.',
  };

  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/images/icon-192.png',
      badge: '/images/badge-72.png',
      data: { url: data.url },
      actions: [
        { action: 'open', title: '열기' },
        { action: 'close', title: '닫기' },
      ],
    })
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'open' || !event.action) {
    const url = event.notification.data?.url || '/';
    event.waitUntil(
      clients.openWindow(url)
    );
  }
});
```

---

## Part 3: 실전 활용

### 3.1 Workbox 라이브러리

Google의 **Workbox**는 Service Worker 개발을 크게 단순화합니다:

```bash
npm install workbox-webpack-plugin
# 또는
npm install workbox-build
```

#### Workbox 기본 설정

```javascript
// sw.js (Workbox 사용)
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import {
  CacheFirst,
  NetworkFirst,
  StaleWhileRevalidate,
} from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';
import { CacheableResponsePlugin } from 'workbox-cacheable-response';

// 빌드 시 생성된 프리캐시 목록 사용
precacheAndRoute(self.__WB_MANIFEST);

// 정적 자산: Cache First
registerRoute(
  ({ request }) =>
    request.destination === 'style' ||
    request.destination === 'script' ||
    request.destination === 'font',
  new CacheFirst({
    cacheName: 'static-assets',
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30일
      }),
    ],
  })
);

// API: Network First
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 5 * 60, // 5분
      }),
    ],
  })
);

// 이미지: Stale While Revalidate
registerRoute(
  ({ request }) => request.destination === 'image',
  new StaleWhileRevalidate({
    cacheName: 'images',
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 7 * 24 * 60 * 60, // 7일
      }),
    ],
  })
);
```

#### Webpack 설정

```javascript
// webpack.config.js
const WorkboxWebpackPlugin = require('workbox-webpack-plugin');

module.exports = {
  // ... 기타 설정
  plugins: [
    new WorkboxWebpackPlugin.GenerateSW({
      clientsClaim: true,
      skipWaiting: true,
      runtimeCaching: [
        {
          urlPattern: /^https:\/\/api\.example\.com\//,
          handler: 'NetworkFirst',
          options: {
            cacheName: 'api-cache',
            expiration: {
              maxEntries: 50,
              maxAgeSeconds: 300,
            },
          },
        },
        {
          urlPattern: /\.(?:png|jpg|jpeg|svg|gif)$/,
          handler: 'StaleWhileRevalidate',
          options: {
            cacheName: 'images',
            expiration: {
              maxEntries: 100,
            },
          },
        },
      ],
    }),
  ],
};
```

#### Vite 설정

> Vite에 대한 자세한 내용은 [Vite 완벽 가이드](/posts/vite-complete-guide/)를 참고하세요.
{: .prompt-tip }

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\//,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-cache',
              expiration: {
                maxEntries: 50,
                maxAgeSeconds: 300,
              },
            },
          },
        ],
      },
      manifest: {
        name: 'My PWA App',
        short_name: 'PWA',
        description: 'My Progressive Web App',
        theme_color: '#4a90d9',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
        ],
      },
    }),
  ],
});
```

### 3.2 React에서 Workers 활용하기

#### Custom Hook: useWorker

```tsx
// hooks/useWorker.ts
import { useEffect, useRef, useState, useCallback } from 'react';

interface WorkerState<T> {
  result: T | null;
  error: Error | null;
  isLoading: boolean;
}

export function useWorker<TInput, TOutput>(
  workerFactory: () => Worker
): [WorkerState<TOutput>, (data: TInput) => void] {
  const workerRef = useRef<Worker | null>(null);
  const [state, setState] = useState<WorkerState<TOutput>>({
    result: null,
    error: null,
    isLoading: false,
  });

  useEffect(() => {
    workerRef.current = workerFactory();

    workerRef.current.onmessage = (event) => {
      setState({
        result: event.data,
        error: null,
        isLoading: false,
      });
    };

    workerRef.current.onerror = (error) => {
      setState({
        result: null,
        error: new Error(error.message),
        isLoading: false,
      });
    };

    return () => {
      workerRef.current?.terminate();
    };
  }, [workerFactory]);

  const postMessage = useCallback((data: TInput) => {
    setState(prev => ({ ...prev, isLoading: true }));
    workerRef.current?.postMessage(data);
  }, []);

  return [state, postMessage];
}
```

```tsx
// 사용 예시
import { useWorker } from './hooks/useWorker';

function HeavyComputation() {
  const [state, compute] = useWorker<number, number[]>(
    () => new Worker(new URL('./prime-worker.js', import.meta.url))
  );

  return (
    <div>
      <button
        onClick={() => compute(100000)}
        disabled={state.isLoading}
      >
        {state.isLoading ? '계산 중...' : '소수 계산하기'}
      </button>

      {state.error && <p>에러: {state.error.message}</p>}

      {state.result && (
        <p>발견된 소수: {state.result.length}개</p>
      )}
    </div>
  );
}
```

#### Comlink을 활용한 Worker 통신

[Comlink](https://github.com/GoogleChromeLabs/comlink)는 Worker 통신을 RPC 스타일로 단순화합니다:

```bash
npm install comlink
```

```javascript
// math-worker.js
import * as Comlink from 'comlink';

const mathFunctions = {
  async calculatePrimes(max) {
    const primes = [];
    for (let i = 2; i <= max; i++) {
      let isPrime = true;
      for (let j = 2; j <= Math.sqrt(i); j++) {
        if (i % j === 0) {
          isPrime = false;
          break;
        }
      }
      if (isPrime) primes.push(i);
    }
    return primes;
  },

  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  },
};

Comlink.expose(mathFunctions);
```

```tsx
// MathWorker.tsx
import { useEffect, useState } from 'react';
import * as Comlink from 'comlink';

interface MathFunctions {
  calculatePrimes(max: number): Promise<number[]>;
  fibonacci(n: number): Promise<number>;
}

export function MathWorker() {
  const [api, setApi] = useState<MathFunctions | null>(null);
  const [primes, setPrimes] = useState<number[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const worker = new Worker(
      new URL('./math-worker.js', import.meta.url)
    );
    const wrapped = Comlink.wrap<MathFunctions>(worker);
    setApi(wrapped);

    return () => worker.terminate();
  }, []);

  const handleCalculate = async () => {
    if (!api) return;

    setIsLoading(true);
    try {
      // 마치 일반 함수 호출처럼 사용
      const result = await api.calculatePrimes(50000);
      setPrimes(result);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div>
      <button onClick={handleCalculate} disabled={isLoading}>
        {isLoading ? '계산 중...' : '소수 계산'}
      </button>
      <p>결과: {primes.length}개의 소수</p>
    </div>
  );
}
```

### 3.3 Next.js에서 Service Worker 설정

> Next.js App Router에 대한 기본 지식은 [Next.js 15 App Router 가이드](/posts/nextjs-15-app-router-guide/)를 참고하세요.
{: .prompt-tip }

```typescript
// next.config.js
const withPWA = require('next-pwa')({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development',
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/api\./,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'api-cache',
        expiration: {
          maxEntries: 32,
          maxAgeSeconds: 24 * 60 * 60,
        },
      },
    },
    {
      urlPattern: /\.(?:png|jpg|jpeg|svg|gif|webp)$/,
      handler: 'CacheFirst',
      options: {
        cacheName: 'image-cache',
        expiration: {
          maxEntries: 64,
          maxAgeSeconds: 30 * 24 * 60 * 60,
        },
      },
    },
  ],
});

module.exports = withPWA({
  // Next.js 설정
});
```

### 3.4 디버깅 팁

#### Chrome DevTools 활용

1. **Application 탭 > Service Workers**
   - 등록된 Service Worker 확인
   - Update on reload 체크로 개발 시 자동 업데이트
   - Offline 체크로 오프라인 테스트

2. **Application 탭 > Cache Storage**
   - 캐시된 리소스 확인
   - 캐시 삭제

3. **Network 탭**
   - "from ServiceWorker" 표시로 SW 응답 확인

```javascript
// sw.js - 디버깅용 로깅
self.addEventListener('fetch', (event) => {
  console.log('[SW] Fetch:', event.request.url);

  // 캐시 히트 여부 로깅
  event.respondWith(
    caches.match(event.request).then(cached => {
      if (cached) {
        console.log('[SW] Cache hit:', event.request.url);
        return cached;
      }
      console.log('[SW] Cache miss:', event.request.url);
      return fetch(event.request);
    })
  );
});
```

#### SW 상태 확인 유틸리티

```typescript
// sw-utils.ts
export async function getSWStatus() {
  if (!('serviceWorker' in navigator)) {
    return { supported: false };
  }

  const registration = await navigator.serviceWorker.getRegistration();

  if (!registration) {
    return { supported: true, registered: false };
  }

  return {
    supported: true,
    registered: true,
    scope: registration.scope,
    installing: !!registration.installing,
    waiting: !!registration.waiting,
    active: !!registration.active,
  };
}

export async function unregisterSW() {
  const registrations = await navigator.serviceWorker.getRegistrations();
  await Promise.all(registrations.map(reg => reg.unregister()));
  console.log('모든 Service Worker 해제됨');
}

export async function clearAllCaches() {
  const cacheNames = await caches.keys();
  await Promise.all(cacheNames.map(name => caches.delete(name)));
  console.log('모든 캐시 삭제됨');
}
```

### 3.5 브라우저 지원 현황

**2024년 기준 지원 현황:**

| 기능 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| Web Workers | 4+ | 3.5+ | 4+ | 12+ |
| Shared Workers | 4+ | 29+ | 16+ | 79+ |
| Service Workers | 40+ | 44+ | 11.1+ | 17+ |
| Push API | 42+ | 44+ | 16+ | 17+ |
| Background Sync | 49+ | X | X | 79+ |

> Web Workers와 Service Workers는 전 세계 브라우저의 **97% 이상**에서 지원됩니다.
{: .prompt-info }

---

## 정리

### Web Workers vs Service Workers

| 특성 | Web Workers | Service Workers |
|------|-------------|-----------------|
| **목적** | 백그라운드 연산 | 네트워크 프록시, 캐싱 |
| **생명주기** | 페이지와 함께 | 페이지와 독립적 |
| **DOM 접근** | 불가 | 불가 |
| **네트워크 가로채기** | 불가 | 가능 |
| **스코프** | 생성한 페이지 | 등록된 경로 전체 |
| **사용 사례** | 이미지 처리, 암호화 | 오프라인 지원, PWA |

### 핵심 포인트

1. **Web Workers**
   - 무거운 연산을 별도 스레드에서 처리
   - `postMessage`로 데이터 통신
   - Transferable Objects로 대용량 데이터 전송 최적화
   - Worker Pool로 병렬 처리

2. **Service Workers**
   - 네트워크 요청 가로채기로 캐싱 제어
   - 캐싱 전략: Cache First, Network First, Stale While Revalidate
   - 오프라인 지원, 푸시 알림, 백그라운드 동기화
   - Workbox로 개발 생산성 향상

3. **실전 적용**
   - React에서 `useWorker` 커스텀 훅 활용
   - Comlink으로 Worker 통신 단순화
   - Next.js에서 `next-pwa`로 간편한 PWA 설정

### 관련 포스트

- [JavaScript Event Loop 완벽 가이드](/posts/javascript-event-loop/) - 싱글 스레드와 비동기 처리
- [웹 성능 최적화 가이드](/posts/web-performance-optimization-guide/) - Core Web Vitals와 최적화 기법
- [Promise와 async/await 완벽 가이드](/posts/javascript-promise-async-await-guide/) - 비동기 프로그래밍

### 참고 자료

- [MDN - Web Workers API](https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API)
- [MDN - Service Worker API](https://developer.mozilla.org/ko/docs/Web/API/Service_Worker_API)
- [Google Workbox](https://developers.google.com/web/tools/workbox)
- [web.dev - Service Workers](https://web.dev/service-workers-cache-storage/)
- [Comlink - Google Chrome Labs](https://github.com/GoogleChromeLabs/comlink)
