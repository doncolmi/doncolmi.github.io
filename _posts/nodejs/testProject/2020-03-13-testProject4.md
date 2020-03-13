---
layout: post
title: Node.js + Express + MariaDB 로 로그인/게시판 구현하기(4) - 작성중
date: 2020-03-13 15:40:00 +0000
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

3. [Node.js + Express + MariaDB 로 로그인/게시판 구현하기(3)](https://doncolmi.github.io/testProject2/)

    해당 글에선 crypto를 이용해 비밀번호를 암호화 하는 회원가입을 만들었다.

### 소개

---

node.js와 express를 공부했지만 아직 갈피를 잡지 못했다. 사실 이론만 공부하는건 내 성격이 아니기도 하고 해서 일단 부딪혀가면서 연습하려고 한다.

일단 기본적인 세팅은 아래와 같다.

- [Node.js 12.16.1 LTS](https://nodejs.org/ko/)
- express 4.16.1
    - 템플릿 엔진은 pug 사용 예정
- [MariaDB](https://mariadb.org)
- IDE는 Jetbrain사의 Webstorm을 사용한다.
- **추가 : Redis**

---

이번엔 express-session와 redis를 이용한 로그인을 만들겠다. 추가로 기본적인 세팅에 **Redis**가 들어갔다.

Redis는 NO SQL의 일종이다. NO SQL은  단순 검색이나 추가 작업을 위한 매우 최적화된 키-값(Key-Value) 형태의 저장 공간으로, 지연 시간과 처리율에 관하여 상당한 성능 이익을 내는게 목적인 데이터베이스 형태이다.

redis는 [인메모리 데이터베이스](https://ko.wikipedia.org/wiki/인메모리_데이터베이스) 이다. 디스크에 최적화된 데이터베이스들과는 다르게 **디스크 접근이 메모리 접근보다 느리기 때문이다.** 내부 최적화 알고리즘이 더 단순하며 더 적은 CPU 명령을 실행한다. 여기서 **redis를 세션 스토리지로 사용한다.**

<center><img src="/assets/img/nodejs/testProject/2020-03-13-testProject4/1.png"></center>
<center><small>레디스를 너무 다루기엔...</small></center>

### Redis 설치

---

Redis는 **윈도우용 설치 파일**이 따로 없다. **tar.gz**파일로 unix환경만 지원한다. 하지만 Microsoft Open Tech 그룹에서 64비트 기반으로 포팅해 개발 및 유지보수를 진행했었다. 현재 레디스의 최신 버전은 5.0.8이며 beta로 6.0버전이 나와있다.

**하지만 윈도우는 3.2.100 버전에서 멈춰있다. 어쩔 수 없지만 3.2.100버전으로 예제를 진행하겠다.**

먼저 [이 링크](https://github.com/microsoftarchive/redis/releases)를 타고 들어가 3.2.100 버전을 눌러부고 msi파일을 눌러 다운받아준다.

<center><img src="/assets/img/nodejs/testProject/2020-03-13-testProject4/2.png"></center>
<center><small>저기 빨간줄 보이시죠?</small></center>

딱히 건드릴거 없이 계속 Next만 눌러주다 보면 설치가 된다. 굳이 건드릴게 없다. 포트도 6379를 사용하므로 그대로 둬주길 바란다. 물론 자신이 기억하기 편한 포트가 있다면 바꾸되 **꼭 기억하도록 하자**

설치가 끝났다면 명령 프롬프트창에 들어가 `redis-cli`를 입력해보자.

    Microsoft Windows [Version 10.0.17763.1098]
    (c) 2018 Microsoft Corporation. All rights reserved.
    
    C:\Users\data-24>redis-cli
    127.0.0.1:6379> 

이렇게 127.0.0.1:6379(localhost:port)가 나온다면 설치가 잘 완료된 것이므로 다음으로 넘어가자.

### views 파일 생성 & Redis 연동

---

redis를 연동하기 전에 테스트를 수월하게 하기 위해서 프론트(views)엔드 파일부터 만들고 시작한다. 현재 views와 public의 폴더 구조는 아래와 같다.

    ├views
    │ ├sign
    │   └join.pug
    │ ├error.pug
    │ ├index.pug
    │ └layout.pug
    ├public
    │ └stylesheets
    │   └style.css
    │ └js
    │   └join.js

먼저 **views/index.pug의 내용을 아래와 같이 바꿔주고  js/index.js 를 만들어 아래의 내용과 같이 만들어준다.**

    // index.pug
    
    extends layout
    
    // 로그인이 되어 세션이 존재 할 때 이 페이지가 나오게 된다.
    block content
      // #{name}은 나중에 routes를 통해 넘겨줄 값이다.
      div #{name}님 안녕하세요!
      br
      button#logout 로그아웃
      script(src="/js/index.js")

    // js/index.js
    
    const main = {
        init : function() {
            const _this = this;
            document.querySelector('#logout').onclick = function() {
                _this.logout();
            }
        },
        logout : function() {
            location.href="/logout"; // 로그아웃 버튼을 눌렀을 때에 이동하게 하는 역할
        }
    };
    main.init();

**public/js 아래에 sign이란 디렉터리를 하나 더 두고 그 안으로 join.js를 옮겨준다. 그리고 그 sign 디렉터리 안에 login.js를 만들어주자. 내용은 아래와 같다.**

    const main = {
        init : function() {
            const _this = this;
            document.querySelector('#send').onclick = function() {
                _this.login(); // send 버튼을 누르면 login 로직이
            };
            document.querySelector('#join').onclick = function() {
                _this.join(); // join 버튼을 누르면 join 로직이
            }
        },
        login : function() {
            const data = {
                userid : document.getElementById('userid').value,
                password : document.getElementById('password').value
            } // 아래에서 만들 login.pug에서 가져올 값들이다.
            axios.post("/login", data)
                .then(res => {
    								// 결과값인 res의 data에 count값이 1이면 세션을 등록한다.
                    if(res.data['COUNT(*)'] === 1) {
                        alert("세션 등록해버리기!"); // 알림창을 띄우고
                        location.href="/"; // index로 이동한다.
                    } else {
                        alert("실패!"); // 실패하면 알림창만 뜬다.
                    }
                });
        },
        join : function() {
    				// join 버튼을 누르면 그냥 이동만 한다.
            location.href="/join";
        }
    };
    main.init();

**그리고  views/sign 디렉터리 아래에 login.pug를 만들어주자**

    // views/sign/login.pug
    // 로그인 창이다.
    
    extends ../layout
    
    block content
        input#userid(type="text")
        input#password(type="password")
        button#send 보내기 // 위 js에서 send역할
        button#join 회원가입 // 위 js에서 join역할
        script(src="/js/sign/login.js")

조금 중구난방이었지만 모두 마쳤다면 아래와 같은 구조를 띄어야 한다.

    ├views
    │ ├error.pug
    │ ├index.pug
    │ ├layout.pug
    │ └sign
    │   ├login.pug
    │   └join.pug
    ├public
    │ └stylesheets
    │   └style.css
    │ └js
    │   ├join.js
    │   └sign
    │     ├login.js
    │     └join.js

이제 redis와 관련된 의존성들을 `npm`을 통해 받아보자. 설치해야할 것은 총 세가지다.

`npm install redis connect-redis express-session --save`를 입력해주자

- redis, connect-redis : redis를 사용하기 위해 받는 의존성이다.
- express-session : express에서 session을 관리하기 위한 의존성이다.

일단 **루트 위치(최상단)에 redis.js를 만들어주자**.

    // redis.js
    
    // redis를 위한 의존성
    const redis = require('redis');
    // session을 관리하기 위한 의존성
    const session = require('express-session');
    // redis와 session을 연결하기 위한 의존성으로 인자로 위 session을 받습니다.
    const RedisStore = require('connect-redis')(session);
    
    // 레디스 저장소를 세팅하기 위한 config 값 입니다.
    const redisConfig = {
        "host": "localhost", // 서버가 켜진 주소입니다.
        "port": 6379, // 설치시 적은 포트입니다.
        "prefix": "sid:", // 세션을 넣을시에 세션의 이름 앞에 붙는 문구입니다.
        "db": 0, // 잘모르겟음 ㅎㅎ;
        "client": redis.createClient(6379, 'localhost') 
    		// 저장소를 연결하기 위해 포트, 아이피를 이용하여
    		// client를 생성해 주는 역할입니다.
    };
    
    // 아래 result는 이 js파일을 require하면 넘겨지게 되는 값입니다.
    // session이 넘겨지게 됩니다.
    const result = session({
        store: new RedisStore(redisConfig), // 위에서 설정한 config 값으로 redisStore를 만듭니다.
        secret: 'Hello', // Secret 값으로 자신이 원하는 문자열을 넣어줍시다.
        resave: true,
        saveUninitialized:true,
        cookie: {
            maxAge : 1000 * 60 * 60 
    				// 이 세션이 지속될 시간을 나타냅니다.
    				// 여기선 1000(밀리초=1초) X 60초 X 60분 이므로 총 한시간 동안 유지됩니다.
        }
    });
    
    module.exports = result;

**그 다음은 app.js로 가줍니다. 중요한 점이 있는데 이는 코드 안에 주석으로 기술하겠습니다.**

    // app.js
    
    
    // too many codes...
    
    var app = express();
    
    // 되도록 DB코드들은 DB코드들끼리 보기 쉽도록 같이 둡시다.
    const maria = require('./maria');
    maria.connect(); // mariaDB
    // 여기 아래에 넣어주셔야 합니다.
    const redis = require('./redis'); // 추가한 코드
    app.use(redis); // 추가한 코드
    
    // 밑에 view engine 코드보다 위에 있어야 **오류가 나지 않습니다**.
    // view engine setup
    app.set('views', path.join(__dirname, 'views'));
    app.set('view engine', 'pug');

위와 같이 이전 글에서 mariaDB에 관한 연결 설정을 했던곳과 같이 두도록 합시다. 중요한 점은 **view engine 코드 보다는 위에 있어야 합니다.**

[아직 작성중 입니다.]