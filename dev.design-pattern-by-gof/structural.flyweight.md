# 구조 패턴 > 플라이웨이 패턴

## 개요

- 객체를 가볍게 만들어 메모리 사용을 줄이는 패턴
- 자주 변하는 속성(extrinsit)과 변하지 않는 속성(intrinsit)을 분리하고 재사용하여 메모리 사용을 줄일 수 있다.
  - 자주 바뀐다, 바뀌지 않는다는 주관적 기준이고 상황에 따라 다르다.
- 어떤 클래스의 인스턴스가 아주 많이 필요하지만 모두 똑같은 방식으로 제어해야 할 때 유용하게 쓰인다.

## 예시 (Before)

```java
public class Character {
    private char symbol;
    private int x;
    private int y;

    public Character(char symbol, int x, int y) {
        this.symbol = symbol;
        this.x = x;
        this.y = y;
    }

    public void display() {
        System.out.println("Character: " + symbol + " at position (" + x + ", " + y + ")");
    }
}

public class BeforeFlyweight {
    public static void main(String[] args) {
        String document = "Hello World";
        char[] chars = document.toCharArray();

        int x = 0;
        int y = 0;

        for (char c : chars) {
            Character character = new Character(c, x, y);
            character.display();
            x++;
        }
    }
}
```
- 글자 1개를 display 할 때마다 Character 객체가 메모리에 생성된다.

## 예시 (After)

1. Extransit으로 좌표를, Intransit으로 심볼으로 잡자
2. 그리고 쪼개자
3. 심볼은 캐시해서 재사용하고 좌표는 동적으로 변경해주자

```java
// Flyweight 인터페이스
interface FlyweightCharacter {
    void display(int x, int y);
}

// ConcreteFlyweight 클래스
class ConcreteFlyweightCharacter implements FlyweightCharacter {
    private final char symbol;

    public ConcreteFlyweightCharacter(char symbol) {
        this.symbol = symbol;
    }

    @Override
    public void display(int x, int y) {
        System.out.println("Character: " + symbol + " at position (" + x + ", " + y + ")");
    }
}

// FlyweightFactory 클래스
class FlyweightCharacterFactory {
    private final Map<Character, FlyweightCharacter> characters = new HashMap<>();

    public FlyweightCharacter getCharacter(char symbol) {
        FlyweightCharacter character = characters.get(symbol);
        if (character == null) {
            character = new ConcreteFlyweightCharacter(symbol);
            characters.put(symbol, character);
        }
        return character;
    }
}

// 클라이언트 코드
public class AfterFlyweight {
    public static void main(String[] args) {
        FlyweightCharacterFactory factory = new FlyweightCharacterFactory();

        String document = "Hello World";
        char[] chars = document.toCharArray();

        int x = 0;
        int y = 0;

        for (char c : chars) {
            FlyweightCharacter character = factory.getCharacter(c);
            character.display(x, y);
            x++;
        }
    }
}
```
- 동일한 문자열은 characters라는 HashMap에 저장해서 메모리 최적화를 하고 있다.
- 변하는 좌표값은 어디에도 저장하지 않는다.

## 장점

- 실행 시 객체 인스턴스의 개수를 줄여서 메모리를 절약할 수 있다.
- 여러 가상 객체의 상태를 한곳에 모아 둘 수 있다.

## 단점

- 일단 이 패턴으로 구현해 놓으면 특정 인스턴스만 다른 인스턴스와 다르게 행동하게 할 수 없다는 단점이 있다.
  - 오히려 장점이 될 수도?
