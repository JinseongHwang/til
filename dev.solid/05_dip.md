## DIP (Dependency Inversion Principle; 의존관계 역전 원칙)

> 프로그래머는 구체화가 아니라 추상화에 의존해야 한다. 추상화는 구체화에 의존하면 안된다. 구체화는 추상화에 의존해야 한다.

- 즉, 구현 클래스(구현체)가 아니라 인터페이스(역할)에 의존하라는 의미이다.
- 코드 관점에서, 어떤 구체적인 구현체의 기능(function)을 사용하기 위해 구현체를 필드로 가지기 보다는 추상적인 것을 필드로 가지라는 의미이다.
그리고 추상적인 것에 구체적인 것을 담으면 된다.

## 클린 아키텍쳐에서는 DIP를 어떻게 표현하는가? (by 로버트 마틴)

> 고수준 정책을 구현하는 코드는 저수준 세부사항을 구현하는 코드에 절대로 의존해서는 안된다. 대신 세부사항이 정책에 의존해야 한다.

- '저수준 세부사항'이 구체 클래스이고, '정책'이 인터페이스이다. 
- 즉, 클라이언트는 인터페이스를 의존하고, 구체 클래스도 인터페이스를 의존해야 한다.
  - (클라이언트 --> 인터페이스 <-- 구체 클래스)

> 자바와 같은 정적 타입 언어에서 이 말은 use, import, include 구문은 오직 인터페이스나 추상 클래스 같은 추상적인 선언만 참조해야 한다는 뜻이다.
> 우리가 의존하지 않도록 피하고자 하는 것은 바로 변동성이 큰 구체적인 요소다.
 
- Java에서 import 해서 다른 클래스를 참조하곤 하는데, 이 때 import 하는 클래스는 구체 클래스면 안되고 인터페이스여야 한다고 주장한다.
- 굉장히 strict한 주장이다.

## 장점

- 모듈 간 결합도가 낮아진다. 즉, 유연성이 올라간다.

## 원칙을 위반한 코드

```java
class Light {
    public void turnOn() {
        System.out.println("Light is turned on");
    }

    public void turnOff() {
        System.out.println("Light is turned off");
    }
}

class Switch {
    private Light light;

    public Switch(Light light) {
        this.light = light;
    }

    public void operate(boolean on) {
        if (on) {
            light.turnOn();
        } else {
            light.turnOff();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Light light = new Light();
        Switch lightSwitch = new Switch(light);
        lightSwitch.operate(true);  // 기대 값: "Light is turned on"
        lightSwitch.operate(false); // 기대 값: "Light is turned off"
    }
}
```
- Switch는 Light(구체 클래스)에 직접적으로 의존한다.
- Light의 구현이 변경되면 Swtich에도 변경이 불가피하다.

## 개선된 코드

```java
interface Switchable {
    void turnOn();
    void turnOff();
}

class Light implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("Light is turned on");
    }

    @Override
    public void turnOff() {
        System.out.println("Light is turned off");
    }
}

class Switch {
    private Switchable device; // 추상화에 의존

    public Switch(Switchable device) {
        this.device = device;
    }

    public void operate(boolean on) {
        if (on) {
            device.turnOn();
        } else {
            device.turnOff();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Switchable light = new Light();
        Switch lightSwitch = new Switch(light);
        lightSwitch.operate(true);  // 기대 값: "Light is turned on"
        lightSwitch.operate(false); // 기대 값: "Light is turned off"
    }
}
```
- Switch는 Light라는 구체 클래스에 대한 어떠한 정보도 가지고 있지 않다. 오직 Switchable이라는 인터페이스(추상화)에 대한 정보만 가지고 있다.
- 기존에는 Switch가 Light를 의존했기 때문에 [Switch] -> [Light] 처럼 표현할 수 있었다.
- 개선 후에는 Switch가 Switchable을 의존하고, Light도 Switchable을 의존한다. 따라서 [Switch] -> [Switchable] <- [Light] 처럼 표현할 수 있다.
- 의존성 화살표가 변경되었으므로 의존성 역전이 발생했다고 표현할 수 있다.
- Switchable 인터페이스를 구현하는 어떠한 구체 클래스가 런타임 의존 된다 하더라도 Switch의 구현에는 변경이 필요하지 않다.