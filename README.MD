# 1. app.py & requirements.txt & Dockerfile 생성

# 2. Docker build: docker build -t flask-practice flask-practice/

# 3. Docker run: docker run -p 5000:5000 flask-practice

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

# 13. 이 과정까지의 인사이트
    13.1. CI Model Selection: 왜 GitHub Actions 안 쓰고 Jenkins 썼는가?
        GitHub Actions: (장점) 설정 간단, GitHub 통합 쉬움, 빠른 시작 / (단점) 복잡한 환경 구성 어려움, self-hosted 환경 구성 제한, 플러그인 생태계 제한
        Jenkins: (장점) 플러그인 생태계 매우 큼, Kubernetes / Docker / Cloud 통합 쉬움, self-hosted 환경 구축 가능, 기업에서 많이 사용 / (단점) 초기 설정 복잡, 관리 부담 있음, UI 오래됨
        Jenkins는 플러그인 기반으로 Docker, Kubernetes 등 다양한 환경과 유연하게 통합할 수 있다는 장점
        또한 향후 Kubernetes 기반 CD 환경까지 확장하는 것을 고려하면 Jenkins를 선택하는 것이 유리
        다만 Jenkins는 초기 구축 비용과 관리 부담이 있다는 단점이 있지만, 이를 통해 실제 운영 환경과 유사한 경험을 할 수 있다는 점에서 선택
    13.2. 왜 docker.sock 방식 사용했는가?
        Docker-in-Docker 방식 대신 docker.sock을 마운트하는 방식을 선택
        빌드 속도가 빠르고 이미지 공유가 쉽다는 장점이 있지만, Jenkins 컨테이너가 Docker 엔진을 직접 제어할 수 있어 보안 측면에서 취약할 수 있다는 Trade-Off가 존재

# 14. Jenkins에서 Docker 이미지를 Docker Hub에 push 하도록 바꾸기
    14.1. 지금은 Jenkins가 로컬에서 이미지 빌드하고 바로 컨테이너 실행까지 하고 있는데(Jenkins -> docker build -> docker run),
    14.2. 앞으로 GKE/ArgoCD로 가려면 클러스터가 가져다 쓸 수 있는 공용 이미지 저장소가 필요함(Jenkins -> docker build -> Docker Hub push)
    14.3. Docker Hub 설정
        14.3.1. Docker Hub 계정 만들기
        14.3.2. Docker Hub에 repository 하나 만들기
		    repository name: flask-practice
		    프라이빗 레포지토리로 설정 가능하나 나중에 추가적으로 설정 필요
    14.4. 이미지 빌드
        Jenkins가 이미지를 빌드할 때, 기존처럼 flask-practice 라고만 만들면 로컬 이미지임
        Docker Hub에 올리려면 DockerHubID/RepositoryName:Tag 형태여야 push 가능함
        예) docker build -t barumk/flask-practice:latest flask-practice/
    14.5. Docker Hub Access Token 만들기
        Jenkins에서 docker login 해야 push가 가능하기 때문에, 비밀번호를 대신할 Access Token을 사용
        계정 설정에 들어가 Personal Access Token 생성 - 권한 Read & Write로 설정
            Run: docker login -u barumk
            Access Token: 거기에서 나온 내용
    14.6. Jenkins에 Access Token 등록
        Manage Jenkins -> Security -> Credentials -> Add Credentials
        Kind: Username with password
        Scope: Global
        Username: 계정 이름
        Password: Docker Hub에서 받은 Access Token
        ID: dockerhub-creds
    14.7. Jenkinsfile 수정: Jenkinsfiles/Jenkinsfile.v6 참고
        현재 구조: Checkout -> Docker Build -> Stop Old Container -> Run Container
        변경하려는 구조: Build → Login → Push → Run (임시 유지), 나중에 GKE 가면 Run은 제거
        핵심 변경 포인트: IMAGE_NAME 추가, Docker Login 추가, Docker Push 추가
    14.8. 이 과정이 완료되면...
        GitHub에 Push를 하면 Webhook이 이를 감지하여 Jenkins로 변경사항을 알려주고, 이를 바탕으로 Pull하여 이미지 빌드를 수행하고, Docker Hub에 빌드가 완료된 이미지를 Push 후 로컬에서는 해당 이미지를 기반으로 도커 컨테이너를 실행함

# 15. GCP 설정
    15.1. GCP Console 접속: https://console.cloud.google.com/
    15.2. 프로젝트 생성
        Project name: cicd-practice
        Location: No organization
    15.3. GCP의 구조
        Google Cloud
            └ Project
                ├ GKE Cluster
                ├ VM
                ├ Network
                ├ Storage
        앞으로 만들 것들은 전부 cicd-practice Project 안에 생성됨
    15.4. GKE API 활성화
        GKE를 쓰려면 Kubernetes Engine API 활성화 해야 함
        좌측 메뉴 -> Kubernetes Engine -> 클러스터 -> cicd-practice 클릭 -> Kubernetes Engine API 사용 버튼 클릭
        해당 클러스터 안으로 진입
    15.5. 클러스터 생성
        Autopilot: Google manages your cluster (Recommended) 선택
            Autopilot 장점: 관리 자동, 설정 간단, 비용 절약, 노드 관리 불필요
            Standard는: 노드 직접 설정, 비용 더 나갈 가능성, 지금 단계에선 불필요
        클러스터 이름: cicd-cluster
        Region 선택: asia-northeast3 (Seoul) / asia-northeast1 (Tokyo) -> 적은 네트워크 지연
        Fleet 사용 안함: 여러 Kubernetes 클러스터를 대규모로 관리할 때 쓰는 기능이기 때문
        Networking / Security: 건드리지 말고 기본값 유지
    15.6. 우리가 만들 클러스터 구조
        GKE Cluster
            └ Pods
                └ Flask App
    15.7. Cloud Shell 실행 통해 kubectl 연결
        GCP 화면 오른쪽 상단의 >_ 아이콘 클릭
        gcloud config get-value project 명령어 입력 시 cicd-practice 나와야 함 -> 만약 다른 값 나오면 gcloud config set project cicd-practice
        GKE -> 클러스터 목록의 클러스터 상세 화면 상단 연결 (Connect) 클릭하여 나온 주소를 Cloud Shell에 연결
        gcloud container clusters get-credentials cicd-cluster --region asia-northeast3 --project cicd-practice-XXXXXX 형태
            Fetching cluster endpoint and auth data.
            kubeconfig entry generated for cicd-cluster.
            -> 라는 메세지 뜨면 정상 연결

# 16. Jenkins에서 Build된 이미지를 GCP에서 Pod으로 실행
    16.1. Cloud Shell에서 파일 생성: vim flask-deployment.yaml (Kubernetes-ConfigFiles/flask-deployment.yaml 참고)
    16.2. Docker Hub Secret 생성: kubectl create secret docker-registry dockerhub-secret --docker-username=barumk --docker-password=YOUR_DOCKER_PASSWORD --docker-email=wookiigg@gmail.com
    16.2. Deployment 실행: kubectl apply -f flask-deployment.yaml
    16.3. Service 생성(외부 접속용): vim flask-service.yaml (Kubernetes-ConfigFiles/flask-service.yaml 참고)
    16.4. Service 실행: kubectl apply -f flask-service.yaml
    16.5. External IP 확인: kubectl get service -> 예) flask-service   LoadBalancer   34.xxx.xxx.xxx

# 17. GitOps 자동 배포 구조 생성
    17.1. ArgoCD Namespace 생성: kubectl create namespace argocd
    17.2. ArgoCD 설치: kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    17.3. ArgoCD Pod 확인: kubectl get pods -n argocd
    17.4. ArgoCD 외부 접속 설정: kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    17.5. External IP 확인: kubectl get svc -n argocd -> 예) argocd-server   LoadBalancer   34.xxx.xxx.xxx
    17.6. 초기 로그인 비밀번호 확인: kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d (아이디는 admin)
    17.7. 해당 External IP로 접속하여 로그인: https://EXTERNAL-IP
    17.8. ArgoCD에서 Application 생성
        왼쪽 상단 NEW APP 클릭
        GENERAL
            Application Name: flask-app
            Project: default
            Sync Policy: Automatic
        SOURCE
            Repository URL: https://github.com/BarumK/cicd-practice
            Revision: main
            Path: Kubernetes-ConfigFiles
        DESTINATION
            Cluster: https://kubernetes.default.svc
            Namespace: default
    17.9. 이렇게 할 경우
        GitHub repo 읽음
        ↓
        deployment.yaml 확인
        ↓
        GKE 자동 배포

# 18. Push → 자동 배포 과정으로 수정: Jenkinsfiles/Jenkinsfile.v7 참고
    18.1. 최종 자동 배포 흐름
        GitHub Push
            ↓
        Webhook
            ↓
        Jenkins 실행
            ↓
        Docker Build
            ↓
        Docker Hub Push
            ↓
        deployment.yaml 수정
            ↓
        GitHub Push
            ↓
        ArgoCD 감지
            ↓
        Kubernetes 자동 배포

# 참고: Jenkins 문법
    인터넷 자료
    - https://www.jenkins.io/doc/book/pipeline/
    - https://dayne-w.tistory.com/19

Jenkinsfile 기본 구조
```groovy
pipeline {
    agent any

    stages {
        stage('StageName') {
            steps {
                sh 'command'
            }
        }
    }
}
```

| 요소       | 의미          |
| -------- | ----------- |
| pipeline | 전체 파이프라인    |
| agent    | 실행 환경       |
| stages   | 단계 묶음       |
| stage    | 하나의 단계      |
| steps    | 실행 명령       |
| sh       | shell 명령 실행 |

    일반적인 CI 흐름:
        Checkout
        ↓
        Test
        ↓
        Build
        ↓
        Deploy
