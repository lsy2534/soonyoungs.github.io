---
layout: post
title: langGraph 기초 학습
date: 2026-11-04 00:00 +0800
last_modified_at: 2026-11-04 00:00:00 +0800
tags: [Claude, AI, langGraph]
categoie: [AI]
topic: true
---


# LangGraph 학습 정리 (1강~29강)

> 학습 방식: 개념 설명 → 코드/예시 → 미션 → 채점 및 보충  
> 목적: LangGraph 기본 개념부터 RAG·Multi-Agent·운영·평가까지 이해하고 실전 프로젝트에 적용

## 학습 진행 요약

- **1~5강:** LangGraph 기초, State, Conditional Edge, LangChain 연동
- **6~9강:** Tool Calling, MessagesState, Checkpoint, Human-in-the-loop
- **10~18강:** RAG, Reranker, 검색 평가, Answer Evaluation, RAGAS
- **19~23강:** Multi-Agent, Supervisor, Send API, Retry, Subgraph
- **24~29강:** Streaming, FastAPI, 통합 프로젝트, Production, 테스트·평가
- **30강:** 최종 프로젝트 아키텍처 리뷰 및 미션 (아직 진행 전)

---

# 1강. LangGraph 개요

## 핵심 개념
- **Graph:** 노드와 연결 관계로 구성된 실행 구조
- **Node:** 실제 작업을 수행하는 함수
- **Edge:** Node 사이의 실행 흐름
- **State:** Graph 실행 중 공유하는 데이터
- **compile():** Graph를 실행 가능한 형태로 구성
- **invoke():** Graph를 실행하고 최종 State를 반환

## 기본 구조
```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    question: str
    answer: str

def answer_node(state: State):
    return {"answer": f"질문: {state['question']}"}

builder = StateGraph(State)
builder.add_node("answer", answer_node)
builder.add_edge(START, "answer")
builder.add_edge("answer", END)

graph = builder.compile()
result = graph.invoke({"question": "안녕하세요"})
```

---

# 2강. State

## 핵심
- State는 Graph Node들이 공유하는 데이터 구조다.
- Node는 전체 State를 반환할 필요 없이 **변경할 필드만 반환**할 수 있다.
- 기본적으로 같은 키는 새 값으로 덮어쓴다.
- Reducer를 지정하면 업데이트를 누적하거나 사용자 정의 방식으로 합칠 수 있다.

## 리스트 누적 예시
```python
from typing import Annotated, TypedDict
import operator

class State(TypedDict):
    answers: Annotated[list, operator.add]

def node_a(state):
    return {"answers": ["HR 결과"]}
```

`operator.add`는 리스트를 연결(concatenation)한다.

---

# 3강. Conditional Edge

## 핵심
- 조건 함수가 다음 Node를 결정한다.
- `add_conditional_edges()`로 라우팅을 구성한다.
- 조건 함수의 반환값과 라우팅 mapping을 연결할 수 있다.
- RAG에서 검색 결과 평가, 검색 재작성, 최대 검색 횟수 제한 등에 활용한다.

```python
def route(state):
    if state["relevant"]:
        return "generate"
    return "rewrite"

builder.add_conditional_edges(
    "evaluate",
    route,
    {
        "generate": "generate",
        "rewrite": "rewrite",
    }
)
```

---

# 4~5강. LangChain과 LLM 연결

## 핵심
- LangChain은 모델, Prompt, Retriever, Tool 등을 구성하는 데 활용한다.
- LangGraph는 여러 작업의 흐름과 상태를 관리한다.
- Prompt와 LLM을 Chain으로 연결하고 Node 안에서 실행할 수 있다.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

llm = ChatOpenAI(model="gpt-4.1-mini")
prompt = ChatPromptTemplate.from_template(
    "다음 질문에 답해줘: {question}"
)
chain = prompt | llm

def llm_node(state):
    result = chain.invoke({"question": state["question"]})
    return {"answer": result.content}
```

---

# 6강. Tool Calling / Agent

## 핵심
- `@tool`로 도구를 정의할 수 있다.
- `bind_tools()`로 모델에 도구 사용 가능성을 알려준다.
- `ToolNode`가 실제 도구를 실행한다.
- `tools_condition` 등으로 도구 호출 여부에 따라 분기한다.
- Agent → Tool → Agent 흐름으로 여러 번 도구를 사용할 수 있다.

**중요:** AIMessage의 `tool_calls`는 도구 실행 요청이지 실행 결과가 아니다. 실제 실행 결과는 ToolMessage로 전달된다.

---

# 7강. AIMessage / ToolMessage / MessagesState

## 핵심
- **AIMessage:** 모델이 생성한 메시지. 도구 호출 요청도 포함할 수 있다.
- **ToolMessage:** 실제 Tool 실행 결과.
- **tool_call_id:** Tool 요청과 결과를 연결한다.
- **MessagesState:** 메시지 목록을 reducer로 누적 관리한다.

---

# 8강. Memory와 Checkpoint

## 핵심
- **State:** 현재 Graph 실행 상태
- **Checkpoint:** 특정 시점의 State 저장본
- **Checkpointer:** Checkpoint 저장·복구 담당
- **thread_id:** 대화/실행 흐름을 구분하는 식별자
- `MemorySaver`: 프로세스 메모리에 저장하므로 재시작 시 사라질 수 있다.
- SQLite/PostgreSQL Saver: 지속 저장에 활용할 수 있다.

```python
config = {
    "configurable": {
        "thread_id": "conversation-001"
    }
}
```

---

# 9강. Human-in-the-loop / interrupt

## 핵심
- `interrupt()`를 통해 실행을 일시 중지하고 사람의 승인/입력을 기다릴 수 있다.
- Checkpointer가 필요하다.
- `Command(resume=...)`로 재개할 수 있다.
- Interrupt 이후 Node가 재실행될 수 있으므로 외부 부작용은 멱등성 등을 고려해야 한다.

---

# 10~11강. RAG + LangGraph

## 기본 흐름
1. Retriever로 문서 검색
2. 검색 결과 평가
3. 필요하면 질문 재작성
4. 다시 검색
5. 근거 기반 답변 생성

## 핵심
- State에 검색 결과, relevance, search_count 등을 저장할 수 있다.
- 최대 검색 횟수를 제한한다.
- 근거가 부족하면 답변을 억지로 생성하지 않고 **정보 부족을 명시**한다.

---

# 12강. Reranker 개념

## 핵심
- Vector Search는 후보 문서를 빠르게 찾는다.
- Reranker는 질문과 후보 문서를 다시 비교해 순위를 조정한다.
- Reranker는 후보군에 없는 문서를 새로 찾아내지 못한다.

---

# 13강. CrossEncoder Reranker

## 핵심
- CrossEncoder는 질문과 문서를 함께 입력해 관련도를 평가한다.
- BiEncoder보다 정밀한 재평가에 유리하지만 후보마다 계산하므로 느릴 수 있다.
- 점수는 일반적으로 확률이나 정확도(예: 95%)로 해석하면 안 된다.

```python
from sentence_transformers import CrossEncoder

model = CrossEncoder("BAAI/bge-reranker-v2-m3")
pairs = [(question, doc.page_content) for doc in docs]
scores = model.predict(pairs)

ranked_docs = sorted(
    zip(docs, scores),
    key=lambda x: x[1],
    reverse=True
)
```

---

# 14강. 검색 평가 지표

## Recall@K
정답 문서 중 상위 K개 검색 결과에 포함된 비율.

## MRR
첫 번째 정답 문서의 순위를 역수로 계산하고 평균한 값.

## NDCG
관련도 등급과 순위를 함께 고려하는 지표. 상위 결과의 순서가 중요하다.

---

# 15강. Golden Dataset

## 핵심
- 질문, 기대 답변, 정답 문서/근거 등을 포함하는 평가 데이터셋.
- Retriever, Reranker, Prompt, 모델 변경 전후 품질 비교에 사용한다.
- 품질 회귀(regression)를 확인하는 기준이 된다.

---

# 16강. Python Recall@K / MRR 구현

## 핵심
- 검색 결과에 정답 문서가 포함됐는지 계산한다.
- 첫 정답의 순위를 기준으로 reciprocal rank를 계산한다.
- 여러 질문에 대한 평균으로 시스템을 비교할 수 있다.

---

# 17강. Answer Evaluation

## Faithfulness
답변이 제공된 근거에 충실한가?

## Answer Relevance
답변이 질문에 적절하게 대응하는가?

## Answer Correctness
답변이 실제 정답/기대 답변과 얼마나 일치하는가?

---

# 18강. RAGAS

## 주요 평가 지표
- **Faithfulness:** 답변과 컨텍스트 근거의 일치
- **Answer Relevancy:** 답변의 질문 적합성
- **Context Precision:** 검색된 컨텍스트의 관련성/순위 품질
- **Context Recall:** 필요한 근거가 검색 컨텍스트에 포함됐는지

평가 라이브러리의 API는 버전에 따라 달라질 수 있으므로 실제 구현 시 설치 버전 문서를 확인한다.

---

# 19강. Multi-Agent / Supervisor

## 핵심
- 단일 Agent: 하나의 Agent가 여러 작업을 처리
- Multi-Agent: 역할별 Agent가 전문 작업을 수행
- Supervisor: 질문을 분석해 적절한 Agent를 선택·조정
- 각 Agent 결과를 State에 모아 최종 답변을 구성한다.

예시:
- HR Agent
- IT Agent
- Finance Agent

---

# 20강. LLM 기반 Supervisor

## 핵심
- Structured Output으로 Agent 선택 결과를 구조화한다.
- Pydantic 모델과 `with_structured_output()` 등을 사용할 수 있다.
- 선택 가능한 Agent를 제한하고 결과를 검증한다.
- 여러 분야 질문이면 여러 Agent를 선택할 수 있다.
- Structured Output은 라우팅 형식을 보장하는 데 도움을 주지만 RAG 답변 정확성까지 보장하지는 않는다.

```python
from typing import Literal
from pydantic import BaseModel

class SupervisorDecision(BaseModel):
    agents: list[Literal["hr", "it", "finance"]]
```

---

# 21강. Send API와 동적 병렬 실행

## 핵심
- `Send`를 사용하면 실행 시점에 Worker 수와 입력을 동적으로 구성할 수 있다.
- Supervisor가 선택한 Agent들을 병렬 실행할 수 있다.
- 결과는 reducer로 누적한다.
- 선택 Agent가 없을 경우 fallback을 고려한다.

```python
from langgraph.types import Send

def route_agents(state):
    return [
        Send("agent_worker", {
            "question": state["question"],
            "agent": agent
        })
        for agent in state["selected_agents"]
    ]
```

## 결과 누적
```python
class AgentState(TypedDict):
    question: str
    selected_agents: list[str]
    answers: Annotated[list[dict], operator.add]
    final_answer: str
```

---

# 22강. Error Handling / Retry

## 핵심
- 일시적 오류: Backoff 기반 제한적 Retry
- 지속 오류: Fallback 또는 실패 상태 기록
- 성공한 Agent 결과는 보존하고 실패한 Agent만 재시도한다.
- Retry 횟수 제한, Timeout, 중복 실행 방지를 고려한다.

## RetryPolicy 예시
```python
from langgraph.types import RetryPolicy

RetryPolicy(
    max_attempts=3,
    initial_interval=1.0,
    backoff_factor=2.0,
    max_interval=10.0,
    jitter=True
)
```

`max_attempts`는 최초 실행을 포함한 전체 시도 횟수다.

## Retry와 Fallback
- Retry: 동일 작업을 다시 수행
- Fallback: 다른 방법/대체 경로로 처리

---

# 23강. Subgraph와 Agent 모듈화

## 핵심
- Subgraph는 Graph 내부에서 다른 Graph를 Node처럼 사용하는 구조다.
- HR/IT/Finance RAG를 각각 독립 Graph로 구성할 수 있다.
- 독립 개발·테스트·재사용이 편리하다.
- 부모 Graph와 Subgraph의 State는 반드시 동일할 필요가 없다.
- State가 다르면 입력/출력 키와 매핑을 적절히 설계해야 한다.

예시:
```python
class MainState(TypedDict):
    question: str
    hr_answer: str
    finance_answer: str

class HRState(TypedDict):
    question: str
    answer: str
```

---

# 24강. LangGraph Streaming

## 주요 stream_mode
- `updates`: Node별 State 업데이트
- `values`: 각 단계의 전체 State
- `messages`: LLM 메시지/토큰 스트리밍
- `custom`: 개발자가 직접 전달하는 이벤트

```python
for chunk in graph.stream(
    {"question": "연차 신청 방법"},
    stream_mode="updates"
):
    print(chunk)
```

LLM 토큰 스트리밍:
```python
for message, metadata in graph.stream(
    {"question": "연차 신청 방법"},
    stream_mode="messages"
):
    if message.content:
        print(message.content, end="")
```

---

# 25강. FastAPI + LangGraph Streaming API

## 핵심
- `graph.astream()`은 비동기 Graph 스트리밍이다.
- FastAPI `StreamingResponse` 또는 SSE 응답과 연결할 수 있다.
- SSE(Server-Sent Events)는 서버에서 클라이언트로 이벤트를 실시간 전달하는 단방향 방식이다.

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.post("/chat")
async def chat(request: ChatRequest):
    async def generate():
        async for message, metadata in graph.astream(
            {"question": request.question},
            stream_mode="messages"
        ):
            if message.content:
                yield message.content

    return StreamingResponse(
        generate(),
        media_type="text/plain"
    )
```

> 실제 서비스에서는 모델 메시지 필터링, SSE 형식(`data: ...\n\n`), 에러 처리, thread_id 등을 추가해야 한다.

---

# 26강. 통합 프로젝트 설계

## 목표
사내 HR/IT/Finance 통합 AI 검색 서비스.

## 전체 흐름
1. FastAPI 요청
2. Main Graph의 Supervisor가 Agent 선택
3. HR/IT/Finance Subgraph 실행
4. 결과 Aggregation
5. 근거 기반 Final Answer 생성
6. SSE Streaming
7. Checkpoint로 대화 상태 유지

## 기술 스택 예시
| 영역 | 기술 |
|---|---|
| API | FastAPI |
| Orchestration | LangGraph |
| LLM | OpenAI |
| Embedding | OpenAI Embeddings |
| VectorDB | Chroma 또는 Qdrant |
| Reranker | BGE Reranker |
| Streaming | SSE |
| Checkpoint | PostgreSQL |
| Evaluation | RAGAS |

---

# 27강. 예외 처리와 실패 상태 관리

## 기본 Worker
```python
def finance_worker(state):
    result = finance_graph.invoke({
        "question": state["question"]
    })

    return {
        "answers": [{
            "agent": "finance",
            "answer": result["answer"]
        }]
    }
```

Finance Graph 실행이 실패하면 예외가 발생하고, 처리하지 않으면 상위 Graph까지 실패할 수 있다.

## try-except 예시
```python
def finance_worker(state):
    try:
        result = finance_graph.invoke({
            "question": state["question"]
        })

        return {
            "answers": [{
                "agent": "finance",
                "answer": result["answer"]
            }],
            "agent_status": {
                "finance": "success"
            }
        }

    except Exception as e:
        return {
            "agent_status": {
                "finance": "failed"
            },
            "errors": [{
                "agent": "finance",
                "error": str(e)
            }]
        }
```

## 중요 구분
- 예외를 그대로 발생시키면 Node RetryPolicy가 재시도할 수 있다.
- 예외를 `except`로 잡고 정상 반환하면 LangGraph는 해당 Node를 성공으로 간주하므로 자동 Retry가 발생하지 않는다.
- 재시도 가능한 오류는 RetryPolicy, 최종 실패는 별도 실패 상태/Fallback으로 관리하는 전략이 유용하다.
- 실패한 Agent 결과를 성공한 것처럼 만들지 않는다.
- 성공한 Agent 결과는 유지하고 실패 사실을 최종 답변에 명시한다.

## State 예시
```python
{
    "answers": [
        {
            "agent": "hr",
            "answer": "연차 신청은 그룹웨어에서 진행합니다."
        }
    ],
    "agent_status": {
        "hr": "success",
        "finance": "failed"
    },
    "errors": [
        {
            "agent": "finance",
            "error": "OpenAI API Timeout"
        }
    ]
}
```

---

# 28강. Production 운영 전략

## 운영 시 고려사항
1. Checkpoint
2. Retry
3. Timeout
4. Observability
5. Idempotency

## LLM Timeout / Retry 예시
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4.1-mini",
    timeout=30,
    max_retries=2
)
```

## Retry 계층 구분
- LLM Retry: 모델 API 호출 재시도
- LangGraph Retry: Node 전체 재실행
- 두 정책이 중첩되면 API 호출 수가 증가할 수 있다.

## Observability
추적할 정보 예시:
```python
{
    "thread_id": "user-123",
    "node": "finance_agent",
    "duration_ms": 3200,
    "status": "success",
    "retry_count": 0
}
```

## PostgreSQL Checkpointer 예시
```python
from langgraph.checkpoint.postgres import PostgresSaver

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup()
    graph = builder.compile(checkpointer=checkpointer)
```

---

# 29강. 테스트와 평가 자동화

## 테스트 계층
- **Node Test:** 개별 Node의 입력/출력 검증
- **Graph Test:** Graph 전체 실행 및 최종 결과 검증
- **RAG Evaluation:** 검색·답변 품질 평가
- **Golden Dataset:** 변경 전후 품질 비교

## Node 테스트 예시
```python
def test_retrieve_node():
    result = retrieve_node({
        "question": "연차 신청 방법"
    })

    assert "documents" in result
    assert len(result["documents"]) > 0
```

## Graph 테스트 예시
```python
def test_hr_graph():
    result = hr_graph.invoke({
        "question": "연차 신청 방법"
    })

    assert result["answer"]
    assert len(result["answer"]) > 0
```

## Golden Dataset 예시
```python
dataset = [
    {
        "question": "미사용 연차 수당 지급 기준은?",
        "expected_answer": "미사용 연차에 대한 수당은..."
    },
    {
        "question": "출장비 정산 방법은?",
        "expected_answer": "영수증 첨부 후..."
    }
]
```

---

# 30강. 최종 프로젝트 (진행 예정)

## 요구사항
사용자 질문:
> 연차 신청 방법과 출장비 정산 방법을 알려줘. 그리고 이전 대화에서 내가 물어본 IT 장애 대응 방법도 다시 알려줘.

시스템 요구사항:
1. Supervisor가 HR, Finance, IT Agent 선택
2. 각 Agent 병렬 RAG
3. 실패 Agent만 재시도
4. 성공 결과 보존
5. 근거 기반 최종 답변 생성
6. SSE 실시간 답변
7. Checkpoint로 대화 상태 유지

## 최종 미션
- Mission 1: Supervisor가 선택할 Agent
- Mission 2: 병렬 실행에 사용하는 LangGraph 기능
- Mission 3: Finance 실패 시 복구 전략
- Mission 4: 이전 대화 기억/이어가기 기능
- Mission 5: FastAPI 실시간 SSE 구성
- Mission 6: Reranker 변경 후 품질 검증 방법

---

# 학습 중 기억할 핵심 정리

| 개념 | 한 줄 요약 |
|---|---|
| State | Graph가 공유하는 데이터 |
| Node | 작업을 수행하는 함수 |
| Edge | 실행 흐름 |
| Conditional Edge | 조건에 따른 라우팅 |
| Reducer | State 업데이트 병합 규칙 |
| ToolNode | Tool 실제 실행 |
| Checkpoint | 실행 상태 저장 |
| thread_id | 대화/실행 흐름 식별 |
| interrupt | 사람의 입력/승인을 위해 중단 |
| Retriever | 후보 문서 검색 |
| Reranker | 후보 문서 재순위화 |
| Recall@K | 정답 문서 검색 포함 비율 |
| MRR | 첫 정답 순위의 역수 평균 |
| Faithfulness | 답변의 근거 충실도 |
| Supervisor | Agent 선택/조정 |
| Send | 동적 병렬 Worker 실행 |
| Subgraph | Graph 내부의 독립 Graph |
| RetryPolicy | Node 재시도 정책 |
| Streaming | 실행/답변을 실시간 전달 |
| SSE | 서버→클라이언트 이벤트 전달 |
| Golden Dataset | 품질 비교 기준 데이터 |

---

## 다음 학습 방향

30강 최종 미션을 완료한 뒤에는 다음 주제로 확장할 수 있다.

- LangGraph 프로젝트 실제 코드 구현
- FastAPI + LangGraph + Qdrant 통합
- Agent별 RAG Subgraph 설계
- PostgreSQL Checkpoint와 대화 Memory
- SSE 프론트엔드 연결
- RAG 평가 자동화 및 LangSmith
- 운영 배포, 모니터링, 비용 최적화
