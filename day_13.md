# 📘 Day 13 — 답글형 게시판 CRUD / JSP 총정리

## 0. 핵심 빠른 참조

| 구분 | 설명 |
|------|------|
| GET | URL 뒤에 데이터 노출 — `<a>`, `location.href`, `sendRedirect()` |
| POST | 데이터 감춰서 전송 — `<form>`, `ajax`, `axios`, `fetch` |
| GET 용도 | 검색어, 페이지 번호, 상세보기 번호 |
| POST 용도 | 데이터가 많거나 보안 필요한 경우 (ID, 비밀번호) |

---

## 1. JSP 전체 기술 스택 총정리

```
[사용자 브라우저]
    ↓ 요청
[Controller (Servlet / Model)]
    ↓ 호출
[Service / Model]
    ↓ 호출
[DAO]
    ↓ Connection 요청
[DBCP — Connection Pool]
    ↓ JDBC로 SQL 실행
[Database (Oracle)]
    ↓ 결과 반환
[DAO → Model → Controller]
    ↓ request.setAttribute()
[View (JSP)]
    ↓ EL / JSTL 출력
[브라우저 화면]
```

| 기술 | 한 줄 설명 | 역할 |
|------|-----------|------|
| JDBC | Java가 DB에 접속하는 표준 API | DAO 내부 SQL 실행 |
| DBCP | Connection 재사용 Pool 기술 | Connection 비용 절감 |
| EL `${}` | Java 코드 없이 값 출력 | View (JSP) |
| JSTL | 반복문/조건문 태그 처리 | View (JSP) |
| MVC | 역할 분리 설계 패턴 | 전체 구조 |

> 중요도 순서: **MVC > JDBC > DBCP > EL/JSTL**

---

## 2. JSP 출력 방식 3가지 비교

| 방식 | XSS 위험 | 가독성 | MVC 적합 | 사용 권장 |
|------|----------|--------|----------|-----------|
| `<%= %>` | ❌ 매우 위험 | 낮음 | ❌ | ❌ 비추천 |
| `${}` EL | ⚠️ 보통 | 높음 | ✅ | ✅ 기본 선택 |
| `<c:out>` | ✅ 안전 | 높음 | ✅ | ✅ 보안 필요 시 |

```jsp
<%-- 구식 — 비추천 --%>
<%= request.getParameter("name") %>

<%-- EL — 기본 선택 --%>
${param.name}

<%-- c:out — 사용자 입력 출력 시 XSS 방어 --%>
<c:out value="${user.comment}" default="내용없음"/>
```

### XSS 공격이란?
> 게시판, 댓글 등에 `<script>` 삽입 → 다른 사용자 브라우저에서 실행
> → 쿠키/세션 탈취, 계정 도용

```
방어 방법
1. <c:out> 사용 (< → &lt; 자동 변환)
2. 입력값에서 <script> 필터링
3. HTTPOnly 쿠키 → JS에서 쿠키 접근 불가
   Set-Cookie: JSESSIONID=...; HttpOnly
```

---

## 3. MV 구조 게시판 전체 흐름

> Model이 데이터 처리 → JSP는 출력만 담당

```
list.jsp     → BoardModel.boardList()     → BoardDAO.boardListData()
detail.jsp   → BoardModel.boardDetail()   → BoardDAO.boardDetail() + hit+1
insert.jsp   → (폼 화면)
insert_ok.jsp→ BoardModel.boardInsert()   → BoardDAO.boardInsert() → sendRedirect
update.jsp   → BoardModel.boardUpdateData()→ BoardDAO.boardUpdateData()
update_ok.jsp→ BoardModel.boardUpdate()   → BoardDAO.boardUpdate() → "yes"/"no" 반환
```

### BoardModel 핵심 패턴
```java
public class BoardModel {

  // 목록 : 데이터 처리 후 request에 담아서 JSP 전달
  public void boardList(HttpServletRequest request) {
    String page = request.getParameter("page");
    if(page == null) page = "1";
    int curpage = Integer.parseInt(page);

    BoardDAO dao = BoardDAO.newInstance();
    List<BoardVO> list = dao.boardListData(curpage);
    int count = dao.boardRowCount();
    int totalpage = (int)(Math.ceil(count / 10.0));
    count = count - ((curpage * 10) - 10);  // 역순 번호 계산

    request.setAttribute("list",      list);
    request.setAttribute("curpage",   curpage);
    request.setAttribute("totalpage", totalpage);
    request.setAttribute("count",     count);
    request.setAttribute("today", new SimpleDateFormat("yyyy-MM-dd").format(new Date()));
  }

  // 글쓰기 : 데이터 저장 후 목록으로 이동
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

  // 수정 : 비밀번호 확인 → "yes"/"no" 문자열 반환 (Ajax 처리)
  public void boardUpdate(HttpServletRequest request,
                          HttpServletResponse response) {
    BoardVO vo = new BoardVO();
    vo.setName(request.getParameter("name"));
    vo.setSubject(request.getParameter("subject"));
    vo.setContent(request.getParameter("content"));
    vo.setPwd(request.getParameter("pwd"));
    vo.setNo(Integer.parseInt(request.getParameter("no")));

    BoardDAO dao = BoardDAO.newInstance();
    boolean bCheck = dao.boardUpdate(vo);  // 비밀번호 확인 후 수정

    response.setContentType("text/html;charset=UTF-8");
    PrintWriter out = response.getWriter();
    out.write(bCheck ? "yes" : "no");  // Ajax로 결과 반환
  }
}
```

---

## 4. BoardDAO 핵심 SQL

```java
// 답글형 게시판 목록 (group_id 내림차순, group_step 오름차순)
String sql = "SELECT no, subject, name, TO_CHAR(regdate,'yyyy-mm-dd'), hit, group_tab "
           + "FROM jspReplyBoard "
           + "ORDER BY group_id DESC, group_step ASC "
           + "OFFSET ? ROWS FETCH NEXT ? ROWS ONLY";

// 글쓰기 (group_id = 현재 MAX + 1)
String sql = "INSERT INTO jspReplyBoard(no, name, subject, content, pwd, group_id) "
           + "VALUES(jrb_no_seq.nextval, ?, ?, ?, ?, "
           + "(SELECT NVL(MAX(group_id)+1, 1) FROM jspReplyBoard))";

// 상세보기 (조회수 증가 + 데이터 조회)
String sql1 = "UPDATE jspReplyBoard SET hit=hit+1 WHERE no=?";
String sql2 = "SELECT no, name, subject, content, hit, "
            + "TO_CHAR(regdate,'yyyy-MM-dd hh24:mi:ss') "
            + "FROM jspReplyBoard WHERE no=?";

// 수정 : 비밀번호 확인 후 UPDATE
String sql1 = "SELECT pwd FROM jspReplyBoard WHERE no=?";
// db_pwd.equals(vo.getPwd()) → true면 아래 실행
String sql2 = "UPDATE jspReplyBoard SET name=?, subject=?, content=? WHERE no=?";
```

---

## 5. JSP 화면 처리 패턴

### list.jsp — JSTL로 역순 번호 처리
```jsp
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<%
  BoardModel model = new BoardModel();
  model.boardList(request);
%>

<c:set var="count" value="${count}"/>
<c:forEach var="vo" items="${list}">
  <td>${count}</td>
  <td>
    <%-- 답글 들여쓰기 --%>
    <c:if test="${vo.group_tab > 0}">
      <c:forEach var="i" begin="1" end="${vo.group_tab}">
        &nbsp;&nbsp;
      </c:forEach>
      <img src="re_icon.png">
    </c:if>

    <a href="detail.jsp?no=${vo.no}">${vo.subject}</a>

    <%-- 오늘 등록된 글 NEW 표시 --%>
    <c:if test="${vo.dbday == today}">
      &nbsp;<sup><img src="new.gif"></sup>
    </c:if>
  </td>
  <c:set var="count" value="${count-1}"/>  <%-- count-- --%>
</c:forEach>

<%-- 페이징 --%>
${curpage} page / ${totalpage} pages
```

### detail.jsp — EL로 출력
```jsp
<%
  BoardModel model = new BoardModel();
  model.boardDetail(request);
%>
<td>${vo.no}</td>
<td>${vo.dbday}</td>
<td>${vo.name}</td>
<td>${vo.hit}</td>
<td colspan="3">${vo.subject}</td>
<td><pre style="white-space:pre-wrap; background:white; border:none">${vo.content}</pre></td>

<%-- 버튼 --%>
<a href="#">답변</a>
<a href="update.jsp?no=${vo.no}">수정</a>
<a href="#">삭제</a>
<a href="list.jsp">목록</a>
```

### update.jsp — jQuery Ajax로 비밀번호 확인
```javascript
$('#sendBtn').click(function() {
  $.ajax({
    type: 'get',
    url:  'update_ok.jsp',
    data: {
      no:      $('#no').val(),
      name:    $('#name').val(),
      subject: $('#subject').val(),
      content: $('#content').val(),
      pwd:     $('#pwd').val()
    },
    success: function(res) {
      if(res.trim() == "yes") {
        location.href = "detail.jsp?no=" + $('#no').val();
      } else {
        alert("비밀번호가 틀립니다!!");
        $('#pwd').val("").focus();
      }
    }
  });
});
```

```jsp
<%-- update_ok.jsp : 한 줄로 처리 --%>
<%
  BoardModel model = new BoardModel();
  model.boardUpdate(request, response);  // "yes" 또는 "no" 반환
%>
```

---

## 6. 답글형 게시판 DB 구조

```
BoardVO 컬럼
├── no          — 글 번호 (PK, SEQUENCE)
├── name        — 작성자
├── subject     — 제목
├── content     — 내용
├── pwd         — 비밀번호
├── hit         — 조회수
├── regdate     — 작성일
├── group_id    — 원글 그룹 번호 (내림차순 정렬)
├── group_step  — 그룹 내 순서 (오름차순 정렬)
├── group_tab   — 들여쓰기 깊이 (0=원글, 1=1단계 답글...)
├── root        — 원글 번호
└── depth       — 답글 깊이
```

> 답글 INSERT 시 SQL 4개 실행 → **트랜잭션 처리 필수**
> 삭제 시에도 SQL 여러 개 → 트랜잭션 처리

---

## 7. EL 구분 출력 문법 총정리

```jsp
<%-- 프레임워크별 데이터 바인딩 방식 --%>
jQuery    → $().text() , $().html()
Vue       → {{}} (기본) / [[]] (delimiter 변경 시)
React     → {}
Django    → {{}}
Thymeleaf → [[]]
JSP EL    → ${}
```

---

## 8. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `BoardVO.java` | Lombok `@Data` — group_id/step/tab/root/depth 포함 |
| `BoardDAO.java` | DBCP JNDI + 목록/총수/글쓰기/상세/수정 |
| `BoardModel.java` | MV 구조 — boardList/Insert/Detail/UpdateData/Update |
| `list.jsp` | JSTL + 역순 번호 + 답글 들여쓰기 + NEW 표시 |
| `detail.jsp` | EL 상세보기 + 답변/수정/삭제/목록 버튼 |
| `insert.jsp` | POST 글쓰기 폼 (required 유효성) |
| `insert_ok.jsp` | boardInsert() 호출 후 list.jsp 이동 |
| `update.jsp` | jQuery Ajax 비밀번호 확인 수정 |
| `update_ok.jsp` | boardUpdate() 호출 → "yes"/"no" 반환 |
| `reply.jsp` | JSTL/EL 전체 주석 정리 + 글쓰기 폼 |
| `style.css` | 공통 레이아웃 CSS |
| `JSP-총정리.docx` | JDBC/DBCP/EL/JSTL/MVC 전체 개념 + XSS 보안 |
