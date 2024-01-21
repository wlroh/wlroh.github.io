---
layout: post
title: "Git 명령어"
date: 2022-02-18
excerpt: "Git 명령어 모음"
categories: Git
tags: 명령어
---

- ### Git 생성 관련

| 목록                   | 명령어                                                     | 내용 설명                                                                              |
| ---------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 새로운 로컬저장소 생성 | $ git init                                                 | .git 디렉토리 생성</br>(폴더를 만뜬 후, 그 안에서 명령 실행 => 새로운 로컬저장소 생성) |
| 원격 저장소 가져오기   | $ git clone https://github.com/{USERNAME}/{REPOSITORY}.git | 원격 저장소 소스코드 clone                                                             |

<br/>

- ### git 상태 관련

| 목록                    | 명령어       | 내용 설명                         |
| ----------------------- | ------------ | --------------------------------- |
| git 상태                | $ git status | 작업중인 디렉토리의 변경사항 확인 |
| 변경된 Staged 파일 확인 | $ git diff   | Stage가 변경된 파일을 확인        |
| 로그보기                | $ git log    | 변경 이력 확인                    |

<br/>

- ### 브랜치 작업 관련

| 목록                    | 명령어                    | 내용 설명 |
| ----------------------- | ------------------------- | --------- |
| 로컬 브랜치 보기        | $ git branch              | -         |
| 로컬과 원격 브랜치 보기 | $ git branch -av          | -         |
| 브랜치 변경             | $ git checkout <branch>   | -         |
| 브랜치 생성             | $ git branch <new_branch> | -         |
| 브랜치 삭제             | $ git branch -d <branch>  | -         |
| 현재 커밋에 태그 달기   | $ git tag <tag_name>      | -         |