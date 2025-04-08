# 📌 GitHub Actions 기반 CI/CD 구성 정리

## 📝 개요

CI/CD는 애플리케이션을 자동으로 빌드하고, 테스트하고, 배포하는 일련의 프로세스를 의미한다. GitHub Actions를 활용하면 GitHub 저장소에 push나 pull request 이벤트가 발생했을 때 자동으로 워크플로를 실행할 수 있다.

이 문서는 GitHub Actions를 기반으로 한 CI/CD를 직접 구축하고, 내 프로젝트에 적용한 방법을 정리한 것이다. 무중단 배포를 위한 구성 방식도 포함되어 있다.

---

## 🔧 1. GitHub Actions란?

- GitHub에서 제공하는 자체 CI/CD 플랫폼
- `.github/workflows` 경로에 YAML 형식의 워크플로 파일을 작성
- 이벤트(예: push, pull_request)에 반응하여 Job을 자동 실행

---

## 🛠️ 2. 기본 CI/CD 구성 흐름

### 📂 2-1. 디렉토리 및 파일 생성

```bash
mkdir -p .github/workflows
cd .github/workflows
touch ci-cd.yml
```

### 🧾 2-2. 기본 workflow 예시 (`ci-cd.yml`)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew

      - name: Build with Gradle
        run: ./gradlew build

      - name: Copy files via SCP (배포 서버 전송)
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.REMOTE_HOST }}
          username: ${{ secrets.REMOTE_USER }}
          password: ${{ secrets.REMOTE_PASSWORD }}
          port: 22
          source: "build/libs/*.jar"
          target: "~/app"

      - name: Execute remote command (서버 재시작)
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.REMOTE_HOST }}
          username: ${{ secrets.REMOTE_USER }}
          password: ${{ secrets.REMOTE_PASSWORD }}
          port: 22
          script: |
            pkill -f 'java -jar' || true
            nohup java -jar ~/app/*.jar > ~/app/log.txt 2>&1 &
```

---

## 🔐 3. GitHub Secrets 설정

GitHub 저장소의 `Settings > Secrets and variables > Actions` 메뉴에서 아래 키 등록

- `REMOTE_HOST`: 배포 대상 서버의 IP 또는 도메인
- `REMOTE_USER`: SSH 사용자명
- `REMOTE_PASSWORD`: SSH 접속 비밀번호 또는 Private Key

---

## 🚀 4. 무중단 배포 구성

CI/CD만으로 무중단 배포가 보장되지는 않기 때문에 아래 방식 중 하나를 선택해서 구성해야 한다.

### ✅ 방식 1: 포트 스위칭 (Nginx 활용)

- 포트를 두 개 운영하며 번갈아 실행 (예: 8080, 8081)
- 새로운 포트로 앱을 실행한 뒤 nginx에서 해당 포트로 트래픽 변경
- 기존 포트 종료 처리

```bash
nohup java -jar app.jar --server.port=8081 &

# nginx.conf에서 proxy_pass를 8081로 변경

pkill -f 'java -jar.*8080'
```

### ✅ 방식 2: 무중단 배포 도구 사용

- AWS CodeDeploy, Jenkins, 자체 무중단 스크립트 등 활용 가능

---

## 📦 5. 적용 시 유의사항

- `.jar` 외 설정 파일(`application.yml`, `nginx.conf` 등)도 필요한 경우 함께 전송 필요
- 배포 서버에 Java 17이 설치되어 있어야 함
- `server.port` 설정으로 포트 충돌 방지
- `nohup`, `pkill` 명령어가 정상 동작하는 환경인지 확인 필요

---

## 📌 요약

- GitHub Actions를 통해 CI/CD 파이프라인을 간단하게 구축할 수 있다
- `.github/workflows/ci-cd.yml` 작성 + Secrets 등록으로 기본 구성 완료
- 무중단 배포를 위해선 추가적인 Nginx 설정 또는 스크립트 연동 필요
- `scp`, `ssh`를 통해 배포 자동화 가능