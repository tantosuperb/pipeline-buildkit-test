# BuildKit 기능 커버리지 (allinone)

빌드 컨텍스트는 모두 **`allinone/`**, Dockerfile 경로만 바꿔 빌드한다.
`# syntax=docker/dockerfile:1-labs` 사용 (labs 기능 포함).

## Dockerfile 문법/기능 커버리지

| 기능 | Dockerfile | Kaniko | BuildKit |
|------|-----------|:------:|:--------:|
| `RUN --mount=type=cache` | mounts | ✗ | ✓ |
| `RUN --mount=type=secret` | mounts | ✗ | ✓ |
| `RUN --mount=type=ssh` | mounts | ✗ | ✓ |
| `RUN --mount=type=tmpfs` | mounts | ✗ | ✓ |
| `RUN --mount=type=bind,from=...` | mounts | ✗ | ✓ |
| `RUN --network=none` | mounts | ✗ | ✓ |
| `COPY --link` | copy-add | ✗ | ✓ |
| `COPY --chown --chmod` | copy-add | △ | ✓ |
| `COPY --parents` (labs) | copy-add | ✗ | ✓ |
| `COPY --exclude` (labs) | copy-add | ✗ | ✓ |
| `COPY <<EOF` heredoc | copy-add | ✗ | ✓ |
| `ADD --chmod` | copy-add | △ | ✓ |
| `ADD <git-ref>` | copy-add | ✗ | ✓ |
| `# syntax=` directive | syntax-misc | ✗ | ✓ |
| FROM 이전 글로벌 ARG | syntax-misc | △ | ✓ |
| 자동 플랫폼 ARG (TARGET/BUILDPLATFORM) | syntax-misc | △ | ✓ |
| `RUN <<EOF` heredoc (sh) | syntax-misc | ✗ | ✓ |
| 커스텀 인터프리터 heredoc (`RUN <<EOT python3`) | syntax-misc | ✗ | ✓ |
| SHELL/ENV/LABEL/EXPOSE/VOLUME/STOPSIGNAL/HEALTHCHECK/USER/ONBUILD/ENTRYPOINT/CMD | syntax-misc | ✓ | ✓ |

## 올인원에 넣지 못한 것 (성격상 별도 처리)

| 기능 | 이유 | 검증 방법 |
|------|------|-----------|
| `--mount=type=secret`/`ssh` 실제 주입 | 빌드 시 `--secret`/`--ssh` 입력 필요 (Task 파라미터) | 입력 줘서 별도 검증, 미입력 시엔 가드로 통과 |
| 멀티플랫폼 빌드 | Dockerfile 아닌 빌드 호출 플래그(`platform=...`) | Task 파라미터로 amd64,arm64 지정 |
| `ADD --checksum` | 원격 파일의 정확한 sha256 필요 | copy-add 의 주석 라인에 체크섬 채워 활성화 |
| `RUN --security=insecure` | buildkitd 엔타이틀먼트 필요(기본 환경 미허용) | 엔타이틀먼트 부여된 환경에서만 |
| 레지스트리 캐시 export/import | 빌드 호출 옵션(`--cache-to/--cache-from`) | 아래 BuildKit 파라미터 표 참고 |

## BuildKit 빌드 파라미터 (Task 파라미터로 테스트)

> 실제 노출되는 파라미터명은 `buildkit-daemonless.yaml` 의 Task `params` / `buildctl` 인자와 대조 필요.

| 파라미터 | buildctl 예시 | 검증 포인트 |
|----------|---------------|-------------|
| build-args | `--opt build-arg:APP_HOME=/srv` | ARG 값이 이미지에 반영 |
| target stage | `--opt target=builder` | 지정 스테이지까지만 빌드 |
| platform (멀티) | `--opt platform=linux/amd64,linux/arm64` | manifest list 생성 |
| no-cache | `--no-cache` | 캐시 무시하고 풀빌드 |
| labels | `--opt label:foo=bar` | 이미지 라벨 주입 |
| secret | `--secret id=mysecret,src=...` | secret mount 동작 |
| ssh | `--ssh default` | ssh mount 동작 |
| inline cache | `--opt build-arg:BUILDKIT_INLINE_CACHE=1` | 이미지에 캐시 메타 포함 |
| cache export | `--export-cache type=registry,ref=...` | 레지스트리 캐시 저장 |
| cache import | `--import-cache type=registry,ref=...` | 캐시 재사용으로 단축 |
| network mode | `--opt network=none` | 빌드 네트워크 제어 |
| output/push | `--output type=image,name=...,push=true` | 빌드+푸시 (검증 완료) |
