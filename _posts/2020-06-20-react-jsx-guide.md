---
title: React JSX 완벽 가이드 - 문법과 스타일링
description: React의 핵심 문법인 JSX의 개념과 사용법을 상세히 알아봅니다. Fragment를 활용한 깔끔한 컴포넌트 구조와 Inline/Import 스타일링 방식을 마스터합니다.
author: changsu
date: 2020-06-20 21:42:32 +0900
categories: [Programming, React]
tags: [react, jsx, fragment, styling, css-modules, inline-style, babel, classnames]
---

## JSX란?

**JSX (JavaScript XML)**는 React에서 사용하는 JavaScript 확장 문법입니다. JavaScript 코드 안에서 HTML과 유사한 마크업을 작성할 수 있게 해줍니다.

### JSX의 변환 과정

JSX 코드는 브라우저가 직접 이해할 수 없으므로, **Babel**을 통해 일반 JavaScript로 변환됩니다.

```jsx
// JSX 코드
const element = <h1 className="greeting">Hello, React!</h1>;

// Babel이 변환한 JavaScript 코드
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, React!'
);
```

### JSX의 장점

#### 1. 가독성과 직관성

```jsx
// JSX 사용 (직관적이고 읽기 쉬움)
function Welcome() {
  return (
    <div className="container">
      <h1>Welcome to React</h1>
      <p>Let's get started!</p>
    </div>
  );
}

// 순수 JavaScript (복잡하고 읽기 어려움)
function Welcome() {
  return React.createElement(
    'div',
    { className: 'container' },
    React.createElement('h1', null, 'Welcome to React'),
    React.createElement('p', null, "Let's get started!")
  );
}
```

#### 2. JavaScript 표현식 사용

중괄호 `{}`를 사용하여 JSX 안에서 JavaScript 표현식을 실행할 수 있습니다.

```jsx
function Greeting() {
  const name = "Alice";
  const isLoggedIn = true;
  const items = ['Apple', 'Banana', 'Cherry'];

  return (
    <div>
      {/* 변수 사용 */}
      <h1>Hello, {name}!</h1>

      {/* 조건부 렌더링 */}
      {isLoggedIn ? <p>Welcome back!</p> : <p>Please log in</p>}

      {/* 배열 렌더링 */}
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>

      {/* 함수 호출 */}
      <p>Current time: {new Date().toLocaleTimeString()}</p>
    </div>
  );
}
```

#### 3. 컴포넌트 기반 구조

```jsx
// 재사용 가능한 컴포넌트
function Button({ text, onClick }) {
  return <button onClick={onClick}>{text}</button>;
}

function App() {
  return (
    <div>
      <Button text="Submit" onClick={() => console.log('Submitted')} />
      <Button text="Cancel" onClick={() => console.log('Cancelled')} />
    </div>
  );
}
```

### JSX 문법 규칙

#### 1. 단일 루트 요소

JSX는 반드시 하나의 루트 요소를 반환해야 합니다.

```jsx
// ❌ 잘못된 예 (여러 루트 요소)
function App() {
  return (
    <h1>Title</h1>
    <p>Content</p>
  );
}

// ✅ 올바른 예 (단일 루트 요소)
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}
```

#### 2. className 사용

HTML의 `class` 대신 `className`을 사용합니다.

```jsx
// ❌ 잘못된 예
<div class="container">Content</div>

// ✅ 올바른 예
<div className="container">Content</div>
```

#### 3. 자체 닫는 태그

내용이 없는 태그는 자체 닫는 형식으로 작성합니다.

```jsx
// HTML 스타일 (JSX에서 사용 가능하지만 권장하지 않음)
<img src="image.jpg"></img>
<input type="text"></input>

// JSX 권장 스타일
<img src="image.jpg" />
<input type="text" />
```

#### 4. camelCase 속성명

HTML 속성명을 camelCase로 작성합니다.

```jsx
<div
  className="box"
  onClick={handleClick}
  onMouseEnter={handleHover}
  tabIndex={0}
  aria-label="Close"  // aria-*, data-* 속성은 예외
  data-id="123"
>
  Content
</div>
```

## Fragment - 불필요한 DOM 노드 제거

**Fragment**는 추가 DOM 노드 없이 여러 자식 요소를 그룹화할 수 있게 해줍니다.

### Fragment가 필요한 이유

```jsx
// ❌ 불필요한 div 래퍼 (DOM 구조 복잡화)
function Table() {
  return (
    <table>
      <tbody>
        <tr>
          <Columns />  {/* Columns 내부에 div가 있으면 HTML 구조 깨짐 */}
        </tr>
      </tbody>
    </table>
  );
}

function Columns() {
  return (
    <div>  {/* ❌ <tr> 안에 <div>는 유효하지 않음 */}
      <td>Column 1</td>
      <td>Column 2</td>
    </div>
  );
}
```

### Fragment 사용법

#### 1. React.Fragment 사용

```jsx
import React, { Component, Fragment } from 'react';

class Columns extends Component {
  render() {
    return (
      <Fragment>
        <td>Column 1</td>
        <td>Column 2</td>
      </Fragment>
    );
  }
}

export default Columns;
```

#### 2. 단축 문법 (빈 태그)

```jsx
function Columns() {
  return (
    <>
      <td>Column 1</td>
      <td>Column 2</td>
    </>
  );
}
```

#### 3. key prop이 필요한 경우

Fragment에 `key`를 전달해야 할 때는 단축 문법을 사용할 수 없습니다.

```jsx
function DescriptionList({ items }) {
  return (
    <dl>
      {items.map(item => (
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}

// ❌ 단축 문법은 key를 받을 수 없음
{items.map(item => (
  <> {/* key를 전달할 수 없음 */}
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </>
))}
```

### Fragment 사용 예시

```jsx
function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  );
}

// 렌더링 결과 (불필요한 래퍼 없음)
<header>...</header>
<main>...</main>
<footer>...</footer>
```

## React 스타일링 방식

React에서는 두 가지 주요 스타일링 방식을 사용합니다.

### 1. Inline Style (인라인 스타일)

JavaScript 객체 형식으로 스타일을 직접 태그에 적용합니다.

#### 기본 사용법

```jsx
function App() {
  const boxStyle = {
    display: 'inline-block',
    width: '100px',
    height: '100px',
    border: '1px solid black',
    backgroundColor: 'orange',  // camelCase 사용
    fontSize: '16px',
    marginTop: '10px'
  };

  return <div style={boxStyle}>Styled Box</div>;
}
```

#### 직접 인라인 스타일

```jsx
function App() {
  return (
    <div
      style={{
        backgroundColor: 'blue',
        color: 'white',
        padding: '20px',
        borderRadius: '8px'
      }}
    >
      Direct Inline Style
    </div>
  );
}
```

#### CSS 속성명 변환 규칙

| CSS | JSX Inline Style |
|-----|------------------|
| `background-color` | `backgroundColor` |
| `font-size` | `fontSize` |
| `margin-top` | `marginTop` |
| `border-radius` | `borderRadius` |

#### 동적 스타일

```jsx
function StatusBadge({ isActive }) {
  const badgeStyle = {
    padding: '5px 10px',
    borderRadius: '4px',
    backgroundColor: isActive ? 'green' : 'gray',
    color: 'white'
  };

  return <span style={badgeStyle}>{isActive ? 'Active' : 'Inactive'}</span>;
}
```

#### Inline Style의 장단점

**장점**:
- 동적 스타일 적용 용이
- 컴포넌트와 스타일이 함께 위치
- 별도 CSS 파일 불필요

**단점**:
- pseudo-class (`:hover`, `:focus`) 사용 불가
- media query 사용 불가
- 자동 prefixing 없음
- 성능상 약간 불리 (매 렌더링마다 객체 생성)

### 2. Import Style (CSS 모듈)

외부 CSS 파일을 import하여 사용하는 방식입니다.

#### CSS 파일 작성

```css
/* App.css */
.test {
  display: inline-block;
  width: 100px;
  height: 100px;
  border: 1px solid black;
  position: fixed;
  top: 50%;
  left: 50%;
}

.anotherTest {
  background-color: blue;
  color: white;
}
```

#### 일반 CSS import

```jsx
import React from 'react';
import './App.css';

function App() {
  return <div className="test">Styled with CSS</div>;
}

export default App;
```

#### CSS Modules 사용

CSS Modules는 클래스명 충돌을 방지하고 스코프를 지역화합니다.

```jsx
import React from 'react';
import styles from './App.module.css';  // .module.css 확장자 사용

function App() {
  return (
    <div className={styles.test}>
      CSS Modules
    </div>
  );
}

export default App;
```

> **참고**: Create React App에서는 CSS Modules가 기본 지원됩니다. `.module.css` 확장자를 사용하면 자동으로 활성화됩니다.

#### 여러 클래스 적용

**방법 1: 배열과 join 사용**

```jsx
function App() {
  return (
    <div className={[styles.test, styles.anotherTest].join(' ')}>
      Multiple Classes
    </div>
  );
}
```

**방법 2: 템플릿 리터럴 사용**

```jsx
function App() {
  return (
    <div className={`${styles.test} ${styles.anotherTest}`}>
      Multiple Classes
    </div>
  );
}
```

**방법 3: classnames 라이브러리 사용** (권장)

```bash
npm install classnames
```

```jsx
import React from 'react';
import classNames from 'classnames';
import styles from './App.css';

function App() {
  const isActive = true;

  return (
    <div className={classNames(styles.test, styles.anotherTest)}>
      With classnames
    </div>
  );
}
```

#### classnames/bind로 간편하게 사용

```jsx
import React, { Component, Fragment } from 'react';
import classNames from 'classnames/bind';
import styles from './App.css';

// styles 객체를 bind
const cx = classNames.bind(styles);

class App extends Component {
  render() {
    const isBlue = true;

    return (
      <Fragment>
        <div className={cx('test', { anotherTest: isBlue })}>
          Conditional Styling
        </div>
      </Fragment>
    );
  }
}

export default App;
```

**조건부 클래스 적용**:

```jsx
const cx = classNames.bind(styles);

function Button({ isPrimary, isLarge, isDisabled }) {
  return (
    <button
      className={cx('button', {
        primary: isPrimary,      // isPrimary가 true면 'primary' 클래스 추가
        large: isLarge,          // isLarge가 true면 'large' 클래스 추가
        disabled: isDisabled     // isDisabled가 true면 'disabled' 클래스 추가
      })}
    >
      Click Me
    </button>
  );
}
```

## 스타일링 방식 비교

| 방식 | 장점 | 단점 | 사용 시기 |
|------|------|------|-----------|
| **Inline Style** | 동적 스타일 쉬움, 간단함 | pseudo-class 불가, media query 불가 | 동적 값이 많을 때 |
| **일반 CSS** | 익숙함, 모든 CSS 기능 사용 | 전역 스코프, 클래스명 충돌 | 간단한 프로젝트 |
| **CSS Modules** | 스코프 지역화, 충돌 방지 | 설정 필요 (CRA는 기본 지원) | 중대형 프로젝트 |
| **Styled Components** | CSS-in-JS, 동적 스타일 | 번들 크기 증가, 러닝 커브 | 컴포넌트 중심 개발 |
| **Tailwind CSS** | 유틸리티 클래스, 빠른 개발 | HTML 복잡, 커스텀 디자인 어려움 | 프로토타입, 빠른 개발 |

## 현대적 스타일링 대안

### Styled Components

```bash
npm install styled-components
```

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;

  &:hover {
    opacity: 0.8;
  }
`;

function App() {
  return (
    <>
      <Button primary>Primary Button</Button>
      <Button>Default Button</Button>
    </>
  );
}
```

### Tailwind CSS

```bash
npm install -D tailwindcss
npx tailwindcss init
```

```jsx
function App() {
  return (
    <div className="flex items-center justify-center h-screen bg-gray-100">
      <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
        Tailwind Button
      </button>
    </div>
  );
}
```

## 모범 사례

### 1. 스타일 분리

```jsx
// ✅ 좋은 예: 스타일 객체를 컴포넌트 밖으로 분리
const styles = {
  container: {
    padding: '20px',
    backgroundColor: '#f0f0f0'
  },
  title: {
    fontSize: '24px',
    fontWeight: 'bold'
  }
};

function App() {
  return (
    <div style={styles.container}>
      <h1 style={styles.title}>Title</h1>
    </div>
  );
}

// ❌ 나쁜 예: 매 렌더링마다 새 객체 생성
function App() {
  return (
    <div style={{ padding: '20px', backgroundColor: '#f0f0f0' }}>
      <h1 style={{ fontSize: '24px', fontWeight: 'bold' }}>Title</h1>
    </div>
  );
}
```

### 2. 조건부 클래스명

```jsx
import classNames from 'classnames';

function Alert({ type, message }) {
  return (
    <div className={classNames('alert', {
      'alert-success': type === 'success',
      'alert-error': type === 'error',
      'alert-warning': type === 'warning'
    })}>
      {message}
    </div>
  );
}
```

### 3. 테마 일관성 유지

```jsx
// theme.js
export const theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    success: '#28a745',
    danger: '#dc3545'
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px'
  }
};

// Component.js
import { theme } from './theme';

function Button() {
  return (
    <button style={{
      backgroundColor: theme.colors.primary,
      padding: theme.spacing.medium
    }}>
      Themed Button
    </button>
  );
}
```

## 참고 자료

- [React 공식 문서 - JSX 소개](https://react.dev/learn/writing-markup-with-jsx)
- [React 공식 문서 - Fragment](https://react.dev/reference/react/Fragment)
- [Babel - Try it out](https://babeljs.io/repl) - JSX 변환 결과 확인
- [classnames - npm](https://www.npmjs.com/package/classnames)
- [CSS Modules 공식 문서](https://github.com/css-modules/css-modules)
- [Styled Components 공식 문서](https://styled-components.com/)
- [Tailwind CSS 공식 문서](https://tailwindcss.com/)

## 마치며

JSX는 React의 핵심 문법으로, HTML과 JavaScript를 자연스럽게 결합하여 직관적인 UI 개발을 가능하게 합니다:

1. **JSX**: JavaScript 안에서 HTML 작성
2. **Fragment**: 불필요한 DOM 노드 제거
3. **Inline Style**: 동적 스타일링
4. **CSS Modules**: 스코프 격리와 재사용성

프로젝트 규모와 팀 선호도에 따라 적절한 스타일링 방식을 선택하고, 일관성 있게 적용하는 것이 중요합니다. 초기에는 간단한 방식으로 시작하여, 필요에 따라 더 강력한 도구로 전환하는 것을 권장합니다.
