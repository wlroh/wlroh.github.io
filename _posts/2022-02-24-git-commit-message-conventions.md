---
layout: post
title: "Commit Message Conventions"
date: 2022-02-24 21:03:36 +0530
excerpt: "좋은 커밋 메시지를 작성하기 위한 메시지 구조는 무엇일까?"
categories: Git
tags: 메시지 구조
---

## 좋은 커밋 메시지를 작성하기 위한 메시지 구조

[좋은 Git 커밋 메시지](https://wlroh.github.io/git/2022/02/23/git-commit-message-guide.html) 를 통해 `커밋 메시지를 잘 쓰려고 노력해야 하는 이유` 를 알게 되었고, 좋은 커밋 메시지를 작성하기 위해 `좋은 커밋 메시지를 작성하기 위한 7가지 약속` 을 바탕으로 좋은 커밋 메시지 구조를 보여드리겠습니다.

```
타입(type): 제목(subject)
    -- Blank Line (빈 줄) --
본문(body)
    -- Blank Line (빈 줄) --
꼬리말(footer)
```

크게 공백 Line을 통해 타입 & 제목 / 본문 / 꼬리말을 구분하고 있습니다. 각각의 내용을 정리해서 보여드리겠습니다.

- ### 타입(type)

  `타입(type)` 은 해당 **commit의 성격**을 나타내며, 아래의 종류를 가지고 있습니다. 해당 종류는 다양한 Commit Message Convention 내용을 바탕으로 저의 주관적 판단을 통해 만들었습니다. 종류를 더 추가하거나 생략해도 괜찮습니다.

  - feat: 새로운 기능 추가 관련 커밋
  - fix: 버그 수정 관련 커밋
  - docs: 문서 추가, 수정 관련 커밋
  - style: code formatting, 세미콜론 누락 등 스타일 관련 커밋 (코드 변경 없음)
  - refactor: 리팩토링 관련 커밋 (Production Code)
  - test: 테스트 코드 관련 커밋 (Test Code)
  - chore: 빌드, 패키지 관련 커밋

  > 요즘은 정보 압축을 위해 [Gitmoji](https://gitmoji.dev/)를 사용하는 Commit Convention도 있습니다.

- ### 제목(subject)

  - 제목은 명령문(동사원형) 형태로 작성합니다.
  - 마지막에는 `.` 을 찍지 않습니다.
  - 제목은 최대 50자를 넘지 않도록 주의합니다.
  - 영어의 경우 첫 글자는 대문자로 입력합니다.
  - 제목과 본문 사이에 한 줄 띄워 분리합니다.
  - 커밋 타입과 함께 작성합니다.

- ### 본문(body)

  - 선택사항이므로 모든 commit에 본문 내용을 작성할 필요는 없습니다.
  - 평서문으로 커밋에 대해 상세내역을 작성합니다.
  - 한 줄에 72자를 넘지 않도록 주의합니다.
  - **_어떻게(How)_** 보다 **_무엇을(What), 왜(Why)_** 에 맞춰 작성합니다.

- ### 꼬리말(footer)

  - 선택사항이므로 모든 commit에 본문 내용을 작성할 필요는 없습니다.
  - Issue tracker ID를 작성할 때 사용합니다.
  - 주로 Closes(종료), Fixes(수정), Resolves(해결), Ref(참고), Related to(관련) 키워드를 사용합니다.

### 레퍼런스

- [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)
- [Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)
- [좋은 git 커밋 메시지를 작성하기 위한 7가지 약속](https://meetup.toast.com/posts/106)
- [How to Write a Git Commit Message](https://cbea.ms/git-commit/)
- [좋은 git commit 메시지를 위한 영어 사전](https://blog.ull.im/engineering/2019/03/10/logs-on-git.html)
- [Github commit 메시지 규칙](https://junhyunny.github.io/information/github/git-commit-message-rule/)
