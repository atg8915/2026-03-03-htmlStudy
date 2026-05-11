# 📘 Day 03 — CSS 선택자 / 입력 폼 심화

## 0. 핵심 빠른 참조

| 선택자 | 예시 | 설명 |
|--------|------|------|
| 태그 | `p { color: red; }` | 해당 태그 전체 적용 |
| `#id` | `#red { color: red; }` | 특정 1개 요소 (중복 불가) |
| `.class` | `.blue { color: blue; }` | 여러 요소 공통 적용 |
| 그룹 | `p, a, b { background: yellow; }` | 여러 태그 동시 적용 |
| 자식 | `div > h1 { color: blue; }` | div 바로 아래 h1만 |
| 후손 | `div span h1 { color: green; }` | div 안의 모든 h1 |
| 인접 | `p + h1 { color: orange; }` | p 바로 다음 h1 |
| 형제 | `p ~ h1 { color: orange; }` | p 이후 모든 h1 |
| 속성 | `a[href='URL'] { color: red; }` | 특정 속성값 일치 |
| 속성 시작 | `a[href^='http'] { }` | 특정 문자로 시작 |
| 속성 끝 | `a[href$='net'] { }` | 특정 문자로 끝 |
| 속성 포함 | `a[href*='www'] { }` | 특정 문자 포함 |
| 가상 클래스 | `a:hover { color: green; }` | 마우스 오버 시 |
| 가상 클래스 | `img:hover { opacity: 0.3; }` | 이미지 투명도 |

---

## 1. CSS 적용 방식 3가지

```html
<!-- 1. 인라인 CSS — 태그 1개만 적용 -->
<h1 style="color: red;">Hello</h1>

<!-- 2. 내부 CSS — 현재 파일만 적용 -->
<style>
  h1 { color: blue; }
</style>

<!-- 3. 외부 CSS — 모든 파일에 공통 적용 (권장) -->
<link rel="stylesheet" href="table.css">
```

---

## 2. id vs class

| 구분 | 선택자 | 중복 | 용도 |
|------|--------|------|------|
| `id` | `#이름` | ❌ 불가 | 페이지에서 유일한 요소 |
| `class` | `.이름` | ✅ 가능 | 여러 요소 공통 스타일 |

```css
/* 같은 class명 중복 선언 시 마지막 것 적용 */
.red { color: red; }
.red { color: magenta; }  /* 이게 적용됨 */
```

---

## 3. 주요 CSS 속성

```css
/* 텍스트 */
color: red;
background-color: yellow;
text-align: center;
font-weight: bold;

/* 박스 */
border: 5px solid red;
border-radius: 50px;      /* 둥근 테두리 */
padding: 16px;
margin: 0px auto;         /* 가운데 정렬 */

/* 이미지 */
opacity: 0.3;             /* 투명도 0.0~1.0 */
width: 150px;
height: 250px;

/* 링크/커서 */
text-decoration: none;    /* 밑줄 제거 */
cursor: pointer;

/* display */
display: block;           /* inline → block 변환 */
```

---

## 4. 가상 클래스 (Pseudo-class)

```css
/* 마우스 오버 */
a:hover { color: green; text-decoration: underline; }
img:hover { opacity: 0.3; border-radius: 30px; }

/* 속성 선택자로 input 스타일 분리 */
input[type='text']     { border-radius: 20px; }
input[type='password'] { border-color: yellow; }
input[type='submit']   { border-color: red; }
```

---

## 5. 입력 폼 심화 — 회원가입 전체 구성

```html
<form>
  <!-- ID + 중복체크 버튼 -->
  <input type="text" size="20">
  <input type="button" value="중복체크">

  <!-- 비밀번호 확인 -->
  <input type="password" size="20" placeholder="재입력">

  <!-- 성별 라디오 (name으로 그룹) -->
  <input type="radio" name="sex" checked> 남자
  <input type="radio" name="sex"> 여자

  <!-- 나이 / 가격대 -->
  <input type="number" min="10" max="90">
  <input type="range" min="10000" max="100000">

  <!-- 우편번호 / 주소 (읽기 전용) -->
  <input type="text" readonly>

  <!-- 취미 체크박스 -->
  <input type="checkbox"> 여행
  <input type="checkbox"> 낚시

  <!-- 소개 여러줄 -->
  <textarea rows="10" cols="45"></textarea>

  <!-- 전화 콤보 + 텍스트 -->
  <select><option>010</option></select>
  <input type="text" size="15">

  <!-- 전송 / 취소 -->
  <input type="submit" value="회원가입">
  <input type="button" value="취소">
</form>
```

### input 속성 정리
| 속성 | 설명 |
|------|------|
| `placeholder` | 입력 전 안내 텍스트 |
| `readonly` | 읽기 전용 (우편번호, 주소 자동입력) |
| `checked` | 기본 선택 상태 |
| `multiple` | 파일 다중 선택, select 다중 선택 |
| `min` / `max` | number, range 범위 지정 |
| `name` | radio 그룹핑 / Java로 값 전송 시 사용 |

---

## 6. select 그룹화 — optgroup

```html
<select multiple>
  <optgroup label="Back-End">
    <option>자바</option>
    <option>오라클</option>
    <option>Spring-Boot</option>
  </optgroup>
  <optgroup label="Front-End">
    <option>HTML/CSS</option>
    <option>VueJS</option>
    <option>ReactJS</option>
  </optgroup>
</select>
```

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `css_1.html` | 태그 선택자, 그룹 선택자 |
| `css_2.html` | id / class 선택자 |
| `css_3.html` | class 중복 선언 우선순위 |
| `css_4.html` | border, opacity, border-radius |
| `css_5.html` | 속성 선택자, input 스타일 분리 |
| `css_6.html` | 자식/후손/인접/형제 선택자 |
| `css_7.html` | hover 가상 클래스 |
| `html_5.html` | 회원가입 폼 전체 구성 |
| `html_6.html` | optgroup, fieldset, figure |
| `html_7.html` | rowspan + 상세보기 카드 |
