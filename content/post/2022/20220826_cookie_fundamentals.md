---
title: "Cookie Fundamentals"
date: 2022-08-26T20:39:54+09:00
description: "쿠키 기초"
categories:
- web
- fundamental
tags:
- cookies
- http
- web
keywords:
- cookie
toc: true
draft: false
---
## HTTP Cookie

브라우저 쿠키, 웹 쿠키, HTTP 쿠키, 등으로 불리는 ‘쿠키' (Cookie)는 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각이다. 브라우저는 그 데이터 조각들을 저장해 놓았다가, 동일한 서버에 재 요청 시 저장된 데이터를 함께 전송한다. 쿠키는 두개의 요청이 같은 브라우저로부터 왔는지 판단할 때 사용된다. 이를 이용하여 사용자의 로그인 상태를 유지할 수 있다. Stateless HTTP 프로토콜에서 상태 정보를 기억시켜주기 때문이다.

(`Stateless` : 컴퓨팅에서 무상태 프로토콜(stateless protocol)은 어떠한 이전 요청과도 무관한 각각의 요청을 독립적인 트랜잭션으로 취급하는 통신 프로토콜로, 통신이 독립적인 쌍의 요청과 응답을 이룰 수 있게 하는 방식이다. 무상태 프로토콜은 서버가 복수의 요청 시간대에 각각의 통신 파트너에 대한 세션 정보나 상태 보관을 요구하지 않는다)

Cookie는 주로 다음의 3가지 목적을 위해 사용된다

- `세션 관리` (Session Management = 로그인 정보, 등)
- `개인화` (Personalization = 웹 페이지 테마, 등)
- `트래킹` (Tracking = 사용자 사용 기록 저장, 등)

각 브라우저들마다 Cookie의 최대 개수 및 최대 크기를 따로 정의하는데 해당 정의는 해마다 조금식 바뀌는 것으로 보인다. 참고 사이트에 의하면 다음과 같이 브라우저 별 Cookie의 최대 개수 및 크기를 정의한다고 한다.

| Browser | Cookie count limit per domain | Total size of cookies (bytes) |
| --- | --- | --- |
| Chrome | 180 | 4096 |
| Firefox | 150 | 4097 |
| Opera | 60 | 4096 |
| Safari | 600 | 4093 |
| Android | 50 | 4096 |
- [https://docs.devexpress.com/AspNet/11912/common-concepts/cookies-support](https://docs.devexpress.com/AspNet/11912/common-concepts/cookies-support)
- [https://www.tutorialspoint.com/What-is-the-maximum-size-of-a-web-browser-s-cookies-value](https://www.tutorialspoint.com/What-is-the-maximum-size-of-a-web-browser-s-cookies-value)

쿠키를 만들 때 다양한 원소들이 필요로하는데, 그중 일부는 다음과 같다.

- Cookie의 이름 (쿠키 식별자)
- Cookie의 값
- Cookie의 유효기간, 브라우저가 얼마나 해당 쿠키를 브라우저에 들고 있을 지 결정하는 시간 (exipration date)
- Cookie의 경로, 쿠키가 어떤 URL 경로에서 유효한지(사용할 수 있는지)를 명시해주는 설정 값이다. 이 경로 외에서는 유효하지 않으며 사용할 수 없다.
- Cookie의 도메인, 어떤 서버 및 도메인에서 해당 쿠키를 사용할 수 있는 지 명시 해준다. 해당 서버 및 도메인 외에서는 사용할 수 없다.
- Secure Connection을 사용할지에 대한 여부. SSL 같은 안전한 암호화 통신에서만 쿠키를 사용할 수 있음을 나타내는 설정 값
- HTTPOnly를 사용할지에 대한 여부. 해당 설정이 걸려 있으면 JavaScript를 통해서 Cookie에 접근 할 수 없다.
- Same-Site 설정. `Lax`, `Strict`, `None`의 설정이 있으며 일부 브라우저에서는 Same-Site가 걸려 있지 않으면 Default로 `Lax` 혹은 `None`을 사용하게 되어 있다.

![Untitled](/images/20220826/Untitled.png)

- 추후에 크롬 Cookie DB의 Column 명을 통해서 더 많은 내용이 있음을 확인할 수 있다.

## 쿠키의 종류

### Session Cookie

in-memory cookie, transient cookie, non-persistent cookie 혹은 temporary cookie 라고도 불리며, 메모리에만 존재하는 쿠키로 쿠키를 만든 서버에서 만료시간이 존재하며 세션 쿠키를 발급받은 세션 (Ex: 브라우저) 종료 시 해당 쿠키를 삭제한다.

어떤 브라우저들은 재시작할 때 세션을 복원해 세션 쿠키가 무기한으로 존재할 수 있다.

해당 쿠키는 사용자의 단말기(브라우저)에 저장되지 않고 Cookie를 발급해준 서버 측에서 저장된다. (Stateful하다)

### Persistent Cookie

File Cookie, 한국어로는 영속 쿠키라고 불리며 해당 쿠키는 `Expires` 속성에 명시된 날짜에 삭제되거나, `Max-Age` 속성에 명시된 기간 이후에 삭제되는 쿠키를 말한다.

```bash
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```

영속 쿠키는 사용자의 단말기에 저장된다. 

- **Question** : 브라우저에서 Session 쿠키와 Persistent 쿠키를 구분하는 방법이 있을까?
    
    ㄴ 브라우저에서 Session 쿠키와 Persistent 쿠키를 구분하는 방법은 `Expires` 속성의 여부에 따라 판단한다고 한다. Session Cookie는 `Expires` 속성을 가질 수 없다.
    
- **Question** : 쿠키는 File 및 브라우저에 저장된다면 어디에 저장되는지? (구체적으로), 메모리에 저장된다면 메모리 어디?
    
    ㄴ 검색한 바로는 Temporary Memory에 저장된다고 한다. 그러나 이게, cache, stack, heap, data, 등의 영역인지 인지하지 못하는 또다른 영역인지 잘 모르겠다. - 추후 분석으로 좀 더 명확한 위치를 알아야 할 것으로 보인다.
    
    ㄴ Persistent Cookie의 경우 Local 컴퓨터에 저장되게 되는데, 이를 확인할려면 다음과 같은 경로에 접근해야한다. `~/Library/Application Support/Google/Chrome/Default` 해당 디렉토리에서 `ls` 명령어를 실행해보면 Local Storage, Session Storage 등 다양한 저장공간도 확인할 수 있다.
    
    ![Untitled](/images/20220826/Untitled%201.png)
    
    또한 files 명령어를 통해 Cookies를 확인해보면 SQLite 형태의 파일로 저장된 것을 확인할 수 있다.
    
    ```bash
    $ file Cookies
    Cookies: SQLite 3.x database, last written using SQLite version 3038005, file counter 96881, database pages 365, 1st free page 11, free pages 204, cookie 0x11, schema 4, UTF-8, version-valid-for 96881
    
    sqlite> .tables
    cookies  meta
    
    sqlite> PRAGMA table_info(cookies);
    0|creation_utc|INTEGER|1||0
    1|host_key|TEXT|1||0
    2|top_frame_site_key|TEXT|1||0
    3|name|TEXT|1||0
    4|value|TEXT|1||0
    5|encrypted_value|BLOB|1||0
    6|path|TEXT|1||0
    7|expires_utc|INTEGER|1||0
    8|is_secure|INTEGER|1||0
    9|is_httponly|INTEGER|1||0
    10|last_access_utc|INTEGER|1||0
    11|has_expires|INTEGER|1||0
    12|is_persistent|INTEGER|1||0
    13|priority|INTEGER|1||0
    14|samesite|INTEGER|1||0
    15|source_scheme|INTEGER|1||0
    16|source_port|INTEGER|1||0
    17|is_same_party|INTEGER|1||0
    18|last_update_utc|INTEGER|1||0
    
    sqlite> select * from cookies where host_key = 'localhost';
    13301250676749963|localhost||Idea-d20d715||xxxxx|/|13616610676749963|0|1|13303134028398267|1|1|1|2|1|63342|0|13301250676749969
    13303134028833694|localhost||JSESSIONID||xxxxx|/|0|0|1|13303134028833694|0|0|1|-1|1|80|0|13303134028833698
    ```
    

위 Cookie Table의 Column 명을 확인해보면, `is_persistent`와 `has_expires`를 통해서 Session Cookie인지, Persistent Cookie인지 판별하는게 아닌가 싶다. (그러나 Session 쿠키는 브라우저의 메모리상에만 존재한다고 했는데, 왜 존재하는거지? - 정확한 이유는 아직 모르겠다)

![Untitled](/images/20220826/Untitled%202.png)

그 이유는 브라우저를 껐다 키지 않아서 였다. 해당 브라우저를 종료 시키고 재시작시 Session이 사라진것을 확인할 수 있었다.

![Untitled](/images/20220826/Untitled%203.png)

```bash
sqlite> select * from cookies where host_key = 'localhost';
13301250676749963|localhost||Idea-d20d715||xxxxxx|/|13616610676749963|0|1|13303751015084115|1|1|1|2|1|63342|0|13301250676749969
```

- !!! 위 기준은 Chrome 브라우저 기준이며, Safari, Firefox는 조금 다를 수 있다. 그리고 SQLite DB에 있는 쿠키의 Value들은 전부 암호화 되어 있다. (양방향 암호화)
- **FireFox 경로** : `~/Library/Application Support/Firefox/Profiles/niftefmu.default-release` 에서 `cookies.sqlite` 파일
    - 참고로 FireFox 쿠키 SQLite DB에는 Session 쿠키가 저장되지 않는다
- Safari 경로 `/Users/dyrandy/Library/Containers/com.apple.Safari/Data/Library/Cookies/Cookies.binarycookies`
    - 사파리 또한 FireFox와 동일하게 세션은 저장되지 않는다.
    - 크롬과 파이어폭스와 달리 SQLite DB 파일이 아닌 Binary 파일이다.
    - 해당 Binary 파일을 풀기 위해 pip 모듈을 사용하면 된다. (참고 : [https://pypi.org/project/binarycookiesreader/](https://pypi.org/project/binarycookiesreader/))

---

## Experiment#1 (Expire 설정)

정말로 `Expire`가 설정되지 않았으면 브라우저가 Session으로 인식할까 테스트해보고자 한다. 먼저 다음과 같은 코드가 있다고 가정해보자.

```jsx
// 환경은 NodeJS
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
    res.cookie("test_cookie", "dyrandy");
    res.send('Hello World!');
})

app.listen(port, () => {
    console.log(`Example app listening on port ${port}`)
})
```

이제 `localhost:3000`에 접속하여 Cookie의 설정 상태를 확인해보면 놀랍게도 다음 이미지와 같이 Session으로 할당된 것을 확인 할 수 있다.

![Untitled](/images/20220826/Untitled%204.png)

```jsx
sqlite> select * from cookies where host_key = '127.0.0.1';
13304010203325805|127.0.0.1||test_cookie||xxxxxxxxxx|/|0|0|0|13304010605994796|0|0|1|-1|1|3000|0|13304010605994799
```

이제 해당 쿠키의 Expire/Max-Age를 설정해보면 어떨까? 다음 코드와 같이 Expire를 추가해보았다.

```jsx
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
    res.cookie("test_cookie", "dyrandy", { expires: new Date(Date.now() + 900000 )});
    res.send('Hello World!');
})

app.listen(port, () => {
    console.log(`Example app listening on port ${port}`)
})
```

![Untitled](/images/20220826/Untitled%205.png)

```jsx
sqlite> select * from cookies where host_key = '127.0.0.1';
13304010203325805|127.0.0.1||test_cookie||xxxxxxxxxxxxx|/|13304011309985761|0|0|13304010409985761|1|1|1|-1|1|3000|0|13304010409985784
```

---

## 쿠키 설정

### Domain 및 Path

`Domain` : 쿠키가 전송되게 될 Host를 명시한다. 이때 호스트가 명시되지 않는다면, Subdomain이 포함되지 않는, 현재 문서 위치의 호스트 일부를 기본값으로 설정한다. 만약 도메인을 명시한다면, Subdomain들은 항상 포함된다. 예를 들면 `Domain=test.com`이 설정되면, 쿠키들은 `sub.test.com`과 같은 subdomain 상에 포함된다.

`Path` : 쿠키 헤더를 전송하기 위하여 요청되는 URL 내에 반드시 존재해야하는 URL 경로다. 

- 참고로 쿠키는 Port Specific하지 않다. 즉, `localhost:8080`에서 특정 쿠키를 만들고 사용한다면, `localhost:8000`에서도 이를 사용할 수 있다. 만약 SetCookie에서 특정 Port를 지정한다면 구분이 가능하다 (위 쿠키 DB의 Column중 16번 Column에 source_port 존재)

### Secure Flag

`Secure` 설정이된 쿠키는 HTTPS 프로토콜 상에서 암호화된 요청일 경우에만 전송된다. 

### HTTPOnly Flag

`HTTPOnly` 설정이된 쿠키는 JavaScript의 Document.Cookie API에 의해 접근할 수 없다. 서버 쪽에서 저장된 세션 쿠키는 JavaScript를 사용할 필요성이 없기 때문에 `HttpOnly` Flag가 설정되도 무방하다.

```bash
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

### SameSite

`SameSite` 쿠키는 쿠키가 Cross-Site 요청과 함께 전송되지 않음을 설정할 수 있다. 이를 통해 CSRF 공격에 대해 보호 방법을 제공하고 있으며, `SameSite`는 여전히 실험단계에 있다고 한다.

`SameSite`에 들어가기 앞서 `SameSite`가 무엇을 의미하는지부터 파악해야한다. 먼저 요기서 “Site”란 Subdomain을 제외 한 최상위 도메인을 일컫는다. 즉, test.example.com에서 example.com을 말하는 것이고 test.example.com과 www.example.com 사이의 요청은 `SameSite` 요청이다. 

그러나 *.github.io 같은 도메인은 서로 다른 “Site”로 본다, 즉, my.github.io와 your.github.io는 서로 다른 “Site”다. 그리고 같은 도메인이지만, http: 및 https: 처럼 다른 Scheme을 사용한다면 다른 사이트로 구분한다고 한다.

또한 SameSite를 들어가기 앞서 `First-Party Cookie`와 `Third-Party Cookie`를 알아야한다. 만약 다음과 같은 `test.com`에 접속해본다고 가정해보자. 해당 `test.com`에서 `example.com`이 제공하는 이미지인 `example.com/image.png`를 사용하고 있다고 가정해보자. 이 경우 브라우저는 `test.com`에 접속해있으면서 `example.com`으로 요청을 보낸다.

```html
<html>
  <head>
    <title>test.com</title>
    <meta property="url" content="http://test.com/" />
  </head>
  <body>
    <img src="http://example.com/image.png" />
  </body>
</html>
```

이 때 만약 `test.com`에 접속해있는 사용자가 `example.com`에 대한 쿠키를 가지고 있다면, image를 갖고오는 요청에서 해당 쿠키가 `example.com`으로 같이 전송된다. 이때 전송되는 `example.com`의 쿠키를 `Third-Party Cookie`라 하며, `test.com`의 쿠키는 `First-Party Cookie`라 부른다.

즉, `Third-Party Cookie`는 사용자가 접속한 페이지와 다른 도메인으로 전송하는 쿠키를 말하고, `First-Party Cookie`는 사용자가 접속한 페이지와 같은 도메인으로 전송되는 쿠키를 말한다.

이때 전송되는 `Third-Party Cookie`에 대하여 제어를 한 것이 바로 `SameSite`이다. 

`SameSite`에는 3가지 설정이 존재한다. `None`, `Strict`, `Lax` 가 존재한다.

- `None` : 해당 값으로 설정된 쿠키의 경우 Cross-Site Request의 경우 항상 전송된다. 즉, `test.com`에서 `example.com`의 이미지로 요청을 날릴때, 혹은 POST같은 요청을 보낼 때 쿠키는 항상 전송된다. `SameSite=None` 설정 시 `Secure Flag`가 필수로 적용되어야 한다고 한다. (크롬 기준)
    
    ![Untitled](/images/20220826/Untitled%206.png)
    
- `Strict` : 가장 강력한 정책으로 어떠한 Cross-Site Request이더라도 `Third-Party Cookie`는 전송되지 않는다.
- `Lax` : Strict 설정과 유사하지만 Strict만큼 강한 정책이 아닌, 상대적으로 느슨한 정책이다. 예를 들면 다음과 같다.
    
    ```text
    A cookie with "SameSite=Lax" will be sent with a same-site request, 
    or a cross-site top-level navigation with a "safe" HTTP method.
    ```
    
    즉, 위 문구를 번역해보자면, SameSite=Lax가 설정된 쿠키는 Same-Site 요청, Top Level Navigation(웹 페이지 이동)과, "안전한" HTTP Method의 경우에만 전송된다는 것이다.
    
    요기서 Top Level Navigation, 즉, 웹 페이지 이동은 <a> 태그에 있는 링크를 클릭하거나, window.location.replace 등으로 자동으로 이동되는 경우, 그리고 302 같은 Redirection을 말한다. <iframe>과 <img>를 삽입하여 발생하는 경우는 포함하지 않는다.
    
    그리고 “안전한" HTTP Method라 하면 GET Method를 일컫는다. POST와 DELETE같은 요청은 서버의 상태를 바꿀 가능성이 있기 때문에 전송되지 않는다고 한다.
    

!!! 참고로 크롬은 `Lax`를 기본설정으로 사용하고 있다. 즉, 예전에는 `SameSite`가 없었을 시 `SameSite=None`이 기본설정이었지만, [2020년 2월 4일 크롬 Version 80 배포](https://www.chromium.org/updates/same-site/)에서 `SameSite`가 설정되지 않으면 `Lax`를 Default 값으로 지정했다.

![Untitled](/images/20220826/Untitled%207.png)

!!! FireFox의 경우 기본적으로 OFF이며, `about:config`에 접속 시 해당 설정을 바꿀 수 있다. (69버전 이후부터)

![Untitled](/images/20220826/Untitled%208.png)

---

## Experiment#2 (Default Lax)

정말로 브라우저 별로 `SameSite`가 걸리지 않는다면 `Lax`로 바뀌는지 및 안바뀌는지 `Chrome`, `Firefox`, `Safari`로 실험 해보겠다.

먼저 다음과 같은 코드가 있다고 가정하자, `Session Cookie`가 만들어질 것이고, 해당 쿠키에는 `SameSite` 설정이 없다.

```jsx
const app = require("express")()

app.get("/", (req, res) => {
    const cookie = "test=dyrandy; Max-Age=9000000;";

    res.setHeader("set-cookie", [cookie])
    res.send("Made Cookie")
})

app.listen(8000, ()=>console.log("listening on port 8000"))
```

### Chrome

먼저 크롬에서 해당 사이트에 접속 해보았다. 쿠키 생성 도메인에 접속하여 `SameSite` 설정을 보았지만, 아무런 설정이 없는 것을 확인 할 수 있다.

![Untitled](/images/20220826/Untitled%209.png)

그리하여 SameSite가 적용이 안되나 싶었지만, SQLite 파일을 통해 쿠키의 설정을 확인해보았다.

```bash
sqlite> select * from cookies where host_key = 'xxx.xxx.xxx.xxx';
13304101943912455|xxx.xxx.xxx.xxx||test||dyrandy|/|0|0|0|13304102248051917|0|0|1|-1|1|8000|0|13304102248051921
```

뒤에서 5번째 Column이 `SameSite` 설정이므로 -1이 값이 된다. 

```python
test = "13304101943912455|xxx.xxx.xxx.xxx||test||dyrandy|/|0|0|0|13304102248051917|0|0|1|-1|1|8000|0|13304102248051921"
val = test.split('|')
print(val[14])
# -1
```

- 참고로, 2 = Strict, 1 = Lax, 0 = None이다.
- 애초에 `SameSite=None`인 쿠키가 `Secure` 옵션 없이 생성되지 않는다.

### Firefox

파이어폭스도 크롬과 동일하게 쿠키 생성 도메인에 접속하여 `SameSite` 설정을 확인해보았다. 그림과 같이 `SameSite=None`이 적용된 것을 확인할 수 있다.

![Untitled](/images/20220826/Untitled%2010.png)

```bash
# /Users/dyrandy/Library/Application Support/Firefox/Profiles/niftefmu.default-release/cookies.sqlite

sqlite> PRAGMA table_info(moz_cookies);
0|id|INTEGER|0||1
1|originAttributes|TEXT|1|''|0
2|name|TEXT|0||0
3|value|TEXT|0||0
4|host|TEXT|0||0
5|path|TEXT|0||0
6|expiry|INTEGER|0||0
7|lastAccessed|INTEGER|0||0
8|creationTime|INTEGER|0||0
9|isSecure|INTEGER|0||0
10|isHttpOnly|INTEGER|0||0
11|inBrowserElement|INTEGER|0|0|0
12|sameSite|INTEGER|0|0|0
13|rawSameSite|INTEGER|0|0|0
14|schemeMap|INTEGER|0|0|0

sqlite> select * from moz_cookies where host="xxx.xxx.xxx.xxx";
130301||test|dyrandy|xxx.xxx.xxx.xxx|/|1660529133|1659629133437204|1659629114551764|0|0|0|1|0|1
```

Python코드를 돌려보면

```python
test = "130301||test|dyrandy|xxx.xxx.xxx.xxx|/|1660529133|1659629133437204|1659629114551764|0|0|0|1|0|1"
val = test.split('|')
print(val[13])
# 0
```

SameSite 설정은 맨뒤에서 3번째, 즉 0이 된다. (요기서 봐야할 것은 `rawSameSite` 설정이다) 

그러면 Lax Cookie는 어떤 값일까?

```python
sqlite> select * from moz_cookies where host="xxx.xxx.xxx.xxx";
130316||test|dyrandy|xxx.xxx.xxx.xxx|/|1660529672|1659629672510695|1659629114551764|0|0|0|1|1|1

test = "130301||test|dyrandy|xxx.xxx.xxx.xxx|/|1660529133|1659629133437204|1659629114551764|0|0|0|1|1|1"
val = test.split('|')
print(val[13])
# 1
```

- !!! 참고 Strict는 2다

### Safari

사파리의 경우는 아래와 같이 `SameSite` 설정이 없으면 아무것도 나오지 않는다. 그리하여 Cookie의 경로에 들어가 직접 Binary Cookie를 읽어보고자 했다.

![Untitled](/images/20220826/Untitled%2011.png)

```python
$ pip install binarycookiesreader

$ bcr -i Cookie.binarycookies

$ cat Cookies.txt | grep xxx.xxx.xxx.xxx
Cookie : test=dyrandy; domain=xxx.xxx.xxx.xxx; path=/; expires=Mon, 01 Jan 2001; Unknown
```

그러나 쿠키는 읽을 수 있으나, `SameSite` 설정에 대한 내용은 없다.

## 참고

- [https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)
- [https://ko.wikipedia.org/wiki/HTTP_쿠키](https://ko.wikipedia.org/wiki/HTTP_%EC%BF%A0%ED%82%A4)
- [https://web.dev/i18n/ko/samesite-cookies-explained/](https://web.dev/i18n/ko/samesite-cookies-explained/)
- [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [https://www.hahwul.com/2020/01/18/samesite-lax/](https://www.hahwul.com/2020/01/18/samesite-lax/)
- [https://docs.devexpress.com/AspNet/11912/common-concepts/cookies-support](https://docs.devexpress.com/AspNet/11912/common-concepts/cookies-support)
- [https://www.whitehatsec.com/glossary/content/persistent-session-cookie](https://www.whitehatsec.com/glossary/content/persistent-session-cookie)
- [https://web.dev/i18n/ko/samesite-cookies-explained/](https://web.dev/i18n/ko/samesite-cookies-explained/)
- [https://seob.dev/posts/브라우저-쿠키와-SameSite-속성/](https://seob.dev/posts/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EC%BF%A0%ED%82%A4%EC%99%80-SameSite-%EC%86%8D%EC%84%B1/)
- [https://www.chromium.org/administrators/policy-list-3/cookie-legacy-samesite-policies/](https://www.chromium.org/administrators/policy-list-3/cookie-legacy-samesite-policies/)