---
layout: post
title: "Commit Message Guide"
date: 2022-02-23 22:59:00 +0530
excerpt: "왜 커밋 메시지를 잘 쓰려고 노력해야 할까?"
categories: Git
tags: 커밋 메시지구조
---

## 커밋 메시지를 잘 쓰려고 노력해야 하는 이유

왜 커밋 메시지를 잘 쓰기 위해 노력해야 할까요? 이유는 서로 다를 수 있지만, 잘 쓰인 커밋 메시지가 더 유익하다는 사실에는 많은 프로그래머들이 동의할 것으로 여겨집니다.

구체적인 이유에 대해 생각해보기 전에, 우선 현재 이야기하고 있는 커밋 메시지는 Git, SVN 등의 툴에서 커밋을 할 때 사용합니다. 이 툴들은 형상관리 툴로써 형상관리에 대한 정의를 살펴보면 아래와 같습니다.

> **형상관리란?**<br/>
> 소프트웨어의 변경사항을 체계적으로 추적하고 통제하는 것으로, 형상관리는 일반적인 단순 버전관리 기반의 소프트웨어 운용을 좀더 포괄적인 학술 분야의 형태로 넓히는 근간을 이야기한다.

형상관리가 버전관리 기반으로 설명이 되어 있어 좀더 리서치를 한 결과, 버전관리와 변경관리라는 개념도 같이 나오게 되었습니다.

형상관리는 변경 관리와 버전관리를 포괄하는 개념으로 쉽게 정리하면 아래와 같습니다.

- 변경관리
  - 말그대로 소스의 변경 상황을 관리한다. 문서의 변경 이력과 복원 등의 기능이 제공된다.
- 버전관리
  - 변경을 관리하기 위한 효과적인 방법이 버전으로 관리하는 것이다. 단순히 이 프로그램이 언제 어떻게 변경되었다를 넘어서 사소한 체크인, 체크아웃부터 릴리즈, 퍼블리싱의 과정을 버전으로 관리할 수 있도록 한다.
- 형상관리
  - 위의 관리개념이 포함되고 프로젝트 진행상황, 빌드와 릴리즈 퍼블리싱까지 모두 관리할 수 있는 통합 시스템이라고 할 수 있다.

즉, 형상관리에서 가장 중요한 것은 **무엇이 어떻게 변했는지 알 수 있어야 한다**는 것입니다. 형상관리 툴인 Git의 경우, 파일의 변경을 커밋 단위로 저장되기 때문에 이 변경 내역을 알려주는 **_커밋 메시지을 보고 무엇이 어떻게 변했는지 알 수 있어야 한다_** 는 것입니다.

이를 통해 아래와 같은 효과를 얻을 수 있습니다.

- 더 좋은 커밋 로그 가독성
- 더 나은 협업과 리뷰 프로세스
- 더 쉬운 코드 유지보수

이렇게 커밋 메시지를 잘 작성해야만 하는 이유에 대해 알아보았습니다.<br/>
그러면 어떻게 해야 커밋 메시지를 잘 작성할 수 있을지 알아봅시다.

## 좋은 커밋 메시지를 작성하기 위한 7가지 약속

좋은 커밋 메시지를 작성하기 위해서 아래의 7가지를 따라서 진행하면 더 좋은 커밋 메시지를 작성할 수 있고, 최종적으로 7가지 약속을 바탕으로 커밋 메시지 구조에 대해 알아보겠습니다.

> 이하 약속은 커밋 메시지를 **English**로 작성하는 경우에 최적화되어 있습니다. 한글 커밋 메시지를 작성하는 경우에는 더 유연하게 적용하셔도 좋을 것 같습니다.

1. 제목과 본문을 한 줄 띄워 분리하기
2. 제목은 영문 기준 50자 이내로
3. 제목 첫글자는 대문자로
4. 제목 끝에 `.` 금지
5. 제목은 **_명령조_** 로
6. 본문은 영문 기준 72자마다 줄 바꾸기
7. 본문은 **_어떻게_** 보다 **_무엇을, 왜_** 에 맞춰 작성하기

### 1. 제목과 분문을 한 줄 띄워 분리하기

제목과 본문 사이에 한 줄 공백을 두면, 얼마나 좋은지 보여드리겠습니다. 아래의 커밋 메시지는 제목, 공백 한줄, 본문의 구조로 구성되어 있습니다.

```shell
Derezz the master control program

MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

`git log` 명령어를 통해 커밋메시지를 확인하면 아래와 같이 보여집니다.

```shell
$ git log
commit 42e769bdf4894310333942ffc5a15151222a87be
Author: Kevin Flynn <kevin@flynnsarcade.com>
Date:   Fri Jan 01 00:00:00 1982 -0200

 Derezz the master control program

 MCP turned out to be evil and had become intent on world domination.
 This commit throws Tron's disc into MCP (causing its deresolution)
 and turns it back into a chess game.
```

`git log --online` 으로 명령어를 치면 더 깔끔하게 보여집니다.

```shell
$ git log --online
42e769 Derezz the master control program
```

하지만, 여기서 제목과 본문사이에 한 줄 공백을 넣지 않을 경우 아래와 같이 보여집니다.

```shell
$ git log --oneline
42e769 Derezz the master control program MCP turned out to be evil and had become intent on world domination. This commit throws Tron's disc into MCP (causing its deresolution) and turns it back into a chess game.
```

제목과 본문 사이의 한 줄 공백 유무의 따라 가독성이 완전히 달라집니다. 그러므로 **제목과 본문 사이에는 한 줄 공백**을 넣어주세요!

추가로 `git shortlog`라는 명령어도 있습니다. 아래의 결과가 보여집니다.

```shell
$ git shortlog
Kevin Flynn (1):
    Derezz the master control program

Alan Bradley (1):
    Introduce security program "Tron"

Ed Dillinger (3):
    Rename chess program to "MCP"
    Modify chess program
    Upgrade chess program

Walter Gibbs (1):
    Introduce protoype chess program
```

### 2. 제목은 영문 기준 50자 이내로

50자 이내 작성의 경우 엄격한 제한이 아니고 경험상의 규칙일 뿐입니다. 제목 줄을 50자 이내로 유지하면 가독성이 보장되고, 어떤 것을 커밋하는지 설명하는 가장 간결한 방법에 대해 생각할 수 있도록 유도해 보다 좋은 제목을 작성할 수 있습니다.

> 팁: 요약이기 어렵다면 한 번에 너무 많은 변경사항을 커밋하고 있는 것일 수 있습니다. atomic commit을 위해 노력해 주세요.

### 3. 제목 철글자를 대문자로

한글의 경우 해당되는 내용은 아니지만, 영어로 작성할 경우 첫글자를 대문자로 작성하는 것이 좋습니다. 영어 문법에 관한 내용이므로 지켜주는 것이 아주 좋습니다.

- Accelerate to 88 miles per hour
- ~~accelerate to 88 miles per hour~~

### 4. 제목 끝에 `.` 금지

제목 끝에 `.`은 필요하지 않습니다. 그리고 50자 이내로 유지하려면 공간이 중요하여 제목 끝에는 `.`을 적지 않습니다.

- Open the pod bay doors
- ~~Open the pod bay doors.~~

### 5. 제목은 **_명령조_** 로

제목을 작성하거나 한 줄 메시지로 커밋을 할때, 즉 커밋 메시지 가장 첫 문장의 영문법은 **_명령조_** 로 합니다. 이는 첫 단어를 **_동사원형_** 으로 쓰라는 의미입니다.

- 회원 서비스 생성
- ~~회원 서비스 생성했습니다~~

명령문은 다소 무례하게 들릴 수 있지만, Git 커밋 제목 라인의 경우 명령형(동사원형)이 적합합니다. 그 이유 중 하나는 Git 자체가 사용자를 대신하여 커밋을 생성할 때마다 명령문(동사원형)을 사용하기 때문입니다.

예를 들어, `git merge` 명령어를 사용할 때 디폴트 메시지로 아래와 같이 생성됩니다.

```shell
Merge branch 'myfeature'
```

그리고 `git revert` 명령어를 사용할 때도 아래와 같이 디폴트 메시지가 생성됩니다.

```shell
Revert "Add the thing with the stuff"

This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```

이처럼, **_명령문(동사원형)_** 으로 커밋 메시지를 작성하는 것은 **Git 자체의 기존 제공 규칙(Built-in Conventions)** 을 따른 다는 것을 의미합니다. 때문에 커밋 제목을 명령문(동사원형)으로 작성하면, 자동 메시지로 채워진 커밋 사이에 자연스레 녹아듭니다.

### 6. 본문은 영문 기준 72자마다 줄 바꾸기

git은 자동으로 커밋 메시지를 줄바꿈하지 않습니다. `git log`에 스타일을 지정해서 자동 줄바꿈이 적용된 로그 출력을 만들 수 있지만, 단순한 `git log` 명령어 입력만으로도 보기 좋은 메시지를 만들고자 한다면 72자 마다 줄을 바꾸는 것을 추천드립니다.

### 7. 본문은 **_어떻게_** 보다 **_무엇을, 왜_** 에 맞춰 작성하기

본문은 어떻게 구현했는지 보다는 무엇과 왜에 맞춰서 작성합니다. 구현 방법의 경우 코드를 통해 확인이 가능하므로, 무엇과 왜에 집중해야 합니다.

무엇(What)에 해당하는 것은 무엇이 잘못되었는지 혹은 변경 전의 방식에 대한 것을 뜻하고, 왜(Why)에 해당하는 것은 이전 방식(혹은 무엇이 잘못되었는지)을 현재 방식으로 해결하기로한 이유에 대해 작성해줍니다.

### 좋은 커밋 메시지 구조란?

위의 7가지 약속을 바탕으로 좋은 커밋 메시지 구조도 같이 살펴보고 싶을 경우 [좋은 Git 메시지 구조](https://github.com/wlroh/Today-I-Learned/blob/master/Git/%EC%A2%8B%EC%9D%80%20Git%20%EB%A9%94%EC%8B%9C%EC%A7%80%20%EA%B5%AC%EC%A1%B0.md)를 참조하면 됩니다.

### 레퍼런스

- [구성관리](https://ko.wikipedia.org/wiki/%EA%B5%AC%EC%84%B1_%EA%B4%80%EB%A6%AC)
- [변경관리, 버전관리, 형상관리의 차이](https://raisonde.tistory.com/entry/%EB%B3%80%EA%B2%BD%EA%B4%80%EB%A6%AC-%EB%B2%84%EC%A0%84%EA%B4%80%EB%A6%AC-%ED%98%95%EC%83%81%EA%B4%80%EB%A6%AC%EC%9D%98-%EC%B0%A8%EC%9D%B4)
- [좋은 git 커밋 메시지를 작성하기 위한 7가지 약속](https://meetup.toast.com/posts/106)
- [How to Write a Git Commit Message](https://cbea.ms/git-commit/)
