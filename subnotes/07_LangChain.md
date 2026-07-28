# LangChain
- LLM 애플리케이션을 구성하는 각 단계를 독립적인 표준화된 컴포넌트로 분리하고 파이프라인으로 연결하는 오픈소스 프레임워크다.
- 반복적으로 필요한 LLM 기능을 표준화된 컴포넌트로 빠르게 조립할 수 있기 때문에 개발 속도를 높이고 유지보수 비용을 줄일 수 있고 변경과 확장이 쉬운 구조를 만들 수 있다.

## LangChain 핵심 구성요소

|모듈|역할|주요 컴포넌트|
|---|---|---|
|Model/Messages|LLM과의 입력, 출력을 관리한다.||
|Retrieval/Indexing|외부 데이터를 불러와서 저장하고 검색하는 컴포넌트||
|Chains (LCEL)|컴포넌트를 파이프라인으로 연결||
|Short-term Memory|대화 기록을 저장하고 다음 호출에 주입||
|Tools|외부 도구를 필요시 호출할 수 있게 한다.|@tool|
|Agents|어떤 도구를 사용할지 판단하고 작업을 수행|create_agent|

## 기초
### Messages
Chat Model의 입력과 출력을 구성하는 역할별 메시지 객체의 집합이다.
#### 메시지 타입
`langchain_core.messages`
- SystemMessage
- HumanMessage
- AIMessage
- ToolMessage : Tool 실행 결과를 모델에게 전달

#### BaseMessage - 공통 구조
공통 속성
- content : 본문 텍스트
- type : 메시지의 역할
- response_metadata
- additional_kwargs
- id
- name : 발신자의 이름

### LCEL (LangChain Expression Language)
- 파이프(`|`) 연산자로 Runnable 컴포넌트를 연결하여 체인을 구성하는 문법 체계
- 일관된 방식으로 연결해 코드 구조를 단순화하고 기능 변경,확장,재사용을 빠르고 안정적으로 할 수 있게 해준다.
#### 파이프(`|`) 연산자
`chain = prompt | model | parser`  
프롬프트 생성 &rarr; 모델 호출 &rarr; 텍스트 추출

#### Runnable 인터페이스
공통 메서드 `invoke()`, `stream()`,`batch()`를 가진 인터페이스.

## 프롬프트 구성
### PromptTemplate