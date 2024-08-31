# 💡 컨슈머 그룹 리스트 보기, 상세보기

- 컨슈머 그룹은 따로 생성하는 명령을 날리지 않고 컨슈머를 동작할 때 컨슈머 그룹이름을 지정하면 새로 생성된다.
- 생성된 컨슈머 그룹의 리스트는 kafka-consumer-groups.sh 명령어로 확인할 수 있다
- 리스트 보기: `--list`
- 상세 보기: `--group`, `--describe`

![image](https://github.com/user-attachments/assets/236e40d8-912b-4c0e-9459-0a17a644cbd6)

### 상세 보기

- `--describe` 옵션을 사용하면 해당 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지 상태를 확인할 수 있다.
- 파티션 번호, 현재까지 가져간 레코드의 오프셋, 파티션 마지막 레코드의 오프셋, 컨슈머 랙, 컨슈머 ID, 호스트를 알 수 있기 때문에 컨슈머의 상태를 조회할 때 유용하다.
- CURRENT-OFFSET : 현재까지 가져간 레코드의 오프셋 위치
- LOG-END-OFFSET : 마지막 레코드의 오프셋 위치
- LAG : LOG-END-OFFSET - CURRENT-OFFSET = 지연된 레코드 수

![image](https://github.com/user-attachments/assets/bf74d873-6ed9-4627-a342-39c2da470d40)

# 💡 오프셋 리셋

- `--reset-offsets` 옵션을 추가하여 오프셋을 리셋 할 수 있다.

### 오프셋 리셋 옵션 종류

- `--to-earliest` : 가장 처음 오프셋으로 리셋
- `--to-latest` : 가장 마지막 오프셋으로 리셋
- `--to-current` : 현 시점 기준 오프셋으로 리셋
- `--to-datetime {YYYY-MM-DDTHH:mmSS.sss}` : 특정 일시로 오프셋 리셋 (레코드 타임스탬프 기준)
- `--to-offset {long}` : 특정 오프셋으로 리셋
- `--shift-by {+/- long}` : 현재 컨슈머 오프셋에서 앞 뒤로 옮겨서 리셋

![image](https://github.com/user-attachments/assets/6d3fa154-f676-4d12-a757-4d2bde5c7ffd)
