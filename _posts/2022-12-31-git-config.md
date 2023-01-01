---
layout: post
title: "회사계정과 개인계정 분리방법"
date: 2022-12-31
excerpt: ".gitconfig 활용법에 대해 알아보자"
categories: Git
---

저의 기존 git 계정 세팅은 회사 업무용 계정이 global 설정으로 되어 있고, 개인용 계정은 각각의 레포지토리마다 local 옵션을 통해 설정해서 사용했습니다.

이렇게 사용을 하다보니 local 설정이 안되어 있는 레포지토리의 경우 의도치 않게 global 설정인 회사 계정으로 커밋되는 문제가 계속 발생되어 회사 계정과 개인 계정을 분리하는 방법에 대해 정리해보고자 합니다.

<br/>
## `Conditional includes` 을 활용해 폴더별로 git 계정을 분리하자

Git Configuration file 관련 docs를 읽어보면 `Conditional includes`에 대한 내용이 있습니다. `Conditional includes`에 대해 알아보면 세팅한 조건에 해당하면 설정한 경로에 다른 config 파일을 포함할 수 있도록 해줍니다.

즉, 조건에 따라 여러 config 파일들을 설정한 경로에 지정해주어 폴더별로 계정을 분리할 수 있습니다. 

<br/>
## `global gitconfig` 에 `Conditional includes` 을 구성을 어떻게 할까?

맥북의 경우 global gitconfig 파일은 `~/.gitconfig` 경로에 위치해 있습니다. global config 파일을 먼저 아래와 같이 구성합니다.
``` bash
[includeIf "gitdir:~/personal/"]
  path = ~/.gitconfig-personal

[includeIf "gitdir:~/company/"]
  path = ~/.gitconfig-company
```

해당 내용은 보면 git 레포지토리가 `~/personal/` 경로에 포함될 경우 path로 설정되어있는 `~/.gitconfig-personal` 설정파일을 적용한다는 의미입니다.

만약 `~/.gitconfig-personal` 파일이 아래와 같이 구성되어 있을 경우

``` bash
[user]
  name = Laewoong
  email = email@gmail.com
```

`~/personal` 경로 내 모든 레포지토리에서 user.name과 user.email을 조회하면 `~/.gitconfig-personal`에 설정한 정보로 조회하게 됩니다. 확인을 해보면,
``` bash
$ cd ~/personal/git-repository
$ git config user.name
Laewoong

$ cd ~
$ git config user.name

```
`~/personal/` 경로 내에 있는 레포지토리여서 설정 정보 조회 명령어를 통해 개인용 설정파일(.gitconfig-personal)에 있는 이름을 조회하는 것을 확인할 수 있고,<br/>
`~` 경로는 현재 별도의 설정파일이 설정되어 있지 않아 아무 정보가 조회되지 않는 것을 확인할 수 있습니다.

이렇게 구성해놓으면, 전역으로 user.name과 user.email이 설정되지 않고 `.gitconfig` 에 구성한 경로에만 설정이 적용되어 업무용과 개인용 계정을 완전히 분리할 수 있게 됩니다.

간단하면서도, 해당 기능을 잘 활용하면 계정 뿐만 아니라 git editor 부터 다양한 설정값들을 손쉽게 관리할 수 있어 `conditional includes` 기능을 활용하면 활용범위는 무긍무진해보입니다.

래퍼런스로 링크를 걸어두었으니, 혹시 더 여러가지 설정을 하고 싶으시면 링크를 참조해 더 작업해보시길 추천드립니다!<br/>
글을 읽어주셔서 감사드리고, 새해 복 많이 받으세요!

<br/>
## 레퍼런스

- [git-config](https://git-scm.com/docs/git-config#_conditional_includes)
