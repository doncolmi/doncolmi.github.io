---
layout: post
title: Node.js + Express + MariaDB 로 로그인/게시판 구현하기(2)
date: 2020-03-09 10:19:00 +0000
description: # Add post description (optional)
img: ./nodejs/testProject/2020-03-08-testProject1/1.png
tags: [nodejs]
---

### 이전 글

---

1. [Node.js + Express + MariaDB 로 로그인/게시판 구현하기(1)](https://doncolmi.github.io/testProject1/)

    해당 글에선 node 설치와 express-generator와 nodemon을 이용한 express 세팅을 했다.

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

이전 글에 이어 MariaDB를 설치해 현재 express 프로젝트에 데이터베이스를 연동시키는 작업까지 진행하겠다.

## MariaDB 설치

---

먼저 MariaDB는 이미 유명하지만 혹시 모를 사람들을 위해서 복습하는겸 적어두고 가려한다. MariaDB는 **오픈소스**의 관계형 데이터베이스 관리 시스템(RDBMS)이다. 

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/1.png"></center>

**RDBMS**란 데이터를 관계로서 표현하는, 행과 열의 집합으로 구성된 테이블의 묶음 형식으로 데이터를 제공하는 데이터베이스 시스템이다. RDBMS 말고도 NOSQL(Not Only SQL/Non relational Database)이 있다. RDBMS와 NOSQL을 설명하면 이 글의 요지와 많이 벗어나니 여기 까지만 설명하겠다.

아무튼 MariaDB는 MySQL이 오라클의 소유가 되면서 불확실한 라이센스 상태에 반발해 만들어졌다. MySQL의 포크판에선 가장 인기가 많은데 이 MariaDB이다. 소스가 소스인지라 MySQL과 거의 비슷하다.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/2.png"></center>
<center><small>오죽하면 이런 짤이...</small></center>

**이제 본격적으로 설치에 들어가자.** 일단 [MariaDB 공식 사이트](https://mariadb.org)에 접속하자z

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/3.png"></center>


여기서 Download를 눌러주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/4.png"></center>
<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/5.png"></center>

간단히 Express를 익히고자 하는 프로젝트이므로 굳이 불안정한 Beta 버전 보단 Stable 버전인 10.4를 다운받아주자. **버전은 크게 상관없으니 대충 가장 Stable이라 나와있는 가장 최신버전을 받아주자.**

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/6.png"></center>

아마 대충 다들 운영체제는 **윈도우 64비트를 쓰고 있을테니 win64.msi**라고 나와있는 파일을 눌러주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/7.png"></center>
<center><small>그리고 대충 열어주자.</small></center>

설치창이 뜨면 계속 Next를 누르며 넘기다가 아래와 같은 창이 뜰 때 **집중하자.**

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/8.png"></center>

왼쪽 화면은 비밀번호를 입력하는 창이다 꼭 모두 체크해주고(맨 아래 **Use UTF8 as default...**는 UTF8설정인데 체크해주자) 비밀번호를 입력해주자.

그리고 오른쪽은 아마 체크가 이미 되어있을건데 되도록 건드리지 말고 Next를 눌러 넘어가자 나머지는 이제 Next만 누르다가 Install을 누르게 되면 설치가 진행된다. 본 프로젝트 진행을 위해 비밀번호를 짧게 만들었다.

자 이제 **Win + Q 키를 누르고 `MySQL`을 검색해보자.**

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/9.png"></center>

이런 터미널같은 앱이 뜰텐데 눌러주자. 그럼 다짜고짜 검은창과 함께 아래와 같이 뜬다.

    Enter password:

그럼 여기에 설치할 때 적었던 비밀번호를 적어주면 된다. 그리고 엔터를 누르면 아래와 같이 root계정으로 MariaDB 서버에 접속된다.

    Enter password: ******
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 9
    Server version: 10.4.12-MariaDB mariadb.org binary distribution
    
    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]>

여기까지 했다면 성공한거다. 왜 MariaDB를 설치했는데 MySQL 콘솔창이 나오냐? 라고 할 수 있는데 이는 MariaDB가 MySQL의 소스를 기반으로 뒀기 때문이다. ~~프로젝트를 진행하면서 mariadb인데도 mysql 기반 노드를 사용할 때가 있다...~~

root 계정에서 진행해도 되지만 MariaDB에 친숙해질겸 계정을 만들고 그 안에 데이터베이스를 생성해보도록 하자. 아래와 같은 명령어를 입력해주자

`CREATE USER '유저이름'@'localhost' IDENTIFIED BY '패스워드';`

유저이름과 패스워드는 자신이 원하는 문구를 집어넣자. 또 localhost말고 %를 넣어도 되는데 자신의 컴퓨터(로컬)에서만 테스트할 예정이기에 localhost로 만들어주자.

    MariaDB [(none)]> CREATE USER 'limc'@'localhost' IDENTIFIED BY 'data19';
    Query OK, 0 rows affected (0.006 sec)

아주 잘 만들어진다. 그리고 아래 명령어로 권한을 주자.

`GRANT ALL PRIVILEGES ON *.* TO '유저이름'@'localhost' WITH GRANT OPTION;`

    MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'limc'@'localhost' WITH GRANT OPTION;
    Query OK, 0 rows affected (0.005 sec)

이제 Ctrl+C를 눌러서 mysql 밖으로 나와주자

    MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'limc'@'localhost' WITH GRANT OPTION;
    Query OK, 0 rows affected (0.005 sec)
    
    MariaDB [(none)]> Ctrl-C -- exit!
    Bye
    
    C:\Program Files\MariaDB 10.4\bin>

이제 `mysql -u 유저이름 -p`를 입력해 방금 만든 계정으로 접근하자. 

    C:\Program Files\MariaDB 10.4\bin>mysql -u limc -p
    Enter password: ******
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 13
    Server version: 10.4.12-MariaDB mariadb.org binary distribution
    
    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]>

나의 경우엔 계정이름은 limc이고 비밀번호는 data19로 만들었기에 위와 같이 접근했다. 일단 우리는 현재

- MariaDB을 설치했고
- MariaDB의 root계정 외에 다른 계정을 하나 만들었다.

이제 마지막 단계로

- 생성한 계정에 DB를 만들어주자.

DB를 만들어야 그 안에 테이블을 만들어 데이터를 집어 넣을 수 있다.

`CREATE DATABASE DB이름;`을 입력하면 된다. DB이름은 자신이 원하는 이름으로 해도 된다. **하지만 필히 기억하자. 이 DB이름을 Express 프로젝트 안에서 입력해야한다.**

그 다음 `user DB이름;`을 통해 데이터베이스에 접속해준다.

    MariaDB [(none)]> create database testProject;
    Query OK, 1 row affected (0.002 sec)
    
    MariaDB [(none)]> use testProject;
    Database changed
    MariaDB [testProject]>

이렇게 데이터베이스에 접속하게되면 (none)이었던게 접속한 DB의 이름으로 바뀐다. 이제 기본 SQL 문장들을 이용해 테이블을 만들고 그 안에 연동이 되었을 때 테스트를 하기 위한 데이터를 넣을 것이다. 기본 SQL이 궁금하다면 [이 링크](https://mariadb.com/kb/ko/basic-sql-statements/)에 가면 알 수 있다.

`CREATE TABLE test ( id INT PRIMARY KEY, name VARCHAR(20) );`

이렇게 입력해주자. int 값이 들어가는 id와 String(엄밀히 말하자면 VARCHAR)값이 들어가는 name 속성을 가진 test라는 이름의 테이블을 만든 것이다.

`INSERT INTO test VALUES ( 1, 'LimC' );`

이제 그 test라는 테이블에 id가 1이고 name이 'LimC'인 행을 집어넣었다.

잘 들어갔는지는 `SELECT * FROM test;`를 통해 알 수 있다.

    MariaDB [testProject]> CREATE TABLE test ( id INT PRIMARY KEY, name VARCHAR(20) );
    Query OK, 0 rows affected (0.023 sec)
    
    MariaDB [testProject]> INSERT INTO test VALUES ( 1, 'LimC' );
    Query OK, 1 row affected (0.013 sec)
    
    MariaDB [testProject]> SELECT * FROM test;
    +----+------+
    | id | name |
    +----+------+
    |  1 | LimC |
    +----+------+
    1 row in set (0.001 sec)

이러면 **지루하고 지루했던 MariaDB세팅은 끝냈다고 볼 수 있다. 이제 웹스톰으로 넘어가 프로젝트를 보자.**

<center>
<ins class="kakao_ad_area" style="display:none; margin-top: 15px;" 
 data-ad-unit    = "DAN-1iykkck0nlqnp" 
 data-ad-width   = "250" 
 data-ad-height  = "250"></ins> 
<script type="text/javascript" src="//t1.daumcdn.net/kas/static/ba.min.js" async></script>
</center>

### Express와 MariaDB 연동

---

제일 먼저 할일은 프로젝트 경로에 가서 npm을 이용해 mysql 커넥터를 설치해야한다. 

    C:\Users\data-24>cd C:\nodejs\testProject
    
    C:\nodejs\testProject>npm install mysql --save
    + mysql@2.18.1
    added 9 packages from 14 contributors and audited 260 packages in 1.878s
    
    2 packages are looking for funding
      run `npm fund` for details
    
    found 1 low severity vulnerability
      run `npm audit fix` to fix them, or `npm audit` for details
    
    C:\nodejs\testProject>

명령 프롬프트를 켜서 해당 경로로 가준 후에 `npm install mysql --save`를 입력해 다운받아주자, 그 다음 해당 프로젝트의 `package.json`으로 가서 `dependencies` 에 mysql이 있는지 확인해주자

    {
      "name": "testapp",
      "version": "0.0.0",
      "private": true,
      "scripts": {
        "start": "nodemon ./bin/www"
      },
      "dependencies": {
        "cookie-parser": "~1.4.4",
        "debug": "~2.6.9",
        "express": "~4.16.1",
        "http-errors": "~1.6.3",
        "morgan": "~1.9.1",
        "mysql": "^2.18.1",
        "pug": "2.0.0-beta11"
      }
    }

일단 가장 상단 위치(루트 위치)에 `maria.js`파일을 만들어주자. 그리고 아래와 같이 내용을 넣어준다.

    const maria = require('mysql');
    const conn = maria.createConnection({
        host:'localhost',
        port:3306,
        user:'limc',
        password:'data19',
        database:'testProject'
    });
    module.exports = conn;

- `const maria = require('mysql');`

    mysql 커넥터를 사용하기 위해 이를 호출해주었다.

- `const conn = maria.createConnection`

    MariaDB에 접속하기 위해 커넥션을 만들어주는 부분이다.

    - `host:'localhost'` : 접속할 ip를 적어준다. 여기선 localhost라고 적어주자
    - `port:3306` : 접속할 포트도 적어줘야한다. 설치할 때 설정했던 3306을 적어주자.
    - 그 아래는 만들었던 user, password, database를 기억해 적어주자.

위처럼 적었다면 이제 `app.js`로 가주자. 

    // app.js
    
    // too many codes...
    
    var app = express();
    
    const maria = require('./maria');
    maria.connect();
    
    // too many codes...

이렇게 코드를 넣어주자. 아무곳에나 넣어줘도 되지만 `var app = express();` 아래에 넣었다. 이제 `routes/index.js`로 가보자.

기존 코드는 아래와 같다.

    var express = require('express');
    var router = express.Router();
    
    /* GET home page. */
    router.get('/', function(req, res, next) {
      res.render('index', { title: 'Express' });
    });
    
    module.exports = router;
    // 기존 코드들

아래와 같이 수정해주자.

    var express = require('express');
    var router = express.Router();
    
    const maria = require('../maria'); // 작성한 maria.js를 불러온다.
    // connection 은 서버가 켜질 때 app.js 에서 수행되었다.
    
    /* GET home page. */
    router.get('/', function(req, res, next) {
      maria.query('select * from test', function(err, rows, fields) { // 쿼리문을 이용해 데이터를 가져온다.
        if(!err) { // 에러가 없다면
          res.send(rows); // rows 를 보내주자
        } else { // 에러가 있다면?
          console.log("err : " + err);
          res.send(err); // console 창에 에러를 띄워주고, 에러를 보내준다.
        }
      });
    });
    
    module.exports = router;

필요한 설명은 주석에 달아놓았다. 이제 터미널에 `npm start`를 입력해 서버를 실행시켜보자.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/10.png"></center>
<center><small>Webstorm에 내장된 터미널을 사용했다.</small></center>

서버가 켜졌다면 [http://localhost:3000/](http://localhost:3000/) 에 가서 확인해보자.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/11.png"></center>
<center><small>데이터가 매우 깔끔하게 들어왔다.</small></center>

혹시 모르니 다시 MariaDB에 접속해 `INSERT INTO test VALUES ( 2, 'wow' );` 를 쳐서 데이터를 추가해보자!

    MariaDB [testProject]> INSERT INTO test VALUES ( 2, 'wow' );
    Query OK, 1 row affected (0.006 sec)
    
    MariaDB [testProject]> SELECT * FROM test;
    +----+------+
    | id | name |
    +----+------+
    |  1 | LimC |
    |  2 | wow  |
    +----+------+
    2 rows in set (0.001 sec)

그리고 다시 [http://localhost:3000/](http://localhost:3000/) 에 접속해주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-09-testProject2/12.png"></center>
<center><small>야호</small></center>

이걸로 mysql, express를 서로 연동해보았다. 다음은 테이블 구조를 짜고 로그인을 구현해보자.