아래 자료는 qwik lab (https://google.qwiklabs.com)을 정리한 자료 입니다.


## Docker
~~~bash
docker images
docker run hello-world
docker ps -a
docker log [container_id]
~~~

## Docker 이미지 만들기
~~~bash
mkdir test && cd test

Dockerfile을 만든다
~~ bash
cat > Dockerfile <<EOF

# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
~~~

같은 디렉토리에 app.js를 만든다
~~~
cat > app.js <<EOF
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
~~~

repository node-app 으로 빌드
이미지 확인에서 보여지는 node 이미지는 node-app의 base 이미지임
~~~bash
docker build -t node-app:0.1 .
docker images
~~~

실행
~~~bash
docker run -p 4000:80 --name my-app -d node-app:0.1
curl http://localhost:8080
~~~

디버그
~~~bash
docker logs -f [container_id]
~~~


Google container Registry (GCR)로 publish하기
~~~
gcloud docker -- push gcr.io/qwiklabs-gcp-527a7cbd4e4ed89a/node-app:0.2
gcloud config list project
http://gcr.io/[project-id]/node-app
~~~
