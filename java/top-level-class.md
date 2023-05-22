# Top Level 클래스

[전체 소스코드](https://github.com/jeonseun/til-code/tree/main/java/src/main/java/me/seun/toplevel)

top level 클래스는 다른 클래스의 멤버가 아닌 클래스 즉 중첩 클래스가 아닌 클래스를 말한다.

다음과 같은 클래스 `A`, `B`, `C` 가 하나의 소스파일 A.java에 있을 때 `A` 와 `C` 가 top level 클래스가 된다.

```java
// A.java
// top level 클래스 A
public class A {
    ...
    // 정적 중첩 클래스 B
    static class B {
        ...
    }
}

// top level 클래스 C
class C {
    ...
}
```

## Top Level 클래스 컴파일

자바 소스파일은 파일하나에 클래스 하나를 정의하지만 필요한 경우 두개 이상의 클래스를 정의할 수 있다. 두개 이상의 클래스가 포함된 소스파일을 컴파일하면 소스코드에 속한 클래스의 개수만큼 `.class` 파일이 생성된다. 이는 여러개의 top level 클래스 뿐아니라 중첩 클래스의 경우에도 마찬가지로 적용된다.

위 예제의 클래스 A.java를 컴파일하면 총 3개의 `.class` 파일이 생성된다.

- `A.class` - `public` top level 클래스 `A` 의 `.class` 파일
- `C.class` - `package-private` top level 클래스 `C` 의 `.class` 파일
- `A$B.class` - `A` 의 정적 중첩 클래스 `B` 의 `.class` 파일

## Top Level 클래스 접근 제한

top level 클래스는 `public`, `package-private` 로 두가지 접근 제한 옵션을 적용할 수 있으며 다음 규칙이 적용된다.

- 하나의 소스파일에는 하나의 `public` top level 클래스만 정의할 수 있다.
- 소스파일의 파일명과 `public` top level 클래스의 이름은 같아야 한다.
- `public` top level 클래스가 없는 경우 소스파일 내 top level 클래스 중 하나의 이름으로 소스파일의 파일명을 결정하면 된다.

## 여러개의 Top Level 클래스

캡슐화 등의 목적이 있다면 하나의 소스파일에 여러개의 top level 클래스를 선언하는 것은 좋은 선택이 될 수 있으나 별다른 이유가 없다면 하나의 소스파일에는 하나의 top level 클래스만 선언하는 것이 바람직하다.