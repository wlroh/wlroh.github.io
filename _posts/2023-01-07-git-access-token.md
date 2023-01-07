---
layout: post
title: "GitHub Access Token 관리방법 with Mac"
date: 2023-01-07
excerpt: "GitHub Access Token 관리방법에 대해 알아보자"
categories: Git
---

2021년 8월 13일부로 `비밀번호 인증` 방식으로 github API를 호출할 수 없고, 개인 토큰(personal access token)을 이용해 접속하도록 정책이 변경되었습니다.

정책이 변경됨에 따라 개인 토큰을 발급받고, 해당 토큰을 이용해 레포지토리를 관리하는 여러 방법이 있어서 소개와 함께 현재 제가 사용하는 방식을 공유해드리려고 정리하게 되었습니다.

<br/>
## 먼저, Github Access Token을 발급받자

github API 호출을 하기 위해 토큰이 필요한데 발급받는 과정부터 설명드리겠습니다.
우선 깃허브 설정화면에 들어간 뒤, 메뉴 항목 제일 하단에 `Developer settings` 항목으로 들어갑니다.

`Developer settings`에 접속한 뒤, 엑세스 토큰을 발급하기 위해 `Tokens (classic)` - `Generate new token` - `Generate new token (classic)`을 클릭합니다.

![이미지](https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/git-access-token/image-1.png)

엑세스 토큰 이름, 토근 만료기한, 토근 권한을 선택한 뒤 하단에 `Generate token`을 클릭해 토큰을 생성할 수 있습니다.

토큰 만료기한의 경우 만료기한 없이 생성도 가능하지만 보안상의 이슈로 저는 되도록 3개월로 설정합니다. 

토큰 권한의 경우 되도록 최소한의 권한만 줄려고 합니다. 저만 사용하는 토큰이여도 권한을 모두 주게 되면 의도치 않게 토큰이 노출되었을 때 보안 이슈가 생길 우려가 있어, 최소한의 권한만 주고 추후 권한이 더 필요하면 그 시점에 권한을 추가해주고 있습니다.
토큰 생성 후 언제든지 권한 변경이 가능하니 부담없이 생성하셔도 됩니다.

토큰을 사용하시려는 목적이 대부분 레포지토리 접근을 위한 것이니 Repo 영역의 권한만 주어도 충분하지 않을까 생각합니다.

![이미지](https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/git-access-token/image-2.png)

토큰을 생성하게 되면, 생성된 토큰 값은 한번만 확인할 수 있어서 잊지 않도록 주의하시길 바랍니다.

<br/>
## 엑세스 토큰을 관리방법은 무엇이 있을까?

엑세스 토큰을 이용해 github 접근을 할 때 아래와 같은 대표적으로 아래와 같은 방법들이 있습니다.

- 매번 access token을 입력하는 방법
- remote url 에 access token 정보를 포함키는 방법
- store 모드를 통해 인증정보를 Disk의 텍스트 파일로 저장하는 방법
- osxkeychain 모드를 통해 Mac에서 제공하는 Keychain 시스템을 이용하는 방법

위의 방법 말고도 다양한 방법들이 있는데 저는 `손쉽게 토큰정보를 변경할 수 있어야한다.` 와 `토큰정보를 암호화해서 안전하게 보관해야한다.` 두가지 요구조건을 충족할 수 있는 방법으로 `osxkeychain` 모드를 사용하였습니다.

`osxkeychain` 모드를 사용하게 되면 깃허브 API를 사용하기 위해 인증정보를 Mac의 Keychain Access 시스템을 이용해 인증하기 때문에 인증 정보를 한곳에서 관리할 수 있어 토큰 정보를 손쉽게 변경해줄 수 있으며, 암호화되어 관리되기 때문에 보안도 되어있습니다.

맥에서 `keychain access` 를 검색하면 키체인 접근 어플리케이션을 찾을 수 있습니다.
상단 우측에 검색창에 `github` 검색하면 아래 이미지와 같이 나옵니다.

![이미지](https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/git-access-token/image-3.png)

검색한 github 를 클릭하면 아래와 같이 화면이 뜨고, 하단 `암호보기`를 클릭 후 해당 값에 발급받은 토큰값을 넣어주면 세팅이 완료가 됩니다.

github credential mode가 기본으로 osxkeychain으로 설정되어 있어서 별도로 설정하지 않는 이상 바로 엑세스 토큰을 통해 접근이 가능하지만 별도의 mode로 세팅되어 있는 경우 `osxkeychain` 모드로 변경해주시면 됩니다.

![이미지](https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/git-access-token/image-4.png)

credential mode 확인하는 방법은 아래와 같이 확인할 수 있습니다.

```bash
$ git config -l

...
credential.helper=osxkeychain
...
```

<br/>
## 마무리

Mac의 Keychain access를 활용해 엑세스 토큰을 관리하는 방법을 정리해보았습니다. 만료기간을 3개월정도로 설정해두기 때문에 3개월 주기로 엑세스토큰을 갱신하고 새로운 토큰값으로 변경이 필요해 `토큰을 한 곳에서 관리가 필요한 점` 과 `엑세스토큰을 안전하게 보관할 수 있는 방법` 으로 `Mac - Keychain access` 를 이용하고 있습니다.

긴 글 읽어주셔서 감사합니다. :)<br/>
엑세스 토큰 관리로 인해 고민이 있으신 분들께 도움이 되길 바라며 글을 마치겠습니다.

<br/>
## 레퍼런스

- [git-tools-credential-storage](https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage)
