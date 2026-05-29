# 📘 Day 14 — MVC 구조 (Servlet Controller)

## 0. MV → MVC → Spring 발전 흐름

```
일반 JSP (All-in-one)
    ↓
MV  (JSP + Model 분리)
    ↓
MVC v1 (Controller Servlet + cmd 파라미터 + Model + View)
    ↓
MVC v2 (Controller + Model Interface + XML 설정)
    ↓
Spring Framework (MVC + DI + AOP)
    ↓
Spring-Boot (Spring 자동 설정)
```

---

## 1. MVC 구조란?

```
사용자 요청
    ↓
Controller (Servlet) ← 요청 받기 / 분기 / JSP 전달
    ↓
Model (Java 클래스) ← 데이터 처리 / request에 결과 저장
    ↓
View (JSP) ← EL/JSTL로 출력만 담당
    ↓
브라우저 화면
```

| 역할 | 담당 | 설명 |
|------|------|------|
| Model | Java 클래스 | 데이터 처리, DB 연동 |
| View | JSP | 화면 출력 (EL/JSTL만 사용) |
| Controller | Servlet | 요청 분기, Model 호출, JSP로 forward |

> 비유 — 음식점
> `손님 화면 = View` / `주문받는 직원 = Controller` / `요리사 = Model`

---

## 2. MVC v1 — cmd 파라미터 방식

> URL: `http://localhost/프로젝트/Controller?cmd=list`
> Controller 하나가 `cmd` 값으로 분기

```java
@WebServlet("/Controller")
public class Controller extends HttpServlet {

  protected void service(HttpServletRequest request,
                         HttpServletResponse response) {
    // 1. cmd 파라미터 수신
    String cmd = request.getParameter("cmd");
    if(cmd == null) cmd = "list";

    // 2. cmd에 따라 Model 호출 + JSP 결정
    String jsp = "";
    if(cmd.equals("list")) {
      new ListModel().execute(request);
      jsp = "view/list.jsp";
    } else if(cmd.equals("detail")) {
      new DetailModel().execute(request);
      jsp = "view/detail.jsp";
    } else if(cmd.equals("insert")) {
      new InsertModel().execute(request);
      jsp = "view/insert.jsp";
    } else if(cmd.equals("update")) {
      new UpdateModel().execute(request);
      jsp = "view/update.jsp";
    } else if(cmd.equals("delete")) {
      new DeleteModel().execute(request);
      jsp = "view/delete.jsp";
    }

    // 3. request 유지하며 JSP로 이동 (forward)
    RequestDispatcher rd = request.getRequestDispatcher(jsp);
    rd.forward(request, response);
  }
}
```

---

## 3. Model v1 — 직접 호출 (jsp 경로 없음)

```java
// Model 클래스 — request에 데이터 저장 후 반환 없음
public class ListModel {
  public void execute(HttpServletRequest request) {
    String msg = "게시판 목록";
    request.setAttribute("msg", msg);  // JSP에서 ${msg}로 사용
  }
}
```

---

## 4. MVC v2 — Model Interface + JSP 경로 반환

> Model이 JSP 경로까지 반환 → Controller가 if문 없어짐

### Model 인터페이스
```java
public interface Model {
  // Model 구현 클래스가 반드시 구현해야 하는 메소드
  public String execute(HttpServletRequest request);
  // Spring : handleRequest()
  // Struts : execute()
}
```

### Model 구현 클래스
```java
public class ListModel implements Model {
  @Override
  public String execute(HttpServletRequest request) {
    request.setAttribute("msg", "게시판 목록");
    return "view/list.jsp";  // JSP 경로를 직접 반환 ⭐
  }
}

public class DetailModel implements Model {
  @Override
  public String execute(HttpServletRequest request) {
    request.setAttribute("msg", "게시판 상세보기");
    return "view/detail.jsp";
  }
}
```

---

## 5. MVC v3 — XML 설정 + 자동 클래스 등록 (Spring 원형) ⭐

> `*.do` URL 패턴으로 모든 요청을 Controller 하나가 받음
> XML에서 URL-클래스 매핑 → `Class.forName()`으로 동적 생성
> 이것이 **Spring의 DispatcherServlet + applicationContext.xml 원형**

### Model 인터페이스 (v3)
```java
public interface Model {
  // JSP 경로를 반환
  public String handleRequest(HttpServletRequest request,
                              HttpServletResponse response);
  // Spring: handleRequest()
  // Struts: execute()
}
```

### Model 구현 — main.jsp 레이아웃 방식
```java
// MainModel : 홈 화면
public class MainModel implements Model {
  @Override
  public String handleRequest(HttpServletRequest request,
                              HttpServletResponse response) {
    request.setAttribute("main_jsp", "../main/home.jsp"); // 콘텐츠 영역
    return "../main/main.jsp";  // 공통 레이아웃
  }
}

// FoodDetailModel : 맛집 상세보기
public class FoodDetailModel implements Model {
  @Override
  public String handleRequest(HttpServletRequest request,
                              HttpServletResponse response) {
    request.setAttribute("main_jsp", "../food/detail.jsp");
    return "../main/main.jsp";
  }
}

// GoodsListModel : 상품 목록
public class GoodsListModel implements Model {
  @Override
  public String handleRequest(HttpServletRequest request,
                              HttpServletResponse response) {
    request.setAttribute("main_jsp", "../goods/list.jsp");
    return "../main/main.jsp";
  }
}
```

> `main.jsp` — 공통 레이아웃 (header/footer 고정)
> `main_jsp` — 동적으로 바뀌는 콘텐츠 영역
> → `<jsp:include page="${main_jsp}"/>` 로 조립

### Controller v3 — XML 읽어서 자동 클래스 등록
```java
@WebServlet("*.do")  // 모든 .do 요청을 받음
public class Controller extends HttpServlet {

  // URL → Model 클래스 매핑 저장소
  private Map<String, Model> clsMap = new HashMap<>();

  // 서버 시작 시 1회 실행 — XML 파싱해서 Map에 등록
  public void init(ServletConfig config) throws ServletException {
    try {
      String path = "WEB-INF/application.xml";

      // XML 파싱 (DOM 방식)
      DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
      DocumentBuilder db = dbf.newDocumentBuilder();
      Document doc = db.parse(new File(path));

      // <bean id="main/main.do" class="com.sist.model.MainModel"> 읽기
      NodeList list = doc.getDocumentElement().getElementsByTagName("bean");
      for(int i = 0; i < list.getLength(); i++) {
        Element bean = (Element) list.item(i);
        String id  = bean.getAttribute("id");    // URL 키
        String cls = bean.getAttribute("class"); // 클래스명

        // 리플렉션으로 클래스 동적 생성 (싱글턴)
        Class clsName = Class.forName(cls);
        Model model = (Model) clsName.getDeclaredConstructor().newInstance();
        clsMap.put(id, model);
      }
    } catch(Exception ex) {}
  }

  // 요청마다 실행 — URL로 Model 찾아서 실행
  protected void service(HttpServletRequest request,
                         HttpServletResponse response) {
    try {
      // http://localhost/프로젝트/main/main.do → key = "main/main.do"
      String uri = request.getRequestURI();
      String key = uri.substring(request.getContextPath().length() + 1);

      // Map에서 Model 클래스 찾기
      Model model = clsMap.get(key);

      // 처리 후 JSP 경로 반환
      String jsp = model.handleRequest(request, response);

      // request 유지하며 JSP로 이동
      RequestDispatcher rd = request.getRequestDispatcher(jsp);
      rd.forward(request, response);
    } catch(Exception ex) {}
  }
}
```

### application.xml — URL ↔ 클래스 매핑
```xml
<beans>
  <bean id="main/main.do"      class="com.sist.model.MainModel"/>
  <bean id="food/detail.do"    class="com.sist.model.FoodDetailModel"/>
  <bean id="goods/list.do"     class="com.sist.model.GoodsListModel"/>
</beans>
```

> ⚠️ Spring의 `applicationContext.xml` + `DispatcherServlet`이 이 구조를 자동화한 것

---

## 6. DBCPUtil 분리 — 공통 유틸 클래스

> DAO마다 반복되던 DBCP 연결 코드를 **공통 유틸 클래스로 분리**

```java
// DBCPUtil.java — commons 패키지
public class DBCPUtil {
  private Connection conn;

  public Connection getConnection() {
    try {
      Context init = new InitialContext();
      Context c = (Context) init.lookup("java://comp/env");
      DataSource ds = (DataSource) c.lookup("jdbc/oracle");
      conn = ds.getConnection();
    } catch(Exception ex) { ex.printStackTrace(); }
    return conn;
  }

  public void disConnection(Connection conn, PreparedStatement ps) {
    try {
      if(ps != null) ps.close();
      if(conn != null) conn.close();  // Pool로 반환
    } catch(Exception ex) {}
  }
}
```

### DAO에서 사용
```java
public class FoodDAO {
  private Connection conn;
  private PreparedStatement ps;
  private DBCPUtil db = new DBCPUtil();  // ⭐ 유틸 클래스 사용

  public List<FoodVO> foodListData(int page) {
    List<FoodVO> list = new ArrayList<>();
    try {
      conn = db.getConnection();  // DBCPUtil로 연결
      String sql = "SELECT no, poster, name, address FROM food "
                 + "ORDER BY no OFFSET ? ROWS FETCH NEXT 12 ROWS ONLY";
      ps = conn.prepareStatement(sql);
      ps.setInt(1, (page * 12) - 12);
      // ...
    } catch(Exception ex) { ex.printStackTrace(); }
    finally {
      db.disConnection(conn, ps);  // DBCPUtil로 반환
    }
    return list;
  }
}
```

---

## 7. MVC 버전별 비교 총정리

| 구분 | v1 (cmd 방식) | v2 (Interface 방식) | v3 (XML 방식) | Spring |
|------|--------------|-------------------|---------------|--------|
| URL 패턴 | `/Controller?cmd=list` | `/Controller?cmd=list` | `/main/main.do` | `/main/main` |
| 분기 방식 | if-else | if-else | XML + Map | `@RequestMapping` |
| Model 형태 | 개별 클래스 | Interface 구현 | Interface 구현 | `@Controller` |
| JSP 경로 | Controller가 결정 | Model이 반환 | Model이 반환 | `return "view/list"` |
| 설정 | 없음 | 없음 | XML | XML / 어노테이션 |
| 클래스 생성 | new 직접 | new 직접 | 리플렉션 자동 | DI 자동 주입 |

---

## 8. forward vs sendRedirect 최종 정리

| 구분 | `forward` | `sendRedirect` |
|------|-----------|----------------|
| request 유지 | ✅ | ❌ 초기화 |
| URL 변경 | ❌ | ✅ |
| 방식 | 서버 내부 이동 | 브라우저에게 재요청 지시 |
| MVC 사용 | Controller → View | 처리 완료 후 목록 이동 |

```java
// MVC Controller → View (forward) : request 유지
RequestDispatcher rd = request.getRequestDispatcher(jsp);
rd.forward(request, response);

// 처리 완료 후 이동 (sendRedirect) : request 초기화
response.sendRedirect("list.jsp");
```

---

## 9. 실습 예제 목록

| 파일 | 프로젝트 | 내용 |
|------|---------|------|
| `Controller.java` (v1) | JSPMVCProject_1 | cmd 파라미터 if-else 분기 |
| `ListModel` ~ `DeleteModel` (v1) | JSPMVCProject_1 | execute() — void 반환 |
| `Model.java` (v2 Interface) | JSPMVCProject_2 | `String execute(request)` |
| `ListModel` ~ `DeleteModel` (v2) | JSPMVCProject_2 | execute() — JSP 경로 반환 |
| `Model.java` (v3 Interface) | JSPMVCProject_3 | `String handleRequest(request, response)` |
| `MainModel`, `FoodDetailModel`, `GoodsListModel` | JSPMVCProject_3 | main_jsp 속성 + main.jsp 레이아웃 |
| `Controller.java` (v3) | JSPMVCProject_3 | `*.do` + XML 파싱 + 리플렉션 자동 등록 |
| `DBCPUtil.java` | commons | DBCP 연결/반환 공통 유틸 분리 |
| `FoodDAO.java` | dao | DBCPUtil 사용 DAO 패턴 |
