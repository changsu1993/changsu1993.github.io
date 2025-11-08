---
title: React 입문 가이드 - UI 라이브러리의 혁신
description: React의 기본 개념과 특징을 학습합니다. 웹 애플리케이션, 라이브러리 vs 프레임워크, Component, JSX, Virtual DOM을 다룹니다.
author: changsu
date: 2020-06-19 13:09:12 +0900
categories: [Programming, React]
tags: [react, javascript, jsx, component, virtual-dom, frontend, spa]
---

## 개요

React는 Facebook(현 Meta)에서 개발한 사용자 인터페이스(UI)를 만들기 위한 JavaScript 라이브러리입니다. 컴포넌트 기반 아키텍처, JSX 문법, Virtual DOM을 통해 효율적이고 유지보수하기 쉬운 웹 애플리케이션을 개발할 수 있습니다. 현재 가장 인기 있는 프론트엔드 라이브러리 중 하나로, 많은 기업과 개발자들이 선택하고 있습니다.

## 1. 웹 애플리케이션(Web Application)

### 정의

**웹 애플리케이션**은 웹 브라우저에서 실행되는 응용 소프트웨어로, 인터넷이나 인트라넷을 통해 접근할 수 있습니다.

### 웹사이트 vs 웹 애플리케이션

| 구분 | 웹사이트 | 웹 애플리케이션 |
|------|----------|----------------|
| **목적** | 정보 제공 | 기능 제공 |
| **상호작용** | 제한적 | 높음 |
| **예시** | 블로그, 뉴스 | Gmail, Netflix, Trello |
| **복잡도** | 낮음 | 높음 |

### 초기 웹 vs 현대 웹

**초기 웹 (Client-Server):**

```
클라이언트 PC
├─ 개별 응용 프로그램 설치
├─ 각자의 UI
└─ 업데이트 시 재설치 필요

단점:
- 유지보수 비용 증가
- 생산성 저하
- 호환성 문제
```

**현대 웹 (Web Application):**

```
웹 브라우저
├─ HTML/CSS/JavaScript
├─ 표준 기술 사용
└─ 자동 업데이트

장점:
- 설치 불필요
- 크로스 플랫폼
- 쉬운 유지보수
- 즉시 업데이트
```

### 웹 애플리케이션의 진화

```
1990s: 정적 HTML 페이지
   ↓
2000s: 동적 웹 (PHP, ASP)
   ↓
2010s: AJAX, jQuery
   ↓
현재: SPA (Single Page Application)
   - React, Vue, Angular
   - 네이티브 앱 수준의 UX
```

### SPA (Single Page Application)

```
전통적인 웹:
페이지1 → 서버 요청 → 전체 페이지 새로고침 → 페이지2

SPA:
페이지1 → JavaScript로 부분 업데이트 → 페이지2
- 새로고침 없음
- 빠른 반응속도
- 앱같은 경험
```

## 2. 라이브러리 vs 프레임워크

### 라이브러리 (Library)

**정의**: 특정 기능을 수행하는 도구나 함수들의 집합

```jsx
// 라이브러리 예시: 개발자가 제어
import React from 'react';
import axios from 'axios';
import lodash from 'lodash';

// 내가 원할 때 호출
const data = await axios.get('/api');
const sorted = lodash.sortBy(data);
```

**특징:**
- 필요한 부분만 선택적으로 사용
- 개발자가 흐름을 제어
- 자유도 높음
- 가볍고 유연함

**예시:**
- React
- jQuery
- Lodash
- Axios
- Moment.js

### 프레임워크 (Framework)

**정의**: 애플리케이션의 구조(뼈대)를 제공하는 완성된 틀

```jsx
// 프레임워크 예시: 프레임워크가 제어
// Angular 라우팅
@Component({
  selector: 'app-root',
  template: '<router-outlet></router-outlet>'
})
export class AppComponent {
  // 프레임워크가 정한 방식대로 작성
  constructor(private router: Router) {}
}
```

**특징:**
- 정해진 구조와 규칙
- 프레임워크가 흐름을 제어 (제어의 역전, IoC)
- 일관성 있는 코드
- 학습 곡선 높음

**예시:**
- Angular
- Vue.js (프레임워크 성향)
- Django (Python)
- Spring (Java)
- Next.js (React 프레임워크)

### 핵심 차이: 제어 흐름 (Control Flow)

<div style="max-width: 700px; margin: 30px auto;">
  <!-- 라이브러리 -->
  <div style="margin-bottom: 40px;">
    <div style="text-align: center; margin-bottom: 12px; font-weight: bold; color: #2c3e50; font-size: 16px;">
      라이브러리: "내가 필요할 때 가져다 쓴다"
    </div>
    <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); border-radius: 10px; padding: 25px; box-shadow: 0 4px 8px rgba(52, 152, 219, 0.3);">
      <div style="text-align: center; color: white; font-size: 18px; font-weight: bold; margin-bottom: 15px;">
        내 코드
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px; margin-bottom: 8px;">
        <div style="color: white; font-size: 14px;">📋 함수1()</div>
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px; margin-bottom: 8px; position: relative;">
        <div style="color: white; font-size: 14px;">📚 라이브러리()</div>
        <div style="position: absolute; right: 15px; top: 50%; transform: translateY(-50%); color: #f39c12; font-weight: bold; font-size: 16px;">
          ← 내가 호출
        </div>
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px;">
        <div style="color: white; font-size: 14px;">📋 함수2()</div>
      </div>
    </div>
  </div>

  <!-- 프레임워크 -->
  <div>
    <div style="text-align: center; margin-bottom: 12px; font-weight: bold; color: #2c3e50; font-size: 16px;">
      프레임워크: "프레임워크가 내 코드를 호출한다"
    </div>
    <div style="background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%); border-radius: 10px; padding: 25px; box-shadow: 0 4px 8px rgba(231, 76, 60, 0.3);">
      <div style="text-align: center; color: white; font-size: 18px; font-weight: bold; margin-bottom: 15px;">
        프레임워크
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px; margin-bottom: 8px;">
        <div style="color: white; font-size: 14px;">⚙️ 초기화</div>
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px; margin-bottom: 8px; position: relative;">
        <div style="color: white; font-size: 14px;">👤 내 코드()</div>
        <div style="position: absolute; right: 15px; top: 50%; transform: translateY(-50%); color: #f39c12; font-weight: bold; font-size: 16px;">
          ← 프레임워크가 호출
        </div>
      </div>
      <div style="background: rgba(255, 255, 255, 0.2); border-radius: 6px; padding: 15px;">
        <div style="color: white; font-size: 14px;">🎨 렌더링</div>
      </div>
    </div>
  </div>
</div>

### 비유로 이해하기

```
라이브러리 = 공구함
- 톱, 망치, 드라이버 등
- 필요할 때 꺼내 쓰기
- 사용 방법은 자유

프레임워크 = 자동차
- 엔진, 핸들, 브레이크 등 완성된 구조
- 정해진 방식으로 운전
- 구조를 바꿀 수 없음
```

## 3. Angular vs Vue vs React

### 비교 표

| 특징 | Angular | Vue | React |
|------|---------|-----|-------|
| **개발사** | Google | Evan You | Facebook (Meta) |
| **타입** | 프레임워크 | 프레임워크 | 라이브러리 |
| **언어** | TypeScript | JavaScript | JavaScript |
| **학습 곡선** | 높음 | 보통 | 보통 |
| **크기** | 무거움 | 가벼움 | 가벼움 |
| **생태계** | 포함됨 | 포함됨 | 별도 |
| **인기도** | ⭐️⭐️⭐️ | ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️⭐️⭐️ |

### Angular

```typescript
// Angular 컴포넌트
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <h1>{{ title }}</h1>
    <button (click)="handleClick()">클릭</button>
  `
})
export class AppComponent {
  title = 'Hello Angular';

  handleClick() {
    console.log('Clicked!');
  }
}
```

**특징:**
- Google에서 개발 및 유지보수
- 완전한 프레임워크 (All-in-One)
- TypeScript 필수
- 양방향 데이터 바인딩
- 의존성 주입 (DI)
- 대규모 엔터프라이즈 앱에 적합

**장점:**
- 완성도 높은 구조
- 일관된 개발 방식
- 강력한 CLI

**단점:**
- 무겁고 복잡함
- 높은 진입장벽
- 보일러플레이트 많음

### Vue

```vue
<!-- Vue 컴포넌트 -->
<template>
  <div>
    <h1>{{ title }}</h1>
    <button @click="handleClick">클릭</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: 'Hello Vue'
    }
  },
  methods: {
    handleClick() {
      console.log('Clicked!');
    }
  }
}
</script>
```

**특징:**
- Evan You가 개발
- 프로그레시브 프레임워크
- 배우기 쉬움
- 양방향 데이터 바인딩
- 빠른 성장세
- 중국에서 특히 인기

**장점:**
- 낮은 진입장벽
- 훌륭한 문서
- 유연성

**단점:**
- 상대적으로 작은 생태계
- 대기업 지원 부족
- 중국 외 채용 시장 작음

### React

```jsx
// React 컴포넌트
import { useState } from 'react';

function App() {
  const [title] = useState('Hello React');

  const handleClick = () => {
    console.log('Clicked!');
  };

  return (
    <div>
      <h1>{title}</h1>
      <button onClick={handleClick}>클릭</button>
    </div>
  );
}
```

**특징:**
- Facebook(Meta)에서 개발
- **View만 담당** (UI 라이브러리)
- JSX 문법
- 단방향 데이터 흐름
- Virtual DOM
- 가장 큰 생태계

**장점:**
- 유연성과 자유도
- 방대한 생태계
- 많은 자료와 커뮤니티
- React Native (모바일 앱)

**단점:**
- 별도 라이브러리 조합 필요
- 선택의 어려움
- 빠른 변화

### 생태계 비교

```
Angular: All-in-One
├─ 라우팅: Angular Router
├─ 상태관리: RxJS
├─ HTTP: HttpClient
└─ 폼: Forms Module

Vue: Progressive
├─ 라우팅: Vue Router
├─ 상태관리: Vuex, Pinia
├─ HTTP: Axios (외부)
└─ SSR: Nuxt.js

React: Library + Ecosystem
├─ 라우팅: React Router, Tanstack Router
├─ 상태관리: Redux, MobX, Zustand, Recoil
├─ HTTP: Axios, Fetch API
├─ SSR: Next.js
└─ SSG: Gatsby
```

## 4. React 특징

### 1) Component (컴포넌트)

**정의**: UI를 구성하는 독립적이고 재사용 가능한 블록

```jsx
// 버튼 컴포넌트
function Button({ text, onClick }) {
  return (
    <button onClick={onClick}>
      {text}
    </button>
  );
}

// 재사용
<Button text="확인" onClick={handleConfirm} />
<Button text="취소" onClick={handleCancel} />
<Button text="저장" onClick={handleSave} />
```

**컴포넌트 기반 아키텍처:**

```
App
├─ Header
│  ├─ Logo
│  └─ Navigation
│     ├─ NavItem
│     └─ NavItem
├─ Main
│  ├─ Sidebar
│  │  └─ Widget
│  └─ Content
│     ├─ Article
│     └─ Article
└─ Footer
```

**장점:**
- **재사용성**: 한 번 만들어 여러 곳에서 사용
- **유지보수**: 독립적으로 수정 가능
- **테스트**: 개별 컴포넌트 단위 테스트
- **협업**: 역할 분담 용이

**컴포넌트 예시:**

```jsx
// 카드 컴포넌트
function Card({ title, content, author }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{content}</p>
      <span>작성자: {author}</span>
    </div>
  );
}

// 사용
function App() {
  return (
    <div>
      <Card title="포스트 1" content="내용..." author="Alice" />
      <Card title="포스트 2" content="내용..." author="Bob" />
      <Card title="포스트 3" content="내용..." author="Charlie" />
    </div>
  );
}
```

### 2) JSX (JavaScript XML)

**정의**: JavaScript 확장 문법으로, HTML과 유사한 구조로 UI를 선언

```jsx
// JSX
const element = (
  <div className="greeting">
    <h1>Hello, {name}!</h1>
    <p>Welcome to React</p>
  </div>
);

// 위 코드는 아래로 변환됨
const element = React.createElement(
  'div',
  { className: 'greeting' },
  React.createElement('h1', null, 'Hello, ', name, '!'),
  React.createElement('p', null, 'Welcome to React')
);
```

**JSX 특징:**

```jsx
// 1. JavaScript 표현식 사용
const name = "Alice";
const element = <h1>Hello, {name}!</h1>;

// 2. 속성 지정
const element = <img src={user.avatarUrl} alt={user.name} />;

// 3. 조건부 렌더링
const element = (
  <div>
    {isLoggedIn ? <UserPanel /> : <LoginButton />}
  </div>
);

// 4. 리스트 렌더링
const element = (
  <ul>
    {items.map(item => (
      <li key={item.id}>{item.name}</li>
    ))}
  </ul>
);
```

**Declarative (선언적) 프로그래밍:**

```jsx
// 명령형 (Imperative)
const div = document.createElement('div');
div.className = 'greeting';
const h1 = document.createElement('h1');
h1.textContent = 'Hello, ' + name;
div.appendChild(h1);
document.body.appendChild(div);

// 선언형 (Declarative) - JSX
const element = (
  <div className="greeting">
    <h1>Hello, {name}</h1>
  </div>
);
```

**장점:**
- **직관적**: 결과물을 명확하게 표현
- **예측 가능**: 코드와 결과가 일치
- **유지보수**: 읽기 쉽고 수정 용이
- **협업**: 디자이너도 이해 가능

### 3) Virtual DOM

**정의**: 실제 DOM의 가벼운 복사본으로, 메모리에만 존재

**DOM 조작의 문제:**

```jsx
// DOM 직접 조작 (느림)
document.getElementById('counter').textContent = count;
document.getElementById('list').innerHTML = '<li>Item</li>';
// 브라우저가 레이아웃 재계산, 리페인트 → 성능 저하
```

**Virtual DOM 동작 원리:**

```
1. 상태 변경 발생
   ↓
2. 새로운 Virtual DOM 생성
   ↓
3. 이전 Virtual DOM과 비교 (Diffing)
   ↓
4. 차이점 계산 (Reconciliation)
   ↓
5. 실제 DOM에 최소한의 변경만 적용 (Batch Update)
```

**예시:**

```jsx
// 상태 변경
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>Count: {count}</h1>
      <p>Some other content</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

// Virtual DOM 동작:
// 1. count 변경: 0 → 1
// 2. 전체 컴포넌트 재렌더링
// 3. 실제 변경: <h1>의 텍스트만 업데이트
// 4. <p>, <button>은 그대로 유지
```

**Virtual DOM 장점:**

```
효율성:
- 변경사항 일괄 처리
- 최소한의 DOM 조작
- 불필요한 리페인트 방지

추상화:
- 개발자는 선언적 코드 작성
- 최적화는 React가 담당
- 브라우저 차이 자동 처리

성능:
- 복잡한 UI도 빠른 업데이트
- 많은 데이터 변경도 효율적
```

**Virtual DOM vs Real DOM:**

| 구분 | Real DOM | Virtual DOM |
|------|----------|-------------|
| **위치** | 브라우저 메모리 | JavaScript 객체 |
| **무게** | 무거움 | 가벼움 |
| **업데이트** | 느림 | 빠름 |
| **조작** | 직접 조작 | React가 관리 |

## 5. React 시작하기

### 설치

```bash
# Node.js 설치 확인
node -v
npm -v

# Create React App (CRA)
npx create-react-app my-app
cd my-app
npm start
```

### 기본 구조

```
my-app/
├── public/
│   └── index.html
├── src/
│   ├── App.js        # 메인 컴포넌트
│   ├── index.js      # 진입점
│   └── index.css
├── package.json
└── README.md
```

### 첫 번째 컴포넌트

```jsx
// src/App.js
import React from 'react';

function App() {
  return (
    <div className="App">
      <h1>Hello React!</h1>
      <p>Welcome to React world</p>
    </div>
  );
}

export default App;
```

### 상태 관리 (useState)

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        증가
      </button>
      <button onClick={() => setCount(count - 1)}>
        감소
      </button>
    </div>
  );
}
```

### Props 전달

```jsx
// 부모 컴포넌트
function App() {
  return (
    <div>
      <Greeting name="Alice" />
      <Greeting name="Bob" />
    </div>
  );
}

// 자식 컴포넌트
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

## 6. React 생태계

### 필수 라이브러리

```
라우팅:
├─ React Router: 페이지 네비게이션
└─ Tanstack Router: 타입 안전 라우팅

상태관리:
├─ Redux: 전통적인 상태관리
├─ Zustand: 가벼운 상태관리
├─ Recoil: Facebook 공식
└─ Context API: React 내장

HTTP 통신:
├─ Axios: Promise 기반
├─ Fetch API: 브라우저 내장
└─ Tanstack Query: 서버 상태 관리

스타일링:
├─ styled-components: CSS-in-JS
├─ Emotion: CSS-in-JS
├─ Tailwind CSS: Utility-first
└─ CSS Modules: 모듈화

UI 라이브러리:
├─ Material-UI (MUI)
├─ Ant Design
├─ Chakra UI
└─ shadcn/ui

프레임워크:
├─ Next.js: SSR, SSG
├─ Gatsby: 정적 사이트
└─ Remix: 풀스택
```

## 7. React vs 다른 기술

### jQuery → React

```jsx
// jQuery (명령형)
$('#button').click(function() {
  const count = parseInt($('#counter').text()) + 1;
  $('#counter').text(count);
});

// React (선언형)
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <span id="counter">{count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

### Vanilla JS → React

```jsx
// Vanilla JS
const button = document.createElement('button');
button.textContent = 'Click me';
button.addEventListener('click', () => {
  alert('Clicked!');
});
document.body.appendChild(button);

// React
function App() {
  const handleClick = () => alert('Clicked!');
  return <button onClick={handleClick}>Click me</button>;
}
```

## 8. React를 선택하는 이유

### 장점

```
1. 학습 곡선
   - JavaScript만 알면 시작 가능
   - 점진적 학습

2. 생태계
   - 방대한 라이브러리
   - 풍부한 자료와 튜토리얼

3. 커뮤니티
   - 세계 최대 규모
   - 활발한 오픈소스

4. 채용 시장
   - 가장 많은 채용 공고
   - 높은 연봉

5. 확장성
   - React Native (모바일)
   - React VR (가상현실)
   - Electron (데스크톱)

6. 기업 지원
   - Facebook/Meta 사용
   - Instagram, WhatsApp
   - Netflix, Airbnb
```

### 단점

```
1. 빠른 변화
   - 자주 바뀌는 best practice
   - 학습 부담

2. 선택의 어려움
   - 너무 많은 라이브러리
   - 결정 피로

3. JSX
   - 초기 학습 곡선
   - HTML/CSS 섞임

4. SEO
   - CSR 기본 (검색 최적화 어려움)
   - SSR 필요 시 Next.js 등 추가 학습
```

## 정리

React는 **사용자 인터페이스를 만들기 위한 JavaScript 라이브러리**로, 다음과 같은 핵심 특징을 가지고 있습니다:

### 핵심 특징 요약

**1. 컴포넌트 기반 (Component-Based)**
- 재사용 가능한 독립적인 UI 블록
- 유지보수와 협업에 유리
- 테스트와 디버깅이 쉬움

**2. JSX 문법 (JavaScript XML)**
- JavaScript와 HTML을 결합한 선언적 문법
- 직관적이고 예측 가능한 코드
- 개발 생산성 향상

**3. Virtual DOM**
- 실제 DOM의 가벼운 복사본
- 효율적인 업데이트와 렌더링
- 성능 최적화 자동화

**4. 단방향 데이터 흐름**
- 예측 가능한 상태 관리
- 디버깅 용이
- 명확한 데이터 추적

**5. 풍부한 생태계**
- 방대한 라이브러리와 도구
- 활발한 커뮤니티
- 다양한 학습 자료

### React를 선택해야 하는 이유

```
✅ React가 적합한 경우:
- 대규모 SPA 개발
- 재사용 가능한 컴포넌트가 많은 프로젝트
- 팀 협업이 필요한 프로젝트
- React Native로 모바일 확장 계획
- 활발한 생태계와 커뮤니티 지원이 필요한 경우

⚠️ 다른 선택을 고려할 경우:
- 간단한 웹사이트 (정적 HTML/CSS만으로 충분)
- SEO가 매우 중요한 경우 (Next.js 고려)
- 완전한 프레임워크를 원하는 경우 (Angular, Vue)
```

## 다음 단계: React 학습 로드맵

React를 처음 시작하는 분들을 위한 단계별 학습 가이드입니다. 각 단계를 순서대로 학습하면서 점진적으로 실력을 쌓아가세요.

#### 1단계: 기초 다지기 (1-2주)

**JavaScript ES6+ 필수 문법**
```javascript
// 학습할 내용
- const, let
- 화살표 함수 (Arrow Function)
- 구조 분해 할당 (Destructuring)
- 스프레드 연산자 (Spread Operator)
- 배열 메서드 (map, filter, reduce)
- Promise, async/await
```

**React 핵심 개념**
- 컴포넌트란 무엇인가
- JSX 문법
- Props로 데이터 전달
- State로 상태 관리
- 이벤트 처리

#### 2단계: React Hooks 마스터 (2-3주)

```jsx
// 필수 Hooks
useState     // 상태 관리
useEffect    // 사이드 이펙트 처리
useContext   // 전역 상태 공유
useRef       // DOM 접근 및 값 유지

// 추가 학습
useMemo      // 연산 최적화
useCallback  // 함수 메모이제이션
```

**실습 프로젝트**
- Todo List
- 간단한 카운터
- 날씨 정보 앱

#### 3단계: 실전 개발 도구 (3-4주)

**라우팅**
```jsx
// React Router
- 페이지 네비게이션
- 동적 라우팅
- 중첩 라우팅
- 라우트 가드
```

**HTTP 통신**
```jsx
// Tanstack Query (React Query)
- 데이터 페칭
- 캐싱
- 로딩/에러 상태 관리
- Mutation
```

**스타일링**
- CSS Modules
- styled-components
- Tailwind CSS

#### 4단계: 상태관리 및 최적화 (2-3주)

**전역 상태관리**
```
Context API  → 간단한 전역 상태
Zustand     → 가벼운 프로젝트
Redux       → 대규모 프로젝트
```

**성능 최적화**
- React.memo
- useMemo, useCallback
- 코드 스플리팅
- Lazy Loading

**고급 개념**
- Custom Hooks
- Error Boundaries
- Portal
- Ref 고급 활용

#### 5단계: 프레임워크 및 심화 (4주 이상)

**Next.js (SSR/SSG)**
```
- 서버 사이드 렌더링
- 정적 사이트 생성
- API 라우트
- 이미지 최적화
- SEO 최적화
```

**실전 프로젝트**
- 블로그 플랫폼
- E-commerce 사이트
- 대시보드 애플리케이션

**추가 학습**
- TypeScript with React
- React Native (모바일 앱)
- 테스트 (Jest, React Testing Library)
- CI/CD 배포

### 학습 팁

```
✅ 추천 학습 방법:
1. 공식 문서를 먼저 읽기
2. 작은 프로젝트로 실습
3. 코드를 직접 타이핑하며 학습
4. 에러 메시지 읽는 습관
5. 커뮤니티 활용 (Stack Overflow, GitHub)

❌ 피해야 할 함정:
1. 튜토리얼만 따라하기
2. 모든 라이브러리 한번에 배우기
3. 이론만 공부하고 실습 안하기
4. 에러를 만나면 바로 포기하기
```

## 참고 자료

- [React 공식 문서](https://react.dev/)
- [React 한국어 문서](https://ko.react.dev/)
- [Create React App](https://create-react-app.dev/)
- [React Router](https://reactrouter.com/)
- [Awesome React](https://github.com/enaqx/awesome-react)
- [React Patterns](https://reactpatterns.com/)
