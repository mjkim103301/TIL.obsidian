3F 오디토리움 3:20PM ~ 4:00PM
김범준 솔루션즈 아키텍트, AWS
박상하 Lead PM, 센드버드
이기범 Lead ML 엔지니어, 센드버드


## 생성형 AI와 아마존 Bedrock
### 생성형 AI란?
* 사람이 만든 콘텐츠에 가까운 콘텐츠를 생성할 수 있음
* 대용량 데이터를 사전 학습한 데이터를 기반으로 정보 제공함.

### Foundation model?
* 기존ML흐름과 대비해서 과정이 간략화됨.

### Amazon Bedrock
* foundation model을 기반으로 개발한 서비스
* 단일한 API로 다양한 ML모델 사용 가능
* 장점: serverless lambda 활용
	* 다양한 foundation model사용 가능, 다양한 AWS 모델과 결합 가능

## Sendbird의 새로운 도전
생성형 AI 챗봇과 Rule Based 챗봇이 모두 필요함
* 지능형 AI 로 비정형화된 답변을 원하는 경우도 있었지만
* 특정 시나리오를 통해 Rule based 응답이 필요한 경우도 있었음.


## 기업 챗봇을 위한 여정
RAG: 사실기반 답변을 도출하는 workflow
Guardrails for bedrock : LLM  output이 고객에게 가기 전 답변 모니터링함.

## 더 안전한 AI 챗봇 with guardrails
![](https://i.imgur.com/ZtDwDpl.jpeg)

![](https://i.imgur.com/3ZDSDMA.jpeg)

### 2가지 상황에서 모니터링함.
* 사용자의 input > LLM
* LLM output > 사용자
![](https://i.imgur.com/LxdeSNP.jpeg)

Input>Denied Topics > Content Filters > PII Redaction > LMM Response > WordFilter > Output
* Denied Topics: 정치, 경제, 경쟁사같은 민감한 단어, 질문들을 사전에 차단함.
* Contents Filter: 프롬프트 응답 필터 (증오, 모욕, 성, 폭력적 단어 임계값 설정 가능)


## Sendbird의 계획
* 축적된 채팅 데이터를 활용한 tuning
* 고객의 데이터베이스에 저장된 데이터로 효과적인 응답 제공
* 다양한 플랫폼과 연동: Web, In-App, SaaS