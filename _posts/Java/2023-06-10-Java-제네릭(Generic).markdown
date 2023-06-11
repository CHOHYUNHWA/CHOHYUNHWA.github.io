---
layout: post
title:  "Java-제네릭(Generic)"
date:   2023-06-10 10:42:00 +0900
categories: java
tags: java
---

## Java-제네릭(Generic)

## 제네릭(Generic)

제네릭(Generic)을 번역하면 '일반적인'이란 뜻이다.<br>
단순히, 영어단어를 해석해서는 무슨 뜻인지 유추하기가 어렵다..<br>
Java에서 제네릭은 **데이터 형식에 의존하지 않고, 하나의 값이 여러개의 데이터 타입들로 사용될 수 있도록 하는 자바의 문법**이다.

일반적으로 우리가 클래수 내부 필드나 메서드를 선언할 때 아래 코드와 같이 구체적인 타입을 지정해주었다.

```java
class Group{
    private String element;

    Group(String item){
        this.item = item;
    }

}
```

위의 예시에서 ```element```의 타입은 ```String```으로 지정되어 있고, 위 클래스로 생성된 인스턴스는 ```String```타입의 ```element```만 저장할 수 있다.<br>

```java
class GroupString{
    private String element;
}

class GroupInteger{
    private int element;
}

class GroupChar{
    private char element;
}
```

하지만 이 방법은 코드의 길이도 길어지며, 리소스가 많이 든다.<br>
이와 같은 문제점을 해결하기 위해서 **제네릭(Generic)**을 사용하면 된다.

### Generic 문법

제네릭은 다음과 같이 사용할 수 있다.
```java
class Group<T>{
    private T element;

    public Group(T element){
        this.element = element;
    }
}

public class GenericExample {
    public static void main(String[] args) {
        Group<String> group1 = new Group<String>("요소");
        Group<Integer> group2 = new Group<Integer>(1);
    }
}
```

위와 같이 클래스명옆에 \<T>가 추가되었으며, 클래스 내부의 필드 ```element```의 타입이 ```T```라는 타입으로 선언되었다.<br>
여기서 \<T>와 T가 제네릭 문법에 해당하며, 객체를 생성할 때 해당 T값을 ```Group<String> group1 = new Group<String>();```과 같이 ```String```타입으로 지정해 준 것이다. **```<T>``` -> ```<String>``` 으로 치환** 된다고 생각하면 쉽다.<br>
이렇게 제네릭을 이용하면, 타입별로 클래스를 지정해주지 않아도 하나의 클래스로 여러타입의 필드를 가진 객체를 생성할 수 있다.

### Generic 컨벤션

일반적으로 Generic 타입명에 대한 제한은 없다. (```<TEST>```라고 지정해도 무방함)<br>
하지만, 대중적으로 사용하는 컨벤션은 존재함으로 해당 컨벤션을 따라서 코드를 작성하면, 코드의 가독성이 높아질 것이다.<br>

|타입|설명|
|:--------:|:--------:|
| \<T> | Type |
| \<E> | Element |
| \<K> | Key |
| \<V> | Value |
| \<N> | Number |

따라서, 위와 같은 컨벤션을 유의하여, 제네릭을 사용하자<br>
Java의 공식문서도 위의 컨벤션으로 작성되어 있음을 참고

### 제한된 제네릭 클래스

제네릭은 상속과 인터페이스의 개념을 통하여, 제네릭 클래스의 타입을 제한하여 줄 수도 있다.

```java
class Human{}
class Singer extends Human{}
class Animal{}

class Group<T extends Human>{
    private T element;
}

public class GenericExample {
    public static void main(String[] args) {
        Group<Singer> groupSinger = new Group<>();
        Group<Animal> groupAnimal = new Group<>(); //에러
    }
}
```

```java
interface Human{}
class Singer implements Human{}
class Animal{}

class Group<T extends Human>{
    private T element;
}

public class GenericExample {
    public static void main(String[] args) {
        Group<Singer> groupSinger = new Group<>();
        Group<Animal> groupAnimal = new Group<>(); //에러
    }
}
```

위의 두 코드예시와 같이 ```extends```키워드를 사용하면 **특정 클래스를 상속받은 클래스**, **특정 인터페이스를 구현한 클래스**만 타입으로 지정할 수 있도록 제한할 수 있다.

* 주의 ```generic```에서의 ```extends```키워드는 확장한다는 의미로 상속받은 클래스 뿐만 아니라 인터페이스를 구현한 클래스에도 함께 사용함

### 제네릭 메서드

제네릭은 메서드 단계에서도 사용할 수있다.

```java
class Group{
    public <T> void test(T element){
        System.out.println("출력은 " + element + " 입니다.");
    }
}

public class GenericExample {
    public static void main(String[] args) {
        Group group = new Group();
        group.test("문자열");
        group.test(123);
    }
}

//출력
출력은 문자열 입니다.
출력은 123 입니다.
```

위의 예시와 같이 메서드를 선언할 때가 아닌 메서드를 호출 시점에 제네릭의 타입을 정의하여 사용할 수 있다.

### 와일드카드와 제한된 제네릭 클래스(2)

제네릭은 또한 앞에서 배운 ```extends```와 ```super```를 통하여, 조금 더 타입의 범위를 특정하고 제한할 수 있다.<br>
그리고 제네릭에서는 **와일드카드**라는 것이 있다.<br>
```?```로 '알 수 없는 타입' 혹은 '무엇이든 가능한 타입' 이라는 의미이다.<br>
<br>
다음 예시를 한번 살펴보자.

```java
<? extends T> //T와 T의 자손 타입만 가능
<? super T> //T와 T의 상위(조상) 타입만 가능
<?> //모든 타입이 가능 <? extends Object>와 같은 의미
```

위와 키워드를 이해하기 쉽게 정리하면 다음과 같다.<br>
* ```? extends T``` : 상한 제한
* ```? super T``` : 하한 제한

```java
class A{}
class B extends A{}
class C extends B{}
class D extends A{}
class E extends D{}
```
일 때,

```java
<T extends B> //B와 B의 하위클래스 (B,C)
<T extends A> //A와 A의 하위클래스 (A,B,C,D,E)
<T extends E> //E만 가능

<? extends B> //B와 B의 하위클래스 (B,C)
<? extends A> //A와 A의 하위클래스 (A,B,C,D,E)
<? extends E> //E만 가능
```

```java
<T super B> //B와 B의 상위 클래스 (A,B)
<T super A> //A와 A의 상위클래스 (A)
<T super E> //E와 E의 상위클래스 (E,D,A)

<? super B> //B와 B의 상위 클래스 (A,B)
<? super A> //A와 A의 상위클래스 (A)
<? super E> //E와 E의 상위클래스 (E,D,A)
```

과 같이 설명할 수 있다.<br>
그렇다면, 얼핏보면 \<T>나 타입을 지정하는 것과 \<?>는 같아보이는데 어떤 차이가 있는 것일까?<br>
아래의 코드 예시를 한번 살펴보자
```java
import java.util.Arrays;
import java.util.List;

public class WildCardExample {
    public static void main(String[] args) {
        List<String> strList = Arrays.asList("one", "two", "three");
        List<Integer> intList = Arrays.asList(0,1,2,3,4);
        printList(strList);//에러
        printList(intList);//에러
    }
    public static void printList(List<Object> list) {
        for(Object element : list) {
            System.out.print(element + " ");
        }
        System.out.println();
    }
}
```
에서는 ```List<String>```,```List<Integer>```타입은 ```Object```의 하위 타입이 아니기 때문에 에러를 반환한다.<br>

하지만 와일드카드를 사용하게 되면 어떠한 객체든 파라미터로 받을 수 있기때문에 아래의 코드들을 실행 시킬 수 있다.

```java
import java.util.Arrays;
import java.util.List;

public class WildCardExample {
    public static void main(String[] args) {
        List<String> strList = Arrays.asList("one", "two", "three");
        List<Integer> intList = Arrays.asList(0,1,2,3,4);
        printList(strList);
        printList(intList);
    }
    public static void printList(List<?> list) {
        for(Object elemement : list) {
            System.out.print(elemement + " ");
        }
        System.out.println();
    }
}

//출력
one two three 
0 1 2 3 4 
```

쉽게 말해서, 와일드카드는 어떤타입이든 올 수 있는 제네릭이라고 생각하면 된다.


### 정리

* 제네릭은 데이터 형식에 의존하지 않고, 하나의 값이 여러개의 데이터 타입들로 사용될 수 있도록 하는 자바의 문법
* 제네릭을 사용하면 코드의 재사용성이 높아진다.
* 클래스 내부가 아닌 클래스 외부에서 타입을 지정하여, 타입을 체크하고 변환할 필요가 없어 관리가 편리하다.
* ```extends```와 ```super```를 통해 타입을 제한해줄 수 있다.
* 와일드카드는 어떠한 타입이든 사용할 수 있다.






