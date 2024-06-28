# Previous

- 자바에서 Future 와 Callback 은 비동기 프로그래밍에서 사용되는 패턴으로 비동기 작업의 결과를 처리하거나 작업이 완료되었을 때 수행 할 동작을 정의하며 사용한다.
- 자바에서는 Future 인터페이스와 구현체들을 제공하고 있으며 다양한 Callback 패턴을 활용하고 있다.
- 두 패턴은 사용하는 방식과 특징이 다르며 각 상황에 맞게 구현해서 사용하도록 한다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/126f56c3-2894-46c3-8993-5844d1da2149)

### 두 패턴이 필요한 이유

- 비동기 작업에서 스레드 간 결과를 받을 방법이 필요하다. (Future)
- 비동기 작업은 스레드 간 실행의 흐름이 독립적이기 때문에 비동기 작업의 완료 시점에 결과를 얻을 수 있어야 한다. (Callback)

# 💡 Future 를 활용한 비동기 작업

![image](https://github.com/shin-je-woo/TIL/assets/39439576/c4503dc5-dec6-4146-8d4f-42515d6eb06c)

# 💡 Callback 을 활용한 비동기 작업

![image](https://github.com/shin-je-woo/TIL/assets/39439576/444a6536-4813-4858-8857-3394f8171739)

# 💡 Future 와 Callback 을 혼합한 비동기 작업

![image](https://github.com/shin-je-woo/TIL/assets/39439576/7eaa3e42-f0fe-4042-bf05-309a6926ef42)
