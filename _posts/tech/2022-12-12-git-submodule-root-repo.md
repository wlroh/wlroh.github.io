---
layout: post
title: "서브모듈에 대한 커밋이력을 계속 관리해야할까?"
date: 2022-12-12
excerpt: "루트 리포지토리에서 서브모듈에 대한 커밋 이력을 관리해야하는지에 대해 알아보자"
categories: Git
---

### `Root Repository` 에서 `SubModule` 에 대한 커밋 히스토리를 관리해야할까?

서브모듈의 `Remote` 가 변경된 후 `Root Repository` 에서 서브모듈을 최신 이력으로 업데이트했을 때 `Root Repository` 의 Stage를 확인하면 `modified {서브모듈} (new commits)` 내용으로 변경이 되었음을 알려줍니다.

``` 
$ git status

...

Changes not staged for commit:
    modified:   {서브모듈} (new commits)

...
```

`Root Repository` 에서는 서브모듈을 `read only` 성격으로 사용하는데 서브모듈의 변경사항에 대해서도 커밋을 해서 관리를 해주어야할까 의문이 들었습니다. '그냥 ignore 처리를 하는 것이 더 낫지 않을까?' 생각이 들어 체크를 해보았습니다.

<br/>

## `Root Repositoy` 에 `SubModule` 을 추가해보자

Root Repository에 서브모듈을 추가하기 위해 먼저 사전세팅을 진행했습니다.

- `Root Repository` 로 사용할 Public Repository를 생성합니다. 생성한 Root Repository의 구조는 아래와 같습니다.
  ```
  root-repository
  ├── .git
  ├── src
  │   ├── java
  │   │   └── Application.java
  │   └── resources
  └── README.md
  ```
- `SubModule` 로 사용할 Private Repository를 생성합니다. 생성한 Private Repository의 구조는 아래와 같습니다.
  ```
  sub-module-repository
  ├── .git
  └── application.properties
  ```
레포지토리 생성이 완료되면 루트 레포지토리에 서브모듈을 구성합니다.

- `Root Repository` 에 `SubModule` 을 구성합니다.
  ``` bash
  # git submodule add [submodule repository] [SubModoule Directory Path]]
  $ git submodule add https://github.com/{submodule-repository}.git src/resources/config

  $ git commit -m "submodule 구성"
  $ git push -u origin master
  ```

서브모듈을 추가하면 추가하는 시점의 서브모듈 레포지토리 최신 커밋 이력상태로 구성하게 됩니다.
``` bash
  root-repository
  ├── .git
  ├── src
  │   ├── java
  │   │   └── Application.java
  │   └── resources
  │       └── config
  │           ├── .git
  │           └── application.properties
  └── README.md
```

이 상황에서 `SubModule` 의 `Remote` 커밋 이력 변경이 된 후 루트 레포지토리에서 서브모듈을 업데이트 후 루트 레포지토리를 서브모듈과 함께 clone을 해보겠습니다.

- (SubModule 의 Remote 커밋 이력 변경이 발생)
- 루트 레포지토리와 서브모듈 함께 clone
  ``` bash
  # git clone --recurse-submodules [Root repository]
  $ git clone --recurse-submodules https://github.com/{root-repository}.git
  ```

<br/>

## 서브모듈과 함께 루트 레포지토리를 클론했을 때 서브모듈의 내용은 어떻게 되어있을까?

만약 서브모듈이 최신 커밋 이력상태로 구성되어 있으면 루트 레포지토리에서 서브모듈에 대해 커밋이력을 관리할 필요가 없다고 생각이 들어서 체크를 해보았습니다.

결과는 최신 커밋 이력이 아닌 루트레포지토리에서 서브모듈을 커밋한 이력으로 가져오는 것을 확인했습니다.

![이미지](https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/2022-12-12-git-submodule.png)

추가로, 깃 허브로 루트 레포지토리에 접속해서 서브모듈을 확인해보니 커밋정보도 같이 포함되어 있는 것을 확인할 수 있었습니다.

  ``` bash
  root-repository
  ├── .git
  ├── src
  │   ├── java
  │   │   └── Application.java
  │   └── resources
  │       └── config@{커밋정보}
  │           ├── .git
  │           └── application.properties
  └── README.md
  ```

<br/>

## `Root Repository` 에서 `SubModule` 에 대한 커밋 히스토리를 관리해주어야 한다.

확인 결과 루트 레포지토리에서 서브모듈을 사용할 때 서브모듈의 레포지토리만 사용하는 것이 아니라 `레포지토리와 커밋을 함께 사용` 하고 있어 서브모듈의 Remote 커밋이력이 변경이 되었을 때 루트 레포지토리에서도 변경된 커밋 이력으로 갱신해서 커밋을 해주어야 추후 루트 레포지토리를 clone 했을 때 서브모듈이 최신 커밋이력을 가져올 수 있습니다.

서브모듈을 사용하면서 협업을 하게 되면 서브모듈을 수정하게 되었을 때 루트 레포지토리도 커밋을 반영해서 꼬이지 않도록 주의가 필요해 보입니다.
