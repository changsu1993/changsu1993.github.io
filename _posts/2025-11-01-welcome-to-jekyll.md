---
title: Jekyll 블로그에 오신 것을 환영합니다
date: 2025-11-01 12:00:00 +0900
categories: [블로그, 튜토리얼]
tags: [jekyll, github-pages, 블로그]
author: changsu
toc: true
comments: true
---

## 환영합니다!

Jekyll 기반 개발 블로그에 오신 것을 환영합니다. 이 포스트는 Jekyll의 기본 사용법을 안내합니다.

## 포스트 작성하기

Jekyll에서 포스트를 작성하려면 `_posts` 디렉토리에 다음 형식의 파일을 생성하면 됩니다:

```
YYYY-MM-DD-title.md
```

### Front Matter

각 포스트의 상단에는 YAML Front Matter가 필요합니다:

```yaml
---
title: 포스트 제목
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [카테고리1, 카테고리2]
tags: [태그1, 태그2]
---
```

## 마크다운 문법

### 제목

```markdown
# H1
## H2
### H3
```

### 강조

- **굵게**: `**텍스트**` 또는 `__텍스트__`
- *기울임*: `*텍스트*` 또는 `_텍스트_`
- ~~취소선~~: `~~텍스트~~`

### 코드 블록

#### 인라인 코드

`인라인 코드`는 백틱으로 감싸면 됩니다.

#### 코드 블록

```python
def hello_world():
    print("Hello, World!")

hello_world()
```

```javascript
function helloWorld() {
    console.log("Hello, World!");
}

helloWorld();
```

### 인용문

> 이것은 인용문입니다.
> 여러 줄로 작성할 수 있습니다.

### 리스트

#### 순서 없는 리스트

- 항목 1
- 항목 2
  - 하위 항목 2.1
  - 하위 항목 2.2
- 항목 3

#### 순서 있는 리스트

1. 첫 번째
2. 두 번째
3. 세 번째

### 링크

[링크 텍스트](https://example.com)

### 이미지

```markdown
![이미지 설명](/path/to/image.jpg)
```

## 팁 박스

> 이것은 팁입니다!
{: .prompt-tip }

> 이것은 정보입니다!
{: .prompt-info }

> 이것은 경고입니다!
{: .prompt-warning }

> 이것은 위험 경고입니다!
{: .prompt-danger }

## 표

| 헤더 1 | 헤더 2 | 헤더 3 |
|--------|--------|--------|
| 셀 1   | 셀 2   | 셀 3   |
| 셀 4   | 셀 5   | 셀 6   |

## 마무리

Jekyll로 멋진 기술 블로그를 만들어보세요!

더 자세한 정보는 [Jekyll 공식 문서](https://jekyllrb.com/docs/)를 참고하세요.
