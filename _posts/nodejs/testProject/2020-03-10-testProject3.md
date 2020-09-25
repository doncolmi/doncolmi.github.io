---
layout: post
title: Node.js + Express + MariaDB 로 로그인/게시판 구현하기(3)
date: 2020-03-10 15:20:00 +0000
description: # Add post description (optional)
img: ./nodejs/testProject/2020-03-08-testProject1/1.png
tags: [nodejs]
---

### 이전 글

---

1. [Node.js + Express + MariaDB 로 로그인/게시판 구현하기(1)](https://doncolmi.github.io/testProject1/)

   해당 글에선 node 설치와 express-generator와 nodemon을 이용한 express 세팅을 했다.

2. [Node.js + Express + MariaDB 로 로그인/게시판 구현하기(2)](https://doncolmi.github.io/testProject2/)

   해당 글에선 MariaDB를 설치하고 해당 DB를 Express 프로젝트에 연동시켰다.

### 소개

---

node.js와 express를 공부했지만 아직 갈피를 잡지 못했다. 사실 이론만 공부하는건 내 성격이 아니기도 하고 해서 일단 부딪혀가면서 연습하려고 한다.

일단 기본적인 세팅은 아래와 같다.

- [Node.js 12.16.1 LTS](https://nodejs.org/ko/)
- express 4.16.1
  - 템플릿 엔진은 pug 사용 예정
- [MariaDB](https://mariadb.org)
- IDE는 Jetbrain사의 Webstorm을 사용한다.

---

이전 글에 이어 테이블 구조를 짜고 로그인을 구현해보자. 이번 글에서는 테이블을 만들고 회원가입까지만 구현해보자. 이 글에선 유효성 까지 적용은 안할 예정이다.

### 테이블 구조

---

일단 회원 테이블을 `user`라고 하고 테이블을 생성하자. 전 글과 다르게 이번 글부턴 MariaDB를 설치하면 같이 설치되는(아마 같이 될걸...? 안되었다면 [이 사이트](https://www.heidisql.com)에 가서 설치하자) **HeidiSQL**을 사용하겠다.

이는 이전에 **'MySQL Front'**로 알려졌었으며 MySQL의 프론트엔드 제품이다. MySQL이 된다는건 MariaDB 또한 사용이 가능하다는 것이다. [HeidiSQL](https://ko.wikipedia.org/wiki/HeidiSQL)의 기능이나 설명이 더 필요하다면 [해당 링크](https://ko.wikipedia.org/wiki/HeidiSQL)를 참고하자.

일단 Win+Q를 눌러 HeidiSQL을 검색해서 실행하자.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/1.png"></center>
<center><small>이거 맞다.</small></center>

그러면 아래와 같은 창이 뜬다. 이미지를 참고하자.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/2.png"></center>

다 설정했다면 열기를 눌러서 접속해주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/3.png"></center>

만약 이런창이 뜬다면 비밀번호를 틀린것이므로 걱정안해도된다. (내가 틀려서 올린게 아니다.)

아마 아무것도 손대지 않았다면 아래와 같은 창이 뜬다.

---

# 주의!

**우리는 전 글에서 testProject라는 데이터베이스를 만들었습니다. 이 데이터베이스에 테이블을 만들어야 합니다.**

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/4.png"></center>

위와 같이 testProject에 오른쪽 마우스를 눌러 아래 과정을 진행해주세요.

아래 사진은 제가 잘못해서 DB고 테이블이고 아무것도 없는 사진이 만들어졌습니다 ㅠㅠ

---

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/5.png"></center>

저기 빨간 네모가 있는 텅 빈 자리에 오른쪽 마우스를 눌러 **새로 생성 → 테이블** 을 선택하자.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/6.png"></center>

이제 이름엔 `user`를 적어주고 코멘트는 안적어도 무방하다. 이제 열 다섯개를 추가할 예정인데 아래와 같이 설정하면 된다.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/7.png"></center>

그리고 이제 `ID`에는 PRIMARY_KEY(기본키)와 AUTO_INCREMENT(자동 증가)를, USERID와 NAME에는 UNIQUE(유일키)를 설정하겠다.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/8.png"></center>

이렇게 해당 열을 오른쪽 마우스로 누른다음 **새 인덱스 생성 → PRIMARY** 를 눌러주면 된다. USERID와 NAME도 똑같이 오른쪽 마우스를 눌러 UNIQUE를 선택해주자!

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/9.png"></center>

그리고 AUTO_INCREMENT는 기본값을 선택하면 이런 창이 뜨는데 기본값 없음 이라고 체크되어있는 값을 AUTO_INCREMENT로 바꿔주자.

추가한 다섯개의 속성들을 아래와 같다.

- ID - 들어오는 데이터들을 구분지어주는 ID값이다. ex) 1,2,3...
- USERID - 유저들이 회원가입시에 적어주는 자신들의 ID값이다.

  ID는 우리가 번호를 매겨주는것 이고, USERID는 유저들이 가입시 치는 아이디이다.

- PASSWORD - 유저의 PASSWORD이다.
- NAME - 유저의 닉네임이다.
- SALT - 이는 비밀번호를 암호화할때 필요한 값이다. 비밀번호를 암호화 하기 위해 해싱 함수를 사용해 비밀번호를 해싱하고 여기에 "소금을 친다(Adding salt)" 라는 방법을 사용한다고 한다. [이 블로그](https://starplatina.tistory.com/entry/비밀번호-해시에-소금치기-바르게-쓰기)를 참고하면 좋다.

**다 설정하고 저장을 누르자. 그럼 아래와 같은 결과가 나온다.**

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/10.png"></center>

**이제 테이블은 구현했고 이제 코드를 짜러 가보자.**

### 로그인 구현

---

    teseapp
    ├bin
    │ └www.js
    ├public
    │ └stylesheets
    │   └style.css
    ├routes
    │ ├index.js
    │ └users.js
    ├views
    │ ├error.pug
    │ ├index.pug
    │ └layout.pug
    ├ app.js
    ├ maria.js // 이건 mariaDB 접속을 위해 이전 글에서 만들었던 js이다.
    ├ package.json
    ├ package-lock.json

이제 최상단에 **model이라는 디렉터리를 그 안에 user라는 디렉터리를 만들자.**

그리고 addUser.js를 생성해주자.

    teseapp
    ├model
    │ └user
    │   └addUser.js
    ├bin
    │ └www.js
    ├public
    │ └stylesheets
    │   └style.css
    ├routes
    │ ├index.js
    │ └users.js
    ├views
    │ ├error.pug
    │ ├index.pug
    │ └layout.pug
    ├ app.js
    ├ maria.js // 이건 mariaDB 접속을 위해 이전 글에서 만들었던 js이다.
    ├ package.json
    ├ package-lock.json

위와 같은 구조가 되면 된다. 이제 addUser.js를 작성하자.

    // addUser.js

    // crypto 는 기본적으로 express 설치시 같이 설치되는 의존성중 하나이다.
    // 비밀번호 암호화를 위해서 이를 불러오자.
    const crypto = require('crypto');
    // 위에서 설명했듯 암호화를 위해 salt를 임의로 생성해준다.
    // 마지막 +""; 는 숫자를 문자열로 만들기 위해 적은것.
    const salt = () => (Math.round((new Date().valueOf() * Math.random()))) + "";
    // getCrypto 라는 함수는 password 와 위에서 만든 salt 를 인자로 받는 함수이다.
    // 둘을 조합해 hash 함수로 암호화 한다.
    const getCrypto = (_salt, password) => (crypto.createHash("sha512").update(password + _salt).digest("hex"));

    class AddUser { // AddUser 는 회원가입시 들어오는 데이터를 클래스화 한 것이다.
        constructor(data) {
            this.salt = salt(); // 위에서 만든 함수로 salt 를 생성한다.
            this.userid = data.userid;
            this.password = getCrypto(this.salt, data.password); // 위에서 만든 함수로 비밀번호를 암호화 한다.
            this.name = data.name;
        }
    }


    // 이제 이 js 파일을 다른 js 에서 불러올 때는
    // data 를 받아 여기서의 addUser 클래스를 사용해
    // 객체를 생성한다.
    module.exports = (data) => new AddUser(data);

그 다음 routes 디렉터리 안에 **sign이라는 디렉터리를 만들고 그 안에 join.js를 생성한다.**

그리고 join.js를 작성하기 전에 **app.js를 가서 내용을 추가해주자.**

    // app.js

    // ...too many codes

    const indexRouter = require('./routes/index');
    const usersRouter = require('./routes/users');
    // 아래 joinRouter를 추가해주자.
    const joinRouter = require('./routes/sign/join');

    // ...too many codes

    app.use(express.static(path.join(__dirname, 'public')));

    app.use('/', indexRouter);
    app.use('/users', usersRouter);
    // 아래 내용을 추가해주자!
    app.use('/join', joinRouter);

    // ...too many codes

그 다음 join.js를 작성하자.

    // routes/sign/join.js

    // 이 파일을 라우터로 사용할 것 이기에 아래 두개를 불러와준다.
    const express = require('express');
    const router = express.Router();

    // 데이터베이스 접속을 위해 maria.js를 불러온다
    const maria = require('../../maria');

    // /join 으로 get 접속하면 views/sign/join.pug 를 렌더링해준다.
    // join.pug 는 추후 접속한다.
    router.get('/', function(req, res, next) {
        res.render('sign/join');
    });

    // /join 으로 post 접속하면 아래와 같은 동작을 수행한다.
    router.post('/', function(req, res, next) {
        // addUser 를 불러온다. req.body 는 axios 를 통해 클라이언트 측에서 넘겨줄 데이터이다.
        const user = require('../../model/user/addUser')(req.body);
        // sql 문은 아래와 같다. ES6의 템플릿 문자열을 사용해서 복잡하게 쓰지 않고 한줄로 적어준다.
        // 테이블의 ID 값은 자동으로 증가(AUTO_INCREMENT)하기에 안적어줘도 된다.
        const sql = `INSERT INTO user(USERID, PASSWORD, NAME, SALT) VALUES  ("${user.userid}","${user.password}","${user.name}",${user.salt})`;

        maria.query(sql, function(err, rows) {
            // 데이터가 잘 들어갔는지 확인을 위해 로그를 찍어준다.
            console.log(rows);
            // 에러가 없다면 "1"을 있다면 "0"을 반환한다.
            // 이는 크게 상관없다.
            if(!err) {res.send("1")} else {res.send("0")};
        })
    });


    module.exports = router;

그 다음 **views 디렉터리 아래에 sign 디렉터리를 만들고 그 안에 join.pug 를 만들어준다. 여기서 ajax 통신을 위해 외부 라이브러리인 axios를 cdn을 통해 가져왔다.**

    // layout은 기존에 있는 걸 불러온것이다.
        extends ../layout

        block content
            input#userid(type="text")
            input#password(type="password")
            input#name(type="text")
            button#send 보내기
    				script(src="https://unpkg.com/axios/dist/axios.min.js")
            script(src="/js/join.js")

이 pug문은 아래와 같다.

그리고 마지막 줄에 스크립트를 "/js/join.js/"를 불러오는데 이를 만들어주자. **public 디렉터리 안에 js 디렉터리를 만들어준 다음 join.js 를 만들어주자.**

    // public/js/join.js

    const main = {
        init : function() {
            const _this = this;
    				// send 버튼을 누르면 아래 join이라는 함수를 실행한다.
            document.querySelector('#send').onclick = function() {
                _this.join();
            }
        },
        join : function() {
    				// 입력한 데이터들을 이용해 json 형태의 data라는 객체를 만든다.
            const data = {
                userid : document.getElementById('userid').value,
                password : document.getElementById('password').value,
                name : document.getElementById('name').value
            }
            console.log(data);
    				// axios를 통해 /join으로 post연결을 한다. 두번째 인자로 data를 넣어 보낸다.
            axios.post("/join", data)
    						// 그다음 then을 통해 콜백함수를 실행한다.
                .then(res => {
                    console.log(res);
    								// 결과가 200 이면서 statusText가 "OK"라면 alert창과 함께
    								// 메인페이지로 이동한다.
                    if(res.status === 200 && res.statusText === "OK") {
                        alert("안녕!");
                        location.href="/";
                    } else {
    										// 실패했다면 alert창만 띄우고 만다.
                        alert("실패!");
                    }
                });
        }


    };
    main.init();

그리고 routes 아래 있는 index.js를 수정해주자.

    // routes/index.js

    var express = require('express');
    var router = express.Router();

    const maria = require('../maria');

    /* GET home page. */
    router.get('/', function(req, res, next) {
    // 전에 만들어놨던 쿼리문을 아래와 같이 바꿔준다.
    // 'select * from test' 를 'select * from user' 로 바꿔주면 된다.
      maria.query('select * from user', function(err, rows, fields) { // 쿼리문을 이용해 데이터를 가져온다.
        if(!err) {
          res.send(rows);
        } else {
          console.log("err : " + err);
          res.send(err);
        }
      });
    });

    module.exports = router;

그리고 터미널을 실행해서 해당 폴더로 간 다음 `npm start`로 서버를 실행시켜주자. 그 다음 [http://localhost:3000/join](http://localhost:3000/join) 으로 가준다. 그러면 아래와 같은 창이 뜬다.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/11.png"></center>

왼쪽부터 첫 번째 칸이 userid, 두번째 칸이 password, 세번째 칸이 name이다. 차례대로 적어주자 그러면 안녕! 이라는 말과 함께 [http://localhost:3000/](http://localhost:3000/) 으로 이동한다.

<center><img src="/assets/img/nodejs/testProject/2020-03-10-testProject3/12.png"></center>

그러면 이렇게 데이터가 들어가 메인화면에 내가 작성했던 데이터가 들어와있는걸 볼 수 있다.

여기까지 회원가입은 완료하였고 다음엔 로그인을 만들어 보도록 하겠다.
