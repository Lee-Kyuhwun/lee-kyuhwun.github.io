---
layout: post
title: "패스트캠퍼스 SNS 만들기 프로젝트-1"
date: 2023-12-25 17:03:36 +0000"
tags: [FastCampus]
categories: Spring, FastCampus
---

간단한 SNS 만들기 프로젝트 시작

인강을 보면서 코드를 작성해보고 따라쳐본다.

## 시스템 아키텍쳐

![시스템 아키텍쳐](/assets/img/fastcampus/simple_sns/sns_architecture.png "시스템 아키텍쳐")

### 기술스택

Spring Boot3
, Kafka
, Redis
, PostgresQL
, Heroku
, Github Action

## Flow Chart

## 회원가입

![회원가입 플로우차트](/assets/img/fastcampus/simple_sns/SignUp.png)

## 로그인

![로그인](/assets/img/fastcampus/simple_sns/login_flowchart.png)

포스트 작성

{% mermaid %}
sequenceDiagram;
autonumber;
client ->> server: 포스트 작성 요청;
alt 성공한 경우;
server ->> db : 포스트 저장 요청;
db -->> server : 저장 성공 반환;
server -->> client: 성공 반환;
else 로그인하지 않은 경우;
server -->> client: reason code와 함께 실패 반환;
else db 에러;
server ->> db : 포스트 저장 요청;
db -->> server : 에러 반환;
server -->> client: reason code와 함께 실패 반환;
else 내부 에러;
server -->> client: reason code와 함께 실패 반환;
{% endmermaid %}

포스트 삭제

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 포스트 삭제 요청
    alt 성공한 경우
    server ->> db : 포스트 삭제 요청
    db -->> server : 삭제 성공 반환
    server -->> client: 성공 반환
    else 로그인하지 않은 경우
    server -->> client: reason code와 함께 실패 반환
    else db 에러
    server ->> db : 포스트 삭제 요청
    db -->> server : 에러 반환
    server -->> client: reason code와 함께 실패 반환
    else 내부 에러
    server -->> client: reason code와 함께 실패 반환
    end

```

포스트 수정

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 포스트 수정 요청
    alt 성공한 경우
    server ->> db : 포스트 수정 요청
    db -->> server : 수 성공 반환
    server -->> client: 성공 반환
    else 로그인하지 않은 경우
    server -->> client: reason code와 함께 실패 반환
    else db 에러
    server ->> db : 포스트 수정 요청
    db -->> server : 에러 반환
    server -->> client: reason code와 함께 실패 반환
    else 내부 에러
    server -->> client: reason code와 함께 실패 반환
    end

```

피드 목록

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 피드 목록 요청
    alt 성공한 경우
    server ->> db : 포스트 목록 요청
    db -->> server : 목록 쿼리 성공 반환
    server -->> client: 목록 반환
    else 로그인하지 않은 경우
    server -->> client: reason code와 함께 실패 반환
    else db 에러
    server ->> db : 포스트 목록 요청
    db -->> server : 에러 반환
    server -->> client: reason code와 함께 실패 반환
    else 내부 에러
    server -->> client: reason code와 함께 실패 반환
    end

```

좋아요 기능 : User A가 B 게시물에 좋아요를 누른 상황

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 좋아요 요청
    alt 성공한 경우
    server ->> db : db update 요청
    db -->> server : 성공 반환
    server -->> client: 성공 반환

    else 로그인하지 않은 경우
    server -->> client: reason code와 함께 실패 반환

    else db 에러
    server ->> db : db update 요청
    db -->> server : 에러 반환
    server -->> client: reason code와 함께 실패 반환

    else B 게시물이 존재하지 않는 경우
    server ->> db : db update 요청
    db -->> server : 에러 반환
    server -->> client: reason code와 함께 실패 반환

    else 내부 에러
    server -->> client: reason code와 함께 실패 반환
    end

```

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 좋아요 요청
    alt 성공한 경우
    server ->> db : 좋아요 누를 수 있는 조건 체크
    db -->> server : 응답
    server --) kafka : event produce request
    server->>client: 성공 반환
    kafka --) server : event consume
    server ->> db : db update 요청
    db -->> server : 성공 반환

    else 로그인하지 않은 경우
    server->>client: reason code와 함께 실패 반환

    else db 에러
    server ->> db : 좋아요 누를 수 있는 조건 체크
    db -->> server : 응답
    server --) kafka : event produce request
    server->>client: 성공 반환
    kafka --) server : event consume
    loop db update 실패시 최대 3회 재시도 한다
        server ->> db : db update 요청
    end
    db -->> server : 실패 반환

    else B 게시물이 존재하지 않는 경우
    server ->> db : 좋아요 누를 수 있는 조건 체크
    db ->> server : 에러 반환
    server->>client: reason code와 함께 실패 반환

    else 내부 에러
    server->>client: reason code와 함께 실패 반환
    end

```

1. 댓글 기능 : User A가 B 게시물에 댓글을 남긴 상황

```mermaid
sequenceDiagram
autonumber
client ->> server: 댓글 작성
    alt 성공한 경우
    server ->> db : db update 요청
    db ->> server : 성공 반환
    server->>client: 성공 반환

    else 로그인하지 않은 경우
    server->>client: reason code와 함께 실패 반환

    else db 에러
    server ->> db : db update 요청
    db ->> server : 에러 반환
    server->>client: reason code와 함께 실패 반환

    else B 게시물이 존재하지 않는 경우
    server ->> db : db update 요청
    db ->> server : 에러 반환
    server->>client: reason code와 함께 실패 반환

    else 내부 에러
    server->>client: reason code와 함께 실패 반환
    end

```

```mermaid
  sequenceDiagram
    autonumber
    client ->> server: 댓글 작성
    alt 성공한 경우
    server ->> db : 좋아요 누를 수 있는 조건 체크
    db -->> server : 응답
    server --) kafka : event produce request
    server->>client: 성공 반환
    kafka --) server : event consume
    server ->> db : db update 요청
    db -->> server : 성공 반환

    else 로그인하지 않은 경우
    server->>client: reason code와 함께 실패 반환

    else db 에러
    server ->> db : 댓글 작성 조건 체크
    db -->> server : 응답
    server --) kafka : event produce request
    server->>client: 성공 반환
    kafka --) server : event consume
    loop db update 실패시 최대 3회 재시도 한다
        server ->> db : db update 요청
    end
    db -->> server : 실패 반환

    else B 게시물이 존재하지 않는 경우
    server ->> db : 댓글 작성 조건 체크
    db ->> server : 에러 반환
    server->>client: reason code와 함께 실패 반환

    else 내부 에러
    server->>client: reason code와 함께 실패 반환
    end

```

알람 기능 : User A의 알람 목록에 대한 요청을 한 상황

```mermaid
sequenceDiagram
autonumber
client ->> server: 알람 목록 요청
    alt 성공한 경우
    server ->> db : db query 요청
    db ->> server : 성공 반환
    server->>client: 성공 반환

    else 로그인하지 않은 경우
    server->>client: reason code와 함께 실패 반환

    else db 에러
    server ->> db : db query 요청
    db ->> server : 에러 반환
    server->>client: reason code와 함께 실패 반환

    else 내부 에러
    server->>client: reason code와 함께 실패 반환
    end

```