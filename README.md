# DMI Agent 2

한국 주식 당일 후보 발굴 에이전트의 독립 실행 저장소입니다.

## 구조

- `prompt/AGENT_PROMPT.md`: 사용자가 직접 추가하는 에이전트 프롬프트
- `WORKFLOW.md`: 독립 실행·저장 규칙
- `templates/OUTPUT.md`: 공통 결과 형식
- `runs/YYYY-MM-DD/`: 실행 결과

이 저장소의 예측 작업은 다른 DMI 에이전트 저장소와 리뷰 저장소를 읽지 않습니다.
리뷰 작업은 `totjae/dmi-market-pipeline`에서 두 에이전트의 결과를 비교합니다.
