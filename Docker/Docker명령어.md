# 💡 Docker image 주요 명령어
### 1. search (image 검색)
```
docker search ubuntu
```
- DockerHub로부터 사용가능한 image를 찾는 명령어
- 도커 이미지는 보통 **이미지명[:태그]** 로 이루어져 있음(태그는 보통 버전 정보)
- 이미지명에 태그를 넣지 않으면, 최신 버전(latest)의 이미지가 기본설정됨
- 대부분의 경우 공식 이미지를 사용할 것 - OFFICAIL항목이 [OK]로 표시됨
- 참고로, nuagebec/ubuntu와 같이 /가 있는 경우 /앞(huagebec)이 DokcerHub사용자명임(즉, 공식이미지 아님)

### 2. pull (image 다운로드)
```
docker pull ubuntu
```
- 위와 같이 태그를 안붙이면 디폴트로 latest(최신 버전)을 다운로드 받음
```
docker pull ubuntu:22.04
```
- 위와 같이 특정 태그에 해당하는 버전을 다운로드 받을 수 있음.

### 3. images (image 목록 보기)
```
# docker images 명령
docker images

# docker image ls 명령
docker image ls
```
- `docker images` 명령과 `docker image ls` 명령은 동일한 기능을 수행함

![image](https://github.com/shin-je-woo/TIL/assets/39439576/0317f867-5f72-473c-a0f4-f6397c8970e3)

### 4. rmi (image 삭제)
```
# docker rmi 명령
docker rmi 이미지ID(또는 이미지 REPOSITORY이름[:태그])

# docker image rm 명령
docker image rm 이미지ID(또는 이미지 REPOSITORY이름[:태그])
```

# 💡 Docker Container 주요 명령어
### 1. create (컨테이너 생성)
```
docker create ubuntu
```
- 이미지는 컨테이너로 만들어야 실행 가능함
- 컨테이너 생성시 docker프로그램에서 이름이 자동 부여됨(--name옵션으로 지정 가능)

### 2. ps (컨테이너 확인)
```
# 실행중인 컨테이너 확인
docker ps

# 모든 컨테이너 확인
docker ps -a
```
![image](https://github.com/shin-je-woo/TIL/assets/39439576/dd4baf22-0f6d-405e-98b5-d86adf73e1e0)


### 3. rm (컨테이너 삭제)
```
docker rm 컨테이너 이름(또는 컨테이너ID)
```

### 4. start (컨테이너 실행)
```
docker start 컨테이너 이름(또는 컨테이너ID)
```

### 5. run (컨테이너 생성하고 실행)
```
docker run -it ubuntu:22.04
```
#### Docker run 주요 옵션
- **-i, --interactive***
  - 표준 입력(stdin)을 활성화하며, 컨테이너와 연결(attach)되어 있지 않더라도 표준 입력을 유지합니다.
  - 보통 이 옵션을 사용하여 Bash 에 명령을 입력합니다.
- **-t, --tty**
  - TTY 모드(pseudo-TTY)를 사용합니다.
  - Bash를 사용하려면 이 옵션을 설정해야 합니다.
  - 이 옵션을 설정하지 않으면 명령을 입력할 수는 있지만, 쉘이 표시되지 않습니다.
- **--name**
  - 컨테이너 이름을 설정합니다.
- **-d, --detach**
  - 보통 데몬 모드라고 부르며, 컨테이너가 백그라운드로 실행됩니다.
- **--rm**
  - 컨테이너 종료시 컨테이너 자동 삭제
- **-p, --publish**
  - 호스트와 컨테이너의 포트를 연결합니다. (포트포워딩)
  - <호스트 포트>:<컨테이너 포트>
    - -p 80:80
- **-v, --volume**
  - 호스트와 컨테이너의 디렉토리를 연결하여, 파일을 컨테이너에 저장하지 않고 호스트에 바로 저장합니다. (마운트)

#### 참고) -it 옵션의 의미
- docker 컨테이너에 표준 입력(stdin)을 오픈하고(-i 옵션)
- pseudo tty를 만들어서 (-t 옵션) 표준 입력을 pseudo tty에 연결한다.
- 따라서, pseudo tty를 통해 키보드 입력을 컨테이너의 표준 입력으로 전달한다.

#### 참고) pseudo tty
- tty는 teletypewriter의 약자로, 리눅스에서는 콘솔 또는 터미널을 의미함
- tty를 통해 리눅스에 키보드 입력을 전달할 수 있으며, 하나의 tty 이외에 다양한 터미널에서 접속을 지원하기 위해 두번째 tty부터 가상(pseudo) 이라는 말이 붙어서 pseudo tty라고 이야기함

### 6. stop (컨테이너 종료)
```
docker stop 컨테이너 이름(또는 컨테이너ID)
```
