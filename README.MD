# 1. app.py & requirements.txt & Dockerfile 생성

# 2. Docker build: docker build -t cicd-flask .

# 3. Docker run: docker run -p 5000:5000 cicd-flask

# 4. localhost:5000 접속

# 5. Docker로 Jenkins 실행
    5.1. 단순 실행: docker run -p 8080:8080 jenkins/jenkins:lts
    5.2. Volume 설정 실행(bash에서): docker run -p 8080:8080 -p 50000:50000 -v $(pwd)/jenkins:/var/jenkins_home jenkins/jenkins:lts
    5.3. Volume 설정 실행(fish에서): docker run -p 8080:8080 -p 50000:50000 -v (pwd)/jenkins:/var/jenkins_home jenkins/jenkins:lts

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

# 9. Pipeline 코드 수정 후 재빌드: Jenkins와 원격 레포지토리를 연결, jenkinsfiles/Jenkinsfile.v2 참고

