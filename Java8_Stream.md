# Stream

* java8에서 추가
* ~~File I/O는 아니다..(전 처음들었을때 File I/O인줄..)~~
* java8 이전에는 배열이나 collection 을 다루는 for or foreach를 사용하여 명시적으로 사용
* ~~간단한 loop는 상관없지만 그렇지 않은경우 소스의 복잡도가 높아지는 문제가 발생~~

## Stream = 데이터의 흐름
* 배열 또는 컬렉션 인스턴스에 함수 여러개를 조합해서 필터링하고 가공된 결과를 얻을 수 있다.
* 또한 람다를 이용해서 코드도 간결하게 표현가능.
* 데이터를 잘게 나눠서 병렬작업도 가능.


```java
int[] arrayInt = {1,2,3,4,5,6,7,8,9,10};

//foreach
for (int i : arrayInt) {
  if (i%2 == 0) log.info("test {}", i);
}

//Stream
Arrays.stream(arrayInt).filter(i->i%2==0).forEach(i->log.info("test {}", i));

```

## Collections Vs. Stream
지금까지 사용했던 Collection과 java8부터 등장한 Stream 데이터들을 순차적으로 처리하는것과 관련된 인터페이스를 제공합니다.
하지만 Collection은 데이터와 관련되어있고 Stream을 데이터를 연산하는데 초점을 두고있습니다.

## Stream 특징
* Stream은 데이터를 변경하지 않는다.

단순히 데이터를 읽어서 계산에 집중, 데이터가 변경된다고(혹은 정렬) 원본 데이터는 변하지 않는다. 필요하는 경우 데이터를 변환해서 담을 수 있다.

```java
int[] array = Arrays.stream(arrayInt).sorted(); //변경없음
int[] array = Arrays.stream(arrayInt).sorted().toArray(); //정렬된 데이터 반환
```

* Stream은 1회용이다.

한번 사용하고 나면 Stream은 닫힌다. 다시 사용하기 위해서 재생성이 필요하다.

```java
    IntStream intStream = Arrays.stream(array); //stream 생성
    intStream.forEach(i->log.info("intStream1 {}", i)); //stream 사용가능
    intStream.forEach(i->log.info("intStream2{}", i)); //stream 사용불가.
```

2번째 사용시 아래와 같은 Exception이 발생.

`java.lang.IllegalStateException: stream has already been operated upon or closed`

* Stream은 작업은 내부반복으로 처리한다.

내부반복이란 반복문의 메소드를 메소드 내부에 숨겨져있다는것을 의미한다.

## Stream 구조


```java
Arrays.stream(array) //1.Stream 생성
.filter().map() //2. 중간(중개)연산
.count(); //3. 종단연산
```

* 중간(중개)연산(intermediate operations)

연산결과를 Stream으로 반환하기때문에 연속적으로 사용이 가능하다.(chain method)

ex) map(), flatMap() 등..

* 종단연산(terminal operations)

종단연산을 작업을 종료 할수있다.
처리결과를 void list boolean등으로 받을 수 있다.
제공받은 데이터를 변경하는 것이 아니라 연산으로 생성된 데이터를 반환한다.

ex) count(), toArray(), collect() 등...

### 지연처리
스트림은 최종연산을 하지않으면 중간동작은 작동하지 않는다.
생성-중간연산-중간연산-중간연산-...을 아무리 많이해도 마지막으로 최종연산을 하지 않으면 중간연산은 동작하지않는다.
생성-중간연산-최종연산 정상동작을 한다.

```java
int[] arrayInt = {1, 3, 2, 4, 6, 5, 7, 8, 9, 10};

//log가 출력되지 않는다.
IntStream intStream1 = Arrays.stream(arrayInt);
intStream1.filter(i->{
		log.info("filter1 : {}" , i );
		return true;
	});

//log가 출력된다.
IntStream intStream2 = Arrays.stream(arrayInt);
intStream2.filter(i->{
	log.info("filter2 : {}" , i );
      return true;
    }).forEach(i->log.info("foreach : {}", i));
```

## Stream 생성

### 배열 Stream
배열은 `Arrays.stream`을 사용해서 생선한다.

```java
String[] arr = new String[]{"a", "b", "c"}; //[a,b,c]
Stream<String> stream = Arrays.stream(arr); //[a,b,c]
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // 1~2 요소 [b, c]
```

### 컬렉션 Stream
컬렉션 타입인(Collection, List, Set) 경우 interface에 추가된 디폴트메소드 `stream` 을 사용해 생성 할 수 있습니다.
> java8 부터 interface class에서 default method라는 기능이 추가 되면서 interface에서 메소드를 정의 할 수 있게 되었습니다.

```java
public interface Collection<E> extends Iterable<E> {
  default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
  }
}
```

다음과 같이 생성 할 수 있습니다.
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

### Stream.builder
`builder` 메소드를 이용하면 직접원하는 값을 집어 넣을수도 있습니다.
```java
Stream<String> builderStream = Stream.<String>builder()
    .add("viewing").add("tving").add("pooq")
    .build();
```

### Stream.generate()
`generate` 메소드를 이용하면 `Supplier<T>`에 해당하는  람다로 값을 넣을 수 있습니다.
> `Supplier<T>`는 인자는 받지 않으며 리턴타입만 존재하는 메소드를 가지고 있다.
> 즉, 항상 같은 값을 리턴하는 메소드이다.

```java
public static<T> Stream<T> generate(Supplier<T> s) { ... }
```

생성되는 스트림 크기는 정해져 있지 않기 때문에 `limit`을 지정해줘야한다.
```java
Stream<String> generatedStream = Stream.generate(() -> "gen").limit(5);
```

### 기본타입형 스트림
제네릭을 사용하면 리스트나 배열을 이용해서 기본타입 스트림을 생성 할 수 있습니다.
하지만 제네릭을 사용하지 않고 직접적으로 해당타입의 스트림을 다룰수도 있습니다.
제네릭을 사용하지 않기 때문에 불필요한 오토박싱이 일어나지 않습니다.
```java
IntStream intStream = IntStream.range(1, 5); // [1, 2, 3, 4]
LongStream longStream = LongStream.rangeClosed(1, 5); // [1, 2, 3, 4, 5]
```

### 파일 스트림
자바 NIO의 `Files.line` 메소드는 해당파일의 각 라인을 스트링타입 스트림으로 만들어줍니다.
```java
Stream<String> lineStream = Files.lines(Paths.get("file.txt"), Charset.forName("UTF-8"));
```

## 중간연산(Intermediate operations)`
중간연산은 스트림을 통한 데이터의 가공 및 변환을 담당합니다.
아래에 소개되는 메소드 외에 다른 메소드도 존재합니다. (자세한 내용은 [JAVA DOCS](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html) 확인요..)

### `<R> Stream<R> map(Function< ? super T,? extends R> mapper)`
스트림 요소의 저장된값 중에서 원하는 필드만 뽑아내거나 특정형태를 변환할때 사용합니다.
T 타입을 변수를 R 타입으로 변환해서 반환하는 함수를 지정해야합니다.
쉽게 말해서 스트림에 들어가있는 T가 input이 되어서 특정 로직을 거친후 R타입의 스트림이 output이 되어서 리턴이 됩니다.

> `Function<T, R>` 하나의 인자와 리턴타입을 가지는 함수.

아래 예제는 스트림을 생성한 후 소문자로 이루어진 매개변수를 대문자로 변환후 반환하는 함수입니다.
> Double colon operation `::` java8에서 추가된 메소드 참조 연산자입니다.
> 메소드 참조를 사용할 때 타겟의 참조(객체, 아래에서는 String)는 구분 기호 앞, 메소드 이름은 콜론 뒤에 오게 됩니다.

```java
List<String> nameList = Arrays.asList("viewing, tving, pooq");
Stream<String> stringStream = nameList.stream().map(String::toUpperCase);
```

### `Stream<R> flatMap(Function<A, R>)`
스트림요소가 배열이거나 `map` 의 연산결과가 배열인경우(Stream[]) 단일 스트림으로 변환해서 사용하면 편리합니다.
`flatmap`을 사용하면 단일스트림으로 변환해줍니다.

> 스트림의 값들이 mapper 함수를 실행 될때 마다 실행결과를 또다른 스트림으로 리턴합니다.
> 여기서 `map`은 스트림에 있는 값들을 각각 다른값으로 매핑한 후 스트림으로 리턴합니다.
> `flatmap`같은 경우 하나의 스트림으로 만들어서 리턴합니다.

```java
Stream<String[]> stream = Stream.of(
                new String[]{"viewing, tving, pooq"}
                , new String[]{"stick, youtube, oksusu"}
        );
Stream<Stream<String>> mapStream = stream.map(Arrays::stream);
Stream<String> flatMap = stream.flatMap(Arrays::stream);

```

### `Stream<T> filter(Predicate<T>)`
`filter`는 스트림내 원소들을 하나씩 판단해서 구분하는 작업을 합니다.
> `Predicate<T>`는 하나의 인자와 리턴타입을 가지고, Function<T,R>과 비슷해보이지만 데이터를 가려내기위한 참과거짓으로 리턴타입은 boolean으로 고정되어 있습니다.

```java
IntStream intStream = IntStream.range(1,10); //[1,2,3,4,5,6,7,8,9]
intStream.filter(i->i%2 == 0).forEach(even -> log.info("even {}" , even));
```

### `Stream<T> peek(Consumer<T>)`
스트림내 데이터 전체를 돌면서 데이터를 확인을 합니다. 이와 비슷한 함수로는 `foreach`가 있습니다.
둘의 동작은 동일하지만 `peek`는 중간연산이고 `foreach`는 최종연산자입니다.
여기서 기억야할점은 위에서 설명했듯이 중간연산은 최종연산이 동작하지 않으며 동작하지 않는다는 겁니다.

> `Consumer<T>` 인자를 받지만 리턴 타입은 void이다.

```java
IntStream intStream = IntStream.range(1,10); //[1,2,3,4,5,6,7,8,9]
intStream.peek(System.out::println); //동작하지 않는다

IntStream intStream1 = IntStream.range(1,10); //[1,2,3,4,5,6,7,8,9]
intStream1.peek(System.out::println).sum(); //동작함
```

## 종단연산(Terminal operations)
### `R collect(Collector)`
`Collector`타입의 인자를 받아서 처리합니다. 자주 사용하는 작업은 `Collectors`객체로 제공하고 있습니다.

```java
//toList
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> integerList = integerStream.collect(Collectors.toList());

//toJoining
Stream<String> stringStream = Stream.of("1","2", "3", "4");
String joining = stringStream.collect(Collectors.joining());
```

### `void forEach(Consumer<T>)`
`foreach` 는 요소를 돌면서 실행되는 최종 작업입니다. 보통 `System.out.println` 메소드를 넘겨서 결과를 출력할 때 사용하곤 합니다.
```java
Stream<String> stringStream = Stream.of("1","2", "3", "4");
stringStream.forEach(System.out::println);
```

### reduce
`reduce` 함수는 최소1개 최대 3개의 인자를 받습니다.
accumulator : 각 요소가 올때마다 처리하는 로직.
identity : 계산을 위한 초기값으로 스트림이 없어도 해당값은 리턴
combiner : 병렬(parallel) 스트림에서 나눠 계산한 결과를 하나로 합치는 동작하는 로직

* `Optional<T> reduce(BinaryOperator<T>)`
* `T reduce(T identity, BinaryOperator<T> accumulator)`
* `<U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner)`

> `BinaryOperator<T>` 동일한 타입의 인자 2개와 인자와 같은 타입의 리턴타입을 가진다.
> `BiFunction<T, U, R>` 서로 다른 타입의 2개의 인자를 받아 또 다른 타입으로 반환한다.

```java
//첫번째 인자가 누적값이 되고 두번째 인자는 순서대로 들어오는 데이터이다.
//0+1+2+3
OptionalInt reduced = IntStream.range(1, 4).reduce(Integer::sum);

//10(초기값)+1+2+3
int reducedTwoParams = IntStream.range(1, 4).reduce(10, Integer::sum);

//단순히 병렬로 덧셈처리하는게 아니라, 계산방법이 달라져 결과값이 달라진다.
//loop를 통해서 순서로 처리되어지는 값은 각각 계산 되어진다.
//언뜻생각해보면 16이 나와야될것 같지만 (10(초기값)+1), (10(초기값)+2), (10(초기값)+3)
//이렇게 계산된값이 다시 더해진다.
//11+12+13
Integer reducedParallel = Arrays.asList(1, 2, 3)
//Combiner은 병렬처리에서만 동작한다
	.stream()
	.reduce(10, Integer::sum, (a, b) -> a + b); //결과값 16

Integer reducedParallel = Arrays.asList(1, 2, 3)
//Combiner은 병렬처리에서만 동작한다, parallelStream()선언하지않으면 동작하지 않는다
	.parallelStream()
	.reduce(10, Integer::sum, (a, b) -> a + b); //결과값 36
```

### Matching
매칭은 조건식을 받아서 해당 조건을 만족하는 요소가 있는지 체크한 결과(boolean)를 리턴합니다. 다음과 같은 세 가지 메소드가 있습니다.
* `boolean allMatch(Predicate<T>)` //모든조건을 만족하는지
* `boolean anyMath(Predicate<T>)`//하나라도 조건을 만족하는지
* `boolean noneMatch(Predicate<T>)` //모든조건을 만족하지 않는지

```java
List<String> names = Arrays.asList("viewing", "tving", "pooq");

boolean anyMatch = names.stream().anyMatch(name -> name.contains("p"));
boolean allMatch = names.stream().allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream().noneMatch(name -> name.endsWith("s"));
```

## 가공순서

```java
 List<String> names = Arrays.asList("viewing", "tving", "pooq");
 names.stream().map(s2 -> {
log.info("map {}", s2);
 return s2.toUpperCase();
 })
 .filter(s -> {
log.info("filter {}", s);
 return true;
 }).forEach(s1 ->  log.info("foreach {}" , s1));

 //output
 22:43:27.445 [main] INFO Test - map viewing
 22:43:27.447 [main] INFO Test - filter VIEWING
 22:43:27.447 [main] INFO Test - foreach VIEWING
 22:43:27.447 [main] INFO Test - map tving
 22:43:27.447 [main] INFO Test - filter TVING
 22:43:27.447 [main] INFO Test - foreach TVING
 22:43:27.447 [main] INFO Test - map pooq
 22:43:27.447 [main] INFO Test - filter POOQ
 22:43:27.447 [main] INFO Test - foreach POOQ
 ```

결과를 보면 예상과 다르게 map-filter-foreach가 마치 하나의 트랜잭션으로 호출 되는것을 알 수 있습니다. 즉 이말은 스트림에서 데이터를 하나씩 꺼내서 중간연산들을 수행한 후 최종연산을 호출한다는것을 짐작 할 수 있습니다.

 ```java
 List<String> names = Arrays.asList("viewing", "tving", "pooq");
 names.stream().filter(s -> {
log.info("filter {}", s);
 return s.startsWith("p");
 }).map(s2 -> {
log.info("map {}", s2);
 return s2.toUpperCase();
     }).forEach(s1 ->  log.info("foreach {}" , s1));

 //output
 22:47:29.671 [main] INFO Test - filter viewing
 22:47:29.673 [main] INFO Test - filter tving
 22:47:29.673 [main] INFO Test - filter pooq
 22:47:29.673 [main] INFO Test - map pooq
 22:47:29.673 [main] INFO Test - foreach POOQ
 ```
첫번째 예제를 조금 수정해서 `filter`에 조건을 추가하고 `map` 앞에두게 되면 `map`과`foreach`는 한번만 호출 되는것을 확인 할 수 있습니다. 
즉 이런 순서에 따라 중간연산 최종연산이 불필요한 수행되는 작업을 줄일수 있습니다. 이렇게 첫번째 `filter`에서 true가 아닌 데이터들은 호출하지 않는 기법을 `ShortCircuit`이고 스트림의 특징중 하나입니다.