---
title: HTML Table 완벽 가이드 - 표 생성과 셀 병합
description: HTML table 태그의 기본 구조부터 thead, tbody, th, td 태그 사용법, colspan/rowspan을 활용한 셀 병합까지 표 작성의 모든 것을 학습합니다.
author: changsu
date: 2020-06-15 10:54:23 +0900
categories: [Web, HTML]
tags: [html, table]
---

## 1. HTML Table 기본

### 1.1 테이블 구조

```html
<table>
  <tr>
    <td>Row 1, Column 1</td>
    <td>Row 1, Column 2</td>
  </tr>
  <tr>
    <td>Row 2, Column 1</td>
    <td>Row 2, Column 2</td>
  </tr>
</table>
```

### 1.2 주요 태그

| 태그 | 설명 |
|------|------|
| `<table>` | 테이블 전체를 감싸는 컨테이너 |
| `<tr>` | Table Row - 행(가로줄) |
| `<td>` | Table Data - 데이터 셀 |
| `<th>` | Table Header - 헤더 셀 (굵게, 중앙 정렬) |
| `<thead>` | 테이블 헤더 그룹 |
| `<tbody>` | 테이블 본문 그룹 |
| `<tfoot>` | 테이블 푸터 그룹 |

## 2. 기본 테이블 예제

```html
<table>
  <tr>
    <th>Dog</th>
    <th>Cat</th>
  </tr>
  <tr>
    <td>Canine</td>
    <td>Feline</td>
  </tr>
  <tr>
    <td>Bark</td>
    <td>Meow</td>
  </tr>
</table>
```

```css
table {
  border-collapse: collapse;
  width: 100%;
}

th, td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
  font-weight: bold;
}
```

## 3. 셀 병합

### 3.1 colspan (열 병합)

```html
<table>
  <tr>
    <th>이름</th>
    <th>국어</th>
    <th>영어</th>
  </tr>
  <tr>
    <td>홍길동</td>
    <td>90</td>
    <td>85</td>
  </tr>
  <tr>
    <td colspan="3">전체 평균: 87.5점</td>
  </tr>
</table>
```

### 3.2 rowspan (행 병합)

```html
<table>
  <tr>
    <th>시간</th>
    <th>내용</th>
    <th>장소</th>
  </tr>
  <tr>
    <th>1시</th>
    <td>HTML 기초</td>
    <td>101호</td>
  </tr>
  <tr>
    <th>2시</th>
    <td rowspan="2">HTML 실습</td>
    <td>102호</td>
  </tr>
  <tr>
    <th>3시</th>
    <td>103호</td>
  </tr>
</table>
```

## 4. 완전한 테이블 구조

```html
<table>
  <thead>
    <tr>
      <th>제품</th>
      <th>가격</th>
      <th>재고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>노트북</td>
      <td>1,500,000원</td>
      <td>5개</td>
    </tr>
    <tr>
      <td>마우스</td>
      <td>30,000원</td>
      <td>20개</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="2">합계</td>
      <td>25개</td>
    </tr>
  </tfoot>
</table>
```

## 5. 실전 스타일링

```css
table {
  width: 100%;
  border-collapse: collapse;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

thead {
  background-color: #007bff;
  color: white;
}

th, td {
  padding: 12px 15px;
  text-align: left;
  border: 1px solid #ddd;
}

tbody tr:hover {
  background-color: #f5f5f5;
}

tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}
```

## 6. 반응형 테이블

```css
@media (max-width: 768px) {
  table {
    display: block;
    overflow-x: auto;
  }
}
```

## 정리

```html
<!-- 기본 구조 -->
<table>
  <thead>
    <tr><th>헤더</th></tr>
  </thead>
  <tbody>
    <tr><td>데이터</td></tr>
  </tbody>
</table>

<!-- 셀 병합 -->
<td colspan="2">열 병합</td>
<td rowspan="2">행 병합</td>
```

**핵심 CSS:**
```css
table { border-collapse: collapse; }
th, td { border: 1px solid #ddd; padding: 12px; }
```

## 참고 자료

- [MDN - table 요소](https://developer.mozilla.org/ko/docs/Web/HTML/Element/table)
- [MDN - colspan](https://developer.mozilla.org/ko/docs/Web/HTML/Element/td#attr-colspan)
- [MDN - rowspan](https://developer.mozilla.org/ko/docs/Web/HTML/Element/td#attr-rowspan)
