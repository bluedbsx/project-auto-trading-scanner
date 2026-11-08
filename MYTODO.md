# MYTODO: 사용자가 직접 준비해야 할 작업

로컬 스캔/백테스트/모의 기록은 준비 없이 바로 동작한다 (`.venv` 구성 완료, 테스트 12건 통과).
아래는 자격 증명, 본인 인증, 과금이 걸려 있어 에이전트가 대신할 수 없는 항목이다.

## 1. 로컬 실주문 (토스증권 API) - `toss/SETUP.md` 참고
- [ ] 토스증권 계좌 개설 (본인 인증)
- [ ] WTS > 설정 > Open API에서 `client_id`, `client_secret` 발급 (secret은 1회만 표시)
- [ ] 허용 IP 목록에 호출할 머신의 공인 IP 등록 (미등록 시 403)
- [ ] 자격 증명을 셸 환경변수로 직접 설정. 채팅/코드/커밋에 붙여넣지 말 것
      `export TOSS_CLIENT_ID=...` / `export TOSS_CLIENT_SECRET=...`
- [ ] accountSeq 확인 후 dry-run 실행: `./.venv/bin/python -m toss.place_orders --account-seq <N>`
- [ ] 실주문은 본인이 직접: `TOSS_LIVE_TRADING=CONFIRM ... --live`

## 2. 무인 운영 (GCE + Docker Hub) - `cloud/README.md` 참고
- [ ] GCP 프로젝트 + 결제 계정 연결, `gcloud auth login`, `gcloud config set project <ID>`
- [ ] `gcloud services enable compute.googleapis.com`
- [ ] Docker Hub 계정 + GitHub 저장소 Secrets에 CI용 Docker Hub 자격 증명 등록
      (이름은 `.github/workflows/docker-build.yml` 참고)
- [ ] VM용 읽기 전용 Docker Hub 토큰 (이미지가 private인 경우)
- [ ] `cloud/deploy.sh` 실행 후 안내되는 수동 단계 수행
- [ ] 고정 IP를 토스 허용 IP에 등록, 자격 증명은 GCP Secret Manager에 저장

## 3. 대시보드 (선택)
- [ ] 공개 GCS 버킷 생성 및 VM 서비스 계정에 쓰기 권한 부여 (`dashboard/publish.sh` 참고)
- [ ] GitHub Pages 활성화 (`docs/`)
