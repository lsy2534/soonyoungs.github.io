---
layout: post
title: AOP
date: 2024-02-23 00:00 +0800
last_modified_at: 2024-02-23 00:08:25 +0800
tags: [Spring Boot AOP ]
categoie: [Spring Boot AOP]
topic:  true
---
**면접 내용**!
{: .message }

> 면접을 진행하면서 설명하지 못한 부분들이 많아 다시 공부하는 위치에서 직접 찾아보고 적어보고 내용을 정리해서, 다음에 또 같은 질문시 대답하기 위해 적어 본다.
> 블로그에서 찾아본 내용을 내느낌대로 한번 간략하게 정의한다.

1. 개요

AOP란?
관점 지향 프로그램이라고도 불린다.
어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나눠보고 그 관점을 기준으로 각각 모듈화 하겠다는 의미.
* 핵심적인 관점 : 개발자가 적용하고자 하는 핵심 비즈니스 로직
* 부가적인 관점 : 핵심 로직을 수행하기 위해 필요한 DB연결(JDBC), 로깅, 파일 입출력 등...

## 한줄 요약
결국 AOP는 공통 부분 코드를 모듈화하여 분리하여 핵심기능을 설계 및 구현할 때 객체지향적인 가치를 지킬 수 있도록 도와주는 것입니다.

