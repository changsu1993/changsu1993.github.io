---
title: WeCode 1차 프로젝트 회고 - Ami Paris 웹사이트 클론
description: React와 Django를 활용한 첫 팀 프로젝트 경험과 배운 점들. 로그인/회원가입부터 API 통신까지 2주간의 개발 여정을 기록합니다.
author: changsu
date: 2020-07-19 23:05:25 +0900
categories: [Projects, Web Development]
tags: [react, django, project, e-commerce, clone-coding, team-project, sass, api, 프로젝트, 회고]
---

WeCode 부트캠프에서 진행한 첫 팀 프로젝트의 경험과 배운 점들을 기록합니다.

## 프로젝트 소개

미니멀한 디자인과 고유의 유니크한 매력을 담아내는 **Ami Paris** 웹사이트를 클론한 프로젝트입니다.

### 프로젝트 개요

- **프로젝트 기간**: 2020년 6월 22일 ~ 7월 3일 (총 2주)
- **팀 구성**: Front-End 3명, Back-End 3명
- **프로젝트 저장소**
  - Front-End: [9-WE_T_S-frontend](https://github.com/wecode-bootcamp-korea/9-WE_T_S-frontend)
  - Back-End: [9-WE_T_S-backend](https://github.com/wecode-bootcamp-korea/9-WE_T_S-backend)

### 프로젝트 데모

[프로젝트 시연 영상 보기](https://play-tv.kakao.com/embed/player/cliplink/416899477)

## 기술 스택

### Front-End

| 카테고리 | 기술/도구 |
|---------|----------|
| **Core** | HTML, React.js, JavaScript (ES6) |
| **Styling** | SASS |
| **Libraries** | slick-slider (이미지 슬라이더)<br>zxcvbn (비밀번호 강도 측정)<br>React Icons |
| **Collaboration** | Git/GitHub, Slack, Trello |

### Back-End

| 카테고리 | 기술/도구 |
|---------|----------|
| **Language** | Python (List comprehension) |
| **Framework** | Django (select_related, prefetch_related) |
| **Authentication** | Email validation, Password validation |
| **API Documentation** | Postman |
| **Deployment** | AWS |

## 담당 페이지 및 구현 기능

### 내가 구현한 기능 (로그인/회원가입/Newsletter/Search)

#### 1. 로그인 & 회원가입

- **비밀번호 표시/숨김 토글 기능**
  - 사용자가 입력한 비밀번호를 확인할 수 있도록 눈 아이콘 구현

- **정규식 기반 유효성 검사**
  - 이메일과 비밀번호 형식 검증
  - 실시간 피드백 제공

- **비밀번호 강도 표시 애니메이션**
  - zxcvbn 라이브러리를 활용한 보안 강도 컬러바
  - 비밀번호 복잡도에 따른 시각적 피드백

- **Back-End API 통신**
  - Fetch API를 통한 회원가입/로그인 처리
  - Access Token 발급 및 저장
  - 로그인 성공 시 Nav 바에 'hello' 표시

#### 2. Modal 기반 기능

- **재사용 가능한 Modal 컴포넌트**
  - Newsletter 구독 팝업
  - 제품 검색(Search) 인터페이스
  - 컴포넌트 재사용성을 고려한 설계

### 팀원들이 구현한 기능

#### 메인/장바구니/위시리스트

- 반응형 Navigation Bar (스크롤 이벤트, 페이지별 반응)
- 서버 통신을 통한 메뉴 카테고리 동적 렌더링
- 이미지 슬라이더 구현
- reduce 함수를 활용한 제품 수량 관리
- 팝업형 장바구니/위시리스트
- 사용자별 위시리스트 서버 연동

#### 제품 리스트/상세/로딩 페이지

- 색상별 필터링 기능
- 가격순 정렬 (sort 함수)
- 반응형 hover 애니메이션
- 상품 상세 스크롤 이벤트
- 사이즈 옵션 선택
- 이미지 확대 Modal
- 서버 통신을 통한 제품 데이터 렌더링
- Fetch 전 로딩 페이지 표시

## 프로젝트를 통해 배운 점

### 긍정적인 경험

#### 1. 실전 개발 경험
실제 웹사이트를 학습한 내용으로 구현하면서 개발자로서의 자신감을 얻었습니다. 단순히 튜토리얼을 따라하는 것이 아닌, 실제 서비스를 만드는 경험이 큰 동기부여가 되었습니다.

#### 2. 협업 방법론 학습
실제 회사에서 사용하는 **Scrum 방식**을 경험했습니다. 매일 스탠드업 미팅을 통해 진행 상황을 공유하고, 막히는 부분을 함께 해결해나가는 과정이 인상적이었습니다.

#### 3. Git/GitHub를 통한 협업
- 브랜치 전략과 Pull Request 워크플로우 학습
- 코드 리뷰와 Merge Conflict 해결 경험
- 각자의 코드가 하나의 완성된 서비스로 합쳐지는 과정의 뿌듯함

#### 4. 라이브러리 활용 경험
회원가입 시 비밀번호 보안강도 표시를 위해 **zxcvbn 라이브러리**를 처음 사용했습니다. 외부 라이브러리를 프로젝트에 통합하고 문서를 읽는 경험이 유익했습니다.

#### 5. Front-End와 Back-End 통신
- Fetch API를 활용한 HTTP 요청
- REST API 엔드포인트 이해
- 요청/응답 데이터 구조 설계
- Access Token 기반 인증 구현

#### 6. 컴포넌트 재사용성
재사용 가능한 Modal 컴포넌트를 만들어 Newsletter와 Search 페이지에 적용하면서, 컴포넌트 설계의 중요성을 배웠습니다.

## 개선이 필요했던 부분

### 프로젝트 관리

1. **트렐로 관리 소홀**
   - 초반에는 체계적으로 관리했으나 후반부에 업데이트가 누락됨
   - 작업 현황 파악이 어려워짐

2. **코드 컨벤션 부재**
   - 전체적인 컨벤션을 처음부터 정하지 않아 코드 스타일이 일관되지 않음
   - 변수명, 파일 구조 등의 규칙이 명확하지 않았음

### 기술적 이해

3. **라이브러리 이해 부족**
   - Modal 구현과 zxcvbn 사용 시 정확한 동작 원리를 이해하지 못함
   - "일단 작동하게 만들기"에 급급했던 점

4. **코드 품질**
   - 마감 기한에 쫓겨 변수명을 대충 지음
   - 전체적으로 코드가 깔끔하지 못하고 정리되지 않음

### 팀워크

5. **팀원 지원 부족**
   - 팀원들이 어려움을 겪을 때 충분한 도움을 주지 못함
   - 각자의 작업에만 집중하는 경향

## 다음 프로젝트를 위한 개선 방안

### 계획 단계

```markdown
1. 구현 기능 상세 분석
   - 각 기능별 우선순위 설정
   - 예상 소요 시간 산정
   - 기술 스택 조사 및 학습 계획

2. 팀 코드 컨벤션 수립
   - 변수/함수 네이밍 규칙
   - 파일/폴더 구조
   - Git commit 메시지 규칙
   - 코드 포맷팅 (Prettier, ESLint)
```

### 개발 단계

```markdown
3. 우선순위 기반 작업
   - 핵심 기능 먼저 구현
   - 세부 기능은 나중에 추가
   - MVP(Minimum Viable Product) 접근

4. 의미 있는 변수명 사용
   - 코드를 읽는 사람을 배려
   - 주석 없이도 이해 가능한 코드 작성
```

### 협업 단계

```markdown
5. 매일 기록하기
   - 트렐로 카드 업데이트
   - 학습 내용 문서화
   - 발생한 이슈와 해결 방법 정리
```

## 기억하고 싶은 코드

### 1. 정규식을 활용한 유효성 검사

조건문이 복잡해지는 것을 방지하기 위해 정규식 검증 로직을 별도 함수로 분리했습니다.

```jsx
// 정규식 함수 정의
const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

const validatePassword = (password) => {
  // 최소 8자, 영문, 숫자, 특수문자 포함
  const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/;
  return passwordRegex.test(password);
};

// 조건문에서 깔끔하게 사용
if (!validateEmail(email)) {
  alert('올바른 이메일 형식이 아닙니다.');
  return;
}

if (!validatePassword(password)) {
  alert('비밀번호는 8자 이상, 영문, 숫자, 특수문자를 포함해야 합니다.');
  return;
}
```

**개선 포인트**: 복잡한 정규식을 함수로 캡슐화하여 가독성과 재사용성을 높였습니다.

### 2. 재사용 가능한 Modal 컴포넌트

Newsletter와 Search 페이지에서 동일한 Modal을 재사용할 수 있도록 설계했습니다.

```jsx
// Nav.js (팀원이 구현한 Navigation Bar)
import Newsletter from './Newsletter';
import Search from './Search';

const Nav = () => {
  const [isNewsletterOpen, setIsNewsletterOpen] = useState(false);
  const [isSearchOpen, setIsSearchOpen] = useState(false);

  return (
    <nav>
      <button onClick={() => setIsNewsletterOpen(true)}>
        Newsletter
      </button>
      <button onClick={() => setIsSearchOpen(true)}>
        Search
      </button>

      {/* 재사용 가능한 Modal 컴포넌트 */}
      {isNewsletterOpen && (
        <Newsletter onClose={() => setIsNewsletterOpen(false)} />
      )}
      {isSearchOpen && (
        <Search onClose={() => setIsSearchOpen(false)} />
      )}
    </nav>
  );
};
```

**개선 포인트**: Props를 통해 열림/닫힘 상태를 제어하여 컴포넌트 재사용성을 확보했습니다.

## 마치며

이번 프로젝트는 제게 첫 팀 개발 경험이었습니다. 내가 구현한 기능이 완성될 때마다 팀원들이 박수를 쳐주며 칭찬해줬는데, 그것이 큰 힘이 되었습니다.

팀원들의 따뜻한 배려 덕분에 스트레스보다는 즐거움이 더 큰 프로젝트였습니다. 부족한 점도 많았지만, 이를 통해 다음 프로젝트에서는 더 나은 개발자가 될 수 있을 것이라 확신합니다.

## 참고 자료

- [React 공식 문서](https://react.dev/)
- [JavaScript 정규 표현식 가이드](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Regular_expressions)
- [zxcvbn 라이브러리](https://github.com/dropbox/zxcvbn)
- [React Slick - 캐러셀 라이브러리](https://react-slick.neostack.com/)
- [Scrum 방법론 이해하기](https://www.scrum.org/resources/what-is-scrum)
