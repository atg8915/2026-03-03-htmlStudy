# 📘 Day 22 — Ajax + jQuery Ajax + Vue.js 맛보기 + 카카오맵 API

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 바닐라 Ajax | `new XMLHttpRequest()` | 브라우저 내장, 코드 복잡 |
| jQuery Ajax | `$.ajax({ url, type, data, success })` | 간결, 실무 레거시 다수 |
| jQuery 4 권장 | `$.ajax().done().fail()` | Promise 스타일 |
| fetch (최신) | `fetch(url).then(res => res.json())` | 바닐라 JS 표준 |
| async/await | `const data = await fetch(url)` | 현재 신규 개발 표준 ⭐ |
| JSON 파싱 | `JSON.parse(문자열)` | 문자열 → JS 객체/배열 |
| JSON 직렬화 | `JSON.stringify(객체)` | JS 객체 → 문자열 |
| Vue 진입점 | `mounted()` | `window.onload` / `$(function(){})` 와 동일 |
| Vue 데이터 | `data() { return { } }` | 멤버변수 선언 |
| Vue 메서드 | `methods: { }` | 함수 선언 |
| Vue 반복 | `v-for="vo in list"` | JSTL `<c:forEach>` 와 동일 |
| Vue 속성 바인딩 | `:src="vo.poster"` | 동적 속성 (단방향) |
| Vue 텍스트 출력 | `{{ vo.name }}` | EL `${}` 와 동일 |
| 카카오맵 | `new kakao.maps.Map(el, option)` | 지도 생성 |
| 장소 검색 | `ps.keywordSearch(keyword, callback)` | 키워드로 장소 검색 |

---

## 1. Ajax 개념

```text
기존 방식 : 버튼 클릭 → 페이지 전체 새로고침 → 서버 요청 → 화면 전체 다시 로딩
Ajax 방식 : 버튼 클릭 → 백그라운드 서버 요청 → 데이터만 수신 → 화면 일부만 변경

핵심 = "HTML 전체가 아닌 JSON/text 데이터만 주고받는다"

진화 흐름
XMLHttpRequest (바닐라, 복잡)
  → $.ajax() (jQuery, 간결)
    → fetch (바닐라 표준)
      → axios (Vue/React 주력)
        → async/await (최신 표준) ⭐
```

---

## 2. 바닐라 Ajax — XMLHttpRequest (ajax.jsp)

```javascript
// 브라우저 내장 객체 직접 사용 → jQuery/axios 이전 방식
// 원리 이해용으로만 학습

function sendRequest(url, params, callback, method) {
    let httpRequest = new XMLHttpRequest()   // 객체 생성
    httpRequest.open(method, url, true)       // true = 비동기
    httpRequest.setRequestHeader("Content-Type",
        "application/x-www-form-urlencoded") // 한글 처리
    httpRequest.onreadystatechange = callback // 콜백 등록
    httpRequest.send(method === 'POST' ? params : null)
}

function ok() {
    /*
      readyState : 0(준비) → 1(open) → 2(연결완료) → 3(전송준비) → 4(전송완료)
      status     : 200(성공) / 404(없음) / 500(서버오류)
    */
    if (httpRequest.readyState === 4 && httpRequest.status === 200) {
        document.querySelector("#print").innerHTML = httpRequest.responseText
    }
}
```

---

## 3. jQuery Ajax — $.ajax() (jquery_3.jsp + ajax.js) ⭐

```javascript
// 기본 구조
$.ajax({
    type: 'POST',          // GET / POST
    url: 'list_ajax.do',  // 요청 URL
    data: { page: 1 },    // 전송 데이터 (파라미터)
    success: function(json) {   // 콜백 : 서버 응답 시 자동 호출
        json = JSON.parse(json) // 문자열 → JS 배열
        foodPrint(json)
    }
})

// jQuery 4 권장 방식 (Promise 스타일)
$.ajax({ url: '/user.json', method: 'GET' })
  .done(function(data) { $('#result').html(data.name) })
  .fail(function() { console.log("에러") })
```

| 항목 | jQuery 3 | jQuery 4 (권장) |
|------|---------|----------------|
| 요청 방식 | `type: 'GET'` | `method: 'GET'` |
| 성공 처리 | `success: function(data){}` | `.done(function(data){})` |
| 실패 처리 | `error: function(){}` | `.fail(function(){})` |

---

## 4. Ajax + 페이징 전체 흐름 (ajax.js + list_ajax.jsp) ⭐

```text
list_ajax.jsp (HTML 껍데기 — #print, #pagination 만 존재)
  └─ <script src="ajax.js">

ajax.js
  └─ window.onload → dataRecv(1) 자동 호출
       └─ $.ajax POST → list_ajax.do?page=1
            └─ 서버: JSON 문자열 반환 (페이징 정보 포함)
                 └─ JSON.parse(json) → foodPrint(json)
                      └─ #print : 카드 목록 innerHTML
                      └─ #pagination : 페이지 버튼 innerHTML
```

```javascript
function dataRecv(page) {
    $.ajax({
        type: 'post',
        url: 'list_ajax.do',
        data: { page: page },
        success: function(json) {
            json = JSON.parse(json)   // ⭐ 서버가 문자열로 보내면 파싱 필수
            foodPrint(json)
        }
    })
}

function foodPrint(json) {
    // 목록 출력
    let html = ''
    json.forEach((food) => {
        html += '<div class="col-sm-3"><img src="' + food.poster + '"><p>' + food.name + '</p></div>'
    })
    $('#print').html(html)

    // 페이징 버튼 출력 — 페이징 데이터는 json[0]에 포함
    let curpage   = json[0].curpage
    let totalpage = json[0].totalpage
    let startPage = json[0].startPage
    let endPage   = json[0].endPage

    let pagePrint = '<ul class="pagination">'
    if (startPage > 1)
        pagePrint += '<li><a onclick="dataRecv(' + (startPage - 1) + ')">&laquo;</a></li>'
    for (let i = startPage; i <= endPage; i++)
        pagePrint += '<li><a onclick="dataRecv(' + i + ')">' + i + '</a></li>'
    if (endPage < totalpage)
        pagePrint += '<li><a onclick="dataRecv(' + (endPage + 1) + ')">&raquo;</a></li>'
    pagePrint += '</ul>'
    $('#pagination').html(pagePrint)
}
```

> ⭐ 페이징 정보(`curpage`, `totalpage`, `startPage`, `endPage`)를 JSON 데이터 각 row에 함께 포함시켜 `json[0]`으로 꺼내는 패턴

---

## 5. JSP 3가지 방식 비교

| 방식 | 파일 | 특징 |
|------|------|------|
| 일반 JSP (MVC) | `list.jsp` | 서버에서 JSTL로 렌더링, 페이지 이동마다 새로고침 |
| Ajax + jQuery | `list_ajax.jsp` + `ajax.js` | HTML 껍데기 + JS가 동적으로 채움, 새로고침 없음 |
| Vue.js + Axios | `list_vue.jsp` | 양방향 바인딩, `v-for` 자동 렌더링, 가장 현대적 |

---

## 6. Vue.js 기초 (list_vue.jsp)

```javascript
// Vue 3 CDN
// <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
// <script src="https://unpkg.com/axios/dist/axios.min.js"></script>

Vue.createApp({
    data() {
        return {
            // 멤버변수 선언 (React의 useState와 같은 역할)
            list: [],
            curpage: 1,
            totalpage: 0,
            startPage: 0,
            endPage: 0
        }
    },
    mounted() {          // window.onload / $(function(){}) 와 동일
        this.dataRecv()
    },
    methods: {
        dataRecv() {
            axios.get('list_ajax.do', { params: { page: this.curpage } })
                 .then(response => {
                     this.list      = response.data          // data() 변수에 저장 → 자동 렌더링
                     this.curpage   = response.data[0].curpage
                     this.totalpage = response.data[0].totalpage
                     this.startPage = response.data[0].startPage
                     this.endPage   = response.data[0].endPage
                 })
        }
    }
}).mount('.container')   // 적용할 HTML 범위 지정
```

```html
<!-- HTML에서 Vue 디렉티브 사용 -->
<a href="#" v-for="vo in list">          <!-- c:forEach 와 동일 -->
    <img :src="vo.poster">               <!-- 동적 속성 바인딩 -->
    <p>{{ vo.name }}</p>                 <!-- 텍스트 출력 -->
</a>
```

> `data()`의 값이 바뀌면 HTML이 자동으로 다시 렌더링됨 → **반응형(Reactive)**

---

## 7. 카카오맵 API (detail.jsp) ⭐

```javascript
// CDN
// <script src="//dapi.kakao.com/v2/maps/sdk.js?appkey=발급키&libraries=services"></script>

// 지도 생성
var map = new kakao.maps.Map(document.getElementById('map'), {
    center: new kakao.maps.LatLng(37.566826, 126.9786567), // 중심 좌표
    level: 3   // 확대 레벨 (숫자 작을수록 확대)
})

// 장소 검색 서비스
var ps = new kakao.maps.services.Places()
ps.keywordSearch("강남 맛집", function(data, status) {
    if (status === kakao.maps.services.Status.OK) {
        // data = 검색 결과 배열
        // data[i].place_name / .road_address_name / .x / .y / .phone
    }
})

// 마커 생성
var marker = new kakao.maps.Marker({ position: new kakao.maps.LatLng(y, x) })
marker.setMap(map)     // 지도에 표시
marker.setMap(null)    // 지도에서 제거

// 인포윈도우 (말풍선)
var infowindow = new kakao.maps.InfoWindow({ zIndex: 1 })
infowindow.setContent('<div>' + title + '</div>')
infowindow.open(map, marker)
infowindow.close()

// 마커 이벤트
kakao.maps.event.addListener(marker, 'mouseover', function() { ... })
kakao.maps.event.addListener(marker, 'mouseout',  function() { ... })
```

> `detail.jsp`에서 `${addr}` (맛집 주소)를 키워드 input 기본값으로 넣어 자동 검색

---

## 8. MyBatis 동적 SQL 핵심 정리

```xml
<!-- if : 조건부 SQL -->
<select id="searchUser" resultType="User">
    SELECT * FROM user WHERE 1=1
    <if test="name != null">AND name = #{name}</if>
</select>

<!-- where : AND 자동 처리 (WHERE 자동 생성/제거) -->
<select id="searchUser" resultType="User">
    SELECT * FROM user
    <where>
        <if test="name != null">name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </where>
</select>

<!-- set : UPDATE용, 마지막 콤마 자동 제거 -->
<update id="updateUser">
    UPDATE user
    <set>
        <if test="name != null">name = #{name},</if>
        <if test="age != null">age = #{age},</if>
    </set>
    WHERE id = #{id}
</update>

<!-- foreach : IN절 처리 -->
<select id="findByIds" resultType="User">
    SELECT * FROM user WHERE id IN
    <foreach item="id" collection="list" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

| 태그 | 역할 |
|------|------|
| `<if>` | 조건부 SQL 추가 |
| `<where>` | WHERE 자동 생성 + 첫 AND/OR 제거 |
| `<set>` | UPDATE SET 자동 생성 + 마지막 콤마 제거 |
| `<choose>/<when>/<otherwise>` | if-else if-else |
| `<foreach>` | IN절, 반복 SQL |

---

## 9. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `ajax.jsp` | 바닐라 Ajax | `XMLHttpRequest`, `readyState`, `status` |
| `jquery_3.jsp` | jQuery Ajax 기초 | `$.ajax()` + `success` 콜백 |
| `list_ajax.jsp` | Ajax 목록 | HTML 껍데기 + `ajax.js` 분리 |
| `ajax.js` | Ajax + 페이징 | `$.ajax` POST, `JSON.parse`, 페이징 버튼 동적 생성 |
| `list.jsp` | 일반 MVC | JSTL `c:forEach` 페이징 |
| `list_vue.jsp` | Vue.js | `Vue.createApp`, `data()`, `mounted()`, `v-for` |
| `detail.jsp` | 상세 + 카카오맵 | `kakao.maps.Map`, `keywordSearch`, 마커, 인포윈도우 |

