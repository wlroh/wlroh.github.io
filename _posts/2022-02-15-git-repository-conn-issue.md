---
layout: post
title: "깃허브 저장소 연동 이슈"
date: 2022-02-15
excerpt: "fatal: remote origin already exists 이슈 해결방안"
categories: Git
tags: github issue
---

- ### Github 저장소 연동 이슈

```shell
$ git remote add origin https://github.com/{username}/{repository}.git
fatal: remote origin already exists.
```

- 원인

  - 이미 원격저장소 정보가 등록되어 있어서 발생되는 이슈
  - 원격저장소 목록 조회 시 이미 다른 원격 저장소가 저장된 것을 알 수 있다.

    ```shell
    $ git remote -v
    ```

- 해결방안

  - 원격 저장소 정보를 지운뒤 다시 등록 해주면 된다.

    ```shell
    $ git remote rm origin
    $ git remote add origin https://github.com/{username}/{repository}.git
    ```
