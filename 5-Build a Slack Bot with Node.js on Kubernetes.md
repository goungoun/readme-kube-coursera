## Build a Slack Bot with Node.js on Kubernetes

## 실습 요약
https://qwiklabs.com/focuses/635?parent=catalog
을 참고하여 Slack Bot을 만들어본다.

1. Cloud Shell 에서 봇 만들어서 띄우기 (+보안)
2. Cloud Shell 에서 docker 명령어를 사용하여 빌드하고 도커 레지스트리 gcr.io에 등록하기
3. Cloud Shell 에서 kubectl 명령어를 사용하여 여러 서버에 배포하기


## 소스 코드
~~~bash
git clone https://github.com/googlecodelabs/cloud-slack-bot.git
~~~

## Slack Bot
https://queryguru2.slack.com/

## 실습 주요 명령어
### Docker
~~~bash
docker build -t gcr.io/${PROJECT_ID}/slack-codelab:v1 .
gcloud docker -- push gcr.io/bot-guru/slack-codelab:v1
docker ps
docker stop [CONTAINER ID]
~~~

### Kubernetes

- 쿠버네티스 클러스터 만들기
~~~bash
gcloud container clusters create my-cluster \
      --num-nodes=2 \
      --zone=us-central1-f \
      --machine-type n1-standard-1
~~~

- kubectl 명령어
~~~bash
kubectl create secret generic slack-token --from-file=./slack-token
kubectl create -f slack-codelab-deployment.yaml --record
kubectl get pods
~~~
