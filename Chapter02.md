# Chapter 02 - 동작 파라미터화 코드 전달하기

소프트웨어를 개발하다보면 유사한 동작을 수행하는데, 그 동작들 내에서 약간의 차이들만 발생하는 경우가 많다. 책에 나온 예를 들어 150그램 이상인 사과를 찾는 작업이라거나, 붉은색인 사과를 찾는 작업 두가지 모두 사과중에서 특정 조건을 만족하는 경우를 찾는것으로 높은 유사성을 가지고 있다. 이렇게 필요한 종류마다 매번 함수를 선언하면, 함수 자체의 갯수도 많아지고 함수의 이름도 길어질것이며 새로운 기능을 추가하거나 기존의 기능을 삭제하는 경우에 장기적인 측면에서 유지보수가 안좋을 것이라고 생각해볼 수 있다.

동작 파라미터화(behavior parameterization)을 활용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 나는 이책을 통해 이 용어를 처음 봤는데, 이 용어의 뜻은 `아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록` 을 의미한다. 즉, 코드블럭의 실행은 나중으로 미뤄져 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다. 즉 코드 블럭에 따라 메서드의 동작이 파라미터화 된다.

책에 나와있는 예제를 통해 단순한 기능부터 시작해서 변화에 대응할 수 있는 코드 작성을 위한 단계를 간단히 밟아보자.

조건에 맞는 사과를 얻는 필터링 과정을 진행한다.

색은 다음과 같이 두가지가 존재한다.

```java
enum Color { RED, GREEN }
```

초기 요구사항이 단순하여 녹색 사과를 얻기 위해 코드를 작성해보자.

```java
public static List<Apple> filterGreenApples(List<Apple> apples) {
	List<Apple> result = new ArrayList();
	for (Apple apple : apples) {
		if (GREEN.eqauls(apple.getColor()) {
			result.add(apple);
		}
	}
	return result;
}
```

빨간사과를 얻어야 한다면 함수명을 `filterRedApples` 로 변경하고 if문의 로직을 바꾸면 원하는 결과를 얻을 수 있겠다.

그러면 동일한 코드를 반복하는 부분이 많은데, 코드를 반복하지 않고 구별하는 방법을 생각해보면, 색을 파라미터화 할 수 있도록 메서드에 파라미터를 추가하면 변화에 대응할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> apples, Color color) {
    List<Apple> result = new ArrayList();
    for (Apple apple : apples) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
    return result;
} 
```

간단하다. 메서드 내 로직에 있던 부분을 파라미터로 꺼내면 조금 더 유연해진다.

이제 사과의 색 뿐만 아니라 사과의 무게로 원하는 사과를 얻고싶다면 파라미터로 무게를 주면 된다.

하지만 색을 구별하는 경우도, 무게를 구별하는 경우도 코드가 매우 유사한 부분이 많고 이는 `DRY(Don't Repeat Yourself)` 원칙을 어기는 것이다.

책에서 다음에 시도할 수 있는 방법으로 플래그를 제공했는데, 사실 기존에 스스로 코드를 작성할 때 플래그를 통해 처리한 경험이 너무나도 많은데, 요구사항의 변경에 따른 코드 수정이 유연하지 않다는 것은 느꼈다.

지금까지 정리하면, 새로운 파라미터를 추가하는 경우 그 파라미터에 관한 값에는 매우 유연해질 수 있으나 (색으로만 구별하는 경우), 무게로 구별하고자 하는 경우 유연성이 매우 떨어질 수 있고, 플래그 또한 확장성이 떨어진다.

## 동작 파라미터화

이러한 경우에 유연한 코드를 작성하기 위하여 동작 파라미터화를 이용할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> apples, ApplePredicate p) {
  List<Apple> result = new ArrayList();
  for (Apple apple : apples) {
      if (p.test(apple)) {
          result.add(apple);
      }
  }
  return result;
}
```

ApplePredicate 인터페이스는 `boolean test(Apple apple)` 하나의 함수만을 가지고 있다. 그러면 이제 중복되는 코드를 작성할 필요가 없어지고, 원하는 조건을 `ApplePredicate` 인터페이스를 구현하는 형태로만 작성해줘서 인자로 전달해주면 되는것이다. 예를 들자면 녹색 사과를 선택하고자 했을때

```java
public class AppleGreenColorPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return GREEN.equals(apple.getColor());
	}
}
```

 됐다. 개인적으로 만족스럽지는 않다.

경험이 많지 않아서 그런가 그런데 보통 이런 조건들은 많이 사용하지 않지 않나?? 라는 생각이 들기도 한다.

그래서 책에서도 조금 더 단계를 나가주고 있다. 👍

```java
List<Apple> redApples = filterApples(apples, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

아까보다 좋아졌다.

그리고 `자바 8` 부터 이렇게 하나의 함수만 사용하는 함수형 인터페이스의 경우 람다로 

```java
List<Apple> redApples = filterApples(apples, apple ->
      RED.equals(apple.getColor())
);
```

와 같이 더 간단하게 줄일 수 있다.

람다도 사실 뒷단원에 나오고 Callable부분의 Future도 뒷단원의 병렬 실행 부분에서 다룬다고 하니 지금 당장 깊게 생각해보지 않고 간단히 Runnable과 Callable의 차이점만 찾아봤는데, 내용정리를 뒷부분의 내용을 배운 후 다시 정리할 것이다.

<details>
<summary>Runnable, Callable</summary>


쓰레드로 실행할 작업을 알려주는 것으로 기존에는 이 인터페이스밖에 알지 못했다. Runnable 인터페이스가 가지고있는 `public void run()` 메서드의 리턴타입을 보다시피 void타입이라 값을 리턴하지 못한다.

```java
public static void main(String[] args) {
    System.out.println("Hello world!");
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() throws Exception {
            throw new Exception("thr exception");
        }
    });
    t.start();
}

==> java: run() in <anonymous Main$1> cannot implement run() in java.lang.Runnable
    overridden method does not throw java.lang.Exception
```

`run()` 메서드에 Exception을 던지도록 하려고 하면, Runnable 인터페이스의 run이 exception을 던지지 않는 형태이므로 try-catch문으로 작성해야 한다.

반면에 Callable 인터페이스를 사용하면 Callable은 리턴타입이 void가 아닌 Object라 값을 리턴할 수 있고, 예외를 던질수 있다.

```java
public static void main(String[] args) throws Exception {
    System.out.println("Hello world!");
    Callable<String> t = new Callable() {
        @Override
        public Object call() throws Exception {
            throw new Exception("thr exception");
        }
    };
    t.call();
}
```




~~둘다 Functional Interface로 람다를 사용하여 간단히 작성할 수 있는데, 어떤 인터페이스를 사용했는지 컴파일러가 구별하지 못하는 경우가 있다고 들었다. 이건 나중에 책에서 Future을 다루면 직접 테스트를 만들어보며 테스트해보고 그 글에 다시 정리하겠다.~~



</details>
