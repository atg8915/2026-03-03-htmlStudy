# 📘 Day 19 — JavaScript 기초 + Axios + 영화진흥원 API 연동

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 변수 | `let` / `const` | 변수 / 상수, 자동 타입 추론 |
| 비교 | `===` / `!==` | 값 + 데이터형 동시 비교 (엄격) |
| 배열 순회 | `forEach()` / `map()` | 가장 많이 사용, 화살표 함수와 조합 |
| 배열 조작 | `push()` / `pop()` / `slice()` | 추가 / 삭제 / 자르기 (새 배열 반환) |
| 배열 검색 | `find()` / `filter()` | 단건 반환 / 조건 맞는 배열 반환 |
| 배열 복사 | `[...arr, 새값]` | 스프레드 연산자, React에서 자주 사용 |
| 함수 | `(a,b) => a+b` | 화살표 함수, function/return 생략 |
| 콜백 | `names.map(fn)` | 함수를 매개변수로 전달, 자동 호출 |
| DOM | `querySelector()` | CSS 선택자로 태그 접근 |
| DOM 삽입 | `innerHTML` / `textContent` | HTML 삽입 / 순수 텍스트 삽입 |
| 서버 연동 | `axios.get(url, {params})` | 비동기 HTTP GET 요청 |
| 외부 API | `HttpURLConnection` | Java에서 외부 URL 호출 |

---

## 1. JSP 파일명 규칙

```text
JSP 파일명 → 서블릿 클래스명으로 변환
a b.jsp / a.b.jsp → ❌ (공백, 점 사용 불가)
a_b.jsp / a$b.jsp → ✅ ( _ , $ 만 허용)
```

---

## 2. 배열과 JSON 객체

```javascript
// 배열 [] = Java ArrayList
let names = ["홍길동", "이순신", "강감찬"]
names.push("을지문덕")              // 추가
names.pop()                         // 마지막 삭제
names.slice(1, 3)                   // 인덱스 1~2 새 배열 반환 (끝 미포함)
names.find(n => n === "이순신")     // 단건 반환
names.filter(n => n !== "이순신")   // 조건 맞는 것만 새 배열
let names2 = [...names, "춘향이"]   // 복사 + 추가 (스프레드)

// JSON {} = Java VO
const sawon = {sabun: 1, name: "홍길동", job: "사원"}
sawon.name       // → "홍길동"
sawon["name"]    // → "홍길동" (동적 키 접근 시 사용)

// 객체 배열 [{},{},{}] = Java ArrayList<VO>
let sawons = [
    {sabun: 1, name: "홍길동"},
    {sabun: 2, name: "이순신"}
]
```

---

## 3. 함수

```javascript
// ① 선언적 함수
function plus(a, b) { return a + b }

// ② 익명의 함수
let plus2 = function(a, b) { return a + b }

// ③ 화살표 함수 (function / return 생략) ⭐
let plus3 = (a, b) => a + b

// ④ 콜백 함수 — 함수를 매개변수로 전달
function func(call) {
    for (let i = 1; i <= 10; i++) { call() }
}
func(() => document.write("호출<br>"))

// ⭐ JS에서 함수는 데이터형 → typeof plus === "function"
// setTimer(fn)    → 지정 시간 후 1회 호출
// setInterval(fn) → 주기적 반복 호출
```

---

## 4. Axios + DOM 연동 (js_8.jsp)

```javascript
window.onload = function() {    // Java main()과 동일한 진입점
    axios.get('js_8.do')
         .then(response => {
             let html = ''
             response.data.map((emp) => {
                 html += '<tr>'
                       + '<td>' + emp.empno + '</td>'
                       + '<td>' + emp.ename + '</td>'
                       + '</tr>'
             })
             document.querySelector("tbody").innerHTML = html  // DOM 삽입 ⭐
         })
}
// window.onload = jQuery의 $(function(){}) = Vue의 mounted() = React의 useEffect()
```

---

## 5. 영화진흥원 API 연동 (MovieModel.java + js_14.jsp) ⭐

### 흐름

```text
브라우저 axios.get('movie_list.do?no=1')
  → MovieModel.java : HttpURLConnection으로 kobis.or.kr 호출
  → JSON 문자열 → response.write(json)
  → 브라우저 response.data 수신 → innerHTML로 테이블 출력
```

### MovieModel.java (서버)

```java
@Controller
public class MovieModel {
    private String[] movie = {"",
        "searchMainDailyBoxOffice.do",   // no=1 박스오피스
        "searchMainRealTicket.do",        // no=2 실시간 예매율
        "searchMainDailySeatTicket.do"};  // no=3 좌석 점유율

    @RequestMapping("js/movie_list.do")
    public void movie_list(HttpServletRequest request, HttpServletResponse response) {
        String no = request.getParameter("no");
        URI uri = new URI(baseURL + movie[Integer.parseInt(no)]);
        HttpURLConnection conn = (HttpURLConnection) uri.toURL().openConnection();

        String json = "", s;
        BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        while ((s = in.readLine()) != null) json += s;
        conn.disconnect();  // 연결 해제 필수

        response.setContentType("text/plain;charset=UTF-8");
        response.getWriter().write(json);  // 브라우저로 JSON 전송
    }
}
```

### js_14.jsp (브라우저)

```javascript
let movie = []  // 전역변수 : 목록/상세 공유 ⭐

function movieList(no) {
    axios.get('movie_list.do', { params: { no: no } })
         .then(response => {
             movie = response.data  // 전역 저장 → 상세에서 재사용
             let html = ''
             movie.forEach((m) => {
                 html += '<tr onmouseover="movieDetail(' + m.rank + ')">'
                       + '<td>' + m.rank + '</td>'
                       + '<td>' + m.movieNm + '</td>'
                       + '</tr>'
             })
             document.querySelector('#list tbody').innerHTML = html
         })
}

function movieDetail(mno) {
    let m = movie.find(m => m.rank === mno)  // 서버 재요청 없이 클라이언트에서 처리
    document.querySelector("#name").textContent     = m.movieNm
    document.querySelector("#director").textContent = m.director
    document.querySelector("#poster").src = "https://www.kobis.or.kr" + m.thumbUrl
}
```

---

## 6. README 한 줄

```markdown
| Day 19 | JavaScript 기초(배열/JSON/함수/콜백), Axios 서버 연동, 영화진흥원 API 연동 | [📄 보기](./day_19.md) |
```
