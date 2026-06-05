# 📘 Day 17 — Spring 구조 직접 구현 (Mini Spring MVC + MyBatis)

## 0. 핵심 개념 한눈에 보기

| 구현 요소 | Spring 실제 | 우리가 직접 구현 |
|-----------|------------|----------------|
| 모든 요청 진입점 | `DispatcherServlet` | `DispatcherServlet.java` (`@WebServlet("*.do")`) |
| 컨트롤러 표시 | `@Controller` | `@Controller` (직접 정의한 어노테이션) |
| URL 매핑 | `@RequestMapping` | `@RequestMapping` (직접 정의한 어노테이션) |
| 클래스 자동 탐색 | `@ComponentScan` | `ComponentScan.java` (직접 구현) |
| XML 설정 | `applicationContext.xml` | `application.xml` |
| MyBatis 연결 | `SqlSessionFactoryBean` | `CreateSqlSessionFactory.java` |
| DB 실행 | `SqlSession` | `session.selectList() / selectOne()` |
| View 이동 | `ViewResolver` + `forward` | `rd.forward(request, response)` |
| redirect | `redirect:url` 문자열 반환 | `jsp.startsWith("redirect:")` 분기 |

---

## 1. 전체 동작 흐름

```
사용자 요청 (food/list.do)
  ↓
DispatcherServlet (@WebServlet("*.do")) — 모든 .do 요청 진입
  ↓
ComponentScan — application.xml 읽기
  → base-package 경로의 .class 파일 전체 스캔
  ↓
@Controller 어노테이션 달린 클래스 찾기
  ↓
클래스 내 메소드에서 @RequestMapping("food/list.do") 매핑 확인
  ↓
매핑 일치 → 리플렉션으로 메소드 실행
  ↓
Model (FoodModel) → DAO (FoodDAO) → MyBatis SqlSession → DB
  ↓
request.setAttribute() → JSP 경로 반환
  ↓
forward 또는 redirect 분기
  ↓
View (JSP) → 브라우저 응답
```

---

## 2. 커스텀 어노테이션 정의

### @Controller — 클래스에 붙이는 표시
```java
// Spring의 @Controller와 동일한 역할을 직접 만든 것
@Retention(RUNTIME)   // 실행 중에도 어노테이션 정보 유지
@Target(TYPE)         // 클래스/인터페이스에 붙일 수 있음
public @interface Controller {
  // 기능 없음 — DispatcherServlet이 읽어서 분기에 활용
  // 어노테이션은 스스로 작동 안 함 → 리플렉션으로 읽어야 동작
}
```

### @RequestMapping — 메소드에 붙이는 URL 매핑
```java
@Retention(RUNTIME)   // 실행 중에도 유지
@Target(METHOD)       // 메소드에만 붙일 수 있음
public @interface RequestMapping {
  public String value(); // 매핑할 URL 값 저장
  // @RequestMapping("food/list.do") → value = "food/list.do"
}
```

> 어노테이션 자체는 기능이 없고 **인덱스(표시)** 역할만 함
> DispatcherServlet이 리플렉션으로 읽어서 URL과 비교 후 메소드 실행

---

## 3. ComponentScan — 클래스 자동 탐색

```java
public class ComponentScan {
  public static List<String> componentScan(String path, String pack) {
    List<String> list = new ArrayList<>();
    try {
      // base-package 경로로 이동
      // 예: com.sist.model → WEB-INF/classes/com/sist/model/
      path = path + File.separator + pack.replace(".", File.separator);
      File dir = new File(path);

      // 폴더 안의 모든 파일 스캔
      for(File f : dir.listFiles()) {
        String name = f.getName();
        String ext  = name.substring(name.lastIndexOf(".") + 1);

        // .class 파일만 수집
        if(ext.equals("class")) {
          String clsName  = name.substring(0, name.lastIndexOf("."));
          String packname = pack + "." + clsName; // 완전한 클래스명
          list.add(packname);
          // 예: "com.sist.model.FoodModel"
        }
      }
    } catch(Exception ex) {}
    return list;
  }
}
```

> Spring의 `@ComponentScan`이 하는 일과 동일
> Spring에서는 이 과정을 내부에서 자동 처리

---

## 4. DispatcherServlet — 핵심 진입점

```java
@WebServlet("*.do") // 모든 .do 요청을 여기서 받음
public class DispatcherServlet extends HttpServlet {

  private List<String> clsList = new ArrayList<>();

  // 서버 시작 시 1회 실행
  public void init(ServletConfig config) {
    try {
      // 1. application.xml 경로 찾기
      URL url = this.getClass().getClassLoader().getResource(".");
      File file = new File(url.toURI());
      String path = file.getPath()
                        .replace("\\", File.separator);
      path = path.substring(0, path.lastIndexOf(File.separator));
      path = path + File.separator + "application.xml";

      // 2. application.xml 파싱
      DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
      DocumentBuilder db = dbf.newDocumentBuilder();
      Document doc = db.parse(new File(path));

      // 3. <context:component-scan base-package="com.sist.model"/> 읽기
      NodeList list = doc.getElementsByTagName("context:component-scan");
      String pack = "";
      for(int i = 0; i < list.getLength(); i++) {
        Element elem = (Element) list.item(i);
        pack = elem.getAttribute("base-package");
      }

      // 4. 해당 패키지의 클래스 전체 수집
      clsList = ComponentScan.componentScan(file.getPath(), pack);

    } catch(Exception ex) {}
  }

  // 요청마다 실행
  protected void service(HttpServletRequest request,
                         HttpServletResponse response) {
    // 1. URI 추출 (컨텍스트 경로 제거)
    String uri = request.getRequestURI();
    uri = uri.substring(request.getContextPath().length() + 1);
    // 예: "food/detail.do"

    try {
      for(String cls : clsList) {
        Class clsName = Class.forName(cls);

        // 2. @Controller 있는 클래스만 처리
        if(!clsName.isAnnotationPresent(Controller.class)) continue;

        Object obj = clsName.getDeclaredConstructor().newInstance();

        // 3. 메소드에서 @RequestMapping 확인
        for(Method m : clsName.getDeclaredMethods()) {
          RequestMapping rm = m.getAnnotation(RequestMapping.class);
          if(rm.value().equals(uri)) {

            // 4. 리플렉션으로 메소드 실행
            String jsp = (String) m.invoke(obj, request, response);

            // 5. 반환값에 따라 분기
            if(jsp == null) {
              return; // Ajax 처리 등 — forward 없이 종료
            } else if(jsp.startsWith("redirect:")) {
              // redirect: 접두사가 있으면 sendRedirect
              String redirectUrl = jsp.substring(jsp.indexOf(":") + 1);
              response.sendRedirect(redirectUrl);
            } else {
              // 일반 JSP로 forward (request 유지)
              RequestDispatcher rd = request.getRequestDispatcher(jsp);
              rd.forward(request, response);
              return;
            }
          }
        }
      }
    } catch(Exception ex) { ex.printStackTrace(); }
  }
}
```

### application.xml
```xml
<beans>
  <!-- 어떤 패키지를 스캔할지 지정 -->
  <context:component-scan base-package="com.sist.model"/>
</beans>
```

---

## 5. FoodModel — Controller 역할 클래스

```java
@Controller // DispatcherServlet이 이 클래스를 처리 대상으로 인식
public class FoodModel {

  @RequestMapping("food/detail.do") // URL과 이 메소드를 연결
  public String man_main(HttpServletRequest request,
                         HttpServletResponse response) {
    // 1. 파라미터 수신
    String no = request.getParameter("no");

    // 2. DAO 호출 (MyBatis 사용)
    FoodVO vo = FoodDAO.foodDetailData(Integer.parseInt(no));

    // 3. JSP로 데이터 전달
    request.setAttribute("vo", vo);
    request.setAttribute("main_jsp", "../food/detail.jsp");

    // 4. 공통 레이아웃 JSP 반환 → DispatcherServlet이 forward
    return "../main/main.jsp";
  }
}
```

> Spring `@Controller` + `@RequestMapping` 과 완전히 동일한 구조
> 차이점은 Spring은 이 과정을 **자동**으로 처리한다는 것

---

## 6. MyBatis SqlSession 연동

### CreateSqlSessionFactory — 싱글턴 팩토리
```java
public class CreateSqlSessionFactory {
  private static SqlSessionFactory ssf;

  // static 블록 — 클래스 로딩 시 1회만 실행 (싱글턴)
  static {
    try {
      // Config.xml 읽기
      Reader reader = Resources.getResourceAsReader("Config.xml");
      // SqlSessionFactory 생성 (Map으로 SQL 저장)
      ssf = new SqlSessionFactoryBuilder().build(reader);
    } catch(Exception ex) { ex.printStackTrace(); }
  }

  public static SqlSessionFactory getSsf() { return ssf; }
}
```

### FoodDAO — MyBatis SqlSession으로 실행
```java
public class FoodDAO {
  private static SqlSessionFactory ssf;
  static { ssf = CreateSqlSessionFactory.getSsf(); }

  // 목록 조회
  public static List<FoodVO> foodListData(int start) {
    List<FoodVO> list = new ArrayList<>();
    SqlSession session = null;
    try {
      session = ssf.openSession();
      // mapper.xml의 id="foodListData" 실행, start를 parameterType으로 전달
      list = session.selectList("foodListData", start);
    } catch(Exception ex) { ex.printStackTrace(); }
    finally {
      if(session != null) session.close(); // Pool 반환 → 재사용 ⭐
    }
    return list;
  }

  // 총 페이지
  public static int foodTotalPage() {
    int total = 0;
    SqlSession session = null;
    try {
      session = ssf.openSession();
      total = session.selectOne("foodTotalPage"); // 파라미터 없음
    } catch(Exception ex) { ex.printStackTrace(); }
    finally { if(session != null) session.close(); }
    return total;
  }

  // 상세보기
  public static FoodVO foodDetailData(int no) {
    FoodVO vo = new FoodVO();
    SqlSession session = null;
    try {
      session = ssf.openSession();
      vo = session.selectOne("foodDetailData", no);
    } catch(Exception ex) { ex.printStackTrace(); }
    finally { if(session != null) session.close(); }
    return vo;
  }
}
```

### SqlSession 메소드 정리
| 메소드 | 반환형 | 용도 |
|--------|--------|------|
| `selectList("id", param)` | `List<T>` | 목록 조회 |
| `selectOne("id", param)` | `T` | 단건 조회 |
| `insert("id", param)` | `int` | 등록 |
| `update("id", param)` | `int` | 수정 |
| `delete("id", param)` | `int` | 삭제 |
| `session.commit()` | — | DML 확정 (insert/update/delete 후 필수) |
| `session.close()` | — | Pool 반환 (finally에서 필수) ⭐ |

---

## 7. Config.xml + food-mapper.xml

### Config.xml
```xml
<configuration>
  <typeAliases>
    <typeAlias type="com.sist.vo.FoodVO" alias="FoodVO"/>
  </typeAliases>

  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver"   value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url"      value="jdbc:oracle:thin:@localhost:1521:XE"/>
        <property name="username" value="hr"/>
        <property name="password" value="happy"/>
      </dataSource>
    </environment>
  </environments>

  <mappers>
    <mapper resource="food-mapper.xml"/>
  </mappers>
</configuration>
```

### food-mapper.xml
```xml
<mapper namespace="food">

  <!-- 목록 조회 (페이징) -->
  <select id="foodListData" resultType="FoodVO" parameterType="int">
    SELECT no, poster, name, address
    FROM food
    ORDER BY no ASC
    OFFSET #{start} ROWS FETCH NEXT 12 ROWS ONLY
  </select>

  <!-- 총 페이지 수 -->
  <select id="foodTotalPage" resultType="int">
    SELECT CEIL(COUNT(*) / 12.0) FROM food
  </select>

  <!-- 상세보기 -->
  <select id="foodDetailData" resultType="FoodVO" parameterType="int">
    SELECT * FROM food WHERE no = #{no}
  </select>

</mapper>
```

---

## 8. 우리가 구현한 것 vs Spring이 하는 것

```
우리 구현                    Spring 자동 처리
─────────────────────────────────────────────────
@WebServlet("*.do")      →  DispatcherServlet (web.xml 등록)
ComponentScan.java       →  @ComponentScan 자동 처리
@Controller (직접 정의)  →  @Controller (Spring 제공)
@RequestMapping (직접)   →  @RequestMapping / @GetMapping
리플렉션으로 메소드 실행  →  HandlerMapping + HandlerAdapter
redirect: 문자열 분기    →  ViewResolver가 처리
application.xml 직접 파싱→  ApplicationContext 자동 관리
SqlSessionFactory 직접   →  SqlSessionFactoryBean (Spring Bean)
```

> 오늘 구현한 Mini Spring MVC가 Spring Framework의 핵심 원리
> Spring은 이 모든 과정을 **설정 한 줄 + 어노테이션**으로 자동화한 것
