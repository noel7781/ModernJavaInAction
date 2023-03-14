# Chapter 05 - 스트림 활용 (2)

## 실전 연습 퀴즈

```java
List<String> words = Arrays.asList("Hello", "World");
System.out.println(
        words.stream()
                .map(word -> word.split(""))
                .flatMap(Arrays::stream)
                .distinct()
                .collect(Collectors.toList())
);

List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);

List<int[]> pairs = numbers1.stream()
        .flatMap(i ->
                numbers2.stream()
                        .map(j ->
                                new int[]{i, j}))
        .collect(Collectors.toList());

Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
        new Transaction(brian, 2011, 300),
        new Transaction(raoul, 2012, 1000),
        new Transaction(raoul, 2011, 400),
        new Transaction(mario, 2012, 710),
        new Transaction(mario, 2012, 700),
        new Transaction(alan, 2012, 950)
);

// Q1
System.out.println(
        transactions.stream()
                .filter(trx -> trx.getYear() == 2011)
                .sorted(Comparator.comparing(Transaction::getValue))
                .collect(Collectors.toList())
);

// Q2
System.out.println(
        transactions.stream()
                .map(Transaction::getTrader)
                .map(Trader::getCity)
                .distinct()
                .collect(Collectors.toList())
);

// Q3
System.out.println(
        transactions.stream()
                .map(Transaction::getTrader)
                .filter(trader -> trader.getCity().equals("Cambridge"))
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList())
);

// Q4
System.out.println(
        transactions.stream()
                .map(Transaction::getTrader)
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList())
);

// Q5
System.out.println(
        transactions.stream()
                .map(Transaction::getTrader)
                .anyMatch(trader -> trader.getCity().equals("Milano"))
);

// Q6
System.out.println(
        transactions.stream()
                .filter(trx -> trx.getTrader().getCity().equals("Cambridge"))
                .map(Transaction::getValue)
                .collect(Collectors.toList())
);

// Q7
System.out.println(
        transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::max)
                .get()
);

// Q8
System.out.println(
        transactions.stream()
                .map(Transaction::getValue)
                .reduce(Integer::min)
                .get()
);
```

스트림으로 계산하게 되면 sum을 구하는 과정등에서 Integer을 기본형으로 언박싱 해야 한다. int를 직접 호출하면 더 좋겠지만 map 메서드가 Stream<T>를 생성하기 때문에 직접 호출할 수 없다. 이 문제를 해결하기 위해 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공한다.

### 기본형 특화 스트림

자바 8에서는 세 가지 기본형 특화 스트림(IntStream, DoubleStream, LongStream)을 제공한다. 특화 스트림은 오직 박싱과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지 않는다.

ex) `mapToInt` 는 IntStream을 반환한다. → IntStream은 mix, min, average, sum 등 다양한 유틸리티 메서드를 지원한다.

숫자 스트림을 만든 다음 원 상태인 특화되지 않은 스트림으로 복원할 수 있는데, `.boxed()` 를 활용하여 특화 스트림을 일반 스트림으로 변환할 수 있다.

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();
```

추가적으로 기본값이 없는 OptionalInt, OptionalDouble, OptionalLong이 있다.

IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공한다. (ranngeClosed는 종료값이 결과에 포함)

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
																.filter(n -> n % 2 == 0);
```

## 스트림 만들기

### 값으로 스트림 만들기

임의의 수를 인수로 받는 정적 메서드 `Stream.of` 를 이용해서 스트림을 만들 수 있다.

`Stream.empty` 를 활용하여 스트림을 비울 수 있다.

null이 될 수 있는 객체는 `Stream.ofNullable` 을 활용하여 객체를 만들 수 있다.

### 배열로 스트림 만들기

배열을 인수로 받는 정적 메서드 `[Arrays.stream](http://Arrays.stream)` 을 이용하여 스트림을 만들 수 있다.

```java
int[] numbers = {2, 3, 5};
int sum = Arrays.stream(number).sum();
```

### 파일로 스트림 만들기

…

### 함수로 무한 스트림 만들기

`Stream.iterate` 와 `Stream.generate` 로 무한 스트림을 만들 수 있도록 제공한다. 두 방법으로 만드는 스트림은 요청할 때마다 주어진 함수를 이용해서 값을 만든다. 무제한으로 값을 계산할 수 있는데, 보통 limit과 함께 사용한다.

```java
Stream.iterate(0, n -> n + 2)
                .limit(10)
                .forEach(System.out::println);
```

```java
Stream.generate(Math::random)
                .limit(5)
                .forEach(System.out::println);
```

`iterate`  메서드는 초기값과 람다를 인수로 받아서 무한 스트림을 생성한다. 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만든다. 이런 스트림을 `unbound stream` 이라고 한다.

일반적으로 연속된 일련의 값을 만들 때는 `iterate` 를 사용한다. 그리고 자바9에서는 `iterate`  메서드가 프레디케이트를 지원하여 특정 조건을 만족할 때 생성을 중단하게 만들 수 있다.

```java
IntStream.iterate(0, n -> n < 100, n -> n+4)
				.forEach(System.out::println);
```

두 번째 인자로 프레디케이트를 받아 언제까지 작업을 수행할 지 기준으로 사용한다. `filter` 를 사용해도 동일한 결과를 얻을 수 없는데, filter 메서드가 언제 이 작업을 중단해야 할지 모르기 때문이고 쇼트서킷이 일어나지 않기 때문이다.

`generate` 메서드는 생산된 값을 연속적으로 계산하지 않고 `Supplier<T>` 를 인수로 받아 새로운 값을 생산한다. limit이 없다면 스트림은 언바운드 상태가 된다.

병렬코드에서 발행자에 상태가 있는 경우 안전하지 않다. 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태를 유지해야 한다.

limit을 사용해서 명시적으로 스트림의 크기를 제한하지 않으면 forEach나 reduce등 작업을 수행할 수 없으므로 주의하자.