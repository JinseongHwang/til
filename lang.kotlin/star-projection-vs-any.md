## `*` vs `Any`

## `*` (Star projection || Asterisk)

- 어떤 타입이 들어올진 몰라도 1가지 타입으로 결정된다.
- 한 번 구체적으로 타입이 결정되면 해당 타입만 받을 수 있다.

## `Any`

- Kotlin에서는 Primitive type, Reference Type 을 딱히 구분하지 않는다. 모두 객체이다. 즉, 모두 `Any`를 상속받는다.
- Kotlin의 `Any`는 Java의 `Object`와 달리 스레드를 제어하는 wait, notify, notifyAll 등 메서드가 없다. 필요하다면 `Object` 타입으로 강제 캐스팅이 필요하다.

## 정리

- `*` 은 결정되면 끝이다. `List<*>`가 `List<Int>`로 추론되었다면 그 이후로는 계속 `Int`타입만 담을 수 있다.
- `Any`는 다양한 타입을 동시에 다룰 수 있다. `List<Any>`에는 Int, Double, String 등 다양한 타입을 담을 수 있다. 

## 추가

### 공변성 (covariance)

```kotlin
val intList: List<Int> = listOf(1, 2, 3)
val anyList: List<out Any> = intList
```

- T가 Any의 하위 타입이라면, List<T>도 List<Any>의 하위 타입이다.
- 따라서 위 코드는 성립한다.

### out Any 활용 예시

```kotlin
fun printAll(objects: List<out Any>) {
    for (obj in objects) {
        println(obj)
    }
}

val intList: List<Int> = listOf(1, 2, 3)
val stringList: List<String> = listOf("A", "B", "C")

printAll(intList)    // Int 리스트를 전달
printAll(stringList) // String 리스트를 전달
```
- `Any`: 어떤 타입이든 담을 수 있는 컨테이너.
- `out Any`: 공변성을 나타내며, 특정 타입을 `Any`의 하위 타입으로 취급한다. 
  - 예를 들어, `List<out Any>`는 `List<Int>` 또는 `List<String>`과 같은 구체적인 타입의 리스트를 참조할 수 있습니다.