---
layout: post
title: Node.js + Express + MariaDB(Mysql)로 게시판 만들기 - 1.  Hello World 및 Nodemon 설정
date: 2020-05-16 00:03:00 +0000
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

<center><img src="/assets/img/nodejs/newProject/2020-05-12-newProject1/1.png"></center>
<center><small>IDE는 MS사의 Visual Studio Code를 사용해도 무방하다. Notepad++도 가능하다.</small></center>

---

### 1-1. Hello World 띄우자(Controller, View)

---

일단 저희 프로젝트는 Node.js의 웹 프레임워크인 Express를 통해 MVC패턴으로 게시판을 만들어 볼겁니다. MVC패턴에 대한 설명은 [이 블로그](https://m.blog.naver.com/jhc9639/220967034588)에 잘 나와있습니다. 일단 Java의 Spring으로 비교하자면 **Controller의 역할**을 하는 부분이 **routes 디렉터리** 부분입니다.

그럼 저번에 세팅한 Express의 경로를 다시 한번 확인해보겠습니다.

```
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

이렇게 되어있어야 합니다. 이게 2020-05-15일 기준으로 그냥 express-generator 쓰면 세팅되는 부분입니다. 물론 —view 세팅을 pug가 아닌 ejs로 하면 또 달라지지만요.

이제 저 곳의 routes/index.js 부분으로 가볼까요?

```jsx
// routes/index.js

const express = require('express');
const router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

먼저 라우터. 즉, 라우팅에 대한 개념을 잡고가야 겠습니다. 여기서 의미하는 라우팅은 URI 라우팅입니다. 사용자가 접근 한 URI(URL이 아닙니다. URL은 URI안에 포함되어있는 개념입니다. URL이 뭐고 URI가 무엇인지 궁금하다면 [이 링크](https://velog.io/@pa324/%EA%B0%9C%EB%B0%9C%EC%83%81%EC%8B%9D-URI-URL-%EC%B0%A8%EC%9D%B4-%EC%A0%95%EB%A6%AC)를 참고해주세요.)에 따라서 메소드를 호출해주는 기능입니다.

이제 위의 코드를 잠깐 뜯어보겠습니다.

- `const express = require('express');`
- `const router = express.Router();`
    - express는 라우팅을 지원합니다. 그 기능을 사용하기 위해서 express를 require으로 불러옵니다. 그리고 router라는 변수안에 express의 Router기능을 넣습니다.
- `router.get('/', function(req, res, next) {`
    - 이제 라우팅을 위해 router 변수를 사용해줍니다. get 메소드는 REST API의 GET 통신을 위한 메소드입니다.
    - 첫 번째 인자인 `'/'` 은 주소값입니다. 만약 사용자가 http://내사이트.com/ 으로 접속했다면 바로 이어지는 두 번째 인자로 들어온 함수를 실행합니다.
    - 두 번째 인자는 앞서 설명해 드렸듯이 해당 함수를 실행합니다. 함수의 인자에 대해서 설명드리겠습니다.
        - `req` : request의 약자로 요청하는 쪽에서 넘어오는 정보를 가지고 있습니다.
        - `res` : reulst의 약자로 저희가 사용자에게 View를 뿌려줄때 사용합니다.
        - `next` : 여기선 get 메소드의 인자가 2개이지만 사실 3개 이상으로 설정할 수 있습니다. 여기서의 next는 다음 인자로 설정된 함수를 의미합니다.
    - 제가 기초가 부족해서 설명이 부족했습니다. REST API의 GET, POST, PATCH, DELETE 에 대한 개념이 확실하지 않으신 분들은 [이 블로그](https://meetup.toast.com/posts/92)에서 공부하고 오시면 편합니다.
- `res.render('index', { title: 'Express' });`
    - 결과값으로 템플릿 페이지를 렌더링해서 뿌려준다는 의미입니다. render 함수의 첫 번째 인자인 `'index'`는 `/views/index.pug` 파일을 의미합니다. 이 것에 대한 설명은 `app.js`에 있습니다.
    - 두 번째 인자는 넘겨줄 값입니다. `index.pug`라는 파일에서 title이라는 변수는 문자열 `'Express'`를 가지게됩니다.
- `module.exports = router;`
    - 이제 다른 js 파일에서 `routes/index.js` 파일을 모듈로 사용할 때에는 router를 사용하게됩니다.

간단하지만 제가 너무 어렵게 설명한건 아닐까 생각이듭니다. 대충 요약하자면 **'/'으로 접속하면 두번째 인자 함수를 통해서 결과값으로 index.pug를 렌더링해서 뿌려주는** 함수였습니다.

이렇게 된 김에 `index.pug` 파일도 확인하겠습니다.

```jsx
// views/index.pug

extends layout

block content
  h1= title
  p Welcome to #{title}
```

일단 템플릿 문법에 관한 설명은 [제 블로그 글](https://doncolmi.github.io/Node.js-%EA%B3%B5%EB%B6%80(9)/)을 보면 대충 나와있습니다. 이 쪽에서 학습하시면 더욱 수월하게 진행할 수 있습니다. 이제 코드를 뜯어보겠습니다. 뜯을것도 없지만요...

- `extends layout`
    - 아래 코드는 views/error.pug 파일입니다.

    ```jsx
    // views/layout.pug

    doctype html
    html
      head
        title= title
        link(rel='stylesheet', href='/stylesheets/style.css')
      body
        **block content**
    ```

    - extends는 위와 같이 공통된 레이아웃을 사용할 때에 해당 pug파일을 다른 pug파일로 불러 오는 역할을합니다.
    - 주목 해야할 부분은 `block content` 입니다. 자세히보면 `index.pug`파일에도 block content가 있죠? **여기서 부터 두 파일이 이어지게 됩니다.**
- `h1= title`
    - 아까 `routes/index.js`에서 `res.render('index', {title : 'Express'});` 로 데이터를 넘긴 것을 기억하시죠?? 여기서 저 title이 바로 controller 역할을 하는 `routes/index.js`에서 넘어온 값입니다.
- `p Welcom to #{title}`
    - 해당 코드는 `<p>Welcome to #{title}</p>`로 렌더링 됩니다.
    - 위 라인에선 `title`을 `#{}`안에 넣지 않고 사용했는데 왜 이번엔 `#{title}`이라고 표현한건가요?
        - 텍스트와 구분하기 위해서 텍스트와 섞어 사용할 때에는 `#{}`을 사용해서 해당 변수를 사용해줍니다.

그러면 이 코드는 어떻게 html로 바뀌게 될까요?

```html
<html>
	<head>
		<title>Express</title>
		<link rel="stylesheet" href="/stylesheets/style.css">
	</head>
	<body>
		<h1>Express</h1>
		<p>Welcome to Express</p>
	</body>
</html>
```

아마도 이렇게 렌더링되어서 사용자의 컴퓨터에 뿌려질 겁니다. 그럼 이제 서버를 켜서 한번 이 사이트에 들어가볼까요?

### 1-2. Hello World 띄우자(진짜로)

---

일단 서버 실행을 위한 방법을 알려드리겠습니다. 일단 가장 최상위 폴더에 위치한 `package.json`을 확인해주세요. 아마도 아래와 비슷한 코드가 있을겁니다.

```jsx
{
  "name": "board",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "morgan": "~1.9.1",
    "pug": "2.0.0-beta11"
  }
}
```

지금 다른 걸 보실 필요는 없습니다. package.json은 npm의 핵심적인 역할을 담당하고 있습니다. 패키지에 관한 정도와 의존하고 있는 모듈들의 버전을 담고있습니다.

일단 `"scripts" : { "start" : "node ./bin/www" }` 부분을 확인해주세요. 이 말의 뜻은 **"너가 콘솔창에 npm start를 입력한다면 내가 node ./bin/www를 실행하지" 라는 뜻입니다.** 즉 우리는 굳이 npm을 사용하지 않더라도 콘솔창에 직접 `node ./bin/www`를 입력한다면 서버를 실행시킬 수 있는거죠.

하지만 굳이 귀찮으니 하진 않겠습니다. 바로 cmd를 띄워서 `npm start`를 입력해보겠습니다.

---

# ⚠잠깐!!

만약 내가 IDE툴로 VSCode나 WebStorm을 사용하고 있다면 아래 과정에서 굳이 `cd`명령어를 통해서 경로를 이동할 필요가 없습니다. VSCode와 WebStorm에서 터미널을 사용하는 방법은 아래와 같습니다.

### VSCode

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/2.png"></center>

1. 위 상단 메뉴에서 Terminal 을 선택하고 하위 메뉴인 New Terminal을 눌러줍니다. 아니면 Ctrl + Shift + `(숫자 1 옆에 있는 키) 를 사용하여 바로 New Terminal을 실행할 수 있습니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/3.png"></center>

2. 그러면 하단에 바로 해당 프로젝트 경로로 터미널이 실행됩니다.

### WebStorm

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/4.png"></center>

1. 왼쪽 하단에 보면 Terminal이라는 버튼이 있습니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/5.png"></center>

2. VSCode와 동일하게 아래에 경로 설정이 된 터미널이 나타납니다.

---

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/6.png"></center>

일단 `cd C:\board` 로 저희의 프로젝트가 있는 폴더로 와주세요. 만약 프로젝트가 저 경로가 아니라면 자신에 맞게 수정해주시면 됩니다.

그리고 나서 `npm start`를 바로 입력해줍니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/7.png"></center>

이렇게 두 줄이 실행되었다면 잘 실행되고 있는겁니다. 이제 [http://localhost:3000/](http://localhost:3000/) 으로 이동해주세요.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/8.png"></center>

그렇다면 이렇게 올바르게 사이트가 나오는 것을 확인할 수 있습니다. 추가로 cmd창을 다시 보면 

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/9.png"></center>

아마 이런식으로 어떤 요청이 들어왔고 어떤 신호를 보냈는지 거기다가 응답하는데 걸린 시간까지 밀리세컨으로 잘 나오고 있습니다.

(stauts 숫자는 304 또는 200으로 나와야합니다.)

드디어 서버를 실행시켰습니다. 그리고 서버를 끌 때는 Ctrl+C 를 누르고 **일괄 작업을 끝내시겠습니까 (Y/N)?** 라는 안내문이 나오면 y를 누르시고 엔터키로 빠져나오실 수 있습니다.

### 1-3. Nodemon으로 작업 환경을 편하게 만들자.

---

이제 우리는 Node.js를 통해 개발을 하게 됩니다. js파일을 수정하고 그 수정 내용을 서버에 적용 시키려면 서버를 껐다 켰다를 반복해야합니다.

지금은 규모가 매우 작은 게시판이여서 개발할 때 큰 불편함은 없겠지만...엄청 규모가 커진다면 개발 내용을 적용할 때마다 서버를 재시동 해야할까요?

여기서 우리는 Nodemon이라는 Node.js 모듈을 사용합니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/10.png"></center>
<center><small>nodemon은 node monitor의 약자입니다.</small></center>

일단 npm을 통해서 nodemon을 설치해야합니다. 위를 참고해 cmd나 각 IDE의 터미널을 이용해 프로젝트 경로로 들어가주자.

그리고 `npm install nodemon`을 입력하여 설치를 해주자.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/11.png"></center>

참고로 저는 이미 설치를 한적이 있어서 처음 nodemon을 설치하는 분들과는 메세지가 다르게 뜰 수 있습니다. error가 뜨지 않았다면 별 문제 없는 것이니 다음 단계로 오시면 됩니다.

이제 nodemon을 설치도 했겠다. 프로젝트에 적용만 하면 모든게 끝납니다. 그럼 이제 다시 프로젝트 최상위 버전에 있는 `package.json`로 가보겠습니다.

```jsx
// package.json

"scripts": {
    "start": "node ./bin/www"
  },

// 해당 부분을 아래와 같이 바꿔주세요.

"scripts": {
    "start": "nodemon ./bin/www"
  },
```

위 부분을 아래부분처럼 바꿔주시길 바랍니다. 윗 부분에서 제대로 이해를 하고 오신게 맞다면 이제 `npm start`을 입력한다면 nodemon으로 서버가 실행되게 됩니다.

<center><img src="/assets/img/nodejs/newProject/2020-05-16-newProject2/12.png"></center>

윗 부분의 과정들과는 다르게 [nodemon]의 로그가 뜨기 시작합니다. 이제 js파일을 저장하면 자동으로 새로고침 됩니다.