

3F 오디토리움 2:20 PM ~ 2:50 PM
최성우 클라우드팀 솔루션즈 아키텍트 매니저, GS Neotek

## Agents for Amazon Bedrock 소개
### 챗봇 수요 증가
* 예약 기능
* 주소 검색 기능
* 알림 기능
### 기존 RAG 기법의 문제점
* 최신 정보를 가지고 있지 않음
* 업로드한 문서 정보까지만 참조 가능
### 최신기술 Agent 필요성 
* 언어모델 학습
* 추론

### Agent란?
* REACT 기법을 기반으로 구현됨
* 질문>사고>행동>관찰 단계로 이루어짐
### REACT 기법 구현 방법
![](https://i.imgur.com/QRWs5ow.jpeg)


### AGENT 동작 방식
* Question: agent 런타임 endpoint 통해 인입된 쿼리
* Thought: 여러 의도가 포함된 사용자 요청 이해, 작업 분할, 작업 간 Orchestration 자동화
* Action: lambda 를 통한 함수 호출
* 근거 도출

### 작업 구성 요소
* Open API 스키마
	* 함수 오케스트레이션을 위한 요청 및 응답 정보를 YAML, JSON 형식으로 정의
	* 스키마 내 주요하게 사용되는 필드
* Lambda 함수
	* Agent 호환 요청/응답형식으로 작성

## 최적화 기법 및 Lesson Learned

신규로 Agent 생성하는 경우 CDK 사용 권장.
git aws-samples/generative-ai-cdk-contstructs
![](https://i.imgur.com/Xe1sg9J.jpeg)


### Lambda Web Adapter
* 기존 java, spring 서버를 사용하고 있다면 먼저 lambda로 작업하고 그다음 Agent 로 변경하는 것을 추천한다.
| 생각: AWS 화면 말고도 코드로도 따로 AWS 인스턴스 생성이 가능하구나..!

### Self Managed Rag : 관리형 Rag에 더하여 직접 람다 함수를 통해 커스텀 구축
* 필요성
	* LLM 호출 많아질수록 비용이 비싸짐
	* 특정 도메인 용어는 이해하지 못함
	* 대규모 문서의 경우 성능 저하
* 커스텀
	* Self Managed Rag를 통해 이 문제점 해결

모든 작업이 Lambda를 통해 실행되기 때문에 멱등성 보장 필요
* 멱등성: 중복 결제 방지
* AWS Lambda Powertools 활용


## 셀프서비스 봇 사례
### Agent가 사용자 의도 분류, 의도와 일치하는 작업 수행
* 검색봇: Hybrid Search, 캐시, 히스토리를 기반으로 빠른 답변
* 주소봇: 가까운 주소 검색, 예약
* 예약봇: 고객이 예약 요청 시 예약테이블 업데이트
* 알림봇: 예약 완료되면 고객에게 SMS 발송
## Wrap up
* LLM빠른 발전 -> 모두 테스트하기에는 역부족
* AWS는 이를 관리할 수 있게 해줌 -> Agents for Amazon bedrock
* Agent를 통한 real word 작업의 자동화
* 기존 백엔드와 Agent 긴밀한 통합을 통한 새로운 비즈니스 가치 창출 가