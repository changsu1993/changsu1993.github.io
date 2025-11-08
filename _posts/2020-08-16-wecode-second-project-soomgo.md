---
title: WeCode 2차 프로젝트 회고 - Soomgo 웹사이트 클론
description: React Hooks, Styled-Components, Kakao API를 활용한 두 번째 팀 프로젝트. 1차 프로젝트에서의 학습을 바탕으로 더 발전된 기술 스택과 협업 경험을 쌓았습니다.
author: changsu
date: 2020-08-16 19:21:17 +0900
categories: [Projects, Web Development]
tags: [react, django, project, soomgo, clone-coding, hooks, styled-components, kakao-api, axios, 프로젝트, 회고]
image:
  path: /assets/img/posts/2020-08-16-soomgo-project.png
  alt: Soomgo 클론 프로젝트
---

WeCode 부트캠프 2차 프로젝트에서 1차 프로젝트의 경험을 바탕으로 더 발전된 기술 스택을 활용하여 진행한 프로젝트 회고입니다.

## 프로젝트 소개

'숨은 고수'를 찾아 일반인과 이어주는 생활서비스 전문가 매칭 플랫폼 **Soomgo(숨고)** 웹사이트를 클론한 프로젝트입니다.

### 프로젝트 개요

- **프로젝트 기간**: 2020년 7월 6일 ~ 7월 17일 (총 2주)
- **팀 구성**: Front-End 3명, Back-End 3명
- **프로젝트 저장소**
  - Front-End: [9-Chango-frontend](https://github.com/wecode-bootcamp-korea/9-Chango-frontend)
  - Back-End: [9-Chango-backend](https://github.com/wecode-bootcamp-korea/9-Chango-backend)

### 프로젝트 데모

[프로젝트 시연 영상 보기](https://play-tv.kakao.com/embed/player/cliplink/416899592)

## 기술 스택

### Front-End

| 카테고리 | 기술/도구 |
|---------|----------|
| **Core** | HTML, React.js, JavaScript (ES6) |
| **React 패턴** | Class 컴포넌트, Functional 컴포넌트 (Hooks) |
| **Styling** | SASS, Styled-Components |
| **HTTP Client** | Axios |
| **External APIs** | Kakao Map API, Kakao Login API |
| **Collaboration** | Git/GitHub (git rebase), Slack, Trello |

> **1차 프로젝트와의 차이점**: React Hooks, Styled-Components, Axios, 외부 API 연동 추가

### Back-End

| 카테고리 | 기술/도구 |
|---------|----------|
| **Language** | Python |
| **Framework** | Django |
| **Database** | MySQL |
| **Security** | CORS 헤더 설정 |
| **DevOps** | AWS, Docker |
| **API Design** | RESTful API |
| **Version Control** | Git, GitHub, Git Rebase |

## 담당 페이지 및 구현 기능

### 내가 구현한 기능 (Footer/고수 찾기/고수 디테일)

#### 1. Footer 컴포넌트

- **함수형 컴포넌트 + Styled-Components**
  - 1차 프로젝트에서 사용하지 못한 기술 스택 적용
  - CSS-in-JS 패턴으로 스타일 관리

#### 2. 고수 찾기 페이지

- **Class 컴포넌트 + SASS**
  - 전문가 목록 표시
  - Fetch API를 통한 Back-End 통신
  - Map 함수를 활용한 데이터 렌더링
  - Modal 창 구현 (필터 옵션 선택)

- **Mock Data 활용**
  - Back-End에서 제공하지 못한 일부 데이터를 Mock Data로 대체
  - 프론트엔드 개발 독립성 확보

#### 3. 고수 디테일 페이지

- **Class 컴포넌트 + SASS**
  - 전문가 상세 정보 표시
  - 리뷰 데이터 렌더링
  - 서버 통신과 Mock Data 혼합 사용

### 팀원들이 구현한 기능

#### 로그인/회원가입/고수 프로필

**인증 및 소셜 로그인**
- 정규식을 활용한 이메일 형식 검증
- 이메일 및 비밀번호 중복 확인
- **Kakao 소셜 로그인** 구현
- JWT 토큰을 localStorage에 저장
- React Hooks와 Styled-Components 사용

**고수 프로필 관리**
- 로컬에서 프로필 이미지 업로드
- 고수 상세 정보 입력 및 저장
- Daum Postcode API로 우편번호 검색
- **Kakao Map API** 연동
  - Geocoder로 주소→좌표 변환
  - Circle로 활동 지역 표시
  - useEffect Hook으로 지도 초기화

#### 메인 카테고리/설문 페이지

**카테고리 페이지**
- Query Parameters를 활용한 카테고리별 페이지네이션
- 점수 기반 별점 표시 함수
- Map 함수로 인기 서비스/전체 서비스 렌더링
- Class 컴포넌트와 Fetch API 사용

**설문 페이지**
- Switch-case문으로 질문별 설문 Form 구현
- React DatePicker로 날짜 선택
- React Hooks와 Styled-Components
- Axios로 Back-End API 통신

## 1차 프로젝트 대비 성장한 부분

### 기술적 성장

#### 1. 새로운 기술 스택 도입

```javascript
// 1차: Class 컴포넌트만 사용
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  // ...
}

// 2차: Hooks를 활용한 함수형 컴포넌트
import { useState, useEffect } from 'react';

const MyComponent = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // 사이드 이펙트 처리
  }, []);
  // ...
};
```

#### 2. Styled-Components 도입

```javascript
// 1차: 별도 SASS 파일
import './Footer.scss';

const Footer = () => <footer className="footer">...</footer>;

// 2차: CSS-in-JS 패턴
import styled from 'styled-components';

const FooterContainer = styled.footer`
  background-color: #f8f9fa;
  padding: 2rem;

  @media (max-width: 768px) {
    padding: 1rem;
  }
`;

const Footer = () => <FooterContainer>...</FooterContainer>;
```

**장점**:
- 컴포넌트와 스타일의 응집도 향상
- Props 기반 동적 스타일링
- CSS 클래스명 충돌 방지
- TypeScript 지원 우수

#### 3. Axios vs Fetch API 비교

```javascript
// Fetch API (1차 프로젝트)
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => setData(data))
  .catch(error => console.error(error));

// Axios (2차 프로젝트)
import axios from 'axios';

axios.get('https://api.example.com/data')
  .then(response => setData(response.data))
  .catch(error => console.error(error));
```

**Axios의 장점**:
- 자동 JSON 변환
- 요청/응답 인터셉터
- 취소 가능한 요청
- 더 나은 에러 핸들링

### 협업 능력 향상

#### 팀원 지원

1차 프로젝트에서는 내 작업만으로도 벅찼지만, 2차에서는 여유를 가지고 팀원을 도울 수 있었습니다:

- 로그인/회원가입 구현 팀원에게 정규식 표현 도움
- 경고 문구 UI/UX 개선 제안
- Modal 창 구현 가이드 제공

#### Git Rebase 경험

```bash
# Merge vs Rebase의 차이 학습
# Merge: 병합 커밋 생성
git merge feature-branch

# Rebase: 커밋 히스토리를 선형으로 정리
git rebase main
```

> 💡 **학습 노트**: Git Rebase를 사용했지만 완전히 이해하지 못한 채 사용했습니다. 추후 Git 전반을 다시 학습할 필요성을 느꼈습니다.

## 기억하고 싶은 코드

### 1. Mock Data 생성 (처음 시도)

Back-End에서 시간 내에 제공하지 못한 리뷰 데이터를 Mock Data로 생성했습니다.

```javascript
// reviewMockData.js
export const REVIEW_MOCK_DATA = [
  {
    id: 1,
    user_name: "김철수",
    rating: 5,
    content: "정말 친절하고 꼼꼼하게 작업해주셨습니다.",
    created_at: "2020-07-10",
    service_images: [
      "https://example.com/review1.jpg"
    ]
  },
  {
    id: 2,
    user_name: "이영희",
    rating: 4,
    content: "가격 대비 만족스러운 서비스였어요.",
    created_at: "2020-07-08",
    service_images: []
  },
  // ... 더 많은 리뷰 데이터
];
```

**배운 점**:
- Front-End 개발이 Back-End API에 의존하지 않도록 Mock Data 활용
- 실제 데이터 구조를 미리 설계하여 통합 시간 단축
- 프론트엔드와 백엔드의 병렬 개발 가능

### 2. Map 함수를 활용한 데이터 렌더링

서버에서 받은 데이터를 Map 함수로 효율적으로 렌더링했습니다.

```javascript
// ProList.js (고수 찾기 페이지)
class ProList extends Component {
  state = {
    proList: [],
    isModalOpen: false
  };

  componentDidMount() {
    fetch('https://api.example.com/professionals')
      .then(res => res.json())
      .then(data => {
        this.setState({ proList: data.results });
      });
  }

  toggleModal = () => {
    this.setState(prevState => ({
      isModalOpen: !prevState.isModalOpen
    }));
  };

  render() {
    const { proList, isModalOpen } = this.state;

    return (
      <div className="pro-list">
        <button onClick={this.toggleModal}>
          필터
        </button>

        {/* Modal 컴포넌트 */}
        {isModalOpen && (
          <FilterModal onClose={this.toggleModal} />
        )}

        {/* 전문가 목록 렌더링 */}
        <div className="pro-grid">
          {proList.map(pro => (
            <ProCard
              key={pro.id}
              name={pro.name}
              rating={pro.rating}
              reviewCount={pro.review_count}
              image={pro.profile_image}
              onClick={() => this.handleProClick(pro.id)}
            />
          ))}
        </div>
      </div>
    );
  }
}
```

**신세계였던 경험**:
> "Back-End와 통신하며 데이터를 Map으로 저 코드 하나만으로 엄청난 양의 데이터를 페이지에 반복해서 구현된다는 것이 신세계였다."

처음 Map 함수의 강력함을 체감한 순간이었고, 지금도 사용할 때마다 그 편리함에 감탄합니다.

### 3. Styled-Components로 Footer 구현

```javascript
// Footer.js
import React from 'react';
import styled from 'styled-components';

const FooterContainer = styled.footer`
  background-color: #f8f9fa;
  padding: 3rem 1rem;
  margin-top: 4rem;
`;

const FooterContent = styled.div`
  max-width: 1200px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2rem;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
`;

const FooterSection = styled.div`
  h4 {
    font-size: 1rem;
    font-weight: 600;
    margin-bottom: 1rem;
    color: #333;
  }

  ul {
    list-style: none;
    padding: 0;
  }

  li {
    margin-bottom: 0.5rem;
    color: #666;
    font-size: 0.9rem;
    cursor: pointer;

    &:hover {
      color: #00c471;
    }
  }
`;

const Footer = () => {
  return (
    <FooterContainer>
      <FooterContent>
        <FooterSection>
          <h4>숨고 소개</h4>
          <ul>
            <li>회사 소개</li>
            <li>채용 안내</li>
            <li>팀 블로그</li>
          </ul>
        </FooterSection>

        <FooterSection>
          <h4>고객 안내</h4>
          <ul>
            <li>이용 안내</li>
            <li>안전 정책</li>
            <li>예상 비용</li>
          </ul>
        </FooterSection>

        <FooterSection>
          <h4>고수 안내</h4>
          <ul>
            <li>고수 가이드</li>
            <li>고수 가입</li>
          </ul>
        </FooterSection>

        <FooterSection>
          <h4>고객 지원</h4>
          <ul>
            <li>공지사항</li>
            <li>자주 묻는 질문</li>
          </ul>
        </FooterSection>
      </FooterContent>
    </FooterContainer>
  );
};

export default Footer;
```

## 프로젝트를 통해 배운 점

### 긍정적인 경험

#### 1. 새로운 기술 도전
1차 프로젝트에서 사용하지 못한 기술들을 적극적으로 도입했습니다:
- React Hooks (useState, useEffect)
- Styled-Components
- Axios
- Kakao Map API / Kakao Login API

#### 2. 팀원 배려
프로젝트 완성도도 중요하지만, **학습을 우선순위**로 두자는 팀원들의 합의가 있었습니다. 덕분에 부담 없이 새로운 기술을 시도할 수 있었습니다.

#### 3. 협업 능력 향상
1차 때는 내 작업에만 집중했다면, 2차에서는 여유를 갖고 팀원들을 도울 수 있었습니다.

#### 4. Mock Data 경험
Back-End가 제공하지 못한 데이터를 Mock Data로 대체하여 독립적으로 작업할 수 있었습니다.

#### 5. Map 함수의 강력함
서버 데이터를 Map 함수 하나로 효율적으로 렌더링하는 경험이 인상적이었습니다.

### 개선이 필요했던 부분

#### 1. 프로젝트 관리
- **트렐로 관리 미흡**: 후반부에 작업 현황 업데이트가 소홀해짐

#### 2. 코드 컨벤션
- 컨벤션을 정했지만 **네스팅을 고려하지 않아** 코드 통합 시 어려움
- 전체적으로 코드가 깔끔하지 못했음

```javascript
// 문제가 있었던 구조
<div className="container">
  <div className="wrapper">
    <div className="content">
      <div className="item">...</div>
    </div>
  </div>
</div>

// 개선 필요: 과도한 네스팅으로 CSS 선택자 복잡도 증가
.container .wrapper .content .item { ... }
```

#### 3. 기술 이해 부족
- **Git Rebase**: 사용했지만 정확한 동작 원리를 이해하지 못함
- **Filter 함수**: 시간 부족으로 적용하지 못함

#### 4. 발표 준비
- Front-End 발표자였지만 긴장으로 팀원 기능을 제대로 설명하지 못함
- 팀원에게 미안한 감정이 남음

#### 5. 시간 관리
- Class형 컴포넌트 구현에 시간을 많이 써서
- Styled-Components를 충분히 활용하지 못함

## 다음 프로젝트를 위한 개선 방안

### 1. 코드 품질

```markdown
✅ 네스팅을 고려한 코드 컨벤션 수립
  - 최대 3단계 이하 네스팅
  - BEM 방법론 또는 CSS Modules 고려
  - Styled-Components에서 과도한 중첩 지양

✅ 코드 리뷰 문화 정착
  - PR 단위로 코드 리뷰 진행
  - 서로의 코드에서 배우기
```

### 2. 기술 학습

```markdown
✅ Git 심화 학습
  - Rebase, Cherry-pick, Reset 등 완전히 이해
  - Conflict 해결 패턴 학습
  - Git Flow 전략 익히기

✅ JavaScript 배열 메서드 마스터
  - Map, Filter, Reduce 조합
  - 성능 최적화 고려
```

### 3. 협업 개선

```markdown
✅ 트렐로 꾸준한 업데이트
  - 매일 작업 시작/종료 시 업데이트
  - 블로커 즉시 공유

✅ 발표 준비
  - 하루 전 거울 보며 연습
  - 팀원 기능 깊이 이해하기
```

### 4. 일정 관리

```markdown
✅ 우선순위 기반 작업
  - 핵심 기능 먼저 구현
  - 여유 시간 확보 후 도전적 기능 시도

✅ 데드라인 준수
  - 계획한 날짜에 맞춰 구현 완료
  - 필요시 야근도 감수
```

### 5. 팀워크

```markdown
✅ 리팩토링 시간 확보
  - 프로젝트 종료 후 팀원과 함께 코드 개선
  - 베스트 프랙티스 공유
```

## 마치며

1차 프로젝트에 이어 2차 프로젝트에서도 훌륭한 팀원들과 함께할 수 있었습니다. 프로젝트 완성도보다 **학습을 우선시**하자는 팀의 합의 덕분에 조급함 없이 새로운 기술들을 시도해볼 수 있었습니다.

### 핵심 성과

- ✅ React Hooks, Styled-Components 등 새로운 기술 습득
- ✅ Kakao Map/Login API 연동 경험
- ✅ Mock Data로 독립적인 프론트엔드 개발
- ✅ 1차 대비 향상된 협업 능력

### 다짐

> "항상 쓴 것을 계속 쓰는 개발자가 아닌, 새로운 기술에 겁내지 않고 도전하는 개발자가 되자."

이 프로젝트에서 배운 **도전 정신**과 **협업의 가치**를 잊지 않고, 계속해서 성장하는 개발자가 되겠습니다.

팀원들과 꼭 시간을 맞춰 리팩토링을 진행하고 싶습니다!

## 참고 자료

- [React Hooks 공식 문서](https://react.dev/reference/react)
- [Styled-Components 공식 문서](https://styled-components.com/)
- [Axios vs Fetch 비교](https://blog.logrocket.com/axios-vs-fetch-best-http-requests/)
- [Kakao Map API 가이드](https://apis.map.kakao.com/web/)
- [Kakao Login API 문서](https://developers.kakao.com/docs/latest/ko/kakaologin/common)
- [Git Rebase 이해하기](https://git-scm.com/book/ko/v2/Git-브랜치-Rebase-하기)
- [JavaScript Map/Filter/Reduce](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array)
