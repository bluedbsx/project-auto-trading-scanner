# 보안 감사 (2026-11-08)

대상: 외부(benyco-dev)에서 가져온 코드 전체 (`*.py`, `toss/`, `cloud/`, `dashboard/`, `docs/`, CI, Dockerfile).
범위: 자격 증명/계좌 정보의 외부 유출 경로, 공급망, 배포 구성.

## 결론
악성 코드나 숨은 외부 전송은 발견되지 않았다. 코드가 접속하는 외부 호스트는
`openapi.tossinvest.com`(토스 API), `en.wikipedia.org`(S&P500 목록), yfinance(Yahoo 시세)뿐이며,
자격 증명은 토스 토큰 엔드포인트로만 전송된다. git 이력 전체에서 하드코딩된 비밀값도 없다.
다만 운영 구성에서 개인 금융 정보가 공개될 수 있는 지점이 있다.

## 발견 사항

| # | 심각도 | 위치 | 내용 | 조치 |
|---|---|---|---|---|
| 1 | 높음 | `docs/index.html:71` | 대시보드가 원작자 버킷 `storage.googleapis.com/paper-trading-dashboard/paper.json`을 기본 데이터로 읽는다. 내 Pages에 원작자 데이터가 표시된다. | 배포 전 본인 버킷 URL로 교체 |
| 2 | 높음 | `dashboard/publish.sh` | `paper.json`(자본, 평가금액, 보유 종목, 손익, 체결 내역)을 **공개** GCS 버킷에 올린다. 실계좌 장부를 쓰면 자산 규모와 포지션이 누구에게나 공개된다. | 모의 장부 전용으로 유지하거나 비공개 버킷 + 인증 사용 |
| 3 | 중간 | `cloud/install-trading-cron.sh` | VM의 `/opt/auto-trading/.env`에 client_secret이 평문 저장되어 `--env-file`로 컨테이너에 전달된다. | 파일 권한 600, 가능하면 Secret Manager에서 실행 시 주입 |
| 4 | 중간 | `.github/workflows/docker-build.yml` | Docker Hub 이미지가 기본 공개 저장소로 push된다. 이미지에 비밀값은 없으나(명시적 COPY만 사용) 코드와 전략이 공개된다. | 필요 시 private 저장소 |
| 5 | 중간 | `requirements.txt`, workflow | 의존성이 하한만 지정(`>=`)되고 액션이 태그로 고정되어 공급망 변조에 노출된다. | 버전 고정/해시, 액션 SHA 고정 |
| 6 | 낮음 | `docs/index.html:73,149` | `?data=` 파라미터로 임의 URL의 JSON을 받아 `innerHTML`에 넣는다. 조작된 링크로 XSS 가능(페이지에 비밀값은 없어 영향 제한적). | 출처 제한 또는 textContent 사용 |
| 7 | 낮음 | `docs/index.html:9` | jsdelivr CDN 스타일시트를 SRI 없이 로드. | `integrity` 속성 추가 |
| 8 | 낮음 | `toss/toss_client.py:211`, 로그 | 계좌 목록, accountSeq, 매수 가능 금액이 stdout/`/var/log/auto-trading/*.log`에 기록된다. | 로그 공유 시 주의 |

## 확인된 양호 사항
- 자격 증명은 환경변수 또는 `.env`에서만 읽고, `.env`/`*.key`/`*.pem`/`*.csv`는 `.gitignore`, `.dockerignore` 처리됨
- 실주문은 `--live` + `TOSS_LIVE_TRADING=CONFIRM` 이중 확인, 기본 dry-run
- CI 권한 `contents: read`, 컨테이너 비루트 실행, VM 보안 강화 스크립트(IAP SSH, 최소 권한 SA) 제공
