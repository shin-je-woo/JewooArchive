# 💡 토픽에 저장된 메시지 값 가져오기

- 토픽으로 전송한 데이터는 kafka-console-consumer.sh 명령어로 확인할 수 있다.
- 이때 필수 옵션으로 --bootstrap-server에 카프카 클러스터 정보, --topic에 토픽 이름이 필요하다.
- 추가로 `--from-beginning` 옵션을 주면 토픽에 저장된 가장 처음 데이터부터 출력한다.

![image](https://github.com/user-attachments/assets/da0b6b6f-1225-4b02-a7fe-2e4701eedb01)

# 💡 토픽에 저장된 메시지 키, 값 가져오기

- 레코드의 키와 값을 함께 출력하고 싶다면 `--property` 옵션을 사용하면 된다.

![image](https://github.com/user-attachments/assets/fa1e43cc-0532-48e9-8245-9b6bbd46d6bd)

# 💡 최대 컨슘 메시지 갯수 지정하기

- `--max-messages` 옵션을 사용하면 최대 컨슘 메시지 갯수를 설정할 수 있다.

![image](https://github.com/user-attachments/assets/9edf5579-22ba-422e-b9ef-93d5c88fe2d0)

# 💡 특정 파티션에 있는 메시지 가져오기

- `--partition` 옵션을 사용하면 특정 파티션만 컨슘할 수 있다.

![image](https://github.com/user-attachments/assets/7f311d8a-1aab-4455-adf7-e4864982208d)

# 💡 그룹 지정해서 메시지 가져오기

- `--group` 옵션으로 그룹을 지정하면 읽은 데이터를 commit하여 다음에 해당 그룹으로 데이터를 읽으면 커밋된 데이터 이후의 데이터부터 가져온다.
- 해당 커밋내용은 `__consumer_offsets` 토픽에 저장된다.

![image](https://github.com/user-attachments/assets/725508fc-08a6-4587-a2ec-0dfd8aee0454)
