# 프로젝트 지침

## 환경
- 기본: `python3 -m venv .venv && ./.venv/bin/pip install -r requirements.txt`
  (`python3.12-venv` 미설치로 실패하면 `uv venv .venv && uv pip install -p .venv -r requirements.txt`)
- 테스트: `./.venv/bin/python -m pytest -q test_checks.py`

## 변경 반영 절차 (PR + merge)
- 모든 변경은 `main`에 직접 push하지 않고, 작업 브랜치 생성 -> 커밋 -> push -> PR 생성 -> merge까지 에이전트가 직접 수행한다.
- 대상 저장소: `bluedbsx/project-auto-trading-scanner` (base: `main`).
- merge 전 `test_checks.py`를 실행해 통과를 확인한다. 실패 시 merge하지 않고 보고한다.
- merge 후 로컬 `main`을 최신화하고 작업 브랜치를 정리한다.

## 사용자 사전 작업
- 사용자가 직접 해야 하는 작업(자격 증명 발급, 본인 인증, 과금 리소스 생성 등)은 `MYTODO.md`에 기록하고 최신 상태로 유지한다.

## 보안
- 실주문(`--live`, `TOSS_LIVE_TRADING=CONFIRM`)은 에이전트가 실행하지 않는다.
- 자격 증명, 계좌 번호(accountSeq), 잔고 등 개인 금융 정보를 파일/커밋/PR/로그에 넣지 않는다.
- 감사 결과와 미해결 위험은 `SECURITY_AUDIT.md` 참고.
