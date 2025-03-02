---
layout: post
title: "DB Failover 시 대응하기 위한 AWS Advanced JDBC Driver"
date: 2025-03-01
excerpt: "AWS Advanced JDBC Driver에 대해 알아보자"
categories: DB
---

서비스를 고객들에게 안정적으로 제공하기 위해서는 서비스의 가용성이 중요합니다. 서비스의 고가용성을 위한 방안 중 DB를 Primary와 Replica 로 구성하는 방안이 있습니다. 

Primary와 Replica 로 구성하면 고가용성 뿐만 아니라 부하 분산, 확장성 등의 장점이 있지만, 이번에는 주제에 맞게 고가용성 측면으로 이야기해보려고 합니다.

<br/>

## DB를 Primary-Replica 로 구성하면 왜 고가용성일까?

서비스를 운영할 때 가장 중요한 것 중 하나가 **가용성(Availability)**입니다. 갑작스러운 장애나 예기치 못한 상황에서도 시스템이 계속 동작해야, 고객들이 불편을 겪지 않고 서비스를 이용할 수 있기 때문이죠.

그럼 어떻게 가용성을 높일 수 있을까요? 그 대표적인 방법 중 하나가 바로 DB 이중화입니다. 보통 **Primary(메인 DB)**와 **Replica(스탠바이 DB)**로 구성을 하는데, 이때 장애가 발생했을 경우 Failover 절차를 통해 ‘메인 DB가 죽었더라도 스탠바이 DB로 빠르게 역할을 전환’하여 서비스를 이어나갈 수 있게 됩니다.

### 단일 장애 지점을 없앤다

DB가 단 하나만 존재한다면, 해당 DB에 문제가 생기는 순간 서비스 전체가 중단될 위험이 있습니다. 이를 **단일 장애 지점**이라고 부르는데, DB 이중화를 통해 이런 단일 장애 지점을 제거하거나 최소화할 수 있습니다.

### 장애 발생 시 빠른 전환으로 다운타임 최소화

Primary DB가 갑자기 장애를 일으켜 서비스가 중단되는 상황이라도, Replica가 대기하고 있다면 자동화된 모니터링과 Failover 프로세스를 통해 곧바로 트래픽을 이어받아 처리할 수 있습니다.

자동화된 Failover를 사용하면 장애 감지 후 매우 짧은 시간 안에 Role이 전환되어, 사용자 입장에선 거의 서비스가 끊기지 않은 듯 이용할 수 있습니다.

> 💡 Failover 란? 
<br/>
시스템(서버, DB, 네트워크 등)에 장애가 발생했을 때, 정상적인 서비스 운용이 가능한 대체 자원(서버나 DB 인스턴스 등)으로 전환하여 서비스를 지속하는 과정을 의미합니다.

### 고가용성을 위한 핵심 전략

결과적으로 Primary와 Replica를 구성하는 것은 고가용성 전략에서 가장 기본적이면서도 중요한 요소입니다.

- 장애를 확실히 감지하고 Failover를 빠르게 할 수 있다는 점이 핵심이며,
- 서비스 운영 시 다운타임을 크게 줄여 **비즈니스 연속성**을 보장하는 데 도움이 됩니다.

<br/>

## Failover 시 다운타임을 최소화하는 방안은 무엇이 있을까?

DB 이중화를 했어도 Failover 과정에 다운타임을 존재합니다. 그렇다면 Failover 시 다운타임이 존재하는 이유와 비즈니스 연속성을 보장하기 위해 다운타임을 최소화하는 방법에 대해 알아보겠습니다.

### 일반적인 자동 Failover 메커니즘의 한계

#### DNS 기반 전환 지연

- 많은 Failover 솔루션은 DNS 레코드(예: CNAME, A 레코드)를 업데이트하여 새 Primary에 연결을 유도합니다.
- 하지만 DNS의 TTL(Time To Live) 문제나, 글로벌 DNS 서버 간의 전파 지연으로 인해, 애플리케이션이 실제로 새 Primary로 넘어가려면 몇 초에서 수십 초까지 지연이 생길 수 있습니다.

#### 커넥션 풀(Connection Pool) 갱신 지연

- 일반 JDBC 드라이버나 Connection Pool 라이브러리는 DB 연결이 끊겼다는 사실을 인지하는 시점부터 새로운 DB 정보를 받아오기까지 시간이 추가로 소요됩니다.
- 자동 Failover 구조에서는 장애 감지 → 새 Primary 선출 → DNS/엔드포인트 변경 → 연결 재확립 순으로 절차가 이어져, 중간 과정들이 쌓여 다운타임이 길어질 수 있습니다.

#### 수동 개입(Manual Intervention)의 가능성

- 자동 Failover가 완벽히 설정되지 않았거나, 예기치 못한 오류가 발생하면 인적 개입이 필요한 경우가 있습니다.
- 이 경우 확실한 복구가 가능하지만, 그만큼 다운타임은 길어질 수 있습니다.

이처럼 일반적인 자동 Failover 방식은 DNS 기반 전환 지연, 커넥션 풀 갱신 지연 등의 문제가 있습니다. 그렇다면 이러한 문제를 해결하기 위해 AWS에서 제공하는 AWS Advanced JDBC Wrapper 살펴보겠습니다.

<br/>

## AWS Advanced JDBC Driver의 FailoverPlugin 동작 방식

AWS Advanced JDBC Driver는 전통적인 JDBC 드라이버와 달리 **Plugin 기반 아키텍처**를 채택하고 있습니다. 이를 통해 다양한 기능을 Plugin 형태로 손쉽게 확장할 수 있는데, 그중에서도 FailoverPlugin은 DB 연결이 불안정하거나 완전히 끊긴 상황을 자동으로 감지하고, 새로운 Primary를 찾아 재연결해주는 핵심 역할을 담당합니다.<br/>
(이외에도 PerformancePlugin, LoggingPlugin 등 여러 Plugin을 연동할 수 있지만, 여기서는 FailoverPlugin에 집중해보겠습니다.)

<div align=center>
  <img width="700" src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/db-failover/image-1.png"/>
</div>

AWS Advanced JDBC Wrapper의 FailoverPlugin 동작 방식은 다음과 같습니다.

### 1. DB 클러스터의 토폴리지 캐싱

- JDBC Wrapper는 DB 클러스터의 상태(Primary와 Reader 인스턴스 정보)를 주기적으로 조회 및 캐싱
- 장애가 발생하면 실시간으로 새로운 Primary를 파악하여 즉각 연결
> 일반적인 Failover보다 빠른 이유: DNS 전파를 기다릴 필요 없음

### 2. Primary Instance 탐색 및 자동 재연결

- DB 클러스터에서 failover 발생하면, FailoverPlugin은 Topology Cache 정보를 활용하거나 DB 클러스터 Topology에 대해 주기적으로 쿼리하여 최신정보를 가져와 새로운 Primary Instance를 자동으로 식별
> 일반적인 Failover보다 빠른 이유: 애플리케이션이 직접 탐색하는 것이 아니라, Wrapper가 자동으로 처리

### 3. DB Instance에 직접 연결

- 클라이언트는 기존의 DB Endpoint를 통해 연결하려고 하지만, Wrapper는 내부적으로 토폴로지 정보를 활용해 새로운 Primary Instance의 실제 호스트로 직접 연결
- DB 엔드포인트(DNS) 변경을 기다릴 필요 없이, 내부적으로 새 Primary의 실제 호스트를 찾아 직접 연결
> 일반적인 Failover보다 빠른 이유: DNS 업데이트로 인한 지연 없음

### 4. Reader Instance 로드밸런싱

- Reader Endpoint의 경우도 마찬가지로, Wrapper가 캐싱된 Topology 정보를 사용해 적절한 Reader Instance로 트래픽을 분산
> 일반적인 Failover보다 빠른 이유: Reader 노드 간 로드밸런싱까지 자동화

정리하자면, AWS Advanced JDBC Wrapper는

- 클러스터 상태를 실시간으로 인식하고,
- 플러그인 기반의 Failover 로직을 통해 장애 상황을 빠르게 포착하며,
- **DNS 전파 지연**이나 수동 개입을 최소화하도록 설계되어

결과적으로 **수 초 내**로 DB 연결을 복구할 수 있습니다. 일반적인 자동 Failover 솔루션과 비교했을 때, 특화된 FailoverPlugin 덕분에 좀 더 빠르고 투명하게 작동한다는 점이 가장 큰 차이점이라고 할 수 있습니다.

<br/>

## 마치며

고가용성을 위해 Primary-Replica 구조를 적용하더라도, Failover 과정에서 다운타임이 발생할 수 있습니다.
특히 DNS 전파 지연과 커넥션 풀 갱신 지연 문제가 주요 원인으로 작용합니다.

이를 해결하기 방안으로 AWS Advanced JDBC Wrapper 가 있지만, 어떻게 가능하게 하는지 궁금하여 조사하며 포스팅을 했는데 저와 같이 궁금하신 분들에게 도움이 되시길 바랍니다.

감사합니다. :)

<br/>

## 레퍼런스

- [aws-advanced-jdbc-wrapper](https://github.com/aws/aws-advanced-jdbc-wrapper)
- [Failover Plugin](https://github.com/aws/aws-advanced-jdbc-wrapper/blob/main/docs/using-the-jdbc-driver/using-plugins/UsingTheFailoverPlugin.md)
