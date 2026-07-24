# XSS (Cross-Site Scripting)

> 웹 페이지에 악성 스크립트를 삽입해 다른 사용자의 브라우저에서 실행되도록 만드는 공격

## 공격 과정

예를 들어 게시판이 사용자 입력을 그대로 출력한다고 가정한다.

```html
<script>
  alert("공격 성공");
</script>
```

서버가 이를 일반 문자열로 처리하지 않고 HTML로 출력하면, 게시글을 본 사용자의 브라우저에서 스크립트가 실행된다.

```text
공격자가 악성 스크립트 입력
    ↓
서버가 필터링 없이 저장 또는 출력
    ↓
사용자가 해당 페이지 접속
    ↓
사용자 브라우저에서 스크립트 실행
```

공격자는 이를 이용해 화면 변조, 피싱 화면 생성, 사용자 권한으로 요청 전송 등을 시도할 수 있다.

## XSS 종류

### 1. Stored XSS

- 악성 스크립트가 DB 등에 저장되는 공격이다.
- 예를 들어 공격자가 게시글에 악성 코드를 등록하면 해당 게시글을 보는 모든 사용자의 브라우저에서 스크립트가 실행될 수 있다.

### 2. Reflected XSS

- URL이나 요청 값에 포함된 악성 코드가 서버 응답에 그대로 반영되는 공격이다.
- `https://example.com/search?q=<악성 코드>`
- 검색어를 안전하게 처리하지 않고 HTML에 그대로 넣으면 스크립트가 실행될 수 있다.

### 3. DOM-based XSS

- 브라우저의 JavaScript가 신뢰할 수 없는 값을 DOM에 삽입하는 과정에서 발생하는 공격이다.

URL이 다음과 같다고 가정한다.

```text
https://example.com/page#<img src=x onerror=alert(1)>
```

이때 `location.hash`는 `#<img src=x onerror=alert(1)>`이다.

JavaScript 코드가 다음과 같이 작성되어 있다면 브라우저는 `location.hash`를 일반 문자열이 아닌 HTML로 해석한다.

```js
result.innerHTML = location.hash;
```

공격자가 URL 뒤에 악성 HTML이나 스크립트를 넣으면 해당 코드가 페이지에서 실행될 수 있다.

```text
공격자가 조작된 URL 생성
→ 사용자가 URL 접속
→ 브라우저 JavaScript가 URL 값을 읽음
→ innerHTML을 사용해 페이지에 삽입
→ 악성 코드 실행
```

## 예방 방법

핵심은 사용자 입력을 그대로 HTML이나 JavaScript 코드로 해석하지 않도록 처리하는 것이다.

### 1. 이스케이핑

사용자 입력의 HTML 특수문자를 일반 문자로 변환하는 방법이다.

```text
& → &amp;
< → &lt;
> → &gt;
" → &quot;
' → &#x27;

<script>alert(1)</script>
→ &lt;script&gt;alert(1)&lt;/script&gt;
```

HTML, HTML 속성, JavaScript, URL 등 출력 위치에 따라 필요한 인코딩 방식이 다르므로 해당 위치에 맞게 처리해야 한다.

### 2. HTML Sanitization

HTML 입력이 필요한 경우 위험한 태그나 속성을 제거하는 방법이다.
`<b>`, `<p>` 같은 태그는 허용하되 `<script>`, `onclick`, `onerror` 등은 제거한다.
이때 DOMPurify와 같은 검증된 라이브러리를 사용한다.

### 3. 안전한 DOM API 사용

```js
element.innerHTML = userInput;
```

보다는 다음과 같이 사용자 입력을 일반 문자열로 처리한다.

```js
element.textContent = userInput;
```

### 4. CSP 적용

`CSP`는 브라우저가 어떤 스크립트를 실행할 수 있는지 제한하는 보안 정책이다.

예를 들어 자기 사이트에서 제공한 스크립트만 실행하도록 설정할 수 있다.

```http
Content-Security-Policy: script-src 'self'
```

악성 스크립트가 HTML에 삽입되더라도 실행을 차단하거나 제한하는 데 도움이 된다. 다만 CSP는 보조 방어 수단이다.
