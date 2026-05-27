# 📘 Day 12 — MV구조 전환 실습 / 페이징 블록

## 0. 핵심 빠른 참조

| 구분 | 일반 JSP | MV 구조 |
|------|----------|---------|
| 데이터 처리 | JSP 안에서 직접 | Model 클래스로 분리 |
| 화면 출력 | `<%= %>` 스크립트릿 | `${}` EL + JSTL |
| DAO 호출 | JSP 안에서 직접 | Model이 호출 후 request에 저장 |
| 유지보수 | 어려움 | 쉬움 (분업 가능) |
| Spring 관계 | - | MVC의 전 단계 |

---

## 1. 기술 발전 흐름

```
JSP (All-in-one)
  → MV  (JSP + Model 분리)
    → MVC (JSP + Model + Servlet Controller)
      → Spring Framework (설정 복잡)
        → Spring-Boot (단순화) ⭐

JDBC → DBCP → MyBatis → JPA
Oracle → MySQL → MariaDB (AWS → Docker)
Git Action → Docker → Docker-Compose → 쿠버네티스 → Jenkins
```

---

## 2. JDBC vs DBCP 비교

| 구분 | JDBC | DBCP |
|------|------|------|
| Connection 생성 | 요청마다 새로 생성 | Pool에서 재사용 |
| 메모리 누수 | 발생 가능 | Pool이 관리 |
| 설정 위치 | DAO 생성자 | context.xml + JNDI |
| 용도 | 단순 실습 | 실무 웹 프로젝트 |

```java
// JDBC 방식 (MusicDAO)
public MusicDAO() {
  Class.forName("oracle.jdbc.driver.OracleDriver");
}
public void getConnection() {
  conn = DriverManager.getConnection(URL, "hr", "happy");
}

// DBCP 방식 (FoodDAO)
public void getConnection() {
  Context init = new InitialContext();
  Context c = (Context) init.lookup("java://comp/env");
  DataSource ds = (DataSource) c.lookup("jdbc/oracle");
  conn = ds.getConnection();  // Pool에서 가져옴
}
// conn.close() → Pool로 반환 (실제 닫기 아님)
```

---

## 3. Lombok @Data

```java
// @Data = @Getter + @Setter + @ToString + @EqualsAndHashCode
@Data
public class FoodVO {
  private int no;
  private String name, type, phone, address, poster;
  private double score;
}

// @Data 없이 직접 선언 시
public class MusicVO {
  private int no, cno, idcrement;
  private String title, singer, album, poster, state;
  // getter/setter 수동 작성 필요
}
```

> `@Data` 사용 시 getter/setter/toString 자동 생성 → 코드 간결

---

## 4. MV 구조 전환 — 핵심 패턴

### Model 클래스 역할
```java
// FoodModel.java — JSP의 Java 코드를 분리
public class FoodModel {
  public void foodList(HttpServletRequest request) {
    // 1. 파라미터 수신
    String strPage = request.getParameter("page");
    if(strPage == null) strPage = "1";
    int curpage = Integer.parseInt(strPage);

    // 2. DAO 호출
    FoodDAO dao = new FoodDAO();
    List<FoodVO> list = dao.foodListData(curpage);
    int totalpage = dao.foodTotalPage();

    // 3. 페이징 블록 계산
    final int BLOCK = 10;
    int startPage = ((curpage-1) / BLOCK * BLOCK) + 1; // 1, 11, 21...
    int endPage   = ((curpage-1) / BLOCK * BLOCK) + BLOCK; // 10, 20, 30...
    if(endPage > totalpage) endPage = totalpage;

    // 4. request에 담아서 JSP로 전달 (Call By Reference)
    request.setAttribute("list",      list);
    request.setAttribute("curpage",   curpage);
    request.setAttribute("totalpage", totalpage);
    request.setAttribute("startPage", startPage);
    request.setAttribute("endPage",   endPage);
  }
}
```

> ⚠️ `request`를 매개변수로 넘기는 것 → Call By Reference
> JSP가 가진 request에 직접 값을 채워주는 방식

---

## 5. 일반 JSP vs MV 구조 — 실제 코드 비교

### 일반 JSP (list.jsp — Food 버전 1)
```jsp
<%@ page import="java.util.*, com.sist.dao.*" %>
<%
  // JSP 안에서 직접 처리
  String strPage = request.getParameter("page");
  if(strPage == null) strPage = "1";
  int curpage = Integer.parseInt(strPage);

  FoodDAO dao = new FoodDAO();
  List<FoodVO> list = dao.foodListData(curpage);
  int totalpage = dao.foodTotalPage();

  final int BLOCK = 10;
  int startPage = ((curpage-1) / BLOCK * BLOCK) + 1;
  int endPage   = ((curpage-1) / BLOCK * BLOCK) + BLOCK;
  if(endPage > totalpage) endPage = totalpage;
%>
<%-- 출력 : 스크립트릿 --%>
<% for(FoodVO vo : list) { %>
  <img src="<%= vo.getPoster() %>" title="<%= vo.getAddress() %>">
  <p><%= vo.getName() %></p>
<% } %>

<%-- 페이징 --%>
<% if(startPage > 1) { %>
  <li><a href="list.jsp?page=<%= startPage-1 %>">&laquo;</a></li>
<% } %>
<% for(int i = startPage; i <= endPage; i++) { %>
  <li <%= i==curpage?"class=active":"" %>>
    <a href="list.jsp?page=<%= i %>"><%= i %></a>
  </li>
<% } %>
<% if(endPage < totalpage) { %>
  <li><a href="list.jsp?page=<%= endPage+1 %>">&raquo;</a></li>
<% } %>
```

### MV 구조 (list.jsp — Food 버전 2) ⭐
```jsp
<%@ page import="com.sist.model.*" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<%
  FoodModel model = new FoodModel();
  model.foodList(request);  // Java 코드는 이 한 줄!
%>
<%-- 출력 : EL + JSTL --%>
<c:forEach var="vo" items="${list}">
  <img src="${vo.poster}" title="${vo.address}">
  <p>${vo.name}</p>
</c:forEach>

<%-- 페이징 --%>
<c:if test="${startPage > 1}">
  <li><a href="list.jsp?page=${startPage-1}">&laquo;</a></li>
</c:if>
<c:forEach var="i" begin="${startPage}" end="${endPage}">
  <li ${i==curpage?"class=active":""}>
    <a href="list.jsp?page=${i}">${i}</a>
  </li>
</c:forEach>
<c:if test="${endPage < totalpage}">
  <li><a href="list.jsp?page=${endPage+1}">&raquo;</a></li>
</c:if>
```

---

## 6. 페이징 블록 계산 공식

```java
final int BLOCK = 10;  // 한 블록에 보여줄 페이지 수

// 시작 페이지 : 1, 11, 21, 31...
int startPage = ((curpage - 1) / BLOCK * BLOCK) + 1;

// 끝 페이지 : 10, 20, 30...
int endPage = ((curpage - 1) / BLOCK * BLOCK) + BLOCK;
if(endPage > totalpage) endPage = totalpage;  // 마지막 블록 보정
```

| curpage | startPage | endPage |
|---------|-----------|---------|
| 1~10 | 1 | 10 |
| 11~20 | 11 | 20 |
| 21~30 | 21 | 30 |

---

## 7. DAO 공통 패턴 정리

```java
// 목록 (페이징)
String sql = "SELECT no, poster, name, address FROM food "
           + "ORDER BY no OFFSET ? ROWS FETCH NEXT 12 ROWS ONLY";
ps.setInt(1, (page * 12) - 12);  // 1페이지=0, 2페이지=12...

// 총 페이지
String sql = "SELECT CEIL(COUNT(*) / 12.0) FROM food";

// 싱글턴 패턴 (메모리 1개만 생성 → 재사용)
private static FoodDAO dao;
public static FoodDAO newInstance() {
  if(dao == null) dao = new FoodDAO();
  return dao;
}
```

---

## 8. 실습 예제 목록

| 파일 | 구조 | 내용 |
|------|------|------|
| `FoodVO.java` | — | Lombok `@Data` 적용 VO |
| `FoodDAO.java` | DBCP | 맛집 목록/총페이지 (JNDI) |
| `FoodModel.java` | Model | 페이징 블록 계산 + request 전달 |
| `list.jsp` (Food v1) | 일반 JSP | 스크립트릿 + for문 직접 처리 |
| `list.jsp` (Food v2) | MV 구조 | EL + JSTL + FoodModel 호출 |
| `MusicVO.java` | — | 일반 VO (getter/setter 수동) |
| `MusicDAO.java` | JDBC | 음악 목록 (일반 JDBC 방식) |
| `MusicModel.java` | Model | musicList() → request 전달 |
| `list.jsp` (Music) | MV 구조 | 음악 목록 JSTL 출력 |
| `EL.docx` | 교재 | EL/JSTL 전체 이론 정리 |
