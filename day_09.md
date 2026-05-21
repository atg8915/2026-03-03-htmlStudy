# 📘 Day 09 — JSP 내장 객체 심화 / Cookie / 파일 다운로드 / Vue+Axios

## 0. 핵심 빠른 참조

| 객체 | 주요 메소드 | 설명 |
|------|------------|------|
| `response` | `sendRedirect("파일명")` | 화면 이동 (request 초기화) |
| `response` | `addCookie(cookie)` | 쿠키 전송 |
| `response` | `setHeader(...)` | 다운로드 헤더 설정 |
| `session` | `setAttribute("key", value)` | 데이터 저장 (로그인) |
| `session` | `getAttribute("key")` | 데이터 읽기 |
| `session` | `invalidate()` | 세션 삭제 (로그아웃) |
| `application` | `getRealPath("/")` | 실제 서버 파일 경로 |
| `application` | `getInitParameter("key")` | web.xml 설정 읽기 |
| `out` | `getBufferSize()` | 전체 버퍼 크기 |
| `out` | `getRemaining()` | 남은 버퍼 크기 |
| `pageContext` | `include("파일명")` | 다른 JSP 포함 |
| `pageContext` | `forward("파일명")` | 화면 이동 (request 유지) |

---

## 1. 내장 객체 범위 비교

| 객체 | 유지 범위 | 공유 여부 |
|------|-----------|-----------|
| `request` | 요청하는 동안 | 같은 JSP (include) |
| `session` | 브라우저 종료 / 로그아웃까지 | 사용자별 유지 |
| `application` | 서버 실행 동안 | 모든 사용자 공유 |
| `page` | 현재 파일 | 자신의 파일만 |

---

## 2. response 객체

```jsp
<%-- 화면 이동 (GET 방식, request 초기화) --%>
response.sendRedirect("list.jsp");
response.sendRedirect("detail.jsp?no=" + no);

<%-- 쿠키 전송 --%>
Cookie cookie = new Cookie("food_" + no, no);
cookie.setMaxAge(60 * 60 * 24);  // 유지 시간 (초)
response.addCookie(cookie);
response.sendRedirect("detail.jsp?no=" + no);

<%-- 파일 다운로드 --%>
response.setHeader("Content-Disposition",
    "attachment;filename=" + URLEncoder.encode(fn, "UTF-8"));
response.setContentLength((int) file.length());
BufferedInputStream  bis = new BufferedInputStream(new FileInputStream(file));
BufferedOutputStream bos = new BufferedOutputStream(response.getOutputStream());
byte[] buffer = new byte[1024];
int i = 0;
while ((i = bis.read(buffer, 0, 1024)) != -1) {
    bos.write(buffer, 0, i);
}
bis.close(); bos.close();
out.clear();
out = pageContext.pushBody();
```

> ⚠️ response는 HTML 전송 / Cookie 전송 중 **한 개만** 가능 (동시 불가)
> ⚠️ `sendRedirect` 는 request 초기화 → 유지하려면 `forward` 사용

---

## 3. session 객체

```jsp
<%-- 로그인 시 세션 저장 --%>
HttpSession session = request.getSession();
session.setAttribute("loginUser", vo);     // 저장
session.setMaxInactiveInterval(60 * 30);   // 유지 시간 (초)

<%-- 다른 페이지에서 읽기 --%>
MemberVO user = (MemberVO) session.getAttribute("loginUser");

<%-- 로그아웃 --%>
session.invalidate();
```

> 세션은 **서버에 저장** → 브라우저 종료 또는 로그아웃 시 소멸
> Cookie는 **브라우저에 저장** → 최근 방문, JWT 등에 활용

---

## 4. application 객체

```jsp
<%-- 서버 정보 --%>
application.getMajorVersion()   <%-- 서블릿 메이저 버전 --%>
application.getServerInfo()     <%-- 서버 이름 --%>

<%-- web.xml 설정 읽기 (DB 계정 보안) --%>
String driver = application.getInitParameter("driver");
String url    = application.getInitParameter("url");
String user   = application.getInitParameter("username");
String pwd    = application.getInitParameter("password");

<%-- 실제 파일 경로 (파일 업로드/다운로드) --%>
String realPath = application.getRealPath("/");
```

---

## 5. out 객체

```jsp
<%-- 버퍼 확인 --%>
out.getBufferSize()                              <%-- 전체 버퍼 크기 (기본 8kb) --%>
out.getRemaining()                               <%-- 남은 버퍼 크기 --%>
out.getBufferSize() - out.getRemaining()         <%-- 사용 중인 버퍼 --%>

<%-- 출력 --%>
out.println("<h1>Hello</h1>");   <%-- HTML 출력 --%>
```

> `<%= %>` → `out.println()` 으로 변환됨
> 최근에는 `${}` (EL 표현식) 으로 대체

---

## 6. pageContext 객체

```jsp
<%-- 다른 JSP 포함 (request 공유) --%>
pageContext.include("header.jsp");

<%-- 화면 이동 (request 유지) --%>
pageContext.forward("list.jsp");
```

### JSP 액션 태그 방식
```jsp
<jsp:include page="header.jsp"/>
<jsp:forward  page="list.jsp"/>
```

### pageContext.include 활용 — 조립식 MVC
```jsp
<%-- main.jsp : mode 값에 따라 다른 JSP 조립 --%>
<%
  String mode = request.getParameter("mode");
  if(mode == null) mode = "1";
  String jsp = "";
  switch(Integer.parseInt(mode)) {
    case 1: jsp = "home.jsp";      break;
    case 2: jsp = "script.jsp";    break;
    case 3: jsp = "directive.jsp"; break;
    case 4: jsp = "object.jsp";    break;
  }
%>
<body>
  <% pageContext.include("header.jsp"); %>
  <% pageContext.include(jsp); %>
</body>
```

---

## 7. JSP 지시자 정리

| 지시자 | 용도 |
|--------|------|
| `page` | 페이지 전체 속성 설정 |
| `include` | 다른 JSP 정적 포함 |
| `taglib` | JSTL 등 태그 라이브러리 사용 |

### page 지시자 주요 속성
| 속성 | 설명 | 예시 |
|------|------|------|
| `contentType` | 응답 형식 | `text/html` / `text/plain`(JSON) / `text/xml` |
| `import` | 클래스 임포트 | `java.util.*,com.sist.dao.*` |
| `errorPage` | 에러 발생 시 이동할 파일 | `error.jsp` |
| `buffer` | 출력 버퍼 크기 | `16kb` (기본 8kb) |

---

## 8. Vue 3 + Axios — JSP 데이터 연동

```javascript
// food.jsp — Vue 3 + Axios로 find.jsp에서 JSON 받기
let app = Vue.createApp({
  data() {
    return { list: [], curpage: 1, totalpage: 0, address: '마포' }
  },
  mounted() {
    this.dataRecv();  // 페이지 로딩 시 자동 실행
  },
  methods: {
    dataRecv() {
      axios.get('find.jsp', {
        params: { page: this.curpage, address: this.address }
      }).then(response => {
        this.list      = response.data.list;
        this.curpage   = response.data.curpage;
        this.totalpage = response.data.totalpage;
      });
    },
    find() {
      this.curpage = 1;
      this.dataRecv();
    }
  }
}).mount('.container');
```

```jsp
<%-- find.jsp — JSON 응답 (contentType: text/plain) --%>
<%@ page contentType="text/plain; charset=UTF-8" %>
<%
  Map map = new HashMap();
  map.put("curpage",   curpage);
  map.put("totalpage", totalpage);
  map.put("list",      list);
  ObjectMapper mapper = new ObjectMapper();
  String json = mapper.writeValueAsString(map);
%>
<%= json %>
```

> Ajax/Vue에서 JSON을 받으려면 `contentType="text/plain"` 설정 필수

---

## 9. sendRedirect vs forward 비교

| 구분 | `sendRedirect` | `forward` (pageContext) |
|------|---------------|------------------------|
| request 유지 | ❌ 초기화 | ✅ 유지 |
| URL 변경 | ✅ 변경됨 | ❌ 변경 안 됨 |
| 방식 | GET | 서버 내부 이동 |
| 용도 | 로그인 후 이동, 등록 후 목록 | MVC Controller → View |

---

## 10. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `response.jsp` | request/response 내장 객체 전체 정리 |
| `object.jsp` | 내장 객체 9가지 역할/범위/메소드 정리 |
| `out.jsp` | out 버퍼 크기 확인, println/write 차이 |
| `application.jsp` | 서버 정보, web.xml 읽기, getRealPath |
| `pageContext.jsp` | include/forward 개념 |
| `main.jsp` | mode 파라미터로 JSP 조립식 화면 전환 |
| `header.jsp` | 공통 네비게이션 바 (Bootstrap) |
| `directive.jsp` | page 지시자 속성 정리 |
| `script.jsp` | JSP 스크립트 요소 정리 |
| `list.jsp` | DAO 연동 카드 목록 + 이전/다음 페이징 |
| `detail.jsp` | 상세보기 (rowspan 레이아웃) |
| `detail_before.jsp` | Cookie 저장 후 detail로 redirect |
| `file.jsp` + `download.jsp` | 파일 다운로드 구현 |
| `find.jsp` | JSON 응답 (ObjectMapper) |
| `food.jsp` | Vue 3 + Axios 검색 + 목록 출력 |
