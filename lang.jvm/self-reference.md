## self 참조의 동작 비밀

```kotlin
open class Parent {

    fun sayHello() = println("Hi, I'm ${this.getName()}")

    open fun getName() = "Parent~"
}

class Child : Parent() {
    override fun getName() = "Child~"
}

fun main() {
    val child = Child()
    child.sayHello() // Hi, I'm Child~
}
```

예시 코드는 Kotlin이지만 Java도 동일.

## 실행 과정

1. 클라이언트(main 메서드)는 Child 객체에게 메시지를 던졌다. (= child.sayHello를 호출했다.)
2. 메시지를 수신한 객체인 child가 self 참조를 임시로 가진다. 
3. 단, child 객체는 sayHello 메서드를 구현하고 있지 않다. 
4. 따라서 상속 그래프를 탐색한다. Parent 객체가 sayHello 메서드를 구현하는 것을 발견했다!
5. 이는 child 객체가 Parent 객체에게 sayHello 실행을 위임한 것이다. 
6. Parent의 sayHello 메서드에서 this를 참조한다. this는 self를 가리키는 포인터이다. 따라서 self.getName()은 child의 getName이다.

## 결과

- "Hi, I'm Child~" 가 출력된다.
