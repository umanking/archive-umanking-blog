---
layout: post
title: "Git 체리픽(CherryPick)이란?"
date: 2020-02-23 13:51 +0900
categories: [Git]
---
## 체리픽(Cherry Pick)?
git cherry pick은 현재 작업 브랜치에서 커밋을 해야 하는데, 다른 브랜치에 커밋을 했을 때 **해당 커밋을 Cherry Pick(체리 따듯이 똑 따서) 지금의 브랜치에 이어 붙이는 것을 의미한다.** 굉장히 편리하고 유용하지만, `merge`나 `rebase` 를 대체 하면 안된다고 한다.

## 사용법
일반적인 커맨드 명령어는 다음과 같다. 
```shell
git cherry-pick [commitSha]
```

## 예제
실제 예제를 하나 살펴보자. `f` 라는 feature 브랜치에 있는 커밋을 master 브랜치로 가져올려고 한다.

```shell
    a - b - c - d   Master
         \
           e - f - g Feature
```

먼저 checkout 을 master 브랜치로 옮긴다.
```shell
git checkout master
```

`f` 라고 했지만, 실제 commit된 hashId를 말한다. 
```shell
git cherry-pick f
```

그러면 다음과 같은 결과가 나온다.
```shell
 a - b - c - d - f   Master
         \
           e - f - g Feature
```

## 참고
- [atlassian.com/git/tutorials/cherry-pick](atlassian.com/git/tutorials/cherry-pick)