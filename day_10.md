# 📘 Day 10 — DBCP / JavaBean / JSP 액션 태그 / Session 로그인

## 0. 핵심 빠른 참조

| 구문 | 예시 | 설명 |
|------|------|------|
| DBCP 연결 | `DataSource ds = c.lookup("jdbc/oracle")` | Pool에서 Connection 획득 |
| 반환 | `conn.close()` | 실제 닫기가 아닌 Pool로 반환 |
| useBean | `<jsp:useBean id="bean" class="com.sist.bean.MemberBean">` | Bean 객체 생성 |
| setProperty | `<jsp:setProperty name="bean" property="*"/>` | 폼 데이터 전체 자동 매핑 |
| getProperty | `<jsp:getProperty name="bean" property="name">` | Bean 값 출력 |
| include | `<jsp:include page="header.jsp">` | 다른 JSP 포함 (동적) |
| forward | `<jsp:forward page="list.jsp">` | 화면 이동 (request 유지) |
| 세션 저장 | `session.setAttribute("id", "hong")` | 로그인 정보 저장 |
| 세션 읽기 | `session.getAttribute("id")` | 로그인 여부 확인 |
| 세션 삭제 | `session.invalidate()` | 로그아웃 |

---

## 1. DBCP (DataBase Connection Pool)

> DB 연결을 미리 여러 개 만들어 Pool에 저장해두고 재사용하는 기술

### 기존 방식 vs DBCP

| 구분 | 기존 JDBC | DBCP |
|------|-----------|------|
| Connection 생성 | 요청마다 새로 생성 | 미리 생성 후 재사용 |
| 메모리 | 누수 발생 가능 | Pool이 관리 |
| 속도 | 연결 시간 소요 | 즉시 획득 |
| 보안 | DDoS 취약 | Connection 수 제한 가능 |

### DBCP 설정 (context.xml)
```xml
<Resource
  driverClassName="oracle.jdbc.driver.OracleDriver"
  url="jdbc:oracle:thin:@localhost:1521:XE"
  username="hr"
  password="happy"
  maxActive="10"
  maxIdle="10"
  maxWait="-1"
  auth="Container"
  name="jdbc/oracle"
  type="javax.sql.DataSource"
/>
```

### DBCP 연결 코드 (JNDI)
```java
// 1. 탐색기 열기
Context init = new InitialContext();
// 2. 환경 컨텍스트 가져오기
Context c = (Context) init.lookup("java://comp/env");
// 3. DataSource 획득
DataSource ds = (DataSource) c.lookup("jdbc/oracle");
// 4. Connection 획득 (Pool에서 꺼내옴)
conn = ds.getConnection();

// 사용 후 반환 (닫는 것이 아닌 Pool로 반환)
conn.close();  // ← 가짜 close → 실제로는 Pool로 반환
```

### DBCP DAO 패턴
```java
public class EmpDAO {
  private static EmpDAO dao;

  // 싱글턴
  public static EmpDAO newInstance() {
    if(dao == null) dao = new EmpDAO();
    return dao;
  }

  public List<EmpBean> empListData() {
    List<EmpBean> list = new ArrayList<>();
    try {
      getConnection();
      String sql = "SELECT empno, ename, job, TO_CHAR(hiredate,'yyyy-MM-dd'), sal FROM emp";
      ps = conn.prepareStatement(sql);
      ResultSet rs = ps.executeQuery();
      while(rs.next()) {
        EmpBean vo = new EmpBean();
        vo.setEmpno(rs.getInt(1));
        vo.setEname(rs.getString(2));
        list.add(vo);
      }
    } catch(Exception ex) {
      ex.printStackTrace();
    } finally {
      disConnection();  // ← 반드시 finally에서 반환
    }
    return list;
  }
}
```

---

## 2. JavaBean / DTO / VO 비교

| 구분 | JavaBean | DTO | VO |
|------|----------|-----|----|
| 목적 | JSP 표준 객체 | 데이터 전달 | 불변 데이터 |
| 규칙 | 엄격 (기본생성자, getter/setter) | 자유로움 | getter만 존재 |
| 사용 | JSP 중심 | Spring / MyBatis | AI 응답 데이터 |

> 최근 실무 : 대부분 **DTO** 사용 (데이터를 모아서 브라우저로 전송)
> AI 도입 후 : 변경 불가 데이터 → **VO** / **record** (Java 14+)

### JavaBean 규칙
```java
public class MemberBean {
  private String name;      // private 변수
  private boolean admin;

  // getter / setter (lombok으로 대체 가능)
  public String getName() { return name; }
  public void setName(String name) { this.name = name; }

  // boolean은 getter가 is로 시작
  public boolean isAdmin() { return admin; }
}
```

---

## 3. JSP 액션 태그

> Java 코드를 직접 쓰지 않고 태그 형태로 기능 수행
> XML 형식 — 대소문자 구분, 반드시 닫기 태그 필요

### jsp:useBean / setProperty / getProperty
```jsp
<%-- Java 방식 (기존) --%>
<%
  String name = request.getParameter("name");
  MemberBean bean = new MemberBean();
  bean.setName(name);
%>

<%-- 액션 태그 방식 (간결) --%>
<jsp:useBean id="bean" class="com.sist.bean.MemberBean">
  <jsp:setProperty name="bean" property="*"/>
  <%-- property="*" : 폼의 모든 name과 일치하는 setter 자동 호출 --%>
</jsp:useBean>

<%-- 출력 --%>
이름: <jsp:getProperty name="bean" property="name">
<%-- = 이름: <%= bean.getName() %> --%>
```

### jsp:include — 조립식 화면 구성 ⭐
```jsp
<%-- main.jsp : 공통 레이아웃 조립 --%>
<body>
  <jsp:include page="header.jsp"/>   <%-- 네비게이션 고정 --%>
  <jsp:include page="content.jsp"/>  <%-- 동적으로 변경되는 영역 --%>
  <jsp:include page="footer.jsp"/>   <%-- 하단 고정 --%>
</body>
```

```
화면 구조
┌─────────────────────────────┐
│  로고     메뉴  → header.jsp │ ← 고정
├─────────────────────────────┤
│  실제 출력 내용              │ ← 동적 변경
├─────────────────────────────┤
│  개인정보방침  → footer.jsp  │ ← 고정
└─────────────────────────────┘
```

---

## 4. Session 로그인 패턴

```jsp
<%-- main.jsp : 로그인 여부에 따라 다른 화면 출력 --%>
<%
  String id = (String) session.getAttribute("id");
%>
<% if(id == null) { %>
  <%-- 비로그인 : 로그인 폼 출력 --%>
  <form method="post" action="../member/login.jsp">
    ID: <input type="text" name="id">
    PW: <input type="password" name="pwd">
    <button>로그인</button>
  </form>
<% } else { %>
  <%-- 로그인 상태 : 이름 + 로그아웃 --%>
  <%= session.getAttribute("name") %>님 로그인 되었습니다
  <form method="post" action="../member/logout.jsp">
    <button>로그아웃</button>
  </form>
<% } %>
```

---

## 5. 웹 기술 전체 스택 흐름

```
Back-End
  Java → JSP → MVC → Spring Framework → Spring-Boot
  JDBC → DBCP → MyBatis → JPA

Front-End
  JavaScript → jQuery → Vue(Pinia) → React(TanStack Query) → Next.js
  TypeScript / Node.js

CI/CD
  Git Action → Docker → Docker-Compose → DockerHub
  Jenkins(사령관) → 쿠버네티스 → AWS(Ubuntu)
```

---

## 6. JSTL 주요 태그 정리

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<c:forEach var="vo" items="${list}">  <%-- for-each --%>
<c:if test="${id != null}">           <%-- if --%>
<c:choose> <c:when> <c:otherwise>    <%-- if-else --%>
<c:set var="name" value="홍길동">    <%-- 변수 설정 --%>
<c:redirect url="list.jsp">          <%-- sendRedirect --%>
<fmt:formatDate value="${date}" pattern="yyyy-MM-dd">
<fmt:formatNumber value="${price}" pattern="#,###">
```

> `<% %>` 스크립트릿 → 최대한 제거 → JSTL + EL(`${}`) 로 대체

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `list.jsp` | DBCP 연동 사원 목록 출력 |
| `EmpDAO.java` | DBCP JNDI 연결 + 싱글턴 패턴 |
| `input.jsp` | 개인정보 입력 폼 |
| `output.jsp` | 기본 파라미터 수신 (스크립트릿) |
| `output2.jsp` | MemberBean으로 폼 데이터 매핑 |
| `output3.jsp` | jsp:useBean + setProperty `*` 자동 매핑 |
| `include.jsp` | jsp:include 조립식 레이아웃 개념 |
| `bean.jsp` | JSP 전체 흐름 + 웹 기술 스택 정리 |
| `main.jsp` | 세션 확인 → 로그인/로그아웃 전환 |
| `header.jsp` | Bootstrap 네비게이션 바 |
