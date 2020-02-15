---
layout: post
title: "IntelliJ에서 Github Remote 레포지토리 만들기"
date: 2019-11-16 13:20 +0900
categories: [IntelliJ]
---
오늘은 IntelliJ 내부에서 **Github Repository를 직접 만들고**, 커밋하고 푸쉬 하는 방법에 대해서 알아보자.

## 기존의 문제점? 
문제점이라기 보다, 불편했던 부분은 다음과 같다. 

1. 로컬에서 프로젝트를 생성한다.
2. git init
3. web 사이트를 열어서 자신의 github 사이트로 이동 한다. 
4. create new repository를 만들고 (기왕이면 로컬에서 만든 프로젝트명과 동일하게 생성함)
5. git remote add `만든 저장소URL` 를 자신의 로컬 해당 프로젝트의 terminal에 원격지로 추가 한다. 
6. 그리고 푸쉬한다. git push origin master


Context Switching 일어나기 때문에 왔다 갔다 하는 작업이 귀찮았다.🤔  

개발자에게 약간의 귀찮니즘은 반드시 필요하다.👍 그래서 방법을 찾아냈다.

## 방법
Intellij에서 단축키 `cmd + shift + a` (Action 키)를 누르고 `share project on Github` 라고 입력합니다.

![](/assets/images/share1.png)

Gitbub 계정에 로그인 되어있으면, `Repository name` 에 만들 이름을 적어준다. 기본 default값이 project 명으로 설정 되어 있다. 
추가로, Remote의 alias 도 지정해 줄 수 있고, `private` 옵션도 지정 가능 하다.
마지막으로 `share`버튼을 클릭하면 
![](/assets/images/share2.png)

다음과 같이 오른쪽 하단에  `성공`했다고 알려준다. 
![](/assets/images/share3.png)


실제 자신의 Github 사이트에 들어가면 Repository가 만들어 진 것을 확인할 수 있다.
![](/assets/images/share4.png)
