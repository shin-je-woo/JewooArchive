# 💡 도커 컴포즈란?
- Docker Compose는 여러 컨테이너를 모아서 하나의 서비스로 제공하기 위한 도구
- 여러 개의 컨테이너가 하나의 애플리케이션으로 동작할 때 이를 실행하려면 각 컨테이너를 하나씩 생성해야 한다.
  - 예를 들어, 백엔드 컨테이너/프론트엔드 컨테이너/데이터베이스 컨테이너 등 여러 개의 컨테이너를 각각 생성해야 한다.
- 즉, 도커 컴포즈를 이용하면 각 컨테이너를 매번 run명령어로 각각 실행하는 불편함을 해소할 수 있다.
- 도커 컴포즈는 여러 개의 컨테이너의 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성하는 방식으로 동작한다.
- 도커 컴포즈의 설정 파일은 run 명령어의 옵션을 그대로 사용할 수 있으며, 각 컨테이너의 의존성, 네트워크, 볼륨 등을 함께 정의할 수 있다.

# 💡 도커 컴포즈 사용법
- 도커 컴포즈는 docker-compose.yml 이라는 `YAML` 형식의 파일로 작성한다.

### 기본 구조
- 기본적으로 다음과 같이 4가지의 큰 카테고리로 작성하며, 보통 version과 sevices만 설정하여 많이 사용한다.
  - volumes는 각 컨테이너 설정의 volumes로 설언할 수 있고, networks는 컨테이너간 네트워크 분리를 위한 추가 설정부분이다.

```yml
# Docker Compose 파일 포맷 버전 지정
version: "3"

# 컨테이너 설정
services:

# 컨테이너에서 사용하는 volume 설정으로 대체 가능 (옵션)
volumes:

# 컨테이너간 네트워크 분리를 위한 추가 설정 부분 (옵션)
networks:
```

### docker-compose.yml 예시
```yml
version: "3"

services:
  nginxproxy:
    depends_on:
      - nginx
      - db
      - wordpress
    image: nginx:latest
    container_name: proxy 
    ports:
      - "80:80"
    restart: always
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/nginx.conf"

  nginx:
    image: nginx:latest
    container_name: myweb 
    restart: always
    volumes:
      - "./myweb:/usr/share/nginx/html"

  db:
    image: mysql:5.7
    container_name: mysqldb 
    volumes:
      - "mydb:/var/lib/mysql"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpressroot
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    build:
      context: ./wp
      dockerfile: Dockerfile
    container_name: wp 
    restart: always
    volumes:
      - "./html:/var/www/html"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  mydb:
```

- `image`, `volumes`, `environment`, `ports` 등등 Docker run 명령의 옵션과 유사한 내용이 많다.

### depends_on
- 여러 컨테이너를 Docker Compose로 실행할 경우 각 컨테이너가 실행을 시작하는 시점이 다르다.
- `depends_on` 은 컨테이너간 의존관계를 정의하여 실행시점을 조정해주는 역할을 한다.
- 워드프레스처럼 MySQL 컨테이너가 먼저 있어야할 경우 예시와 같이 `depends_on`을 명시한다.

### restart
- 컨테이너가 다운되었을 경우, 재시작여부를 결정한다.
- always로 설정하면 컨테이너가 다운되면 항상 재시작한다.

### build
- 이미지를 Dockerfile로 작성할 때 사용한다.
  - context: Dockerfile이 있는 디렉토리
  - dockerfile: Dockerfile 파일명
 
# 💡 도커 컴포즈 실행/중지
- 도커 컴포즈는 `docker-compose` 명령을 사용한다.
- 자주 사용하는 커맨드로 `up` `down` `stop`이 있다.
- `up` 커맨드는 컴포즈 파일에 정의된 컨테이너 및 네트워크, 볼륨을 생성한다.
- `down` 커맨드는 생성된 컨테이너와 네트워크를 종료하고 삭제한다.

```
# -f 옵션은 docker-compose.yml의 이름이 아닌 특정 이름으로 도커컴포즈파일을 작성했을 때 사용한다.
# -f 옵션을 지정하지 않으면 현재 디렉토리의 docker-compose.yml파일을 실행한다.
docker-compose -f [정의_파일_경로] up [옵션]
docker-compose -f [정의_파일_경로] down [옵션]
```

- 침고) `down` 커맨드를 사용하면 컨테이너와 네트워크는 삭제되지만, 이미지와 볼륨은 따로 삭제해야 한다.
