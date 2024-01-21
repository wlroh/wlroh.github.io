---
layout: post
title: "Intellij 롬복, 빌드 관련 초기 세팅"
date: 2022-03-02
excerpt: "롬복 설정과 빌드관련 속도 향상을 위한 세팅법"
categories: IDE
tags: Intellij
---

### Build / Run / Run Test 관련 설정

- 설정창 - Gradle 검색
  - [Build and run] 확인
    > Build and run using: Gradle -> IntelliJ IDEA 변경<br/>
    > Run tests using: Gradle => IntelliJ IDEA 변경

### Lombok 세팅

- 설정창 - Plugins
  - Lombok 검색 후 설치
- 설정창 - Annotation Processors 검색
  - `Enable annotation processing` 체크 상태 확인
