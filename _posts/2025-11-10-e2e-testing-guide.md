---
title: "E2E 테스트 실전 가이드 - Playwright와 Cypress로 사용자 시나리오 테스트하기"
date: 2025-11-10 14:00:00 +0900
categories: [Testing, E2E]
tags: [e2e-testing, playwright, cypress, end-to-end, integration-test, automation, ui-testing, 테스트자동화]
author: changsu
toc: true
comments: true
description: Playwright와 Cypress를 활용한 E2E(End-to-End) 테스트 완벽 가이드. 실제 사용자 시나리오를 자동화하는 방법과 두 도구의 장단점 비교, 실전 예제로 로그인부터 결제까지 테스트하는 방법을 배웁니다. CI/CD 통합과 Best Practices로 안정적인 테스트 자동화를 구축합니다.
---

## 개요

E2E(End-to-End) 테스트는 실제 사용자가 애플리케이션을 사용하는 것처럼 전체 워크플로우를 테스트하는 방법입니다. 이 가이드에서는 현재 가장 인기 있는 두 도구인 Playwright와 Cypress를 활용한 실전 E2E 테스트 작성법을 다룹니다.

**이 글에서 배울 수 있는 것:**
- E2E 테스트의 개념과 필요성
- Playwright vs Cypress 비교 및 선택 가이드
- 실전 E2E 테스트 시나리오 작성
- CI/CD 환경에서의 테스트 자동화
- 안정적인 E2E 테스트 작성 Best Practices

**사전 요구사항:**
- JavaScript/TypeScript 기본 지식
- 웹 애플리케이션 개발 경험
- HTML, CSS, DOM 기본 이해
- 단위 테스트 개념 이해 ([프론트엔드 테스팅 완벽 가이드](/posts/frontend-testing-guide/) 참고)

**예상 소요 시간:** 약 30분

---

## 목차

1. [E2E 테스트란?](#e2e-테스트란)
2. [Playwright vs Cypress](#playwright-vs-cypress)
3. [Playwright 시작하기](#playwright-시작하기)
4. [Cypress 시작하기](#cypress-시작하기)
5. [실전 E2E 테스트 작성](#실전-e2e-테스트-작성)
6. [Best Practices](#best-practices)
7. [CI/CD 통합](#cicd-통합)
8. [자주 묻는 질문 (FAQ)](#자주-묻는-질문-faq)
9. [참고 자료](#참고-자료)

---

## E2E 테스트란?

### 정의

**E2E(End-to-End) 테스트**는 애플리케이션의 시작부터 끝까지 전체 사용자 워크플로우를 테스트하는 방법입니다.

### 테스트 피라미드에서의 위치

<div style="max-width: 500px; margin: 30px auto;">
  <div style="position: relative; height: 300px;">
    <!-- E2E 테스트 (정상) -->
    <div style="position: absolute; bottom: 0; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 250px solid transparent; border-right: 250px solid transparent; border-bottom: 80px solid #e74c3c;">
      <div style="position: absolute; top: 30px; left: -180px; width: 360px; text-align: center; color: white; font-weight: bold; font-size: 16px;">
        E2E 테스트
      </div>
      <div style="position: absolute; top: 50px; left: -180px; width: 360px; text-align: center; color: rgba(255,255,255,0.9); font-size: 13px;">
        느림, 비용 높음, 적은 수
      </div>
    </div>

    <!-- Integration 테스트 (파랑) -->
    <div style="position: absolute; bottom: 80px; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 200px solid transparent; border-right: 200px solid transparent; border-bottom: 100px solid #3498db;">
      <div style="position: absolute; top: 40px; left: -150px; width: 300px; text-align: center; color: white; font-weight: bold; font-size: 16px;">
        통합 테스트
      </div>
      <div style="position: absolute; top: 60px; left: -150px; width: 300px; text-align: center; color: rgba(255,255,255,0.9); font-size: 13px;">
        보통 속도, 보통 수
      </div>
    </div>

    <!-- Unit 테스트 (초록) -->
    <div style="position: absolute; bottom: 180px; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 150px solid transparent; border-right: 150px solid transparent; border-bottom: 120px solid #2ecc71;">
      <div style="position: absolute; top: 50px; left: -120px; width: 240px; text-align: center; color: white; font-weight: bold; font-size: 16px;">
        단위 테스트
      </div>
      <div style="position: absolute; top: 70px; left: -120px; width: 240px; text-align: center; color: rgba(255,255,255,0.9); font-size: 13px;">
        빠름, 비용 낮음, 많은 수
      </div>
    </div>
  </div>
</div>

### E2E 테스트가 검증하는 것

1. **사용자 워크플로우**: 로그인 → 상품 검색 → 장바구니 → 결제
2. **브라우저 호환성**: Chrome, Firefox, Safari, Edge
3. **실제 환경**: 네트워크 지연, 데이터베이스 연동, API 통신
4. **UI/UX**: 버튼 클릭, 폼 입력, 페이지 이동
5. **통합**: 프론트엔드 + 백엔드 + 데이터베이스

### E2E vs 단위 테스트 vs 통합 테스트

| 구분 | 단위 테스트 | 통합 테스트 | E2E 테스트 |
|------|------------|------------|-----------|
| **범위** | 개별 함수/컴포넌트 | 여러 모듈 조합 | 전체 애플리케이션 |
| **속도** | ⚡️ 매우 빠름 | ⚡️ 빠름 | 🐢 느림 |
| **비용** | 💰 낮음 | 💰💰 보통 | 💰💰💰 높음 |
| **신뢰도** | 낮음 | 보통 | 높음 |
| **유지보수** | 쉬움 | 보통 | 어려움 |
| **개수** | 수백~수천 개 | 수십~수백 개 | 수개~수십 개 |

---

## Playwright vs Cypress

현재 가장 인기 있는 두 E2E 테스팅 도구를 비교해봅시다.

### 기본 비교

| 구분 | Playwright | Cypress |
|------|-----------|---------|
| **제작사** | Microsoft | Cypress.io |
| **출시** | 2020년 | 2017년 |
| **언어** | JavaScript, TypeScript, Python, Java, .NET | JavaScript, TypeScript |
| **브라우저** | Chromium, Firefox, WebKit (Safari) | Chrome, Edge, Firefox, Electron |
| **병렬 실행** | ✅ 기본 지원 | ⚠️ 유료 플랜 |
| **속도** | ⚡️⚡️⚡️ 매우 빠름 | ⚡️⚡️ 빠름 |
| **학습 곡선** | 보통 | 쉬움 |
| **디버깅** | 좋음 | 매우 좋음 (Time Travel) |

### Playwright의 장점

✅ **멀티 브라우저 지원**
```javascript
// Chromium, Firefox, WebKit 모두 지원
const { chromium, firefox, webkit } = require('@playwright/test');
```

✅ **병렬 실행 기본 지원**
```bash
# 모든 테스트를 병렬로 실행
npx playwright test --workers=4
```

✅ **자동 대기 (Auto-waiting)**
```javascript
// 요소가 나타날 때까지 자동 대기
await page.click('button');
```

✅ **네트워크 인터셉션**
```javascript
// API 응답 모킹
await page.route('**/api/users', route => {
  route.fulfill({ body: JSON.stringify([...]) });
});
```

✅ **모바일 에뮬레이션**
```javascript
const iPhone13 = devices['iPhone 13'];
const context = await browser.newContext({ ...iPhone13 });
```

### Cypress의 장점

✅ **개발자 경험 (DX)**
- 실시간 리로드
- 타임 트래블 디버깅
- 직관적인 UI

✅ **쉬운 학습 곡선**
```javascript
// 매우 직관적인 API
cy.visit('/login')
cy.get('[data-testid="email"]').type('user@example.com')
cy.get('[data-testid="password"]').type('password')
cy.get('button[type="submit"]').click()
cy.url().should('include', '/dashboard')
```

✅ **강력한 디버깅 도구**
- 각 단계별 스냅샷
- 명령 실행 전후 상태 확인
- 에러 발생 시 자동 스크린샷

✅ **풍부한 플러그인 생태계**
```javascript
// Testing Library 통합
import '@testing-library/cypress/add-commands'

cy.findByRole('button', { name: /submit/i }).click()
```

### 선택 가이드

**Playwright를 선택하세요:**
- ✅ 여러 브라우저 테스트가 필요할 때
- ✅ 빠른 실행 속도가 중요할 때
- ✅ Python, Java, .NET 사용 시
- ✅ 복잡한 네트워크 시나리오 테스트
- ✅ 모바일 웹 테스트

**Cypress를 선택하세요:**
- ✅ 개발자 경험(DX)이 최우선일 때
- ✅ 디버깅을 자주 해야 할 때
- ✅ 팀원들이 테스트에 익숙하지 않을 때
- ✅ Chrome/Firefox만 지원하면 될 때
- ✅ 풍부한 커뮤니티 플러그인 활용

> **추천**: 신규 프로젝트라면 **Playwright**, 이미 Cypress에 익숙하다면 **Cypress** 유지
{: .prompt-tip }

---

## Playwright 시작하기

### 설치 및 설정

```bash
# Playwright 설치
npm init playwright@latest

# 또는 기존 프로젝트에 추가
npm install -D @playwright/test
npx playwright install
```

설치 시 자동으로 생성되는 파일:

**playwright.config.ts:**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

### 첫 번째 Playwright 테스트

**tests/example.spec.ts:**

```typescript
import { test, expect } from '@playwright/test';

test('홈페이지 제목 확인', async ({ page }) => {
  await page.goto('https://playwright.dev/');

  await expect(page).toHaveTitle(/Playwright/);
});

test('Get Started 링크 클릭', async ({ page }) => {
  await page.goto('https://playwright.dev/');

  await page.getByRole('link', { name: 'Get started' }).click();

  await expect(page.getByRole('heading', { name: 'Installation' }))
    .toBeVisible();
});
```

### 테스트 실행

```bash
# 모든 테스트 실행
npx playwright test

# 특정 파일만 실행
npx playwright test tests/example.spec.ts

# 헤드풀 모드 (브라우저 보면서 실행)
npx playwright test --headed

# 디버그 모드
npx playwright test --debug

# UI 모드 (인터랙티브)
npx playwright test --ui
```

---

## Cypress 시작하기

### 설치 및 설정

```bash
# Cypress 설치
npm install -D cypress

# Cypress 열기 (초기 설정)
npx cypress open
```

**cypress.config.js:**

```javascript
const { defineConfig } = require('cypress')

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
    setupNodeEvents(on, config) {
      // 플러그인 이벤트 등록
    },
  },
})
```

### 첫 번째 Cypress 테스트

**cypress/e2e/example.cy.js:**

```javascript
describe('홈페이지 테스트', () => {
  beforeEach(() => {
    cy.visit('/')
  })

  it('페이지 제목을 확인한다', () => {
    cy.title().should('include', 'My App')
  })

  it('로고가 표시된다', () => {
    cy.get('[data-testid="logo"]').should('be.visible')
  })

  it('네비게이션 메뉴가 동작한다', () => {
    cy.get('nav').within(() => {
      cy.contains('About').click()
    })

    cy.url().should('include', '/about')
    cy.get('h1').should('contain', 'About Us')
  })
})
```

### 테스트 실행

```bash
# UI 모드로 실행
npx cypress open

# 헤드리스 모드로 실행
npx cypress run

# 특정 브라우저로 실행
npx cypress run --browser chrome

# 특정 파일만 실행
npx cypress run --spec "cypress/e2e/example.cy.js"
```

---

## 실전 E2E 테스트 작성

실제 이커머스 사이트의 주요 워크플로우를 테스트해봅시다.

### 시나리오 1: 로그인 플로우

#### Playwright 버전

```typescript
import { test, expect } from '@playwright/test';

test.describe('로그인 기능', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('이메일과 비밀번호로 로그인', async ({ page }) => {
    // Arrange: 테스트 데이터 준비
    const email = 'test@example.com';
    const password = 'password123';

    // Act: 로그인 수행
    await page.getByLabel('이메일').fill(email);
    await page.getByLabel('비밀번호').fill(password);
    await page.getByRole('button', { name: '로그인' }).click();

    // Assert: 결과 확인
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('환영합니다')).toBeVisible();
  });

  test('잘못된 비밀번호로 로그인 실패', async ({ page }) => {
    await page.getByLabel('이메일').fill('test@example.com');
    await page.getByLabel('비밀번호').fill('wrongpassword');
    await page.getByRole('button', { name: '로그인' }).click();

    await expect(page.getByText('이메일 또는 비밀번호가 올바르지 않습니다'))
      .toBeVisible();
  });

  test('이메일 유효성 검사', async ({ page }) => {
    await page.getByLabel('이메일').fill('invalid-email');
    await page.getByLabel('비밀번호').fill('password123');
    await page.getByRole('button', { name: '로그인' }).click();

    await expect(page.getByText('올바른 이메일 형식을 입력하세요'))
      .toBeVisible();
  });
});
```

#### Cypress 버전

```javascript
describe('로그인 기능', () => {
  beforeEach(() => {
    cy.visit('/login')
  })

  it('이메일과 비밀번호로 로그인', () => {
    // Arrange
    const email = 'test@example.com'
    const password = 'password123'

    // Act
    cy.get('[data-testid="email-input"]').type(email)
    cy.get('[data-testid="password-input"]').type(password)
    cy.get('[data-testid="login-button"]').click()

    // Assert
    cy.url().should('include', '/dashboard')
    cy.contains('환영합니다').should('be.visible')
  })

  it('잘못된 비밀번호로 로그인 실패', () => {
    cy.get('[data-testid="email-input"]').type('test@example.com')
    cy.get('[data-testid="password-input"]').type('wrongpassword')
    cy.get('[data-testid="login-button"]').click()

    cy.contains('이메일 또는 비밀번호가 올바르지 않습니다')
      .should('be.visible')
  })

  it('이메일 유효성 검사', () => {
    cy.get('[data-testid="email-input"]').type('invalid-email')
    cy.get('[data-testid="password-input"]').type('password123')
    cy.get('[data-testid="login-button"]').click()

    cy.contains('올바른 이메일 형식을 입력하세요')
      .should('be.visible')
  })
})
```

### 시나리오 2: 쇼핑 플로우 (상품 검색 → 장바구니 → 결제)

#### Playwright 버전

```typescript
import { test, expect } from '@playwright/test';

test.describe('쇼핑 플로우', () => {
  test('상품 검색부터 결제까지 전체 프로세스', async ({ page }) => {
    // 1. 로그인
    await page.goto('/login');
    await page.getByLabel('이메일').fill('test@example.com');
    await page.getByLabel('비밀번호').fill('password123');
    await page.getByRole('button', { name: '로그인' }).click();
    await expect(page).toHaveURL('/dashboard');

    // 2. 상품 검색
    await page.getByPlaceholder('상품 검색').fill('노트북');
    await page.getByRole('button', { name: '검색' }).click();

    await expect(page.getByText('검색 결과')).toBeVisible();
    await expect(page.getByTestId('product-item')).toHaveCount(5);

    // 3. 상품 선택 및 상세 페이지
    await page.getByText('맥북 프로 14인치').click();
    await expect(page.getByRole('heading', { name: '맥북 프로 14인치' }))
      .toBeVisible();

    // 4. 장바구니 추가
    await page.getByRole('button', { name: '장바구니에 추가' }).click();
    await expect(page.getByText('장바구니에 추가되었습니다'))
      .toBeVisible();

    // 5. 장바구니 확인
    await page.getByRole('link', { name: '장바구니' }).click();
    await expect(page).toHaveURL('/cart');
    await expect(page.getByText('맥북 프로 14인치')).toBeVisible();

    // 수량 변경
    await page.getByTestId('quantity-increase').click();
    await expect(page.getByTestId('quantity')).toHaveText('2');

    // 총 가격 확인
    const totalPrice = await page.getByTestId('total-price').textContent();
    expect(totalPrice).toContain('5,000,000원');

    // 6. 결제 진행
    await page.getByRole('button', { name: '결제하기' }).click();
    await expect(page).toHaveURL('/checkout');

    // 배송 정보 입력
    await page.getByLabel('받는 사람').fill('홍길동');
    await page.getByLabel('연락처').fill('010-1234-5678');
    await page.getByLabel('주소').fill('서울시 강남구 테헤란로 123');

    // 결제 방법 선택
    await page.getByRole('radio', { name: '신용카드' }).check();

    // 최종 결제
    await page.getByRole('button', { name: '결제 완료' }).click();

    // 7. 주문 완료 확인
    await expect(page).toHaveURL(/\/order\/\d+/);
    await expect(page.getByText('주문이 완료되었습니다')).toBeVisible();

    // 주문 번호 확인
    const orderNumber = await page.getByTestId('order-number').textContent();
    expect(orderNumber).toMatch(/ORD-\d{8}/);
  });
});
```

#### Cypress 버전

```javascript
describe('쇼핑 플로우', () => {
  it('상품 검색부터 결제까지 전체 프로세스', () => {
    // 1. 로그인
    cy.visit('/login')
    cy.get('[data-testid="email-input"]').type('test@example.com')
    cy.get('[data-testid="password-input"]').type('password123')
    cy.get('[data-testid="login-button"]').click()
    cy.url().should('include', '/dashboard')

    // 2. 상품 검색
    cy.get('[data-testid="search-input"]').type('노트북')
    cy.get('[data-testid="search-button"]').click()

    cy.contains('검색 결과').should('be.visible')
    cy.get('[data-testid="product-item"]').should('have.length', 5)

    // 3. 상품 선택
    cy.contains('맥북 프로 14인치').click()
    cy.get('h1').should('contain', '맥북 프로 14인치')

    // 4. 장바구니 추가
    cy.get('[data-testid="add-to-cart"]').click()
    cy.contains('장바구니에 추가되었습니다').should('be.visible')

    // 5. 장바구니 확인
    cy.get('[data-testid="cart-link"]').click()
    cy.url().should('include', '/cart')
    cy.contains('맥북 프로 14인치').should('be.visible')

    // 수량 변경
    cy.get('[data-testid="quantity-increase"]').click()
    cy.get('[data-testid="quantity"]').should('have.text', '2')

    // 총 가격 확인
    cy.get('[data-testid="total-price"]')
      .should('contain', '5,000,000원')

    // 6. 결제 진행
    cy.get('[data-testid="checkout-button"]').click()
    cy.url().should('include', '/checkout')

    // 배송 정보 입력
    cy.get('[data-testid="recipient-name"]').type('홍길동')
    cy.get('[data-testid="phone"]').type('010-1234-5678')
    cy.get('[data-testid="address"]').type('서울시 강남구 테헤란로 123')

    // 결제 방법 선택
    cy.get('[data-testid="payment-card"]').check()

    // 최종 결제
    cy.get('[data-testid="complete-payment"]').click()

    // 7. 주문 완료 확인
    cy.url().should('match', /\/order\/\d+/)
    cy.contains('주문이 완료되었습니다').should('be.visible')

    // 주문 번호 확인
    cy.get('[data-testid="order-number"]')
      .invoke('text')
      .should('match', /ORD-\d{8}/)
  })
})
```

### 시나리오 3: 폼 검증 (회원가입)

#### Playwright 버전

```typescript
import { test, expect } from '@playwright/test';

test.describe('회원가입 폼 검증', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/signup');
  });

  test('모든 필드를 올바르게 입력하면 회원가입 성공', async ({ page }) => {
    await page.getByLabel('이메일').fill('newuser@example.com');
    await page.getByLabel('비밀번호').fill('StrongPass123!');
    await page.getByLabel('비밀번호 확인').fill('StrongPass123!');
    await page.getByLabel('이름').fill('김철수');
    await page.getByLabel('전화번호').fill('010-9876-5432');
    await page.getByRole('checkbox', { name: '이용약관 동의' }).check();
    await page.getByRole('button', { name: '가입하기' }).click();

    await expect(page).toHaveURL('/welcome');
    await expect(page.getByText('회원가입이 완료되었습니다')).toBeVisible();
  });

  test('비밀번호 불일치 시 에러 표시', async ({ page }) => {
    await page.getByLabel('비밀번호').fill('password123');
    await page.getByLabel('비밀번호 확인').fill('password456');
    await page.getByRole('button', { name: '가입하기' }).click();

    await expect(page.getByText('비밀번호가 일치하지 않습니다'))
      .toBeVisible();
  });

  test('약한 비밀번호 경고', async ({ page }) => {
    await page.getByLabel('비밀번호').fill('123');

    await expect(page.getByText('8자 이상, 영문/숫자/특수문자 포함'))
      .toBeVisible();
  });

  test('이미 사용 중인 이메일', async ({ page }) => {
    // API 모킹
    await page.route('**/api/signup', (route) => {
      route.fulfill({
        status: 400,
        body: JSON.stringify({ error: '이미 사용 중인 이메일입니다' })
      });
    });

    await page.getByLabel('이메일').fill('existing@example.com');
    await page.getByLabel('비밀번호').fill('StrongPass123!');
    await page.getByLabel('비밀번호 확인').fill('StrongPass123!');
    await page.getByRole('button', { name: '가입하기' }).click();

    await expect(page.getByText('이미 사용 중인 이메일입니다'))
      .toBeVisible();
  });
});
```

#### Cypress 버전

```javascript
describe('회원가입 폼 검증', () => {
  beforeEach(() => {
    cy.visit('/signup')
  })

  it('모든 필드를 올바르게 입력하면 회원가입 성공', () => {
    cy.get('[data-testid="email"]').type('newuser@example.com')
    cy.get('[data-testid="password"]').type('StrongPass123!')
    cy.get('[data-testid="password-confirm"]').type('StrongPass123!')
    cy.get('[data-testid="name"]').type('김철수')
    cy.get('[data-testid="phone"]').type('010-9876-5432')
    cy.get('[data-testid="terms-checkbox"]').check()
    cy.get('[data-testid="signup-button"]').click()

    cy.url().should('include', '/welcome')
    cy.contains('회원가입이 완료되었습니다').should('be.visible')
  })

  it('비밀번호 불일치 시 에러 표시', () => {
    cy.get('[data-testid="password"]').type('password123')
    cy.get('[data-testid="password-confirm"]').type('password456')
    cy.get('[data-testid="signup-button"]').click()

    cy.contains('비밀번호가 일치하지 않습니다').should('be.visible')
  })

  it('약한 비밀번호 경고', () => {
    cy.get('[data-testid="password"]').type('123')

    cy.contains('8자 이상, 영문/숫자/특수문자 포함')
      .should('be.visible')
  })

  it('이미 사용 중인 이메일', () => {
    // API 인터셉트
    cy.intercept('POST', '**/api/signup', {
      statusCode: 400,
      body: { error: '이미 사용 중인 이메일입니다' }
    }).as('signupRequest')

    cy.get('[data-testid="email"]').type('existing@example.com')
    cy.get('[data-testid="password"]').type('StrongPass123!')
    cy.get('[data-testid="password-confirm"]').type('StrongPass123!')
    cy.get('[data-testid="signup-button"]').click()

    cy.wait('@signupRequest')
    cy.contains('이미 사용 중인 이메일입니다').should('be.visible')
  })
})
```

---

## Best Practices

### 1. 선택자 우선순위

**권장 순서:**

```typescript
// 1. 역할 기반 (가장 권장)
await page.getByRole('button', { name: '로그인' })

// 2. 레이블 (폼 요소)
await page.getByLabel('이메일')

// 3. Placeholder
await page.getByPlaceholder('검색어를 입력하세요')

// 4. 텍스트
await page.getByText('환영합니다')

// 5. Test ID (안정적이지만 HTML 수정 필요)
await page.getByTestId('submit-button')

// ❌ 피해야 할 선택자
await page.locator('.btn-primary') // CSS 변경 시 깨짐
await page.locator('div > button:nth-child(3)') // 구조 변경 시 깨짐
```

### 2. Test ID 활용

HTML에 `data-testid` 속성 추가:

```html
<button data-testid="login-button">로그인</button>
<input data-testid="email-input" type="email" />
<div data-testid="error-message">에러 메시지</div>
```

테스트 코드:

```typescript
// Playwright
await page.getByTestId('login-button').click();

// Cypress
cy.get('[data-testid="login-button"]').click();
```

> **장점**: CSS 클래스나 구조 변경에 영향 받지 않음
{: .prompt-tip }

### 3. 자동 대기 활용

**Playwright는 자동 대기 기본 제공:**

```typescript
// ✅ 좋은 예 - 자동으로 요소가 나타날 때까지 대기
await page.click('button');

// ❌ 나쁜 예 - 명시적 대기 불필요
await page.waitForTimeout(1000);
await page.click('button');
```

**Cypress는 자동 재시도:**

```javascript
// ✅ 좋은 예 - 자동으로 재시도하며 대기
cy.get('button').click()

// ❌ 나쁜 예 - 고정 대기
cy.wait(1000)
cy.get('button').click()
```

### 4. API 모킹

**Playwright:**

```typescript
// API 응답 모킹
await page.route('**/api/users', (route) => {
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([
      { id: 1, name: '홍길동' },
      { id: 2, name: '김철수' }
    ])
  });
});

// 요청 확인
const [request] = await Promise.all([
  page.waitForRequest('**/api/login'),
  page.click('[data-testid="login-button"]')
]);
expect(request.postDataJSON()).toEqual({ email, password });
```

**Cypress:**

```javascript
// API 인터셉트
cy.intercept('GET', '**/api/users', {
  statusCode: 200,
  body: [
    { id: 1, name: '홍길동' },
    { id: 2, name: '김철수' }
  ]
}).as('getUsers')

cy.visit('/users')
cy.wait('@getUsers')

// 요청 검증
cy.wait('@getUsers').its('request.body').should('deep.equal', {
  page: 1,
  limit: 10
})
```

### 5. 페이지 객체 모델 (POM)

**LoginPage.ts:**

```typescript
import { Page } from '@playwright/test';

export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.getByLabel('이메일').fill(email);
    await this.page.getByLabel('비밀번호').fill(password);
    await this.page.getByRole('button', { name: '로그인' }).click();
  }

  async getErrorMessage() {
    return await this.page.getByTestId('error-message').textContent();
  }
}
```

**테스트에서 사용:**

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('로그인 테스트', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('test@example.com', 'password123');

  await expect(page).toHaveURL('/dashboard');
});
```

### 6. 테스트 격리

**각 테스트는 독립적이어야 합니다:**

```typescript
// ❌ 나쁜 예 - 테스트 간 의존성
test('상품 추가', async ({ page }) => {
  // 상품 추가
  await addProduct(page, 'Product A');
});

test('상품 삭제', async ({ page }) => {
  // 이전 테스트의 상품에 의존
  await deleteProduct(page, 'Product A');
});

// ✅ 좋은 예 - 독립적인 테스트
test('상품 추가 및 삭제', async ({ page }) => {
  await addProduct(page, 'Product A');
  await deleteProduct(page, 'Product A');
});

test('상품 추가만 테스트', async ({ page }) => {
  await addProduct(page, 'Product B');
  await expect(page.getByText('Product B')).toBeVisible();
});
```

### 7. 스크린샷과 비디오

**Playwright 설정:**

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
});
```

**수동 스크린샷:**

```typescript
await page.screenshot({ path: 'screenshot.png' });
await page.screenshot({ path: 'fullpage.png', fullPage: true });
```

**Cypress 설정:**

```javascript
// cypress.config.js
module.exports = defineConfig({
  e2e: {
    video: true,
    screenshotOnRunFailure: true,
  },
})
```

### 8. 테스트 데이터 관리

**fixtures 활용:**

```typescript
// tests/fixtures/users.json
{
  "validUser": {
    "email": "test@example.com",
    "password": "password123"
  },
  "adminUser": {
    "email": "admin@example.com",
    "password": "admin123"
  }
}
```

**테스트에서 사용:**

```typescript
import users from './fixtures/users.json';

test('관리자 로그인', async ({ page }) => {
  const { email, password } = users.adminUser;
  await login(page, email, password);
});
```

---

## CI/CD 통합

### GitHub Actions with Playwright

**.github/workflows/playwright.yml:**

```yaml
name: Playwright Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - uses: actions/setup-node@v3
      with:
        node-version: 18

    - name: Install dependencies
      run: npm ci

    - name: Install Playwright Browsers
      run: npx playwright install --with-deps

    - name: Run Playwright tests
      run: npx playwright test

    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

### GitHub Actions with Cypress

**.github/workflows/cypress.yml:**

```yaml
name: Cypress Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'

      - name: Upload screenshots
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots

      - name: Upload videos
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: cypress-videos
          path: cypress/videos
```

### Docker 환경에서 실행

**Dockerfile:**

```dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

**docker-compose.yml:**

```yaml
version: '3.8'
services:
  playwright:
    build: .
    volumes:
      - ./tests:/app/tests
      - ./playwright-report:/app/playwright-report
    environment:
      - CI=true
```

**실행:**

```bash
docker-compose up --abort-on-container-exit
```

---

## 자주 묻는 질문 (FAQ)

### Q1. Playwright와 Cypress 중 어느 것을 선택해야 하나요?

**A:** 프로젝트 요구사항에 따라 다릅니다.

**Playwright 선택:**
- 멀티 브라우저 테스트 필요 (Safari 포함)
- 빠른 실행 속도 중요
- Python/Java/.NET 사용
- 복잡한 네트워크 시나리오

**Cypress 선택:**
- 개발자 경험(DX) 최우선
- 디버깅 자주 필요
- Chrome/Firefox만 지원하면 됨
- 팀원들이 테스트 초보

### Q2. E2E 테스트를 몇 개나 작성해야 하나요?

**A:** 핵심 사용자 워크플로우만 커버하세요.

**권장 개수:**
- 소규모 앱: 5-10개
- 중규모 앱: 10-30개
- 대규모 앱: 30-100개

**우선순위:**
1. ✅ 회원가입/로그인
2. ✅ 핵심 비즈니스 로직 (주문, 결제 등)
3. ✅ 데이터 손실 위험 작업
4. ⚠️ 엣지 케이스 (선택적)

> **70/20/10 법칙**: 단위 테스트 70%, 통합 테스트 20%, E2E 테스트 10%
{: .prompt-info }

### Q3. E2E 테스트가 너무 느려요. 어떻게 하나요?

**A:** 여러 최적화 방법이 있습니다.

1. **병렬 실행**
```bash
# Playwright
npx playwright test --workers=4

# Cypress (유료)
npx cypress run --parallel
```

2. **헤드리스 모드**
```typescript
// playwright.config.ts
use: {
  headless: true, // 기본값
}
```

3. **불필요한 대기 제거**
```typescript
// ❌ 나쁜 예
await page.waitForTimeout(3000);

// ✅ 좋은 예
await page.waitForSelector('[data-testid="content"]');
```

4. **API 모킹 활용**
```typescript
// 실제 API 대신 모킹
await page.route('**/api/**', (route) => {
  route.fulfill({ body: mockData });
});
```

### Q4. 테스트가 자주 실패해요 (Flaky Tests)

**A:** 다음을 확인하세요.

1. **명시적 대기 대신 자동 대기**
```typescript
// ❌ 불안정
await page.waitForTimeout(1000);

// ✅ 안정적
await page.waitForSelector('button');
```

2. **네트워크 안정성**
```typescript
// API 응답 대기
await page.waitForResponse('**/api/users');
```

3. **애니메이션 비활성화**
```typescript
// playwright.config.ts
use: {
  actionTimeout: 10000,
  // 애니메이션 비활성화
  reducedMotion: 'reduce',
}
```

4. **재시도 설정**
```typescript
// playwright.config.ts
retries: process.env.CI ? 2 : 0,
```

### Q5. 로그인 상태를 테스트마다 유지하려면?

**A:** 세션 저장 및 재사용하세요.

**Playwright:**

```typescript
// global-setup.ts
import { chromium } from '@playwright/test';

async function globalSetup() {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  // 로그인
  await page.goto('/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password');
  await page.click('button[type="submit"]');

  // 세션 저장
  await context.storageState({ path: 'auth.json' });
  await browser.close();
}

export default globalSetup;
```

```typescript
// playwright.config.ts
export default defineConfig({
  globalSetup: require.resolve('./global-setup'),
  use: {
    storageState: 'auth.json',
  },
});
```

**Cypress:**

```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', () => {
  cy.session('user-session', () => {
    cy.visit('/login')
    cy.get('[data-testid="email"]').type('test@example.com')
    cy.get('[data-testid="password"]').type('password')
    cy.get('[data-testid="login-button"]').click()
    cy.url().should('include', '/dashboard')
  })
})
```

```javascript
// 테스트에서 사용
beforeEach(() => {
  cy.login()
  cy.visit('/products')
})
```

### Q6. 모바일 브라우저 테스트는 어떻게 하나요?

**A:** 에뮬레이션 기능을 사용하세요.

**Playwright:**

```typescript
import { test, devices } from '@playwright/test';

test.use({ ...devices['iPhone 13'] });

test('모바일 테스트', async ({ page }) => {
  await page.goto('/');
  // iPhone 13 뷰포트로 실행
});
```

**여러 기기 동시 테스트:**

```typescript
// playwright.config.ts
projects: [
  { name: 'Desktop Chrome', use: { ...devices['Desktop Chrome'] } },
  { name: 'iPhone 13', use: { ...devices['iPhone 13'] } },
  { name: 'iPad Pro', use: { ...devices['iPad Pro'] } },
  { name: 'Pixel 5', use: { ...devices['Pixel 5'] } },
],
```

**Cypress:**

```javascript
// cypress.config.js
module.exports = defineConfig({
  e2e: {
    viewportWidth: 375,
    viewportHeight: 667,
  },
})
```

```javascript
// 테스트에서 뷰포트 변경
cy.viewport('iphone-x')
cy.viewport(375, 812)
```

### Q7. CI 환경에서 테스트가 로컬과 다르게 동작해요

**A:** 환경 차이를 최소화하세요.

1. **타임존 통일**
```typescript
// playwright.config.ts
use: {
  timezoneId: 'Asia/Seoul',
  locale: 'ko-KR',
}
```

2. **Docker 사용**
```bash
# 로컬에서도 CI와 동일한 환경
docker run -it --rm \
  -v $(pwd):/work \
  -w /work \
  mcr.microsoft.com/playwright:v1.40.0-jammy \
  npx playwright test
```

3. **환경변수 관리**
```typescript
// .env.test
BASE_URL=http://localhost:3000
API_URL=http://localhost:4000
```

4. **브라우저 버전 고정**
```json
{
  "dependencies": {
    "@playwright/test": "1.40.0"
  }
}
```

---

## 참고 자료

### 공식 문서

- [Playwright 공식 문서](https://playwright.dev/)
- [Cypress 공식 문서](https://docs.cypress.io/)
- [Testing Library](https://testing-library.com/)

### 도구 비교

- [Playwright vs Cypress 비교](https://playwright.dev/docs/why-playwright)
- [E2E 테스팅 도구 벤치마크](https://github.com/cypress-io/cypress-example-recipes)

### 관련 아티클

- [프론트엔드 테스팅 완벽 가이드](/posts/frontend-testing-guide/)
- [TDD 실전 가이드](/posts/tdd-practical-guide/)

### 학습 자료

- [Playwright 튜토리얼](https://playwright.dev/docs/intro)
- [Cypress Real World App](https://github.com/cypress-io/cypress-realworld-app)
- [Test Automation University](https://testautomationu.applitools.com/)

---

## 결론

E2E 테스트는 실제 사용자 경험을 검증하는 가장 확실한 방법입니다. Playwright와 Cypress는 각각 장단점이 있으므로 프로젝트 요구사항에 맞게 선택하세요.

**핵심 요약:**

1. ✅ **선택 기준**: Playwright (멀티 브라우저, 속도) vs Cypress (DX, 디버깅)
2. ✅ **Best Practices**: Test ID 사용, 자동 대기, API 모킹, POM 패턴
3. ✅ **안정성**: 재시도 설정, Flaky 테스트 방지, 환경 통일
4. ✅ **CI/CD**: GitHub Actions, Docker, 병렬 실행
5. ✅ **적정 개수**: 핵심 워크플로우만 (70/20/10 법칙)

E2E 테스트는 초기 설정 비용이 있지만, **배포 전 버그를 잡아내어 사용자에게 안정적인 서비스를 제공**할 수 있습니다. 핵심 사용자 시나리오부터 시작하여 점진적으로 테스트를 확장해나가세요!

---

**다음 단계:**

E2E 테스트를 마스터했다면 다음을 학습해보세요:
- **시각적 회귀 테스트**: Percy, Chromatic
- **성능 테스트**: Lighthouse CI, WebPageTest
- **접근성 테스트**: axe-core, Pa11y
- **부하 테스트**: k6, Artillery

**피드백 환영:**

이 글이 도움이 되셨나요? 댓글로 피드백을 남겨주시면 더 좋은 콘텐츠로 보답하겠습니다! 🚀
