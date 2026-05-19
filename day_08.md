# 📘 Day 08 — JSP 내장 객체 / request / float / 페이징

## 0. 핵심 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| 단일값 수신 | `request.getParameter("name")` | text/radio/select/password |
| 다중값 수신 | `request.getParameterValues("hobby")` | checkbox/multiple |
| 한글 변환 | `request.setCharacterEncoding("UTF-8")` | POST 한글 깨짐 방지 |
| 서버 이름 | `request.getServerName()` | localhost |
| URI | `request.getRequestURI()` | /프로젝트명/파일경로 |
| 클라이언트 IP | `request.getRemoteAddr()` | 접속자 IP |
| 데이터 저장 | `request.setAttribute("key", value)` | MVC forward 시 사용 |
| 데이터 읽기 | `request.getAttribute("key")` | MVC에서 데이터 전달 |
| 에러 페이지 | `<%@ page errorPage="error.jsp" %>` | 예외 발생 시 이동 |
| float 좌 | `float: left` | 왼쪽 정렬 (가로 배치) |
| float 우 | `float: right` | 오른쪽 정렬 |
| flex 카드 | `display: flex; flex-wrap: wrap;` | 카드 갤러리 레이아웃 |

---

## 1. JSP 내장 객체 9가지

> `_jspService()` 메소드 안에 자동 생성되어 있는 객체들

| 객체 | 역할 | 주요 사용처 |
|------|------|------------|
| `request` | 사용자 요청 정보 | 파라미터 수신, 서버/브라우저 정보 |
| `response` | 응답 / 화면 이동 | sendRedirect, contentType 설정 |
| `session` | 상태 유지 (서버 저장) | 로그인 정보, 장바구니 |
| `out` | 브라우저 출력 | out.println() |
| `application` | 서버/자원 정보 | 전체 공유 데이터 |
| `pageContext` | include/forward, 내장 객체 읽기 | MVC 전환 |
| `config` | web.xml 설정 읽기 | DB 경로, 계정 정보 |
| `exception` | 예외 처리 | isErrorPage="true" 설정 시 사용 |
| `page` | this (현재 서블릿) | 거의 사용 안 함 |

> ⚠️ `request`/`response`는 화면 이동(새로고침) 시 초기화 — 유지하려면 `session` 또는 `forward` 사용

---

## 2. request 객체 주요 메소드

```jsp
<%-- 서버 정보 --%>
request.getServerName()     <%-- localhost --%>
request.getProtocol()       <%-- HTTP/1.1 --%>
request.getRequestURL()     <%-- 전체 URL --%>
request.getRequestURI()     <%-- /프로젝트명/파일경로 --%>
request.getContextPath()    <%-- /프로젝트명 --%>

<%-- 브라우저 정보 --%>
request.getRemoteAddr()     <%-- 클라이언트 IP ⭐ --%>
request.getServerPort()     <%-- 포트 번호 --%>

<%-- 파라미터 수신 --%>
request.getParameter("name")          <%-- 단일값 ⭐ --%>
request.getParameterValues("hobby")   <%-- 다중값 ⭐ (checkbox) --%>
request.setCharacterEncoding("UTF-8") <%-- 한글 변환 ⭐ --%>

<%-- MVC 데이터 전달 --%>
request.setAttribute("key", value)    <%-- 데이터 저장 ⭐ --%>
request.getAttribute("key")           <%-- 데이터 읽기 ⭐ --%>
```

---

## 3. 폼 → JSP 파라미터 전달 패턴

```jsp
<%-- input.jsp : 폼 작성 --%>
<form method="get" action="output.jsp">
  <input type="text"     name="name">
  <input type="password" name="pwd">
  <input type="radio"    name="sex" value="남자">
  <select name="loc"><option>서울</option></select>
  <textarea name="content"></textarea>
  <input type="checkbox" name="hobby" value="등산">
  <input type="checkbox" name="hobby" value="낚시">
</form>

<%-- output.jsp : 값 수신 --%>
<%
  String name    = request.getParameter("name");
  String sex     = request.getParameter("sex");
  String[] hobby = request.getParameterValues("hobby"); // 다중값

  // 다중값은 반드시 null 체크
  if(hobby != null) {
    for(String h : hobby) {
      out.println("<li>" + h + "</li>");
    }
  }
%>
```

> URL 구조: `output.jsp?name=홍길동&sex=남자` → `?` 뒤 값이 request에 저장됨

---

## 4. 에러 페이지 처리

```jsp
<%-- 에러 발생 페이지 --%>
<%@ page errorPage="error.jsp" %>

<%-- error.jsp : 에러 수신 페이지 --%>
<%@ page isErrorPage="true" %>
<h1><%= exception.getMessage() %></h1>
```

---

## 5. float 레이아웃

> Flex/Grid 이전의 가로 배치 방법 — 댓글창, 갤러리 카드에 활용

```css
/* 기본 float */
.left  { float: left; }
.right { float: right; }

/* 갤러리 카드 */
div.gallery {
  float: left;
  width: 180px;
  margin: 5px;
  border: 1px solid #ccc;
}
div.gallery img {
  width: 100%;
  height: 200px;
}

/* 댓글 입력창 */
textarea { float: left; }
button   { float: left; height: 90px; }
```

---

## 6. Flex 카드 레이아웃 + 페이징

```css
/* 카드 그리드 */
.card-wrap {
  display: flex;
  flex-wrap: wrap;     /* 넘치면 아래로 */
  gap: 20px;
}
.card {
  width: calc(25% - 15px);  /* 한 줄에 4개 */
  border-radius: 12px;
  overflow: hidden;
}
.card img {
  width: 100%;
  height: 220px;
  object-fit: cover;   /* 이미지 꽉 채우기 */
}
/* 텍스트 한 줄 처리 */
.card-title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;  /* ... 말줄임 */
}

/* 페이지네이션 */
.pagination {
  display: flex;
  gap: 10px;
  justify-content: center;
}
.pagination a.active {
  background: #0000FF;
  color: white;
}
```

### 페이징 계산 공식
```java
final int BLOCK = 10;
int startPage = ((curpage - 1) / BLOCK * BLOCK) + 1;  // 1, 11, 21...
int endPage   = ((curpage - 1) / BLOCK * BLOCK) + BLOCK; // 10, 20, 30...
if(endPage > totalpage) endPage = totalpage;
```

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `object_165.jsp` | JSP 내장 객체 9가지 + request 메소드 정리 |
| `input.jsp` | 폼 작성 (Bootstrap + 다양한 input 타입) |
| `output.jsp` | 파라미터 수신 (단일값/다중값/null 체크) |
| `request.jsp` | DAO 연동 카드 목록 + Flex 레이아웃 + 페이징 |
| `error.jsp` | 에러 페이지 (`isErrorPage="true"`) |
| `position.jsp` | CSS position 4종 실습 |
| `float.jsp` | float left/right 기본 |
| `float2.jsp` | float 댓글 입력창 |
| `float3.jsp` | float 갤러리 카드 |
