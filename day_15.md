# 📘 Day 15 — JSP MVC 총정리 / MyBatis 입문 / 파일 업로드

## 0. 핵심 빠른 참조

| 구분 | 설명 |
|------|------|
| MVC 흐름 | 요청 → Controller → Model(DAO) → DB → request저장 → View(JSP) → 응답 |
| forward | 서버 내부 이동, request 유지, URL 변경 안 됨 → 목록/상세 화면 전환 |
| sendRedirect | 브라우저 재요청, request 초기화, URL 변경 → 등록/수정/삭제 후 이동 |
| PRG 패턴 | POST(등록) → Redirect → GET(목록) — 새로고침 중복 제출 방지 |
| `#{}` | PreparedStatement 방식 — SQL 인젝션 방지 ⭐ |
| `${}` | Statement 방식 — 컬럼명/테이블명 동적 지정 시만 사용 |
| `@MultipartConfig` | 파일 업로드 Servlet 필수 어노테이션 |
| `request.getPart("upload")` | 파일 데이터 추출 |
| `filePart.write(경로)` | 서버에 파일 저장 ⭐ |

---

## 1. JSP MVC 구조 총정리

### 역할 분리 원칙
```
Controller (Servlet)  → 요청 받기, Service 호출, View 선택
Service / Model       → 비즈니스 로직, 트랜잭션 처리
DAO                   → SQL 실행, DB 접근
View (JSP)            → EL/JSTL로 출력만 담당

JSP View에서 금지 ❌    허용 ✅
DB 연결               EL ${}
SQL 작성              JSTL 태그
복잡한 Java 코드      HTML 출력
```

### 전체 요청 흐름
```
사용자 요청
  ↓
Controller (Servlet)
  ↓ Service 호출
Service (비즈니스 로직)
  ↓ DAO 호출
DAO (SQL 실행)
  ↓ DBCP → JDBC
Database
  ↓ 결과 반환
DAO → Service → Controller
  ↓ request.setAttribute()
View (JSP)
  ↓ EL/JSTL 출력
브라우저 화면
```

### forward vs sendRedirect
```java
// forward — request 유지, URL 변경 없음 (목록 조회, 화면 전환)
request.getRequestDispatcher("list.jsp").forward(request, response);

// sendRedirect — request 초기화, URL 변경 (등록/수정/삭제 후)
response.sendRedirect("list.do");
```

### PRG (Post-Redirect-Get) 패턴
```
form 제출(POST)
  ↓ Controller
  ↓ DAO.insert()
  ↓ DB 저장
response.sendRedirect("list.do")  ← Redirect ⭐
  ↓ 브라우저 재요청(GET)
목록 화면 출력
```
> F5 새로고침 시 중복 제출 방지

---

## 2. EL 핵심 정리

### Scope 탐색 순서
```
${name} 검색 순서:
pageScope → requestScope → sessionScope → applicationScope
(보통 생략해서 사용, 구분 필요할 때만 명시)
```

### EL 주요 내장 객체
```jsp
${param.id}                        <%-- request.getParameter("id") --%>
${paramValues.hobby[0]}            <%-- getParameterValues("hobby")[0] --%>
${requestScope.list}               <%-- request.getAttribute("list") --%>
${sessionScope.loginUser}          <%-- session.getAttribute("loginUser") ⭐ --%>
${applicationScope.count}          <%-- application.getAttribute("count") --%>
${pageContext.request.contextPath} <%-- 루트 경로 (/프로젝트명) --%>
```

### EL 연산자
```jsp
<%-- 산술 --%>
${10 div 3}    <%-- 3.333 --%>
${10 mod 3}    <%-- 1 --%>

<%-- 비교 (HTML 태그 충돌 방지용 영문 연산자) --%>
${a eq b}  <%-- == --%>   ${a ne b}  <%-- != --%>
${a lt b}  <%-- <  --%>   ${a gt b}  <%-- >  --%>
${a le b}  <%-- <= --%>   ${a ge b}  <%-- >= --%>

<%-- empty : null, 빈문자열, 빈List → true --%>
${empty sessionScope.loginUser}      <%-- 비로그인 --%>
${not empty sessionScope.loginUser}  <%-- 로그인 상태 --%>

<%-- 삼항 --%>
${param.age >= 20 ? "성인" : "미성년"}
```

---

## 3. JSTL 핵심 정리

### taglib 선언
```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt"  prefix="fmt" %>
```

### 조건문
```jsp
<%-- c:if (else 없음) --%>
<c:if test="${user.age >= 20}">성인</c:if>

<%-- c:choose (if-else if-else) --%>
<c:choose>
  <c:when test="${user.grade == 'VIP'}">VIP 10% 할인</c:when>
  <c:when test="${user.grade == 'GOLD'}">GOLD 5% 할인</c:when>
  <c:otherwise>일반 회원</c:otherwise>
</c:choose>
```

### 반복문 (게시판 필수)
```jsp
<c:forEach items="${boardList}" var="b" varStatus="status">
  <tr>
    <td>${status.count}</td>     <%-- 1부터 시작하는 순번 --%>
    <td>${status.index}</td>     <%-- 0부터 시작하는 인덱스 --%>
    <td>${b.title}</td>
    <td>${b.writer}</td>
    <c:if test="${b.stock == 0}"><span>[품절]</span></c:if>
  </tr>
</c:forEach>
```

### 날짜/숫자 포맷
```jsp
<%-- 날짜 변환 --%>
<fmt:formatDate value="${now}" pattern="yyyy-MM-dd"/>
<fmt:formatDate value="${now}" pattern="yyyy년 MM월 dd일 HH시 mm분"/>

<%-- 숫자 변환 --%>
<fmt:formatNumber value="5000000" pattern="#,###원"/>  <%-- 5,000,000원 --%>
<fmt:formatNumber value="0.75"    type="percent"/>     <%-- 75% --%>
```

---

## 4. 내장 객체 Scope 비교

| 객체 | 저장 위치 | 유지 기간 | 주요 용도 |
|------|-----------|-----------|-----------|
| `request` | 서버 메모리 | 하나의 요청-응답 | Controller → View 데이터 전달 |
| `session` | 서버 메모리 | 브라우저 닫을 때까지 | 로그인 정보, 장바구니 |
| `application` | 서버 메모리 | 서버 켜져 있는 동안 | 전체 방문자 수, 전역 설정 |
| `Cookie` | 사용자 PC | setMaxAge 설정 시간 | 아이디 저장, 오늘 창 닫기 |

```java
// Cookie 생성 및 전송
Cookie myCookie = new Cookie("savedId", "hong123");
myCookie.setMaxAge(60 * 60 * 24);  // 하루 유지
response.addCookie(myCookie);       // response로 전송 필수
```

---

## 5. XML 파싱 — DOM vs SAX

| 항목 | DOM | SAX |
|------|-----|-----|
| 방식 | 전체 로딩 → Tree 구조 | 순차 읽기 → 이벤트 처리 |
| 메모리 | 많이 사용 | 적게 사용 |
| 속도 | 느림 | 빠름 |
| 수정 가능 | ✅ | ❌ |
| 대용량 | 부적합 | ✅ 적합 |
| 구현 난이도 | 쉬움 | 어려움 |
| 선택 기준 | 파일 작고 수정 필요 | 파일 크고 읽기만 |

```java
// DOM 파싱 (MVC v3 Controller의 XML 설정 읽기에 활용)
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(new File(path));

NodeList list = doc.getDocumentElement().getElementsByTagName("bean");
for(int i = 0; i < list.getLength(); i++) {
  Element bean = (Element) list.item(i);
  String id  = bean.getAttribute("id");
  String cls = bean.getAttribute("class");
}
```

> 실무에서는 XML보다 **JSON + Jackson** 사용 비중이 훨씬 높음

---

## 6. MyBatis 입문

> JDBC의 반복 코드를 줄이고 SQL을 XML로 분리 관리하는 SQL 매퍼 프레임워크

### JDBC vs MyBatis 비교
```
JDBC                          MyBatis
Connection 직접 관리   →      SqlSession이 관리
ResultSet 수동 매핑    →      resultType으로 자동 매핑
SQL이 Java 코드 안에   →      XML에 분리 관리
try-catch 반복        →      프레임워크가 처리
```

### 동작 흐름
```
mybatis-config.xml (설정)
  ↓
SqlSessionFactory (빌더가 1회 생성, 싱글턴)
  ↓
SqlSession (요청마다 생성)
  ↓
Mapper XML (SQL 실행)
  ↓
ResultSet → Java 객체 자동 매핑
```

### mybatis-config.xml
```xml
<typeAliases>
  <typeAlias type="com.example.model.UserDTO" alias="UserDTO"/>
</typeAliases>

<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC"/>
    <dataSource type="POOLED">
      <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
      <property name="url"    value="jdbc:mysql://localhost:3306/mydb"/>
      <property name="username" value="root"/>
      <property name="password" value="1234"/>
    </dataSource>
  </environment>
</environments>

<mappers>
  <mapper resource="mappers/UserMapper.xml"/>
</mappers>
```

### Mapper Interface + XML
```java
// UserMapper.java — 메서드명 = XML id와 반드시 일치 ⭐
@Mapper
public interface UserMapper {
  UserDTO findById(Long id);
  List<UserDTO> findAll();
  int insert(UserDTO user);
}
```

```xml
<!-- UserMapper.xml -->
<mapper namespace="com.example.mapper.UserMapper">

  <!-- 단건 조회 -->
  <select id="findById" parameterType="long" resultType="UserDTO">
    SELECT id, name, email FROM users WHERE id = #{id}
  </select>

  <!-- 목록 조회 -->
  <select id="findAll" resultType="UserDTO">
    SELECT id, name, email FROM users
  </select>

  <!-- 등록 -->
  <insert id="insert" parameterType="UserDTO">
    INSERT INTO users (name, email) VALUES (#{name}, #{email})
  </insert>

  <!-- 수정 -->
  <update id="update" parameterType="UserDTO">
    UPDATE users SET name=#{name}, email=#{email} WHERE id=#{id}
  </update>

  <!-- 삭제 -->
  <delete id="delete" parameterType="long">
    DELETE FROM users WHERE id=#{id}
  </delete>

</mapper>
```

### #{} vs ${}
| 구분 | `#{}` | `${}` |
|------|-------|-------|
| 방식 | PreparedStatement (? 바인딩) | Statement (직접 대입) |
| SQL 인젝션 | ✅ 방지 | ❌ 취약 |
| 문자열 따옴표 | 자동 처리 | 처리 안 함 |
| 사용 권장 | ✅ 기본 | 컬럼명/테이블명 동적 지정 시만 |

### 동적 SQL
```xml
<!-- if — 조건부 WHERE -->
<select id="searchUser" resultType="UserDTO">
  SELECT * FROM users WHERE 1=1
  <if test="name != null">AND name = #{name}</if>
  <if test="age != null">AND age = #{age}</if>
</select>

<!-- foreach — IN 절 -->
<select id="findByIds" resultType="UserDTO">
  SELECT * FROM users WHERE id IN
  <foreach collection="ids" item="id" open="(" separator="," close=")">
    #{id}
  </foreach>
</select>
```

### ResultMap — 컬럼명 ≠ 필드명 매핑
```xml
<resultMap id="userMap" type="User">
  <id property="id" column="id"/>
  <result property="userName" column="user_name"/>  <!-- DB: user_name ↔ Java: userName -->
</resultMap>
```

### MyBatis vs JPA
| 항목 | MyBatis | JPA |
|------|---------|-----|
| 개발 방식 | SQL 중심 | 객체 중심 |
| SQL 작성 | 직접 작성 | 자동 생성 |
| 복잡한 쿼리 | ✅ 강점 | 상대적으로 불편 |
| 러닝커브 | 낮음 | 높음 |
| 적합한 상황 | 복잡한 SQL, 레거시 DB, 금융/공공 | CRUD 중심, 객체 지향 |

---

## 7. 파일 업로드 심화

### HTML 폼 — enctype 필수
```html
<form method="post" action="UploadServlet" enctype="multipart/form-data">
  이름: <input type="text"  name="name">
  파일: <input type="file"  name="upload">
  <button>업로드</button>
</form>
```

### Servlet — @MultipartConfig 설정
```java
@WebServlet("/UploadServlet")
@MultipartConfig(
  fileSizeThreshold = 1024 * 1024,     // 1MB — 초과 시 디스크 임시 저장
  maxFileSize       = 1024 * 1024 * 10, // 10MB — 파일 1개 최대 크기
  maxRequestSize    = 1024 * 1024 * 50  // 50MB — 요청 전체 최대 크기
)
public class UploadServlet extends HttpServlet {

  private static final String UPLOAD_DIR = "uploads";

  protected void doPost(HttpServletRequest request,
                        HttpServletResponse response) {
    // 1. 서버 실제 경로
    String uploadPath = getServletContext().getRealPath("")
                      + File.separator + UPLOAD_DIR;

    // 2. 폴더 없으면 생성
    File uploadDir = new File(uploadPath);
    if(!uploadDir.exists()) uploadDir.mkdir();

    try {
      // 3. 한글 처리
      request.setCharacterEncoding("UTF-8");

      // 4. 텍스트 데이터
      String name    = request.getParameter("name");
      String subject = request.getParameter("subject");

      // 5. 파일 데이터 추출 ⭐
      Part filePart = request.getPart("upload");

      DataBoardVO vo = new DataBoardVO();
      vo.setName(name);
      vo.setSubject(subject);

      if(filePart == null || filePart.getSize() == 0) {
        // 파일 없는 경우
        vo.setFilename("");
        vo.setFilesize(0);
      } else {
        // 6. 원본 파일명 추출
        String fileName = filePart.getSubmittedFileName();

        // 7. 서버에 저장 ⭐
        filePart.write(uploadPath + File.separator + fileName);

        vo.setFilename(fileName);
        vo.setFilesize((int) new File(uploadPath + File.separator + fileName).length());
      }

      // 8. DB 저장
      DataBoardDAO.newInstance().databoardInsert(vo);

      // 9. PRG 패턴 — 중복 제출 방지
      response.sendRedirect("main/main.jsp?mode=3");

    } catch(Exception ex) { ex.printStackTrace(); }
  }
}
```

### 파일 업로드 전체 흐름
```
1. 사용자 폼 제출 (multipart/form-data)
2. request.getParameter()    → 텍스트 데이터
3. request.getPart("upload") → 파일 데이터 ⭐
4. filePart.write(경로)      → 서버 폴더에 저장 ⭐
5. VO에 파일명/크기 저장
6. dao.insert(vo)            → DB 저장
7. sendRedirect()            → PRG 패턴 (중복 방지)
```

> ⚠️ 향후 개선 사항: 파일명 중복 처리 (UUID), 파일 다운로드 구현

---

## 8. 학습 로드맵 정리

```
JSP (All-in-one)
  → MV  (JSP + Model 분리)
    → MVC (Servlet Controller)
      → MyBatis (SQL 분리)
        → Spring Framework
          → Spring-Boot ⭐

JDBC → DBCP → MyBatis → JPA
Oracle → MySQL → MariaDB (AWS + Docker)
```
