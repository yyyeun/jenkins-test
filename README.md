# [ 🧶 Jenkins를 사용한 SpringBoot Application 배포 워크플로우 구현 ]
![CICD Architecture](https://github.com/user-attachments/assets/6122a28d-a6b8-45a2-b766-a6f7d0180f29)

## 🧹 개요
Docker Container 기반의 Jenkins와 Ubuntu 기반의 Host 환경을 Bind Mount 방식으로 연동하여 SpringBoot Application 빌드 및 구동을 위한 **CI/CD 파이프라인을 구축**합니다. 용도에 따라 **서버를 분리**하여 개발 서버에서 JAR 파일이 새롭게 빌드되면 운영 서버로 파일을 전달 및 실행함으로써 **자동화된 배포 프로세스를 구현**합니다.

<br>

## 🚍 작업 Workflow
1. Host의 일반 폴더와 Jenkins Container를 **Bind Mount** 방식으로 동기화
2. **개발 및 테스트 서버**에서 최신 빌드를 감지하기 위해 JAR 파일에 대한 모니터링을 수행
    - 기존 JAR 파일 백업
    - 운영 서버로 JAR 파일 전송
3. **운영 서버**에서 포트 검사 후 Application 실행

<br>

## 1. Jenkins 환경 설정 및 Host와의 Bind Mount
### 1. Host에 마운트 폴더 생성
```bash
$ mkdir appjardir
```

### 2. Jenkins 컨테이너 생성 및 실행
```bash
$ docker run --name myjenkins --privileged -p 8080:8080 -v $(pwd)/appjardir:/var/jenkins_home/appjar jenkins/jenkins:lts-jdk17
```
- Bind Mount 옵션 적용
- privileged Mode → 시스템의 모든 장치에 접근 가능

<br>
<div align="center">
<img src="https://github.com/user-attachments/assets/6a124437-8e62-4d24-8bff-ea07610e2410" width="600">
</div>

### 3. 자동 생성된 Mount 디렉토리 확인 & 폴더에 쓰기 권한 부여
Mount 디렉토리 확인
```bash
$ docker inspect myjenkins
```

<div align="center">
<img src="https://github.com/user-attachments/assets/cebc8799-9eba-45b7-a20e-b744c0021f8b" width="600">
</div>

<br>
Jenkins Container에 root 계정로 접근 후 Mount 디렉토리에 권한 부여 (비밀번호 입력 불필요)

```
username@servername:~$ docker exec -u root -it myjenkins bash

root@0f78dad283e0:/# whoami
root

root@0f78dad283e0:/# chmod -R 755 /var/jenkins_home/appjar
```

### 4. ngrok 실행 & GitHub 연동(webhook 지정)

```bash
ngrok http http://127.0.0.1:<jenkins port num>
```

<div align="center">
<img src="https://github.com/user-attachments/assets/e87ab3e2-c269-4fe1-b494-2eec0b2d736e" width="600">
</div>

### 5. SpringBoot Application 준비 후 GitHub Repository에 push

<div align="center">
<img src="https://github.com/user-attachments/assets/c149f868-8ca3-414e-8fa2-a42bec38030e" width="600">
</div>

### 6. Jenkins Pipeline 구성
<div align="center">
<img src="https://github.com/user-attachments/assets/1634b869-55d4-49d6-a1a3-01c505e77f10" width="600">
</div>

<br> 
Pipeline

```
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/bigkhk/fisatest.git'
            }
        }
          
        stage('Build') {
            steps {
                dir('./') {                   
                    sh 'chmod +x gradlew'                    
                    sh './gradlew clean build -x test'
                    sh 'echo $WORKSPACE'  # echo /var/jenkins_home/workspace/step02_jarcicd
                }
            }
        }
        
        stage('Copy jar') { 
            steps {
                script {
                    def jarFile = 'build/libs/SpringApp-0.0.1-SNAPSHOT.jar'                   
                    sh "cp ${jarFile} /var/jenkins_home/appjar/"                  }
            }  # 결과: host 폴더에 jar 파일 생성됨
		           # 깃 push 시 Jenkins 워크플로우 거쳐 host jar 파일이 업데이트됨
        }
    }
}
```

<br>
빌드 후 workspace 확인

<div align="center">
<img src="https://github.com/user-attachments/assets/f883e60f-3e9d-4f0f-a5f1-bd40efd1aa2c" width="600">
</div>

<br>

## 2. Development/Staging Server
### 1. 파일 변화 감지 패키지 설치
```bash
$ sudo apt-get install inotify-tools
```

### 2. SSH 설정
```bash
# SSH key 생성
$ ssh-keygen -t rsa -b 4096

# key 생성 후 확인
$ ls -l .ssh  # id_rsa, id_rsa.pub, authorized_keys 확인

# 원격 서버에 SSH 공개 키 추가
$ ssh-copy-id username@<remote_server_ip>

# 비밀번호 없이 접속 확인
$ ssh username@<remote_server_ip>
```
<br>

## 3. Production Server
### 1. 

<br>

## 🧵 결론 및 고찰
> 



<div align="center">
<img src="" width="460">
</div><br>