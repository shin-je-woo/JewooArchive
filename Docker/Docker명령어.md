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
