+++
date = '2025-11-18T17:24:56+09:00'
draft = true
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
웹 취약점 중 가장 핵심인 **XSS(Cross-Site Scripting)**를 DVWA를 통해 공격/우회 실습을 해보며 분석.

---

### 학습 배경 & 동기

토스(Toss)의 여러 포지션을 살펴보면 공통적으로 다음 역량을 매우 강조한다.

- 웹 서비스 내부 동작에 대한 깊은 이해
- Security mindset + 공격/방어 로직 분석 능력
- 웹 공격 Payload 작성 및 우회 능력
- 브라우저 보안 메커니즘 이해
- 입력 검증/출력 인코딩의 실제 영향 분석

XSS는 웹 취약점의 기초이지만 실제 서비스에서 빈번하게 발생한다. 이를 Low -> Medium -> High 난이도로 직접 분석/우회해 보며,
토스(Toss)가 요구하는 **문제 자체를 깊게 이해하고, 직접 분석해서 해결할 수 있는 능력**을 기르는데 초점을 맞추었다.

---

## 목차

0. 실습 환경 구축
1. XSS (Reflected, Stored, DOM) Low 단계 실습
2. Medium -> High 단계 실습
3. Impossible 코드 분석

---

## 0. 실습 환경 구축

### 실습 환경
현재 나는 **MacBook Air 13**(M3, 2024)에서 **VMware Fusion**(Debian 12.x arm64)을 이용하고 있다.

### 환경 구축
먼저 실습을 하기 위한 환경을 만들어야한다. 먼저 리눅스에 접속한 뒤 업데이트 후 필수 패키지를 설치하였다.

```bash
sudo apt update && sudo apt upgrade -y
```

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

이후 본격적으로 DVWA를 이용한 XSS 실습을 시작하였다.

---

## 1. XSS(Reflected/Stored/DOM) 실습

### Reflected XSS (Level: Low)

입력창에 hello를 입력해보니 ***Hello hello***가 출력되었다.

Burp Suite를 이용하여 Intercept 해보니 

```java
GET /vulnerabilities/xss_r/?name=hello HTTP/1.1
```
**name 파라미터에 삽입된 글씨가 그대로 페이지에 실행되는 구조인걸 확인할 수 있었다.**<br>
따라서 스크립트가 파라미터에 삽입되면 아무 필터링 없이 스크립트가 실행이 되게 된다.

입력창에 가장 기본적인 스크립트를 입력해보았다.

```javascript
<script>alert(1)</script>
```

브라우저에 팝업이 뜨며 성공하였다.

이 또한 Burp Suite로 Intercept 해보니
```java
GET /vulnerabilities/xss_r/?name=<script>alert(1)</script> HTTP/1.1
```
위와 같이 파라미에 삽입된 스크립트가 그대로 페이지에 반사(Reflect)되어 실행되는 구조임을 한 번 더 확인 할 수 있다.

이제 Low단계를 해결했으니 기본적인 스크립트 외의 다른 스크립트도 궁금하여 찾아보았다.
찾아본 스크립트들은 아래와 같다.

#### oneeroor 이벤트 활용 (스크립트가 막힐때)
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

#### HTML 종료 태그 + svg를 이용한 스크립트
```html
"><svg onload=alert('HACKED')>
```

미리 이야기하자면 바로 위의 svg를 이용한 스크립트를 이용하여 모든 레벨의 XSS 실습에 이용하여 공격에 성공하였다.

---

### Stored XSS (Low)

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

이제 스크립트들을 삽입하여 보자 만약 이름과 메시지를 DB에 저장할때 아무 검증이 이루어지지 않으면 취약점이 발생할 것이다. <br>
이를 확인하기 위해 Name에는 'hacker', Message에는 아래와 같은 스크립트를 입력 해보았다.

```javascript
"><svg/onload=alert('HACKED')>
```

결과는 다음과 같이 팝업이 뜨는 걸 확인할 수 있었다. 이를 통해 **아무 검증이 없다**는 것을 알 수 있었다.

![stored_result_1](/img/1week/storedXSS_result_2.png)

---
이번 실습에서도 다른 Payload가 궁금하여 더 찾아보았다.

***Stored XSS는 한 번 DB에 들어가 순간 그 게시판을 이용하는 모든 사람에게 피해가 가는 아주 위험한 취약점이다.***

아래와 같은 스크립트를 이용하여 게시판의 페이지 전체를 바꿔버릴 수도 있다.

```javascript
<script>document.body.innerHTML='Hacked by KU';</script>
```
![stored_result_1](/img/1week/storedXSS_result_3.png)

그러나 위 코드를 삽입하면 다음 스크립트를 실습할 수 없기에 DB에 접속하여 지워줘야한다.

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
---
**HTML 코드를 분석하여 HTML 구조를 파괴하며 스크립트를 삽입**할 수도 있었다.
```html
</textarea><script>alert('HACKED')</script>
```
![stored_result_1](/img/1week/storedXSS_result_4.png)


---
#### Stored XSS 분석
정리하면 Stroed XSS는 공격자가 악성 스크립트를 입력하여 DB에 저장되게 되면 서버는 그걸 읽어서 HTML에 넣어버린다.<br>
따라서 그 악성 스크립트가 들어간 페이지에 들어가는 사람은 전부 감염되는 구조인 것이다.

지속적으로 공격이 가능하고, 관리자까지 공격하여 csrf token, 쿠키 등의 정보를 탈취 할 수도 있다.

Stroed XSS가 실제 해킹에서도 자주 쓰이는 이유가 위와 같다.

---

### DOM XSS (Low)