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
    7.6. Pipeline Script 작성: jenkinsfiles/Jenkinsfile.v1 참고

# 8. Git 초기화 및 GitHub 원격 레포지토리 연결

# 9. Pipeline 코드 수정 후 재빌드: Jenkins와 원격 레포지토리를 연결, jenkinsfiles/Jenkinsfile.v2 & jenkinsfiles/Jenkinsfile.v3 참고
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
    10.2. jenkinsfiles/Jenkinsfile.v4 참고
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

# 참고: Jenkins 문법
    https://dayne-w.tistory.com/19