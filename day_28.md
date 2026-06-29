# 📘 Day 28 — Vue.js로 게시판 CRUD 구현 (목록/상세/등록/수정/삭제)

## 0. 핵심 빠른 참조

| 구분 | 문법 | 설명 |
|------|------|------|
| ES Module 방식 | `<script type="module">` + `import {createApp} from "vue"` | 최신 방식, importmap 필요 |
| 일반 CDN 방식 | `<script src="vue.global.js">` + `Vue.createApp({})` | 기존 방식 |
| async/await | `async dataRecv()` + `await axios.get()` | 비동기 순차 처리 ⭐ |
| 이전/다음 페이징 | `prev()` / `next()` | 삼항 연산자로 범위 초과 방지 |
| 역순 번호 | `{{count - index}}` | v-for index 활용 |
| 토글 버튼 | `isOn = !isOn` + `{{isOn?'삭제':'취소'}}` | boolean 반전으로 텍스트 전환 |
| 삭제 패널 토글 | `bShow = !bShow` + `v-show="bShow"` | 비밀번호 입력창 show/hide |
| 유효성 검사 | `this.$refs.ref명.focus()` + `return` | 빈 값 → 포커스 후 중단 |
| 수정 데이터 로드 | `mounted()` → axios GET → `this.변수 = response.data.필드` | 기존 값 미리 채우기 |
| 삭제 결과 처리 | `response.data === 'yes'` → 이동 / 아니면 alert | 서버 문자열로 분기 |
| 화면 이동 | `window.location.href = "URL"` | sendRedirect와 동일 |
| axios POST 패턴 | `axios.post(url, {}, {params:{데이터}})` | body 비우고 params에 전달 |

---

## 1. JSP vs Vue 게시판 비교

```text
JSP 방식
  form submit → 서버 → sendRedirect → 새 페이지 로딩 (깜박임)
  입력 데이터 손실 가능

Vue + Axios 방식
  버튼 @click → axios → 서버 응답 → 화면 일부만 갱신 (깜박임 없음)
  data() 변수에 값 유지 → v-model로 양방향 연결

기술 발전 흐름
  JSP (MVC) → Ajax (HTML 조각 삽입) → Vue (HTML 태그 직접 제어)
  최종 목표 : ThymeLeaf + Spring-Boot + Pinia → MyBatis
```

---

## 2. ES Module 방식 (list.jsp) ⭐

```html
<!-- importmap : import 경로 별칭 지정 -->
<script type="importmap">
{
    "imports": {
        "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
    }
}
</script>

<!-- type="module" : import 구문 사용 가능 -->
<script type="module">
    import { createApp } from "vue"   // Java의 import와 동일
    const app = createApp({ ... })
    app.mount(".container")
</script>
```

| 방식 | 선언 | 특징 |
|------|------|------|
| CDN 전역 | `Vue.createApp({})` | 기존 방식, 간단 |
| ES Module | `import {createApp} from "vue"` | 최신 방식, 모듈 분리 가능 |

---

## 3. 목록 + 페이징 (list.jsp)

```javascript
// async/await : 비동기 처리를 순차적으로 실행
async dataRecv() {
    await axios.get('../board/list_vue.do', {
        params: { page: this.curpage }
    }).then(response => {
        this.list      = response.data.list
        this.curpage   = response.data.curpage
        this.totalpage = response.data.totalpage
        this.count     = response.data.count   // 전체 글수 (역순 번호용)
    })
},

// 이전/다음 버튼 : 삼항 연산자로 범위 초과 방지
prev() {
    this.curpage = this.curpage > 1 ? this.curpage - 1 : this.curpage
    this.dataRecv()
},
next() {
    this.curpage = this.curpage < this.totalpage ? this.curpage + 1 : this.curpage
    this.dataRecv()
}
```

```html
<!-- v-for : (vo, index) → index로 역순 번호 계산 -->
<tr v-for="(vo, index) in list" :key="index">
    <td>{{count - index}}</td>               <!-- 역순 번호 ⭐ -->
    <td><a :href="'../board/detail.do?no=' + vo.no">{{vo.subject}}</a></td>
    <td>{{vo.name}}</td>
    <td>{{vo.dbday}}</td>
    <td>{{vo.hit}}</td>
</tr>
<tr>
    <td colspan="5" class="text-center">
        <button @click="prev()">이전</button>
        {{curpage}} page / {{totalpage}} pages
        <button @click="next()">다음</button>
    </td>
</tr>
```

---

## 4. 상세보기 + 삭제 (detail.jsp) ⭐

```javascript
data() {
    return {
        bShow: false,   // 비밀번호 입력창 표시 여부
        isOn: true,     // true='삭제' / false='취소' 텍스트 전환
        pwd: '',
        detail: {},     // VO (빈 객체로 초기화)
        no: ${param.no} // JSP EL로 URL 파라미터 받기
    }
},
mounted() {
    axios.get('../board/detail_vue.do', { params: { no: this.no } })
         .then(response => { this.detail = response.data })
},
methods: {
    // 삭제 버튼 토글 : 버튼 텍스트 + 비밀번호 입력창 동시 제어
    btnClick() {
        this.isOn  = !this.isOn   // true↔false 반전
        this.bShow = !this.bShow  // 비밀번호 입력창 show/hide
    },
    del() {
        if (this.pwd.trim() === "") {
            this.$refs.pwdRef.focus()
            return
        }
        axios.get('../board/delete_vue.do', {
            params: { no: this.no, pwd: this.pwd }
        }).then(response => {
            if (response.data === 'yes') {
                window.location.href = "../board/list.do"  // 삭제 성공 → 목록
            } else {
                alert("비밀번호가 틀립니다!!")
                this.pwd = ''
                this.$refs.pwdRef.focus()
            }
        })
    }
}
```

```html
<!-- 삭제 버튼 : isOn에 따라 텍스트 전환 -->
<a @click="btnClick()">{{isOn ? '삭제' : '취소'}}</a>

<!-- 비밀번호 입력창 : bShow에 따라 표시/숨김 -->
<tr v-show="bShow">
    <td>
        비밀번호: <input type="password" ref="pwdRef" v-model="pwd">
        <button @click="del()">삭제</button>
    </td>
</tr>
```

---

## 5. 글쓰기 (insert.jsp)

```javascript
data() {
    return { name: '', subject: '', content: '', pwd: '' }
    // mounted() 없음 — 기존 데이터 로드 불필요
},
methods: {
    // 유효성 검사 패턴 : 빈 값 → ref.focus() → return으로 중단
    write() {
        if (this.name.trim()    === "") { this.$refs.nameRef.focus(); return }
        if (this.subject.trim() === "") { this.$refs.subRef.focus();  return }
        if (this.content.trim() === "") { this.$refs.contRef.focus(); return }
        if (this.pwd.trim()     === "") { this.$refs.pwdRef.focus();  return }

        // axios POST : body 비우고 params에 데이터 전달
        axios.post('../board/insert_ok.do', {}, {
            params: {
                name: this.name, subject: this.subject,
                content: this.content, pwd: this.pwd
            }
        }).then(response => {
            window.location.href = "../board/list.do"  // 등록 후 목록 이동
        })
    },
    cancel() { window.location.href = "../board/list.do" }
}
```

```html
<!-- v-model + ref 동시 사용 : 양방향 바인딩 + DOM 직접 접근 -->
<input type="text" ref="nameRef" v-model="name">
<input type="text" ref="subRef"  v-model="subject">
<textarea         ref="contRef" v-model="content"></textarea>
<input type="password" ref="pwdRef" v-model="pwd">

<button @click="write()">글쓰기</button>
<button @click="cancel()">취소</button>
```

---

## 6. 수정하기 (update.jsp)

```javascript
data() {
    return {
        no: ${param.no},  // URL에서 글번호 받기
        name: '', subject: '', content: '', pwd: ''
    }
},
// ⭐ mounted : 기존 데이터를 서버에서 받아 입력창에 미리 채움
mounted() {
    axios.get('../board/update_vue.do', { params: { no: this.no } })
         .then(response => {
             // v-model 변수에 대입 → input에 자동 반영
             this.name    = response.data.name
             this.subject = response.data.subject
             this.content = response.data.content
             // ⚠️ pwd는 보안상 서버에서 내려주지 않음 → 사용자가 직접 입력
         })
},
methods: {
    write() {
        // insert와 동일한 유효성 검사 패턴
        if (this.name.trim()    === "") { this.$refs.nameRef.focus(); return }
        if (this.subject.trim() === "") { this.$refs.subRef.focus();  return }
        if (this.content.trim() === "") { this.$refs.contRef.focus(); return }
        if (this.pwd.trim()     === "") { this.$refs.pwdRef.focus();  return }

        axios.post('../board/update_ok.do', {}, {
            params: {
                no: this.no, name: this.name,
                subject: this.subject, content: this.content, pwd: this.pwd
            }
        }).then(response => {
            if (response.data === 'yes') {
                window.location.href = "../board/detail.do?no=" + this.no
            } else {
                alert("비밀번호가 틀립니다!!")
                this.pwd = ''
                this.$refs.pwdRef.focus()
            }
        })
    },
    cancel() { window.location.href = "../board/detail.do?no=" + this.no }
}
```

---

## 7. Vue 게시판 CRUD 전체 흐름 요약

```text
[목록] list.do → list.jsp
  └─ mounted() → axios GET list_vue.do → list[] + 페이징 데이터
  └─ v-for (vo, index) → count-index 역순 번호
  └─ prev()/next() → curpage 증감 → dataRecv() 재호출

[상세] detail.do → detail.jsp
  └─ no = ${param.no} (JSP EL로 URL 파라미터 수신)
  └─ mounted() → axios GET detail_vue.do → detail{}
  └─ 삭제 버튼 : btnClick() → isOn/bShow 토글
  └─ del() → axios GET delete_vue.do → 'yes'/'no' 분기

[등록] insert.do → insert.jsp
  └─ mounted() 없음 (빈 폼)
  └─ write() → 유효성 검사 → axios POST insert_ok.do → list.do 이동

[수정] update.do → update.jsp
  └─ mounted() → axios GET update_vue.do → 기존 값 input에 채우기
  └─ write() → 유효성 검사 → axios POST update_ok.do → 'yes'/'no' 분기
```

---

## 8. 패턴 정리 — 다시 만들 때 체크리스트

```text
① data() 변수 설계
   - VO 타입 → {}  (detail:{})
   - 배열 타입 → [] (list:[])
   - URL 파라미터 → no:${param.no}
   - 토글용 boolean → bShow:false, isOn:true

② mounted() 필요 여부
   - 목록/상세/수정 → 필요 (서버에서 데이터 로드)
   - 등록 → 불필요 (빈 폼)

③ 유효성 검사 순서
   name → subject → content → pwd
   → 각각 this.$refs.ref명.focus() + return

④ axios 방식 선택
   - 데이터 조회 → GET + params
   - 등록/수정 → POST + ({}, {params:{}})
   - 삭제 → GET + params (간단한 경우)

⑤ 서버 응답 처리
   - 목록/상세 → response.data.필드명 으로 변수에 대입
   - 등록 → 완료 후 바로 이동
   - 수정/삭제 → response.data === 'yes' 분기
```

---

## 9. 실습 파일 목록

| 파일 | 역할 | 핵심 내용 |
|------|------|---------|
| `list.jsp` | 게시판 목록 | ES Module, async/await, v-for index, prev/next |
| `detail.jsp` | 상세보기 + 삭제 | isOn 토글, v-show, ref, 삭제 'yes'/'no' 분기 |
| `insert.jsp` | 글쓰기 | v-model + ref 유효성 검사, axios POST |
| `update.jsp` | 수정하기 | mounted() 기존 값 로드, POST + 비밀번호 검증 |

---
