## Kubernetes Basics
Q1) 왜 쿠버네티스를 쓰나? 컨테이너를 여러 노드에 배포할 때의 API를 제공해준다.

Q2) POD가 뭔데? 네트워크와 스토리지를 공유하는 컨테이너들

Q3) API 서버로 request 보낼 때 쓰는 command는? kubectl

### QwikLab >> Kubernetes
github에서 code를 가져다가 앱을 배포, 관리 업그레이드하는 예제

##### 1: 5개 노드로 된 쿠버네티스 클러스터를 커맨드로 생성
~~~
> git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
> cd orchestrate-with-kubernetes/kubernetes
> ls
./deployment
./pods
./services
./nginx
./tls
> gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
> kubectl version
> kubectl cluster-info
~~~
클라우드 콘솔로 가서 Compute Engine > VM Instance 5개가 떠 있는지 확인

##### 2: 탭을 누르면 자동완성 되는 기능 테스트
~~~
> source < (kubectl completion bash)
~~~
실행 후 tab을 눌러본다.

#####  3: nginx pod를 1개 띄웠다가 3개로 늘려주는 예제 (서버 부하 심할 때 사용)
~~~
> kubectl run nginx --image=nginx:1.10.0
> kubectl get pods
> kubectl expose deployment nginx --port 80 --type LoadBalancer
> kubectl get services
> kubectl scale deployment nginx --replicas 3
> kubectl get pods
~~~
내부적으로 deployment를 생성
get pods 명령어로 ngnx 컨테이너가 있는 single pod가 실행되는 것을 확인

##### 4: nginx 서비스 내리기
~~~
> kubectl delete deployment nginx
> kubectl delete service nginx
~~~

##### 5: Pods
kubectl explain 명령어로 설명 보기
~~~
> kubectl explain pods
> kubectl explain pods.spec.containers
~~~

yaml 설정 확인 후 pod를 생성한 뒤 10080 포트를 80으로 forwarding해서 연결해본다
(pod는 monolith라고 하는 하나의 컨테니어로 구성, HTTP는 80포트에서)
~~~
> kubectl create -f pods/monolith.yaml
> kubectl describe pods monolith
> kubectl get pods
> kubectl port-forward monolith 10080:80
> TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
> curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
~~~

로그 확인
~~~
> kubectl logs -f monolith
2018/07/21 16:03:21 Starting server...
2018/07/21 16:03:21 Health service listening on 0.0.0.0:81
2018/07/21 16:03:21 HTTP service listening on 0.0.0.0:80
127.0.0.1:58130 - - [Sat, 21 Jul 2018 16:35:39 UTC] "GET / HTTP/1.1" curl/7.52.1
127.0.0.1:58142 - - [Sat, 21 Jul 2018 16:35:57 UTC] "GET /secure HTTP/1.1" curl/7.52.1
127.0.0.1:58160 - - [Sat, 21 Jul 2018 16:36:28 UTC] "GET /login HTTP/1.1" curl/7.52.1
~~~

Health Check
~~~
> kubectl create -f pods/healthy-monolith.yaml
> kubectl describe pod healthy-monolith
~~~


##### 6: Services
~~~
> kubectl create secret generic tls-certs --from-file tls/
> kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
> cat nginx/proxy.conf
> kubectl create -f pods/secure-monolith.yaml
> kubectl create -f services/monolith.yaml
> gcloud compute firewall-rules create allow-monolith-nodeport --allow=tcp:31000
> gcloud compute instances list
> https://<EXTERNAL_IP>:31000
~~~
