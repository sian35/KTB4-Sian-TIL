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

### 2. Agent 방식
LLM이 상황을 판단해서 다음 행동이나 도구 호출을 선택하는 방식