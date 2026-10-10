> 사용자 제공 참고자료(2026-09-11). 원문의 모델 사양·성능 비교·공식 출처 주장은 별도 검증되지 않았으며, 실제 환경과 확인된 공식 정보가 우선한다.

## 탐색용 목차

- 1. 문서 역할
- 2. GPT-5.6 프롬프팅 원칙
- 3. GPT-6 Astra 프롬프팅 원칙
- 4. GPT-5.6 vs GPT-6 요약
- 5. Chat과 Work의 프롬프트 설계 차이
- 6. Work용 Agentic Prompt Blocks
- 7. Prompt Architecture 선택 규칙
- 8. Legacy Prompt → 최신 Prompt 변환 규칙
- 9. 프롬프트 설계 GPT의 모델 적응 규칙
- 10. 실전 템플릿
- 11. 최종 체크리스트
- 12. 공식 참고자료

# [지식 파일] OpenAI 최신 프롬프팅 가이드 — GPT-5.6·GPT-6 / Chat·Work

> 기준일: 2026-09-11  
> 목적: OpenAI 최신 모델 특성을 프롬프트 설계에 반영하기 위한 운영 지식  
> 적용 범위: ChatGPT Chat 및 ChatGPT Work 중심  
> 제외 범위: Codex 전용 프롬프팅·코딩 에이전트 세부 구현

---

## 1. 문서 역할

이 문서는 공통 프롬프트 엔지니어링 원리를 대체하지 않는다.

- 공통 설계 원리 → 「프롬프트 엔지니어링 마스터 가이드 2026」 우선
- GPT-5.6·GPT-6 특성, Chat·Work 차이 → 본 문서 우선
- 최신성이 중요한 기능·모델 사양 → OpenAI 공식 문서 확인 우선

핵심 변화는 다음과 같이 요약한다.

> GPT-5.6: 목표·맥락·제약·성공기준을 명확히 하고, 불필요한 절차·반복 지시를 줄인다.  
> GPT-6: 여기에 자율성·질문 조건·승인 경계·완료 기준 등 Agent 운영 규칙을 더 정교하게 설계한다.

---

## 2. GPT-5.6 프롬프팅 원칙

### 2.1 Lean Prompt

GPT-5.6에서는 반복적인 지시, 불필요한 예시, 과도하게 긴 도구 설명을 줄이는 것이 중요하다.

실무 원칙:
- 같은 규칙은 한 번만 명확하게 작성
- 목적에 기여하지 않는 예시는 제거
- 동일 의미의 "반드시", "꼭", "절대" 반복 금지
- 기존 모델에서 쓰던 장황한 프롬프트를 그대로 이식하지 않음

주의:
OpenAI가 보고한 프롬프트 간소화 효과 수치는 특정 내부 coding-agent 평가의 사례이며, 일반 Chat에서 동일한 개선율을 보장하지 않는다. 방향성만 참고한다.

### 2.2 Outcome-focused Prompting

세부 작업 절차보다 다음 항목을 우선한다.

- Goal
- Context
- Hard Constraints
- Success Criteria
- Required Evidence
- Output Format

모델이 사용자의 의도와 원하는 작업 수준을 맥락에서 더 잘 추론하므로 모든 단계를 일일이 지정할 필요가 줄었다.

### 2.3 Important Ambiguity Only

질문은 결과를 실질적으로 바꾸는 중요한 모호성이 있을 때 사용한다.

- 사소한 선택 → 합리적으로 판단
- 핵심 목표·범위·평가기준이 달라질 수 있음 → 질문
- 이미 대화에 있는 정보 → 재질문 금지

### 2.4 Reasoning은 네이티브 기능 우선

"더 깊이 생각해라", "모든 사고과정을 단계별로 보여라"와 같은 지시를 기본값으로 사용하지 않는다.

모델의 Reasoning/Thinking 기능이 제공되면 해당 설정을 우선 고려하고, 프롬프트는 결과 계약에 집중한다.

### 2.5 Autonomy & Approval Boundary

다단계 작업에서는 다음을 구분할 수 있다.

- 안전하고 범위 내인 작업
- 외부 영향이 있는 작업
- 비가역적 작업
- 비용·범위가 확대되는 작업

Work 환경에서 특히 중요하다.

같은 승인 규칙을 여러 곳에서 반복하지 않는다. 승인 정책은 가능하면 한 곳에서 정의한다.

---

## 3. GPT-6 Astra 프롬프팅 원칙

GPT-6 Astra는 장기·다단계 작업에서 더 일관되게 작업할 수 있고, 결과를 크게 바꿀 추가 정보가 부족하면 GPT-5.6보다 질문하려는 경향이 강하다.

따라서 GPT-6 프롬프팅의 핵심은 단순한 "좋은 질문 작성"보다 **Agent 행동 조율**에 가까워진다.

### 3.1 Initiative & Follow-through

모델이 계획이나 중간 결과에서 불필요하게 멈추지 않고 사용자의 실제 목적까지 수행하도록 지시할 수 있다.

권장 원칙:
- 명백한 실행 요청이면 설명만 하지 말고 결과까지 완성
- 사소한 빈칸은 합리적으로 가정
- 결과를 실질적으로 바꿀 불확실성만 질문
- 승인 없이 가능한 준비 작업은 먼저 수행
- 완료 조건이 충족될 때까지 범위 내 작업 지속

### 3.2 Clarification Rule

다음처럼 명시하면 효과적이다.

- 결과를 크게 바꿀 정보가 부족하면 질문
- 그 외에는 맥락에 근거한 합리적 가정으로 진행
- 가정이 중요하면 최종 결과에서 간단히 밝힘

질문을 "항상 먼저" 하도록 설계하지 않는다.

### 3.3 Instruction Following & Priority

GPT-6는 긴 컨텍스트와 여러 지시를 잘 활용하지만, 그만큼 서로 다른 지시의 충돌 관리가 중요하다.

원칙:
- 같은 규칙을 여러 문서에서 다르게 반복하지 않음
- 안정적 행동 규칙은 Instructions에 배치
- 모델별 세부 지식은 Knowledge에 배치
- 사용자 최신 지시와 과거 예시가 충돌하지 않도록 관리
- 프롬프트 안에서 핵심 규칙의 우선순위를 불필요하게 복잡하게 만들지 않음

### 3.4 Personality & Writing Style

GPT-6는 상세하고 구조화된 응답을 생성하는 경향이 있으므로 원하는 스타일을 구체적 행동으로 지정한다.

예:
- 결론 먼저
- 800자 이내
- 표는 비교가 필요한 경우만
- 전문용어는 필요한 경우 설명
- 동일 주장 반복 금지
- 도입부 최소화

"전문적으로", "친절하게" 같은 형용사만으로 스타일을 제어하지 않는다.

### 3.5 Delegation

실제 multi-agent/subagent 기능이 존재하는 환경에서만 Delegation 규칙을 사용한다.

구분:
- Multi-Perspective: 한 모델이 여러 관점에서 분석
- Subagent Delegation: 실제 여러 agent에게 독립 과업을 위임

일반 Chat에서 Subagent 기능이 확인되지 않은 경우 이를 가정하여 프롬프트를 만들지 않는다.

### 3.6 Verification Calibration

모든 작업에 동일한 검증 강도를 적용하지 않는다.

검증 수준은 다음에 따라 조절한다.

- 오류 비용
- 최신성 요구
- 외부 실행 여부
- 결과의 되돌림 가능성
- 의사결정 영향
- 사실·수치 비중

가벼운 생성 작업에는 과도한 검증 루프를 만들지 않는다.

---

## 4. GPT-5.6 vs GPT-6 요약

| 항목 | GPT-5.6 | GPT-6 Astra |
|---|---|---|
| 기본 방향 | Lean + Outcome-focused | Outcome + Agent Operating Rules |
| 절차 지시 | 축소 | 모델 자율 분해 적극 활용 |
| 질문 | 중요한 모호성만 | 질문 성향이 강해 Clarification Rule 중요 |
| 자율성 | 다단계 작업에서 중요 | 핵심 설계 요소 |
| 승인 경계 | 명시 권장 | Work에서 특히 중요 |
| 완료 지속성 | Proactive/Persistent | Long-task follow-through 강화 |
| 스타일 | 출력 형식 명확화 | 세밀한 writing behavior 제어 |
| Multi-agent | 필요 시 | 지원 환경에서 delegation 조절 |
| 핵심 사고 | Prompt Engineering | Prompt + Agent Operating Policy |

---

## 5. Chat과 Work의 프롬프트 설계 차이

### 5.1 Chat

주요 용도:
- 질문 답변
- 요약
- 분석
- 리서치
- 전략
- 문서 작성
- 콘텐츠 생성
- 아이디어 개발

기본 구조:

```text
Goal
Context
Constraints
Success Criteria
Output
```

필요 시:
- Role
- Input
- Evidence
- Verification
- Process
- Examples

핵심 질문:
> "좋은 답은 무엇인가?"

Chat에서는 Autonomy·Approval 같은 Agent 규칙을 습관적으로 넣지 않는다.

---

### 5.2 Work

주요 용도:
- 웹·브라우저 기반 조사
- 여러 파일 분석
- 앱·플러그인 활용
- 반복적인 다단계 작업
- 문서·파일 산출물 제작
- 여러 시스템을 오가는 업무 수행

기본 구조:

```text
Goal
Context
Constraints
Autonomy
Clarification Rule
Approval Boundary
Verification
Success Criteria
Completion Criteria
Deliverables
```

모든 항목을 항상 사용하지 않는다.

핵심 질문:
> "무엇을 달성할 것인가?"  
> "어디까지 스스로 해도 되는가?"  
> "언제 멈추고 물어봐야 하는가?"  
> "무엇을 확인하면 완료인가?"

---

## 6. Work용 Agentic Prompt Blocks

### 6.1 Autonomy

모델이 별도 승인 없이 수행해도 되는 범위.

예:
- 공개 웹 검색
- 제공 파일 읽기
- 데이터 정리
- 비교 분석
- 초안 작성
- 가역적인 문서 편집
- 범위 내 추가 조사

권장 표현:
> 목표 달성에 필요한 조사·분석·초안 작성은 범위 내에서 스스로 판단하여 진행한다.

### 6.2 Clarification Rule

질문이 필요한 조건.

예:
> 결과를 실질적으로 바꿀 핵심 정보가 부족한 경우에만 질문하고, 그 외에는 합리적인 가정을 두고 진행한다.

### 6.3 Approval Boundary

사용자 확인이 필요한 행동.

대표 예:
- 외부 발송
- 게시·공개
- 삭제
- 구매
- 결제
- 계약
- 중요한 계정·권한 변경
- 사용자가 정한 범위를 넘어서는 작업

원칙:
가능한 준비 작업을 먼저 완료하고, 실제 영향이 발생하는 직전에 필요한 승인을 받도록 설계한다.

### 6.4 Verification

완료 전 확인해야 할 내용.

예:
- 최신 날짜 확인
- 출처 신뢰성
- 중복 제거
- 조건 충족 여부
- 링크 유효성
- 산출물 형식

### 6.5 Completion Criteria

작업을 끝냈다고 판단할 조건.

예:
- Top 5 사례 선정 완료
- 각 사례에 출처 포함
- 비교표 완성
- 보고서 파일 생성
- 저장 위치 확인

"최선을 다하라"보다 완료 조건을 구체적으로 정의한다.

### 6.6 Deliverables

최종 산출물을 명확히 한다.

예:
1. 요약
2. 상세 보고서
3. 표
4. Google Doc
5. 데이터 파일

---

## 7. Prompt Architecture 선택 규칙

### A. 간단 Chat
사용:
- Goal
- Context
- Output

### B. 일반 Chat
사용:
- Goal
- Context
- Constraints
- Success Criteria
- Output

### C. Research Chat
사용:
- Goal
- Context
- Scope
- Evidence
- Verification
- Success Criteria
- Output

### D. Agentic Work
사용:
- Goal
- Context
- Scope/Constraints
- Autonomy
- Clarification
- Approval
- Verification
- Success Criteria
- Completion
- Deliverables

프롬프트 설계 GPT는 템플릿 전체를 기계적으로 복사하지 않고, 작업에 필요한 블록만 선택한다.

---

## 8. Legacy Prompt → 최신 Prompt 변환 규칙

### Role 중심 → Goal 중심
기존:
> 당신은 세계 최고 수준의 전문가다.

개선:
> 무엇을 달성해야 하는지와 좋은 결과 기준을 명확히 한다.

Role은 특정 관점이 필요할 때만 유지한다.

### Task Steps 중심 → Success Criteria 중심
기존:
> 1단계 A → 2단계 B → 3단계 C

개선:
> 반드시 만족해야 할 결과와 검증 기준을 먼저 정의한다.

절차가 품질·안전·재현성에 중요할 때만 Process를 추가한다.

### 무조건 질문 → Conditional Clarification
기존:
> 작업 전에 필요한 질문을 먼저 하라.

개선:
> 결과를 크게 바꿀 중요한 정보가 부족한 경우에만 질문한다.

### Self-Correction 반복 → Calibrated Verification
기존:
> 항상 최소 1회 이상 자기비판하고 다시 작성하라.

개선:
> 목표·제약·성공기준 충족 여부를 점검하고 필요한 부분만 수정한다.

### Multi-Agent Role-play → 개념 분리
기존:
> 여러 전문가 에이전트가 토론한다.

개선:
- 여러 관점 검토가 필요하면 Multi-Perspective
- 실제 agent 기능이 있으면 Subagent Delegation
- 지원되지 않는 기능을 가정하지 않음

### 장황한 강조 → 규칙 1회
기존:
> 반드시, 절대로, 꼭, 다시 한 번...

개선:
> 중요한 규칙을 한 번 명확하게 정의한다.

---

## 9. 프롬프트 설계 GPT의 모델 적응 규칙

사용자가 모델을 지정한 경우:

1. 해당 모델의 공식 최신 가이드를 우선 확인
2. 공통 Prompt Architecture는 유지
3. 모델 특성 때문에 달라지는 부분만 조정
4. 모델 자체 기능과 중복되는 기법은 제거
5. 기능이 확인되지 않으면 단정하지 않음

사용자가 모델을 지정하지 않은 경우:
- 일반적인 최신 ChatGPT 사용을 가정
- 모델 버전을 과도하게 프롬프트에 고정하지 않음
- 결과에 큰 차이가 있을 때만 모델·모드 질문

---

## 10. 실전 템플릿

### 10.1 Chat 기본형

```text
# Goal
[달성할 결과]

# Context
[대상·목적·배경]

# Constraints
[핵심 조건·제외사항]

# Success Criteria
[좋은 결과의 판단 기준]

# Output
[구조·분량·형식·문체]
```

### 10.2 Research Chat

```text
# Goal
[조사의 최종 목적]

# Context
[대상·용도·시간 범위]

# Scope
[포함/제외 범위]

# Evidence
[우선할 출처와 근거 수준]

# Verification
[최신성·사실성 검증]

# Success Criteria
[분석이 만족해야 할 기준]

# Output
[보고서 구조·분량·인용 방식]
```

### 10.3 Work 기본형

```text
# Goal
[최종 업무 결과]

# Context
[업무 배경·관련 파일·데이터·시스템]

# Scope & Constraints
[범위·제외사항·필수 조건]

# Autonomy
[스스로 판단·실행 가능한 범위]

# Clarification
[질문이 필요한 조건]

# Approval Boundary
[승인이 필요한 외부·비가역 행동]

# Verification
[완료 전 확인 사항]

# Success Criteria
[품질 기준]

# Completion Criteria
[완료 판단 기준]

# Deliverables
[최종 산출물]
```

---

## 11. 최종 체크리스트

### Chat
- Goal이 명확한가?
- Context가 충분한가?
- Constraints가 과도하지 않은가?
- Success Criteria가 있는가?
- Output이 명확한가?
- 불필요한 절차를 강제하지 않는가?
- 같은 지시가 반복되지 않는가?

### Work
위 항목에 추가:
- Autonomy 범위가 필요한가?
- 질문 조건이 필요한가?
- Approval Boundary가 명확한가?
- Verification 수준이 적절한가?
- Completion Criteria가 있는가?
- 외부 영향이 있는 행동 전에 필요한 승인이 정의되어 있는가?
- 계획 설명에서 멈추지 않고 실제 결과까지 수행하도록 되어 있는가?

---

## 12. 공식 참고자료

본 문서는 다음 OpenAI 공식 모델 가이드의 프롬프팅 원칙을 기준으로 작성되었다.

- OpenAI Model Guidance — GPT-5.6  
  https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.6

- OpenAI Model Guidance — GPT-6 Astra  
  https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6-astra

모델·제품 기능은 변경될 수 있으므로 최신성이 중요한 설계에서는 공식 문서를 다시 확인한다.
