# 💡 nGrinder란?
- nGrinder는 네이버에서 제공하는 서버 부하 테스트 오픈 소스 프로젝트이다.
- 웹 애플리케이션을 서비스하기 전에 서버가 얼마나 많은 사용자를 수용할 수 있는지 요청을 전송해봄으로써 서버의 성능을 측정해볼 수 있다.
- nGrinder는 다음과 같은 주요 기능을 제공한다.
  - 스크립트 레코딩: 웹 브라우저에서 사용자 동작을 녹화하여 테스트 스크립트를 생성할 수 있다.
  - 분산 테스트: 다수의 에이전트 머신을 사용하여 대규모 테스트를 수행할 수 있다.
  - 실시간 모니터링: 테스트 진행 중에 성능 지표를 실시간으로 모니터링할 수 있다.
  - 분석 및 리포팅: 테스트 결과를 다양한 그래프와 테이블로 분석하고, 리포트를 생성할 수 있다.
 
 # 💡 nGrinder 구조
 ![image](https://github.com/shin-je-woo/TIL/assets/39439576/87cf8754-500f-4576-86c4-fb134f3eae6c)

- nGrinder는 Controller와 Agent로 이루어져 있다.

## Controller
- 성능 측정을 위한 웹 인터페이스를 제공하며, Web Application으로 Tomcat과 같은 웹서버 엔진을 이용하여 구동할 수 있다.
- 사용자와의 인터페이스를 담당하여 테스트 프로세스 정의, 스크립트 작성 등을 지원한다.
- 스크립트 수정 기능 제공하며, Agent에 스크립트를 전달하여 테스트를 위임한다.
- 테스트 프로세스 조정
- 테스트 통계를 수집하고 표시

## Agent
- Controller의 명령을 받아 실행하며, target에 프로세스와 스레드를 실행시켜 부하를 발생시킨다.
- 에이전트 모드에서 실행할 때 대상 시스템에 부하를 주는 프로세스 및 스레드를 실행한다.
- 모니터 모드에서 실행 시 대상 시스템 성능(CPU/메모리) 모니터링

# 💡 nGrinder 설치 및 실행
- nGrinder는 Docker로도 제공되고, 아래 깃헙에서 컨트롤러.war 설치파일을 제공한다.
- [nGrinder 깃헙 - 설치페이지](https://github.com/naver/ngrinder/releases)
- 컨트롤러.war 파일을 다운로드 하고, 아래 명령어로 프로그램을 실행한다.
  - java -jar ngrinder-controller-3.5.5-p1.war --port=8300
- 설정한 8300으로 http 접속 후 id/password에 admin/admin 입력하여 로그인
- 아래 이미지처럼 agent 다운로드

![image](https://github.com/shin-je-woo/TIL/assets/39439576/12601b1b-9e6a-4a81-a492-7d0eeb1cf9ba)

- 다운로드된 에이전트 tar 압축을 풀고, agent를 실행한다.
  - mac: `./run_agent.sh`
  - windows: `./run_agent.bat`
- 다시 페이지로 이동하여 Agent Management 탭으로 이동
- agent가 정상적으로 띄어져 있는지 확인(아래 이미지와 같이 agend 프로그램이 실행되어 있어야 함

![image](https://github.com/shin-je-woo/TIL/assets/39439576/475d7990-91d2-4c35-93a1-f213aba3cf4a)

# 💡 시나리오 정의 및 실행 스크립트 작성
- Script 탭으로 이동해서 create script를 클릭해 스크립트를 작성할 수 있다.
- 스크립트 이름과 테스트할 URL을 지정해준다. 그 후 create 버튼 클릭하여 스크립트 생성한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/d91cdf58-909b-4893-a723-e4649a5fd242)

- 그 후, validate 버튼을 클릭하여 스크립트가 정상인지 확인한다.

# 💡 부하테스트 작성 및 실행
- Performance Test 탭으로 이동하고, create Test 버튼 클릭
- 만들어둔 스크립트를 설정하고, 테스트할 시나리오를 작성한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/a8de3e36-f851-4c47-9c3d-6becac24f3c2)

# 💡 ngrinder 관련 용어 정리
### TPS (처리량)
- TPS는 "Transactions Per Second"의 약어로, "초당 처리가능한 트랜잭션의 수"
- TPS가 100이라면 초당 처리 할 수 작업이 100이다 라고 생각하면 된다.
- TPS 수치가 높을 수록 짧은 시간에 많은 작업을 처리할 수 있다고 생각하면 된다.

 ### Vuser
- 가상 사용자(Virtual User)
- ex) "vUser 10"은 가상 사용자(Virtual User)가 10명을 의미한다.
- 이것은 부하 테스트 시나리오에서 10명의 가상 사용자가 동시에 시스템 또는 애플리케이션에서 동작하고 있다는 것을 의미한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/62517a94-d87d-4e26-981f-e0d05da3b0cb)

- TPS: 평균 TPS
- Peak TPS: 최고 TPS
- Mean Test Time: 평균 테스트시간
- Executed Tests: 테스트 실행 횟수
- Successful Tests: 테스트 성공 횟수
- Errors: 에러 횟수
- Run time: 테스트 실행시간

> TPS 수치는 높고 Mean Test Time은 낮을 수록 성능적으로 긍정적이다.
