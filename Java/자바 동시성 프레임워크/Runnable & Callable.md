# 💡 Runnable & Callable

- Runnable 과 Callable 은 모두 별도의 스레드에서 실행할 수 있는 작업을 나타내는 데 사용되는 인터페이스이다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/2b97c1ff-fc34-4dd4-8f8b-c6adf96f0daf)

![image](https://github.com/shin-je-woo/TIL/assets/39439576/057412f7-53b7-4653-8379-c52f2380199d)

- 기존의 Runnable 인터페이스는 결과를 반환할 수 없다는 한계점이 있었다.
- 반환값을 얻으려면 공용 메모리나 파이프 등을 사용해야 했는데, 이러한 작업은 상당히 번거롭다.
- 그래서 Runnable의 발전된 형태로써 Java5에 함께 추가된 제네릭을 사용해 결과를 받을 수 있는 Callable이 추가되었다.

![image](https://github.com/shin-je-woo/TIL/assets/39439576/adbd21b8-558c-49bb-a3b8-70d4bbff826a)

# 💡 Callable & Future

![image](https://github.com/shin-je-woo/TIL/assets/39439576/9c9df0c8-23de-4ca4-b5e4-aba7b313e1f9)
