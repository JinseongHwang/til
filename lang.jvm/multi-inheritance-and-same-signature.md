## 다중 상속 + 동일한 시그니쳐 일 떄 어떻게 동작할까?

```kotlin
interface Foo {
    fun sayHello() {
        println("Hi Foo")
    }
}

interface Bar {
    fun sayHello() {
        println("Hi Bar")
    }
}

class Foobar : Foo, Bar {
    override fun sayHello() {
        println("Hi FooBar")
//        super.sayHello() ==> 컴파일 에러!
    }
}

fun main() {
    val foobar = Foobar()
    foobar.sayHello()
}
```

예시 코드는 Kotlin이지만 Java도 동일.

## 배경

- 상속 관계에서, 부모 클래스의 메서드와 동일한 시그니쳐 메서드를 자식 클래스에 만드는 것을 보고 '메서드 오버라이딩'이라고 한다.
- Java에서는 클래스 다중 상속을 허용하지 않는다. 즉, 하나의 클래스만 상속받을 수 있다.
- 다만 인터페이스는 다중 상속을 받을 수 있다.

## 궁금증

- 인터페이스에는 'default method'를 구현할 수 있다.
- 여러 인터페이스를 상속받을 때, 동일한 시그니쳐 메서드가 각 인터페이스에 구현되어 있다면 자식 클래스에서 호출할 때 어떤 인터페이스로 결정되어 메서드를 호출할까?

## 결과

- 동일한 시그니쳐일 경우 반드시 오버라이딩을 해서, 자식 클래스만의 구현을 가져야 한다.
- 구현하지 않고, 상위 참조 메서드를 호출할 경우 컴파일 에러가 발생한다.
