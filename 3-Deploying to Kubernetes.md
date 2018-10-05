## Deploying to Kubernetes
Q1) ReplicaSet의 목적? pod를 원하는 수 만큼 생성하기 위해서

Q2) rolling deployments가 trigger 되는 시점은? deployment pod 템플릿이 변경되었을 때 (실습)

Q3) 두번째 카나리 배포에서 instance를 어떻게 선택하나?
배포 시 인스턴스를 가리키는 서비스가 동일한 selector를 가지고 있음
The service points to instances in both deployments with a common selector.


### QwikLab >> Deploying to Kubernetes
##### 1. 셋업
~~~
> git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
> cd orchestrate-with-kubernetes/kubernetes
> gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
~~~

##### 2. 배포 생성
deployment, services (with your pod running, put it behind a service.)
~~~
> cat deployments/auth.yaml
> kubectl create -f deployments/auth.yaml
> kubectl create -f deployments/hello.yaml
> kubectl create -f deployments/frontend.yaml
> kubectl create -f services/auth.yaml
> kubectl create -f services/hello.yaml
> kubectl create -f services/frontend.yaml
> kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
> kubectl create secret generic tls-certs --from-file tls/
~~~

##### 3. 확인
~~~
> kubectl get deployments
> kubectl get replicasets
> kubectl get pods
~~~

~~~
> kubectl get services frontend
> curl -ks https://<EXTERNAL-IP>
~~~

##### 4. Scale
replica 옵션을 바꿔가면서 확인해본다.
~~~
> kubectl scale deployment hello --replicas=5
> kubectl get pods | grep hello- | wc -l
~~~

#### 5. Rolling Updates
hello app 버전을 2로 바꾸면 rolling update가 시작
DESIRED, CURRENT, READY가 모두 0인 것이 생겨있음
~~~
> kubectl edit deployment hello
> kubectl get replicaset
> kubectl rollout history deployment/hello
containers:
- name: hello
  image: kelseyhightower/hello:2.0.0

> kubectl get replicaset
  NAME                  DESIRED   CURRENT   READY     AGE
  auth-6c8d5c8688       1         1         1         1h
  frontend-86dfbfdcc7   1         1         1         1h
  hello-649b5864dc      3         3         3         24m
  hello-75d9768d4d      0         0         0         1h

> kubectl rollout history deployment/hello
deployments "hello"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
~~~

멈추기
~~~
kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
~~~

rollout undo 커맨드로 이전 버전으로 돌려본다.
~~~
> kubectl rollout undo deployment/hello
> kubectl rollout history deployment/hello
> deployment "hello" successfully rolled out
> kubectl rollout undo deployment/hello
> kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
~~~

#### 6. Canary Deployments
~~~
> cat deployments/hello-canary.yaml
> kubectl create -f deployments/hello-canary.yaml
> kubectl get deployments
> curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
~~~

#### 7. Blue-Green Deployment
~~~
> kubectl apply -f services/hello-blue.yaml
> kubectl create -f deployments/hello-green.yaml
> curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
> kubectl apply -f services/hello-green.yaml
> curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
~~~


clean-up
~~~
kubectl delete deployment hello-canary
~~~
