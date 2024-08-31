# 💡 토픽 생성하기

- `--create` 옵션은 카프카 클러스터 정보와 토픽 이름만으로 토픽을 생성할 수 있다.
- 클러스터 정보와 토픽 이름은 토픽을 만들기 위한 필수 값이다.
- 이렇게 만들어진 토픽은 파티션 개수, 복제 개수 등과 같이 다양한 옵션이 포함되어 있지만 모두 브로커에 설정된 기본값으로 생성되었다.

```sh
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test-topic
```

![image](https://github.com/user-attachments/assets/8ab70c3f-e0ac-4ede-8a4b-f7c54305ef87)

# 💡 원하는 옵션 입력하여 토픽 생성하기

- 파티션 개수, 복제 개수, 토픽 데이터 유지 기간 옵션들을 지정하여 토픽을 생성하고 싶다면 다음과 같이 명령을 실행하면 된다.
- 생성된 토픽들의 이름을 조회하려면 --list 옵션을 사용한다.

```sh
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --partitions 10 --replication-factor 1 --topic test-topic2 --config retention.ms=172800000
```

![image](https://github.com/user-attachments/assets/3d200a63-9091-42a2-b1fc-e71ecfa6345a)

# 💡 토픽 설정정보 확인하기

- `--describe` 옵션을 사용하면 토픽의 설정정보를 확인할 수 있다.

```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic test-topic --describe
```

# 💡 토픽 설정 변경하기

- `--alter` 옵션을 사용하면 파티션 개수를 늘릴 수 있다.

![image](https://github.com/user-attachments/assets/dcd0394b-0b78-4219-b711-6db527f437a2)

### 📌 주의!

- 파티션 개수를 늘릴 수 있지만 줄일 수는 없다.
- 다시 줄이는 명령을 내리면 `InvalidPartitionsException`이 발생한다.
- 만약 파티션 개수를 줄여야 할 때는 토픽을 새로 만드는편이 좋다.

![image](https://github.com/user-attachments/assets/effaf1b0-65a6-48d5-81ad-bb97eadf1c6b)
