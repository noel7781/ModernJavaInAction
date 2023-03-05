# Chapter 05 - 스트림 활용 (1)

스트림을 활용하면 데이터를 어떻게 처리할지는 스트림 API가 관리하므로 편리하게 데이터 관련 작업을 할 수 있다. 따라서 스트림 API 내부적으로 다양한 최적화가 이뤄질 수 있다. 스트림 API는 내부 반복 뿐 아니라 코드를 병렬로 실행할지 여부도 결정할 수 있다. 이러한 일은 순차적인 반복을 단일 스레드로 구현하는 외부 반복으로는 달성할 수 없다.

이 장에서는 스트림 API가 지원하는 다양한 연산을 사렾본다.

## 필터링

### 프레디케이트로 필터링

스트림 인터페이스는 filter 메서드를 지원한다. filter메서드는 Predicate를 인수로 받아서 Predicate와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

### 고유 요소로 필터링

스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다. (Unique 여부는 스트림에서 만든 객체의 hashCode, equals로 결정한다.)

## 스트림 슬라이싱

탐색과정에서 스트림의 요소를 선택하거나 스킵하는 방법도 존재한다.

### 프레디케이트를 이용한 슬라이싱

자바9부터 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두가지 메서드를 지원한다.

- takeWhile → 주어진 리스트가 특정 값을 기준으로 정렬되어 있는 상태일 때, filter를 통해 원하는 데이터를 찾고자 한다면 리스트 전부를 순회해야한다는 특징이 있다. 이런 경우에 takeWhile을 사용하여 조건을 넘어가는 경우 탐색을 멈춰 작업을 끝낼 수 있다.
- dropWhile → takeWhile과 반대로, 정렬되어 있는 경우 일정 조건을 만족하는것을 제외한 모든 요소를 얻는 경우에 사용할 수 있다.

### 스트림 축소

스트림에서 원하는 크기만큼 새로운 스트림을 가지고 싶을 때 limit(n)을 통해 최대 n개 반환할 수 있다.

### 요소 건너뛰기

스트림에서 skip(n)을 통해 처음 n개의 요소를 제외한 스트림을 반환할 수 있다. 전체가 n개 이하라면 빈 스트림이 반환된다.

## 매핑

특정 객체에서 특정 데이터를 선택하는 작업에 사용되는 연산이다. map과 flatMap을 사용한다.

### 스트림의 각 요소에 함수 적용하기

스트림은 함수를 인수로 받는 map 메서드를 지원한다. 

flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

```java
public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent()) {
        return empty();
    } else {
        return Optional.ofNullable(mapper.apply(value));
    }
}

public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent()) {
        return empty();
    } else {
        @SuppressWarnings("unchecked")
        Optional<U> r = (Optional<U>) mapper.apply(value);
        return Objects.requireNonNull(r);
    }
}
```

### 스트림 평면화

리스트의 문자들에서 고유 문자로 이루어진 리스트를 반환하는 문제를 풀어보자.

```java
List<String> words = Arrays.asList("Hello", "World");
System.out.println(
        words.stream() // Stream<String>
                .map(word -> word.split("")) // Stream<String[]>
                .distinct() // Stream<String[]>
                .collect(Collectors.toList()) // List<String[]>
);
-> [H, e, l, o], [W, o, r, l, d]
```

map을 사용해서 구하고자 하는 경우 원하는 값이 나오지 않는다.

한 단계씩 보면

1. `words` 는 `List<String>` 타입이다.
2. `words.stream()` 은 `Stream<String>` 타입이 된다.
3. `String.split()` 의 리턴타입은 `String[]` 이다.
    
    따라서 `words.stream().map(word -> word.split("")` 는 `Stream<String[]>` 타입을 가진다.
    

```java
public String[] split(String regex, int limit) {}
```

1. `distinct()` 는 타입을 바꾸지 않는다.
2. `collect(Collectors.toList())` 를 통해 `List<String[]>` 타입으로 바뀐다.

결국 `distinct` 가 적용될 때 String[]을 비교해서 중복이 없는지 판단하게 되고, 원하는 값을 얻지 못한다.

또 다른 방식으로 중간에 String에서 String[]이 아니라 개별요소로 만들면 이 문제를 해결할 수 있다.

```java
System.out.println(
        words.stream()
                .map(word -> word.split(""))
                .flatMap(Arrays::stream)
                .distinct()
                .collect(Collectors.toList())
);
```

위와의 차이점은 `flatMap(Arrays::stream)` 인데, 이 과정을 통해 `Stream<String[]>` 에서 `Stream<String>` 으로 변환이 이뤄진다.

## 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용되는데, allMatch, anyMatch, noneMatch, findFirst, findAny 등 유틸리티 메서드를 제공한다.

### 요소 검색

```java
Optional<T> findAny();
```

findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다. 스트림 파이프라인은 쇼트서킷을 이용해 결과를 찾는 즉시 실행을 종료한다. 그리고 해당 요소가 없을수도 있으므로 `Optional` 을 리턴한다.

### 첫 번째 요소 찾기

```java
public Optional<S> findFirst() {
    Iterator<S> iterator = iterator();
    if (iterator.hasNext()) {
        return Optional.of(iterator.next());
    } else {
        return Optional.empty();
    }
}
```

findFirst와 findAny가 모두 필요한 이유는 병렬 환경에서는, 첫 번째 요소를 찾기 어렵다. 따라서 요소의 반환 순서가 상관 없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

```java
List<Integer> nums = Arrays.asList(1,2,3,4,5,6,7,8,9);
System.out.println(
        nums.parallelStream()
                .filter(n -> n % 3 == 0)
                .findAny() or .findFirst()
                .get()
);
-> 
findAny() 실행 시 3, 6 다양한 결과
findFirst() 실행 시 3 출력
```

## 리듀싱

리듀싱 연산이란 모든 스트림 값을 처리해서 값으로 도출하는 연산이다.

```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
Optional<Integer> sum = numbers.stream().reduce((a, b) -> a + b);
```

초깃값이 없으면 reduce는 Optional 객체를 반환한다. 스트림이 비어있는 경우 합계를 구할 수 없을수 있기 때문이다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max); // 최댓값 구하기
```

## 스트림 연산 : 상태 없음과 상태 있음

스트림 연산은 상태 없는 연산과 상태 있는 연산이 있다.

map, filter는 입력 스트림에서 각 요소를 받아 결과를 출력 스트림으로 보낸다. 이들은 보통 상태가 없는, stateless operation이다.

하지만 reduce, sum, max같은 연산은 결과를 누적할 내부 상태가 필요하다. 이 경우 스트림에서 처리하는 요소의 수와 관계 없이 내부 상태의 크기는 한정(bound) 되어 있다.

반면 sorted나 distinct같은 연산은 중복을 제거하거나 정렬을 하기 위해 과거의 이력을 알고 있어야 한다. 예를 들어 어떤 요소를 출력 스트림으로 추가하려면 모든 요소에 버퍼가 추가되어 있어야 한다. 이러한 연산을 내부상태를 갖는 연산, stateful operation이라고 한다.