# 개발 블로그

Jekyll 기반의 개발 블로그입니다.

## 주요 기능

- 카테고리 및 태그 분류
- 전체 검색 기능
- SEO 최적화
- 반응형 디자인
- 다크/라이트 모드
- 목차(TOC) 자동 생성
- 코드 하이라이팅
- GitHub Pages 자동 배포

## 로컬 개발 환경 설정

### 사전 요구사항

- Ruby (>= 3.0)
- RubyGems
- Bundler

### 설치 및 실행

```bash
# 의존성 설치
bundle install

# 로컬 서버 실행
bundle exec jekyll serve

# 또는 초안(drafts)과 함께 실행
bundle exec jekyll serve --drafts

# 브라우저에서 http://localhost:4000 접속
```

## 새 포스트 작성하기

1. `_posts` 폴더에 `YYYY-MM-DD-title.md` 형식으로 파일 생성
2. Front Matter 작성:

```yaml
---
title: 포스트 제목
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [카테고리1, 카테고리2]
tags: [태그1, 태그2, 태그3]
---
```

3. 마크다운으로 내용 작성
4. 커밋 후 푸시하면 자동으로 배포됨

## 설정 변경

`_config.yml` 파일에서 블로그 설정을 변경할 수 있습니다:

- 블로그 제목, 설명
- 소셜 미디어 링크
- Google Analytics
- 댓글 시스템
- 기타 테마 옵션

## GitHub Pages 배포

이 블로그는 GitHub Actions를 통해 자동으로 배포됩니다.

1. GitHub 저장소 Settings > Pages로 이동
2. Source를 "GitHub Actions"로 설정
3. main 브랜치에 푸시하면 자동 배포

## 사용된 기술

- **Jekyll 4.3**: 정적 사이트 생성기
- **Chirpy Theme**: 인기있는 Jekyll 테마
- **GitHub Pages**: 무료 호스팅
- **GitHub Actions**: 자동 배포

## 라이선스

이 프로젝트는 MIT 라이선스를 따릅니다.

## 테마 크레딧

이 블로그는 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마를 사용합니다.