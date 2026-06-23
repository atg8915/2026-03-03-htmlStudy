# 📘 Day 26 — 예약 시스템 + 마이페이지 + 관리자 페이지 + Ajax 화면 분할 구조

## 0. 핵심 빠른 참조

| 구분 | 내용 | 비고 |
|------|------|------|
| 화면 분할 include | `<jsp:include page="${변수}">` | 변수로 동적 include ⭐ |
| JSTL 다중 조건 | `<c:choose><c:when><c:otherwise>` | if-else if-else |
| JSTL 변수 | `<c:set var="k" value="0"/>` | JSP 안에서 변수 선언 |
| JSTL 토큰 분리 | `<c:forTokens items="${time}" delims=",">` | 콤마 구분 문자열 반복 |
| 날짜 | `new Date()` + `SimpleDateFormat("yyyy-M-d")` | 오늘 날짜 문자열 변환 |
| 달력 | `Calendar.getInstance()` | 년/월/일/요일/마지막날 계산 |
| 요일 시작 | `cal.get(DAY_OF_WEEK) - 1` | 일=1 → 0으로 보정 |
| 마지막 날 | `cal.getActualMaximum(Calendar.DATE)` | 해당 월의 마지막 날 |
| 문자열 분리 | `StringTokenizer(str, "-")` | `-` 기준으로 분리 |
| 랜덤 배열 | `Math.random() * length` | 중복 없는 랜덤 인덱스 |
| 중복 제거 후 정렬 | `Arrays.sort(com)` | 시간 슬롯 오름차순 정렬 |
| 세션 EL 출력 | `${sessionScope.name}` | JSP에서 세션값 직접 출력 |
| Ajax 화면 교체 | `$('#영역').html(result)` | 서버 HTML 조각 → 영역 교체 |
| 실시간 알림 | Stomp (WebSocket 라이브러리) | 승인 버튼 → 알림 전송 |

---

## 1. 전체 프로젝트 페이지 구조

```text
main.jsp (공통 레이아웃)
  └─ <jsp:include page="${main_jsp}">   ← Model이 지정
       ├─ food/list.jsp
       ├─ food/detail.jsp
       ├─ board/list.jsp
       └─ mypage/mypage_main.jsp        ← 마이페이지 레이아웃
            └─ <jsp:include page="${mypage_jsp}">
                 ├─ mypage_home.jsp     (회원정보)
                 ├─ jjim_list.jsp       (찜 목록)
                 ├─ cart_list.jsp       (장바구니)
                 └─ buy_list.jsp        (구매 목록)

adminpage/admin_main.jsp (관리자 전용 레이아웃)
  └─ <jsp:include page="${admin_jsp}">
       └─ admin_home.jsp

reserve/reserve_main.jsp (예약 전용 레이아웃 — Ajax 조립)
  └─ #food_list   ← reserve_food.jsp (Ajax)
  └─ #food_rdays  ← diary.jsp        (Ajax)
  └─ #r_time      ← reserve_time.jsp (Ajax)
  └─ #r_inwon     ← reserve_inwon.jsp (Ajax)
```

> ⭐ `<jsp:include page="${변수}">` 패턴 : Model에서 `setAttribute("main_jsp", "경로")`로 지정  
> → 레이아웃(틀)은 고정, 내용(콘텐츠)만 동적으로 교체

---

## 2. 마이페이지 구조 (MyPageModel + mypage_main.jsp)

```java
// MyPageModel : 모든 메서드가 동일한 패턴
// mypage_jsp (내용) + main_jsp (레이아웃) 두 개를 setAttribute
@RequestMapping("mypage/mypage_main.do")
public String mypage_main(HttpServletRequest request, HttpServletResponse response) {
    request.setAttribute("mypage_jsp", "../mypage/mypage_home.jsp"); // 내용
    request.setAttribute("main_jsp",   "../mypage/mypage_main.jsp"); // 레이아웃
    return "../main/main.jsp";  // 공통 레이아웃
}
// jjim_list.do / cart_list.do / buy_list.do 모두 동일 구조
// mypage_jsp 경로만 바뀜
```

```jsp
<%-- mypage_main.jsp : 사이드바 + 동적 콘텐츠 --%>
<div class="mypage-container">
  <aside class="mypage-sidebar">
    <h3>${sessionScope.name}님</h3>   <%-- 세션값 EL 직접 출력 --%>
    <ul class="menu">
      <li><a href="../mypage/cart_list.do">장바구니 관리</a></li>
      <li><a href="../mypage/buy_list.do">결제 관리</a></li>
      <li><a href="../mypage/jjim_list.do">찜 관리</a></li>
    </ul>
  </aside>
  <jsp:include page="${mypage_jsp}"/>  <%-- 동적 콘텐츠 영역 --%>
</div>
```

---

## 3. 관리자 페이지 구조 (AdminPageModel)

```java
// 마이페이지와 동일한 include 패턴
// admin_main.jsp가 레이아웃, admin_jsp가 콘텐츠
@RequestMapping("adminpage/admin_main.do")
public String adminpage_main(HttpServletRequest request, HttpServletResponse response) {
    request.setAttribute("admin_jsp", "../adminpage/admin_home.jsp");
    return "../adminpage/admin_main.jsp";
    // ⚠️ main.jsp 거치지 않고 admin_main.jsp 직접 반환
    //    → 관리자 전용 레이아웃 분리
}
```

```text
일반 페이지 : main.jsp → ${main_jsp}
마이페이지  : main.jsp → mypage_main.jsp → ${mypage_jsp}  (2단계 include)
관리자 페이지: admin_main.jsp → ${admin_jsp}              (별도 레이아웃)
```

---

## 4. 예약 시스템 흐름 — Ajax 단계별 조립 ⭐

```text
① reserve_main.jsp 로드
   └─ $(function(){ $.ajax → reserve_food.do (type=한식) })
        └─ #food_list에 reserve_food.jsp 삽입

② 카테고리 버튼 클릭 (한식/양식/일식/중식)
   └─ $.ajax → reserve_food.do?type=양식
        └─ #food_list 교체

③ 맛집 클릭 (food-item)
   └─ data-poster / data-name 속성 읽어 우측 예약정보 표시
   └─ $.ajax → diary.do
        └─ #food_rdays에 diary.jsp (달력) 삽입

④ 날짜 클릭 (day-link)
   └─ 예약일 텍스트 표시
   └─ $.ajax → reserve_time.do
        └─ #r_time에 reserve_time.jsp (시간 버튼) 삽입

⑤ 시간 버튼 클릭 (times)
   └─ 예약시간 텍스트 표시
   └─ $.ajax → reserve_inwon.do
        └─ #r_inwon에 reserve_inwon.jsp (인원 버튼) 삽입

⑥ 인원 버튼 클릭 (inwons)
   └─ 예약인원 텍스트 표시
   └─ #ok (예약하기 버튼) show()
```

> ⭐ 각 단계가 Ajax로 서버에서 HTML 조각을 받아 영역에 `html(result)`로 삽입  
> → 전체 페이지 새로고침 없이 단계별로 화면 구성

---

## 5. data-* 속성 패턴 (HTML 커스텀 속성)

```jsp
<%-- JSP : 태그에 data-* 속성으로 값 숨기기 --%>
<tr class="food-item"
    data-no="${vo.no}"
    data-poster="${vo.poster}"
    data-name="${vo.name}">
```

```javascript
// jQuery : data-* 속성 읽기
$('.food-item').on('click', function() {
    let poster = $(this).attr('data-poster')
    let name   = $(this).attr('data-name')
    $('#food_poster').attr('src', poster)
    $('#food_name').text(name)
})
// HTML 속성 = 사용자 정의가 가능 → 값을 숨긴 채로 JS에서 읽기 ⭐
// input hidden과 동일한 역할이지만 태그에 붙여서 사용
```

---

## 6. Calendar — 달력 로직 (ReserveModel + diary.jsp)

```java
// 오늘 날짜 추출
Date date = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-M-d");
// MM=01~12 / M=1~12 (앞 0 없음) ← 파라미터와 비교하려면 M 사용
String today = sdf.format(date);  // "2026-6-15"
StringTokenizer st = new StringTokenizer(today, "-");
String sYear  = st.nextToken();  // "2026"
String sMonth = st.nextToken();  // "6"
String sDay   = st.nextToken();  // "15"

// Calendar로 달력 계산
Calendar cal = Calendar.getInstance();
cal.set(Calendar.YEAR,  year);
cal.set(Calendar.MONTH, month - 1); // ⭐ MONTH는 0부터 시작 → -1 필수
cal.set(Calendar.DATE,  1);          // 해당 월 1일로 설정

int week    = cal.get(Calendar.DAY_OF_WEEK) - 1; // 1일의 요일 (일=0, 월=1...)
int lastday = cal.getActualMaximum(Calendar.DATE); // 해당 월의 마지막 날짜

// JSP로 전달
request.setAttribute("year",    year);
request.setAttribute("month",   month);
request.setAttribute("today",   day);    // 오늘 일
request.setAttribute("week",    week);   // 1일의 요일 위치
request.setAttribute("lastday", lastday);
request.setAttribute("strWeek", new String[]{"일","월","화","수","목","금","토"});
```

```jsp
<%-- diary.jsp : JSTL로 달력 그리기 핵심 --%>

<%-- 1. 요일 헤더 : c:choose로 일=빨강, 토=파랑 --%>
<c:set var="k" value="0"/>
<c:forEach var="w" items="${strWeek}">
  <c:choose>
    <c:when test="${k==0}"><c:set var="color" value="red"/></c:when>
    <c:when test="${k==6}"><c:set var="color" value="blue"/></c:when>
    <c:otherwise><c:set var="color" value="black"/></c:otherwise>
  </c:choose>
  <th style="color:${color}">${w}</th>
  <c:set var="k" value="${k+1}"/>
</c:forEach>

<%-- 2. 날짜 출력 : 1일 앞에 빈칸(week개) 채우기 --%>
<c:set var="week" value="${week}"/>
<c:forEach var="i" begin="1" end="${lastday}">
  <c:if test="${i==1}">           <%-- 첫 날만 빈칸 처리 --%>
    <tr>
    <c:forEach var="j" begin="1" end="${week}">
      <td>&nbsp;</td>
    </c:forEach>
  </c:if>

  <%-- 과거=회색, 오늘이후=초록(클릭가능) --%>
  <c:choose>
    <c:when test="${today > i}">
      <td><h4 style="color:gray">${i}</h4></td>   <%-- 과거 : 클릭 불가 --%>
    </c:when>
    <c:otherwise>
      <td><h4 class="day-link" style="color:green">${i}</h4></td>  <%-- 예약 가능 --%>
    </c:otherwise>
  </c:choose>

  <%-- 토요일(6)마다 줄바꿈 --%>
  <c:set var="week" value="${week+1}"/>
  <c:if test="${week > 6}">
    <c:set var="week" value="0"/>
    </tr><tr>
  </c:if>
</c:forEach>

<%-- 연도/월 select : onChange → form submit → 달력 새로고침 --%>
<select name="year" id="year">
  <c:forEach var="i" begin="2026" end="2030">
    <option ${i==year ? "selected" : ""}>${i}</option>
  </c:forEach>
</select>
```

---

## 7. TimeConfig — 랜덤 시간 슬롯 생성

```java
public static String reserveTime() {
    String[] times = {"10:00","11:00","12:00","12:30","13:00","13:30",
                      "14:00","15:00","16:00","17:00","18:00",
                      "18:30","19:00","19:30","20:00","21:00","22:00"};

    // 3~9개 랜덤 개수 결정
    int[] com = new int[(int)(Math.random() * 7) + 3];

    // 중복 없는 랜덤 인덱스 선택
    for (int i = 0; i < com.length; i++) {
        boolean bCheck = true;
        int su = 0;
        while (bCheck) {
            su = (int)(Math.random() * times.length);
            bCheck = false;
            for (int j = 0; j < i; j++) {
                if (com[j] == su) { bCheck = true; break; }
            }
        }
        com[i] = su;
    }

    Arrays.sort(com);  // 시간 오름차순 정렬

    // 콤마 구분 문자열로 변환 → JSP에서 c:forTokens로 분리
    String time = "";
    for (int i = 0; i < com.length; i++) time += times[com[i]] + ",";
    return time.substring(0, time.lastIndexOf(","));  // 마지막 콤마 제거
    // 결과 예: "10:00,12:30,14:00,18:00,20:00"
}
```

```jsp
<%-- reserve_time.jsp : c:forTokens로 콤마 분리 출력 --%>
<c:forTokens items="${time}" delims="," var="t">
    <span class="btn btn-success times">${t}</span>
</c:forTokens>
```

---

## 8. 예약 시스템 확장 방향 (주석 요약)

```text
현재 구현 : 화면 UI + 단계별 Ajax 조립 (DB 연동 없음)

향후 추가할 것
  ① DB 연동 (reserve 테이블 INSERT)
     └─ Cookie / Session으로 로그인 사용자 정보 포함
  ② 관리자 승인 기능
     └─ 승인 버튼 클릭 → 사용자에게 알림
     └─ Stomp (WebSocket 기반 JS 라이브러리) 활용
        = 실시간 양방향 통신 (채팅, 알림에 사용)
        = 없으면 DB 상태값 polling으로 대체 가능

기술 스택 로드맵 (주석에서 언급)
  JSP (JSTL/EL) → Spring → Spring-Boot
  Spring-Boot 템플릿 엔진
    JSP    : JSTL / EL
    Thymeleaf : 자체 제어문 / EL / {{}} (Vue와 충돌 시 설정 필요)
  Oracle → MySQL → AWS 배포
```

---

## 9. JSTL 핵심 태그 총정리

| 태그 | 역할 | 예시 |
|------|------|------|
| `<c:if>` | 단일 조건 | `<c:if test="${i==0}">` |
| `<c:choose>` | 다중 조건 (if-else) | `<c:when> / <c:otherwise>` |
| `<c:forEach>` | 반복 (배열/리스트) | `begin="1" end="12"` |
| `<c:forTokens>` | 구분자 분리 반복 | `items="${str}" delims=","` |
| `<c:set>` | 변수 선언/변경 | `<c:set var="k" value="0"/>` |
| `<c:redirect>` | 화면 이동 | `sendRedirect`와 동일 |

---

## 10. 실습 파일 목록

| 파일 | 역할 | 핵심 내용 |
|------|------|---------|
| `MyPageModel.java` | 마이페이지 Model | mypage_jsp + main_jsp 2단계 include |
| `AdminPageModel.java` | 관리자 Model | admin_main.jsp 별도 레이아웃 |
| `ReserveModel.java` | 예약 Model | 달력/음식/시간/인원 각 Ajax 응답 |
| `TimeConfig.java` | 시간 슬롯 생성 | 중복없는 랜덤 → 정렬 → 콤마 문자열 |
| `DataBoardDAO.java` | 게시판 DAO | Day 25와 동일 (복습용) |
| `reserve_main.jsp` | 예약 메인 레이아웃 | 4개 영역 Ajax로 조립 |
| `reserve_food.jsp` | 음식 목록 (Ajax 조각) | data-* 속성 + 카테고리 버튼 |
| `diary.jsp` | 달력 (Ajax 조각) | Calendar + JSTL c:choose 달력 렌더링 |
| `reserve_time.jsp` | 시간 선택 (Ajax 조각) | c:forTokens 콤마 분리 |
| `reserve_inwon.jsp` | 인원 선택 (Ajax 조각) | c:forEach 2~5 + 단체 |
| `mypage_main.jsp` | 마이페이지 레이아웃 | 사이드바 + jsp:include |
| `mypage_home.jsp` | 회원정보 화면 | sessionScope.name EL 출력 |
| `jjim_list.jsp` | 찜 목록 | 틀만 구성 (DB 연동 예정) |
| `cart_list.jsp` | 장바구니 | 틀만 구성 (DB 연동 예정) |
| `buy_list.jsp` | 구매 목록 | 틀만 구성 (DB 연동 예정) |

---
