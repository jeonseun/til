# final 변수, 필드, 상수

변수에 사용하는 `final` 키워드의 효과와 특징

> [!IMPORTANT] Tags
> java, final, variable, constant

> [!NOTE] Environments
>
> - JDK 21

<br>

## TOC

- [final 변수](#final-변수)
  - [final 지역변수](#final-지역변수)
  - [final 매개변수](#final-매개변수)
- [final 필드](#final-필드)
- [기본형 final vs 참조형 final](#기본형-final-vs-참조형-final)
- [상수](#상수)

<br>

## final 변수

`final` 로 변수를 선언하면 값이 변경되지 않는 변수를 만들 수 있다.

### final 지역변수

블록 내에서 `final` 로 선언된 지역변수는 초기화 이후 값을 변경할 수 없다.

- 이미 초기화된 `final` 지역변수에 재할당 시도 시 컴파일 에러
- `final` 지역변수는 소멸할 때까지 최초 할당된 값이 유지됨

```java
public static void main(String[] args) {
    final int finalVariable = 10;
    // finalVariable = 20; // 컴파일 에러!
}
```

### final 매개변수

메서드 매개변수를 `final` 로 선언하면 호출자가 인자로 넘겨준 값을 변경할 수 없다.

- 메서드 내에서 `final` 매개변수에 값 할당 시도 시 컴파일 에러
- `final` 매개변수는 메서드 종료까지 호출자가 넘겨준 값이 유지됨

```java
public void method(final int finalParam) {
    // finalParam = 20; // 컴파일 에러!
}
```

<br>

## final 필드

클래스 정의 시 `final` 로 선언된 필드는 객체 생성 이후 필드값을 변경할 수 없다.

- `final` 로 선언된 필드는 반드시 명시적으로 초기화해야 함, 초기화하지 않으면 컴파일 에러
- `final` 필드는 초기화 이후 값을 변경할 수 없으며 객체 소멸까지 값이 유지됨

```java
public class FinalField {
    public final int finalField1 = 10;
    public final int finalField2;
    // public final int finalField3; // 컴파일 에러!

    public FinalField(int finalField2) {
        this.finalField2 = finalField2;
    }
}
```

```java
public static void main(String[] args) {
    FinalField obj = new FinalField(20);

    // obj.finalField1 = 15; // 컴파일 에러!
    // obj.finalField2 = 15; // 컴파일 에러!
}
```

<br>

## 기본형 final vs 참조형 final

`final` 은 변수에 재할당을 막기 때문에 기본형은 최초 할당된 실제 값이 고정되며 참조형은 최초 할당된 참조값이 고정된다. 단 `final` 로 선언된 참조형은 참조 대상을 변경하지 못할 뿐 참조 대상의 상태는 메서드를 통해 변경할 수 있다.

```java
public class FinalReference {
    private final Data data; // 참조 대상 변경 불가능!

    public FinalReference(Data data) {
        this.data = data;
    }

    public void setData(Data data) {
        // this.data = data; // 컴파일 에러!, 참조 대상 변경 불가능
    }

    public void setNum(int num) {
        data.setNum(num); // 참조 대상의 메서드를 통한 객체 상태 변경!
    }
}

class Data {
    private int num;

    public Data(int num) {
        this.num = num;
    }

    public void setNum(int num) {
        this.num = num;
    }
}
```

```java
public static void main(String[] args) {
    Data data = new Data(10);
    FinalReference finalReference = new FinalReference(data);
    finalReference.setNum(20);
}
```

만약 참조 대상 객체의 상태도 변경하지 못하게 하려면 해당 객체의 모든 필드를 `final` 로 선언해야 한다.

```java
class Data {
    private final int num;
    ...
}
```

<br>

## 상수

`static` 으로 선언된 정적 필드에 `final` 을 붙이면 해당 필드는 상수(constant)가 된다. 상수를 사용하면 매직 넘버의 사용을 피하고 의미있는 이름을 쓸 수 있으며 여러곳에서 사용되어도 한 곳에서 관리할 수 있다.

- `static final` 은 정적 멤버이므로 클래스당 하나만 존재
- `static final` 은 `final` 이므로 클래스 로딩 시점에 초기화이후 재할당 불가능
- `static final` 로 선언된 필드를 선언 또는 정적 초기화 블록에서 초기화하지 않으면 컴파일 에러
- `static final` 로 선언된 필드의 이름은 대문자로 작성하고 단어사이는 `_` 로 연결하는 것이 관례
- 프로그램 전반에 걸쳐 사용되면 `public`, 클래스 내에서만 쓰이는 경우 `private` 로 지정하는 것이 일반적

```java
public class Constant {

    public static final double PI = 3.14;

    public static final int CONSTANT_NUM1 = 10;
    public static final int CONSTANT_NUM2;
    // public static final int CONSTANT_NUM3; // 컴파일 에러!

    static {
        CONSTANT_NUM2 = 20;
    }

    private static final int MAX_STUDENT = 10;   // 학생 수 상수
    private int[] scores = new int[MAX_STUDENT]; // 학생별 점수 배열
}
```

```java
public static void main(String[] args) {
    double radius = 10.0; // 원의 반지름
    double round = 2 * radius * Constant.PI; // 원둘레 구하기, PI 상수 활용

    // Constant.CONSTANT_NUM1 = 20; // 컴파일 에러
    // Constant.CONSTANT_NUM2 = 20; // 컴파일 에러
}
```

> [!TIP] 컴파일 타임 상수
>
> 기본형, 문자열 상수가 컴파일 시점에 값을 알 수 있다면 컴파일러는 해당 상수가 사용된 부분의 코드를 값으로 변경하며 이를 컴파일 타임 상수라 한다.

<br>

## References

- [김영한의 실전 자바 - 기본편 - 김영한, Inflearn](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B8%B0%EB%B3%B8%ED%8E%B8)
- [Java Language Specification se21 #ch4.12.4 - Oracle](https://docs.oracle.com/javase/specs/jls/se21/html/jls-4.html#jls-4.12.4)
- [Java Language Specification se21 #ch8.3.1.2 - Oracle](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.3.1.2)
- [Java Language Specification se21 #ch13.4.9 - Oracle](https://docs.oracle.com/javase/specs/jls/se21/html/jls-13.html#jls-13.4.9)
- [More on Classes #constants - Dev.java](https://dev.java/learn/classes-objects/more-on-classes/#constants)
