# syntax=docker/dockerfile:1
# =====================================================================
# "빌드 명령어"(사전 훅) 검증용
#   빌드 명령어가 build-info.txt 를 생성해야만 아래 COPY 가 성공한다.
#   (build-info.txt 는 repo 에 커밋하지 않음 → 빌드 명령어가 안 돌면 빌드 실패)
# =====================================================================
FROM alpine:3
WORKDIR /app
COPY build-info.txt /app/build-info.txt
RUN cat /app/build-info.txt
CMD ["cat", "/app/build-info.txt"]
