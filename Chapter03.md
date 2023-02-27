# Chapter 03 - 람다 표현식

익명 클래스로 다양한 동작을 추가할 수 있다. 그러나, 코드가 깔끔하지는 않다. 깔끔하지 않은 코드는 동작 ㅏㅍ라미터를 실전에 적용하는 것을 막는 요소다.

## 함수형 인터페이스

함수형 인터페이스는 추상 메서드가 오직 하나면 함수형 인터페이스이다.

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

```java
public static void main(String[] args) {

    Runnable r1 = () -> System.out.println("Hello world 1");
    Runnable r2 = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello world 2");
        }
    };

    process(r1);
    process(r2);
    process(() -> System.out.println("Hello world 3"));

}
public static void process(Runnable r) {
    r.run();
}
```

람다 표헌식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 우선 기억하자.

예를 들면 위에 process 함수에서 Runnable 인터페이스의 run 메서드 시그니처는 인수가 없고, void를 리턴한다. `() -> System.out.println("Hello world 3")` 도 마찬가지로 인수가 없으며 void를 반환하는 람다 표헌식이다.

왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까? 에 대한 답으로는 20장, 21장에 다룰 특별한 표기법을 추가하는 방법도 대안으로 고려했으나, 언어 설계자들이 언어를 더 복잡하게 만들지 않는 현재 방법을 택했다.