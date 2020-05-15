---
layout: post
title: Node.js + Express + MariaDB(Mysql)로 게시판 만들기 - 0. 개요 / 준비
date: 2020-05-12 23:03:00 +0000
description: # Add post description (optional)
img: ./nodejs/newProject/2020-05-12-newProject1/1.png
tags: [boardMaker]
---

Created: May 12, 2020 9:34 PM

Node.js + Express + MariaDB로 로그인/게시판을 만들고자 했던 시리즈 글이 있다. 물론 지금 블로그에서 흔적을 쉽게 찾을 수 있다. 그런데 오늘 내 블로그를 참고해서 실제로 NodeJS를 공부하시는 분이 있다는걸 알게되었다.

그래서 그 동안 마무리 짓지 않았던 저 시리즈를 일단 게시판만이라도 마무리 해보려고 한다. 거기다가 무책임하게 블로그를 방치했던 날을 반성했다.

기본적인 세팅은 아래 세팅을 따라간다. 이는 전 시리즈와 똑같은 기본세팅이다.

- [Node.js 12.16.1 LTS](https://nodejs.org/ko/)
- express 4.16.1
    - 템플릿 엔진은 pug 사용 예정
- [MariaDB](https://mariadb.org)
- IDE는 Jetbrain사의 Webstorm을 사용한다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/1.png"></center>
<center><small>IDE는 MS사의 Visual Studio Code를 사용해도 무방하다. Notepad++도 가능하다.</small></center>

---

# 0-0. 해당 글을 보기 전 주의사항

---

해당 글은 Node.js, MariaDB, Express 설치 및 세팅을 설명하는 글입니다. 이미 **전 시리즈를 따라하셨거나 이미 깔려있다면 생략해도 되는** 글입니다.

<ins class="kakao_ad_area" style="display:none;" 
 data-ad-unit    = "DAN-u8dziawcgw1g" 
 data-ad-width   = "728" 
 data-ad-height  = "90"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>

### 0-1. MariaDB 설치

---

일단 MariaDB 재단의 사이트에 가서 Stable 버전을 다운받는다. 링크는 [여기](https://downloads.mariadb.org/)를 클릭하면 된다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/2.png"></center>
<center><small>얼마나 큰 차이가 나겠냐 하지만 불안정한 Beta보다는 Stable 버전을 받아주자.</small></center>

초록색 Download 버튼을 누르게 된다면 

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/3.png"></center>

.msi 확장자로 끝나는 파일들이 윈도우 설치파일이다. 내가 32비트 윈도우를 사용한다면 아래 win32.msi를 받아도 되지만 나는 그냥 윈도우 자체를 64비트로 바꿔버리는걸 추천한다.

설치는 평범하다 포트(기본 포트는 3306으로 설정되어있다.)하고 비밀번호만 기억해주자.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/4.png"></center>
<center><small>대략 이런 창이 뜬다. 위 사진과 같이 설정하는 것을 추천한다.</small></center>

설치가 끝나면 **Win + Q키를 눌러주고 `MySQL`을 검색해 준다.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/5.png"></center>

그중 MySQL Client를 실행해준다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/6.png"></center>

위와 같은 창이 나온다면 **설치할 때에 입력했었던 비밀번호를 입력해주자.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/7.png"></center>

그러면 이렇게 접속이 될 것이다. MariaDB는 **MySQL**에 뿌리를 두고 있어서 기본적으로 MySQL과 공유하는 부분이 많다. 그래서 중간중간에 나오는 MySQL에 친숙해지자.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/8.png"></center>

그리고 이제 DB를 사용할 계정을 만들어야 한다. 위와 같이 명령어를 입력하면 된다. 입력하기 귀찮다고 해도 자기 손으로 직접 한번씩 작성하는 것을 추천한다.

<ins class="kakao_ad_area" style="display:none;" 
 data-ad-unit    = "DAN-u8dziawcgw1g" 
 data-ad-width   = "728" 
 data-ad-height  = "90"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>

`CREATE USER '유저이름'@'localhost' IDENTIFIED BY '패스워드';`

유저이름엔 자신이 원하는 이름을 적으면 된다. 해당 예제에선 'pigstew'라는 이름을 사용한다. 또한 우리는 서버를 로컬 환경에서만 돌릴 예정이기에 접속 가능한 주소도 localhost로만 해둔다.

**혹시라도 다른 컴퓨터나 가상환경의 DB를 접속해야할 때는 localhost에 접속하는 PC의 IP를 적어주면 된다. 또한 어디에서나 접근 가능하게 만들고 싶다면 localhost를 %로 바꿔주면 된다.**

또 패스워드는 자신이 원하는 것으로 바꿔준다. **중요한건 다 기억해야한다. 절대로 까먹지 말자.** 이 예제에서는 'password'로 진행한다.

이제 Enter를 누르고 `Query OK, 0 rows affected (0.006 sec)` 라는 문장이 나왔다면 잘 생성 된것이다. 물론 유저를 만들었다고 바로 사용할 수는 없다. 아래 명령어로 권한을 부여해야한다.

`GRANT ALL PRIVILEGES ON *.* TO '유저이름'@'localhost' WITH GRANT OPTION;`

그리고 엔터를 눌러주면 전과 똑같은 문장이 뜬다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/9.png"></center>
<center><small>이런식으로 잘 따라왔다면 문제 없이 설정되고 있는 중이니 걱정하지말자.</small></center>

이제 **Ctrl+C를 눌러 콘솔창을 벗어나주자.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/10.png"></center>

이제 `mysql -u 유저이름 -p`를 입력해서 내가 만든 계정으로 접근해보자. 해당 예제에서 만든 아이디는 pigstew였으므로 `mysql -u pigstew -p`로 접근하겠다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/11.png"></center>

아주 잘 들어와진 모습이다. 이제 마지막 단계니깐 할게 너무 많다고 화내지 말자. 이 계정으로 들어왔으니 이제 DB를 생성해줘야 한다.

`show databases;` 를 입력하면 현재 MariaDB 서버 내에 존재하는 db를 보여준다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/12.png"></center>
<center><small>위에 오타난건 무시해주자.</small></center>

**우리는 간단한 게시판을 만들예정이니 board라는 이름의 DB를 하나 만들어주자.** 만드는 법은 간단하다. `**CREATE DATABASE DB이름`** 을 입력해주자. 그렇다면 우리는 `CREATE DATABASE board;` 라고 입력하면 된다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/13.png"></center>

board라는 DB를 만들어 주고 `show databases;`로 확인한뒤에 `use board;` 를 사용하여 데이터베이스를 만들어줬다. 일단 마리아DB세팅은 여기 까지 한다. 나중에 저 DB안에 테이블을 두개정도를 만들 거지만 나중에 하도록 하고 NodeJS를 설치하러 가자.

혹시라도 미리 테이블을 만들고자 하는 사람들을 위해 대충 아래에 적어두겠다.

```sql
CREATE TABLE `notice` (
	`id` BIGINT(20) NOT NULL AUTO_INCREMENT,
	`title` VARCHAR(300) NULL DEFAULT NULL,
	`contents` LONGTEXT NULL DEFAULT NULL,
	`user` VARCHAR(30) NULL DEFAULT NULL,
	`createdDate` DATE NULL DEFAULT NULL,
	`modifiedDate` DATE NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
);
-- id는 게시글 번호를 나타내고
-- title은 글 제목
-- contents는 글 내용
-- user는 글 작성자
-- Date들은 작성시간, 수정시간을 의미한다.
```

<ins class="kakao_ad_area" style="display:none;" 
 data-ad-unit    = "DAN-u8dziawcgw1g" 
 data-ad-width   = "728" 
 data-ad-height  = "90"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>

### 0-2. Node.js 설치

---

[해당 링크](https://nodejs.org/ko/)를 눌러 LTS버전을 받아주도록 하자

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/14.png"></center>

그냥 예, 다음, 예, 다음을 누르다보면 설치가 끝나있다. 제대로 설치되었는지 보려면 **명령 프롬프트** 창을 띄워서 `node -v`를 입력해보자.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/15.png"></center>

저는 이전에 받아둔 상태라 12.13.1v이 나옵니다. 12.xx.x버전이면 상관없습니다.

### 0-3. WebStorm 설치

---

웹스톰은 JetBrain사에서 만든 IDE로써 **VSCode나 Eclipse같은 제품입니다.** JetBrain사에서 만든 또다른 유명 IDE는 Java IDE인 **IntelliJ**와 안드로이드 어플을 만들 때 사용하는 **Android Studio**가 있습니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/16.png"></center>
<center><small>영롱합니다.</small></center>

[해당 링크](https://www.jetbrains.com/ko-kr/webstorm/)를 통해 들어가면 WebStorm을 다운받을 수 있습니다.

WebStorm을 사용해도 되고 VSCode를 사용해도됩니다. 개인의 취향이니 굳이 강요하진 않으나 이 시리즈에선 WebStrom을 사용합니다.

### 0-4. Express 세팅(WebStorm을 통한)

---

Webstorm을 설치하고 테마 설정등을 하고 나면 아래와 같은 창이 뜹니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/17.png"></center>
<center><small>왼쪽이 지저분한건 꽤나 많은 삽질이...</small></center>

여기서 Create New Project를 선택해줍니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/18.png"></center>

물론 Empty Project를 사용해서 만드는 방법(다른 IDE를 사용하신다면 아래 **0-5. Express 세팅하기**를 참고하시면 됩니다.)이 있지만 편하게 웹스톰을 활용할 수 있습니다.

왼쪽 **Node.js Express App을 선택해주세요.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/19.png"></center>

- Location : 해당 **프로젝트가 존재할 위치**입니다. 편한 위치로 잡아주세요. 저는 편하게 C:\ 바로 밑에 board라는 폴더를 만들어 진행하겠습니다.
- Node InterPreter : Node의 인터프리터로 무엇을 사용할 것인지 묻는 것 입니다. node가 설치되어있다면 자동으로 설정되어있습니다. **굳이 건드실 필요가 없습니다.**
- express-generator : express-generator은 express 프로젝트를 빠르게 만들 수 있게 도와주는 Node.js 모듈입니다. 이 경우엔 npx를 통해서 직접 다운받아 실행시키는 역할을 합니다. **굳이 건드실 필요가 없습니다.**
- View Engine : 렌더링 엔진을 설정합니다. 해당 프로젝트에선 Pug를 사용할 예정입니다. **그대로 둡시다.**
- Stylesheet Engine : 스타일시트로 무엇을 사용할지 묻는 단계입니다. 저의 능력부족으로 plain CSS 밖에 다루지 못하니 **그대로 둡시다.**

딱히 건드릴 건 Location말고 없네요.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/20.png"></center>

이렇게 창이 잘 떴다면 놀랍게도 이게 Express 세팅의 끝입니다. 예전에는 비교적 많은 노력이 필요했지만 express-generator의 등장으로 이제 손쉽게 Express 세팅이 가능합니다.

**거기에 저흰 WebStorm의 도움을 받아 아주아주 쉽게 세팅할 수 있었습니다.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/21.png"></center>
<center>**"근데 잠시만요, WebStorm을 안 쓰면 어떻게 하죠?"**</center>

아...

<ins class="kakao_ad_area" style="display:none;" 
 data-ad-unit    = "DAN-u8dziawcgw1g" 
 data-ad-width   = "728" 
 data-ad-height  = "90"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>

### 0-5. Express 세팅하기

---

**0-4를 통해서 Express를 세팅했다면 해당 부분은 건너 뛰어도 됩니다.**

사실 JetBrain사의 제품은 **유료 버전이 기능이 훨씬 우수하여** 부담이 되는 것이 사실입니다. 만일 학생이라면 학생인증을 받아 JetBrain사의 제품을 사용하는 것을 권장합니다.

그러면 이제 WebStorm 없이 Express-generator만을 이용해서 프로젝트를 만들어 봅시다. npm을 사용하게될 예정입니다. 제 블로그 [해당 글](https://doncolmi.github.io/Node.js-%EA%B3%B5%EB%B6%80(7)/) 에서 npm에 대해 설명하고있으니 참고 바랍니다.

cmd창을 띄워주세요.  npm을 사용하게 됩니다. `npm install express-generator -g` 를 입력해서 express-generator를 전역설치 해줍시다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/22.png"></center>

그리고 우리가 프로젝트를 만들 폴더의 위치는 `C:\board`였습니다. 기억나시죠? cmd창에서 `cd C:\board`를 입력해서 해당 위치로 가주세요.

그리고 `express --view=pug`명령어를 입력 해주세요. **중요한 점은 꼭 cmd의 위치가 C:\board(내가 프로젝트를 만들어야할 위치)이어야 합니다.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/23.png"></center>

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/24.png"></center>

다 된거 같지만 아직 한 단계 남았습니다. 위 cmd 창에서 `npm install`을 눌러준다면 필요한 node의 모듈들이 설치가 됩니다. **꼭 이 단계를 거쳐야 express 세팅이 완료됩니다.**

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/25.png"></center>

이제 express-generator를 이용해 이렇게 express세팅이 완료됩니다. 이제 여기서 몇가지 수정만 한다면 준비단계가 마무리됩니다.

### 0-6. 마무리

---

이제 WebStorm이나 기타 IDE로 해당 프로젝트를 열어주세요.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/26.png"></center>

express-generator으로 만들어진 프로젝트는 아래와 같은 구조를 띄게 됩니다.

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```

하지만 express 공식 사이트에서도 **"생성기에 의해 작성된 앱 구조는 Express 앱을 구조화하는 여러 방법 중 하나에 불과합니다. 이러한 구조를 사용하거나 사용자의 요구사항에 가장 적합하도록 구조를 수정하십시오."**라고 권장하고 있습니다.

저는 실제로 프로젝트를 만질 때에는 위와 같은 구조를 가지지는 않으나 해당 예제는 위와같은 구조로 진행하겠습니다.

추가로 app.js와 www에 관한 내용은 제 블로그에서 [해당 글](https://doncolmi.github.io/Node.js-%EA%B3%B5%EB%B6%80(8)/)을 통해 알 수 있습니다.

이제 app.js에 한번 가주세요.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/27.png"></center>

app.js에 가면 상관없긴 하지만 **var 천국입니다.** 현재 JS에서 권장하는 **let 이나 const를 사용하지 않고 있습니다. 저는 var로 되어있는 모든 것을 const로 바꾸고 진행하겠습니다.** 물론 굳이 안하셔도 되고 지장은 없을겁니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/28.png"></center>
<center><small>어우 깔끔해</small></center>


그리고 bin/www.js도 이와 같은 방법으로 var를 모두 지워주시면 모든 세팅이 끝났습니다.

재미없는 글을 따라오느라 수고 많으셨고 다음 장부터는 실제로 서버를 켜보고 게시판을 만들어보도록 하겠습니다.

<ins class="kakao_ad_area" style="display:none;" 
 data-ad-unit    = "DAN-u8dziawcgw1g" 
 data-ad-width   = "728" 
 data-ad-height  = "90"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>