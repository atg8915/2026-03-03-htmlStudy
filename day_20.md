# 📘 Day 20 — DOM 객체 모델 + 이벤트 처리 + 외부 JS 분리

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 태그 1개 선택 | `document.querySelector("CSS선택자")` | id / class / tag / 자손 / 후손 / 속성 |
| 태그 여러개 선택 | `document.querySelectorAll("CSS선택자")` | 배열 `[]` 반환 → `for` / `forEach` |
| id로 선택 | `document.getElementById("id명")` | id 속성이 있을 때 |
| class로 선택 | `document.getElementsByClassName("class명")` | 배열 반환 |
| 글자 변경 | `el.textContent = "값"` | 순수 텍스트만 (HTML 태그 파싱 안 함) |
| HTML 삽입 | `el.innerHTML = "<태그>값</태그>"` | HTML 포함 삽입 ⭐ |
| 스타일 변경 | `el.style.속성 = "값"` | `background-color` → `backgroundColor` |
| 속성 변경 | `img.src = "경로"` / `a.href = "url"` | 태그 속성 직접 변경 |
| 태그 생성 | `document.createElement("태그명")` | 동적 태그 생성 |
| 태그 추가 | `부모.appendChild(자식)` | 마지막 자식으로 추가 |
| 태그 삭제 | `el.remove()` | 해당 태그 DOM에서 제거 |
| 인라인 이벤트 | `<button onclick="함수()">` | 태그에 직접 이벤트 등록 |
| 고전 이벤트 | `el.onclick = function(){}` | JS에서 이벤트 등록 |
| 이벤트 리스너 | `el.addEventListener('click', fn)` | 권장 방식 ⭐ |
| 자동 반복 | `setInterval(fn, ms)` | ms마다 fn 반복 호출 |
| 주기 정지 | `clearInterval(id)` | setInterval 중단 |
| DOM 로딩 후 실행 | `DOMContentLoaded` | HTML 파싱 완료 후 실행 (외부 JS용) |
| 실시간 검색 | `el.addEventListener('keyup', fn)` | 키 입력마다 검색 필터링 |
| 키워드 검색 | `str.includes("키워드")` | 문자열 포함 여부 확인 |
| 행 숨기기 | `row.style.display = 'none'` | 검색 결과 필터링에 활용 |

---

## 1. DOM 이란

```text
HTML → 메모리에 트리 구조로 저장
태그 = 클래스 / 속성 = 멤버변수

         html
          |
    ─────────────
    |            |
   head        body
                |
         ────────────────
         |       |      |
        div     div   span

DOM : 이 트리를 JS로 제어할 수 있게 만든 구조
1) 문서 객체 선택
2) 글자 / 스타일 / 속성 조작
3) 이벤트 등록
```

---

## 2. 문서 객체 선택 (js_1.jsp ~ js_2.jsp)

```javascript
// 1개 선택
document.querySelector("#id명")          // id
document.querySelector(".class명")       // class
document.querySelector("태그명")         // tag
document.querySelector("div > p")        // 자손 (바로 아래)
document.querySelector("div span p")     // 후손 (깊이 무관)
document.querySelector("input[type=text]")        // 속성 선택자
document.querySelector("li:nth-child(2n)")        // 구조 선택자

// 여러개 선택 → 배열 []
document.querySelectorAll("h1")          // 태그명 전체
document.querySelectorAll(".box")        // class 전체

// 구형 방식
document.getElementById("id명")          // id로 1개
document.getElementsByClassName("cls")  // class → 배열
document.getElementsByName("name값")     // name → 배열

// 스타일 / 글자 조작
el.style.color = "red"
el.style.backgroundColor = "yellow"     // background-color → camelCase
el.textContent = "글자만"               // HTML 태그 파싱 안 함
el.innerHTML = "<b>HTML포함</b>"        // HTML 포함 삽입 ⭐
```

---

## 3. 속성 변경 + setInterval 자동 슬라이드 (js_3.jsp)

```javascript
let index = 1

let next = () => {
    index++
    if (index > 7) index = 1
    document.querySelector("img").src = "../images/m" + index + ".jpg"
    // img.src = 속성 직접 변경
}

// setInterval : ms마다 반복 실행 → 실시간 뉴스/날씨/슬라이드
let auto = () => {
    setInterval(() => { next() }, 1000)  // 1000ms = 1초
}
// a.href = "" / input.value = "" 도 동일한 속성 변경 방식
```

---

## 4. 이벤트 처리 3가지 방식 (js_5.jsp ~ js_7.jsp)

```javascript
// ① 인라인 이벤트 — 태그에 직접 (Vue/React 방식과 유사)
// <button onclick="btnClick()">클릭</button>
function btnClick() { alert("버튼 클릭!!") }

// ② 고전 이벤트 — JS에서 등록 (jQuery 방식)
let img = document.querySelector("img")
img.onmouseover = function() { img.style.opacity = 0.3 }
img.onmouseout  = function() { img.style.opacity = 1.0 }
img.onclick     = function() { alert("상세보기로 이동") }

// ③ 이벤트 리스너 — 권장 방식 ⭐
// jQuery : $("#green").on('click', function(){})
// Vue    : <button @click="함수">
// React  : <button onClick={함수}>
btn.addEventListener('click', function() {
    h1.style.color = "green"
})
```

| 방식 | 문법 | 사용처 |
|------|------|--------|
| 인라인 | `onclick="fn()"` | 태그 1개 제어 / Vue·React |
| 고전 | `el.onclick = fn` | jQuery 스타일 |
| 리스너 | `addEventListener('click', fn)` | 권장 ⭐ / jQuery 4 권장 |

### 주요 이벤트 종류

| 이벤트 | 발생 시점 | 주요 용도 |
|--------|---------|---------|
| `click` | 클릭 | 버튼, 이미지 |
| `mouseover` / `mouseout` | 마우스 진입/이탈 | 이미지 hover |
| `mousedown` / `mouseup` | 마우스 누름/뗌 | 드래그, 색상 변경 |
| `keyup` / `keydown` | 키보드 입력 | 실시간 검색, 채팅 |
| `change` | 값 변경 | select, input |
| `submit` | 폼 제출 | form 유효성 검사 |

---

## 5. 동적 태그 생성 / 삭제 (js_12.jsp)

```javascript
// DOMContentLoaded : HTML 파싱 완료 후 실행
// window.onload와 차이 : 이미지 등 리소스 로딩 기다리지 않음
// 외부 JS 파일(.js)에서 DOM 접근 시 사용 ⭐
document.addEventListener("DOMContentLoaded", function() {

    let fileIndex = 0

    // 태그 동적 추가
    addBtn.addEventListener('click', function() {
        const tbody = document.querySelector("#user-table tbody")
        const tr = document.createElement("tr")   // 태그 생성
        tr.id = "m" + fileIndex
        tr.innerHTML = '<td>파일 ' + (fileIndex + 1) + '</td>'
                     + '<td><input type=file></td>'
        tbody.appendChild(tr)    // 부모에 자식으로 추가
        fileIndex++
    })

    // 태그 동적 삭제
    removeBtn.addEventListener('click', function() {
        if (fileIndex > 0) {
            document.getElementById('m' + (fileIndex - 1)).remove()  // 삭제
            fileIndex--
        }
    })
})
```

---

## 6. Axios + 실시간 검색 필터 (js_10.jsp + music.js) ⭐

```javascript
let list = []

window.onload = () => {
    // 서버 데이터 수신 → 테이블 출력
    axios.get('music.do')
         .then(response => {
             list = response.data
             let html = ''
             list.forEach((m) => {
                 // 순위 변동 상태에 따라 색상 표시
                 let s = m.state === '상승' ? '<font color="red">▲</font> ' + m.id
                       : m.state === '하강' ? '<font color="blue">▼</font> ' + m.id
                       : '-'
                 html += '<tr>'
                       + '<td>' + m.no + '</td>'
                       + '<td>' + s + '</td>'
                       + '<td><img src="' + m.poster + '" width=30 height=30></td>'
                       + '<td>' + m.title + '</td>'
                       + '<td>' + m.singer + '</td>'
                       + '</tr>'
             })
             document.querySelector('#user-table tbody').innerHTML = html

             // 헤더 배경색 변경
             document.querySelector("#user-table thead tr:first-child")
                     .style.backgroundColor = 'orange'
         })

    // keyup 이벤트 : 실시간 검색 필터링 ⭐
    document.querySelector('#keyword').addEventListener('keyup', function() {
        const keyword = this.value.trim()
        const rows = document.querySelectorAll('#user-table > tbody > tr')

        rows.forEach((row) => {
            const title = row.querySelector('td:nth-child(4)').textContent
            // includes() : 문자열 포함 여부 확인
            row.style.display = title.includes(keyword) ? '' : 'none'
        })
    })
}
// 외부 JS 파일로 분리 → js_10.jsp에서 <script src="music.js"> 로 불러옴
```

---

## 7. 외부 결제 API 연동 (js_11.jsp)

```javascript
// 아임포트(iamport) : 결제 외부 라이브러리 CDN으로 불러옴
var IMP = window.IMP
IMP.init("imp61461128")  // 가맹점 코드 초기화

function requestPay() {
    IMP.request_pay({
        pg: "kakaopay",
        pay_method: "card",
        merchant_uid: "ORD20180131-0000011",  // 주문번호 (서버에서 생성)
        name: '선풍기',
        amount: 30000,
        buyer_email: 'hong@co.kr',
        buyer_name: '홍길동'
    }, function(rsp) {  // 결제 완료 콜백
        alert("구매가 완료되었습니다.")
        // window.location.href = "../mypage/buy_list.do"  // 결제 후 이동
        parent.Shadowbox.close()  // 팝업 닫기
    })
}
// 구조 : CDN 라이브러리 로드 → init() → request_pay() → 콜백 처리
```

---

## 8. 외부 JS 파일 분리 패턴

```text
js_10.jsp
  └── <script src="https://unpkg.com/axios/dist/axios.min.js">  ← axios CDN
  └── <script src="music.js">  ← 외부 JS 파일 불러오기

music.js
  └── window.onload / axios / 이벤트 처리 모두 여기에 작성
```

> 외부 JS에서 `window.onload` 대신 `DOMContentLoaded` 사용 권장  
> `window.onload` : 이미지 포함 모든 리소스 로딩 완료 후 실행  
> `DOMContentLoaded` : HTML 파싱 완료 즉시 실행 (더 빠름)

---

## 9. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `js_1.jsp` | DOM 개념 | JSON 객체 → `querySelector` → `textContent` / `style` |
| `js_2.jsp` | 태그 선택 | `querySelector` / `querySelectorAll` / `getElementById` 비교 |
| `js_3.jsp` | 속성 변경 | `img.src` 변경 + `setInterval` 자동 슬라이드 |
| `js_4.jsp` | 글자 조작 | `textContent` vs `innerHTML` 차이 |
| `js_5.jsp` | 이벤트 개념 | 인라인 이벤트 기초 (`onclick`) |
| `js_6.jsp` | 고전 이벤트 | `onmouseover` / `onmouseout` / `onclick` |
| `js_7.jsp` | 이벤트 리스너 | `addEventListener('click', fn)` ⭐ |
| `js_8.jsp` | 다중 이벤트 | `querySelectorAll` + `mousedown` / `mouseup` |
| `js_9.jsp` | 자손/후손 선택 | `div > p` (자손) vs `div span p` (후손) |
| `js_10.jsp` | 외부 JS 연동 | `music.js` 분리, axios CDN |
| `js_11.jsp` | 외부 결제 API | 아임포트 CDN + `request_pay()` 콜백 |
| `js_12.jsp` | 동적 태그 | `createElement` / `appendChild` / `remove` |
| `music.js` | Axios + 검색 | `keyup` 실시간 검색 / `includes()` 필터링 ⭐ |

---

