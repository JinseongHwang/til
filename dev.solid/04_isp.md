## ISP (Interface Segregation Principle; 인터페이스 분리 원칙)

> 하나의 클래스에서 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다.

- 원칙은 위와 같다. 하지만 이를 조금 더 와닿게 풀어보자면,
  1. 어떤 클래스가 다른 클래스에 종속할 때는 최소한의 인터페이스만 가져야 한다.
  2. 인터페이스에 포함된 불필요하게 많은 기능들을 구현하지 않는 것이 좋다.
  3. 범용 인터페이스로 하나 만드는 것보단 클라이언트 관점에서 필요한 여러 개의 인터페이스로 분리하여 만드는 것이 더 좋다.
- 각 클라이언트가 자신에게 필요한 메서드만 갖는 인터페이스에 의존하도록 해야 한다.

## SRP vs ISP

- SRP는 클래스의 단일 책임을 강조하는 원칙
- ISP는 인터페이스의 단일 책임을 강조하는 원칙
- 둘 다 목표는 동일하다. 목표는 "변화에 유연하게 적응하는 소프트웨어를 만드는 것".
  - SRP는 클래스 분리를 통해 이 목표를 이루고자 하는 것이고, ISP는 인터페이스 분리를 통해 목표를 이루고자 하는 것이다.


## 예시 장점

- 자동차라는 범용 인터페이스가 있다면, 이를 운전 인터페이스, 정비 인터페이스로 나누는 것이 좋다.
- 한 쪽에 문제가 생겼을 때, 문제가 생긴 쪽에만 수정을 하면 되고, 문제가 생기지 않은 쪽은 건드리지 않으면 된다.
- 인터페이스의 의미가 더욱 명확해진다는 장점이 있다.

## 주의 사항

- 클라이언트에서 의존하고 있는 인터페이스는 변경하기가 굉장히 까다롭다.
- 따라서 인터페이스를 만들고 분리할 때 신중하게 결정해야 한다.


## 원칙을 위반한 코드

```java
interface Worker {
    void work();
    void eat();
}

class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Human working");
    }

    @Override
    public void eat() {
        System.out.println("Human eating");
    }
}

class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working");
    }

    @Override
    public void eat() {
        // 로봇은 먹지 않지만, 메서드를 구현해야 함
        throw new UnsupportedOperationException("Robots don't eat");
    }
}

public class Main {
    public static void main(String[] args) {
        Worker human = new HumanWorker();
        human.work();
        human.eat();

        Worker robot = new RobotWorker();
        robot.work();
        // robot.eat(); // 이 라인을 실행하면 예외가 발생함
    }
}
```

## 개선된 코드

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class HumanWorker implements Workable, Eatable {
    @Override
    public void work() {
        System.out.println("Human working");
    }

    @Override
    public void eat() {
        System.out.println("Human eating");
    }
}

class RobotWorker implements Workable {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
}

public class Main {
    public static void main(String[] args) {
        Workable humanWorker = new HumanWorker();
        humanWorker.work();
        ((Eatable) humanWorker).eat();

        Workable robotWorker = new RobotWorker();
        robotWorker.work();
    }
}
```
