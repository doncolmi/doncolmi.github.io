---
layout: post
title: Node.js + Express + MariaDB 로 로그인/게시판 구현하기(1)
date: 2020-03-08 08:18:00 +0000
description: # Add post description (optional)
img: ./nodejs/testProject/2020-03-08-testProject1/1.png
tags: [nodejs]
---

# 소개

---

node.js와 express를 공부했지만 아직 갈피를 잡지 못했다. 사실 이론만 공부하는건 내 성격이 아니기도 하고 해서 일단 부딪혀가면서 연습하려고 한다.

일단 기본적인 세팅은 아래와 같다.

- [Node.js 12.16.1 LTS](https://nodejs.org/ko/)
- express 4.16.1
    - 템플릿 엔진은 pug 사용 예정
- [MariaDB](https://mariadb.org)
- IDE는 Jetbrain사의 Webstorm을 사용한다.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/1.png"></center>
<center><small>IDE는 MS사의 Visual Studio Code를 사용해도 무방하다. Notepad++도 가능하다.</small></center>

### Node.js 세팅

---

먼저 Node.js가 설치되어 있지않다면 설치해주자. 만약 Node.js가 뭔지 잘 모른다면 [이 링크](https://doncolmi.github.io/tags/#nodejs)에서 nodejs로 분류 되어있는 이전글 들을 참고하면 된다.

내 컴퓨터에 Node.js가 설치되어있는지 궁금하다면 윈도우 + Q키를 눌러 cmd를 친 다음 명령 프롬프트를 켜준다.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/2.png"></center>

그리고 `node -v`를 친다면 설치된 노드의 버전이 나온다.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/3.png"></center>
<center><small>난 예전에 설치해놓은 버전이기에 최신 LTS 버전이 아니다.</small></center>

**명령 프롬프트창에서 나가지 않고,** `npm install express -g` 를 입력해준다. 웹 프레임워크로 선택한 express를 다운받는 과정이다.

**그리고 또 다시** `npm install express-generator -g`를 입력해서 express-generator도 받아준다. 이는 Express framework를 사용하는 애플리케이션의 골격을 만들어주는 생성 도구이다.

**편의를 위해서** `npm install nodemon -g`도 해주기로 하자. nodemon은 코드를 수정했을 때 자동으로 애플리케이션을 재시작해주는 역할을 해준다.

모두 설치가 완료되었다면 자신이 프로젝트를 만들고 싶은 위치에 **명령 프롬프트를 이용해 접근해주자.** 프로젝트에선 `C:\nodejs`폴더를 기준으로 진행한다.

    Microsoft Windows [Version 10.0.18362.657]
    (c) 2019 Microsoft Corporation. All rights reserved.
    
    C:\Users\j>cd C:\nodejs
    
    C:\nodejs>express --view=pug testapp

위는 명령 프롬포트 창이다. `cd C:\nodejs`를 이용하여 **해당 위치로 이동한다음** `express --view=pug 프로젝트이름`을 입력해준다.

프로젝트 이름은 자신이 원하는 것으로 하면 된다. 나는 `testapp`으로 정했다.

`--view=pug`은 템플릿 엔진을 무엇으로 쓸건지 설정해주는 것인데, 이를 설정하지 않으면 자동으로 `jade`라는 템플릿 엔진으로 설정된다.

본래 **jade였던 이름이 pug로 바뀌었을 뿐이지만 편의를 위해 나는 pug로 바꾼다음 진행한다. 템플릿엔진은 jade/pug 말고도 유명한 것은 ejs가 있다. node.js의 템플릿 엔진은 [이 링크](https://doncolmi.github.io/Node.js-공부(9)/)를 통해 확인하자.** 

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/4.png"></center>

이렇게 나왔다면 정상적으로 프로젝트가 만들어 진것이다. 이제 Webstorm(이나 vscode)를 이용해 저 폴더로 접근해주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/5.png"></center>

대충 이런창이 뜨면 된다. 하단의 Terminal을 눌러 IDE에서 터미널을 띄워 작업할 수 있다. 일단 오른쪽에 `Install Dependencies`가 보이는데 `Run 'npm install'` 을 눌러서 의존성들을 설치해주자.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/6.png"></center>
<center><small>오른쪽 하단에 위치해있다.</small></center>

`express-generator`에 의해 생성된 애플리케이션은 아래와 같은 구조를 가진다.

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

사실 express-generator나 express의 구조에 대해서는 [이미 글을 쓴 바](https://doncolmi.github.io/Node.js-공부(8)/) 있다. 이해가 안가는 부분이 있다면 댓글을 통해서나 블로그 왼쪽 하단 이메일버튼을 통해 질문해준다면 아는만큼 알려드릴게여;

간단히 설명하자면 `app.js`가 전반적인 프로젝트의 설정을 담당하고있다. `package.json`은 프로젝트의 의존성이나 이름 설명등 정보를 담고있다.

`bin/www.js`는 http와 같은 포트나 각종 서버 설정에 관한 내용을 담고있다.

`public`은 정적파일들(이미지/js/css 등)을 관리해주는 디렉터리이다.

`routes`는 MVC패턴에서 controller를 담당한다고 보면 되고 `views`는 MVC의 view를 담당한다.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/7.png"></center>
<center><small>이게 뭔데? 라고 질문하면 제가 직접 대답하는거보단 검색하는게 더 빠를거 같습니다.</small></center>



이제 다시 프로젝트로 돌아와서 `npm install`을 눌러서 모든 의존성이 설치되었다면 `package.json`으로 들어가 내용을 살펴보자.

    {
      "name": "testapp",
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

이렇게 되어있어야 정상이다. 버전은 일부 차이가 있을 수 있다. 대충 이런 형태를 지니고 있어야 한다. 이제 아까전에 설치한 `nodemon`을 프로젝트에 적용시켜보자.

그렇게 거창한 작업이 아니고 그저 위 `package.json`에서 `"start" : "node ./bin/www"` 이 부분을 `"start" : "nodemon ./bin/www"`로 바꿔주기만 하면 된다.

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
        "pug": "2.0.0-beta11"
      }
    }

수정이 되었다면 이런 형태를 띄고 있어야 한다. 그리고 터미널로 가서 `npm start`를 입력해주자.

**여기서 주의할 점은 현재 터미널(명령 프롬프트) 경로가 express-generator로 만든 프로젝트 경로 안이어야 한다는 점이다.**

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/8.png"></center>
<center><small>명령 프롬프트를 사용한다면 필이 cd를 이용해 경로를 설정해주어야 한다.</small></center>

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/9.png"></center>
<center><small>이렇게 IDE에 내장된 터미널을 사용하면 바로 프로젝트 경로로 설정되기에 아주 편리하다.</small></center>

위 사진처럼 `nodemon`이 뜬다면 `nodemon`이 잘 적용된 것이다.

그리고 [http://localhost:3000/](http://localhost:3000/) 으로 접속해 아래와 같은 창이 뜬다면 모두 잘 설정되고 서버가 켜진 것이다.

<center><img src="/assets/img/nodejs/testProject/2020-03-08-testProject1/10.png"></center>
다음 글에선 **MariaDB를 설치하고 프로젝트에 연동**을 해보겠다.