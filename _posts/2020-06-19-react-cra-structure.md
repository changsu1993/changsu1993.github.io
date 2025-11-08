---
title: Create React App 프로젝트 구조 완벽 가이드
description: CRA로 생성된 React 프로젝트의 초기 파일 구조와 각 파일의 역할을 상세히 알아봅니다. package.json, node_modules, index.js, App.js 등 핵심 파일들의 관계를 이해합니다.
author: changsu
date: 2020-06-19 16:31:32 +0900
categories: [Programming, React]
tags: [react, cra, package-json, node-modules, project-structure, webpack, babel, 프로젝트구조]
---

## CRA 프로젝트 초기 구조

Create React App으로 프로젝트를 생성하면 다음과 같은 기본 구조가 만들어집니다:

```
my-app/
├── node_modules/          # 설치된 패키지 실제 코드
├── public/                # 정적 리소스
│   ├── index.html        # HTML 템플릿
│   ├── favicon.ico       # 파비콘
│   ├── manifest.json     # PWA 설정
│   └── robots.txt        # 검색엔진 크롤러 설정
├── src/                   # 소스 코드
│   ├── App.js            # 메인 컴포넌트
│   ├── App.css           # App 컴포넌트 스타일
│   ├── App.test.js       # App 컴포넌트 테스트
│   ├── index.js          # 애플리케이션 진입점
│   ├── index.css         # 전역 스타일
│   ├── logo.svg          # React 로고
│   ├── reportWebVitals.js # 성능 측정
│   └── setupTests.js     # 테스트 설정
├── .gitignore            # Git 제외 파일 목록
├── package.json          # 프로젝트 메타데이터 및 의존성
├── package-lock.json     # 의존성 잠금 파일
└── README.md             # 프로젝트 문서
```

## 핵심 파일 그룹 이해하기

### 1. 패키지 관리: node_modules + package.json + .gitignore

이 세 파일은 프로젝트의 의존성 관리 시스템을 구성합니다.

#### package.json

**package.json**은 프로젝트의 메타데이터와 의존성 정보를 담은 설정 파일입니다.

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "devDependencies": {
    "@testing-library/react": "^13.4.0"
  }
}
```

**주요 섹션 설명**:

| 섹션 | 역할 | 예시 |
|------|------|------|
| **name** | 프로젝트 이름 | "my-app" |
| **version** | 프로젝트 버전 | "0.1.0" |
| **dependencies** | 프로덕션 의존성 | react, react-dom |
| **devDependencies** | 개발 의존성 | testing-library |
| **scripts** | npm 스크립트 명령어 | start, build, test |

#### dependencies vs devDependencies

```bash
# 프로덕션 의존성 설치 (배포 환경에서도 필요)
npm install react --save

# 개발 의존성 설치 (개발 환경에서만 필요)
npm install eslint --save-dev
```

**차이점**:
- **dependencies**: 실제 앱 실행에 필요 (React, Axios 등)
- **devDependencies**: 개발/빌드에만 필요 (ESLint, Testing Library 등)

#### package.json의 scripts

```json
{
  "scripts": {
    "start": "react-scripts start",      // 개발 서버 실행
    "build": "react-scripts build",      // 프로덕션 빌드
    "test": "react-scripts test",        // 테스트 실행
    "eject": "react-scripts eject"       // 설정 노출 (비가역)
  }
}
```

**실행 방법**:
```bash
npm start       # 또는 npm run start
npm run build
npm test        # 또는 npm run test
```

#### node_modules

**node_modules**는 설치된 모든 패키지의 실제 코드가 저장되는 디렉토리입니다.

**특징**:
- package.json의 dependencies에 명시된 패키지들이 설치됨
- 각 패키지의 의존성도 함께 설치됨 (중첩 구조)
- 매우 큰 용량 (수백 MB ~ 수 GB)
- Git에 커밋하지 않음 (.gitignore에 포함)

**복원 방법**:
```bash
# package.json을 기반으로 의존성 재설치
npm install

# 또는
npm ci  # 더 빠르고 안정적 (CI/CD 환경 권장)
```

#### .gitignore

**.gitignore**는 Git 버전 관리에서 제외할 파일/디렉토리를 지정합니다.

```gitignore
# 의존성
/node_modules
/.pnp
.pnp.js

# 테스트
/coverage

# 프로덕션 빌드
/build

# 환경 변수
.env.local
.env.development.local
.env.test.local
.env.production.local

# 로그
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# 에디터
.vscode/
.idea/
*.swp
*.swo
```

**node_modules를 Git에 올리지 않는 이유**:
1. **용량**: 수백 MB ~ 수 GB로 저장소 크기 증가
2. **불필요**: package.json만 있으면 `npm install`로 복원 가능
3. **플랫폼 차이**: OS별로 다른 바이너리 필요할 수 있음
4. **충돌**: 여러 개발자가 수정할 이유가 없음

### 2. 라이브러리 설치 워크플로우

#### 새 패키지 설치

```bash
# 방법 1: 자동으로 package.json에 추가 (npm 5+ 기본 동작)
npm install axios

# 방법 2: 명시적으로 --save 사용 (이전 버전 호환)
npm install axios --save

# 개발 의존성으로 설치
npm install eslint --save-dev
```

> **참고**: npm 5 이후 버전에서는 `--save` 플래그 없이도 자동으로 package.json에 추가됩니다.

#### 설치 후 변화

```json
// package.json (자동 업데이트)
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.4.0"  // 새로 추가됨
  }
}
```

```
node_modules/
├── axios/           // 새로 설치된 패키지
├── react/
└── react-dom/
```

#### 버전 관리 기호

```json
{
  "dependencies": {
    "react": "18.2.0",      // 정확한 버전만
    "axios": "^1.4.0",      // 마이너/패치 업데이트 허용
    "lodash": "~4.17.21"    // 패치 업데이트만 허용
  }
}
```

| 기호 | 의미 | 예시 | 허용 범위 |
|------|------|------|-----------|
| **없음** | 정확한 버전 | `1.2.3` | `1.2.3`만 |
| **^** | 마이너 업데이트 | `^1.2.3` | `>=1.2.3 <2.0.0` |
| **~** | 패치 업데이트 | `~1.2.3` | `>=1.2.3 <1.3.0` |

### 3. 코드 진입점: index.html + index.js + App.js

이 세 파일은 React 애플리케이션의 렌더링 흐름을 구성합니다.

#### public/index.html

HTML 템플릿 파일로, React 앱이 마운트될 진입점을 제공합니다.

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>

    <!-- React 앱이 렌더링될 위치 -->
    <div id="root"></div>
  </body>
</html>
```

**핵심 포인트**:
- `<div id="root"></div>`: React 앱의 마운트 포인트
- 빌드 시 번들된 JS/CSS 파일이 자동으로 삽입됨
- `%PUBLIC_URL%`: public 폴더 경로 변수

#### src/index.js

**React 애플리케이션의 진입점(Entry Point)**입니다.

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

// React 18 방식
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// 성능 측정 (선택사항)
reportWebVitals();
```

**주요 구성 요소**:

1. **ReactDOM.createRoot()**
   ```javascript
   // React 18+
   const root = ReactDOM.createRoot(document.getElementById('root'));
   root.render(<App />);

   // React 17 이하 (레거시)
   ReactDOM.render(<App />, document.getElementById('root'));
   ```

2. **render() 메서드의 두 매개변수**
   ```javascript
   root.render(
     <App />,                          // (A) 무엇을 렌더링할지
     // document.getElementById('root') // (B) 어디에 렌더링할지 (createRoot에서 지정)
   );
   ```

3. **StrictMode**
   ```javascript
   <React.StrictMode>
     <App />
   </React.StrictMode>
   ```
   - 개발 모드에서만 작동
   - 잠재적 문제 감지 및 경고
   - 프로덕션 빌드에서는 영향 없음

**실제 사용 예시**:
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import Login from './Pages/Login/Login';  // App 대신 다른 컴포넌트

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Login />);  // Login 컴포넌트를 렌더링
```

**주의사항**:
- `index.js` 파일명을 임의로 변경하면 안 됨
- Webpack 설정에서 진입점으로 지정되어 있음

#### src/App.js

**실제 화면에 표시될 UI 코드를 작성하는 메인 컴포넌트**입니다.

**함수형 컴포넌트 (권장)**:
```javascript
import React from 'react';
import './App.css';

function App() {
  return (
    <div className="App">
      <h1>Hello React!</h1>
      <p>Welcome to my app</p>
    </div>
  );
}

export default App;
```

**클래스형 컴포넌트**:
```javascript
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <h1>Hello React!</h1>
        <p>Welcome to my app</p>
      </div>
    );
  }
}

export default App;
```

**함수형 vs 클래스형 비교**:

| 특징 | 함수형 | 클래스형 |
|------|--------|----------|
| **문법** | 간결함 | 장황함 |
| **상태 관리** | useState Hook | this.state |
| **생명주기** | useEffect Hook | componentDidMount 등 |
| **성능** | 약간 더 가볍고 빠름 | 약간 무거움 |
| **추세** | 현재 표준, 권장됨 | 레거시 코드에서 사용 |

> **권장 사항**: 새 프로젝트에서는 함수형 컴포넌트와 Hooks를 사용하세요.

## 권장 프로젝트 구조

실제 프로젝트에서는 다음과 같이 구조를 확장합니다:

```
src/
├── components/          # 재사용 가능한 공통 컴포넌트
│   ├── Button/
│   │   ├── Button.js
│   │   ├── Button.css
│   │   └── Button.test.js
│   ├── Header/
│   └── Footer/
├── pages/              # 페이지 컴포넌트
│   ├── Home/
│   ├── Login/
│   └── Dashboard/
├── hooks/              # 커스텀 Hooks
│   ├── useAuth.js
│   └── useFetch.js
├── utils/              # 유틸리티 함수
│   ├── api.js
│   └── helpers.js
├── styles/             # 전역 스타일
│   ├── reset.css
│   ├── common.css
│   └── variables.css
├── assets/             # 정적 리소스
│   ├── images/
│   ├── fonts/
│   └── icons/
├── services/           # API 서비스
│   └── authService.js
├── contexts/           # Context API
│   └── AuthContext.js
├── App.js
└── index.js
```

### 컴포넌트 구조 패턴

#### 1. 기능별 그룹핑 (Feature-based)

```
src/
├── features/
│   ├── auth/
│   │   ├── Login.js
│   │   ├── Register.js
│   │   ├── authSlice.js
│   │   └── authAPI.js
│   └── products/
│       ├── ProductList.js
│       ├── ProductDetail.js
│       └── productSlice.js
```

#### 2. 타입별 그룹핑 (Type-based)

```
src/
├── components/
├── containers/
├── reducers/
└── actions/
```

### 파일 명명 규칙

```javascript
// 컴포넌트: PascalCase
Button.js
UserProfile.js
ProductCard.js

// 유틸리티/서비스: camelCase
formatDate.js
apiClient.js
authService.js

// 상수: UPPER_SNAKE_CASE
API_ENDPOINTS.js
CONFIG.js

// Hooks: use 접두사 + camelCase
useAuth.js
useFetch.js
useWindowSize.js
```

## 개발 모드 vs 프로덕션 모드

### 개발 모드 (Development)

```bash
npm start
```

**특징**:
- 소스맵 포함 (디버깅 용이)
- Hot Module Replacement (실시간 리로드)
- 상세한 에러 메시지
- React DevTools 지원
- 최적화 없음 (빠른 빌드)

### 프로덕션 모드 (Production)

```bash
npm run build
```

**특징**:
- 코드 압축 (minification)
- 트리 쉐이킹 (사용하지 않는 코드 제거)
- 해시된 파일명 (캐싱 최적화)
- 환경 변수 주입
- 소스맵 제거 또는 별도 파일

**빌드 결과물**:
```
build/
├── static/
│   ├── css/
│   │   └── main.abc123.css
│   ├── js/
│   │   ├── main.def456.js
│   │   └── runtime.ghi789.js
│   └── media/
├── index.html
└── asset-manifest.json
```

## 환경 변수 관리

### .env 파일 생성

```bash
# .env.local
REACT_APP_API_URL=https://api.example.com
REACT_APP_API_KEY=your-secret-key
REACT_APP_ENVIRONMENT=development
```

### 코드에서 사용

```javascript
const apiUrl = process.env.REACT_APP_API_URL;
const apiKey = process.env.REACT_APP_API_KEY;

console.log(`API URL: ${apiUrl}`);
```

**중요 규칙**:
1. 환경 변수명은 반드시 `REACT_APP_`으로 시작
2. `.env` 파일은 `.gitignore`에 추가
3. 민감한 정보는 서버에서 처리 (클라이언트 노출 위험)

### 환경별 .env 파일

```
.env                # 모든 환경 공통
.env.local          # 로컬 오버라이드 (Git 제외)
.env.development    # 개발 환경
.env.production     # 프로덕션 환경
```

**우선순위** (높음 → 낮음):
1. `.env.development.local` / `.env.production.local`
2. `.env.local`
3. `.env.development` / `.env.production`
4. `.env`

## 모범 사례

### 1. 절대 경로 import

```javascript
// jsconfig.json 또는 tsconfig.json 설정
{
  "compilerOptions": {
    "baseUrl": "src"
  }
}
```

```javascript
// Before: 상대 경로
import Button from '../../../components/Button/Button';

// After: 절대 경로
import Button from 'components/Button/Button';
```

### 2. 컴포넌트별 디렉토리

```
components/
└── Button/
    ├── index.js          # export { default } from './Button'
    ├── Button.js         # 컴포넌트 로직
    ├── Button.css        # 스타일
    ├── Button.test.js    # 테스트
    └── Button.stories.js # Storybook
```

### 3. package.json 스크립트 확장

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false",
    "lint": "eslint src/**/*.js",
    "format": "prettier --write src/**/*.{js,jsx,css,md}"
  }
}
```

## 문제 해결

### 의존성 충돌

```bash
# package-lock.json 삭제 후 재설치
rm -rf node_modules package-lock.json
npm install
```

### 캐시 문제

```bash
# npm 캐시 정리
npm cache clean --force

# 의존성 재설치
rm -rf node_modules
npm install
```

### 빌드 크기 분석

```bash
# 빌드 후 번들 분석
npm run build
npx source-map-explorer 'build/static/js/*.js'
```

## 마치며

Create React App의 프로젝트 구조를 이해하는 것은 React 개발의 기초입니다:

1. **package.json**: 프로젝트 메타데이터와 의존성 관리
2. **node_modules**: 설치된 패키지 실제 코드
3. **.gitignore**: 버전 관리 제외 파일 설정
4. **index.html**: React 마운트 포인트
5. **index.js**: 애플리케이션 진입점
6. **App.js**: 메인 UI 컴포넌트

이 구조를 기반으로 프로젝트 규모에 맞게 디렉토리를 확장하고, 팀의 컨벤션을 정립하여 유지보수성 높은 코드를 작성하세요.
