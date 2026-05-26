# 📘 Day 11 — EL / JSTL / MV구조 게시판 / 파일 업로드

## 0. 핵심 빠른 참조

| 구문 | 예시 | Java 대응 |
|------|------|-----------|
| EL 출력 | `${name}` | `<%= request.getAttribute("name") %>` |
| EL param | `${param.id}` | `request.getParameter("id")` |
| EL paramValues | `${paramValues.hobby[0]}` | `request.getParameterValues("hobby")[0]` |
| EL 객체 | `${vo.name}` | `vo.getName()` |
| EL Map | `${map.id}` | `map.get("id")` |
| EL List | `${list[0]}` | `list.get(0)` |
| EL empty | `${empty list}` | `list == null || list.isEmpty()` |
| EL 삼항 | `${age>=20?"성인":"미성년"}` | Java 삼항 동일 |
| requestScope | `${requestScope.name}` | `request.getAttribute("name")` |
| sessionScope | `${sessionScope.id}` | `session.getAttribute("id")` |
| applicationScope | `${applicationScope.count}` | `application.getAttribute("count")` |
| c:set | `<c:set var="msg" value="Hello"/>` | 변수 선언 |
| c:if | `<c:if test="${age ge 20}">` | if문 |
| c:choose | `<c:choose><c:when><c:otherwise>` | if-else |
| c:forEach | `<c:forEach var="vo" items="${list}">` | for-each |
| c:forTokens | `<c:forTokens items="a,b,c" delims=",">` | StringTokenizer |
| c:redirect | `<c:redirect url="list.jsp"/>` | sendRedirect |
| fmt:formatDate | `<fmt:formatDate value="${today}" pattern="yyyy-MM-dd"/>` | SimpleDateFormat |
| fmt:formatNumber | `<fmt:formatNumber value="1234567" type="currency"/>` | DecimalFormat |
| fn:length | `${fn:length(msg)}` | `msg.length()` |
| fn:substring | `${fn:substring(msg,0,3)}` | `msg.substring(0,3)` |
| fn:replace | `${fn:replace(msg,"홍","심")}` | `msg.replace("홍","심")` |

---

## 1. EL (Expression Language)

> `<%= %>` 를 대체하는 **출력 전용** 표현식
> JSP 안에서 Java 코드를 최소화하고 데이터를 간단하게 출력

```
EL 탐색 순서 (키 이름이 같은 경우)
pageScope → requestScope → sessionScope → applicationScope
```

### EL 사용 준비
```jsp
<%-- request/session에 담아야 ${} 로 출력 가능 --%>
<%-- 일반 지역변수(String name="홍길동")는 ${name} 출력 불가 --%>
<%
  request.setAttribute("name", "홍길동");   // ${name}
  session.setAttribute("id", "hong");       // ${sessionScope.id}
%>
```

### EL 연산자
```jsp
<%-- 산술 --%>
${10 + 20}         <%-- 30 --%>
${10 / 3}          <%-- 3.333... (정수/정수=실수) --%>
${10 div 3}        <%-- 동일 --%>
${10 % 3}          <%-- 1 --%>
${10 mod 3}        <%-- 동일 --%>
${"10" + 10}       <%-- 20 (자동 정수변환) --%>
${"10" += "20"}    <%-- 1020 (문자열 결합은 +=) --%>
${10 + null}       <%-- 10 (null은 0으로 인식) --%>

<%-- 비교 (문자/숫자/날짜 모두 사용 가능) --%>
${10 == 10}  ${10 eq 10}   <%-- true --%>
${10 != 10}  ${10 ne 10}   <%-- false --%>
${10 > 5}    ${10 gt 5}    <%-- true --%>
${10 < 5}    ${10 lt 5}    <%-- false --%>
${10 >= 10}  ${10 ge 10}   <%-- true --%>
${10 <= 10}  ${10 le 10}   <%-- true --%>

<%-- 논리 --%>
${10==10 and 5>3}   <%-- true (둘 다 참) --%>
${10==10 or 5<3}    <%-- true (하나만 참) --%>
${not(10==10)}      <%-- false --%>

<%-- empty : null이거나 비어있으면 true --%>
${empty list}       <%-- list가 null이거나 size==0 이면 true --%>
${not empty list}   <%-- 데이터가 있으면 true --%>

<%-- 삼항 --%>
${age >= 20 ? "성인" : "미성년"}
```

### EL 내장 객체
```jsp
${requestScope.name}       <%-- request.getAttribute("name") --%>
${sessionScope.id}         <%-- session.getAttribute("id") (생략 불가) --%>
${applicationScope.count}  <%-- application.getAttribute("count") --%>
${param.id}                <%-- request.getParameter("id") --%>
${paramValues.hobby[0]}    <%-- request.getParameterValues("hobby")[0] --%>
${pageContext.request.contextPath}  <%-- 루트 경로 (CSS/JS 경로에 활용) --%>
```

### 과거 방식 vs 현재 방식 비교
```jsp
<%-- 과거 방식 --%>
<%
  String id = request.getParameter("id");
  String[] hobby = request.getParameterValues("hobby");
  for(String h : hobby) { out.println(h); }
%>
ID: <%= id %>

<%-- 현재 방식 (EL) --%>
ID: ${param.id}
취미: ${paramValues.hobby[0]} ${paramValues.hobby[1]}
```

---

## 2. JSTL (Java Standard Tag Library)

> `<% %>` 를 대체하는 **제어문/기능 처리** 태그 라이브러리
> JSTL은 XML 형식 → 대소문자 구분, 반드시 닫는 태그 필요

### taglib 선언
```jsp
<%@ taglib prefix="c"   uri="jakarta.tags.core" %>       <%-- 핵심 --%>
<%@ taglib prefix="fmt" uri="jakarta.tags.fmt" %>         <%-- 날짜/숫자 --%>
<%@ taglib prefix="fn"  uri="jakarta.tags.functions" %>   <%-- 문자열 --%>
```

### core (c:) — 핵심 태그
```jsp
<%-- 변수 선언 --%>
<c:set var="name" value="홍길동"/>
<c:set var="list" value="<%= names %>"/>
<c:remove var="name"/>

<%-- 조건문 (else 없음) --%>
<c:if test="${age ge 20}">성인</c:if>
<c:if test="${age lt 20}">미성년</c:if>

<%-- 다중 조건문 (if-else) --%>
<c:choose>
  <c:when test="${score ge 90}">A</c:when>
  <c:when test="${score ge 80}">B</c:when>
  <c:otherwise>F</c:otherwise>
</c:choose>

<%-- 별점 예제 --%>
<c:set var="star" value="3"/>
<c:choose>
  <c:when test="${star==1}">★☆☆☆☆</c:when>
  <c:when test="${star==2}">★★☆☆☆</c:when>
  <c:when test="${star==3}">★★★☆☆</c:when>
  <c:when test="${star==5}">★★★★★</c:when>
</c:choose>

<%-- 반복문 : 숫자 범위 --%>
<c:forEach var="i" begin="1" end="10" step="1">
  ${i}&nbsp;
</c:forEach>

<%-- 반복문 : List/배열 (varStatus로 인덱스) --%>
<c:forEach var="vo" items="${list}" varStatus="s">
  ${s.index+1}. ${vo.name}
</c:forEach>

<%-- 구구단 --%>
<c:forEach var="i" begin="1" end="9">
  <tr>
    <c:forEach var="j" begin="2" end="9">
      <td>${j}*${i}=${i*j}</td>
    </c:forEach>
  </tr>
</c:forEach>

<%-- 문자열 분리 (StringTokenizer 대체) --%>
<c:forTokens items="red,blue,green" delims="," var="color">
  <li>${color}</li>
</c:forTokens>

<%-- 화면 이동 (sendRedirect 대체) --%>
<c:redirect url="list.jsp"/>
```

### fmt: — 날짜/숫자 변환
```jsp
<%-- 날짜 변환 (주로 오라클 TO_CHAR로 처리) --%>
<c:set var="today" value="<%= new Date() %>"/>
<fmt:formatDate value="${today}" pattern="yyyy-MM-dd"/>

<%-- 숫자 변환 --%>
<fmt:formatNumber value="1234567" type="currency"/>   <%-- ₩1,234,567 --%>
<fmt:setLocale value="en_US"/>
<fmt:formatNumber value="1234567" type="currency"/>   <%-- $1,234,567 --%>
<fmt:formatNumber value="0.75"    type="percent"/>    <%-- 75% --%>
```

### fn: — 문자열 함수
```jsp
<c:set var="msg" value="홍길동입니다"/>
${fn:length(msg)}           <%-- 7 --%>
${fn:substring(msg, 0, 3)}  <%-- 홍길동 --%>
${fn:replace(msg, "홍", "심")} <%-- 심길동입니다 --%>
```

---

## 3. MV 구조 게시판

> Java(Model)와 HTML(View)을 분리 → 협업 가능한 구조
> JSP 안에서 Java 코드를 최대한 줄인다

```
MV 구조
JSP(View) ←→ Model(Java) ←→ DAO ←→ Oracle

MVC 구조 (Spring)
JSP(View) ← Controller(Servlet) → Model → DAO → Oracle
```

### BoardModel.java — 비즈니스 로직 분리
```java
public class BoardModel {
  // 목록 데이터를 request에 담아서 JSP로 전달
  public void boardListData(HttpServletRequest request) {
    String strPage = request.getParameter("page");
    if(strPage == null) strPage = "1";
    int curpage = Integer.parseInt(strPage);

    BoardDAO dao = BoardDAO.newInstance();
    List<BoardVO> list = dao.boardListData(curpage);
    int totalpage = dao.boardTotalPage();

    // JSP에서 ${list}, ${curpage}, ${totalpage} 로 사용
    request.setAttribute("list",      list);
    request.setAttribute("curpage",   curpage);
    request.setAttribute("totalpage", totalpage);

    // 오늘 날짜 → 새글 표시에 활용
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    request.setAttribute("today", sdf.format(new Date()));
  }

  // 글쓰기 처리 후 목록으로 이동
  public void boardInsert(HttpServletRequest request,
                          HttpServletResponse response) {
    BoardVO vo = new BoardVO();
    vo.setName(request.getParameter("name"));
    vo.setSubject(request.getParameter("subject"));
    vo.setContent(request.getParameter("content"));
    vo.setPwd(request.getParameter("pwd"));

    BoardDAO dao = BoardDAO.newInstance();
    dao.boardInsert(vo);
    response.sendRedirect("list.jsp");
  }

  // 상세보기 : 조회수 증가 후 데이터 반환
  public void boardDetailData(HttpServletRequest request) {
    int no = Integer.parseInt(request.getParameter("no"));
    BoardDAO dao = BoardDAO.newInstance();
    BoardVO vo = dao.boardDetailData(no);
    request.setAttribute("vo", vo);  // ${vo.no} ${vo.subject} ...
  }
}
```

### BoardDAO.java — DB 처리
```java
// 목록 조회 (페이징)
String sql = "SELECT no, subject, name, TO_CHAR(regdate,'yyyy-mm-dd'), hit "
           + "FROM jspBoard ORDER BY no DESC "
           + "OFFSET ? ROWS FETCH NEXT 10 ROWS ONLY";
ps.setInt(1, (page * 10) - 10);  // 1페이지=0, 2페이지=10, 3페이지=20

// 총 페이지 수
String sql = "SELECT CEIL(COUNT(*)/10.0) FROM jspboard";

// 글쓰기 (no는 SEQUENCE 자동증가)
String sql = "INSERT INTO jspboard(name, subject, content, pwd) VALUES(?,?,?,?)";

// 상세보기 (조회수 증가 후 SELECT)
String sql = "UPDATE jspBoard SET hit=hit+1 WHERE no=?";
sql = "SELECT no, name, subject, content, hit, "
    + "TO_CHAR(regdate,'yyyy-mm-dd hh24:mi:ss') FROM jspBoard WHERE no=?";
```

### list.jsp — JSTL + EL로 화면 출력
```jsp
<%@ taglib prefix="c" uri="jakarta.tags.core"%>
<jsp:useBean id="model" class="com.sist.model.BoardModel"/>
<% model.boardListData(request); %>

<%-- 목록 출력 --%>
<c:forEach var="vo" items="${list}">
  <tr>
    <td>${vo.no}</td>
    <td>
      <a href="detail.jsp?no=${vo.no}">${vo.subject}</a>
      <%-- 오늘 등록된 글에 NEW 표시 --%>
      <c:if test="${today == vo.dbday}">
        <sup><img src="new.gif"></sup>
      </c:if>
    </td>
    <td>${vo.name}</td>
    <td>${vo.dbday}</td>
    <td>${vo.hit}</td>
  </tr>
</c:forEach>

<%-- 페이징 (삼항 연산자로 범위 제한) --%>
<a href="list.jsp?page=${curpage>1 ? curpage-1 : curpage}">이전</a>
${curpage} page / ${totalpage} pages
<a href="list.jsp?page=${curpage<totalpage ? curpage+1 : curpage}">다음</a>
```

### detail.jsp — 삭제 버튼 토글 (jQuery)
```javascript
let i = 0;
$('.delbtn').click(function() {
  if(i === 0) {
    i = 1;
    $(this).text("취소");
    $('#del').show();    // 비밀번호 입력창 표시
  } else {
    i = 0;
    $(this).text("삭제");
    $('#del').hide();
  }
});
```

---

## 4. 파일 업로드

> `multipart/form-data` 방식으로 파일 + 텍스트 동시 전송
> Servlet의 `@MultipartConfig` 어노테이션 필수

### HTML 폼
```html
<form method="post" action="UploadServlet" enctype="multipart/form-data">
  이름: <input type="text" name="name">
  파일: <input type="file" name="upload">
  <button>업로드</button>
</form>
```

### Servlet 파일 업로드 처리
```java
@WebServlet("/UploadServlet")
@MultipartConfig  // ← 파일 업로드 필수 어노테이션
public class UploadServlet extends HttpServlet {

  private static final String UPLOAD_DIR = "uploads";

  protected void doPost(HttpServletRequest request,
                        HttpServletResponse response) {
    // 1. 업로드 경로 설정
    String uploadPath = getServletContext().getRealPath("")
                      + File.separator + UPLOAD_DIR;

    // 2. 폴더 없으면 생성
    File uploadDir = new File(uploadPath);
    if(!uploadDir.exists()) uploadDir.mkdir();

    try {
      // 3. 한글 처리
      request.setCharacterEncoding("UTF-8");

      // 4. 텍스트 데이터 받기
      String name    = request.getParameter("name");
      String subject = request.getParameter("subject");

      // 5. 파일 데이터 받기 ⭐
      Part filePart = request.getPart("upload");

      DataBoardVO vo = new DataBoardVO();
      vo.setName(name);
      vo.setSubject(subject);

      if(filePart == null || filePart.getSize() == 0) {
        // 파일 없는 경우
        vo.setFilename("");
        vo.setFilesize(0);
      } else {
        // 6. 파일명 추출
        String fileName = filePart.getSubmittedFileName();
        // 7. 서버에 저장 ⭐
        filePart.write(uploadPath + File.separator + fileName);
        // 8. 파일 크기 저장
        File f = new File(uploadPath + File.separator + fileName);
        vo.setFilename(fileName);
        vo.setFilesize((int) f.length());
      }

      // 9. DB 저장
      DataBoardDAO dao = DataBoardDAO.newInstance();
      dao.databoardInsert(vo);

      // 10. 페이지 이동
      response.sendRedirect("list.jsp");

    } catch(Exception e) {
      e.printStackTrace();
    }
  }
}
```

### 파일 업로드 전체 흐름
```
1. 사용자 form 전송 (multipart/form-data)
2. Servlet에서 데이터 수신
   - request.getParameter()   → 텍스트 데이터
   - request.getPart()        → 파일 데이터 ⭐
3. filePart.write()           → 서버 폴더에 저장
4. dao.insert(vo)             → DB에 파일명/크기 저장
5. response.sendRedirect()    → 완료 후 이동
```

### 핵심 메소드 정리
| 메소드 | 설명 |
|--------|------|
| `request.getPart("name")` | 파일 영역 추출 |
| `filePart.getSubmittedFileName()` | 원본 파일명 |
| `filePart.getSize()` | 파일 크기 (byte) |
| `filePart.write(경로)` | 서버에 파일 저장 ⭐ |
| `getServletContext().getRealPath("")` | 서버 실제 경로 |
| `File.separator` | OS 구분자 자동 처리 (윈도우`\` / 리눅스`/`) |

---

## 5. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `el.jsp` | EL 전체 개념 + 연산자 정리 |
| `el_1.jsp` | VO 객체 → request.setAttribute → EL 출력 |
| `el_2.jsp` | requestScope / sessionScope / applicationScope 비교 |
| `el_3.jsp` | pageContext.request.contextPath 루트 경로 |
| `input.jsp` / `output.jsp` | 폼 전송 → param / paramValues EL 수신 |
| `core.jsp` | JSTL core 태그 전체 정리 |
| `for.jsp` | c:forEach 숫자 반복 |
| `for2.jsp` | c:forEach List 반복 + 구구단 |
| `if.jsp` | c:if / c:choose / c:when + 별점 예제 |
| `fortoken.jsp` | c:forTokens (StringTokenizer 대체) |
| `fmt.jsp` | fmt:formatDate / fmt:formatNumber |
| `fn.jsp` | fn:length / fn:substring / fn:replace |
| `list.jsp` | MV구조 게시판 목록 + 페이징 + 새글 표시 |
| `detail.jsp` | 게시판 상세보기 + 삭제 토글 (jQuery) |
| `insert.jsp` | 글쓰기 폼 |
| `insert_ok.jsp` | BoardModel.boardInsert() 호출 |
| `BoardDAO.java` | DBCP JNDI + 목록/총페이지/글쓰기/상세보기 |
| `BoardModel.java` | MV 구조 비즈니스 로직 분리 |
| `el-jstl-upload.docx` | EL/JSTL/파일업로드 전체 개념 정리 교재 |
