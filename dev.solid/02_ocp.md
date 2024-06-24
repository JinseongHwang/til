## OCP (Open/Closed Principle; 개방/폐쇄 원칙)

> 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.

- 소프트웨어가 높은 응집도와 낮은 결합도를 가질 때 OCP를 잘 지키는 것이다.
  - 높은 응집도?
    - 응집도가 높다는 것은 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻이다.
    - 같은 책임, 관심사를 기반으로 하나의 객체를 설계하기 때문에 객체에 변경이 발생하더라도 다른 곳에 미치는 영향이 제한적이다.
  - 낮은 결합도?
    - 책임과 관심사가 다른 객체 다른 모듈과는 낮은 결합도를 유지해야 한다.
    - 결합도란 하나의 클래스에 변경이 일어날 때 관계를 맺고 있는 다른 클래스에게 변화를 요구하는 정도이다.
    - 즉, 낮은 결합도란 하나의 변경이 발생할 때 다른 모듈과 객체로 변경에 대한 요구가 전파되지 않는 상태라고 볼 수 있다.
- "확장에는 열려 있으나 변경에는 닫혀 있다"가 무슨 말인가?
  - 확장에 열려있다?
    - 모듈의 확장성을 보장하는 것을 의미한다.
    - 새로운 변경사항이 발생했을 때 유연하게 코드를 추가 또는 수정할 수 있어야 한다.
  - 변경에 닫혀있다?
    - 잘 만들어진 클래스 Foo가 있고, 클래스 Foo는 기능들을 잘 제공하고 있다.
    - 요구사항이 추가/변경 되더라도 잘 동작하는 Foo는 건드리지 않고 기능을 추가/변경할 수 있어야 한다.
    - OCP를 잘 지키면 Foo를 직접 수정하지 않고 기능 추가/변경이 가능하므로 변경에 닫혀 있다고 표현한다.
- OCP의 핵심은 "추상화"이다.
  - OCP를 구현하기 위해서는 DI와 IoC가 필요하다.

## OCP를 지키는 리팩토링 Tip

- 바뀌는 부분은 따로 뽑아서 캡슐화한다. 그러면 나중에 바뀌지 않는 부분에는 영향을 미치지 않고 그 부분만 고치거나 확장할 수 있다.

## 애플의 브랜드 원칙과 사용자 경험

- "Apple"이라는 브랜드 원칙이 OCP를 닮아있다.
- iPhone, iPad, Macbook, ... 등 제품을 가지고 있다.
- iPhone은 iOS라는 운영체제를 사용하며, iOS는 공개되지 않았기 때문에 소비자 입장에서 기능을 변경하는 것은 매우 힘들다. 즉, 변경에 닫혀있다.
- Macbook은 MacOS라는 운영체제를 사용하며, 마찬가지로 공개되지 않았기 때문에 변경에 닫혀있다.
- 하지만 iPhone과 MacOS는 Apple에서 만든 독자적인 인터페이스를 공유한다. 뛰어난 호환성을 가지는데, 마치 여러 Apple 기기가 하나의 기기처럼 동작할 수 있다. 확장에 열려있다.

##  “확장”과 “변경”에 집중하기

- 말이 상당히 어려운데, “확장”과 “변경”이라는 단어를 예시를 들어 쉽게 풀어보자.
- JpaRepository 인터페이스가 있고, `save()`, `findById()` 등의 메서드가 존재한다. 구현체로는 UserRepository, PostRepository 가 있다.
  - “확장”을 한다는 것은 인터페이스 명세를 지키는 구현체를 더 만드는 것을 의미한다. (ex. CommentRepository를 구현한다)
  - “변경”을 한다는 것은 기존 코드 자체가 변경되는 것을 의미한다. 변경은 발생하지 않아야 한다.
- 즉, OCP를 지키는 것은 인터페이스의 구현체를 더 만드는 것은 가능한데 기존에 잘 동작하던 코드는 변경할 필요가 없는 것을 의미한다.

## 원칙을 위반한 코드

```java
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.radius * circle.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return rectangle.width * rectangle.height;
        }
        return 0;
    }
}

class Circle {
    double radius;
    Circle(double radius) {
        this.radius = radius;
    }
}

class Rectangle {
    double width, height;
    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
}
```
- Circle과 Rectangle 2가지 도형이 있다.
- 각 도형별로 넓이를 계산하는 기능이 AreaCalculator에 있다.
- 구현체의 타입으로 분기해서 넓이를 계산하는 로직을 수행한다.

## 개선된 코드

```java
// 도형 인터페이스
interface Shape {
    double calculateArea();
}

// 원 클래스
class Circle implements Shape {
    double radius;
    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// 직사각형 클래스
class Rectangle implements Shape {
    double width, height;
    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

// 새로운 도형을 추가할 때는 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하면 됩니다.
class Triangle implements Shape {
    double base, height;
    Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// 면적 계산기 클래스
public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
}
```
- 도형을 나타내는 Shape이라는 인터페이스가 등장했다. 인터페이스는 넓이를 계산하는 calculateArea 라는 추상 메서드를 가진다.
- Circle과 Rectangle은 모두 Shape 인터페이스를 구현하고 있으며 calculateArea를 반드시 구현한다.
- 계산하기 위해서는 AreaCalculator에서 calculateArea 함수를 호출하기만 하면 된다.
- 여기서 Triangle이 추가(확장)되어도 기존의 다른 코드는 수정(변경)하지 않아도 잘 동작한다.
