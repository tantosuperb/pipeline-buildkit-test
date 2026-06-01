# pipeline-buildkit-test

Pipeline 이미지 빌드 Task **Kaniko → BuildKit 전환** 검증용 테스트 저장소.

Maven / nginx 두 스택에 대해 **구문법(legacy)** 과 **신문법(modern)** Dockerfile 페어를 제공한다.
- `legacy` : Docker 17~18 수준 문법 → Kaniko·BuildKit 모두 성공해야 함 (회귀)
- `modern` : BuildKit 신문법(`syntax` directive, `RUN --mount=type=cache`, `COPY --link/--chown/--chmod`, Heredoc, `RUN --network=none`) → Kaniko 실패 / BuildKit 성공

## 빌드 매트릭스

| 스택 | 컨텍스트 | Dockerfile | 분류 |
|------|----------|-----------|------|
| Maven | `maven/` | `maven/Dockerfile.legacy` | 회귀 |
| Maven | `maven/` | `maven/Dockerfile.modern` | 신문법 |
| nginx | `nginx/` | `nginx/Dockerfile.legacy` | 회귀 |
| nginx | `nginx/` | `nginx/Dockerfile.modern` | 신문법 |
| (예외) | `negative/` | `negative/Dockerfile.syntax-error` | 빌드 실패 처리 |

자세한 테스트 케이스는 `TEST-CASES.md` 참고.
