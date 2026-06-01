# 테스트 케이스 — Kaniko → BuildKit 전환 검증 (Pipeline/1029)

> **공통 사전조건**: BuildKit 적용 Pipeline(`image-build-stage-new.yml`)과 비교용 Kaniko Pipeline(`image-build-stage-origin.yml`)을 **같은 입력**(repo/브랜치/Dockerfile 경로/이미지 태그)으로 준비. 푸시 대상은 NCR. 빌드 파라미터는 배포 없이 기존 구조 그대로 사용.

## A. 회귀 (구문법 — Kaniko·BuildKit 모두 성공)

| TC | 제목 | 대상 / 파라미터 | 절차 | 기대결과 (BuildKit) | Kaniko(참고) |
|----|------|----------------|------|--------------------|--------------|
| TC-01 | Maven 기본 빌드 & NCR 푸시 | `maven/Dockerfile.legacy`, context `maven/` | 파이프라인 실행 | PipelineRun Succeeded, NCR 푸시됨 | 성공 |
| TC-02 | nginx(node) 기본 빌드 & 푸시 | `nginx/Dockerfile.legacy`, context `nginx/` | 파이프라인 실행 | Succeeded, 푸시됨 | 성공 |

## B. 신문법 (Kaniko 실패 / BuildKit만 성공) — 전환의 핵심

| TC | 제목 | 대상 / 검증 문법 | 기대결과 (BuildKit) | Kaniko(참고) |
|----|------|------------------|--------------------|--------------|
| TC-03 | syntax directive + FROM 이전 글로벌 ARG | `maven/Dockerfile.modern` | 성공 | 실패 |
| TC-04 | `RUN --mount=type=cache` (Maven .m2) | `maven/Dockerfile.modern` | 성공, 캐시 마운트 동작 (회차 단축은 daemonless 캐시 export 설정에 따름) | 실패 |
| TC-05 | `COPY --link --chown --chmod` 조합 | `maven/Dockerfile.modern` | 성공, 권한/소유자 반영 | 실패 |
| TC-06 | Heredoc `RUN <<EOF` 멀티라인 | `maven/Dockerfile.modern` | 성공 | 실패 |
| TC-07 | `RUN --network=none` 네트워크 격리 | `maven/Dockerfile.modern` | 성공 | 실패 |
| TC-08 | nginx 신문법(cache mount + `COPY --link`) | `nginx/Dockerfile.modern` | 성공 | 실패 |

## C. 파라미터 호환성 (drop-in)

| TC | 제목 | 절차 | 기대결과 |
|----|------|------|---------|
| TC-09 | 기존 파라미터 구조 그대로 동작 | Kaniko용 동일 파라미터 입력값을 BuildKit Task에 그대로 전달 | 추가/변경 없이 정상 빌드 (배포 불필요 검증) |
| TC-10 | build-args 전달 | `APP_HOME`, `SERVICE_NAME` build-arg 지정 | 이미지에 반영됨 |
| TC-11 | Dockerfile 경로 / 컨텍스트 지정 | `.legacy`/`.modern` 경로 변경 지정 | 지정 파일 기준 빌드 |
| TC-12 | 이미지 태그 지정 / 멀티태그 푸시 | 태그 파라미터 지정 | 지정 태그로 NCR 푸시 |

> ※ TC-09~12의 정확한 파라미터 이름은 `buildkit-daemonless.yaml`의 Task `params` 정의와 대조해 확정할 것.

## D. 예외 / 음성 케이스

| TC | 제목 | 대상 | 기대결과 |
|----|------|------|---------|
| TC-15 | 문법 오류 Dockerfile | `negative/Dockerfile.syntax-error` | 빌드 실패가 PipelineRun Failed로 정확히 전파, 비정상 exit code |
| TC-16 | 존재하지 않는 base image | base 태그 임의 변조 | pull 실패 → 빌드 실패 처리 |

## E. 참고 (성능)

| TC | 제목 | 측정 |
|----|------|------|
| TC-17 | 빌드 소요시간 비교 | 동일 Dockerfile에 대해 BuildKit vs Kaniko 소요시간 기록 (Dooray 사례: 기본 BuildKit 37s / Kaniko 47s) |
