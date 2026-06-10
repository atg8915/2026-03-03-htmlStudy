# 📘 Day 21 — jQuery 기초 + JS 내장 객체 + 우편번호 팝업

## 0. 핵심 빠른 참조

| 구분 | 바닐라 JS | jQuery |
|------|----------|--------|
| 진입점 | `window.onload = () => {}` | `$(function(){})` |
| 태그 선택 (1개) | `document.querySelector("선택자")` | `$("선택자")` |
| 태그 선택 (다중) | `document.querySelectorAll("선택자")` | `$("선택자")` (동일) |
| 텍스트 읽기/쓰기 | `el.textContent` | `$().text()` |
| HTML 읽기/쓰기 | `el.innerHTML` | `$().html()` |
| 값 읽기/쓰기 | `el.value` | `$().val()` |
| 속성 변경 | `img.src = ""` | `$().attr("src", "")` |
| 스타일 변경 | `el.style.color = "red"` | `$().css("color","red")` |
| 태그 추가 | `부모.appendChild(자식)` | `$().append("HTML")` |
| 태그 삭제 | `el.remove()` | `$().remove()` |
| 클래스 추가/삭제 | — | `$().addClass()` / `$().removeClass()` / `$().toggleClass()` |
| 이벤트 (권장) | `el.addEventListener('click', fn)` | `$().on('click', fn)` ⭐ |
| hover | mouseover + mouseout 별도 | `$().hover(fn, fn)` |
| 숫자 천단위 | — | `number.toLocaleString()` |
| 자동 스크롤 | — | `$().scrollTop(값)` |
| 포커스 | `el.focus()` | `$().focus()` |

---

## 1. jQuery 란

```text
바닐라 JS의 복잡한 DOM 코드를 $ 하나로 통일한 라이브러리
"Write less, do more"

document.querySelector("#btn")  →  $("#btn")
document.querySelectorAll("h3") →  $("h3")   (동일 문법으로 다중 선택도 가능)

⚠️ 버전 충돌 시 작동 안 함 → 프로젝트당 버전 하나만 사용
⚠️ main.jsp에 CDN 한 번만 포함

// jQuery 3 CDN
<script src="http://code.jquery.com/jquery.js"></script>
// jQuery 4 CDN  
<script src="http://code.jquery.com/jquery-4.0.0.min.js"></script>
```

---

## 2. 선택자 + 스타일 조작 (jquery_1.jsp)

```javascript
$(function() {
    // 단일 선택 : id / class / tag
    $('#h1').css("color", "yellow")
    $('.h1').css("backgroundColor", "cyan")

    // 인덱스 선택 : eq(n) → n번째 (0부터)
    $('h2:eq(0)').css("color", "magenta")
                 .css("backgroundColor", "black")   // 체이닝

    // 여러 속성 한 번에 : 객체 {} 전달 (jQuery 4 권장 방식)
    $('h2:eq(1)').css({
        "color": "pink",
        "backgroundColor": "blue"
    })

    // 다중 태그 한 번에
    $('h3').css({ "backgroundColor": "blue", "color": "white" })

    // hover : mouseover + mouseout 묶음
    $('img').hover(function() {
        $(this).css({ "cursor": "pointer", "border": "3px solid green" })
    }, function() {
        $(this).css({ "cursor": "none", "border": "none" })
    })
})
```

---

## 3. 문자 / 속성 / 값 조작 (jquery_2.jsp ~ jquery_3.jsp)

```javascript
// 텍스트 읽기(getter) / 쓰기(setter)
$('h1').text()           // getter
$('h1').text("새 텍스트") // setter

// HTML 읽기/쓰기
$('h2').html('<font color=green>jQuery</font>')

// 속성 변경
$('img').attr('src', '이미지경로')   // img.src = "" 와 동일

// 값 읽기/쓰기 (input / select / textarea)
$("input[type='text']").val()         // getter
$("input[type='text']").val("hong")   // setter

// ⭐ getter/setter 패턴 정리
// text()     → getter  /  text("값")   → setter
// html()     → getter  /  html("값")   → setter
// val()      → getter  /  val("값")    → setter
// attr("속성") → getter /  attr("속성","값") → setter
```

---

## 4. 동적 태그 추가/삭제 (jquery_4.jsp)

```javascript
// append() : 자식 마지막에 HTML 추가 (innerHTML과 차이 : 누적 가능)
// html()   : 기존 내용 덮어쓰기 (1회성)
// append() : 기존 내용 유지하며 추가 ⭐

let fileIndex = 0
$(function() {
    $('#add').on('click', function() {
        $('#user-table tbody').append(
            '<tr id="m' + fileIndex + '">'
          + '<td>File ' + (fileIndex + 1) + '</td>'
          + '<td><input type=file></td>'
          + '</tr>'
        )
        fileIndex++
    })

    $('#remove').on('click', function() {
        if (fileIndex > 0) {
            $('#m' + (fileIndex - 1)).remove()
            fileIndex--
        }
    })
})
```

---

## 5. 숫자 포맷 + 이벤트 실습 (jquery_5.jsp ~ jquery_6.jsp)

```javascript
// 숫자 천단위 콤마
let total = 1234567
$('#total').text(total.toLocaleString() + "원")  // → "1,234,567원"

// keydown 이벤트 + Enter 감지 + 채팅 구현
$('#sendMsg').on('keydown', function(key) {
    if (key.keyCode === 13) {            // Enter키 = 13
        // jQuery 4에서는 key.key === 'Enter'
        let msg = $(this).val()
        if (msg.trim() === "") {         // trim() : 좌우 공백 제거
            $(this).focus()
            return
        }
        $('#recvMsg').append(msg + "<br>")
        $(this).val("").focus()          // 입력창 초기화 + 포커스 복귀

        // 스크롤 자동 하단 이동
        let ch = $('#chatArea').height()
        let m  = $('#recvMsg').height() - ch
        $('#chatArea').scrollTop(m)
    }
})
```

| jQuery 이벤트 (고전) | jQuery 이벤트 (리스너 ⭐) | 설명 |
|---------------------|------------------------|------|
| `$().click(fn)` | `$().on('click', fn)` | 클릭 |
| `$().keyup(fn)` | `$().on('keyup', fn)` | 키보드 뗌 |
| `$().keydown(fn)` | `$().on('keydown', fn)` | 키보드 누름 |
| `$().change(fn)` | `$().on('change', fn)` | 값 변경 |
| `$().hover(fn,fn)` | — | mouseover + mouseout |

---

## 6. JS 내장 객체 정리 (js_2.jsp)

### String
```javascript
let str = 'red,black,green'
str.length                    // 문자 개수
str.indexOf("black")          // 위치 찾기 (없으면 -1)
str.replace("red", "blue")    // 첫 번째 일치 변경
str.replaceAll("a", "!")      // 전체 변경
str.split(",")                // → ["red","black","green"] 배열로 분리
str.substring(0, 3)           // 인덱스 0~2 자르기 (끝 미포함)
str.trim()                    // 좌우 공백 제거
str.includes("black")         // 포함 여부 true/false
str.startsWith("red")         // 시작 여부
str.endsWith("green")         // 끝 여부
```

### Number / Math / Date
```javascript
Number("10")             // 문자 → 숫자
parseInt("10.5")         // → 10 (정수만)
(1234567).toLocaleString() // → "1,234,567" 천단위 콤마

Math.round(4.5)          // → 5 반올림
Math.ceil(4.1)           // → 5 올림
Math.floor(4.9)          // → 4 내림

let today = new Date()
today.getFullYear()      // 연도
today.getMonth() + 1     // 월 (0부터 시작이므로 +1)
today.getDate()          // 일
today.getDay()           // 요일 (0=일, 1=월 ... 6=토)
// 요일 배열 활용
let strWeek = ["일","월","화","수","목","금","토"]
strWeek[today.getDay()]
```

### BOM (브라우저 내장 객체)
```javascript
window.open("url", "팝업명", "width=530,height=450,scrollbars=yes")  // 팝업 열기
window.close()          // 현재 창 닫기 (self.close())
location.href = "url"   // 화면 이동 (sendRedirect와 동일)
history.back()          // 이전 페이지 (go(-1))
```

---

## 7. 성적 계산 + onchange 실습 (js_1.jsp)

```javascript
function gesan() {
    let kor = document.getElementById("kor")
    if (kor.value === "") { kor.focus(); return }  // 빈값 검사 → focus

    let hap = Number(kor.value) + parseInt(eng.value) + parseInt(math.value)
    // ⭐ 웹에서 input 값은 항상 String → Number() / parseInt()로 변환 필수
    document.getElementById("total").value = hap

    let av = hap / 3
    document.getElementById("avg").value = Math.round(av)

    let score = av >= 90 ? 'A' : av >= 80 ? 'B' : av >= 70 ? 'C' : av >= 60 ? 'D' : 'F'
    document.getElementById('score').value = score
}

// onchange : select 값 변경 시 → 장바구니 수량 계산에 활용
document.querySelector("#account").addEventListener('change', function() {
    let total = Number(this.value) * Number(document.querySelector("#price").textContent)
    document.querySelector('#sum').innerHTML = '<font color=red>' + total + '원</font>'
})
```

---

## 8. 우편번호 검색 팝업 (join.jsp + postfind.jsp)

```javascript
// ① 부모창 : 팝업 열기
function postfind() {
    window.open('postfind.do', 'postfind', 'width=530,height=450,scrollbars=yes')
    // window.open(URL, 창이름, 옵션)
}

// ② 팝업창 : 선택 시 부모창으로 값 전달 후 닫기
function ok(zip, addr) {
    opener.frm.post1.value = zip.substring(0, 3)   // 111
    opener.frm.post2.value = zip.substring(4, 7)   // 111
    opener.frm.addr.value  = addr
    self.close()   // 팝업 닫기
    // opener : 부모창 참조 / self : 현재 팝업창 참조
}
```

```text
흐름 요약
join.jsp (부모창)
  └─ 우편번호 검색 버튼 클릭 → window.open('postfind.do')
       └─ postfind.jsp (팝업)
            └─ 동/읍/면 입력 → 서버 검색 → 목록 출력
                 └─ 주소 클릭 → ok(zip, addr) 호출
                      └─ opener.frm에 값 주입 → self.close()
```

---

## 9. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `jquery_1.jsp` | jQuery 기초 | `$()`, `css()`, `eq()`, `hover()` |
| `jquery_2.jsp` | 문자 조작 | `text()` / `html()` getter·setter |
| `jquery_3.jsp` | 속성/값 조작 | `attr()`, `val()` |
| `jquery_4.jsp` | 동적 추가/삭제 | `append()` / `remove()` |
| `jquery_5.jsp` | 숫자 포맷 | `toLocaleString()` |
| `jquery_6.jsp` | 이벤트 리스너 | `on('keydown')`, Enter 감지, 채팅 스크롤 |
| `js_1.jsp` | DOM 종합 실습 | 성적 계산, `onchange` 수량 계산 |
| `js_2.jsp` | JS 내장 객체 | String / Number / Date / BOM 정리 |
| `join.jsp` | 팝업 연동 | `window.open()` 우편번호 검색 팝업 |
| `postfind.jsp` | 팝업 결과 전달 | `opener.frm`, `self.close()`, `substring()` |

---
