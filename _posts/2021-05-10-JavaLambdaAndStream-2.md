---
title: "Java Lambda & Stream(2)"
layout: post
author: jsh
cover: "/assets/cover.jpg"
categories: java
---

Java Stream & Lambda (2)
---


#### 동작 순서

다음 스트림에서는 최종 작업인 findFirst 메소드를 호출합니다.    
과연 출력 결과는 어떨까요?

```java
list.stream()
  .filter(el -> {
    System.out.println("filter() was called.");
    return el.contains("a");
  })
  .map(el -> {
    System.out.println("map() was called.");
    return el.toUpperCase();
  })
  .findFirst();
```

요소는 3개인데 결과는 다음처럼 filter 두 번, map 이 한 번 출력됩니다.

```java
filter() was called.
filter() was called.
map() was called.
```

여기서 스트림이 동작하는 순서를 알아낼 수 있습니다. 모든 요소가 첫 번째 중간 연산을 수행하고 남은 결과가 다음 연산으로 넘어가는 것이 아니라, 한 요소가 모든 파이프라인을 거쳐서 결과를 만들어내고, 다음 요소로 넘어가는 순입니다.

좀 더 자세히 살펴보면,

+ 처음 요소인 “Eric” 은 “a” 문자열을 가지고 있지 않기 때문에 다음 요소로 넘어갑니다. 이 때 “filter() was called.” 가 한 번 출력됩니다.
+ 다음 요소인 “Elena” 에서 "filter() was called."가 한 번 더 출력됩니다. "Elena"는 "a"를 가지고 있기 때문에 다음 연산으로 넘어갈 수 있습니다.
+ 다음 연산인 map 에서 toUpperCase 메소드가 호출됩니다. 이 때 "map() was called"가 출력됩니다.
+ 마지막 연산인 findFirst 는 첫 번째 요소만을 반환하는 연산입니다. 따라서 최종 결과는 “ELENA” 이고 다음 연산은 수행할 필요가 없어 종료됩니다.


위와 같은 과정을 통해서 수행됩니다.

#### 성능 향상

위에서 살펴봤듯이 스트림은 한 요소씩 수직적으로(vertically) 실행됩니다.    
여기에 스트림의 성능을 개선할 수 있는 힌트가 숨어있습니다.    
다음 예제를 살펴보시죠.

```java
list.stream()
  .map(el -> {
    wasCalled();
    return el.substring(0, 3);
  })
  .skip(2)
  .collect(Collectors.toList());

System.out.println(counter); // 3
```

첫 번째 요소 "Eric"은 먼저 문자열을 잘라내고, 다음 skip 메소드 때문에 스킵됩니다.   
다음 요소인 "Elena"도 마찬가지로 문자열을 잘라낸 후 스킵됩니다.   
마지막 요소인 “Java” 만 문자열을 잘라내어 “Jav” 가 된 후 스킵되지 않고 결과에 포함됩니다.   
여기서 map 메소드는 총 3번 호출됩니다.

여기서 메소드 순서를 바꾸면 어떨까요? skip 메소드가 먼저 실행되도록 해봅시다.

```java
List<String> collect = list.stream()
  .skip(2)
  .map(el -> {
    wasCalled();
    return el.substring(0, 3);
  })
  .collect(Collectors.toList());

System.out.println(counter); // 1
```

그 결과 스킵을 먼저 하기 때문에 map 메소드는 한 번 밖에 호출되지 않습니다.   
이렇게 요소의 범위를 줄이는 작업을 먼저 실행하는 것이 불필요한 연산을 막을 수 있어 성능을 향상시킬 수 있습니다.   
이런 메소드로는 skip, filter, distinct 등이 있습니다.


#### 스트림 재사용

종료 작업을 하지 않는 한 하나의 인스턴스로서 계속해서 사용이 가능합니다.   
하지만 종료 작업을 하는 순간 스트림이 닫히기 때문에 재사용은 할 수 없습니다.   
스트림은 저장된 데이터를 꺼내서 처리하는 용도이지 데이터를 저장하려는 목적으로 설계되지 않았기 때문입니다.

```java
Stream<String> stream = 
  Stream.of("Eric", "Elena", "Java")
  .filter(name -> name.contains("a"));

Optional<String> firstElement = stream.findFirst();
// IllegalStateException: stream has already been operated upon or closed
Optional<String> anyElement = stream.findAny(); 
```

위 예제에서 findFirst 메소드를 실행하면서 스트림이 닫히기 때문에 findAny 하는 순간 런타임 예외(runtime exception)이 발생합니다.   
컴파일러가 캐치할 수 없기 때문에 Stream 이 닫힌 후에 사용되지 않는지 주의해야 합니다.

위 코드는 아래 코드처럼 바꿀 수 있습니다.   
데이터를 List 에 저장하고 필요할 때마다 스트림을 생성해 사용합니다.

```java
List<String> names = 
  Stream.of("Eric", "Elena", "Java")
  .filter(name -> name.contains("a"))
  .collect(Collectors.toList());

Optional<String> firstElement = names.stream().findFirst();
Optional<String> anyElement = names.stream().findAny();
```

#### 지연 처리 Lazy Invocation

스트림에서 최종 결과는 최종 작업이 이루어질 때 계산됩니다. 호출 횟수를 카운트하는 예제입니다.

```java
private long counter;
private void wasCalled() {
  counter++;
}
```

다음 예제에서 리스트의 요소가 3개이기 때문에 총 세 번 호출되어 결과가 3이 출력될 것으로 예상됩니다. 하지만 출력값은 0입니다.

```java
List<String> list = Arrays.asList("Eric", "Elena", "Java");
counter = 0;
Stream<String> stream = list.stream()
  .filter(el -> {
    wasCalled();
    return el.contains("a");
  });
System.out.println(counter); // 0 ??
```

왜냐하면 최종 작업이 실행되지 않아서 실제로 스트림의 연산이 실행되지 않았기 때문입니다.   
다음 예제처럼 최종 작업인 collect 메소드를 호출한 결과 3이 출력됩니다.

```java
list.stream().filter(el -> {
  wasCalled();
  return el.contains("a");
}).collect(Collectors.toList());
System.out.println(counter); // 3
```

#### Null-safe 스트림 생성하기

NullPointerException 은 개발 시 흔히 발생하는 예외입니다.   
Optional 을 이용해서 null에 안전한(Null-safe) 스트림을 생성해보겠습니다.

```java
public <T> Stream<T> collectionToStream(Collection<T> collection) {
    return Optional
      .ofNullable(collection)
      .map(Collection::stream)
      .orElseGet(Stream::empty);
  }
```


위 코드는 인자로 받은 컬렉션 객체를 이용해 옵셔널 객체를 만들고 스트림을 생성후 리턴하는 메소드입니다.   
그리고 만약 컬렉션이 비어있는 경우라면 빈 스트림을 리턴하도록 합니다.


제네릭을 이용해 어떤 타입이든 받을 수 있습니다.

```java
List<Integer> intList = Arrays.asList(1, 2, 3);
List<String> strList = Arrays.asList("a", "b", "c");

Stream<Integer> intStream = 
  collectionToStream(intList); // [1, 2, 3]
Stream<String> strStream = 
  collectionToStream(strList); // [a, b, c]
```

이제 null 로 테스트를 해보겠습니다. 다음과 같이 리스트에 null 이 있다면 NPE(NullPointerException) 가 날 수 밖에 없는 상황입니다.   
외부에서 인자로 받은 리스트로 작업을 하는 경우에 일어날 수 있는 상황입니다.

```java
List<String> nullList = null;

nullList.stream()
  .filter(str -> str.contains("a"))
  .map(String::length)
  .forEach(System.out::println); // NPE!
```

하지만 우리가 만든 메소드를 이용하면 NPE 가 발생하는 대신 빈 스트림으로 작업을 마칠 수 있습니다.

```java
collectionToStream(nullList)
  .filter(str -> str.contains("a"))
  .map(String::length)
  .forEach(System.out::println); // []
```

#### 줄여쓰기 Simplified

스트림 사용 시 다음과 같은 경우에 같은 내용을 좀 더 간결하게 줄여쓸 수 있습니다.   
IntelliJ 를 사용하면 다음과 같은 경우에 줄여쓸 것을 제안해줍니다.   
그 중에서 많이 사용되는 것만 추렸습니다.

```java
collection.stream().forEach() 
  → collection.forEach()
  
collection.stream().toArray() 
  → collection.toArray()

Arrays.asList().stream() 
  → Arrays.stream() or Stream.of()

Collections.emptyList().stream() 
  → Stream.empty()

stream.filter().findFirst().isPresent() 
  → stream.anyMatch()

stream.collect(counting()) 
  → stream.count()

stream.collect(maxBy()) 
  → stream.max()

stream.collect(mapping()) 
  → stream.map().collect()

stream.collect(reducing()) 
  → stream.reduce()

stream.collect(summingInt()) 
  → stream.mapToInt().sum()

stream.map(x -> {...; return x;}) 
  → stream.peek(x -> ...)

!stream.anyMatch() 
  → stream.noneMatch()

!stream.anyMatch(x -> !(...)) 
  → stream.allMatch()

stream.map().anyMatch(Boolean::booleanValue) 
  → stream.anyMatch()

IntStream.range(expr1, expr2).mapToObj(x -> array[x]) 
  → Arrays.stream(array, expr1, expr2)

Collection.nCopies(count, ...) 
  → Stream.generate().limit(count)

stream.sorted(comparator).findFirst() 
  → Stream.min(comparator)
```

하지만 주의점이 있습니다. 특정 케이스에서 조금 다르게 동작할 수 있습니다.

예를 들면 다음의 경우 stream 을 생략할 수 있지만,

```java
collection.stream().forEach() 
  → collection.forEach()
```

다음 경우에서는 동기화(synchronized)는 차이가 있습니다.

```java
// not synchronized
Collections.synchronizedList(...).stream().forEach()
  
// synchronized
Collections.synchronizedList(...).forEach()
```

다른 예제는 다음과 같이 collect 를 생략하고 바로 max 메소드를 호출하는 경우입니다.

```java
stream.collect(maxBy()) 
  → stream.max()
```

하지만 스트림이 비어서 값을 계산할 수 없을 때의 동작은 다릅니다.    
전자는 Optional 객체를 리턴하지만, 후자는 NullPointerExcpetion 이 발생할 가능성이 있습니다.


```java
collect(Collectors.maxBy()) // Optional
Stream.max() // NPE 발생 가능
```
