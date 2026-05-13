# 📘 Day 05 — CSS 속성 심화

## 0. 핵심 빠른 참조

| 속성 | 예시 | 설명 |
|------|------|------|
| `display: block` | `img { display: block; }` | inline → 세로 출력 |
| `display: inline-block` | `div { display: inline-block; }` | 크기 지정 + 가로 출력 |
| `display: none` | `div { display: none; }` | 숨김 (더보기 토글) |
| `background-color` | `background-color: rgba(0,0,0,0.5)` | 배경색 (투명도 포함) |
| `background-image` | `background-image: url('파일경로')` | 배경 이미지 |
| `background-size` | `cover` / `contain` / `100% 100%` | 배경 이미지 크기 |
| `background-repeat` | `no-repeat` / `repeat-x` / `repeat-y` | 반복 여부 |
| `background` 단축 | `background: url('img.jpg') no-repeat center/cover` | 단축 속성 |
| `border-collapse` | `border-collapse: collapse` | 테이블 테두리 통합 |
| `font-family` | `font-family: sans-serif, "맑은 고딕"` | 글꼴 (우선순위 순) |

---

## 1. display 속성

| 값 | 설명 | 대표 태그 |
|----|------|-----------|
| `block` | 한 줄 전체 차지, 세로 출력 | `div`, `h1~h6`, `p`, `ul`, `table` |
| `inline` | 내용 크기만큼만 차지, 가로 출력 | `img`, `a`, `span`, `input`, `b` |
| `inline-block` | 가로 출력 + width/height 지정 가능 | 메뉴 버튼, 카드 |
| `none` | 화면에서 숨김 | 더보기/접기 토글 |

```css
img { display: block; }           /* inline → block : 세로 나열 */
div { display: inline-block; }    /* block → 가로 나열 + 크기 지정 */
div { display: none; }            /* 숨김 */
```

### jQuery 더보기 토글 패턴
```javascript
let i = 0;
$(function() {
  $('a').click(function() {
    if (i == 0) {
      $('a').text("닫기");
      $('div').show();
      i = 1;
    } else {
      $('a').text("더보기");
      $('div').hide();
      i = 0;
    }
  });
});
```

---

## 2. background 속성

```css
/* 배경색 */
background-color: lightblue;
background-color: rgba(255, 255, 255, 0.5);  /* 투명도 0.0~1.0 */

/* 배경 이미지 */
background-image: url('이미지경로');
background-repeat: no-repeat;     /* repeat / repeat-x / repeat-y */
background-size: cover;           /* 비율 유지, 영역 전체 채움 */
background-size: contain;         /* 비율 유지, 이미지 전체 보임 */
background-size: 100% 100%;       /* 영역에 맞게 늘림 */
background-position: center;

/* 단축 속성 */
background: url('img.jpg') no-repeat center / cover;
```

### img vs background-image
| 구분 | `<img>` | `background-image` |
|------|---------|-------------------|
| 위에 텍스트 | ❌ | ✅ 가능 |
| 크기 조절 | width/height | background-size |
| 용도 | 콘텐츠 이미지 | 장식용 배경 |

---

## 3. table CSS 스타일

```css
.table {
  border-collapse: collapse;   /* 테두리 이중선 → 단선 통합 */
  width: 600px;
  margin: 50px auto;           /* 가운데 정렬 */

  border-top: 2px solid #333;
  border-bottom: 1px solid #333;
}
.table th {
  background-color: #999;
  color: #fff;
  padding: 8px;
}
.table td {
  padding: 8px;
  border-bottom: 1px dotted #666;
}
```

---

## 4. CSS 속성 전체 분류

| 분류 | 속성 | 설명 |
|------|------|------|
| 박스 | `margin` / `padding` / `border` | 여백, 테두리 |
| 가시 | `display` / `overflow` | 표시 방식 |
| 배경 | `background` | 배경색, 이미지 |
| 텍스트 | `font` / `text` | 글꼴, 정렬 |
| 위치 | `position` | static / absolute / relative / fixed |
| 유동 | `float` / `z-index` | 겹침, 레이아웃 |
| 레이아웃 | `flex` / `grid` | 카드, 반응형 |

### 위치 속성 (position) 개요
| 값 | 설명 |
|----|------|
| `static` | 기본값 — 소스 순서대로 |
| `relative` | 원래 위치 기준으로 이동 |
| `absolute` | 부모 기준 절대좌표 |
| `fixed` | 브라우저 기준 고정 (상단 메뉴바) |

---

## 5. 텍스트 / 글꼴 속성

```css
body {
  font-family: sans-serif, "맑은 고딕", 궁서체; /* 우선순위 순 */
  margin: 20px;
}
h2 {
  border-bottom: 2px solid #333;   /* 제목 아래 구분선 */
  padding-bottom: 5px;
}
```

---

## 6. 앞으로 배울 것

```
레이아웃    Flex / Grid
시멘틱 태그  header / nav / section / aside / footer
반응형      @media 쿼리
CSS 라이브러리  Bootstrap / TailwindCSS
```

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `css_1.html` | 박스 모델 종합 (margin / padding / border / border-radius) |
| `css_2.html` | table CSS 스타일링 (border-collapse) |
| `css_3.html` | display: block / inline-block |
| `css_4.html` | display: none + jQuery 더보기 토글 |
| `css_5.html` | background-image vs img 비교 |
| `css_6.html` | background 속성 전체 정리 |
