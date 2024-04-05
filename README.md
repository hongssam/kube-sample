# 1. nest project생성
## a. nest.js cli 설치
```
npm i -g @nestjs/cli
```

## b. 프로젝트 생성
```
# 패키지 매니저 설정: npm 선택
nest new [프로젝트 폴더명]
```
# 2. 도커이미지 생성
## a. 프로젝트 루트경로에 Dockerfile, .dickerignore 생성
```
# Dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```
```
# .dockerignore
node_modules
dist
.git
Dockerfile
```

## b. 빌드 & 실행
```
# 프로젝트 루트경로에서 도커 이미지 생성
# dockerhub에 올릴꺼면 docker hub repo명과 도커이미지명이 같아야함
docker build -t [도커이미지 명] .

# 이미지 확인
docker images

# 로컬에서 이미지 실행
docker run -it -p [접속port]:[도커port] [도커이미지 명]

# docker hub push
docker login --username=[dockerhub Id]
docker push [도커이미지 명]:[image version]
```
 
# 3. minikube + kubectl로 service띄우기
## a. minikube 설치
```
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# Usages
minikube start
```

## b. kubectl설치
- mac : https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/
- window : https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
- linux : https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

## c. deployment.yaml파일 생성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nest
  labels:
    app: nest
spec: # specification
  replicas: 2
  selector: 
    matchLabels:
      app: nest
  template:
    metadata:
      labels: 
        app: nest
    spec:
      containers:
      - name: nest
        image: [docker hub image명] # either from docker registries or other
        ports:
        - containerPort: 3000
```
 
## d. service.yaml파일 생성
```
# depolyment.yaml파일 하나로 관리할거기 때문에 맨아래 --- 입력후 하단에 접속정보 추가

depolyment.yaml내용

---
apiVersion: v1
kind: Service
metadata:
  name: nest-service
spec:
  selector:
    app: nest
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30000 # 30000~32000 포트범위 정해져 있음
```

## e. eployment & service등록
```
# deplyment.yaml하나에 service 내용까지 작성했기 때문에 아래 명령어만 실행하면 service까지 등록됨
kubectl apply -f deployment.yaml

# deplyments 확인
kubectl get deplyments

# service 확인
minikube service list

# service 실행
minikube service [service명]

# minikube 대쉬보드 접속
minikube dashboard
```
- 출처: https://hongsamm.tistory.com/56 [홍쌈's:티스토리]