# 💡 레코드 전송 (메시지 키 X)

- 키 설정을 하지 않으면 키는 null 로 설정되고 값만 전송
- 만약 파티션이 여러 개일 경우 각 데이터는 라운드로빈으로 파티션에 전송

![image](https://github.com/user-attachments/assets/9b3a4eca-e37c-41c2-bf21-0884df933c6d)

# 💡 레코드 전송 (메시지 키 O)

- 키 설정을 하려면 다음 옵션을 추가
- 키 설정을 할 경우 같은 키로 전송된 데이터는 같은 파티션으로 전송

![image](https://github.com/user-attachments/assets/91e4f66f-e790-423b-a19d-d83dc56c5391)

### 참고) 메시지 키 여부

![image](https://github.com/user-attachments/assets/447f13c4-be20-4b78-869b-c12db0bd5c7b)

- 메시지 키가 null 인 경우에는 프로듀서가 파티션으로 전송할 때 레코드 배치 단위(레코드 전송 묶음)로 라운드 로빈으로 전송한다.
- 메시지 키가 존재하는 경우에는 키의 해시값을 작성하여 존재하는 파티션 중 한개에 할당된다.
- 이로 인해 메시지 키가 동일한 경우에는 동일한 파티션으로 전송된다.
