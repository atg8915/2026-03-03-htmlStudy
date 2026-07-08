# 📘 Day 30 — Vue 심화 디렉티브 + computed/watch + WebSocket 채팅 + MyBatis 동적 SQL

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 1회 렌더링 | `v-once` | 데이터 변경 후에도 화면 갱신 안 함 |
| 깜박임 방지 | `v-cloak` + CSS `[v-cloak]{display:none}` | Vue 로딩 전 `{{}}` 노출 방지 |
| 캐시 렌더링 | `v-memo="[변수]"` | 해당 변수가 바뀔 때만 재렌더링 |
| 계산된 값 | `computed:{ 함수명(){ return 값 } }` | data 기반 자동 계산, 캐싱 ⭐ |
| 데이터 감시 | `watch:{ 변수명(newVal, oldVal){} }` | 특정 변수 변경 시 자동 실행 |
| Vue + jQuery | `mounted()`안에서 `$().on()` | 혼용 가능하나 this 주의 |
| WebSocket 연결 | `new WebSocket("ws://서버주소")` | 실시간 양방향 통신 |
| 메시지 수신 | `websocket.onmessage = onMessage` | 서버 → 브라우저 자동 호출 |
| 메시지 전송 | `websocket.send(msg)` | 브라우저 → 서버 전송 |
| 서버 어노테이션 | `@ServerEndpoint("/chat")` | WebSocket 서버 엔드포인트 등록 |
| 접속자 관리 | `Collections.synchronizedMap(new HashMap<>())` | 동기화 Map으로 안전하게 저장 |
| 입장 알림 | `@OnOpen` + 전체 순회 sendText | 입장 시 다른 접속자에게 알림 |
| MyBatis `#{}` | `WHERE id = #{id}` | PreparedStatement, SQL Injection 방지 ⭐ |
| MyBatis `${}` | `ORDER BY ${column}` | 문자열 직접 치환, 컬럼명 동적 처리 시만 |
| 동적 WHERE | `<where>` + `<if>` | WHERE 자동 생성 + 첫 AND 제거 |
| 동적 UPDATE | `<set>` + `<if>` | SET 자동 생성 + 마지막 콤마 제거 |
| IN절 처리 | `<foreach>` | 리스트 → IN(1,2,3) 자동 생성 |
| SQL 재사용 | `<sql id>` + `<include refid>` | 공통 컬럼 분리 재사용 |

---

## 1. Vue 디렉티브 심화 (vue_1~4.jsp)

### v-once — 1회만 렌더링 (vue_2.jsp)
```html
<!-- 데이터 변경 후에도 이 태그는 갱신되지 않음 -->
<h4 v-once>{{message}}</h4>   <!-- 고정 출력 : 회사명, 로고, 개인정보방침 등 -->
<h4>{{message}}</h4>           <!-- 일반 출력 : 변경 시 갱신됨 -->
<button @click="change()">변경</button>
```
> 사용 예 : 초기 로딩 후 바뀌지 않는 정적 데이터 (게시물 번호, 회사명 등)

### v-cloak — 깜박임 방지 (vue_3.jsp)
```css
/* CSS에 반드시 추가 — Vue가 로딩되기 전 {{ }} 노출 방지 */
[v-cloak] { display: none }
```
```html
<div id="app" v-cloak>    <!-- Vue 준비 완료 전까지 숨김 -->
    <h2>{{title}}</h2>
</div>
```

### v-memo — 캐시 렌더링 최적화 (vue_4.jsp)
```html
<!-- name이 바뀔 때만 이 영역 재렌더링, count는 무시 -->
<div v-memo="[name]">
    <h3>{{name}}</h3>
    <h3>{{count}}</h3>   <!-- name 안 바뀌면 count 갱신 안 됨 -->
</div>

<!-- 아래는 항상 갱신 -->
<h3>{{name}}</h3>
<h3>{{count}}</h3>
```
```html
<!-- 댓글 추가 시 기존 댓글 재렌더링 방지 패턴 (vue_5.jsp) -->
<div v-for="reply in replyList" :key="reply.no" v-memo="[]">
    <!-- v-memo=[] : 의존 변수 없음 → 최초 1회만 렌더링 -->
</div>
```

---

## 2. computed + watch (goods.jsp) ⭐

```javascript
let goods = Vue.createApp({
    data() {
        return {
            price: 10000,
            account: 1
        }
    },
    // computed : data 기반 자동 계산값, 캐싱 지원
    // methods와 차이 : 의존 데이터 변경 시만 재계산 (성능 유리)
    computed: {
        totalPrice() {
            return (this.price * this.account).toLocaleString()
            // account 변경 시 자동으로 재계산 → 화면 갱신
        }
    },
    // watch : 특정 변수 변경을 감시 → 외부 API 호출, 로그 처리 등
    watch: {
        account(newVal, oldVal) {
            console.log("수량 변경: " + oldVal + " → " + newVal)
            // 예: 수량 변경 시 서버에 재고 확인 요청 가능
        }
    }
}).mount(".container")
```

```html
<button @click="account--" :disabled="account <= 1">-</button>
<!-- :disabled : account가 1이하면 버튼 비활성화 ⭐ -->
<span>{{account}}</span>
<button @click="account++">+</button>
<p>총금액: {{totalPrice}}</p>   <!-- computed 자동 갱신 -->
```

| | `computed` | `methods` | `watch` |
|--|-----------|-----------|---------|
| 호출 시점 | 의존 data 변경 시 자동 | 명시적 호출 | 특정 변수 변경 감지 |
| 캐싱 | O (같은 값이면 재계산 안 함) | X | X |
| 주요 용도 | 계산된 출력값 (총금액, 필터링) | 이벤트 처리 | 변경 감지 후 API 호출 |

---

## 3. Vue + jQuery 혼용 패턴 (vue_1.jsp)

```javascript
// mounted() 안에서 jQuery 이벤트 등록 가능
// ⚠️ jQuery 콜백 안에서 this = jQuery 객체 (Vue this 아님)
// → Vue data 변수 접근 시 this.no 대신 app.no 또는 외부 변수 활용

let reply = Vue.createApp({
    data() {
        return { msg: '홍길동', no: 0 }
    },
    mounted() {
        // jQuery로 show/hide, Vue로 v-model 양방향 바인딩 동시 활용
        $('.btn-danger').on('click', function() {
            if (this.no === 0) {  // ⚠️ 여기서 this는 jQuery → 오류 가능
                $('#print').show()
                $('.btn-danger').text("취소")
            }
        })
    }
}).mount(".container")
```

> ⭐ 실무 권장 : jQuery 혼용 최소화, Vue 디렉티브(`v-show`, `@click`)로 통일  
> 혼용이 필요하면 `const self = this`로 Vue this를 미리 저장

---

## 4. WebSocket 채팅 — 전체 구조 (chat.jsp + ChatManager.java)

### 개념 비교

```text
HTTP (기존 Ajax)                 WebSocket
────────────────────────────    ──────────────────────────
요청 → 응답 → 연결 끊김          연결 유지 (상시 통신)
클라이언트가 먼저 요청           서버도 클라이언트에 먼저 전송 가능
채팅에 부적합 (polling 필요)     채팅 / 알림 / 실시간 데이터에 적합

관련 라이브러리
  WebSocket  : 브라우저 기본 내장 API (순수)
  SockJS     : WebSocket 미지원 브라우저 폴백 처리
  Stomp      : WebSocket 위의 메시징 프로토콜 (1:1 / 그룹 / 알림)
  Spring 이후: Spring WebSocket + Stomp 조합 주로 사용
```

### 브라우저 (chat.jsp)
```javascript
let websocket

// 1. 페이지 로드 시 WebSocket 연결
window.onload = function() { connection() }

function connection() {
    websocket = new WebSocket("ws://localhost/프로젝트명/chat")
    //                         ↑ ws:// = WebSocket 프로토콜 (http:// 아님)
    websocket.onopen    = onOpen     // 연결 성공 시 자동 호출
    websocket.onclose   = onClose    // 연결 해제 시 자동 호출
    websocket.onmessage = onMessage  // 서버에서 메시지 수신 시 자동 호출
}

function onOpen(event)    { alert("채팅서버에 연결되었습니다") }
function onClose(event)   { alert("채팅서버와 연결 해제되었습니다") }

// 2. 메시지 수신 : "msg:내용" 형식 파싱
function onMessage(event) {
    let data = event.data
    if (data.substring(0, 4) === "msg:") {
        appendMessage(data.substring(4))  // "msg:" 이후 실제 메시지만 추출
    }
}

function appendMessage(msg) {
    $('#chatBox').append(msg + "<br>")
    $('#chatBox').scrollTop($('#chatBox')[0].scrollHeight)  // 스크롤 맨 아래
}

// 3. 메시지 전송
function send() {
    let msg = $('#messageInput').val()
    if (msg.trim() === "") { $('#messageInput').focus(); return }
    websocket.send(msg)           // 서버로 전송
    $('#messageInput').val("").focus()
}

$(function() {
    $('#sendBtn').on('click', function() { send() })
    $('#messageInput').on('keydown', function(key) {
        if (key.keyCode === 13) send()   // Enter 전송
    })
})
```

### 서버 (ChatManager.java)
```java
@ServerEndpoint(value="/chat", configurator=WebSocketSessionConfigurator.class)
// /chat 경로로 들어오는 WebSocket 연결을 이 클래스가 처리
// configurator : HttpSession을 WebSocket에서도 사용하기 위한 설정
public class ChatManager {

    // 접속자 저장 : Session → ChatVO (id, name, session)
    // synchronizedMap : 여러 사용자 동시 접속 시 안전한 동기화 처리 ⭐
    private static Map<Session, ChatVO> users =
        Collections.synchronizedMap(new HashMap<>());

    // 연결 시 자동 호출
    @OnOpen
    public void onOpen(Session session, EndpointConfig config) throws Exception {
        // HttpSession에서 로그인 정보 읽기
        HttpSession hs = (HttpSession) config.getUserProperties()
                         .get(HttpSession.class.getName());
        ChatVO vo = new ChatVO();
        vo.setId((String) hs.getAttribute("id"));
        vo.setName((String) hs.getAttribute("name"));
        vo.setSession(session);
        users.put(session, vo);   // 접속자 명단에 추가

        // 본인 제외 전체에게 입장 알림
        for (Session ss : users.keySet()) {
            if (!ss.getId().equals(session.getId())) {
                ss.getBasicRemote().sendText("msg:[⏰알림]" + vo.getName() + "님 입장하셨습니다");
            }
        }
    }

    // 메시지 수신 시 자동 호출 → 전체 발송
    @OnMessage
    public void onMessage(String message, Session session) throws Exception {
        ChatVO vo = users.get(session);   // 보낸 사람 정보
        for (Session ss : users.keySet()) {
            ss.getBasicRemote().sendText("msg:[" + vo.getName() + "]" + message);
            // 전체 접속자에게 "msg:[이름]내용" 형식으로 전송
        }
    }

    // 연결 해제 시 자동 호출
    @OnClose
    public void onClose(Session session) throws Exception {
        ChatVO vo = users.get(session);
        for (Session ss : users.keySet()) {
            if (!ss.getId().equals(session.getId())) {
                ss.getBasicRemote().sendText("msg:[⏰알림]" + vo.getName() + "님 퇴장하셨습니다");
            }
        }
        users.remove(session);   // 접속자 명단에서 제거
    }

    @OnError
    public void onError(Session session, Throwable ex) { ex.printStackTrace(); }
}
```

### 채팅 전체 흐름
```text
브라우저 로드
  └─ connection() → new WebSocket("ws://서버/chat")
       └─ @OnOpen : 접속자 Map 저장 + 입장 알림 전송

사용자 메시지 입력 (버튼/Enter)
  └─ send() → websocket.send(msg) → 서버 전달
       └─ @OnMessage : 전체 접속자에게 "msg:[이름]내용" 브로드캐스트
            └─ onMessage() → appendMessage() → chatBox에 출력

창 닫기 / 페이지 이동
  └─ @OnClose : 퇴장 알림 전송 + Map에서 제거
```

---

## 5. MyBatis 동적 SQL 총정리 (프로젝트 정리 문서)

### `#{}` vs `${}` 차이
```xml
WHERE id = #{id}       <!-- PreparedStatement → SQL Injection 방지 ⭐ -->
ORDER BY ${column}     <!-- 문자열 치환 → 컬럼명 동적 처리 시만 사용 (위험) -->
```

### 동적 SQL 핵심 태그
```xml
<!-- 1. if + where : 조건 검색 (가장 많이 사용) -->
<select id="search" resultType="BoardVO">
    SELECT * FROM board
    <where>                              <!-- WHERE 자동 생성 + 첫 AND/OR 제거 -->
        <if test="title != null">
            title LIKE '%'||#{title}||'%'
        </if>
        <if test="writer != null">
            AND writer = #{writer}
        </if>
    </where>
</select>

<!-- 2. choose/when/otherwise : 다중 조건 선택 (if-else if-else) -->
<choose>
    <when test="type == 'title'">  title LIKE '%'||#{keyword}||'%' </when>
    <when test="type == 'writer'"> AND writer = #{keyword} </when>
    <otherwise>                    AND category = 'DEFAULT' </otherwise>
</choose>

<!-- 3. set : UPDATE 동적 컬럼 (마지막 콤마 자동 제거) -->
<update id="updateBoard">
    UPDATE board
    <set>
        <if test="title != null">   title = #{title},   </if>
        <if test="content != null"> content = #{content} </if>
    </set>
    WHERE id = #{id}
</update>

<!-- 4. foreach : IN절 / 체크박스 다중 선택 -->
<select id="findByIds" resultType="BoardVO">
    SELECT * FROM board WHERE id IN
    <foreach collection="list" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
    <!-- List<Integer> → IN(1,2,3) 자동 생성 -->
</select>

<!-- 5. sql + include : 공통 컬럼 재사용 -->
<sql id="baseColumns"> id, title, content, regdate </sql>

<select id="getList" resultType="BoardVO">
    SELECT <include refid="baseColumns"/> FROM board
</select>
```

| 태그 | 역할 | Java 대응 |
|------|------|----------|
| `<if>` | 조건부 SQL | `if` |
| `<where>` | WHERE 자동 + 첫 AND 제거 | `WHERE 1=1` 패턴 대체 |
| `<choose>/<when>/<otherwise>` | 다중 선택 | `if-else if-else` |
| `<set>` | UPDATE 콤마 자동 처리 | — |
| `<foreach>` | IN절 / 반복 | `for` |
| `<sql>/<include>` | SQL 조각 재사용 | 메서드 분리 |
| `<trim>` | 접두사/접미사 커스텀 제거 | `<where>` 확장판 |

### MyBatis 실행 흐름
```text
Mapper 호출 (개발자)
  └─ SqlSession
       └─ Executor (SQL 실행 전략)
            └─ StatementHandler (#{} 바인딩, PreparedStatement 생성)
                 └─ JDBC → DB 실행
                      └─ ResultSetHandler (ResultSet → Java 객체 변환)
                           └─ resultType / resultMap 적용
```

---

## 6. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `vue_1.jsp` | Vue + jQuery 혼용 | `mounted()`에서 jQuery 이벤트, v-model |
| `vue_2.jsp` | v-once | 1회 렌더링, 정적 데이터 고정 |
| `vue_3.jsp` | v-cloak | `[v-cloak]{display:none}` CSS 필수 |
| `vue_4.jsp` | v-memo | 의존 변수 변경 시만 재렌더링 |
| `vue_5.jsp` | v-memo 활용 | 댓글 목록 불필요한 재렌더링 방지 |
| `goods.jsp` | computed + watch | 총금액 자동 계산, 수량 변경 감시 |
| `chat.jsp` | WebSocket 브라우저 | connection/send/onMessage 흐름 |
| `ChatManager.java` | WebSocket 서버 | @OnOpen/Message/Close, synchronizedMap |
| `chat.css` | 채팅 UI | flex 레이아웃, 말풍선 좌우 정렬 |
| `프로젝트-정리.docx` | MyBatis 동적 SQL | if/where/choose/foreach/set/sql 총정리 |

---

