# 📘 Day 24 — MVC 총정리 (Cookie/Session/댓글) + jQuery 효과

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| Cookie 생성 | `new Cookie("key", "value")` | 브라우저에 저장 |
| Cookie 유효기간 | `cookie.setMaxAge(초)` | 60\*60\*24 = 하루 |
| Cookie 경로 | `cookie.setPath("/")` | 전체 경로 접근 허용 |
| Cookie 응답 | `response.addCookie(cookie)` | 브라우저로 전송 |
| Cookie 읽기 | `request.getCookies()` | 배열 반환, null 체크 필수 |
| Session 생성 | `request.getSession()` | 서버에 저장 |
| Session 저장 | `session.setAttribute("key", value)` | Object 단위 저장 |
| Session 읽기 | `session.getAttribute("key")` | Object 반환 → 캐스팅 |
| Session 삭제 | `session.invalidate()` | 로그아웃 시 전체 해제 |
| MyBatis 자동커밋 | `ssf.openSession(true)` | INSERT/UPDATE/DELETE 시 사용 |
| MyBatis 조회 | `session.selectList("id", param)` / `selectOne("id", param)` | 목록 / 단건 |
| redirect | `return "redirect:../경로"` | request 초기화, URL 변경 |
| forward | `return "../경로.jsp"` | request 유지, URL 유지 |
| jQuery hide/show | `$().hide(ms)` / `$().show(ms)` | 표시/숨기기 (ms: 속도) |
| jQuery fade | `$().fadeIn(ms)` / `$().fadeOut(ms)` | 투명도로 표시/숨기기 |
| jQuery slide | `$().slideDown(ms)` / `$().slideUp(ms)` | 슬라이드 효과 |
| jQuery animate | `$().animate({속성}, ms)` | 커스텀 애니메이션 |
| jQuery 클래스 | `$().addClass()` / `$().removeClass()` / `$().toggleClass()` | 클래스 제어 |
| jQuery 삽입 위치 | `append` / `prepend` / `after` | 뒤 / 앞 / 외부 다음 |

---

## 1. 전체 프로젝트 구조

```text
JSPFrontProject_7
├── com.sist.mapper
│     └── food-mapper.xml
│         member-mapper.xml
│         reply-mapper.xml
│
├── com.sist.commons
│     └── CreateSqlSessionFactory.java   MyBatis 싱글턴 팩토리
│
├── com.sist.model  (@Controller)
│     ├── MainModel.java    메인/로그인/로그아웃
│     ├── FoodModel.java    맛집 상세 + Cookie
│     └── ReplyModel.java   댓글 등록
│
├── com.sist.dao
│     ├── FoodDAO.java      맛집 목록/상세
│     ├── MemberDAO.java    로그인 처리
│     └── ReplyDAO.java     댓글 목록/등록
│
├── com.sist.vo
│     ├── FoodVO.java
│     ├── MemberVO.java     (msg 필드 포함 : OK/NOID/NOPWD)
│     └── ReplyVO.java
|
├── Config.xml
```

---

## 2. MyBatis 연결 — CreateSqlSessionFactory (싱글턴)

```java
// static 블록 : 클래스 로딩 시 딱 1번만 실행
// 프로그램 전체에서 SqlSessionFactory 1개만 생성 → 재사용
public class CreateSqlSessionFactory {
    @Getter
    private static SqlSessionFactory ssf;

    static {
        Reader reader = Resources.getResourceAsReader("Config.xml");
        ssf = new SqlSessionFactoryBuilder().build(reader);
    }
}

// DAO에서 사용
private static SqlSessionFactory ssf;
static { ssf = CreateSqlSessionFactory.getSsf(); }
```

> `ssf.openSession()` → 조회용 (autoCommit X)  
> `ssf.openSession(true)` → INSERT/UPDATE/DELETE용 (autoCommit O) ⭐

---

## 3. Mapper XML 핵심 패턴

### food-mapper.xml
```xml
<!-- 목록 (페이징) -->
<select id="foodListData" resultType="FoodVO" parameterType="int">
    SELECT no, name, poster, address FROM food
    ORDER BY no ASC
    OFFSET #{start} ROWS FETCH NEXT 12 ROWS ONLY
</select>

<!-- 총 페이지 수 -->
<select id="foodTotalPage" resultType="int">
    SELECT CEIL(COUNT(*) / 12.0) FROM food
</select>

<!-- 상세 -->
<select id="foodDetailData" resultType="FoodVO" parameterType="int">
    SELECT * FROM food WHERE no = #{no}
</select>
```

### member-mapper.xml
```xml
<!-- ID 존재 여부 먼저 확인 -->
<select id="memberIdCount" resultType="int" parameterType="string">
    SELECT COUNT(*) FROM member WHERE id = #{id}
</select>

<!-- 비밀번호 비교용 -->
<select id="memberGetPassword" resultType="MemberVO" parameterType="string">
    SELECT id, name, pwd FROM member WHERE id = #{id}
</select>
```

### reply-mapper.xml
```xml
<!-- 댓글 목록 (최신순) -->
<select id="replyListData" resultType="ReplyVO" parameterType="int">
    SELECT no, fno, id, name, msg,
           TO_CHAR(regdate, 'YYYY-MM-DD HH24:MI:SS') as dbday
    FROM reply WHERE fno = #{fno}
    ORDER BY no DESC
</select>

<!-- 댓글 등록 : Sequence 없이 MAX(no)+1 방식 -->
<insert id="replyInsert" parameterType="ReplyVO">
    <selectKey keyProperty="no" resultType="int" order="BEFORE">
        SELECT NVL(MAX(no)+1, 1) as no FROM reply
    </selectKey>
    INSERT INTO reply VALUES(#{no}, #{fno}, #{id}, #{name}, #{msg}, SYSDATE)
</insert>
```

> `<selectKey order="BEFORE">` : INSERT 실행 전에 no 값을 먼저 계산 후 VO에 주입

---

## 4. Cookie — 최근 본 맛집 (FoodModel)

```java
// ⚠️ response는 동시에 2가지 일을 못 함 (Cookie 응답 + HTML 응답 불가)
// → Cookie 저장 후 redirect → 새 요청에서 HTML 응답

@RequestMapping("food/detail_before.do")
public String food_detail_before(HttpServletRequest request, HttpServletResponse response) {
    String no = request.getParameter("no");

    // 1. Cookie 생성 (key: food_1, value: 1)
    Cookie cookie = new Cookie("food_" + no, no);
    cookie.setMaxAge(60 * 60 * 24);  // 유효기간 : 하루
    cookie.setPath("/");              // 전체 경로 접근 허용
    response.addCookie(cookie);       // 브라우저에 전송

    // 2. redirect → request 초기화, URL 변경 (새 요청)
    return "redirect:../food/detail.do?no=" + no;
}
```

```java
// Cookie 읽기 (MainModel — 최근 본 맛집 목록)
Cookie[] cookies = request.getCookies();
List<FoodVO> cList = new ArrayList<>();
int j = 0;
if (cookies != null) {  // ⭐ null 체크 필수
    for (int i = cookies.length - 1; i >= 0; i--) {  // 최신순 역순
        if (cookies[i].getName().startsWith("food_")) {
            if (j >= 9) break;  // 최대 9개
            String no = cookies[i].getValue();
            cList.add(FoodDAO.foodDetailData(Integer.parseInt(no)));
            j++;
        }
    }
}
request.setAttribute("cList", cList);
```

| | Cookie | Session |
|--|--------|---------|
| 저장 위치 | 브라우저 | 서버 |
| 저장 타입 | 문자열만 | Object 모두 |
| 보안 | 취약 | 안전 |
| 용도 | 자동 로그인, 장바구니, 최근 방문 | 로그인 정보 유지 |
| 생성 | `new Cookie(key, value)` | `request.getSession()` |

---

## 5. Session — 로그인/로그아웃 (MainModel)

```java
// 로그인 처리 : Ajax로 호출 → JSON 텍스트 반환 (void)
@RequestMapping("member/login.do")
public void member_login(HttpServletRequest request, HttpServletResponse response) {
    String id  = request.getParameter("id");
    String pwd = request.getParameter("pwd");

    MemberVO vo = MemberDAO.memberLogin(id, pwd);

    if (vo.getMsg().equals("OK")) {
        HttpSession session = request.getSession();
        session.setAttribute("id",   vo.getId());    // 서버 메모리에 저장
        session.setAttribute("name", vo.getName());
    }
    // 결과 문자열 브라우저로 전송 (OK / NOID / NOPWD)
    response.setContentType("text/plain;charset=UTF-8");
    response.getWriter().println(vo.getMsg());
}

// 로그아웃
@RequestMapping("member/logout.do")
public void member_logout(HttpServletRequest request, HttpServletResponse response) {
    request.getSession().invalidate();  // 세션 전체 해제
    response.getWriter().println("yes");
}
```

### MemberDAO 로그인 로직
```java
public static MemberVO memberLogin(String id, String pwd) {
    MemberVO vo = new MemberVO();
    SqlSession session = ssf.openSession();

    int count = session.selectOne("memberIdCount", id);
    if (count == 0) {
        vo.setMsg("NOID");   // ID 없음
    } else {
        MemberVO dbVO = session.selectOne("memberGetPassword", id);
        if (pwd.equals(dbVO.getPwd())) {
            vo.setId(dbVO.getId());
            vo.setName(dbVO.getName());
            vo.setMsg("OK");     // 로그인 성공
        } else {
            vo.setMsg("NOPWD"); // 비밀번호 불일치
        }
    }
    session.close();
    return vo;
}
// 결과 : OK / NOID / NOPWD → 브라우저 JS에서 분기 처리
```

---

## 6. 댓글 등록 흐름 (ReplyModel + ReplyDAO)

```java
// ReplyModel : 세션에서 로그인 정보 꺼내서 VO에 담아 DAO 호출
@RequestMapping("reply/insert.do")
public String reply_insert(HttpServletRequest request, HttpServletResponse response) {
    String fno = request.getParameter("fno");
    String msg = request.getParameter("msg");

    // Session에서 로그인 정보 꺼내기
    HttpSession session = request.getSession();
    String id   = (String) session.getAttribute("id");
    String name = (String) session.getAttribute("name");

    ReplyVO vo = new ReplyVO();
    vo.setFno(Integer.parseInt(fno));
    vo.setMsg(msg);
    vo.setId(id);
    vo.setName(name);

    ReplyDAO.replyInsert(vo);
    return "redirect:../food/detail.do?no=" + fno;  // 상세 페이지로 복귀
}

// ReplyDAO : autoCommit(true)로 즉시 반영
public static void replyInsert(ReplyVO vo) {
    SqlSession session = ssf.openSession(true);  // ⭐ true = 자동 커밋
    session.insert("replyInsert", vo);
    session.close();
}
```

```text
댓글 등록 전체 흐름
detail.jsp (댓글 폼 submit)
  → reply/insert.do
      → ReplyModel : Session에서 id/name 꺼냄 → ReplyDAO.replyInsert()
          → reply-mapper.xml : selectKey(MAX+1) → INSERT
              → redirect:../food/detail.do?no={fno}
                  → FoodModel.food_detail() → detail.jsp 재출력
```

---

## 7. forward vs redirect 정리

```text
forward  : return "../food/detail.jsp"
  → request 유지 (setAttribute 데이터 살아있음)
  → URL 변경 없음
  → 브라우저가 모름 (서버 내부 이동)

redirect : return "redirect:../food/detail.do?no=1"
  → request 초기화 (새 요청)
  → URL 변경됨
  → 브라우저가 새 URL로 재요청
  → Cookie 응답 후 HTML 응답 분리할 때 필수 ⭐
  → 중복 submit 방지에 활용
```

---

## 8. jQuery 효과 + UI 패턴

### 효과 함수 (jquery_5.jsp)
```javascript
const duration = 1000  // 1초

$('#box').hide(duration)       // 숨기기 (크기 + 투명도 동시)
$('#box').show(duration)       // 보이기
$('#box').fadeOut(duration)    // 투명도만으로 숨기기
$('#box').fadeIn(duration)     // 투명도만으로 보이기
$('#box').slideUp(duration)    // 위로 접기
$('#box').slideDown(duration)  // 아래로 펼치기

// animate : CSS 속성 직접 지정
$('#box').stop().animate({
    left: '200px',
    width: '300px',
    opacity: '0.5'
}, duration)
// stop() : 진행 중인 애니메이션 즉시 중단 후 새로 시작 (겹침 방지)
```

### 탭 메뉴 (jquery_3.jsp / jquery_4.jsp)
```javascript
// 클릭한 탭만 active, 나머지 제거 → 실무에서 자주 쓰는 패턴 ⭐
$('.menu li').on('click', function() {
    $('.menu li').removeClass('active')  // 전체 제거
    $(this).addClass('active')           // 클릭한 것만 추가

    // data-tab 속성으로 연결된 콘텐츠 영역 제어
    $('.content').removeClass('active')
    const tab = $(this).attr('data-tab')
    $('#tab' + tab).addClass('active')
})
// HTML : <li data-tab="1">홈</li> / <div id="tab1" class="content">
```

### 태그 삽입 위치 비교 (jquery_2.jsp)
```javascript
$('#list').append(newItem)    // ul 내부 맨 뒤에 추가
$('#list').prepend(newItem)   // ul 내부 맨 앞에 추가
$('#list').after(newItem)     // ul 바깥 다음에 추가 (형제)
```

### 라이트박스 슬라이더 핵심 (jquery_7.jsp)
```javascript
var $imgs = $('.gallery img')
var currentIndex = 0

// 이미지 변경 : fadeOut 후 src 교체 → fadeIn
function changeImage(index) {
    currentIndex = index
    $('#lightbox-img').fadeOut(150, function() {
        $(this).attr('src', $imgs.eq(currentIndex).attr('src')).fadeIn(200)
    })
}

$imgs.click(function() {
    currentIndex = $imgs.index(this)  // 클릭한 이미지 인덱스
    changeImage(currentIndex)
    $('#lightbox').css('display', 'flex').hide().fadeIn(300)
})

$('#prev').click(function() {
    var next = currentIndex - 1
    if (next < 0) next = $imgs.length - 1  // 첫 장이면 마지막으로
    changeImage(next)
})
// $imgs.index(this) : 전체 배열에서 클릭한 요소의 위치(인덱스) 반환
```

### Swiper 슬라이더 CDN (jquery_6.jsp)
```javascript
// CDN만 추가하면 자동 슬라이드 완성
const swiper = new Swiper(".mySwiper", {
    loop: true,
    autoplay: { delay: 2000 },
    pagination: { el: '.swiper-pagination', clickable: true },
    navigation: { nextEl: '.swiper-button-next', prevEl: '.swiper-button-prev' }
})
```

---

## 9. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `MainModel.java` | 메인/로그인/로그아웃 | Cookie 읽기, Session 저장/해제, 페이징 |
| `FoodModel.java` | 맛집 상세 + Cookie | Cookie 생성, redirect 분리 |
| `ReplyModel.java` | 댓글 등록 | Session에서 id/name 꺼내기, replyInsert |
| `FoodDAO.java` | 맛집 DAO | selectList / selectOne / openSession() |
| `MemberDAO.java` | 로그인 DAO | ID 존재확인 → 비밀번호 비교 → msg 반환 |
| `ReplyDAO.java` | 댓글 DAO | openSession(true) 자동커밋 |
| `food-mapper.xml` | 맛집 SQL | 페이징 OFFSET / FETCH, 상세 |
| `member-mapper.xml` | 회원 SQL | COUNT로 ID 확인, 비밀번호 SELECT |
| `reply-mapper.xml` | 댓글 SQL | selectKey MAX+1, INSERT |
| `jquery_1.jsp` | hide/show/toggle | 기본 표시 제어 |
| `jquery_2.jsp` | 태그 삽입 | append / prepend / after 비교 |
| `jquery_3.jsp` | 탭 메뉴 | data-tab 속성 + addClass/removeClass |
| `jquery_4.jsp` | 메뉴 active | $(this).addClass / 전체 removeClass |
| `jquery_5.jsp` | 애니메이션 | hide/fade/slide/animate + stop() |
| `jquery_6.jsp` | Swiper CDN | 자동 슬라이더 |
| `jquery_7.jsp` | 라이트박스 | fadeOut 콜백 + index() |

---
