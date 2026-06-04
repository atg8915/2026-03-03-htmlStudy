# 📘 Day 16 — Java 어노테이션 / XML / MyBatis 심화 / MVC 총정리

## 0. 핵심 빠른 참조

| 구분 | 설명 |
|------|------|
| `@Target` | 어노테이션을 붙일 위치 지정 (METHOD / FIELD / TYPE) |
| `@Retention(RUNTIME)` | 실행 중에도 어노테이션 정보 유지 (가장 많이 사용) |
| `isAnnotationPresent()` | 메소드에 특정 어노테이션 있는지 리플렉션으로 확인 |
| XML 5대 규칙 | 루트 1개, 대소문자 구분, 반드시 닫기, 올바른 네스팅, 속성값 따옴표 |
| DOM 파싱 | 전체 메모리 로딩 → Tree 구조 → 수정/탐색 자유 |
| SAX 파싱 | 순차 읽기 → 이벤트 처리 → 대용량 적합 |
| `#{}` | PreparedStatement — SQL 인젝션 방지 ⭐ |
| `${}` | Statement — 컬럼명/테이블명 동적 지정 시만 사용 |
| `<where>` | AND/OR 자동 제거 + WHERE 자동 추가 |
| `<set>` | 불필요한 콤마(,) 자동 제거 + SET 자동 추가 |
| `<foreach>` | IN 절 동적 생성 |
| `<sql>` + `<include>` | SQL 조각 재사용 |
| `mapUnderscoreToCamelCase` | user_name → userName 자동 매핑 |

---

## 1. Java 어노테이션

> 코드에 붙이는 스마트한 포스트잇 — 컴파일러/프레임워크가 읽고 약속된 작업 수행
> XML 설정 파일을 대체하여 코드가 직관적이고 간결해짐

### 메타 어노테이션
```java
@Target(ElementType.METHOD)       // 메소드에만 적용 가능
@Retention(RetentionPolicy.RUNTIME) // 실행 중에도 정보 유지 ⭐
public @interface WeekendBlock {
  String message() default "주말에는 접속할 수 없습니다!";
}
```

| `@Target` 설정값 | 적용 위치 |
|-----------------|-----------|
| `ElementType.METHOD` | 메소드 |
| `ElementType.FIELD` | 멤버변수 |
| `ElementType.TYPE` | 클래스/인터페이스 |

| `@Retention` 설정값 | 유지 시점 |
|--------------------|-----------|
| `SOURCE` | 컴파일하면 사라짐 |
| `CLASS` | 클래스 파일까지 남음 |
| `RUNTIME` | 실행 중에도 유지 ⭐ |

### 어노테이션 적용 + 리플렉션으로 읽기
```java
// 적용
public class WorkService {
  @WeekendBlock
  public void startWork() { System.out.println("업무 시작"); }
}

// 리플렉션으로 읽기
Method method = service.getClass().getMethod("startWork");

if(method.isAnnotationPresent(WeekendBlock.class)) {
  DayOfWeek today = LocalDate.now().getDayOfWeek();
  if(today == DayOfWeek.SATURDAY || today == DayOfWeek.SUNDAY) {
    WeekendBlock ann = method.getAnnotation(WeekendBlock.class);
    System.out.println("❌ " + ann.message());
    return;  // 실행 차단
  }
}
service.startWork();  // 평일이면 정상 실행
```

> Spring의 `@Transactional`, `@WebServlet`, `@Controller` 등이 모두 이 원리로 동작

---

## 2. XML 핵심 정리

> 태그를 내 맘대로 정의해서 데이터에 이름표를 붙이는 구조화된 문서 형식
> Spring의 `applicationContext.xml`, MyBatis의 `config.xml`, `mapper.xml` 등에 활용

### XML 5대 규칙
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 1. 루트 엘리먼트는 반드시 1개 -->
<library>

  <!-- 2. 모든 태그는 반드시 닫기 -->
  <!-- 3. 대소문자 엄격 구분 (<Book> ≠ <book>) -->
  <!-- 4. 올바른 네스팅 (열린 순서 역순으로 닫기) -->
  <!-- 5. 속성값은 반드시 따옴표 -->
  <book id="101">
    <title>자바의 정석</title>
    <author>남궁성</author>
  </book>

</library>
```

### DOM 파싱 코드
```java
// 1. 파서 생성
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new File("library.xml"));
doc.getDocumentElement().normalize();  // 구조 정상화

// 2. 태그 목록 추출
NodeList bookList = doc.getElementsByTagName("book");

// 3. 반복 처리
for(int i = 0; i < bookList.getLength(); i++) {
  Element book = (Element) bookList.item(i);
  String id     = book.getAttribute("id");          // 속성 읽기
  String title  = book.getElementsByTagName("title")
                      .item(0).getTextContent();    // 태그 내용 읽기
  System.out.println(id + " : " + title);
}
```

---

## 3. MyBatis 심화 — config.xml 설정

> 태그 순서 필수: `properties → settings → typeAliases → environments → mappers`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

  <!-- 외부 properties 파일 연결 (DB 계정 보안) -->
  <properties resource="config/db.properties"/>

  <!-- 전역 설정 -->
  <settings>
    <!-- user_name → userName 자동 변환 ⭐ -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <!-- null값도 setter 호출 -->
    <setting name="callSettersOnNulls" value="true"/>
  </settings>

  <!-- 클래스 별칭 (XML에서 짧게 사용) -->
  <typeAliases>
    <typeAlias type="com.example.model.User" alias="User"/>
  </typeAliases>

  <!-- DB 연결 환경 -->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver"   value="${db.driver}"/>
        <property name="url"      value="${db.url}"/>
        <property name="username" value="${db.username}"/>
        <property name="password" value="${db.password}"/>
      </dataSource>
    </environment>
  </environments>

  <!-- Mapper XML 등록 -->
  <mappers>
    <mapper resource="mapper/UserMapper.xml"/>
  </mappers>

</configuration>
```

---

## 4. MyBatis 심화 — mapper.xml

### resultMap — 컬럼명 ≠ 필드명 매핑
```xml
<!-- DB: user_name ↔ Java: userName -->
<resultMap id="userResultMap" type="User">
  <id     property="userId"    column="user_id"/>
  <result property="userName"  column="user_name"/>
  <result property="userEmail" column="email"/>
</resultMap>

<select id="getUser" parameterType="long" resultMap="userResultMap">
  SELECT user_id, user_name, email FROM users WHERE user_id = #{id}
</select>
```

### useGeneratedKeys — 자동 생성 PK 반환
```xml
<insert id="insertUser" parameterType="User"
        useGeneratedKeys="true" keyProperty="userId">
  INSERT INTO users (user_name, email) VALUES (#{userName}, #{userEmail})
</insert>
```

### sql + include — SQL 조각 재사용
```xml
<sql id="userColumns">
  user_id, user_name, email, create_date
</sql>

<select id="selectAllUsers" resultType="User">
  SELECT <include refid="userColumns"/>
  FROM users
</select>
```

---

## 5. MyBatis 동적 SQL ⭐

### if — 조건부 쿼리
```xml
<select id="findUser" resultType="User">
  SELECT * FROM users WHERE status = 'ACTIVE'
  <if test="userName != null and userName != ''">
    AND user_name LIKE CONCAT('%', #{userName}, '%')
  </if>
</select>
```

### where — AND/OR 자동 제거
```xml
<!-- 조건이 없으면 WHERE 자체가 붙지 않음 / 맨 앞 AND 자동 제거 -->
<select id="search" resultType="User">
  SELECT * FROM users
  <where>
    <if test="email != null">email = #{email}</if>
    <if test="name  != null">AND user_name = #{name}</if>
  </where>
</select>
```

### set — UPDATE 동적 컬럼 / 콤마 자동 제거
```xml
<update id="updateUserDynamic">
  UPDATE users
  <set>
    <if test="userName  != null">user_name = #{userName},</if>
    <if test="userEmail != null">email = #{userEmail},</if>
  </set>
  WHERE user_id = #{userId}
</update>
```

### choose — switch-case
```xml
<select id="findUserOrder" resultType="User">
  SELECT * FROM users
  <choose>
    <when test="sortBy == 'name'">ORDER BY user_name ASC</when>
    <when test="sortBy == 'date'">ORDER BY create_date DESC</when>
    <otherwise>ORDER BY user_id DESC</otherwise>
  </choose>
</select>
```

### foreach — IN 절 동적 생성
```xml
<select id="selectUserList" resultType="User">
  SELECT * FROM users WHERE user_id IN
  <foreach collection="idList" item="id"
           open="(" separator="," close=")">
    #{id}
  </foreach>
</select>
<!-- 결과: WHERE user_id IN (1,2,3) -->
```

### 동적 SQL 태그 요약
| 태그 | 역할 | Java 대응 |
|------|------|-----------|
| `<if>` | 단순 조건 | if문 |
| `<choose>/<when>/<otherwise>` | 다중 조건 | switch-case |
| `<where>` | AND/OR 자동 제거 + WHERE 추가 | - |
| `<set>` | 콤마 자동 제거 + SET 추가 | - |
| `<foreach>` | 컬렉션 반복 → IN 절 | for-each |
| `<sql>/<include>` | SQL 조각 재사용 | 메소드 추출 |
| `<trim>` | prefix/suffix 커스텀 제어 | - |

---

## 6. JSP MVC 최종 정리

### 전체 요청 흐름
```
① HTTP 요청 (/board/list.do)
  ↓
② Controller (Servlet)
  ↓ DAO/Service 호출
③ Model (DAO → DB → DTO 반환)
  ↓ request.setAttribute("list", list)
④ Controller → forward()
  ↓
⑤ View (JSP) — EL/JSTL 출력
  ↓
⑥ HTML 응답 → 브라우저
```

### MVC 역할 분리 규칙
```
Controller (Servlet)    요청 받기 / Service 호출 / View 선택
Service                 비즈니스 로직 / 트랜잭션 처리
DAO                     SQL 실행 / DB 접근

JSP (View) 금지 ❌      JSP (View) 허용 ✅
DB 연결                 EL ${}
SQL 작성                JSTL 태그
복잡한 Java 코드         HTML 출력
```

### forward vs sendRedirect 최종 정리
| 구분 | forward | sendRedirect |
|------|---------|--------------|
| 이동 방식 | 서버 내부 이동 | 브라우저 재요청 |
| URL 변경 | ❌ | ✅ |
| request 유지 | ✅ | ❌ 초기화 |
| 사용 시점 | 조회(SELECT) 후 화면 전환 | 등록/수정/삭제 후 이동 |
| 코드 | `dispatcher.forward(req, res)` | `response.sendRedirect(url)` |

> PRG 패턴 — POST(등록) → Redirect → GET(목록) : 새로고침 중복 제출 방지

### Front Controller 패턴 (Spring의 모태)
```
기존 MVC
  요청마다 별도 Servlet → Servlet 파일 무수히 증가

Front Controller (DispatcherServlet 원형)
  모든 요청 → 하나의 Servlet → URL/cmd로 분기 → 각 Model 호출
  → Spring의 DispatcherServlet이 이 구조를 자동화
```

---

## 7. 기술 발전 전체 흐름

```
JSP (Model 1 — All-in-one)
  ↓
MV  (JSP + Model 분리)
  ↓
MVC (Servlet Controller — cmd 방식)
  ↓
MVC + Front Controller (*.do 방식 + XML 매핑)
  ↓
Spring Framework (DI + AOP + DispatcherServlet)
  ↓
Spring-Boot (자동 설정 + 내장 톰캣) ⭐

JDBC → DBCP → MyBatis → JPA
XML 설정 → 어노테이션 설정
```
