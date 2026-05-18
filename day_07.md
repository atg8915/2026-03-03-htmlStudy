# 📘 Day 07 — JSP 기초 / Servlet 입문

## 0. 핵심 문법 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| 스크립트릿 | `<% int a = 10; %>` | 자바 코드 실행 (지역변수) |
| 표현식 | `<%= a %>` | 브라우저에 값 출력 |
| 선언문 | `<%! int p = 1; %>` | 멤버변수/메소드 선언 |
| JSP 주석 | `<%-- 주석 --%>` | 번역 안됨, 브라우저에 미출력 |
| page 지시어 | `<%@ page import="java.util.*" %>` | import, 인코딩 설정 |
| JSTL forEach | `<c:forEach var="i" begin="1" end="9">` | 반복문 |
| JSTL 출력 | `${i}` | EL 표현식으로 변수 출력 |
| Servlet URL | `@WebServlet("/EmpList")` | URL 매핑 |
| Servlet 출력 | `response.getWriter()` → `out.println()` | HTML 출력 |
| GET 처리 | `doGet()` | 기본 요청 처리 |
| POST 처리 | `doPost()` | 폼 전송 처리 |

---

## 1. JSP란?

> Java + HTML을 합친 동적 페이지 기술
> 브라우저 요청 → JSP → 자바 번역 → HTML 응답

```
정적 페이지 : HTML/CSS → 데이터 변경 불가
동적 페이지 : JSP → 자바 코드로 데이터 변경 가능
```

---

## 2. JSP 스크립트 요소

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" import="java.util.*"%>

<%-- 1. 선언문 : 멤버변수 / 메소드 (사용 빈도 낮음) --%>
<%!
  int p = 1;
  public int add(int a, int b) { return a + b; }
%>

<%-- 2. 스크립트릿 : 자바 코드 실행 (지역변수, 제어문) --%>
<%
  int a = 10;
  int b = 20;
  int c = add(a, b);
%>

<%-- 3. 표현식 : 브라우저에 값 출력 --%>
<%= c %>
<%= p %>
```

> `<%-- --%>` JSP 주석 → 번역 안 됨, 브라우저 미출력
> `<!-- -->` HTML 주석 → 브라우저에 그대로 출력

---

## 3. JSP 제어문 — Java vs JSTL

### Java 방식 (스크립트릿)
```jsp
<table>
  <tr>
  <% for(int i = 2; i <= 9; i++) { %>
    <th><%= i %>단</th>
  <% } %>
  </tr>
  <% for(int i = 1; i <= 9; i++) { %>
    <tr>
      <% for(int j = 2; j <= 9; j++) { %>
        <td><%= j+"*"+i+"="+(j*i) %></td>
      <% } %>
    </tr>
  <% } %>
</table>
```

### JSTL 방식 (권장) ⭐
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<table>
  <tr>
    <c:forEach var="i" begin="2" end="9">
      <th>${i}단</th>
    </c:forEach>
  </tr>
  <c:forEach var="i" begin="1" end="9">
    <tr>
      <c:forEach var="j" begin="2" end="9">
        <td>${j}*${i}=${j*i}</td>
      </c:forEach>
    </tr>
  </c:forEach>
</table>
```

---

## 4. JSP + List — 동적 이미지 배치

```jsp
<%@ page import="java.util.*" %>
<%
  List<String> images = new ArrayList();
  images.add("../images/m1.jpg");
  images.add("../images/m2.jpg");

  List<String> keys = new ArrayList();
  keys.add("a");
  keys.add("b");
%>
<div class="wrap">
  <% for(int i = 0; i < images.size(); i++) { %>
    <img src="<%= images.get(i) %>" class="<%= keys.get(i) %>">
  <% } %>
</div>
```

---

## 5. Servlet 기본 구조

> JSP 이전의 방식 — Java로 HTML을 직접 출력
> 현재는 JSP와 함께 MVC 패턴에서 Controller 역할로 사용

```java
@WebServlet("/EmpList")  // URL 매핑
public class EmpList extends HttpServlet {

  // GET 요청 처리
  protected void doGet(HttpServletRequest request,
                       HttpServletResponse response) throws ... {
    // 1. 응답 형식 설정
    response.setContentType("text/html;charset=UTF-8");
    // 2. 출력 스트림
    PrintWriter out = response.getWriter();
    // 3. DAO 연결 → 데이터 조회
    EmpDAO dao = new EmpDAO();
    List<EmpVO> list = dao.empListData();
    // 4. HTML 출력
    out.println("<table>");
    for(EmpVO vo : list) {
      out.println("<tr><td>" + vo.getEname() + "</td></tr>");
    }
    out.println("</table>");
  }

  // POST 요청 처리 (폼 전송)
  protected void doPost(...) { }

  // 서블릿 생성 시 1회 실행
  public void init(ServletConfig config) { }

  // 서블릿 종료 시 실행
  public void destroy() { }
}
```

### Servlet vs JSP 비교
| 구분 | Servlet | JSP |
|------|---------|-----|
| 주 역할 | Controller (요청 처리) | View (화면 출력) |
| 코드 방식 | Java 안에 HTML | HTML 안에 Java |
| 현재 사용 | MVC Controller | 화면 템플릿 |

---

## 6. CSS 텍스트 속성 정리

| 속성 | 값 | 설명 |
|------|-----|------|
| `color` | `red` / `#FF0000` / `rgb(255,0,0)` | 글자색 |
| `font-size` | `16px` / `2em` / `150%` | 글자 크기 |
| `font-family` | `"맑은 고딕"`, 구글 폰트 | 글꼴 |
| `font-style` | `normal` / `italic` / `oblique` | 기울기 |
| `font-weight` | `bold` / `lighter` / `900` | 두께 |
| `text-align` | `left` / `center` / `right` | 정렬 |
| `text-decoration` | `none` / `underline` / `line-through` | 줄 장식 |
| `letter-spacing` | `5px` / `-2px` | 글자 간격 |
| `white-space` | `nowrap` / `pre-wrap` | 줄바꿈 처리 |

```css
/* 구글 폰트 사용 */
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR&display=swap" rel="stylesheet">
font-family: 'Noto Sans KR', sans-serif;

/* 카드 텍스트 한 줄 처리 */
white-space: nowrap;
overflow: hidden;
```

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `jsp_basic2.jsp` | CSS 텍스트 속성 전체 + JSP 표현식 |
| `jsp_basic3.jsp` | 선언문 + 스크립트릿 + 표현식 |
| `jsp_basic4.jsp` | 구구단 (Java 스크립트릿) |
| `jsp_basic5.jsp` | 구구단 (JSTL forEach) |
| `jsp_basic7.jsp` | List + for문 + position absolute 이미지 배치 |
| `jsp_basic8.jsp` | position:fixed 스크롤 이미지 고정 |
| `jsp_basic9.jsp` | z-index 레이어 겹침 |
| `EmpServlet.java` | Servlet 기본 구조 (init/doGet/doPost/destroy) |
| `EmpList.java` | Servlet + DAO + List → HTML 테이블 출력 |
