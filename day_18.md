# 📘 Day 18 — JavaScript 기초

## 0. 핵심 빠른 참조

| 구분 | 예시 | 설명 |
|------|------|------|
| 변수 선언 | `let a = 10` | 재할당 가능, 블록 스코프 ⭐ |
| 상수 선언 | `const PI = 3.14` | 재할당 불가 |
| 구식 변수 | `var a = 10` | 중복 선언 가능, 스코프 불명확 → 사용 자제 |
| 자료형 확인 | `typeof a` | number/string/boolean/object/function/undefined |
| 출력 (콘솔) | `console.log(a)` | 개발자 도구에서 확인 |
| 출력 (화면) | `document.write("<h1>"+a+"</h1>")` | 브라우저에 직접 출력 |
| 출력 (태그) | `document.getElementById("id").innerHTML = "값"` | 특정 태그에 값 삽입 ⭐ |
| 알림 팝업 | `alert("메시지")` | 팝업창 출력 |
| 숫자 변환 | `Number("10")` / `parseInt("10")` | 문자 → 숫자 |
| 문자 변환 | `String(10)` | 숫자 → 문자 |
| 논리 변환 | `Boolean(0)` → false | 0, 0.0, null, "" 만 false |
| 동등 비교 | `==` | 자료형 무관 비교 |
| 일치 비교 | `===` | 자료형까지 비교 ⭐ |
| 삼항 연산자 | `(6%2==0) ? "짝수" : "홀수"` | if-else 축약 |
| for in | `for(let i in arr)` | 인덱스 반복 |
| for of | `for(let v of arr)` | 값 직접 반복 ⭐ |
| forEach | `arr.forEach((v,i) => {})` | 콜백 반복 ⭐ |
| map | `arr.map((v,i) => {})` | 변환 + 반복 ⭐ |

---

## 1. JavaScript 개요

```
역할     HTML/CSS로 만든 정적 페이지를 동적으로 변경
실행환경  브라우저 (인터프리터 방식 — 한 줄씩 실행)
특징     컴파일 없음 / 객체 기반 / 이벤트 중심 / 비동기 처리 가능

사용처
  웹 동적 처리  로그인, 팝업, 검색, 자동완성, 유효성 검사
  Back-End     NodeJS
  모바일       React Native
  데스크탑     VSCode, VuErd
  게임/채팅/AI

자바스크립트 단독으로는 DB 연결 불가
  자바스크립트 ←→ Java(Spring-Boot) ←→ Oracle
  Vue / React + Spring-Boot → MSA
```

### JavaScript 작성 방식
```html
<!-- 1. 내부 script — 현재 파일만 적용 -->
<head>
  <script type="text/javascript">
    window.onload = function() { /* 처리 */ }
  </script>
</head>

<!-- 2. 외부 script — .js 파일로 분리 -->
<script src="파일명.js"></script>

<!-- 3. 인라인 — 태그에 직접 -->
<button onclick="history.back()">뒤로</button>
```

### 프레임워크별 시작 함수
```javascript
window.onload = function() {}  // 바닐라JS
$(function() {})               // jQuery
mounted() {}                   // Vue
componentDidMount() {}         // React (클래스형)
useEffect(() => {}, [])        // React (함수형)
```

---

## 2. 변수 / 자료형

### 변수 선언
```javascript
let   a = 10        // 재할당 가능, 블록 스코프 ⭐ (권장)
const b = 3.14      // 재할당 불가 (서버에서 받은 값 저장 시)
var   c = "hello"   // 중복 선언 가능, 스코프 불명확 → 사용 자제

// var 단점 예시 (중복 선언 가능 — 버그 원인)
var x = 10
var x = 20  // 에러 없이 덮어씀 → let은 에러 발생
```

### 자료형 — 데이터형 선언 없이 자동 인식
```javascript
let a = 10            // number (정수)
let b = 10.5          // number (실수)
let c = "Hello"       // string
let d = 'Hello'       // string (작은따옴표도 동일)
let e = [1, 2, 3]     // object (배열)
let f = {name:"홍길동", age:20}  // object (객체/JSON)
let g = function() {} // function
let h = true          // boolean
let m                 // undefined (값 없음)

// 자료형 확인
console.log(typeof a)  // "number"
console.log(typeof c)  // "string"
console.log(typeof e)  // "object"
console.log(typeof g)  // "function"
console.log(typeof m)  // "undefined"
```

### 자료형 구분
| 분류 | 자료형 | 설명 |
|------|--------|------|
| 기본형 | `number` | 정수/실수 구분 없음 |
| 기본형 | `string` | "" '' 모두 문자열 |
| 기본형 | `boolean` | true / false |
| 기본형 | `null` | 값이 없음 (명시적) |
| 기본형 | `undefined` | 값이 없음 (선언만 된 상태) |
| 참조형 | `object` | 배열, 객체, null |
| 참조형 | `function` | 함수도 데이터형으로 취급 ⭐ |

> TypeScript를 쓰면 `let a:string = ""` 처럼 자료형 명시 가능 → 가독성 향상

### 형변환
```javascript
// 문자 → 숫자
Number("10")    // 10
parseInt("10")  // 10 (정수만)
parseFloat("10.5") // 10.5

// 숫자 → 문자
String(10)      // "10"

// Boolean 변환
Boolean(0)      // false
Boolean(0.0)    // false
Boolean(null)   // false
Boolean("")     // false
Boolean("Hi")   // true  ← 0, 0.0, null, "" 외 모두 true
Boolean(1)      // true

// ⚠️ HTML input에서 읽은 값은 항상 문자열
// 숫자 계산 시 반드시 변환 필요
let val = document.getElementById("num").value  // "10" (string)
parseInt(val) + 5  // 15 ✅
val + 5            // "105" ❌ 문자열 결합
```

---

## 3. 연산자

### 산술 연산자
```javascript
let a = 5, b = 2
console.log(a + b)  // 7
console.log(a - b)  // 3
console.log(a * b)  // 10
console.log(a / b)  // 2.5  ← Java와 달리 정수/정수 = 실수
console.log(a % b)  // 1

// + 연산자 주의 — 문자열이 있으면 결합
console.log("10" + 20)  // "1020" (문자열 결합)
console.log(parseInt("10") + 20)  // 30 ✅

// 연산 불가 → NaN
console.log("abc" * 2)  // NaN
console.log(10 / 0)     // Infinity (Java와 다름 — 에러 아님)
```

### 비교 연산자
```javascript
let m = 10, n = "10"
console.log(m == n)   // true  ← 자료형 무관 비교 (암묵적 변환)
console.log(m === n)  // false ← 자료형까지 비교 ⭐ (권장)
console.log(m !== n)  // true

// 실무 활용
if(frm.id.value === "")  // NOT NULL 검사 ⭐
if(frm.pwd.value !== frm.pwd1.value)  // 비밀번호 확인 ⭐
```

### 논리 / 대입 / 삼항 연산자
```javascript
// 논리
(6 < 7) && (6 == 7)  // false (둘 다 참이어야)
(6 < 7) || (6 == 7)  // true  (하나만 참이면)

// 대입
let k = 10
k += 10  // k = 20
k -= 10  // k = 10

// 삼항 ⭐
let result = (6 % 2 === 0) ? "짝수" : "홀수"  // "짝수"

// 증감 연산자
let a = 10
let b = a++   // b=10, a=11 (후치: 대입 후 증가)
let c = ++a   // a=12, c=12 (전치: 증가 후 대입)
```

---

## 4. 제어문

### 조건문
```javascript
// if — 단일
if(score >= 90) {
  document.write("<h1>A등급</h1>")
}

// if~else if~else — 다중
if(score >= 90)      grade = "A"
else if(score >= 80) grade = "B"
else if(score >= 70) grade = "C"
else                 grade = "F"

// switch — 선택 (break 빠뜨리면 다음 case 계속 실행 주의)
let i = 3
switch(i) {
  case 1: document.write("문장1")  // break 없음 → 아래로 진행
  case 2: document.write("문장2")
  case 3: document.write("문장3")  // i=3이면 여기서 시작
  case 4: document.write("문장4")
    break   // 여기서 종료
  case 5: document.write("문장5")
}
// 출력: 문장3, 문장4
```

### 폼 유효성 검사 패턴 ⭐
```javascript
function ok() {
  let frm = document.frm

  if(frm.id.value === "") {
    alert("아이디 입력!!")
    return  // 이후 코드 실행 중단
  }
  if(frm.pwd.value === "") {
    alert("비밀번호 입력!!")
    return
  }
  if(frm.pwd.value !== frm.pwd1.value) {
    alert("비밀번호가 일치하지 않습니다")
    return
  }
  // 모든 검사 통과 → 폼 전송
  frm.submit()
}
```

---

## 5. 반복문

```javascript
// do~while — 반드시 1번 이상 실행 (조건을 나중에 검사)
let i = 1
do {
  document.write("<h3>" + i + "</h3>")
  i++
} while(i <= 10)

// while — 반복 횟수 모를 때
i = 1
while(i <= 10) {
  document.write("<h3>" + i + "</h3>")
  i++
}

// for — 반복 횟수 정해진 경우 ⭐
for(let i = 1; i <= 10; i++) {
  document.write("<h3>" + i + "</h3>")
}
```

### 배열 전용 반복문 4가지 ⭐
```javascript
let names = ["홍길동", "심청이", "이순신", "강감찬", "춘향이"]

// 1. for in — 인덱스 반복
for(let index in names) {
  document.write("<li>" + names[index] + "</li>")
}

// 2. for of — 값 직접 반복 ⭐
for(let name of names) {
  document.write("<h3>" + name + "</h3>")
}

// 3. forEach — 콜백 함수 (name:값, index:번호)
names.forEach(function(name, index) {
  document.write("<h3>" + name + "</h3>")
})

// 4. map — 변환 + 반복 ⭐ (React에서 목록 렌더링에 주로 사용)
names.map(function(name, index) {
  document.write("<h3>" + (index + 1) + ". " + name + "</h3>")
})
```

### 배열 반복문 비교
| 방식 | 특징 | 주요 사용처 |
|------|------|------------|
| `for in` | 인덱스 반환 | 인덱스가 필요한 경우 |
| `for of` | 값 직접 반환 | 값만 필요한 경우 ⭐ |
| `forEach` | 콜백, 반환값 없음 | 단순 출력/처리 |
| `map` | 콜백, 새 배열 반환 | React 목록 렌더링, 데이터 변환 ⭐ |

---

## 6. Axios — 비동기 서버 통신

```javascript
// Axios CDN
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

// GET 요청 예시
function movieList(no) {
  let site = "https://www.kobis.or.kr/kobis/business/main/"
  if(no == 1) site += "searchMainDailyBoxOffice.do"
  if(no == 2) site += "searchMainRealTicket.do"
  if(no == 3) site += "searchMainDailySeatTicket.do"

  axios.get(site).then(response => {
    console.log(response.data)  // 서버 응답 데이터 출력
  })
}
```

> Axios = jQuery의 `$.ajax()` 와 동일한 역할
> 서버에서 받은 JSON → 자바스크립트 배열(`[]`) / 객체(`{}`) 로 변환해서 사용

---

## 7. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `js_1.jsp` | 변수/자료형/typeof/출력 방법 4가지 |
| `js_2.jsp` | 단항연산자(증감/부정/형변환) + 계산기 예제 (innerHTML) |
| `js_3.jsp` | 산술/비교(==, ===)/논리 연산자 + 폼 유효성 검사 |
| `js_4.jsp` | 논리/대입/삼항 연산자 + `10/0 = Infinity` |
| `js_5.jsp` | switch-case (break 생략 fall-through) + Axios GET |
| `js_6.jsp` | do~while / while / for 반복문 비교 |
| `js_7.jsp` | 배열 반복문 4종 (for in / for of / forEach / map) |
