# 📘 Day 27 — Vue.js 심화 (생명주기/디렉티브/컴포넌트) + Axios + 카카오맵 연동

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| 텍스트 출력 | `{{변수}}` / `v-text="변수"` | 머스태시 / 디렉티브 |
| HTML 출력 | `v-html="변수"` | innerHTML과 동일 |
| 속성 바인딩 | `:src="변수"` / `v-bind:src` | 동적 속성 연결 |
| 양방향 바인딩 | `v-model="변수"` | input ↔ data 자동 동기화 ⭐ |
| 조건문 | `v-if` / `v-else-if` / `v-else` | 조건에 따라 DOM 추가/제거 |
| 토글 | `v-show="변수"` | display:none 토글 (DOM 유지) |
| 반복문 | `v-for="vo in list"` | 배열 반복 렌더링 |
| 이벤트 | `@click="함수"` / `v-on:click` | 이벤트 리스너 |
| 키 이벤트 | `@keydown.enter="함수"` | Enter 키 감지 |
| DOM 참조 | `ref="이름"` → `this.$refs.이름` | Vue 내부에서 태그 직접 접근 |
| 부모 접근 | `$parent.변수` / `$parent.메서드` | 컴포넌트에서 부모 접근 |
| 부모→자식 전달 | `v-bind:prop명="값"` + `props:['prop명']` | 컴포넌트 props ⭐ |
| 진입점 | `mounted()` | window.onload / $(function(){}) 와 동일 |
| 데이터 갱신 감지 | `updated()` | 데이터 변경 후 DOM 갱신 완료 시 |
| axios GET | `axios.get(url, {params:{}})`  | 쿼리스트링 전달 |
| axios POST | `axios.post(url, {}, {params:{}})` | body + 쿼리스트링 동시 전달 |
| 카카오맵 동적 로드 | `document.createElement('script')` | 데이터 수신 후 지도 로드 |
| 주소→좌표 | `geocoder.addressSearch(주소, callback)` | Geocoder API |

---

## 1. Vue.js 개념 정리

```text
목적 : 상태 관리 (data() 변수 변경 → HTML 자동 갱신)

MVVM 구조
  Model     : data() 안의 변수 (VO 역할)
  View      : HTML 템플릿
  ViewModel : Vue 인스턴스 (변수 변경 → HTML 자동 반영)

가상 DOM (Virtual DOM)
  Vue → 가상 메모리에 DOM 트리 저장
       diff 알고리즘으로 실제 DOM과 비교
       변경된 부분만 실제 브라우저에 적용 (성능 최적화)

JSP/jQuery 방식 : HTML 문자열을 직접 만들어서 innerHTML로 삽입
Vue 방식        : 실제 HTML 태그를 직접 제어 (디렉티브)

Vue 생태계
  Vue 3          : UI 프레임워크
  Pinia          : 최신 상태 관리 (Vuex의 후속)
  Vue Router     : 화면 이동 처리
  Nuxt.js        : Vue 기반 풀스택 프레임워크
  TypeScript     : 데이터형 불일치 방지
```

---

## 2. Vue 기본 구조 (vue_1.jsp ~ vue_3.jsp)

```javascript
// 기본 형식
let app = Vue.createApp({
    data() {
        return {
            msg: '',         // 문자열
            list: [],        // 배열 (Java ArrayList)
            vo: {},          // 객체 (Java VO)
            isShow: false,   // boolean
            count: 0         // 숫자
        }
    },
    mounted() {
        // window.onload와 동일 — 화면 로드 후 서버 데이터 요청
        this.msg = "Hello Vue3"
    },
    methods: {
        // 사용자 정의 함수 — 이벤트 처리
        // this.변수명 으로 data() 변수 접근
    },
    components: {
        // 외부 컴포넌트 등록
    },
    computed: {
        // 계산된 값 (천단위 콤마 등)
    },
    watch: {
        // 특정 변수 변경 감지
    }
}).mount('.container')  // CSS 선택자로 적용 범위 지정 (부분 처리 가능)

// ⭐ 여러 영역에 각각 다른 Vue 인스턴스 적용 가능
Vue.createApp({ data(){ return { msg:'Hello a영역' } } }).mount("#a")
Vue.createApp({ data(){ return { msg:'Hello b영역' } } }).mount("#b")
```

---

## 3. 생명주기 함수 (vue_2.jsp)

```javascript
Vue.createApp({
    beforeCreate()  { console.log("Vue 객체 생성 전") },
    created()       { console.log("Vue 객체 생성 완료") },
    beforeMount()   { console.log("가상돔에 올라가기 전") },
    mounted()       { console.log("가상돔 저장 완료 ← 여기서 서버 연결 ⭐") },
    beforeUpdate()  { console.log("데이터 갱신 전") },
    updated()       { console.log("데이터 갱신 완료 ← 이벤트 발생 후") },
    beforeUnmount() { console.log("가상돔 해제 전") },
    unmounted()     { console.log("가상돔 해제 완료") }
})
```

| 생명주기 | 시점 | 주요 용도 |
|---------|------|---------|
| `mounted()` | DOM 준비 완료 | 서버 데이터 요청, 라이브러리 연동 ⭐ |
| `updated()` | data 변경 후 DOM 갱신 완료 | 갱신 후 추가 처리 |
| `unmounted()` | 컴포넌트 제거 | 타이머/이벤트 정리 |

---

## 4. 디렉티브 총정리 (vue_4.jsp ~ vue_5.jsp)

```html
<!-- v-model : 양방향 바인딩 (input ↔ data 동기화) -->
<input type="text" v-model="msg">
<div>{{msg}}</div>

<!-- v-if / v-else-if / v-else -->
<div v-if="type===0">선택 없음</div>
<div v-else-if="type===1">한식</div>
<div v-else-if="type===2">양식</div>
<div v-else>기타</div>

<!-- v-show : DOM은 유지, display만 토글 (v-if와 차이) -->
<div v-show="isShow">상세 정보</div>

<!-- v-for : 반복 렌더링 -->
<tr v-for="vo in movies">
    <td>{{vo.rank}}</td>
    <td><img :src="'https://www.kobis.or.kr' + vo.thumbUrl"></td>
</tr>
<!-- ⚠️ v-for와 v-if 동시 사용 불가 → 상위 태그 분리 필요 -->

<!-- v-bind : 속성 동적 연결 (:으로 생략 가능) -->
<img :src="vo.poster" :title="vo.name">

<!-- v-on : 이벤트 (@으로 생략 가능) -->
<button @click="select(1)">한식</button>
<button v-on:click="select(2)">양식</button>
<input @keydown.enter="find()">   <!-- Enter키 감지 -->

<!-- v-text / v-html -->
<p v-text="vo.name"></p>          <!-- textContent와 동일 -->
<div v-html="vo.content"></div>   <!-- innerHTML과 동일 -->
```

---

## 5. function vs 화살표 함수 — this 차이 (js.jsp)

```javascript
const obj = { name: 'Hong Gil Dong' }

function aaa() {
    console.log(this.name)  // 호출 시점에 this 결정
}
const bbb = () => {
    console.log(this.name)  // 선언 시점에 this 결정 (외부 this 고정)
}

aaa.call(obj)  // → "Hong Gil Dong" (this = obj로 변경됨)
bbb.call(obj)  // → undefined (화살표 함수는 call로 this 변경 불가)
```

| | 일반 function | 화살표 함수 |
|--|-------------|-----------|
| this 결정 시점 | 호출 시 | 선언 시 |
| this 변경 | call/apply/bind 가능 | 불가 |
| Vue methods | 사용 가능 | 사용 가능 |
| 콜백 (geocoder 등) | this 잃어버림 주의 ⭐ | 외부 this 유지 → 권장 |

> geocoder.addressSearch() 콜백에서 `this.food_detail` 접근하려면 반드시 화살표 함수 사용

---

## 6. axios POST 방식 + ref 유효성 검사 (find.jsp)

```javascript
// axios.post(url, body, config)
// body는 비워두고 config의 params에 쿼리스트링 전달
axios.post('../food/find_vue.do', {}, {
    params: {
        page: this.curpage,
        column: this.column,  // v-model로 select 값
        ss: this.ss           // v-model로 input 값
    }
}).then(response => {
    this.food_list  = response.data.food_list
    this.startPage  = response.data.startPage
    this.endPage    = response.data.endPage
    this.curpage    = response.data.curpage
    this.totalpage  = response.data.totalpage
})

// ref : Vue 내부에서 DOM 직접 접근 (jQuery $() 대신)
// HTML : <input ref="ssInput">
// JS   : this.$refs.ssInput.focus()
find() {
    this.curpage = 1
    if (this.ss.trim() === "") {
        this.$refs.ssInput.focus()  // ⭐ Vue에서 DOM 직접 접근
        return
    }
    this.dataRecv()
}
```

---

## 7. 컴포넌트 (Component) 패턴 ⭐

```javascript
// ① 컴포넌트 정의 (외부 JS 파일로 분리)
// page_card.js
const page_card = {
    template: `
        <ul class="pagination">
            <li v-if="$parent.startPage > 1">
                <a @click="$parent.move($parent.startPage - 1)">&laquo;</a>
            </li>
            <li v-for="i in $parent.range($parent.startPage, $parent.endPage)"
                :class="i === $parent.curpage ? 'active' : ''">
                <a @click="$parent.move(i)">{{i}}</a>
            </li>
            <li v-if="$parent.endPage < $parent.totalpage">
                <a @click="$parent.move($parent.endPage + 1)">&raquo;</a>
            </li>
        </ul>
    `
}

// food_detail.js — props로 부모 데이터 수신
const food_detail = {
    props: ['food_detail'],  // 부모가 v-bind로 전달한 값 수신
    template: `
        <table class="table">
            <tr>
                <td><img :src="food_detail.poster"></td>
                <td>{{food_detail.name}}</td>
            </tr>
        </table>
    `
}

// ② 부모 앱에서 등록 + 사용
Vue.createApp({
    components: {
        pagecard: page_card,
        food_detail: food_detail  // kebab-case로 HTML에서 사용
    }
}).mount('.container')
```

```html
<!-- ③ HTML에서 컴포넌트 태그로 사용 -->
<pagecard></pagecard>
<food_detail v-bind:food_detail="food_detail"></food_detail>
<!--          ↑ 컴포넌트의 props명   ↑ 부모의 data() 변수명  -->
```

| 방식 | 데이터 접근 |
|------|-----------|
| `$parent.변수` | 컴포넌트에서 부모 data 직접 접근 |
| `props: ['변수']` | 부모가 v-bind로 전달한 값 수신 (권장) |

---

## 8. 목록+상세 동시 표시 패턴 (list.jsp + main.js)

```javascript
// data()
data() {
    return {
        food_list: [],   // [] = Java ArrayList
        food_detail: {}, // {} = Java VO (빈 객체로 초기화)
        isShow: false    // 상세 패널 표시 여부
    }
},
methods: {
    // 목록 카드 클릭 → 상세 패널 표시
    detail(no) {
        this.isShow = true  // v-show로 우측 패널 표시
        axios.get('../food/detail_vue.do', { params: { no: no } })
             .then(response => {
                 this.food_detail = response.data  // {} VO에 데이터 저장
             })
    },
    // 페이징 : range() 함수로 숫자 배열 생성 (v-for용)
    range(start, end) {
        let arr = []
        for (let i = 0; i <= end - start; i++) arr[i] = start + i
        return arr
    },
    move(page) {
        this.curpage = page
        this.dataRecv()
    }
}
```

```html
<!-- 목록(8칸) + 상세(4칸) 분할 레이아웃 -->
<div class="col-sm-8">
    <div v-for="vo in food_list" @click="detail(vo.no)">
        <img :src="'https://www.menupan.com' + vo.poster">
        <p v-text="vo.name"></p>
    </div>
    <pagecard></pagecard>
</div>
<div class="col-sm-4" v-show="isShow">        <!-- 클릭 전 숨김 -->
    <food_detail :food_detail="food_detail"></food_detail>
</div>
```

---

## 9. 카카오맵 + Geocoder — 데이터 수신 후 동적 로드 (detail.jsp)

```javascript
mounted() {
    axios.get('../food/detail_vue.do', { params: { no: this.no } })
         .then(response => {
             this.food_detail = response.data

             // 카카오맵 SDK가 이미 로드됐으면 바로 초기화
             if (window.kakao && window.kakao.maps) {
                 this.initMap()
             } else {
                 this.addScript()  // 없으면 동적으로 script 태그 추가
             }
         })
},
methods: {
    // 카카오맵 SDK 동적 로드
    addScript() {
        const script = document.createElement("script")
        script.onload = () => kakao.maps.load(this.initMap)
        script.src = "http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=발급키&libraries=services"
        document.head.appendChild(script)
    },
    initMap() {
        var map = new kakao.maps.Map(document.getElementById('map'), {
            center: new kakao.maps.LatLng(33.450701, 126.570667),
            level: 3
        })
        var geocoder = new kakao.maps.services.Geocoder()

        // ⭐ 화살표 함수 사용 → this.food_detail 접근 가능
        geocoder.addressSearch(this.food_detail.address, (result, status) => {
            if (status === kakao.maps.services.Status.OK) {
                var coords = new kakao.maps.LatLng(result[0].y, result[0].x)

                new kakao.maps.Marker({ map: map, position: coords })

                var infowindow = new kakao.maps.InfoWindow({
                    content: '<div style="padding:6px">' + this.food_detail.name + '</div>'
                })
                infowindow.open(map, marker)
                map.setCenter(coords)  // 검색 주소로 지도 중심 이동
            }
        })
    }
}
```

> ⭐ `autoload=false` → SDK 로드 완료 후 `kakao.maps.load(callback)` 으로 수동 초기화  
> ⭐ 데이터 수신 후 지도 로드 → 화살표 함수로 `this` 유지 필수

---

## 10. 실습 파일 목록

| 파일 | 주제 | 핵심 내용 |
|------|------|---------|
| `vue_1.jsp` | Vue 기초 | `v-model` 양방향, jQuery와 비교 |
| `vue_2.jsp` | 생명주기 | beforeCreate → mounted → updated 순서 |
| `vue_3.jsp` | 다중 인스턴스 | 영역별 각각 다른 Vue 앱 적용 |
| `vue_4.jsp` | 디렉티브 | `v-if/else-if/else`, `@click`, `v-on` |
| `vue_5.jsp` | v-for + axios | 영화진흥원 API, `:src` 동적 바인딩 |
| `js.jsp` | this 차이 | function vs 화살표 함수 call/apply |
| `find.jsp` | 검색 + ref | `axios.post`, `v-model` select, `$refs` |
| `list.jsp` | 목록+상세 | 컴포넌트 2개, `v-show`, `isShow` 토글 |
| `detail.jsp` | 상세+카카오맵 | Geocoder, 동적 SDK 로드, 화살표함수 this |
| `main.js` | 목록 앱 | `range()`, `move()`, `detail()`, components |
| `page_card.js` | 페이징 컴포넌트 | `$parent` 접근, `v-for` 페이지 버튼 |
| `food_detail.js` | 상세 컴포넌트 | `props`, template 리터럴 |

---
