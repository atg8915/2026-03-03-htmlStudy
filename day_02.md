# 📘 Day 02 — 입력 태그 / 목록 태그 / table / 링크 / jQuery

## 0. 핵심 태그 빠른 참조

| 태그 | 예시 | 설명 |
|------|------|------|
| `input type="text"` | `<input type="text" size="15">` | 한 줄 텍스트 입력 |
| `input type="password"` | `<input type="password">` | 비밀번호 입력 |
| `input type="file"` | `<input type="file" multiple>` | 파일 업로드 |
| `input type="date"` | `<input type="date">` | 달력 (체크인/생년월일) |
| `input type="number"` | `<input type="number">` | 숫자 spinner |
| `input type="range"` | `<input type="range" min="0" max="100">` | 슬라이더 |
| `input type="radio"` | `<input type="radio" name="sex">` | 라디오 버튼 (name으로 그룹) |
| `input type="checkbox"` | `<input type="checkbox">` | 체크박스 |
| `input type="hidden"` | `<input type="hidden">` | 숨김 필드 (보안) |
| `input type="submit"` | `<input type="submit" value="전송">` | 전송 버튼 → JSP로 전달 |
| `input type="button"` | `<input type="button" value="취소">` | 일반 버튼 (JS 연결) |
| `input type="reset"` | `<input type="reset">` | 초기화 버튼 |
| `textarea` | `<textarea rows="5" cols="30"></textarea>` | 여러 줄 입력 |
| `select` | `<select><option>항목</option></select>` | 콤보박스 |
| `fieldset` | `<fieldset><legend>그룹명</legend></fieldset>` | 입력창 그룹화 |
| `label` | `<label for="name">이름</label>` | 입력 필드 설명 |
| `figure` | `<figure><img><figcaption>설명</figcaption></figure>` | 이미지 + 설명 카드 |
| `ul` | `<ul><li>항목</li></ul>` | 순서 없는 목록 (메뉴) |
| `ol` | `<ol><li>항목</li></ol>` | 순서 있는 목록 (단계) |
| `dl` | `<dl><dt>제목</dt><dd>설명</dd></dl>` | 상세보기형 목록 |
| `table` | `<table><thead><tbody><tfoot>` | 2차원 표 (게시판) |
| `colspan` | `<td colspan="3">` | 가로 셀 병합 |
| `rowspan` | `<td rowspan="6">` | 세로 셀 병합 |
| `a` | `<a href="URL" target="_blank">텍스트</a>` | 링크 이동 |

---

## 1. 입력 태그

```html
<!-- 주요 input 타입 -->
<input type="text">        <!-- ID, 이름, 주소 -->
<input type="password">    <!-- 비밀번호 -->
<input type="file" multiple> <!-- 파일 업로드 -->
<input type="date">        <!-- 달력 -->
<input type="number">      <!-- 수량 -->
<input type="range" min="10000" max="100000"> <!-- 가격 슬라이더 -->
<input type="hidden">      <!-- 보안값 숨김 -->
<input type="radio" name="sex"> 남자  <!-- name으로 그룹 -->
<input type="checkbox"> 낚시

<!-- 여러줄 입력 -->
<textarea rows="5" cols="30"></textarea>

<!-- 콤보박스 -->
<select>
  <option>업체명</option>
  <option>주소</option>
</select>
```

### fieldset — 입력창 그룹화
```html
<fieldset>
  <legend>배송정보</legend>
  <label>주소 <input type="text"></label>
  <label>전화 <input type="text"></label>
</fieldset>
```

### figure — 이미지 카드
```html
<figure>
  <img src="m1.jpg" width="250" height="300">
  <figcaption>이미지 설명 텍스트</figcaption>
</figure>
```

---

## 2. 목록 태그

```html
<!-- ul : 순서 없는 목록 → 메뉴 -->
<ul>
  <li>HTML/CSS</li>
  <li>JavaScript</li>
</ul>

<!-- ol : 순서 있는 목록 → 단계 설명, 요리 순서 -->
<ol>
  <li>자바</li>
  <li>오라클</li>
</ol>

<!-- dl : 제목 + 설명 → 상세보기 -->
<dl>
  <dt>자바</dt>
  <dd>데이터 관리, 오라클/브라우저 연동</dd>
  <dt>오라클</dt>
  <dd>데이터 저장 (SQL)</dd>
</dl>
```

---

## 3. table

```html
<table border="1" width="700">
  <thead>
    <tr>
      <th>번호</th><th>이름</th><th>점수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td><td>홍길동</td><td>90</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="3" align="right">총인원: 1</td>
    </tr>
  </tfoot>
</table>
```

### colspan / rowspan
```html
<!-- 가로 병합 -->
<td colspan="3">제목</td>

<!-- 세로 병합 (이미지 + 여러 행) -->
<td rowspan="6"><img src="m1.jpg"></td>
```

> `thead` → 한 번만 / `tbody` → 반복 데이터 / `tfoot` → 통계

---

## 4. 링크 태그 `<a>`

```html
<!-- 기본 링크 -->
<a href="http://naver.com" target="_blank">네이버</a>

<!-- 이미지 클릭 → 페이지 이동 -->
<a href="detail.html">
  <img src="m1.jpg" width="300" height="350">
</a>

<!-- 뒤로 가기 -->
<a href="javascript:history.back()">목록</a>
```

```css
/* 링크 기본 스타일 제거 */
a { text-decoration: none; color: black; }
a:hover { text-decoration: underline; color: green; }
```

---

## 5. jQuery — 실시간 검색

```html
<script src="http://code.jquery.com/jquery.js"></script>
<script>
$(function() {
  $('#keyboard').keyup(function() {
    let k = $('#keyboard').val();  // 입력값 가져오기
    $('.tlb tbody tr').hide();     // 전체 숨기기
    let temp = $('.tlb tbody tr td:nth-child(5n+2):contains("' + k + '")');
    $(temp).parent().show();       // 일치하는 행만 보이기
  });
});
</script>
```

> `$(function(){})` = jQuery의 `main()` — DOM 로딩 완료 후 실행
> 바닐라JS: `window.onload` / Vue: `mounted()` / React: `componentDidMount()`

---

## 6. 실습 예제 목록

| 파일 | 내용 |
|------|------|
| `html_1.html` | input 태그 전체 종류 실습 |
| `html_2.html` | figure, fieldset, label 실습 |
| `html_3.html` | fieldset CSS 스타일링 |
| `html_4.html` | table + jQuery 실시간 검색 |
| `html_5.html` | 게시판 상세보기 (colspan/rowspan) |
| `html_6.html` | rowspan 상세보기 + 카카오맵 연동 |
| `login.html` | 로그인 폼 |
| `write.html` | 게시판 글쓰기 폼 |
