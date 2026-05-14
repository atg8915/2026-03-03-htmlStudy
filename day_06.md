# 📘 Day 06 — CSS position / 가시 속성 / jQuery Ajax

## 0. 핵심 빠른 참조

| 속성 | 예시 | 설명 |
|------|------|------|
| `position: static` | 기본값 | 소스 순서대로 배치 |
| `position: relative` | `top: 10px; left: 10px` | 자기 원래 위치 기준 이동 |
| `position: absolute` | `top: 10px; right: 10px` | 부모(relative) 기준 배치 |
| `position: fixed` | `top: 0; right: 0` | 브라우저 기준 고정 (메뉴바) |
| `position: sticky` | `top: 0` | 스크롤 시 특정 위치에서 고정 |
| `display: none` | — | 숨김 + 공간도 사라짐 |
| `visibility: hidden` | — | 숨김 + 공간은 유지 |
| `opacity: 0` | — | 투명 + 공간 유지, 클릭 가능 |
| `overflow: hidden` | — | 넘치는 내용 잘라냄 |
| `overflow: auto` | — | 넘칠 때만 스크롤 생성 |
| `z-index` | `z-index: 10` | 숫자 클수록 위에 표시 |
| 가운데 정렬 | `top:50%; left:50%; transform:translate(-50%,-50%)` | absolute 가운데 정렬 |

---

## 1. position 속성

```css
/* 1. static — 기본값, 소스 순서대로 */
position: static;

/* 2. relative — 원래 위치 기준으로 이동 (공간 유지) */
position: relative;
top: 10px;
left: 10px;

/* 3. absolute — 가장 가까운 relative 부모 기준 */
/* 부모에 position: relative 필수 */
position: absolute;
top: 10px;
right: 10px;

/* 4. fixed — 브라우저 화면 기준 고정 (스크롤해도 안 움직임) */
position: fixed;
top: 20px;
right: 20px;

/* 5. sticky — 스크롤 따라가다 특정 위치에서 고정 */
position: sticky;
top: 0;
```

### 실무 활용 패턴

```css
/* NEW / HOT 배지 */
.card { position: relative; }
.badge {
  position: absolute;
  top: 10px; right: 10px;
  background: crimson; color: white;
  padding: 4px 8px; border-radius: 6px;
}

/* 이미지 위 텍스트 오버레이 */
.overlay {
  position: absolute;
  bottom: 0; left: 0; width: 100%;
  background: rgba(0,0,0,0.5);
  color: white; padding: 10px;
}

/* 가운데 정렬 */
.center {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
}

/* 쇼핑몰 카드 hover 효과 */
.product { transition: 0.3s; }
.product:hover { transform: translateY(-6px); }
.product .info { opacity: 0; transition: 0.3s; }
.product:hover .info { opacity: 1; }
```

---

## 2. 가시 속성 비교

| 속성 | 화면 | 공간 | 클릭 | 용도 |
|------|------|------|------|------|
| `display: none` | ❌ | ❌ | ❌ | 더보기/토글, v-if |
| `visibility: hidden` | ❌ | ✅ | ❌ | 레이아웃 유지하며 숨김 |
| `opacity: 0` | ❌ | ✅ | ✅ | 페이드 효과, hover 전환 |

```css
display: none;         /* 완전 숨김 */
visibility: hidden;    /* 공간 유지 숨김 */
opacity: 0;            /* 투명 (공간·클릭 유지) */
```

---

## 3. overflow 속성

```css
overflow: visible;  /* 기본값 — 넘쳐도 그대로 표시 */
overflow: hidden;   /* 넘치는 내용 잘라냄 (카드 이미지 둥글게 처리 시 필수) */
overflow: scroll;   /* 항상 스크롤바 표시 */
overflow: auto;     /* 넘칠 때만 스크롤 생성 */
```

---

## 4. z-index — 겹침 순서

```css
/* 숫자 클수록 위에 표시, position이 있어야 동작 */
.red   { z-index: 10; position: absolute; }  /* 맨 위 */
.green { z-index: 5;  position: absolute; }
.blue  { z-index: 1;  position: absolute; }  /* 맨 아래 */
```

---

## 5. 가로 메뉴 + 카드 레이아웃

```css
/* 가로 메뉴 */
ul { list-style: none; }
ul li {
  display: inline-block;
  width: 120px;
  height: 30px;
}

/* 카드 가로 나열 */
div {
  display: inline-block;
  width: 300px;
  height: 300px;
  margin: 3px;
}
```

---

## 6. jQuery Ajax — 동적 데이터 로딩

```javascript
$.ajax({
  type: "get",
  url: "result.jsp",
  data: { "no": no },          // 서버로 전달할 파라미터
  success: function(res) {
    let json = JSON.parse(res); // JSON 문자열 → 객체 변환
    let html = '';
    for (let i = 0; i < json.length; i++) {
      html += '<tr>'
            + '<td>' + json[i].rank + '</td>'
            + '<td>' + json[i].movieNm + '</td>'
            + '</tr>';
    }
    $('.list tbody').html(html); // 테이블에 동적 삽입
  }
});
```

> Ajax 흐름: 버튼 클릭 → JSP 요청 → JSON 응답 → HTML 동적 생성 → 테이블 갱신
> jQuery `.show()` / `.hide()` → `display: block` / `display: none` 제어

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `css_1.html` | display 속성 + 가로 메뉴 + 카드 레이아웃 |
| `css_1_1.html` | Bootstrap + jQuery Ajax 영화 목록 |
| `css_2.html` | 가시 속성 정리 (none / hidden / opacity / overflow / z-index) |
| `css_8.html` | position 5종 예제 (static/relative/absolute/fixed/sticky) |
| `css_9.html` | position 실무 패턴 (배지/오버레이/hover 카드) |
