---
layout: post
title: Node.js + Express + MariaDB(Mysql)로 게시판 만들기 - 2. 게시판 글쓰기 만들기(C)
date: 2020-05-24 18:03:00 +0000
description: # Add post description (optional)
img: ./nodejs/newProject/2020-05-12-newProject1/1.png
tags: [boardMaker]
---

Node.js + Express + MariaDB로 로그인/게시판을 만들고자 했던 시리즈 글이 있다. 물론 지금 블로그에서 흔적을 쉽게 찾을 수 있다. 그런데 오늘 내 블로그를 참고해서 실제로 NodeJS를 공부하시는 분이 있다는걸 알게되었다.

그래서 그 동안 마무리 짓지 않았던 저 시리즈를 일단 게시판만이라도 마무리 해보려고 한다. 거기다가 무책임하게 블로그를 방치했던 날을 반성했다.

기본적인 세팅은 아래 세팅을 따라간다. 이는 전 시리즈와 똑같은 기본세팅이다.

- [Node.js 12.16.1 LTS](https://nodejs.org/ko/)
- express 4.16.1
    - 템플릿 엔진은 pug 사용 예정
- [MariaDB](https://mariadb.org)
- IDE는 Jetbrain사의 Webstorm을 사용한다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/1.png"></center>
<center><small>IDE는 MS사의 Visual Studio Code를 사용해도 무방하다. Notepad++도 가능하다.</small></center>

---

### 2-0. CRUD?

---

게시판을 만들 때 흔히 **CRUD**를 사용한다고 합니다. 아주 기본적이지만 처음 듣는 분들이라면 모르는게 당연합니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/2.png"></center>
<center><small>처음 듣는 이야기는 모르는 이야기 입니다.</small></center>


CRUD는 기본적인 데이터 처리 기능을 말합니다. 그 데이터 처리 기능엔 네가지가 있습니다.

- **Create(생성)**
    - SQL문에서는 INSERT를 사용합니다.
- **Read(읽기)**
    - SQL문에서는 SELECT를 사용합니다.
- **Update(갱신)**
    - SQL문에서는 UPDATE를 사용합니다.
- **Delete(삭제)**
    - SQL문에서는 DELETE를 사용합니다.

대충 보고 눈치채신 분도 있을 겁니다. 이 네가지의 앞글자를 따서 **CRUD**라고 부른다는 것을요. 이는 사용자 인터페이스가 갖추어야할 기본적인 기능을 가리키는 용어 입니다.

CRUD 예제로는 게시판이 최고인거 같습니다. 그래서 대부분의 기초적인 예제들은 게시판 제작이 필히 들어가죠. 게시판에는 게시글의 생성, 읽기, 수정, 삭제 모두 들어가기 때문입니다.

그래서 이번 글에선 저희는 MariaDB를 Node.js Express 프레임워크에 연동해보고 CRUD중 **C인 게시글 생성을 해보도록 하겠습니다.**

### 2-1. MariaDB 세팅 및 연동

---

일단 MariaDB에 게시판 테이블을 만들어야 한다. 분명 이 시리즈 맨 처음에 코드를 올려놓은게 있는데 거기에 깜빡하고 게시글 삭제에 필요한 비밀번호 컬럼을 빼먹었었다. 

일단 맨 처음에 테이블 생성을 안한 분들을 위해 테이블 생성 방법을 설명하겠다.

일단 Win + Q 를 눌러 MySQL Client를 검색하여 맨 위에 나오는 창을 클릭해서 열어주자. 그럼 아래와 같은 창이 뜬다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/3.png"></center>

MariaDB를 설치할 때 작성했던 비밀번호를 적어주자. 그러면 root 계정으로 접근이 되는데 여기서 Ctrl+C를 눌러주자

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/4.png"></center>

그러면 아주 친절하게 MariaDB가 설치되어 있는 경로로 위치가 잡혀있다. 여기서 `mysql -u pigstew -p` 를 입력해주자. 여기서 pigstew가 아닌 맨 처음글에서 만들었던 유저이름으로 하면된다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/5.png"></center>

그러면 위와같은 창이 뜬다 여기서 `show databases;` 를 입력해주자. 그러면 내가 만든 데이터베이스 목록이 뜰텐데 우리는 맨 처음 해당 프로젝트를 위해 `board`라는 이름의 데이터베이스를 만들었다. `use board;` 를 사용해서 해당 데이터베이스를 선택해주자.

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| board              |
| information_schema |
| mysql              |
| news               |
| performance_schema |
| testproject        |
+--------------------+
6 rows in set (0.014 sec)

MariaDB [(none)]> use board;
Database changed
MariaDB [board]>
```

위와같이 터미널에 출력이 되었다면 잘 진행된것이니 걱정하지 말도록 하자. 이제 여기서 `show tables;`를 입력해서 내 DB에 어떤 테이블이 있는지 알아보자.

```
MariaDB [board]> show tables;
+-----------------+
| Tables_in_board |
+-----------------+
| notice          |
+-----------------+
1 row in set (0.002 sec)
```

이제 두 경우로 나뉜다.

- **저는 위 화면처럼 notice라는 테이블이 존재합니다.**

    위에서 설명했듯이 저의 실수로 깜빡하고 하나의 컬럼을 넣지 못했습니다. 이에 따라 테이블을 삭제하고 다시 입력해주시면 됩니다. `ALTER`를 이용해 수정하는 방법이 있지만 여기선 삭제하고 다시 생성하는 과정을 진행하겠습니다.

    테이블을 삭제하는 명령어는 아래와 같습니다.

    `DROP TABLE [테이블명]` 입니다. 저희는 `DROP TABLE notice;`를 통해서 삭제할 수 있습니다.

    그리고 이제 아래 내용을 따라가시면 됩니다.

- **저는 아무 테이블도 존재하지 않습니다.**

    그렇다면 이 상태 그대로 아래 쿼리문을 복사해서 입력해줍니다.

    ```sql
    CREATE TABLE `notice` (
    	`id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    	`title` VARCHAR(300) NULL DEFAULT NULL,
    	`contents` LONGTEXT NULL DEFAULT NULL,
    	`user` VARCHAR(30) NULL DEFAULT NULL,
    	`password` VARCHAR(50) NULL DEFAULT NULL,
    	`createdDate` DATE NULL DEFAULT NULL,
    	`modifiedDate` DATE NULL DEFAULT NULL,
    	PRIMARY KEY (`id`)
    );
    -- id는 게시글 번호를 나타내고
    -- title은 글 제목
    -- contents는 글 내용
    -- user는 글 작성자
    -- password는 글 비밀번호
    -- Date들은 작성시간, 수정시간을 의미한다.
    ```

    복사한 다음 엔터를 누르면 아래와 같은 창이 떠야 합니다.

    ```
    MariaDB [board]> CREATE TABLE `notice` (
        -> `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
        -> `title` VARCHAR(300) NULL DEFAULT NULL,
        -> `contents` LONGTEXT NULL DEFAULT NULL,
        -> `user` VARCHAR(30) NULL DEFAULT NULL,
        -> `password` VARCHAR(50) NULL DEFAULT NULL,
        -> `createdDate` DATE NULL DEFAULT NULL,
        -> `modifiedDate` DATE NULL DEFAULT NULL,
        -> PRIMARY KEY (`id`)
        -> );
    Query OK, 0 rows affected (0.023 sec)

    MariaDB [board]> show tables
        -> ;
    +-----------------+
    | Tables_in_board |
    +-----------------+
    | notice          |
    +-----------------+
    1 row in set (0.001 sec)
    ```

    위 내용은 `show tables;` 로 테이블이 존재하는지 까지 알아본 내용입니다.

다시 복기하겠습니다.

저희는 현재 MariaDB에서 계정 이름은 `pigstew`를 사용하고 있고 비밀번호는 맨 처음 게시글 에서 계정 생성 때 만든 비밀번호입니다. 그리고 DB는 `board`를 사용하고 있고 그 DB안에 `notice`라는 테이블이 있습니다.

- 계정명 : pigstew
    - 비밀번호는 각자 입력하신대로
- DB명 : board
- 테이블명 : notice

이제 해당 DB를 저희 nodejs 프로젝트에 연동하겠습니다.

**mariaDB 연동을 위해서 먼저 npm을 이용해 해당 관련 모듈을 받아야합니다.** 터미널을 띄워서 해당 프로젝트 경로로 가신 뒤에 `npm install mysql` 을 입력해주세요.

```
C:\board>npm install mysql
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.1.3 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ mysql@2.18.1
added 9 packages from 14 contributors and audited 248 packages in 2.914s
found 1 low severity vulnerability
  run `npm audit fix` to fix them, or `npm audit` for details
```

위와 같이 설치가 잘 되었다는 문구를 확인했다면 설치가 잘 된것입니다. 그리고 `package.json`에 들어가 dependencies부분을 확인해주세요.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/6.png"></center>

여기 보면 `mysql`이 추가 되었습니다. 앞으로 npm을 통해 프로젝트에 적용할 모듈들이 있다면 해당 `package.json`을 확인하면 됩니다.

이제 bin 폴더에 `maria.js` 라는 파일을 하나 만들어주세요.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/7.png"></center>
<center><small>예 www.js 가 있는 그 폴더 맞습니다.</small></center>

```jsx
// bin/maria.js

const maria = require('mysql');
// 설치한 모듈인 mysql을 사용하기 위해 불러왔습니다.

const conn = maria.createConnection({
    host:'localhost', // 이건 localhost또는 127.0.0.1 로 설정하시면 됩니다.
    port:3306, // port는 mariadb 설치시에 딱히 건드리게 없다면 해당 포트가 맞습니다.
    user:'pigstew', // 계정이름
    password:'password', // 계정비밀번호
    database:'board' // DB명
});
// 연결하기 전에 정보를 설정해야 합니다.
// 위에서 제가 DB에 관한 내용을 다시 복기한 이유가 있습니다. 잘 적어주셔야 합니다.

module.exports = conn;
// 그리고 해당 js파일이 모듈로 사용될 때 conn을 담습니다.
```

그리고 app.js에 가서 아래와 같이 수정해줍니다.

```jsx
const usersRouter = require('./routes/users');

const app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

// 위와 같은 부분을 아래와 같이 바꿔줍니다.

const usersRouter = require('./routes/users');

const app = express();

const maria = require('./bin/maria'); // 해당부분을
maria.connect(); // 추가해주세요.

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

```

위에서 주의할 점은 view engine setup 부분보다 위에 들어가야 합니다.

### 2-2. 대충 프로젝트의 골격을 짜보자.

---

최근 이 글 작성이 늦어진 것은 학교 개인 프로젝트도 해야하고 AWS 관련해서도 열심히 배우고 있어서 블로그가 잠시 등한시 되었습니다. 잘 따라오고 계시던분들에겐 사과의 말씀드립니다.

그런데 개인 프로젝트를 하다보니 문서화가 중요하다는 사실을 몸으로 깨우쳤습니다. 저희는 간단한 예제이니 거창한 문서화보다는 어떤 주소로 들어가면 어떤 동작을 하게되는지 정하고 가도록 합시다. 

```
GET "/" : 메인 페이지 접속
GET "/board", param(int page) : 게시판 목록 이동
GET "/board/:idx" : 상세 게시글 이동
GET "/board/post" : 게시글 작성 페이지 이동
POST "/board/post", body(board) : 게시글 작성
GET "/board/post/:idx" : 게시글 수정 페이지
PUT "/board/post/:idx", body(board) : 게시글 수정
DELETE "/board/post:idx" : 게시글 삭제
```

위 기능들을 모두 만들었을 때 이 시리즈가 마무리 됩니다. 오늘은 저중에서 게시글 작성 페이지 이동과 게시글 작성을 만들어보겠습니다.

### 2-3. 게시글 작성 페이지로 이동하자.

---

이 글에서 디자인 적인 내용은 전혀 다루지 않을 예정입니다. 기능은 모두 작동하지만 겉모양은 허접할 수 있습니다. 결과물을 보고 절대 실망하지마시고 html, css도 한번 배우셔서 나중에 이쁘게 꾸미는 것 까지 목표로 잡아보시길 바랍니다.

먼저 routes 폴더 아래에 `board.js` 파일을 생성하겠습니다. 그렇다면 현재 구조는 아래와 같아야 합니다.

```
```
├── app.js
├── bin
│   ├── maria.js
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── **board.js <--- 이 부분이 추가되었습니다.**
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```
```

그리고 board.js 부분은 아래와 같이 만들어주세요.

```jsx
// routes/board.js

const express = require('express');
const router = express.Router();
const maria = require('../bin/maria');

/* 게시판 관련 컨트롤러 */
/* "/board" */

module.exports = router;
```

위에 게시판 관련 컨트롤러, "/board" 이런 내용들은 모두 주석이므로 무시하셔도 됩니다. routes 폴더안에 파일을 만들었다면 `app.js`에 가서도 수정해주셔야 합니다. `app.js`로 이동하겠습니다.

```jsx
// app.js

...
const indexRouter = require('./routes/index');
const usersRouter = require('./routes/users');
...

// 위처럼 되어있는 코드를
// 아래처럼 수정해주세요.

...
const indexRouter = require('./routes/index');
const usersRouter = require('./routes/users');
const boardRouter= require('./routes/board');
...
```

```jsx
// app.js

...
app.use('/', indexRouter);
app.use('/users', usersRouter);
...

// 위처럼 되어있는 코드를
// 아래처럼 수정해주세요.

...
app.use('/', indexRouter);
app.use('/users', usersRouter);
app.use('/board', boardRouter);
...
```

위 두 부분을 수정하셔야 router 파일을 제대로 적용할 수 있습니다. **참고로 boardRouter의 기본 URL 은 /board 부터 시작합니다.** 

이제 다시 `router/board.js`로 와서 컨트롤러를 작성해보겠습니다. 일단 "/board" 에 대한 get 메소드를 작성하겠습니다.

```jsx
// routes/board.js

const express = require('express');
const router = express.Router();
const maria = require('../bin/maria');

/* 게시판 관련 컨트롤러 */
/* "/board" */

// 게시판 목록으로 이동
router.get('/', function(req, res, next) {
    res.render('board/list');
});

// 게시판 작성 페이지로 이동
router.get('/post', function(req, res, next) {
    res.render('board/post');
});

module.exports = router;
```

제가 방금 위에서 기본 URL이 /board부터 시작한다고 말했습니다. 즉 router.get의 요청주소를 **코드에선 '/'라고 작성했지만 접속할 때에는 /board로 접속하는 요청을 받아들입니다.**

첫 번째 코드의 뜻은 " **'/'로 요청이 왔다면 views폴더 아래 board폴더 안에 있는 list.pug 파일을 렌더링해서 보여줘라** " 정도가 됩니다.

두 번째 코드의 뜻이 무엇일지는 스스로 생각해보면 좋을거 같습니다.

그런데 저희는 지금 view 폴더 아래에 board 폴더가 없죠? **board폴더는 물론이고 그 안에 list.pug, post.pug 파일도 만들어주세요.**

```java
// list.pug

doctype html
html
    head
        title 게시판 목록
    body
        h1 게시판 목록
        a(href="/board/post") 게시글 작성
```

```java
// post.pug
doctype html
html
    head
        title 게시글 작성
    body
        h1 게시글 작성
        div
            input#name(type="text", placeholder="닉네임")
            br
            input#password(type="password", placeholder="비밀번호")
        div
            input#title(type="text", placeholder="제목")
            br
            textarea#contents(placeholder="내용", rows=5)
        div
            a#write(href="#") 게시글 작성
            br
            a(href="/board") 돌아가기
    script(src="/javascripts/post.js")
```

아직 pug 템플릿의 문법이 어렵다면 제 블로그에 있는 pug 관련 글을 필히 읽어주셔야 합니다.

일단 게시판 목록이지만 이번엔 CRUD 기능중 C만 만들어볼 예정이니 게시글 출력은 다음에 진행하겠습니다. 

자 이렇게까지 하고 모두 저장하신 다음 터미널에 `npm start`를 이용하여 서버를 켜주세요. 그리고 [localhost:3000/board](http://localhost:3000/board) 에 접속을 해보시길 바랍니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/8.png"></center>

이런 사진이 떴다면 성공입니다. 게시글 작성 버튼을 눌러볼까요?

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/9.png"></center>

이렇게 뜬다면 아주 잘된 케이스입니다. 

### 2-4. 게시글 작성은 어떻게 되는건가요

---

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/10.png"></center>

**"그래서 게시글 작성은 어떻게 하는건데요?"**

먼저 우리는 /board/post 페이지에 들어가 **닉네임, 비밀번호, 제목, 내용을 입력해야합니다.**

`post.pug`의 내용을 보시면 아시겠지만 맨 아래에 js파일 하나를 불러옵니다. `public/javascripts/post.js`이죠. 이 파일은 저희가 지금까지 작성했던 nodejs와는 다르게 웹에서 작동하는 js입니다. 제일 큰 차이점으로는 DOM구조가 없고 있고의 차이죠.

"이게 무슨 개소리지" 하지 말고 대충 설명드리자면, 저 /board/post 페이지에서 작성한 내용들을 `public/javascripts/post.js`에서 JSON 형태로 만들어 저희 서버에 넘겨줄 예정입니다.

그러면 그 정보들을 가지고 `express`서버에서 저희가 연동한 MariaDB서버에 그 정보들을 잘 넣어줄 겁니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/11.png"></center>

대충 글 작성과 조회를 파워포인트를 이용해서 대충 보여드렸습니다. 그럼 이제 저희는 무엇을 해야할까요? `post.pug`파일과 함께 제공해야 할 `public/javascripts/post.js`를 작성해야합니다.

### 2-4. post.js를 작성하자

---

`public/javascripts` 안에 `post.js`파일을 만들어주시길 바랍니다.

```jsx
// public/javascripts/post.js

const main = {
    init : function () {
        const _this = this;
    }, 
};

main.init();
```

일단 이렇게 작성해주었습니다. `post.pug`를 보시면 글 작성 버튼의 id값이 `write`로 정해져있습니다. 그럼 write 버튼을 눌렀을 경우에 어떤 동작을 해야하는지에 대한 코드를 써야합니다.

```jsx
// public/javascripts/post.js

const main = {
    init : function () {
        const _this = this;
        document.getElementById('write').onclick = function() {
            _this.write();
        }
    },
    write : function() {
        // 글 작성 로직
    }
};

main.init();
```

글 작성 로직을 사용하기전에 서버와 통신하기 위해서 ajax를 사용할 예정입니다. ajax는 특정한 기술이 아닙니다. 기존의 여러 기술을 사용하는 새로운 접근법입니다. ajax란 **JavaScript를 사용한 비동기 통신, 클라이언트와 서버간에 XML 데이터를 주고받는 기술이라고 할 수 있겠습니다.** 이라고 [해당 블로그에서 설명해주고 있으니 참고](https://coding-factory.tistory.com/143)하시길 바랍니다.

서버와 통신을 하는 방법이라고 인지하시면 될 것 같습니다. ajax통신을 가능케 하는 모듈 중 저희는  `axios`라는 모듈을 사용할 예정입니다.

axios는 node.js에선 `npm install axios` 라는 명령어로 쉽게 설치하고 `require`로 불러올 수 있습니다. 웹 상에서는 cdn이라는 기능을 통해서 사용할 수 있습니다. [cdn이 궁금하다면 해당 링크를 눌러 공부해보세요.](https://goddaehee.tistory.com/173)

axios의 cdn을 사용하고 싶다면 pug 파일 아래에 `<script src="https://unpkg.com/axios/dist/axios.min.js"></script>` 을 넣으면 됩니다. 그럼 `post.pug`로 가서 해당 cdn 링크를 넣어줍시다.

```jsx
// post.pug
doctype html
html
    head
        title 게시글 작성
    body
        h1 게시글 작성
        div
            input#name(type="text", placeholder="닉네임")
            br
            input#password(type="password", placeholder="비밀번호")
        div
            input#title(type="text", placeholder="제목")
            br
            textarea#contents(placeholder="내용", rows=5)
        div
            a#write(href="#") 게시글 작성
            br
            a(href="/board") 돌아가기
    **script(src="https://unpkg.com/axios/dist/axios.min.js")**
    script(src="/javascripts/post.js")
```

보시면 해당 script src는 `public/javascripts/post.js`보다 상단에 있어야 합니다. 그래야 `post.js`에서 `axios`를 사용할 수 있게 됩니다.

다시 `post.js`로 가볼까요? 이제 write라는 함수를 만들어야 합니다. 일단 DOM에 있는 내용들을 JSON형태로 만들어주어야 합니다.

JSON형태는 아래와 같습니다.

```json
{
	"key" : "value",
	"title" : "제목"
}
```

위와 같이 키와 값으로 이루어진 형태입니다. 중요한점은 **키는 중복되면 안되지만 값을 중복되도 됩니다.**

자 이제 `post.js`를 작성해보겠습니다.

```jsx
// post.js
const main = {
    init : function () {
        const _this = this;
        document.getElementById('write').onclick = function() {
            _this.write();
        }
    },
    write : function() {
        const post = {
            "name" : document.getElementById('name').value,
            "password" : document.getElementById('password').value,
            "title" : document.getElementById('title').value,
            "contents" : document.getElementById('contents').value,
            "createdDate" : new Date().toLocaleString(),
            "modifiedDate" : new Date().toLocaleString()
        }
        axios.post("/board/post", post)
            .then(function(result) {
                if(result.data) {
										// 해당 주소에 접속한 결과값이 true이면 /board로 이동
                    location.href = "/board";
                } else {
										// 만약 false라면 오류입니다 라는 문구를 띄워줌.
                    alert("오류입니다.");
                }
            });
    }
};

main.init();
```

write부분을 작성했습니다. 하나하나 짚고 넘어가겠습니다.

- `const post = { ... }`
    - `name, password, title, contents` : 이 부분들은 직접 DOM 객체에서 value값들을 가져옵니다. `getElementById`는 html에서 괄호안의 아이디를 가지고 있는 DOM 객체를 꺼내옵니다.
- `axios.post(url, data)`
    - 보시면 알겠듯 post 통신을 위해 통신할 url 그리고 보낼 data를 썼습니다. 저희는 `"/board/post"`로 통신해야 하고 위에서 만든 post란 데이터를 보냈습니다.
- `.then(function() ~~~~)`
    - `axios`는 비동기 함수라 콜백이 필요합니다. 이 또한 해당 글의 범위에 많이 벗어나고 실제로 잘 설명되어 있는 블로그가 따로 있어 제가 굳이 다루지 않아도 될 내용입니다. [해당 링크를 클릭하시면 비동기에 대해 공부할 수 있습니다.](https://joshua1988.github.io/web-development/javascript/javascript-asynchronous-operation/)
    - 간단히 설명하자면 `axios.post`를 통해 통신을 했습니다. 그 통신이 끝나고 나서 실행할 함수를 then 안에 넣어놓습니다. function에 result라는 인자는 통신의 결과물을 나타냅니다.

잘 이해가 되지 않아도 어떻게 굴러가는지 대충 개념만 파악하시면 됩니다. 이제 `/board/post`에 post로 값을 보내는 함수를 만들었으니 저희 서버쪽에서도 이에 대한 요청을 받아들일 수 있게 추가해야합니다.

`routes/board.js`로 가줍시다.

```jsx
// board.js
const express = require('express');
const router = express.Router();
const maria = require('../bin/maria');

/* 게시판 관련 컨트롤러 */
/* "/board" */

// 게시판 목록으로 이동
router.get('/', function(req, res, next) {
    res.render('board/list');
});

// 게시판 작성 페이지로 이동
router.get('/post', function(req, res, next) {
    res.render('board/post');
});

**// 게시글 작성
router.post('/post', function(req, res, next) {
    try{
        const post = req.body;
        const sql = 'INSERT INTO notice(title, contents, user, password, createdDate, modifiedDate) values ';
        const sqlValue = `("${post.title}", "${post.contents}", "${post.name}", "${post.password}", NOW(), NOW());`;
        maria.query(sql + sqlValue, function(err, rows, fields) {
            if(!err) {
                res.json(true);
            } else {
                console.log(err);
                res.json(false);
            }
        });
    } catch (e) {
        console.log(e);
        res.json(false);
    }
});**

module.exports = router;
```

네 저기 게시글 작성부분을 추가해줘야 합니다.

- `router.post('/post', function (req,res,next) {`
    - 이 부분은 /board/post에 대한 요청이 post통신으로 들어왔을 때를 의미합니다.
- `const post = req.body;`
    - post라는 변수에 req.body를 넣었습니다. post 통신할 때엔 저희가 보낸 데이터들은 모두 body에 묶여서 옵니다. body는 아래와 같은 형태를 띄겠죠.

    ```jsx
    // req.body
    {
      name: 'asdf',
      password: '1234',
      title: 'asdf',
      contents: 'asdf',
      createdDate: '2020. 5. 24. 오후 4:33:34',
      modifiedDate: '2020. 5. 24. 오후 4:33:34'
    }
    ```

    - 그러면 현재 name값을 뽑아내고 싶다면 `req.body.name`을 써서 뽑아낼 수 있겠지만 우리는 post라는 변수에 넣어 사용하도록 합니다.
- `const sql = 'INSERT INTO notice(title, contents, user, password, createdDate, modifiedDate) values';`
    - 이 부분은 sql문을 작성한겁니다. INSERT INTO문은 DB에 데이터를 집어넣을 때 사용하는 sql문입니다. 해당 내용은 noitce라는 테이블의(title컬럼,contents컬럼,user컬럼,password컬럼,createdDate컬럼,modifiedDate컬럼)에 값을 넣겠다는거구요.
    - 원래 INSERT INTO문은 이런 형태입니다. `INSERT INTO 테이블(컬럼1, 컬럼2 ...) values (컬럼1에넣을값, 컬럼2에 넣을값 ...`)
    - 그런데 현재 형태를 보면 values 뒤에 값들이 빠져있죠?
- `const sqlValue = `("${post.title}", "${post.contents}", "${post.name}", "${post.password}", NOW(), NOW());`;`
    - 예 sqlValue라는 변수를 통해 값을 집어넣을 겁니다. 중요한점은 '(작은 따옴표)나 "(큰 따옴표)가 아닌 ```를 사용했습니다. 이는 ES6의 문법인 템플릿 리터럴 입니다.
    - ''이나 ""이 아닌 ``을 사용하면 +나 다른 문법을 사용하지 않고도 **${변수명}**구조로 안에 값을 넣을 수 있습니다. [자세한 내용은 링크를 따라가시면 학습할 수 있습니다.](https://poiemaweb.com/es6-template-literals)
    - 그리고 맨 뒤에 `NOW()`를 사용해서 현재 시간을 createdDate와 modifiedDate에 넣어줍니다. 여기서 `NOW()`는 sql 문법으로 현재 시간을 가져오게 합니다.
- **`maria.query(sql + sqlValue, function(err, rows, fields) {`**
    - 해당 js 파일 상단에 보면 초기에 작성했던 mariadb 설정파일을 require로 가져오는 `const maria = require('../bin/maria');`문을 볼 수 있습니다.
    - 해당 변수를 이용해 처음 인자엔 sql문을 넣고, 그 뒤엔 콜백 함수를 넣었습니다.
- 그 뒤에 if문
    - 만약 err(에러)가 없다면 true를 결과로 내보내주고, 에러가 있으면 에러를 출력하고 false를 출력합니다.
    - 그리고 `res.render`가 아닌 `res.json`은 json값을 내보냅니다. `res.json("안녕하세요")`라고 적었다면 실제로 해당 주소로 접속시 안녕하세요 라는 글자말고 아무것도 나오지 않습니다.
    - 즉 이상태에선 res.json(true)는 에러가 없을시 true를 내보내고 그 반대는 당연히 false를 내보내겠죠.

- 그리고 `try {} catch {}`문구
    - try안의 메서드를 진행하다 오류가 나면 바로 false를 반환하고 해당 오류 내용을 콘솔에 출력해줍니다.
    - 이를 예외 처리라고 합니다. [해당 링크에서 더 자세히 다루고 있습니다.](https://ko.javascript.info/try-catch)

보통 저는 `routes`안에서 끝내지 않고 `service`라는 폴더를 만들어 그 안에 비즈니스 로직을 수행하는 구성을 추가합니다. 하지만 간단한 예제이므로 굳이 `service`를 만들지 않고 `routes`에서 모든 로직을 구성했습니다.

이제 `npm start`로 서버를 켜서 한번 예제를 확인해주러 갑시다. (저희는 nodemon을 통해 자동으로 수정사항이 생기면 서버를 '재시작하므로 아까 켜놓으셨다면 굳이 서버를 끄고 키실 필요 없이 바로 접속하셔도 됩니다.)

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/12.png"></center>

여기서 게시글 작성을 누르면...

<center><img src="/assets/img/nodejs/newProject/2020-05-24-newProject3/13.png"></center>

아무 결과 없이 게시판 목록으로 이동하는 군요. 데이터가 잘 들어갔는지 확인하기 위해 MariaDB 콘솔을 통해 확인합시다.

콘솔을 이용해 DB에 접속하는 방법은 해당 게시글 상단에 있습니다. `use board;` 

```
C:\Program Files\MariaDB 10.4\bin>mysql -u pigstew -p
Enter password: ********
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.4.11-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use board;
Database changed
```

이렇게 해당 과정을 진행하셨다면 `select * from notice;`를 이용하여 notice 테이블 내용을 확인해줍시다.

```
MariaDB [board]> select * from notice;
+----+------------+-------------+--------+----------+-------------+--------------+
| id | title      | contents    | user   | password | createdDate | modifiedDate |
+----+------------+-------------+--------+----------+-------------+--------------+
|  1 | 비밀번호는 | 1234입니다. | 아무개 | 1234     | 2020-05-24  | 2020-05-24   |
+----+------------+-------------+--------+----------+-------------+--------------+
1 row in set (0.001 sec)
```

위 과정을 문제없이 따라오셨다면 해당 정보가 뜰 것입니다.

이렇게 CRUD중 하나인 C를 마무리 지었습니다. 생각보다 결과물이 허접하다고 낙담하지마세요. 해당 시리즈를 모두 따라해 게시판을 만들고 나서 "아 다 했다! 이제 공부 끝" 이 아니라 html,css를 이용해 더 이쁘게 만든다거나 CRUD를 배웠으니 게시판이 아닌 다른 과정에 도전한다던가 하셔야 합니다.

예제로 공부하실 때 그 예제로 다른 형태의 예제를 만드시면 훨씬 도움이 됩니다.

그리고 중간에 제가 설명이 부족해서 진행이 안되거나 이해가 안되는 부분이 있을 겁니다. 그럴 때 블로그 왼쪽 하단에 이메일 주소가 있으니 해당 이메일로 부담없이 질문하시면 성심성의껏 대답해드리겠습니다.

이제 다음은 CRUD중의 R을 진행하겠지?? 하겠지만 아직 C에서 할 게 있긴 합니다. 다음 글에선 유효성 검사와 HeidiSQL을 통해 DB를 쉽게 접근하는 방법에 대해 알아보겠습니다.