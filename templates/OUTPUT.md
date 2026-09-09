# Agent Output Template

TEMPLATE_VERSION: DMI_AGENT_OUTPUT_v1.3

확정된 원본 보고서를 보존하고 아래 저장 구조에 기록한다. 이 템플릿은 새 분석을 요구하지 않는다. 후보·순위·확신도·가격·예상폭을 저장 단계에서 새로 만들거나 변경하지 않는다.
에이전트 프롬프트와 명시적으로 충돌하는 지시는 임의로 우선순위를 정해 우회하지 말고 충돌 내용을 보고한다.

## 저장 구조

아래 꺾쇠 설명은 실제 값으로 대체한다. 본문 전체를 보존하며 결과 캡슐은 한 번만 기록한다.

```text
[DMI_RUN_META]
SCHEMA_VERSION: DMI_AGENT_v1.3
AGENT_ID:
DATE:
RUN_TIME_KST:
SCHEDULED_AT_KST:
STARTED_AT_KST:
ANALYSIS_TIME_KST:
REPORT_COMPLETED_AT_KST:
DATA_CUTOFF_KST:
PROMPT_PATH: /prompt/AGENT_PROMPT.md
PROMPT_COMMIT:
DOCUMENT_VERSIONS:
<아래 문서 버전 규칙의 JSON 배열>
RUN_TYPE: NORMAL
RERUN_SEQUENCE: 0
[/DMI_RUN_META]

[STAGE_REPORT]
<에이전트 프롬프트가 요구하는 전체 원본 보고서>

[STAGE_RESULT]
SCHEMA_VERSION: DMI_AGENT_v1.3
AGENT_ID:
DATE:
RUN_TIME_KST:
TOP_COUNT:
TOP:
<RowNo>|Name|Code|Market|Rank|ExpectedMoveFE|Confidence|CoreReason

OPTIONAL_DETAILS:
<필요한 후보별 선택 블록만 기록. 없으면 NONE>
[/STAGE_RESULT]
[/STAGE_REPORT]
```

## 공통 결과행

- TOP_COUNT는 본문의 핵심 선정 목록 수(0~5). 실제 후보만 한 줄씩 기록한다. 0개이면 TOP: NONE.
- 기존 행 호환성을 위해 RowNo와 Rank를 유지하며 둘은 같아야 한다. Rank는 1부터 연속이며 동일 종목을 중복 기록하지 않는다.
- Name, Code, Market은 본문에서 확인된 값을 그대로 사용한다. Code는 앞자리 0을 보존한 6자리 문자열, Market은 KOSPI/KOSDAQ. 미확인은 N/A이며 저장 단계에서 추정하지 않는다.
- CoreReason은 본문에 있는 핵심 이유의 한 줄 요약. 모든 결과행 자유 텍스트에서 줄바꿈과 구분자 |는 공백으로 치환한다. 원문은 본문에 보존한다.
- ExpectedMoveFE는 오직 전일 KRX 정규장 종가 대비 당일 KRX 정규장 예상 고가 상승률이다. 단위는 %이다. 목표가격, 종가 수익률, 진입가 대비 수익률, 예상 OFE를 이 필드에 넣거나 변환하지 않는다.
- 본문에 이 정의의 예상 FE가 명시되면 구간 또는 값을 그대로 기록한다. 불확실이면 UNCERTAIN. 예측을 제공하지 않으면 N/A. 구간 경계와 달리 새로운 예측을 만들지 않는다.
- Confidence: 낮음→LOW, 보통→MEDIUM, 높음→HIGH로만 정규화한다. 기존 LOW/MEDIUM/HIGH는 유지한다. 미제공·판독 불가는 N/A. 이는 증거의 품질에 대한 판단이며 상승 확률·수익률이 아니다.
- 메타데이터 중 확인 불가능한 값도 N/A로 기록하고 확인한 척 채우지 않는다.

## 선택 필드

선택 블록은 본문에서 이미 제공한 내용을 보존하기 위한 것이다. 존재하지 않는 계획이나 평가를 만들지 않는다. 값이 없는 선택 필드는 나열하지 않고 생략한다. 생략은 낮은 위험·조건 미충족·실패를 의미하지 않는다.

후보별로 필요한 경우 다음 형식을 사용한다.

```text
[CANDIDATE_DETAIL]
Rank: <TOP의 Rank>
ExpectedMoveFEStatus: <PROVIDED / UNCERTAIN / NOT_PROVIDED / UNVERIFIED / NOT_APPLICABLE>
ConfidenceStatus: <PROVIDED / NOT_PROVIDED / UNVERIFIED / NOT_APPLICABLE>
PostOpenRoom: <본문의 개장 후 여력>
TradeRisk: <본문의 위험 수준>
HighRiskOpportunity: <본문에 명시된 고위험 기회 여부>
EntryStatus: <관찰 또는 조건부 진입 검토 등 원문>
TradeType: <본문의 매매 유형>
EntryPlan: <진입 가격·구간과 모든 전제조건>
StopPlan: <손절 가격과 조건>
TargetPlan: <목표 가격과 조건>
RewardRisk: <본문의 비율과 비용 포함 여부>
Invalidation: <취소·무효화 조건>
ExitPlan: <청산 조건>
MissingReasons: <판단 불가라고 명시된 항목과 원문 사유>
[/CANDIDATE_DETAIL]
```

- 위 필드 전체를 채우는 것은 의무가 아니다.
- ExpectedMoveFE 또는 Confidence가 N/A이면 해당 Status는 기록한다. NOT_PROVIDED=본문에 해당 예측 없음, UNVERIFIED=평가를 시도했으나 확인 불가, NOT_APPLICABLE=본문에서 적용 불가로 명시. 추정해 분류하지 않는다.
- UNCERTAIN은 예상 FE의 명시적 불확실성이다. Confidence N/A는 위 결측 상태로 설명한다.
- 가격 계획은 기준 시장·가격 종류·시각·전제조건을 포함해 의미를 보존한다. 숫자만 떼어내지 않는다. 필요한 상세 설명은 본문의 정확한 절 제목을 참조할 수 있다.
- TOP 밖의 보조 관찰군은 원본 본문에 그대로 보존한다. TOP 행으로 승격하거나 TOP_COUNT에 더하지 않는다.

## 검증

본문과 TOP의 종목·순위·개수, FE 정의와 단위, 확신도 매핑을 대조한다.
선택 블록의 Rank가 실제 TOP에 존재하는지 확인한다.
N/A와 UNCERTAIN을 0으로 바꾸지 않는다. 선택 필드 누락을 이유로 원본 보고서에 없는 분석을 추가하지 않는다.

## 시간 필드

DATE와 RUN_TIME_KST는 예약 대상 거래일과 슬롯이며 캡슐에도 동일하게 기록한다. DATA_CUTOFF_KST=SCHEDULED_AT_KST=해당 DATE의 슬롯(+09:00)이다. 실제 시작·판단 확정·문서 완성은 별도 시각으로 기록한다. NORMAL은 번호 0, RECOVERY는 경로의 rerun 번호다. REPORT_COMPLETED_AT_KST는 GitHub 저장 성공시각이 아니다. 최종 보고에는 실제 저장 commit과 재열람 검증 결과를 남긴다.

## 문서 버전 규칙

DOCUMENT_VERSIONS는 다음 객체 세 개의 JSON 배열이다. 역할은 WORKFLOW, AGENT_PROMPT, OUTPUT이며 각 객체는 실제로 읽은 해당 파일을 나타낸다.

```json
[
  {"role":"WORKFLOW","repository":"<현재 저장소>","path":"WORKFLOW.md","commit_sha":"N/A","blob_sha":"<조회에서 확인한 SHA 또는 N/A>","read_at_kst":"<ISO 8601 또는 N/A>","status":"VERIFIED","reason":"NONE"}
]
```

예시 객체를 역할별로 작성한다. VERIFIED는 최소 blob SHA와 실제 읽은 내용의 연결이 확인된 경우이며 그렇지 않으면 UNVERIFIED와 사유를 기록한다. PROMPT_COMMIT은 AGENT_PROMPT 객체의 commit_sha와 동일하게 기록하고 blob SHA를 대신 넣지 않는다. 문서 버전 문자열은 Git SHA의 대체물이 아니다. 현재 보고서 자신의 저장 commit은 사전에 알 수 없으므로 이 목록에 넣지 않고 저장 후 응답으로 보고한다. 버전 기록은 분석 내용을 바꾸는 근거가 아니다.
