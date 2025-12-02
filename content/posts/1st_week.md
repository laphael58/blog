+++
date = '2025-11-18T17:24:56+09:00'
draft = false
title = '1주차 : 실습 환경 구축'
author = "신승환"
keywords = ["1st", "dvwa", "web"]
ShowReadingTime = false
searchHidden = false
ShowCodeCopyButtons = true
+++

---

### 1주차 학습 목표

토스(Toss)의 Application Security 영역 및 Backend/Platform Security Engineering 직무 역량 기준을 참고하여,
웹 취약점 중 가장 핵심인 **XSS(Cross-Site Scripting)**를 DVWA를 통해 공격/우회 실습을 해보며 분석.<br><br>

---

### 학습 배경 & 동기

토스(Toss)의 여러 포지션을 살펴보면 공통적으로 다음 역량을 매우 강조한다.

- 웹 서비스 내부 동작에 대한 깊은 이해
- Security mindset + 공격/방어 로직 분석 능력
- 웹 공격 Payload 작성 및 우회 능력
- 브라우저 보안 메커니즘 이해
- 입력 검증/출력 인코딩의 실제 영향 분석

XSS는 웹 취약점의 기초이지만 실제 서비스에서 빈번하게 발생한다. 이를 Low -> Medium -> High 난이도로 직접 분석/우회해 보며,
토스(Toss)가 요구하는 **문제 자체를 깊게 이해하고, 직접 분석해서 해결할 수 있는 능력**을 기르는데 초점을 맞추었다.<br><br>

---

## 목차

0. [실습 환경 구축](#0-실습-환경-구축)
   - [실습 환경](#실습-환경)
   - [환경 구축](#환경-구축)
   - [DVWA 설치](#dvwa-설치)
1. [XSS (Reflected, Stored, DOM) Low 단계 실습](#1-xssreflectedstoreddom-실습)
   - [Reflected XSS](#reflected-xss-low)
   - [Stored XSS](#stored-xss-low)
   - [DOM XSS](#dom-xss-low)
2. [Medium -> High 단계 실습](#2-medium---high-단계-실습)
3. Impossible 단계 코드 분석
4. 대응 방안
5. 마무리<br><br>

---

## 0. 실습 환경 구축

### 실습 환경
현재 나는 **MacBook Air 13**(M3, 2024)에서 **VMware Fusion**(Debian 12.x arm64)을 이용하고 있다.

<br>

### 환경 구축
먼저 실습을 하기 위한 환경을 만들어야한다. 먼저 리눅스에 접속한 뒤 업데이트 후 필수 패키지를 설치하였다.

```bash
sudo apt update && sudo apt upgrade -y
```

<br>

#### 오류 발생 & 해결 과정
그런데 터미널에 입력한 후에 아래와 같은 오류가 발생하였다.<br>
오류를 해결하기 위해 자세히 읽어보니 공간이 필요하다고 한다. '공간은 넉넉할텐데 왜 부족하다고 하지' 라는 궁금증이 생겼다.

```bash
Warning: More space needed than available: 1501 MB > 242 MB, 
installation may fail Error: You don't have enough free space in /var/cache/apt/archives/.
```

일단 찾아보니 아래의 코드를 터미널에 입력하면 캐시가 정리된다고 하여 입력해보았다.

```bash
sudo apt clean
sudo apt autoclean -y
```

완료된 것을 확인한 후 다시 업데이트와 업그레이드를 진행하니 똑같은 오류가 발생하였다.<br>
그래서 계속 찾아보았더니 현재 마운트 되지 않은 스토리지를 볼 수 있는 아래의 코드를 입력해보라고 한다.

```bash
lsdlk
```

이를 터미널에 입력한 후 결과를 보니 아래와 같았다.<br>
VMware에서는 40G로 잡혀있지만 Debian은 18G만 사용하고 있어서 공간 부족이 발생한 것이다!

```bash
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  3.5G  0 rom  
nvme0n1     259:0    0   40G  0 disk 
|-nvme0n1p1 259:1    0   16M  0 part 
|-nvme0n1p2 259:2    0  967M  0 part /boot/efi
|-nvme0n1p3 259:3    0   18G  0 part /
-nvme0n1p4 259:4    0    1G  0 part [SWAP]
```

이를 해결하기 위해 Gparted를 이용하여 루트 파티션을 확장하였다.
방법은 아래와 같다.

```bash
sudo apt intsall gparted
```
```bash
gparted
```

Gparted를 실행 시킨 후 SWAP 파티션(nvme0n1p4)를 선택 후 Swap off -><br>
SWAP 파티션을 맨 끝으로 이동 (할당되지 않은 파티션이 루트 파티션 뒤쪽으로 오게) -><br>
루트 파티션을 선택한 후 Redize/Move 공간 확장 -><br>
이후 SWAP 다시 활성화

<br>

#### 오류 해결

이제 다시 업그레이드와 얻데이트를 진행하니 성공적으로 완료되었다.<br>
이후 웹해킹 실습 기본 패키지를 설치하였다.

```bash
sudo apt install -y git curl wget vim build-essential net-tools openssh-server \
    python3 python3-pip python3-venv default-jdk unzip nginx apache2 \
    mariadb-server php php-mysqli php-cli php-curl php-xml php-mbstring \
    nodejs npm docker.io docker-compose nmap tcpdump sqlmap
```

주요 패키지들이 설치가 잘 되었는지 확인하기 위해 아래의 코드를 터미널에 입력하였다.

```bash
python3 --version
node -v
php -v
apache2 -v
mysql --version
docker --version
```

**결과**

```bash
Python 3.13.9
v20.19.5
PHP 8.4.11 (cli) (built: Aug 15 2025 23:56:47) (NTS)
Copyright (c) The PHP Group
Built by Debian
Zend Engine v4.4.11, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.11, Copyright (c), by Zend Technologies
Server version: Apache/2.4.65 (Debian)
Server built:   2025-08-15T08:32:25
mysql from 11.8.3-MariaDB, client 15.2 for debian-linux-gnu (aarch64) using  EditLine wrapper
Docker version 27.5.1+dfsg4, build cab968b3
```

잘 설치가 된 것을 볼 수 있었다.<br>
이후 **Burp Suite를 설치**해야지만 나는 이미 설치되어 있기에 바로 **DVWA를 설치**하였다.

<br>

### DVWA 설치

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo mv DVWA dvwa
```

설치를 완료한 후 **웹서버가 파일을 읽고 쓰기가 가능하게 권한을 설정**해주었다.

```bash
sudo chown -R www-data:www-data /var/www/html/dvwa
sudo chmod -R 755 /var/www/html/dvwa
```

이후 **DB를 생성**하였다.

```bash
sudo mariadb
```

MariaDB 콘솔에 **DB를 설정**하기 위해 코드 입력

```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'dvwapass';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
exit;
```

위 SQL 명령어에 대해서 궁금하여 찾아보았다<br>
2번째 줄은 이름이 dvwa인 새 데이터베이스를 생성하는 구문이다.<br>
3번째 줄은 dvwa라는 사용자 계정을 만드는 것이고,<br>
4번쨰 줄은 dvwa 데이터베이스에 대한 모든 권한을 dvwa계정에 부여하는 구문이다.<br>
dvwa.* 은 dvwa 데이터베이스 안의 모든 테이블을 의미한다.<br>
5번째 줄은 지금까지 변경한 내용을 서버에 즉시 적용하라는 의미의 구문이다.<br>

DB를 생성한 후 **DVWA 설정 파일을 수정**하여야한다.

```bash
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```

아래의 부분만 수정해주었다.

```bash
$_DVWA['db_user'] = 'dvwa';
$_DVWA['db_password'] = 'dvwa123!';
```

수정이 완료 되었으면 **apache를 재시작**해주었다.

```bash
sudo systemctl restart apache2
```

완료한 후 FireFox를 이용하여 **DVWA에 접속**하였다.

```URL
http://localhost/dvwa
```

admin / password 로 로그인을 한 후<br>
Setup -> Create / Reset Database 를 클릭하여 데이터베이스를 생성/초기화 하였다.<br>
이후 Security Level을 'Low'로 설정한 뒤 실습을 진행하였다.<br>
그런데 창을 껐다가 다시 열었더니 Security Level이 'Impossible'로 초기화 되어있어 이를 해결하기 위해 다음과 같이 하였다.

```bash
sudo nano /var/www/html/dvwa/config/config.inc.php
```

파일을 열어 디폴트 값을 'Impossible'에서 'Low'로 변경하였다.

이후 본격적으로 DVWA를 이용한 XSS 실습을 시작하였다.<br><br>

---

## 1. XSS(Reflected/Stored/DOM) 실습


### Reflected XSS (Low)

#### 구조 확인

입력창에 hello를 입력해보니 ***Hello hello***가 출력되었다.

Burp Suite를 이용하여 Intercept 해보니 

```java
GET /vulnerabilities/xss_r/?name=hello HTTP/1.1
```
**name 파라미터에 삽입된 글씨가 그대로 페이지에 실행되는 구조인걸 확인할 수 있었다.**<br>
따라서 스크립트가 파라미터에 삽입되면 아무 필터링 없이 스크립트가 실행이 되게 된다.

<br>

#### 공격 시도

입력창에 가장 기본적인 스크립트를 입력해보았다.

```javascript
<script>alert(1)</script>
```

브라우저에 팝업이 뜨며 성공하였다.

<br>

#### Burp SUite로 분석

이 또한 Burp Suite로 Intercept 해보니
```http
GET /vulnerabilities/xss_r/?name=<script>alert(1)</script> HTTP/1.1
```
위와 같이 파라미에 삽입된 스크립트가 그대로 페이지에 반사(Reflect)되어 실행되는 구조임을 한 번 더 확인 할 수 있다.

<br>

이제 Low단계를 해결했으니 기본적인 Payload 외의 다른 Patload도 궁금하여 찾아보았다.
찾아본 Payload들은 아래와 같다.

#### oneeroor 이벤트 활용 (script가 막힐때)
```javascript
<img src=x onerror=alert('HACKED')>
```

#### 펄터링 테스트용
```javascript
<ScRiPt>alert('HACKED')</ScRiPt>
```

#### HTML 종료 태그로 HTML 구조 파괴
```html
"><h1>Injected</h1>
```

#### HTML 종료 태그 + svg를 이용한 Payload
```html
"><svg onload=alert('HACKED')>
```

미리 이야기하자면 바로 위의 svg를 이용한 Payload를 이용하여 모든 레벨의 XSS 실습에 이용하여 공격에 성공하였다.<br><br>

---

### Stored XSS (Low)

#### 구조 확인

Stored는 이름과 내용을 입력하는 입력창이 있고 Sign Guestbook 버튼을 눌러 DB에 저장하는 방식인 거 같았다.

먼저 이름에 'name', 내용에는 "hello, world!"를 적고 Sign Guestbook 버튼을 클릭해 DB에 저장해보았다.

결과는 다음과 같았다.<br>
![stored_result_1](/img/1week/storedXSS_result_1.png)

**입력을 하면 DB에 저장한 뒤 자동으로 출력하는 구조였다.**

이제 구조를 더 자세히 살펴보기 위해 Name과 Messagge 입력창에 아무거나 막 입력해보니 글자수 제한이 있다는 것을 확인할 수 있었다. <br>
정확히 확인하기 위해 **개발자 도구(F12)를 이용**하여 살펴보니 아래와 같은 코드를 확인 할 수 있었다.

```html
<input name="txtName" type="text" size="30" maxlength="10">

<textarea name="mtxMessage" cols="50" rows="3" maxlength="50"></textarea>
```

- Name : 10자 까지 제한
- Message : 50자 까지 제한

이를 **개발자 도구(F12)를 이용하여 maxlength를 모두 100으로 수정**을 한 뒤에 입력을 해보니 제한이 100자 까지 늘어난 것을 볼 수 있었다. <br>

<br>

#### 공격 시도

이제 Payload들을 삽입하여 보자 만약 이름과 메시지를 DB에 저장할때 아무 검증이 이루어지지 않으면 취약점이 발생할 것이다. <br>
이를 확인하기 위해 Name에는 'hacker', Message에는 아래와 같은 Payload를 입력 해보았다.

```javascript
"><svg/onload=alert('HACKED')>
```

결과는 다음과 같이 팝업이 뜨는 걸 확인할 수 있었다. 이를 통해 **아무 검증이 없다**는 것을 알 수 있었다.

![stored_result_1](/img/1week/storedXSS_result_2.png)

<br>

이번 실습에서도 다른 Payload가 궁금하여 더 찾아보았다.

***Stored XSS는 한 번 DB에 들어가 순간 그 게시판을 이용하는 모든 사람에게 피해가 가는 아주 위험한 취약점이다.***

아래와 같은 스크립트를 이용하여 게시판의 페이지 전체를 바꿔버릴 수도 있다.

```javascript
<script>document.body.innerHTML='Hacked by KU';</script>
```
![stored_result_1](/img/1week/storedXSS_result_3.png)

그러나 위 코드를 삽입하면 다음 Payload를 실습할 수 없기에 DB에 접속하여 지워줘야한다.

```bash
mariadb
```
```sql
USE dvwa;
SELECT * FROM guestbook;
```

스크립트가 들어가 있는 걸 확인 한 뒤 guestbook 테이블에 저장된 내용을 삭제를 한다.
```sql
DELETE  FROM guestbook
```

<br>

**HTML 코드를 분석하여 HTML 구조를 파괴하며 스크립트를 삽입**할 수도 있었다.
```html
</textarea><script>alert('HACKED')</script>
```
![stored_result_1](/img/1week/storedXSS_result_4.png)

<br>

#### Stored XSS 분석
정리하면 Stroed XSS는 공격자가 악성 Payload를 입력하여 DB에 저장되게 되면 서버는 그걸 읽어서 HTML에 넣어버린다.<br>
따라서 그 악성 Payload가 들어간 페이지에 들어가는 사람은 전부 감염되는 구조인 것이다.

지속적으로 공격이 가능하고, 관리자까지 공격하여 CSRF Token, 쿠키 등의 정보를 탈취 할 수도 있다.

Stroed XSS가 실제 해킹에서도 자주 쓰이는 이유가 위와 같다.

<br>

#### Payload 분석 (JavaScript)

XSS(DOM)으로 넘어가기 전에 지금까지 사용하였던 Payload에 대해 궁금하여 Payload가 어떻게 작동하고 JavaScript에 대해 간단히 알아보았다.

1. `<tag>내용</tag>` → HTML 태그
2. `<script> ... </script>` → 자바스크립트 실행 영역
3. `<img ... onerror="JS코드">` → HTML 태그 속성에서 실행되는 자바스크립트
4. `<a href="javascript:JS코드">` → 링크 클릭 시 실행되는 자바스크립트

**브라우저는 "script가 들어있거나 event 핸들러가 들어있으면" 무조건 JS라고 생각하고 실행한다.**

#### script 태그 직접 실행

```js
<script>alert('HACKED')</script>
```

브라우저는 위의 코드를 JS 태그로 인식하여 그대로 실행한다.

#### 이벤트 실행(onerror)

```html
<img src=x onerror=alert('HACKED')>
```

여기서 자바스크립트 부분은 아래와 같다.

```js
alert('HACKED')
```

근데 `<img>` 태그가 이미 HTML이라고 인식되기 떄문에
속성 속에서 자바스크립트를 넣는 것만으로 실행되는 것이다.

#### javascript: 프로토콜

```html
<a href="javascript:alert('HACKED')">click</a>
```

`javascript:` 는 브라우저가 자바스크립트 실행하라는 특수 규칙이다.

<br>

XSS Payload는:

* 브라우저가 스크립트로 인식하는 구조
* 그 안에 자바스크립트 코드
* 실행 시 팝업, 쿠키 탈취 등 공격 수행

이렇게 구성됨을 알 수 있었다.<br><br>

---

### DOM XSS (Low)

DOM XSS는 Reflected/Stored와 아예 동작 방식 자체가 다르다.
서버에서 스크립트를 반사하거나(DB 저장 후 출력) 하는 게 아니라,
**브라우저 내부의 DOM(Document Object Model)을 조작해서 발생하는 취약점**이다.

**서버는 공격 Payload를 전혀 모르는 상태**에서도 클라이언트 단에서 스크립트가 실행될 수 있다.

이게 DOM XSS가 더 위험한 이유다.

<br>

#### DOM 구조 확인

Burp Suite Intercept 결과를 보면 DOM XSS 페이지는 다음과 같은 요청을 보낸다:

```http
GET /vulnerabilities/xss_d/?default=English HTTP/1.1
```

그리고 페이지 아래쪽에는 다음과 같은 JS 코드가 존재한다.

```javascript
var defaultLang = document.getElementById("default").value;
document.getElementById("lang").innerHTML = defaultLang;
```

여기서 핵심은:

* **input 값(default 파라미터)** → **HTML 내부의 element.innerHTML에 직접 삽입된다.**
* 서버는 default 값을 그대로 넘겨주기만 하고
  **실제 DOM 요소 변조는 브라우저가 직접 수행한다.**

**DOM XSS는 서버가 아닌 클라이언트에서 스크립트가 실행되기 때문에 POST/GET 과도 크게 상관이 없다.**

<br>

#### 공격 시도

Reflected XSS처럼 가장 기본적인 Payload부터 넣어보았다.

```javascript
"><svg onload=alert('HACKED')>
```

결과는 성공이었다.

브라우저는 URL 파라미터인 `default=` 값을 가져와 그대로 DOM 요소에 삽입했고,
innerHTML이 그대로 실행되면서 **alert 팝업이 발생**했다.

<br>

#### Burp Suite로 흐름 분석

Intercept된 요청은 단순히 다음과 같았다:

```http
GET /vulnerabilities/xss_d/?default="><svg/onload=alert('DOM')> HTTP/1.1
```

서버 응답에는 위험한 코드가 포함되지 않았다.
하지만 브라우저에 로딩되는 최종 HTML은 아래와 같이 변형된다.

```html
<div id="lang">"><svg onload=alert('HACEKD')></div>
```

➡ 여기서 **innerHTML에 직접 삽입되는 구조** 때문에 브라우저가 JS를 실행한다.

➡ DOM XSS의 근본 원인은 **innerHTML 사용 + 입력 검증 미비**다.

<br>

#### DOM XSS와 Reflected/Stored XSS의 결정적 차이

| 유형        | 스크립트가 실행되는 곳             | 서버가 Payload를 보나? | 지속성           |
| --------- | ------------------------ | ---------------- | ------------- |
| Reflected | 서버가 HTML로 반사             | **본다**           | 1회성           |
| Stored    | DB → HTML로 출력            | **본다 (DB로 저장됨)** | 지속적           |
| DOM       | **브라우저의 JS 코드가 삽입하여 실행** | **안 본다**         | 페이지 구조에 따라 다름 |

➡ DOM XSS는 **서버 단 로그나 필터에서 탐지되지 않아 더욱 위험**하다.

<br>

#### DOM XSS Payload 추가 테스트

추가로 DOM 환경에서도 잘 실행되는 Payload들을 테스트했다.

```html
"><img src=x onerror=alert('HACKED')>
```

또는 DOM 기반에서 자주 쓰는 이벤트 기반 스크립트도 작동한다.

```javascript
" autofocus onfocus=alert('HACKED')>
```

이 구조가 특히 위험한 이유는:

* 사용자가 **클릭하지 않아도**,
* 공격자가 만든 URL을 보거나 자동 리디렉션만 되어도,

Payload가 실행되므로 피싱, 쿠키 탈취에 악용되기 쉽다.

<br>

#### DOM XSS 코드 분석

DOM XSS는 기본적으로 아래와 같은 JS 구문에서 발생한다:

#### innerHTML (위험)

```javascript
element.innerHTML = userInput;
```

#### document.write (매우 위험)

```javascript
document.write(location.hash);
```

#### location.search / hash 값을 직접 파싱하는 경우

```javascript
var lang = location.search.split("=")[1];
document.getElementById("lang").innerHTML = lang;
```

- DOM XSS는 **innerHTML** + **location 파라미터 사용** 조합이 핵심 위험 요소다.

<br>

#### DOM XSS 핵심 정리

> **DOM XSS는 서버 사이드가 아닌 클라이언트 사이드에서 발생한다.**
> 서버는 Payload를 전혀 처리하지 않지만, 브라우저는 DOM을 조작하며 스크립트를 실행한다.

따라서 방어 방법도 크게 다르다:

* `innerHTML` 보다는 `textContent`, `innerText`
* DOMPurify 같은 sanitizer 이용
* location 파라미터 직접 사용 금지
<br><br>

---

## 2. Medium -> High 단계 실습

### Reflected XSS (Medium & High)

#### 구조 분석

구조를 확인하기 위해 Reflected XSS의 모든 단계 소스 코드를 비교 분석 해보았다.

> **Low**
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?>
```

Low 단계의 소스 코드를 살펴보면 `$_GET['name']`값을 아무런 검증 없이 그대로 출력한다. <br>
따라서 `<script>`, `<img>`, `<svg>`등 모든 태그가 실행 가능하다.<br><br>

> **Medium**
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello {$name}</pre>";
}

?>
```

Medium 단계의 소스 코드에서는 `<script>`문자열과 정확하게 일치하는 경우에만 제거를 하는 검증이 하나 추가되었다. <br>
```php
$name = str_replace( '<script>', '', $_GET[ 'name' ] );
```
하지만 이도 매우 취약하다. `<Script>`, `<img>`, `<svg>`등 대문자로 바꾸거나 이벤트 기반의 태그로 너무 쉽게 우회가 가능하다.<br><br>

> **High**
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello {$name}</pre>";
}

?>
```

High 단계의 소스 코드에서는 정규식 기반 필터링이 적용되었다. `<script>`문자열을 `<scriiiipt>`, `<sCrIpT>`와 같은 다양한 형태를 검증하여 제거한다. <br>
```php
preg_replace('/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $input)
```
하지만 이 또한 `<script>`태그만 검증하기 때문에 다른 태그들을 이용하여 우회가 가능하다.

<br>

#### 공격 시도

Payload를 찾아보며 알게된 가장 짧고 간단하지만 `<script>`태그를 사용하지 않고 잘 검증하지 않는 태그를 사용하는 Payload로 공격시도를 해보았다.

```javascript
"><svg onload=alert('HACKED')>
```

Medium & High 단계 모두 공격에 성공하였다.

<br>

#### 분석

단계가 올라갈수록 필터가 강화되는 것처럼 보이지만, 결국 `<script>`태그만 검증하는 매우 취약한 필터링에 머물러 있다.
실제 공격은 `<svg onload>`, `<img onerror>`, 이벤트 기반 Payload등 다양한 벡터를 활용하므로 대부분 간단하게 우회가 가능했다.
이를 통해 **간단한 필터링으로는 XSS를 막을 수 없다"는 점을 확인할 수 있었다.

<br>

---

### Stored XSS (Medium & High)

#### 구조 분석

> **Low**
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = stripslashes( $message );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitize name input
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```



> **Medium**
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = str_replace( '<script>', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```



> **High**
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```



<br>

#### 공격 시도

<br><br>
# Soon...