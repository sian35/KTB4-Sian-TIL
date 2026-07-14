# LangGraph
State, Node, Edge로 구성된 그래프 구조로 설계하고 실행 
- State : 그래프가 실행되는 동안 계속 들고 다니는 데이터
- Node : 실제 작업을 수행하는 단계
- Edge : 어떤 Node로 이동할지 정하는 연결선  

Low-level Orchestration : 개발자가 실행 흐름을 그래프로 직접 설계할 수 있다.  

## 기본 흐름
State 정의 → Node 작성 → Edge 연결 → compile → invoke

## Lang Chain과의 차이점
### Lang Chain : 단순하고 직선적인 흐름  
한 방향 흐름을 빠르게 만들 수 있지만 에이전트를 만들거나 복잡한 워크플로를 만들기 어렵다. 
### Lang Graph : 상태를 유지하면서 흐름이 갈라지거나 반복되는 구조

## LangGraph 방식
### 1. Workflow 방식
개발자가 실행 순서와 분기 조건을 코드로 미리 정의하는 방식  
핵심은 LLM이 전체 실행 순서를 결정하지 않는 것  
#### 장점
실행 경로가 고정되어 있어 예측하기 쉽고 비용 통제, 디버깅이 쉬운 AI 파이프라인 생성이 가능하다.
#### 적합한 대표 시나리오
- RAG 파이프라인
- 문서 처리 자동화
- 데이터 변환 파이프라인
- 사람의 확인이 필요한 흐름

### 2. Agent 방식
LLM이 상황을 판단해서 다음 행동이나 도구 호출을 선택하는 방식
#### 동작 루프 (ReAct)
1. 추론
2. 행동
3. 관찰
4. 재판단 : 종료 또는 추가 행동

#### 적합한 대표 시나리오
- 대화형 어시스턴트
- 멀티스텝 리서치
- 도구 선택이 필요한 업무 자동화
- 문제 해결형 고객 지원

#### 주의 사항
- 실행 경로가 매번 달라질 수 있다.
- 판단 오류가 다음 단계로 연쇄된다.
- 반복 호출로 인해 비용과 응답 지연이 커진다.
- 무한 루프  
    - 해결 : 반복 횟수 제한, 종료 조건, 사람 승인 지점 설계
- 판단 예측 불가
- 프롬프트 인젝션 : 외부 문서나 웹 검색 결과에 악의적 지시가 담아있다면 문제 발생.  
    - 해결 : 시스템 프롬프트와 외부 데이터를 명확히 분리하기, 도구 결과를 무조건 신뢰하지 않는 설계.

#### Tool Node

#### Agent Node


### 3. 실무
Workflow와 Agent 방식을 혼합해서 사용한다. 동적 의사결정이 반드시 필요한 부분만 Agent로 사용

## LangGraph 구현
### StateGraph
상태 스키마를 사용해 그래프를 구축하는 핵심 빌더 클래스이다.

#### Graph State Schema (상태 스키마)
그래프에서 모든 노드가 공유하는 State의 키, 타입, 병합 규칙을 선언한 상태 구조에 대한 정의다.  
즉, State의 키와 타입을 정리하는 설계도로 이해하면 된다.

#### Reducer
특정 상태 키의 업데이트를 기존 값과 어떻게 합칠지 정의하는 함수다.  
Reducer를 사용하면 값이 덮어씌워지지 않고 추가, 누적됨.
```
Annotated[list[str], operator.add]
```

#### MessagesState
##### add_messages
ID 기반 메시지 관리를 지원한다.

##### MessagesState 확장
MessagesState를 상속받아 필요한 필드를 추가한다.

#### Edge
순차 연결 : add_edge
조건부 분기 : add_conditional_edges(소스, [라우팅 함수](), 매핑)
#### Compile
빌더 객체(StateGraph)를 실행 가능한 그래프 객체(CompiledStateGraph)로 변환하는 메서드
##### 파라미터
- checkpointer
- interrupt_before
- interrupt_after
- store
- debug
- name

##### graph.invoke

##### graph.stream

## LangGraph 핵심
### Tool Calling
LLM이 판단하여 호출해야 할 도구의 이름과 인자를 구조화된 형태로 응답에 담아 반환하는 기능이다. 즉, 판단 결과를 구조화된 도구 호출 요청으로 반환.  
호출 요청을 하는 단계이지, 실행하는 단계가 아니다. 이 두 단계를 분리하여 실행 전에 검증할 수 있다.  
`AIMessage`의 `tool_calls` 필드에 담긴다.

#### Tool Node
실제로 도구 함수를 실행하고 결과를 ToolMessage로 감싸서 메시지 리스트에 추가해주는 사전 구성 노드이다.
`ToolNode(tools=tools)`

#### Agent Node
LLM을 호출하여 다음 행동(도구 호출 또는 최종 답변)을 결정한다.

### Routing
다음에 실행할 노드를 선택하는 분기 패턴이다.
#### 라우팅 함수
현재 State를 보고 다음 경로를 선택

### Parallelization & Send
#### 차이점
- Parallelization : 그래프를 만들 때 병렬 노드의 개수가 미리 정해진다.
- Send : 실행 중에 만들어진 데이터 개수에 따라 병렬 실행 개수가 달라진다.  
하나의 노드 정의를 여러 개의 실행 인스턴스로 확장하는 방식.  
Map-Reduce 패턴에 사용됨.

### Loop

#### Loop Termination

### ReAct Pattern
