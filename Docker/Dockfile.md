# 💡 Dockerfile 이란?
- docker에서 이미지를 생성하기 위한 용도로 작성하는 파일이다.
- Dockerfile 문법으로 이미지 생성을 위한 스크립트를 작성할 수 있고, 이를 기반으로 이미지를 생성할 수 있다.
- docker build [옵션] [작성한 dockerfile 경로] 와 같은 명령으로 Dockerfile 을 기반으로 이미지를 만들 수 있다.

# 💡 Dockerfile 주요 명령
### 1. FROM
- FROM은 베이스 이미지(Base image)를 지정하는 명령이다.
- FROM은 Dockerfile에서 반드시 작성해야 하는 명령이다.
```Dockerfile
FROM httpd:alpine
```

### 2. RUN
- docker는 이미지 생성시 각 단계를 layer로 나누어 작성한다.
- RUN 명령은 이미지 생성시 layer를 만드는 명령으로, 보통 베이스 이미지에 패키지(프로그램)을 설치하여 새로운 이미지를 만들 때 사용한다.
- RUN 뒤에 소프트웨어/라이브러리 설치 명령어 또는 파일/디렉토리 생성 명령어를 작성하면 된다.
- 즉, RUN은 이미지 생성시 실행하는 명령이다.
```Dockerfile
FROM ubuntu:18.04

# 패키지(프로그램) 정보 업데이트
RUN apt-get update
# apache2 패키지(프로그램) 설치
RUN apt-get install -y apache2
```

### 3. CMD
- 컨테이너가 시작될 때 실행할 커맨드를 지정하는 명령이다.
- 도커 파일 내에 여러 CMD가 있을 경우 맨 마지막에 설정된 CMD만 적용된다.
```Dockerfile
FROM httpd:apline
LABEL maintainer="shinjw0926@naver.com"

CMD ["/bin/sh", "-c", "httpd-foreground"]
```

### 4. ENTRYPOINT
- ENTRYPOINT 또한 컨테이너 시작 시 실행될 커맨드를 지정한다.
- CMD 명령은 docker run 시에 함께 넣어지는 CMD명령이 덮어지지만, ENTRYPOINT는 덮어지지 않는다.
- 따라서, 반드시 실행해야 할 커맨드를 작성할 때 사용한다.
  - 이 때, docker run 시 함께 넣어지는 커맨드는 ENTRYPOINT의 인자로 넣어지게 된다.
- 다음과 같이 도커파일이 작성되어 있다고 가정해보자.
```Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world"]
```
- 여기서 CMD가 ENTRYPOINT의 추가 인자가 되는 형태이다.
- 컨테이너 실행 시 커맨드 지정 없이 일반적으로 실행할 경우 결과는 다음과 같다.
```
$ docker run -it test

Hello world
```
- 만약 컨테이너를 실행할 때 커맨드를 지정하는 경우는 다음과 같다.
```
$ docker run -it test shinjewoo

Hello shinjewoo
```
- 즉, ENTRYPOINT의 인자인 'Hello'가 아닌 CMD의 인자인 'world'가 대체 된다.
- 그래서 보통 CMD는 ENTRYPOINT와 같이 쓰이면서 default값을 갖는 인자 역할을 하는 경우가 많다.

### 5. LABEL
- key-value 형식으로 작성된 메타데이터를 이미지에 추가한다.
- 보통 저자, 버전, 설명, 작성일자 등을 지정할 때 사용한다.
```Dockerfile
FROM httpd:alpine

LABEL maintainer = "shinjw0926@naver.com"
LABEL version="1.0.0"
LABEL description="Let's test Dockerfile"
```

### 6. ENV
- 컨테이너 내부에서 사용할 환경 변수를 지정한다.
```Dockerfile
FROM mysql:5.7

ENV MYSQL_ROOT_PASSWORD=12345678
ENV MYSQL_DATABASE=dbtest
```

### 7. EXPOSE
- 컨테이너 외부에 오픈할 포트를 설정한다.
```Dockerfile
FROM ubuntu:18.04
LABEL maintainer="shinjw0926@naver.com"

RUN apt-get update
RUN apt-get install -y apache2 apt-utils

EXPOSE 80
```
- 프로토콜을 따로 지정하지 않을 경우 tcp로 적용되며, 다른 프로토콜을 사용할 경우 다음과 같이 작성하면 된다.
```Dockerfile
EXPOSE 80/udp
```

### 8. COPY
- Host 내에 있는 파일 또는 디렉토리를 컨테이너의 파일시스템으로 복사한다.
```Dockerfile
FROM httpd:alpine

COPY ./MY_HTML /usr/local/apache2/htdocs
```
