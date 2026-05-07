# 📘 Day 12 — HTML 기초

## 0. 핵심 태그 빠른 참조

| 태그 | 예시 | 설명 |
|------|------|------|
| `h1~h6` | `<h1>제목</h1>` | 제목 — 숫자 클수록 작아짐, bold, block |
| `br` | `<br>` | 줄바꿈 (독립 태그) |
| `p` | `<p>단락</p>` | 단락 나누기 (`<br><br>` 대체) |
| `hr` | `<hr>` | 수평선 구분선 |
| `pre` | `<pre>내용</pre>` | 있는 그대로 출력 (상세보기 본문) |
| `b` | `<b>굵게</b>` | 굵은 글씨 |
| `strong` | `<strong>강조</strong>` | 강조체 |
| `font` | `<font color="red">텍스트</font>` | 색상·글꼴 지정 |
| `sup` | `X<sup>2</sup>` | 위첨자 (새글 아이콘 등) |
| `sub` | `H<sub>2</sub>O` | 아래첨자 |
| `mark` | `<mark>형광</mark>` | 형광색 강조 |
| `del` | `<del>60,000원</del>` | 취소선 |
| `ins` | `<ins>30,000원</ins>` | 밑줄 |
| `blockquote` | `<blockquote>인용</blockquote>` | 들여쓰기 인용 |
| `marquee` | `<marquee>전광판</marquee>` | 흐르는 텍스트 |
| `img` | `<img src="파일명" title="설명">` | 이미지 출력 (inline, 독립 태그) |
| `embed` | `<embed src="URL" width="" height="">` | 동영상 삽입 |
| `iframe` | `<iframe src="파일명" width="" height="">` | 다른 HTML 파일 삽입 |

---

## 1. 웹 기본 개념

| 구분 | 설명 |
|------|------|
| HTML | 웹 페이지 구조를 만드는 마크업 언어, 확장자 `.html` `.htm` |
| 웹 | HTTP 기반으로 HTML 문서를 전달하는 서비스 |
| 인터넷 | 네트워크를 활용한 정보 공유 공간 |
| 브라우저 | HTML을 해석해서 화면에 렌더링 |
| 웹서버 | 요청 받기 / 응답하기 (톰캣: Java 번역 → HTML 변환) |

### 역할 분담
| 영역 | 기술 | 역할 |
|------|------|------|
| Front-End | HTML / CSS | 화면 UI (정적) |
| Front-End | JavaScript | 사용자 이벤트 처리, 동적 페이지 |
| Back-End | Java | 요청 처리, DB 연동 중간 매개 |
| Back-End | Oracle | 데이터 저장 |

### 주요 에러 코드
| 코드 | 원인 |
|------|------|
| `404` | 요청한 파일 없음 |
| `403` | 서버 접근 거부 |
| `400` | 전송값 불일치 |
| `415` | 한글 인코딩 오류 (UTF-8 확인) |
| `500` | Java 문법 에러 (JSP) |

---

## 2. HTML 문서 구조

```html
<!DOCTYPE html>        <!-- HTML5 버전 선언 -->
<html>                 <!-- 문서 시작 -->
<head>
  <meta charset="UTF-8">          <!-- 한글 설정 -->
  <title>페이지 제목</title>       <!-- 브라우저 탭 이름 -->
  <!-- CSS / JavaScript 파일 포함 위치 -->
</head>
<body>
  <!-- 실제 화면에 출력되는 태그 -->
</body>
</html>
```

> `head` 안의 내용은 브라우저 화면에 출력되지 않음
> `body` 안의 내용만 화면에 출력됨

---

## 3. 태그 속성 기초

```
inline  → 가로 출력 (System.out.print)   예: img, span, b, strong
block   → 세로 출력 (System.out.println) 예: h1~h6, div, p
독립 태그 → 닫는 태그 없음               예: br, hr, img, input
```

### 특수문자
| 코드 | 출력 |
|------|------|
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&nbsp;` | 공백 |
| `&laquo;` | `<<` |
| `&raquo;` | `>>` |

---

## 4. 이미지 출력

```html
<!-- 기본 -->
<img src="m1.jpg" title="설명">

<!-- CSS로 크기 일괄 지정 -->
<style>
  img { width: 130px; height: 250px; }
</style>

<!-- 이미지 없을 때 대체 이미지 -->
<img src="m1.jpg" onerror="this.src='no.png'">

<!-- 외부 URL -->
<img src="https://example.com/image.jpg">
```

> 사용 가능 형식: `gif`, `png`, `jpg`
> 경로: 같은 폴더 `m1.jpg` / 하위 폴더 `images/m1.jpg` / 상위 폴더 `../images/m1.jpg`

---

## 5. 앞으로 배울 태그 목록

```
목록      ul / ol / dl / table
입력창    input / textarea / select / button
분할      div (세로) / span (가로)
이동      a (링크)
시멘틱    header / nav / section / footer / article
```
