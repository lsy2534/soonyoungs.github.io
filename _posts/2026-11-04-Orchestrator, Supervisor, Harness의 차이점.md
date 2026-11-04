---
layout: post
title: Orchestrator, Supervisor, Harness의 차이점
date: 2026-11-04 00:00 +0800
last_modified_at: 2026-11-04 16:00:00 +0800
tags: [AI, Agent, Orchestrator, Supervisor, Harness]
categoie: [AI, Agent Architecture]
topic: true
---

**\*\*Orchestrator vs Supervisor vs Harness\*\***!
{: .message }

## 개요

AI Agent 시스템을 설계할 때 **Orchestrator**, **Supervisor**, **Harness**는 모두 실행을 관리하는 것처럼 보이지만, 담당하는 책임과 관리 범위가 다르다.

세 개념은 반드시 별도의 프레임워크나 객체로 구현해야 하는 것은 아니다. LangGraph와 같은 프레임워크 안에서 각각의 역할을 구현하거나, 일부 역할을 하나의 컴포넌트에 통합할 수도 있다.

## 핵심 차이점

| 구분 | Orchestrator | Supervisor | Harness |
|---|---|---|---|
| 핵심 역할 | 전체 작업 흐름 조율 | 다음 작업·Agent 판단 및 분배 | Agent 실행 환경과 안정성 관리 |
| 주요 질문 | 어떤 순서로 작업을 진행할까? | 지금 상황에서 누구에게 무엇을 맡길까? | Agent가 안전하고 안정적으로 실행되는가? |
| 판단 방식 | 사전에 정의된 흐름 또는 조건 | 규칙 또는 LLM 기반 동적 판단 | 정책·제한·검증·운영 로직 |
| 관리 대상 | 단계, 노드, Agent 간 연결 | 작업 선택, 라우팅, 위임 | 실행 시간, 재시도, 오류, 권한, 비용, 로그 |
| 대표 기능 | 순차 실행, 병렬 실행, 조건 분기 | Agent 선택, 작업 재분배, 완료 판단 | Timeout, Retry, Checkpoint, 검증, Observability |
| LangGraph 구현 예 | `StateGraph`, `add_edge`, `add_conditional_edges` | Supervisor 노드, LLM 라우터 | `RetryPolicy`, Checkpointer, 실행 래퍼, 모니터링 |
| 필수 여부 | 복잡한 흐름에서 유용 | 동적 분배가 필요할 때 유용 | 운영 안정성과 통제가 필요할 때 유용 |

## 쉽게 이해하는 비유

뉴스 분석 서비스를 하나 만든다고 가정한다.

- **Orchestrator**: 뉴스 수집 → 분석 → 보고서 작성 → 저장이라는 전체 업무 순서를 관리한다.
- **Supervisor**: 뉴스 내용을 보고 정치·경제·시장 Agent 중 필요한 분석 담당자를 선택한다.
- **Harness**: Agent가 무한 반복하거나 API 오류로 실패하지 않도록 실행 횟수, 시간, 권한, 로그, 결과 검증 등을 관리한다.

## 함께 사용하는 구조

```text
                    Harness
       실행 제한 · 오류 처리 · 검증 · 로깅
                      |
                      v
                Orchestrator
             전체 작업 흐름 조율
                      |
                      v
                 Supervisor
          다음 Agent 선택 및 작업 분배
                      |
           +----------+----------+
           |          |          |
           v          v          v
       Politics    Economy     Market
        Agent       Agent       Agent
           |          |          |
           +----------+----------+
                      |
                      v
               Final Report
                      |
                      v
              Harness 검증·종료
```

> 실제 구현에서는 Supervisor가 Orchestrator의 워크플로우 안에 하나의 노드로 포함될 수 있으며, Harness가 실행 흐름 일부를 직접 담당할 수도 있다.

## LangGraph 관점에서 구분하기

| 개념 | LangGraph에서의 구현 예시 |
|---|---|
| Orchestrator | `StateGraph`로 노드와 엣지를 구성하고 전체 흐름 정의 |
| Supervisor | LLM 또는 규칙으로 다음 Agent를 결정하는 노드 |
| Harness | Graph 실행 래퍼, `RetryPolicy`, `recursion_limit`, Checkpointer, 로깅·검증 코드 |

## 설계 시 기억할 점

1. 세 개는 **서로 대체 관계라기보다 서로 다른 책임을 가진 개념**이다.
2. 작은 프로젝트는 Orchestrator와 기본적인 실행 통제만으로도 충분할 수 있다.
3. 작업 분배가 동적으로 바뀌어야 할 때 Supervisor를 도입한다.
4. 실제 운영 환경에서는 Harness를 통해 실패 복구, 실행 제한, 관측 가능성, 결과 검증을 강화한다.
5. Harness는 단순 재시도 코드만 의미하지 않는다. Agent가 작업할 수 있는 환경과 실행·검증 체계를 설계하는 더 넓은 개념이다.

## 한 줄 요약

- **Orchestrator** = 전체 작업 흐름을 조율한다.
- **Supervisor** = 상황에 따라 다음 작업 또는 Agent를 선택한다.
- **Harness** = Agent의 실행 환경과 안정성·통제·검증을 관리한다.
