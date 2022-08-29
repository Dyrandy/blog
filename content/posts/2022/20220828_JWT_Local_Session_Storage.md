---
title: "JWT, Local & Session Storage"
date: 2022-08-28T22:54:51+09:00
description: "JWT, Local & Session Storage"
categories:
- web
- fundamental
- JWT
tags:
- jwt
- web
- storage
keywords:
- jwt
toc: true
draft: false
---
## Intro

앞 선 글에서, OAuth 2.0에 대해 살펴보았다. OAuth 2.0은 인가를 위임해주기 위해 나타난 Framework고 인증이라는 추가 기능을 지원하기 위해, OpenID Connect가 OAuth 2.0에 추가되었음을 살펴보았다. 그리고 이 인증을 위해 사용되는 토큰이 바로 JWT라는 것을 보았다. 그러면 JWT가 무엇이고, Session과는 무엇이 다르고, 어디에 저장되는지 살펴보겠다.

## What is a JWT, Json Web Token

### Basics

JWT는 Json Web Token의 축약어다. 이름에서 알 수 있듯이 Json형식으로 만들어진 Web Token이다. JWT는 OAuth 2.0에서 ID Token으로 사용되며, OAuth 2.0의 인증 기능이 추가된 OpenID Connect에서 사용된다. 

JWT는 어떻게 생겼으며 어떤 정보가 담겨 있길레 인증을 위해 사용되는 것일까?

### JWT Architecture

아래와 같은 JWT를 예시로 살펴보자.

![/images/20220828/Untitled.png](/images/20220828/Untitled.png)

JWT는 크게 3가지 파트로 구성되어있다. 그리고 그 3가지 파트를 점(.)으로 구분한다. 위 예시에서도 점(.)으로 3 부분이 나뉘어진 것을 확인 할 수 있다. 그 3 부분은 다음과 같다.

1. **Header** : JWT가 암호화된 방법, Token의 종류, 등 Token에 대한 기본적인 정보가 담겨 있는 부분이다. (ALGORITHM & TOKEN TYPE)
2. **Payload** : 실제 데이터가 담기는 부분이다. 계정정보, 이메일, 시간, 등의 정보가 담겨 있을 수 있으며, 실제로 OAuth 2.0에서 Client가 사용하고자 하는 정보가 담겨 있는 부분이다. (요기서 Client는 사용자가 아닌 Token을 요구한 사이트다, OAuth 2.0 참고)
3. **Signature** : 해당 토큰이 위변조 되지 않았음을 증명하기 위한 부분이다. Header 와 Payload 각각 Base64로 인코딩되고 Header에 명시된 Algorithm으로 Secret Key와 함께 암호화 된다. Resource Server는 Signature을 통해 변조되지 않았음을 알 수 있다.

### JWT in Detail

위에서 용어들과 각 부분이 무엇을 나타내는지 살펴보았으니, 실제로 데이터가 어떻게 담기는지 살펴보겠다. 

1. **Header**

```html
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

Header 부분은 JSON 형태의 Data를 Base64로 인코딩한 데이터다. 그러면 반대로 위 데이터를 Base64 디코드한다면 우리는 안에 어떤 데이터가 들어있는지 볼 수 있을 것이다.

![/images/20220828/Untitled%201.png](/images/20220828/Untitled%201.png)

보면 데이터는 JSON 형식으로 표현되어 있다. 먼저 'alg'는 알고리즘을 나타내는 부분은 Signature를 어떤 알고리즘으로 암호화할 것인지 혹은 했는지 나타내는 부분이다. 그리고 'typ'는 해당 토큰이 어떤 종류의 토큰인지 명시한 부분이다. (이 경우 너무나 당연히 JWT다)

1. **Payload**

```html
eyJpZCI6IjEiLCJuYW1lIjoidGVzdDEyMyIsImVtYWlsIjoidGVzdEB0ZXN0LmNvbSIsImlhdCI6MTUxNjIzOTAyMn0
```

Payload 부분 또한 JSON 형태의 Data를 Base64로 인코딩한 것이다. Payload안에는 Client가 사용하고자 하는 사용자에 대한 정보가 담겨있다. 

![/images/20220828/Untitled%202.png](/images/20220828/Untitled%202.png)

위를 보면 index값이 1이고, name은 test123, email은 test@test.com인 것을 알 수 있다. 또한 Payload에는 중요한 정보가 담겨있는데, 바로 iat라는 데이터다. 이 iat는 Issued At의 약자로 발급된 시간을 나타낸다. 이 정보가 왜 들어가는지는 조금 있다가 보도록 하겠다.

1. **Signature**

```html
WCHSnjHKVtCGEs8zwiV69VbCZmH4ZEi8bCoGT4g3kUM
```

Signature은 이전 2부분과는 살짝 다르다. JSON형식을 띄거나, Base64로 인코딩 되어 있지 않다. Signature의 본래 목적은 JWT가 변조 되지 않았음을 증명하는 것이다. 예를 들면 Header와 Payload는 매우 쉽게 디코딩 된다. 안에 내용을 볼 수 있다. 만약 어떤 Client가 이를 이용하여 test123이 아닌 asdf123의 정보를 열람하고 싶다고 가정해보자. 단순히 안에 있는 내용만 바꾸고 요청을 날릴 수 있다. 만약 Signature검증이 없다면, 이를 그대로 수용할 것이고 문제를 일으킬 것이다.

그러나 Signature 검증을 수행하면, Secret Key로 Signing된 이 값이 기존과 다르다는 것을 Resource Server는 알 수 있다. 

![/images/20220828/Untitled%203.png](/images/20220828/Untitled%203.png)

위를 보면 header의 Base64와 payload의 Base64를 점(.)으로 이어붙이고 요기에 Secret Key를 이어붙여 HMacSHA256으로 암호화한 것을 알 수 있다. 

## Difference Between JWT and Session

JWT와 Session의 차이는 살펴보자.

### JWT Characteristics

- Client Side에서 관리되고 저장된다.
- Expiration이 되지 않아 무한히 존재할 수 있다.
- Stateless하다. (세션 관리를 Server에서 하지 않는다는 뜻)

### Session Characteristics

- Server Side에서 관리되고 저장된다.
- Expiration이 존재하여 Server에서 언제든지 삭제하거나 없엘 수 있다.
- Stateful하다. (세션 관리를 Server에서 수행한다는 뜻)

정말 간단하게 비교하면 JWT는 사용자에게 저장되고 Session은 Server에 저장된다는 특징이 있다. 그러면 JWT는 클라이언트 어디에 저장될까?

## Where is JWT Stored

JWT는 Client단에 저장된다는 것을 알았다. 그런데, 이 JWT는 Client의 어디에 저장되는 것일까? 

크게 2 곳에 저장된다고 보면된다.

1. Web Storage (Local or Session Storage)
2. Cookie

Web Storage라는 개념이 좀 생소한 사람들이 있을 것이다. Web Storage라는 것은 그냥 브라우저 안에 있는 작은 저장소다. 이 저장소 안에는 2가지의 저장소가 있는데 JWT가 Web Storage안에 저장될 때 이 두가지 중 하나에 저장된다. (대부분 Local Storage다)

![Chrome Web Storage](/images/20220828/Untitled%204.png)

Chrome Web Storage

### Local Storage

- 오리진이 같은 경우 브라우저에 있는 모든 탭과 창에서 데이터를 공유한다.
- 브라우저 혹은 OS를 다시 시작해도(재부팅) 데이터는 파기되지 않는다.

![첫 텝에서 localStorage에 데이터 생성](/images/20220828/Untitled%205.png)

첫 텝에서 localStorage에 데이터 생성

![첫 탭에 데이터가 존재하는 것을 확인](/images/20220828/Untitled%206.png)

첫 탭에 데이터가 존재하는 것을 확인

![두번째 탭에 똑같은 데이터가 존재하는 것을 확인](/images/20220828/Untitled%207.png)

두번째 탭에 똑같은 데이터가 존재하는 것을 확인

### Session Storage

- 현 탭에서만 데이터가 공유된다 (같은 페이지더라도 다른 탭이면 다른 저장소다.)
- 한 탭에 있는 여러 Iframe들은 서로 데이터를 공유한다.
- 페이지를 새로고침(Refresh)하면 저장된 데이터가 사라지지 않지만, 탭을 닫았다가 키면 그 데이터는 사라진다.

![첫 탭에 Session Storage에 데이터 저장](/images/20220828/Untitled%208.png)

첫 탭에 Session Storage에 데이터 저장

![첫 탭에 데이터가 저장된 것을 확인](/images/20220828/Untitled%209.png)

첫 탭에 데이터가 저장된 것을 확인

![두번째 탭에 데이터가 없음을 확인](/images/20220828/Untitled%2010.png)

두번째 탭에 데이터가 없음을 확인

## 참고

[https://jwt.io/](https://jwt.io/)

[https://jihyun03.tistory.com/52](https://jihyun03.tistory.com/52)

[https://ko.javascript.info/localstorage](https://ko.javascript.info/localstorage)

[https://www.redhat.com/ko/topics/cloud-native-apps/stateful-vs-stateless](https://www.redhat.com/ko/topics/cloud-native-apps/stateful-vs-stateless)

[https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-서비스와-HTTP-및-REST](https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%99%80-HTTP-%EB%B0%8F-REST)