# 1. app.py & requirements.txt & Dockerfile 생성

# 2. Docker build: docker build -t cicd-flask .

# 3. Docker run: docker run -p 5000:5000 cicd-flask

# 4. localhost:5000 접속

# 5. Docker로 Jenkins 실행
    5.1. 이미지 빌드: docker build -t jenkins-docker ./jenkins-image
    5.2. Volume 설정 실행(bash에서): docker run -p 8080:8080 -p 50000:50000 -v $(pwd)/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker
    5.3. Volume 설정 실행(fish에서): docker run -p 8080:8080 -p 50000:50000 -v (pwd)/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins-docker

# 6. localhost:8080 접속 후 계정 설정

# 7. 첫 번째 CI pipeline 만들기
    7.1. 좌상단 새로운 Item 클릭 혹은 중앙 Create a job 클릭
    7.2. Enter an item name: flask-cicd
    7.3. Select an item type: Pipeline
    7.4. OK 버튼 클릭
    7.5. General: 설명
        Stage 1 : Checkout (Git pull)
        Stage 2 : Build
        Stage 3 : Test
        Stage 4 : Docker Build
        Stage 5 : Deploy
    7.6. Pipeline Script 작성: Jenkinsfiles/Jenkinsfile.v1 참고

# 8. Git 초기화 및 GitHub 원격 레포지토리 연결

# 9. Pipeline 코드 수정 후 재빌드: Jenkins와 원격 레포지토리를 연결, Jenkinsfiles/Jenkinsfile.v2 & Jenkinsfiles/Jenkinsfile.v3 참고
    9.1. 이 시점까지 완료된 것
        Jenkins 실행
        ↓
        GitHub repo clone
        ↓
        Docker image build
        ↓
        컨테이너 실행
    9.2. localhost:5000 접속 시 정상적으로 Hello CI/CD 정상적으로 뜨면 됨
    9.3. 현재 구조의 문제: Jenkins 실행할 때마다 계속 새로운 컨테이너 생성
    9.4. 따라서 컨테이너 기존거 삭제 → 새로 실행하는 과정이 필요

# 10. 기존 컨테이너를 삭제하고 새로 실행하는 과정을 파이프라인에 추가
    10.1. 추가하려는 내용
        stop old container
        remove old container
        run new container
    10.2. Jenkinsfiles/Jenkinsfile.v4 참고
    10.3. 이 시점까지 완료된 것
        Git Push
        ↓
        Jenkins 자동 실행
        ↓
        Docker build
        ↓
        기존 컨테이너 종료
        ↓
        새 컨테이너 실행

# 11. ngrok를(로컬 서버를 인터넷에서 접근 가능한 임시 주소로 공개해주는 도구) 사용하여 Github에서 Jenkins에 접근을 가능하게 함
    11.1. ngrok 설치 및 토큰 등록 과정 필요
    11.2. ngrok을 사용하는 이유는 로컬호스트 상에서 하는 것이기에 임시방편으로 연결을 위해 열어주는 것
    11.3. ngrok http 8080 통해 실행

# 12. Webhook 설정 추가: Jenkinsfiles/Jenkinsfile.v5 참고
    12.1. 기존의 방식: Build Now 버튼 누름 → Jenkins가 GitHub에서 코드 가져옴 → Docker 이미지 빌드 → 컨테이너 실행
    12.2. 변경하고자 하는 방식: Build Now 버튼 누름 → Jenkins가 GitHub에서 코드 가져옴 → Docker 이미지 빌드 → 컨테이너 실행
    12.3. 위에서 ngrok을 사용하는 이유는 GitHub가 Jenkins에게 알림을 가능하게 하기 위함(= Webhook)
    12.4. Webhook: 이벤트가 생기면 상대 서버로 바로 HTTP 요청을 보내는 방식
    12.5. 설정 추가 방식: GitHub
        12.5.1. GitHub Repo 들어가서 Settings 클릭
        12.5.2. Webhooks -> Add Webhook 클릭
        12.5.3. Payload URL 입력: https://{ngrok에서 나온 주소}/github-webhook/
        12.5.4. Content Type: application/json
        12.5.5. Trigger: Just the push event
        12.5.6. Add webhook 클릭
    12.6. 설정 추가 방식: Jenkins
        12.6.1. 위에서 설정한 파이프라인 설정에 들어가 Configure(구성) 클릭
        12.6.2. Triggers -> GitHub hook trigger for GITScm polling 클릭 후 Save
    12.7. 이 시점에서 완료되는 것:
        git push
        ↓
        GitHub webhook
        ↓
        ngrok
        ↓
        Jenkins
        ↓
        Docker build
        ↓
        Container restart
    12.8. 현재 상태로는 어디서 실행되는가
        원래 구조는 이러하다
            [내 PC / WSL]
                │
                ├─ Docker Engine (호스트)
                │
                └─ Jenkins Container
                        │
                        └─ docker build / docker run 실행
        즉 Jenkins 컨테이너 안에서 컨테이너가 실행되어야 하나, docker.sock 마운트를 진행했기에 Jenkins 컨테이너가 호스트 도커 엔진을 직접 제어함
        따라서 실제로는 Jenkins 컨테이너 안에서 컨테이너가 만들어지는 게 아니라 호스트 Docker에서 컨테이너가 만들어짐(Docker outside of Docker)
        이 배포하는 과정은 CD

# 참고: Jenkins 문법
    https://dayne-w.tistory.com/19