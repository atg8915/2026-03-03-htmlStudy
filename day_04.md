# 📘 Day 04 — CSS 선택자 심화 / 박스 모델

## 0. 핵심 빠른 참조

| 선택자 | 예시 | 설명 |
|--------|------|------|
| 자손 | `div > h1` | div 바로 아래 h1만 |
| 후손 | `div span h1` | div 안 모든 h1 |
| 인접 | `p + h1` | p 바로 다음 h1 |
| 형제 | `p ~ h1` | p 이후 모든 형제 h1 |
| 속성 일치 | `input[type="text"]` | 속성값 동일 |
| 속성 포함 | `img[src*="m1"]` | 값 포함 |
| 속성 시작 | `a[href^="http"]` | 값으로 시작 |
| 속성 끝 | `a[href$="net"]` | 값으로 끝 |
| 첫 번째 | `li:first-child` | 첫 번째 자식 |
| 마지막 | `li:last-child` | 마지막 자식 |
| n번째 | `li:nth-child(2n)` | 짝수 번째 |
| 반응 | `a:hover` | 마우스 오버 |
| 반응 | `h3:active` | 클릭 중 |
| 상태 | `input:focus` | 포커스 |
| 상태 | `input:disabled` | 비활성화 |
| 상태 | `input:checked` | 체크된 상태 |
| 문자 | `li.new::after` | 태그 뒤에 콘텐츠 삽입 |
| 문자 | `li.new::before` | 태그 앞에 콘텐츠 삽입 |

---

## 1. 선택자 우선순위

```
인라인 style > ID(#) > CLASS(.) > 태그
```

> 같은 class 중복 선언 시 → **나중에 선언된 것** 적용

---

## 2. 구조 선택자 — nth-child

```css
li:first-child  { }          /* 첫 번째 */
li:last-child   { }          /* 마지막 */
li:nth-child(2n)   { }       /* 짝수 (2,4,6...) */
li:nth-child(2n+1) { }       /* 홀수 (1,3,5...) */
li:nth-child(3n+1) { }       /* 1,4,7,10... */
```

### 활용 — 가로 메뉴바
```css
li {
  list-style: none;
  float: left;
  padding: 15px;
}
li:first-child { border-radius: 20px 0 0 20px; }
li:last-child  { border-radius: 0 20px 20px 0; }
li:nth-child(2n)   { background: #FF0003; }
li:nth-child(2n+1) { background: #800000; }
```

### 활용 — 테이블 줄 색상
```css
table thead tr:first-child { background: orange; }
table tbody tr:nth-child(2n) { background: rgb(255,255,220); }
```

---

## 3. 반응 / 상태 선택자

```css
/* 반응 선택자 */
a:hover  { color: gray; }        /* 마우스 오버 */
h3:active { color: red; }        /* 클릭 중 */

/* 상태 선택자 */
input:enabled  { background: yellow; }   /* 활성화 */
input:disabled { background: black; }    /* 비활성화 */
input:focus    { background: white; }    /* 포커스 */
input[type="checkbox"]:checked {
  box-shadow: 0 0 0 3px hotpink;
}
```

---

## 4. 문자 선택자 — ::before / ::after

> 태그 앞뒤에 콘텐츠 삽입 — NEW 뱃지, 특가 표시 등에 활용

```css
li.new::after {
  content: "NEW";
  background: red;
  border-radius: 5px;
}
li.new::before {
  content: "NEW";
  background: blue;
}
```

---

## 5. 속성 선택자 — id/class 없을 때

```css
/* input 타입별 스타일 분리 */
input[type="text"]     { width: 200px; height: 30px; }
input[type="submit"]   { width: 100px; border-radius: 10px; }

/* img src 포함 여부 */
img[src*="m1"] { border-radius: 20px; }
img[src*="m5"] { border-radius: 100px; }

/* a href 조건 */
a[href^="http"]  { color: orange; }  /* http로 시작 */
a[href$="net"]   { color: blue; }    /* net으로 끝 */
a[href*="www"]   { color: black; }   /* www 포함 */
```

---

## 6. 박스 모델

```
┌─────────────────────────┐
│         margin          │  ← 외부 여백
│  ┌───────────────────┐  │
│  │      border       │  │  ← 테두리
│  │  ┌─────────────┐  │  │
│  │  │   padding   │  │  │  ← 내부 여백
│  │  │  ┌───────┐  │  │  │
│  │  │  │content│  │  │  │  ← 실제 콘텐츠
│  │  │  └───────┘  │  │  │
│  │  └─────────────┘  │  │
│  └───────────────────┘  │
└─────────────────────────┘
```

### margin — 외부 여백
```css
margin: 10px;                    /* 상하좌우 동일 */
margin: 10px 20px;               /* 상하 | 좌우 */
margin: 10px 20px 30px 40px;     /* 상 우 하 좌 (시계방향) */
margin: 0px auto;                /* 가운데 정렬 */
margin-top / right / bottom / left
```

### padding — 내부 여백
```css
padding: 10px;
padding: 10px 20px;
padding: 25px;                   /* 태그 안쪽 여백 */
padding-top / right / bottom / left
```

### border — 테두리
```css
border: 2px solid red;           /* 두께 종류 색상 */
border-radius: 50px;             /* 둥근 모서리 */
border-top-left-radius: 50px;    /* 개별 모서리 */

/* 선 종류 */
solid   /* 실선 */
dotted  /* 점선 ...... */
dashed  /* 파선 ------ */
double  /* 이중선 */
groove / inset / outset
```

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `css_1.html` | class 다중 적용 (`class="img img1"`) |
| `css_2.html` | 속성 선택자 + jQuery 이미지 클릭 이벤트 |
| `css_3.html` | 자손/후손/동위 선택자 |
| `css_4.html` | border 선 종류 7가지 / 반응·상태 선택자 |
| `css_5.html` | border-radius 개별 모서리 |
| `css_6.html` | nth-child 테이블 줄 색상 |
| `css_7.html` | ::before / ::after 문자 선택자 |
| `css_1~3 (box)` | margin / padding / border 박스 모델 |
| `select_total.html` | 가로 메뉴바 + JS 이벤트 리스너 |
