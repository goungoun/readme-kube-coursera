## Introduction to Containers and Docker
Q1) vm에 비해 컨테이너의 장점은?
|3:26 컨테이너는 Application code와 dependency만 들어가고 Kernel이 완벽하게 분리된다.
They share parts of the OS, but not common dependencies, so they start fast and can run anywhere with the same kernel.

Q2) 도커 파일에 들어가는 action - FROM, CMD에 체크
~~~
FROM ubuntu:15.04
COPY . /app
RUN make /app
CMD python /app/app.py
~~~

https://docs.docker.com


Q3) 도커 이미지를 구글 레지스트리에 publish하기 위한 default hostname
gcr.io

시나리오:
로그인하면 k8s-workshop-module-1-lab 이름으로 된 vm inistance가 이미 만들어져있음,
ls /  커맨드를 실행했을 때 kickstart directory가 있는지 확인 (8분정도 소요)

- 도커 그룹에 유저 등록 (groups 명령어로 확인 : -a 옵션 필수, 그렇지 않으면 기존 group을 날림)
- /kickstart 디렉토리의 tornado와 미리 만들어져있는 Dockerfile 확인. pip install
- 도커 이미지 빌드
- 도커 이미지를 구글 클라우드 레지스트리에 등록
- 도커 컨테이너 실행

### QwikLab >> Docker
##### 1 : 도커 이미지 굽기
~~~
> sudo usermod -aG docker $USER
> groups|grep docker
> cd /kickstart
> docker build -t py-web-server:v1 .
> docker run -d -p 8888:8888 --name py-web-server -h my-web-server py-web-server:v1
> docker ps -a
> curl http://localhost:8888
> docker rm -f py-web-server
~~~

##### 2 : 구글 컨테이너 레지스트리에 등록
~~~
> docker build -t "gcr.io/${GCP_PROJECT}/py-web-server:v1" .
> PATH=/usr/lib/google-cloud-sdk/bin:$PATH
> gcloud auth configure-docker
> docker push gcr.io/${GCP_PROJECT}/py-web-server:v1
~~~

##### 3: public하게 접근 가능하도록 퍼미션
~~~
> gsutil defacl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
> gsutil acl ch -r -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
> gsutil acl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
~~~

gcr.io란? 지속적 배포 시스템과 호환되는 비공개 Docker 저장소.
Spinnaker( An open source, multi-cloud continuous delivery platform )와 연동
