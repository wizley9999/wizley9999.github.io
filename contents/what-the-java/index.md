---
title: "자바는 어지러워요"
date: "2025-07-10"
hidden: true
---

스스로 1일 1커밋에 잡아 먹히느라 오늘도 글을 써본다.

오늘은 자바에 대해서 공부했다. 정처기를 위해서지만 어차피 해야할 거, 불만은 없다.

나는 자바를 별로 좋아하지 않는다. 아니, 정확히는 이클립스 IDE를 좋아하지 않는다.

이클립스를 처음 알게된 건 14~15년도 때였던 것 같다. 안드로이드 개발에 호기심이 생겨 도전해보고 싶었는데, 그 때 이클립스 IDE를 사용해서 개발한다고 해서 무식하게 이클립스 깔아보고 SDK 깔고 그랬던 기억이 있다. 물론 호기심에 그치긴 했지만 말이다.

그 때의 이클립스를 또 마주하게 될지는 몰랐다.

바야흐로 대학교 2학년 시절, 수업 때 한창 자바랑 JSP 배운다고 이클립스 IDE를 (강제로) 썼었는데 몸이 거부하던 그 느낌을 지울 수가 없다.

JDK, Tomcat, JDBC.. 뭐가 그렇게 깔아야 하는 것도 많은 건지.. 가볍게 사용하기가 진짜 어려웠다.

> 아니 심지어 JDK는 OpenJDK도 있고, JSE도 있고 JSE는 Oracle JDK이고. 뭐가 이렇게 여기저기 복잡한 거야;

그래서 사실 해당 과목 이후에 고이고이 접어둔 것 중에 하나가 자바인 것 같다.

뭐 어쩌겠나. 또 하려면 해야지. 다시 펼쳐보자..

## 동적 바인딩과 정적 바인딩에 대해서 아시나요?

자바 기준으로

1. 필드에 접근할 때는 정적 바인딩이고,
2. 메소드에 접근할 때는 동적 바인딩이래요.

```java
public class Main {
    public static void main(String[] args) {
      B a = new D();
      D b = new D();

      System.out.print(a.x);
      System.out.print(a.getX());

      System.out.print(b.x);
      System.out.print(b.getX());
  }
}

class B {
    int x = 3;

    int getX() {
        return x * 2;
    }
}

class D extends B {
    int x = 7;

    int getX() {
        return x * 3;
    }
}
```

> `B a = new D();` 아잇! 인스턴스를 이렇게 생성하지 말아다오..

컴파일 타임에 a는 B로 인식한다. 근데 런타임에는 D로 사용한다. (띠용?) 실제 사용하는 객체는 `D `타입이다. `a`에 `getClass`를 찍어보면 `class D`로 나온다.

근데 컴파일은 뭐냐. 텍스트를 머신 코드로 바꾸는 행위 아닌가!

`a` 객체의 필드에 접근하는 코드는 컴파일 타임에 정적으로 바인딩 되어 `B`로 인식되고 그게 `B`의 `x`에 접근하게 된다.

반대로 `a`의 메소드에 접근하는 코드는 런타임에 동적으로 바인딩 되어 `D`로 인식되고 그게 `D`의 오버라이딩 된 `getX` 함수로 접근하게 된다.

> 이유는 묻지 마라. 자바 설계 원칙이 그렇다는데 어떻게 해..

그래서 `a.x`는 `3`이고 `b.x`는 `7`이다. `getX()`는 둘 다 `D`의 `getX`를 사용하면 된다.

> 오버라이드 어노테이션 안 쓰면 그냥 컴파일 에러 나게 해주세요..

### 짜잔~ 저는 `static` 키워드예요~

클래스 필드랑 메소드에는 `static` 키워드를 사용할 수 있다.

```java
public class Main {
    public static void main(String[] args) {
      B a = new D();
      D b = new D();

      System.out.print(a.x);
      System.out.print(a.getX());

      System.out.print(b.x);
      System.out.print(b.getX());
  }
}

class B {
    static int x = 3;

    static int getX() {
        return x * 2;
    }
}

class D extends B {
    static int x = 7;

    static int getX() {
        return x * 3;
    }
}
```

참고로 `static` 메소드는 오버라이딩이 아니고 하이딩이라고 부른다. 왜냐? 오버라이딩은 동적 바인딩(런타임)에 덮어져 쓰여서 "재정의"가 이루어지는 거고, 하이딩은 정적 바인딩(컴파일 타임)에 둘 중에 뭐가 쓰일지 결정되는 방식이라 그렇다.

다시 돌아와서, `a.x`랑 `b.x`에는 뭐가 나올까요~?

아까 본 코드랑 마찬가지로 컴파일 타임에 결정되는 건 똑같다. 그래서 각각 `3`이랑 `7`이 나온다.

그럼 `getX` 메소드는요? `a.getX()`는 이미 컴파일 타임에 `B`의 `getX`를 사용하기로 결정했기 때문에 오버라이딩과 다르게 `x * 2`의 값이 나온다.

이를 메소드 하이딩이라고 부르는 것 같다.

## 상속을 사용하면 `this`는 더이상 당신의 것이 아니에요..

```java
public class Test {
    public static void main(String[] args) {
        int x = 7;
        ClassB cal = new ClassB();
        cal.prn(x);
      }
}

class ClassA {
    ClassA(){
        System.out.print('A');
        this.prn();
    }

    void prn() {
        System.out.print('B');
    }
}

class ClassB extends ClassA {
    ClassB() {
        System.out.print('D');
    }

    void prn() {
        System.out.print('E');
    }

    void prn(int x){
        System.out.print(x);
    }
}
```

`main` 함수의 코드 흐름을 먼저 봐야한다.

cal 인스턴스를 생성할 때 당연히 클래스 A의 생성자를 먼저 호출한다. `ClassB` 생성자의 바디중 가장 맨 윗줄에 `super();`가 생략되어 있음을 까먹지 말자.

`super();`를 호출하는 건 결국 `ClassA` 생성자를 호출하는 것과 같은데 그 안에 있는 `this.prn();`은 과연 어느 클래스의 메소드를 호출할 것인가?

ClassA의 메소드였다면 마음이 편했을 것을.. cal의 실제 객체 타입인 B의 `prn`을 호출한단다.

결국 `this`는 cal의 실제 객체 타입인 ClassB를 가리키고 그 클래스의 오버라이드된 메소드를 사용한다.

> static 메소드면 정적 바인딩으로 ClassA의 메소드를 사용한다.

### this 키워드가 없더라도 당신의 것이 아니에요..

```java
public class Test {
    public static void main(String[] args) {
        SuperObject a = new SubObject();
        a.paint();
        a.draw();
    }
}

class SuperObject {
    public void draw() {
        System.out.println("A");
        draw();
    }

    public void paint() {
        System.out.print('B');
        draw();
    }
}

class SubObject extends SuperObject {
    public void paint() {
        super.paint();
        System.out.print('C');
        draw();
    }

    public void draw() {
        System.out.print('D');
    }
}
```

앞서 말했던 맥락과 비슷하게 `SuperObject`의 `paint`, `draw` 함수를 호출하는 객체를 잘 봐야한다. 실제로 객체는 `SubObject` 타입이기 때문에 클래스 메소드 내에서 `paint();`를 호출하더라도 오버라이드된 메소드인 `SubObject`의 `paint`가 호출이 된다.

즉, `paint();`는 `this.paint();`가 되는 것이다.

> `error: method is ambiguous`
>
> 그냥 이렇게 컴파일 에러 내줘 ㅜ.ㅜ

## 예외처리

```java
try {
    // 예외가 발생할 가능성이 있는 코드
} catch (ExceptionType1 e1) {
    // ExceptionType1 예외가 발생했을 경우 실행할 코드
} catch (ExceptionType2 e2) {
    // ExceptionType2 예외가 발생했을 경우 실행할 코드
} finally {
    // 예외 발생 여부와 상관없이 항상 실행
}
```

Exception catch는 항상 사용하던 느낌 그대로 사용하면 될 것 같고, finally만 마지막에 무조건 실행된다고 외우면 된다.

그리고 try나 catch에서 return을 호출하더라도 finally는 꼭꼭 무조건 호출하고 반환됨을 잊지 말자.

### `finally`에서 return 하면 어떡하나요..?

그럼 사실 try, catch에 있는 return은 `unreachable`하다고 보는 게 맞다. (물론 조건 분기가 있다면 실행이 될 수도 있겠지만..)

그러니 무조건 finally는 항상 실행이 됨을 기억하자.

## 제네릭? 타입이 도망가버렸어요!

자바의 제네릭은 컴파일 타임에만 존재한다는 특징이 있다. 다시 말해, 런타임에는 타입을 전혀 신경쓰지 않는다는 말이다.

제네릭은 컴파일러가 타입을 검사하는 데만 사용되고, JVM에는 실제 타입 정보가 전달되지 않는다.

그래서 런타임에는 타입이 가장 최상위 클래스인 `Object`로 대체된다.

이를 "타입 소거"라고 부른다.

```java
class Box<T> {
    T value;

    public Box(T value) {
        this.value = value;
        this.print(value);
    }

    public void print(Integer x) {
        System.out.println("Integer");
    }

    public void print(Object x) {
        System.out.println("Object");
    }
}
```

위와 같은 `Box` 제네릭 클래스가 있다.

```java
Box<Integer> box = new Box<>(123);
```

자, 오버로딩된 메소드이 출력의 결과가 뭘까? 바로바로 앞서 얘기했듯이 `Object`가 출력된다.

이게 타입 소거다.

> 솔직히 이건 컴파일 타임을 먼저 알아봐야 오히려 이해하기 쉬울 것 같다. 인터넷에 제네릭 컴파일 타임에 대한 내용들이 많이 있으니 검색해서 보길 추천한다.
