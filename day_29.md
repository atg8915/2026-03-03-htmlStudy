# 📘 Day 29 — Vue 컴포넌트 emit + 상품 상세 + 아임포트 결제 연동

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 컴포넌트 props (객체형) | `props:{ curPage:0, endPage:0 }` | 타입 대신 기본값으로 선언 |
| 커스텀 이벤트 선언 | `emits:['page-change']` | 자식이 발생시킬 이벤트 등록 |
| 이벤트 발생 | `$emit('page-change', 값)` | 자식 → 부모로 값 전달 ⭐ |
| 이벤트 수신 (부모) | `@page-change="메서드"` | 부모가 자식의 emit을 받음 |
| props 전달 (kebab-case) | `:start-page="startPage"` | camelCase props는 HTML에서 kebab-case로 |
| 여러 Vue 앱 mount | `.mount("#listApp")` | id 선택자로 영역 구분 |
| 뒤로가기 | `window.history.back()` | 브라우저 이전 페이지 |
| 아임포트 초기화 | `IMP.init("가맹점코드")` | 결제 라이브러리 초기 설정 |
| 결제 요청 | `IMP.request_pay({...}, callback)` | 결제창 호출 + 완료 콜백 |
| EL → Vue data 초기값 | `no: ${param.no}` | JSP가 Vue data 초기값을 직접 주입 ⭐ |

---

## 1. BOM(Browser Object Model) 구조 (주석 정리)

```text
window : 브라우저 자체 (메뉴/상태바/주소창) 담당
  ├─ document : HTML 화면 (DOM의 시작점)
  ├─ location : 주소 관리 → location.href = 화면 이동
  ├─ history  : 방문 기록 → history.back() / history.forward()
  └─ screen   : 화면 해상도 정보

window.open() / window.close() : 새 창 열기/닫기
history.back()                 : 이전 페이지로 이동 (go(-1)과 동일)
```

---

## 2. 컴포넌트 간 통신 — props + emit (핵심 패턴) ⭐

```text
지난 시간(Day 27) : $parent.변수로 부모 데이터 직접 접근
오늘(Day 29)      : emit으로 이벤트만 발생 → 부모가 직접 제어 (더 권장되는 방식)

부모 → 자식 : props로 값 전달 (내려보내기)
자식 → 부모 : emit으로 이벤트 발생 (올려보내기)
```

### 자식 컴포넌트 (pagecard.js)
```javascript
const pagecard = {
    // props : 객체 형태로 선언 — key가 기본값
    props: {
        curPage: 0,
        endPage: 0,
        startPage: 0,
        totalPage: 0
    },

    // emits : 이 컴포넌트가 발생시킬 이벤트 목록 미리 선언 (필수는 아니지만 명시 권장)
    emits: ['page-change'],

    methods: {
        range(start, end) {
            let arr = []
            for (let i = 0; i <= end - start; i++) arr[i] = start + i
            return arr
        }
    },

    template: `
        <ul class="pagination">
            <li v-if="startPage > 1">
                <a @click="$emit('page-change', startPage - 1)">&laquo;</a>
            </li>
            <li v-for="i in range(startPage, endPage)" :class="i === curPage ? 'active' : ''">
                <a @click="$emit('page-change', i)">{{i}}</a>
            </li>
            <li v-if="endPage < totalPage">
                <a @click="$emit('page-change', endPage + 1)">&raquo;</a>
            </li>
        </ul>
    `
}
// ⭐ $emit('이벤트명', 전달할값) → 부모의 @이벤트명="메서드" 가 그 값을 인자로 받음
```

### 부모 (list.jsp)
```html
<!-- props는 kebab-case로 전달, emit은 @이벤트명으로 받음 -->
<pagecard
    :start-page="startPage"
    :end-page="endPage"
    :cur-page="curpage"
    :total-page="totalpage"
    @page-change="pageChange"
></pagecard>
```

```javascript
methods: {
    // 자식이 emit('page-change', i) 호출 → page 매개변수로 i값 수신
    pageChange(page) {
        this.curpage = page
        this.dataRecv()
    }
}
```

| | $parent 방식 | props + emit 방식 ⭐ |
|--|--------------|---------------------|
| 부모→자식 | 자식이 `$parent.변수` 직접 참조 | `props`로 명시적 전달 |
| 자식→부모 | 자식이 `$parent.메서드()` 직접 호출 | `$emit`으로 이벤트만 발생 |
| 결합도 | 높음 (부모 구조에 의존) | 낮음 (독립적, 재사용 쉬움) |
| 권장 여부 | 간단한 경우만 | 실무 권장 패턴 |

---

## 3. 상품 목록 — list.jsp (페이징 컴포넌트 사용)

```javascript
let list = Vue.createApp({
    data() {
        return { list: [], curpage: 1, totalpage: 0, endPage: 0, startPage: 0 }
    },
    mounted() { this.dataRecv() },
    methods: {
        async dataRecv() {
            await axios.get('../goods/list_vue.do', { params: { page: this.curpage } })
                 .then(response => {
                     this.list      = response.data.list
                     this.curpage   = response.data.curpage
                     this.totalpage = response.data.totalpage
                     this.startPage = response.data.startPage
                     this.endPage   = response.data.endPage
                 })
        },
        pageChange(page) {       // 자식 pagecard로부터 emit 수신
            this.curpage = page
            this.dataRecv()
        }
    },
    components: { pagecard: pagecard }
}).mount("#listApp")   // ⭐ id 선택자로 mount (class와 달리 페이지당 1개만 가능)
```

```html
<div class="col-sm-3" v-for="(vo, index) in list" :key="index">
    <a :href="'../goods/detail.do?no=' + vo.no">
        <img :src="vo.goods_poster" style="width:250px;height:150px;object-fit:cover">
        <p>{{vo.goods_name}}</p>
    </a>
</div>
```

---

## 4. 상품 상세 + 아임포트 결제 (detail.jsp) ⭐

```javascript
var IMP = window.IMP
IMP.init("imp61461128")     // 가맹점 식별 코드로 초기화 (모듈 로드 시 1회)

let detail = Vue.createApp({
    data() {
        return {
            no: ${param.no},  // ⭐ JSP EL이 Vue data 초기값을 직접 채워줌
            vo: {}
        }
    },
    mounted() {
        axios.get('../goods/detail_vue.do', { params: { no: this.no } })
             .then(response => { this.vo = response.data })
    },
    methods: {
        go() {
            window.history.back()   // 뒤로가기 = 목록으로 복귀
        },
        buyBtn() {
            this.requestPay(this.vo.goods_name, this.vo.goods_price)
        },
        requestPay(name, price) {
            IMP.request_pay({
                pg: "html5_inicis",          // PG사
                pay_method: "card",
                merchant_uid: "ORD20180131-0000011",  // 주문번호 (실무: 서버에서 고유 생성)
                name: name,
                amount: price,               // 숫자 타입 필수
                buyer_email: '', buyer_name: '', buyer_tel: '',
                buyer_addr: '', buyer_postcode: ''
            }, function(rsp) {   // 결제 완료 콜백
                alert("구매가 완료되었습니다.\n마이페이지에서 확인하세요")
                // window.location.href = "../mypage/buy_list.do"  // 실무: 완료 후 이동
            })
        }
    }
}).mount("#detailApp")
```

```html
<button class="btn btn-primary">장바구니</button>
<button class="btn btn-danger" @click="buyBtn()">바로구매</button>
<button class="btn btn-success" @click="go">목록</button>
```

> ⭐ `no: ${param.no}` — JSP가 서버 단계에서 URL 파라미터를 Vue data 초기값으로 직접 박아넣는 패턴. JS와 JSP EL이 한 줄에서 섞여 동작.

---

## 5. 공통 레이아웃 — header.jsp + main.jsp

```jsp
<%-- main.jsp : 모든 페이지의 틀 --%>
<jsp:include page="../main/header.jsp"></jsp:include>   <%-- 상단 네비게이션 --%>

<div class="container text-right">
  ID:<input type="text" ref="idRef">
  PW:<input type="password" ref="pwdRef">
  <button class="btn-sm btn-danger">로그인</button>
</div>
<hr>
<jsp:include page="${main_jsp}"></jsp:include>   <%-- Model이 지정한 동적 콘텐츠 --%>
```

```jsp
<%-- header.jsp : 공통 네비게이션 메뉴 --%>
<nav class="navbar navbar-inverse">
  <a class="navbar-brand" href="../main/main.do">MiniProject</a>
  <ul class="nav navbar-nav">
    <li><a href="../main/main.do">Home</a></li>
    <li><a href="../member/join.do">회원가입</a></li>
    <li><a href="../chat/chat.do">채팅</a></li>
    <li><a href="../news/news.do">실시간 뉴스</a></li>
  </ul>
</nav>
```

> Day 26에서 배운 `<jsp:include page="${변수}">` 패턴이 그대로 재사용됨 — header는 고정, 본문만 교체

---

## 6. CSS 카드 디자인 패턴 (product.css)

```css
/* hover 시 그림자 효과 — 클릭 유도 UI */
.product-card { transition: .3s; }
.product-card:hover { box-shadow: 0 5px 15px rgba(0,0,0,.2); }

/* 이미지 비율 고정 — object-fit으로 잘림 처리 */
.product-card img {
    width: 100%; height: 250px;
    object-fit: cover;   /* 비율 유지하며 영역 꽉 채움, 넘치는 부분 잘림 */
}

/* 텍스트 줄임 처리 — 카드 높이 고정 시 필수 ⭐ */
.product-name { height: 40px; overflow: hidden; font-weight: bold; }
.product-desc { height: 60px; color: #666; overflow: hidden; }
```

---

## 7. 다시 만들 때 체크리스트 — 컴포넌트 emit 패턴

```text
① 자식 컴포넌트 설계
   - props : 부모에게 받을 값 (객체 형태로 기본값 선언)
   - emits : 부모에게 보낼 이벤트 이름 미리 선언
   - template 안에서 @click="$emit('이벤트명', 전달값)"

② 부모에서 사용
   - :prop명="data변수"  (kebab-case 변환 주의 ⭐ curPage → cur-page)
   - @이벤트명="메서드명"
   - methods에 해당 메서드 정의 → 매개변수로 emit 값 수신

③ 결제 연동 시
   - IMP.init()는 모듈 로드 직후 1회만
   - request_pay()의 merchant_uid는 실무에서 서버가 생성해서 내려줘야 함 (중복 방지)
   - 콜백 함수 안에서 결제 후처리 (페이지 이동, DB 저장 요청 등)

④ EL + Vue 혼용 시
   - data() 초기값에 ${param.xxx} 형태로 JSP 변수를 바로 주입 가능
   - 단, 이 JSP가 서버에서 렌더링된 후 브라우저에 전달되는 시점 이해 필수
```

---

## 8. 실습 파일 목록

| 파일 | 역할 | 핵심 내용 |
|------|------|---------|
| `list.jsp` | 상품 목록 | `#listApp` mount, pagecard 컴포넌트 사용 |
| `detail.jsp` | 상품 상세 + 결제 | `${param.no}` 초기값 주입, IMP 결제, `history.back()` |
| `pagecard.js` | 페이징 컴포넌트 | `props`(객체형), `emits`, `$emit('page-change', i)` |
| `header.jsp` | 공통 네비게이션 | 고정 메뉴, `main.jsp`에 include됨 |
| `main.jsp` | 공통 레이아웃 | header include + `${main_jsp}` 동적 include |
| `product.css` | 카드 UI 스타일 | hover 그림자, `object-fit:cover`, 텍스트 줄임 |

---
