# 📘 Day 25 — MVC 프로젝트 구축법 + 파일 업로드/다운로드 + CRUD 게시판

## 0. 핵심 빠른 참조

| 구분 | 내용 | 비고 |
|------|------|------|
| DB 설정 분리 | `db.properties` | driver/url/username/password/pool 설정 |
| MyBatis 설정 | `Config.xml` | properties 불러오기, typeAlias, mapper 등록 |
| MyBatis 캐시 | `cacheEnabled=true` | 동일 쿼리 반복 시 DB 재조회 안 함 |
| 언더스코어 변환 | `mapUnderscoreToCamelCase=true` | `user_name` → `userName` 자동 변환 |
| 로그 출력 | `logImpl=STDOUT_LOGGING` | 실행 SQL 콘솔 출력 (개발 시 활용) |
| 파일 업로드 | `@MultipartConfig` + `request.getPart()` | Servlet에서만 가능 (JSP 불가) |
| 파일 저장 경로 | `getServletContext().getRealPath("")` | 서버 실제 경로 조회 |
| 파일 다운로드 | `Content-Disposition: attachment` | 브라우저에 다운로드 강제 |
| 한글 파일명 | `URLEncoder.encode(fn, "UTF-8")` | 다운로드 파일명 인코딩 필수 |
| 조회수 증가 | `session.update()` + `session.commit()` | 조회 전 UPDATE → commit → SELECT |
| 비밀번호 수정 | DB 비밀번호 SELECT → equals() 비교 | 수정 전 비밀번호 검증 패턴 |
| 근처 맛집 | `address LIKE '%'||#{address}||'%'` | 주소 두 번째 split으로 구 추출 |
| 총 페이지 (게시판) | `Math.ceil(count / (double)ROWSIZE)` | ROWSIZE=10 기준 |
| 총 페이지 (맛집) | `CEIL(COUNT(*)/12.0)` | SQL에서 직접 계산 |

---

## 1. MVC 프로젝트 만드는 순서 ⭐

```text
새 프로젝트 시작 시 이 순서대로 만들면 된다

① 프로젝트 생성 (Dynamic Web Project)
   └─ Tomcat 연결, Java Build Path 설정

② VO 작성 (테이블 컬럼 → 멤버변수)
   └─ @Data (Lombok) → getter/setter/toString 자동 생성

③ Config.xml + db.properties 작성
   └─ DB 접속 정보, typeAlias 등록, mapper 경로 등록

④ Mapper XML 작성 (SQL)
   └─ select / insert / update / delete

⑤ CreateSqlSessionFactory.java (싱글턴)
   └─ static 블록에서 Config.xml 읽어 SqlSessionFactory 1회 생성

⑥ DAO 작성
   └─ ssf.openSession() → selectList/selectOne/insert/update/delete → close()

⑦ Model 작성 (@Controller + @RequestMapping)
   └─ 파라미터 수신 → DAO 호출 → request.setAttribute → JSP 경로 반환

⑧ JSP 작성
   └─ EL(${}) + JSTL(<c:forEach>) 로 출력

⑨ DispatcherServlet 확인
   └─ application.xml에 base-package 등록 확인
```

---

## 2. Config.xml + db.properties 분리 패턴

### db.properties
```properties
driver=oracle.jdbc.driver.OracleDriver
url=jdbc:oracle:thin:@192.168.0.33:1521:XE
username=hr
password=happy
maxActive=20
maxIdle=10
maxWait=-1
```

### Config.xml
```xml
<configuration>
  <!-- db.properties 불러오기 → ${변수명} 으로 사용 -->
  <properties resource="db.properties"/>

  <settings>
    <setting name="cacheEnabled"            value="true"/>  <!-- 동일 쿼리 캐시 -->
    <setting name="mapUnderscoreToCamelCase" value="true"/>  <!-- user_name → userName -->
    <setting name="callSettersOnNulls"       value="true"/>  <!-- null도 setter 호출 -->
    <setting name="logImpl"                  value="STDOUT_LOGGING"/> <!-- SQL 로그 -->
  </settings>

  <typeAliases>
    <!-- mapper.xml에서 resultType="FoodVO" 처럼 단축 이름 사용 가능 -->
    <typeAlias type="com.sist.vo.FoodVO"       alias="FoodVO"/>
    <typeAlias type="com.sist.vo.DataBoardVO"  alias="DataBoardVO"/>
  </typeAliases>

  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver"   value="${driver}"/>
        <property name="url"      value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
        <property name="poolMaximumActiveConnections" value="${maxActive}"/>
        <property name="poolMaximumIdleConnections"   value="${maxIdle}"/>
        <property name="poolTimeToWait"               value="${maxWait}"/>
        <!-- 기본값 : maxActive=8 / maxIdle=8 / maxWait=10000ms -->
      </dataSource>
    </environment>
  </environments>

  <mappers>
    <!-- 새 mapper 추가 시 여기에 등록 필수 ⭐ -->
    <mapper resource="com/sist/mapper/food-mapper.xml"/>
    <mapper resource="com/sist/mapper/databoard-mapper.xml"/>
  </mappers>
</configuration>
```

> `db.properties` 분리 이유 : DB 접속 정보만 따로 관리 → 서버 이전/변경 시 이 파일만 수정

---

## 3. CRUD 게시판 전체 구조 (DataBoard)

### VO (DataBoardVO)
```java
@Data  // getter/setter/toString 자동
public class DataBoardVO {
    private int no, filesize, hit;
    private String name, subject, content, pwd, filename, dbday;
    private Date regdate;
    // ⭐ dbday : Oracle TO_CHAR() 결과를 받는 String 필드
    //    regdate : 날짜 원본 (Date 타입)
}
```

### Mapper XML (CRUD 전체)
```xml
<!-- 목록 -->
<select id="boardListData" resultType="DataBoardVO" parameterType="int">
    SELECT no, subject, name, TO_CHAR(regdate,'yyyy-mm-dd') as dbday, hit
    FROM mvcdataboard
    ORDER BY no DESC
    OFFSET #{start} ROWS FETCH NEXT 10 ROWS ONLY
</select>

<!-- 전체 행수 (페이지 계산용) -->
<select id="boardRowCount" resultType="int">
    SELECT COUNT(*) FROM mvcdataboard
</select>

<!-- 등록 : selectKey로 no 채번 후 INSERT -->
<insert id="boardInsert" parameterType="DataBoardVO">
    <selectKey keyProperty="no" resultType="int" order="BEFORE">
        SELECT NVL(MAX(no)+1, 1) as no FROM mvcdataboard
    </selectKey>
    INSERT INTO mvcdataboard(no, name, subject, content, pwd, filename, filesize)
    VALUES(#{no}, #{name}, #{subject}, #{content}, #{pwd}, #{filename}, #{filesize})
</insert>

<!-- 조회수 증가 + 상세보기 -->
<update id="boardHitIncrement" parameterType="int">
    UPDATE mvcdataboard SET hit=hit+1 WHERE no=#{no}
</update>
<select id="boardDetailData" parameterType="int" resultType="DataBoardVO">
    SELECT no, name, subject, content,
           TO_CHAR(regdate,'yyyy-mm-dd hh24:mi:ss') as dbday,
           filename, filesize
    FROM mvcdataboard WHERE no=#{no}
</select>

<!-- 수정 : 비밀번호 먼저 확인 후 UPDATE -->
<select id="boardGetPassword" parameterType="int" resultType="string">
    SELECT pwd FROM mvcdataboard WHERE no=#{no}
</select>
<update id="boardUpdate" parameterType="DataBoardVO">
    UPDATE mvcdataboard SET
    name=#{name}, subject=#{subject}, content=#{content},
    filename=#{filename}, filesize=#{filesize}
    WHERE no=#{no}
</update>

<!-- 삭제 전 파일 정보 조회 (파일도 함께 삭제하기 위해) -->
<select id="boardFileInfoData" parameterType="int" resultType="DataBoardVO">
    SELECT filename, filesize FROM mvcdataboard WHERE no=#{no}
</select>
<delete id="boardDelete" parameterType="int">
    DELETE FROM mvcdataboard WHERE no=#{no}
</delete>
```

---

## 4. DAO 핵심 패턴 정리

```java
// 조회 : openSession() (autoCommit X)
public static List<DataBoardVO> boardListData(int start) {
    SqlSession session = ssf.openSession();
    List<DataBoardVO> list = session.selectList("boardListData", start);
    session.close();
    return list;
}

// 등록 : openSession(true) — autoCommit ⭐
public static void boardInsert(DataBoardVO vo) {
    SqlSession session = ssf.openSession(true);
    session.insert("boardInsert", vo);
    session.close();
}

// 상세 : UPDATE(조회수) → commit() → SELECT 순서 중요 ⭐
public static DataBoardVO boardDetailData(int no) {
    SqlSession session = ssf.openSession();
    session.update("boardHitIncrement", no);
    session.commit();                              // UPDATE는 명시적 commit 필요
    DataBoardVO vo = session.selectOne("boardDetailData", no);
    session.close();
    return vo;
}

// 수정 : DB 비밀번호 검증 후 UPDATE
public static boolean boardUpdate(DataBoardVO vo) {
    boolean bCheck = false;
    SqlSession session = ssf.openSession(true);
    String db_pwd = session.selectOne("boardGetPassword", vo.getNo());
    if (db_pwd.equals(vo.getPwd())) {
        bCheck = true;
        session.update("boardUpdate", vo);
    }
    return bCheck;  // true=수정성공 / false=비밀번호불일치
}
```

| 메서드 | openSession | commit 필요 |
|--------|------------|------------|
| selectList / selectOne | `openSession()` | X |
| insert | `openSession(true)` | 자동 |
| update (단독) | `openSession(true)` | 자동 |
| update → select (같은 세션) | `openSession()` | 명시적 `commit()` 후 select |
| delete | `openSession(true)` | 자동 |

---

## 5. 게시판 페이지 계산 방식 비교

```java
// 맛집 (food) : SQL에서 계산
// SELECT CEIL(COUNT(*)/12.0) FROM food  → totalpage 바로 반환

// 게시판 (board) : Java에서 계산
int count     = DataBoardDAO.boardRowCount();    // 전체 행수
int totalpage = (int)(Math.ceil(count / (double)ROWSIZE));  // 총 페이지

// ⭐ 역순 번호 표시 (게시판에서 최신글이 1번)
count = count - ((curpage * ROWSIZE) - ROWSIZE);
// → 현재 페이지 첫 번째 글의 번호
// request.setAttribute("count", count) → JSP에서 번호로 출력
```

---

## 6. 파일 업로드 — UploadServlet ⭐

```java
// ⚠️ 파일 업로드는 반드시 Servlet으로 처리 (JSP 불가)
// Spring에서는 MultipartResolver Bean으로 처리

@WebServlet("/UploadServlet")
@MultipartConfig(
    fileSizeThreshold = 1024 * 1024,      // 1MB : 메모리 임시 저장 기준
    maxFileSize       = 1024 * 1024 * 100, // 100MB : 파일 최대 크기
    maxRequestSize    = 1024 * 1024 * 50   // 50MB : 요청 전체 최대 크기
)
public class UploadServlet extends HttpServlet {
    private static final String UPLOAD_DIR = "uploads";

    protected void doPost(HttpServletRequest request, HttpServletResponse response) {
        // 1. 업로드 실제 경로 확인
        String uploadPath = getServletContext().getRealPath("") + File.separator + UPLOAD_DIR;
        // getServletContext() = JSP의 application 내장 객체

        // 2. 폴더 없으면 생성
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists()) uploadDir.mkdir();

        // 3. 한글 인코딩
        request.setCharacterEncoding("UTF-8");

        // 4. 텍스트 파라미터 수신
        String name    = request.getParameter("name");
        String subject = request.getParameter("subject");
        String content = request.getParameter("content");
        String pwd     = request.getParameter("pwd");

        // 5. 파일 파트 수신
        Part filePart = request.getPart("files");  // input name="files"
        if (filePart == null || filePart.getSize() == 0) {
            vo.setFilename(""); vo.setFilesize(0);  // 파일 없음
        } else {
            String fileName = filePart.getSubmittedFileName();  // 원본 파일명
            filePart.write(uploadPath + File.separator + fileName); // 저장
            File f = new File(uploadPath + File.separator + fileName);
            vo.setFilename(f.getName());
            vo.setFilesize((int) f.length());
        }

        // 6. DB 저장 후 목록으로 이동
        DataBoardDAO.boardInsert(vo);
        response.sendRedirect("board/list.do");
    }
}
```

```text
업로드 흐름
insert.jsp (form enctype="multipart/form-data" action="UploadServlet")
  → UploadServlet.doPost()
      → request.getPart("files") → 서버 uploads/ 폴더에 저장
          → DataBoardDAO.boardInsert(vo) → DB 저장
              → sendRedirect("board/list.do")
```

---

## 7. 파일 다운로드 — BoardModel

```java
@RequestMapping("board/download.do")
public void board_download(HttpServletRequest request, HttpServletResponse response) {
    String fn = request.getParameter("fn");  // 파일명 파라미터
    String path = request.getServletContext().getRealPath("") + File.separator + "uploads";
    File file = new File(path + File.separator + fn);

    // 브라우저에 다운로드로 인식시키는 헤더
    response.setHeader("Content-Disposition",
        "attachment;filename=" + URLEncoder.encode(fn, "UTF-8"));  // 한글 파일명 인코딩
    response.setContentLength((int) file.length());

    // 파일 → 응답 스트림으로 전송
    BufferedInputStream  bis = new BufferedInputStream(new FileInputStream(file));
    BufferedOutputStream bos = new BufferedOutputStream(response.getOutputStream());

    byte[] buffer = new byte[1024];
    int i;
    while ((i = bis.read(buffer, 0, 1024)) != -1) {
        bos.write(buffer, 0, i);
    }
    bis.close(); bos.close();
}
```

> `Content-Disposition: attachment` → 브라우저가 파일을 열지 않고 다운로드 강제  
> `URLEncoder.encode()` → 한글 파일명 깨짐 방지 필수  
> 반환형 `void` → JSP forward 없음, 스트림으로 직접 응답

---

## 8. 맛집 상세 — 근처 맛집 추천 패턴

```java
@RequestMapping("food/detail.do")
public String food_detail(HttpServletRequest request, HttpServletResponse response) {
    String no = request.getParameter("no");
    FoodVO vo = FoodDAO.foodDetailData(Integer.parseInt(no));

    // 주소에서 '구' 추출 : "서울 강남구 역삼동" → split(" ")[2] = "역삼동"
    String[] addrs = vo.getAddress().split(" ");
    List<FoodVO> list = FoodDAO.foodRearData(addrs[2]);
    // LIKE '%역삼동%' 으로 근처 맛집 최대 7개 조회

    request.setAttribute("list", list);  // 근처 맛집
    request.setAttribute("vo", vo);      // 현재 맛집 상세
    request.setAttribute("main_jsp", "../food/detail.jsp");
    return "../main/main.jsp";
}
```

```xml
<!-- rownum : Oracle 전용, 조회 행수 제한 -->
<select id="foodRearData" resultType="FoodVO" parameterType="string">
    SELECT no, name, poster, address, rownum
    FROM food
    WHERE address LIKE '%'||#{address}||'%'
    AND rownum &lt;= 7
</select>
<!-- &lt; = < (XML 특수문자 이스케이프) -->
```

---

## 9. 실습 파일 목록

| 파일 | 역할 | 핵심 내용 |
|------|------|---------|
| `db.properties` | DB 설정 | driver/url/계정/pool 분리 관리 |
| `Config.xml` | MyBatis 설정 | properties 참조, settings, typeAlias, mapper 등록 |
| `CreateSqlSessionFactory.java` | 싱글턴 팩토리 | static 블록, SqlSessionFactory 1회 생성 |
| `FoodVO.java` | 맛집 VO | @Data, Oracle 컬럼 → Java 필드 |
| `DataBoardVO.java` | 게시판 VO | @Data, dbday(String) + regdate(Date) 이중 관리 |
| `food-mapper.xml` | 맛집 SQL | 페이징, 상세, 조회수, 근처맛집(LIKE+rownum) |
| `databoard-mapper.xml` | 게시판 SQL | CRUD 전체 + selectKey + boardHitIncrement |
| `FoodDAO.java` | 맛집 DAO | foodListData / foodDetailData / foodRearData |
| `DataBoardDAO.java` | 게시판 DAO | 전체 CRUD + 비밀번호 검증 후 update |
| `UploadServlet.java` | 파일 업로드 | @MultipartConfig, getPart(), write() |
| `FoodModel.java` | 맛집 Model | list/detail_before/detail, Cookie, 근처맛집 |
| `BoradModel.java` | 게시판 Model | list/insert/detail/download, 역순번호 |
| `MainModel.java` | 메인 Model | home.jsp 연결 |

---
