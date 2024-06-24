## LSP (Liskov Substitution Principle; 리스코프 치환 원칙)

> 프로그램의 객체는 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.

- 여기서는 하위 타입을 “인터페이스의 구현체” 정도로 이해하면 좋을 듯 하다.
- 인터페이스 구현체를 만들 때 인터페이스 규약을 모두 지키면서 개발해야 한다.
- 이 규칙은 컴파일 성공을 넘어선 이야기다.

## LSP의 핵심 개념

- 자식 클래스는 부모 클래스의 계약(제공되는 기능)을 위반해서는 안된다. 부모 클래스에서 기대되는 모든 행동을 자식 클래스에서도 유지해야 한다.
- 자식 클래스가 부모 클래스를 대체하더라도 프로그램의 동작이 변하지 않아야 한다.
- 자식 클래스는 부모 클래스의 인터페이스를 준수해야 한다.

## 상속

- 코드 재사용을 목적으로 하는 상속은 사용하지 말아야 한다. IS-A 관계가 있을 때만 상속을 사용하도록 제한해야 한다.
- 코드 재사용을 목적으로 할 때는 합성(composition)을 사용하는 것이 좋다.

## 정리

- 상위 타입의 퍼블릭 인터페이스(예외까지)를 잘 지키는 하위 타입은, 상위 타입을 상속받는 어떠한 타입으로도 치환 가능해야 한다.
- 상속은 다형성과 함께 생각해야 한다. 다형성으로 인한 확장 효과를 얻기 위해서는 OCP를 제공해야 한다. 즉, LSP는 OCP를 구성하는 요소 중 하나이다.
- LSP는 규약을 준수하는 상속 구조를 제공한다. 

### 예시 상황

LSP를 준수하면 상속 계층 구조에서 부모 클래스를 자식 클래스로 안전하게 대체할 수 있으며, 이는 코드의 재사용성과 유지보수성을 높인다. 
인터페이스를 통해 특정 기능을 분리하여, 자식 클래스가 부모 클래스의 계약을 준수하고 일관된 행동을 보장하도록 설계하는 것이 중요하다.

## 원칙을 위반한 코드

```java
class Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly");
    }
}

public class Main {
    public static void main(String[] args) {
        Bird bird = new Bird();
        bird.fly();  // "Flying"

        Bird penguin = new Penguin();
        penguin.fly();  // "UnsupportedOperationException"
    }
}
```
- Penguin은 Bird를 상속받기 때문에 반드시 fly 기능을 퍼블릭하게 공개해야 한다.
- 하지만 Penguin은 날 수 없는 새이기 때문에 fly 실행 시 예외가 발생한다.
- 따라서 Bird 대신에 Penguin이 들어가면 정상적으로 동작하지 않기 때문에 치환할 수 없다. 따라서 LSP를 위반한다.
- 컴파일 에러가 발생하지 않기 때문에 문제가 없어 보이지만, LSP 관점에서는 원칙을 위반하는 것이 맞다. 

## 개선된 코드

```java
interface Flyable { 
    void fly();
}

abstract class Bird {
    public abstract void makeSound();
}

class Eagle extends Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Flying");
    }

    @Override
    public void makeSound() {
        System.out.println("Eagle sound");
    }
}

class Penguin extends Bird {
    @Override
    public void makeSound() {
        System.out.println("Penguin sound");
    }
}

public class Main {
    public static void main(String[] args) {
        Flyable eagle = new Eagle();
        eagle.fly();  // "Flying"

        Bird penguin = new Penguin();
        penguin.makeSound();  // "Penguin sound"
    }
}
```
- fly 메서드를 가지는 Flyable 인터페이스를 추가한다.
- Flyable을 구현하는 Eagle 클래스가 있고, Penguin은 더 이상 Flyable을 구현하지 않는다.
