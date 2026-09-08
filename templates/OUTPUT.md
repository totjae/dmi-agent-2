# Agent Output Template

아래 wrapper와 필드명은 유지한다. `STAGE_REPORT` 안의 표현과 추가 항목은 에이전트 프롬프트에 맞게 자유롭게 작성할 수 있다.

```text
[DMI_RUN_META]
SCHEMA_VERSION: DMI_AGENT_v1
AGENT_ID:
DATE:
RUN_TIME_KST:
ANALYSIS_TIME_KST:
DATA_CUTOFF_KST:
PROMPT_PATH: /prompt/AGENT_PROMPT.md
PROMPT_COMMIT:
RUN_TYPE: NORMAL
RERUN_SEQUENCE: 0
[/DMI_RUN_META]

[STAGE_REPORT]
# 당일 종목 발굴 결과

에이전트 프롬프트가 요구하는 전체 분석을 작성한다.

[STAGE_RESULT]
SCHEMA_VERSION: DMI_AGENT_v1
AGENT_ID:
DATE:
RUN_TIME_KST:
TOP_COUNT:
TOP:
1|Name|Code|Market|Rank|ExpectedMoveFE|Confidence|CoreReason
2|...
3|...
4|...
5|...
[/STAGE_RESULT]
[/STAGE_REPORT]
```

## 공통 필드

- `ExpectedMoveFE`: 전일 KRX 정규장 종가 대비 당일 정규장 예상 고가 상승 구간. 프롬프트가 다른 예측 단위를 사용하면 그대로 기록하되 정의를 본문에 밝힌다.
- `Confidence`: `LOW / MEDIUM / HIGH`
- `CoreReason`: 한 줄 핵심 근거. 텍스트 안에 `|`를 사용하지 않는다.
- 후보가 5개보다 적으면 실제 후보만 기록한다.
- 후보가 없으면 `TOP_COUNT: 0`, `TOP: NONE`으로 기록한다.
