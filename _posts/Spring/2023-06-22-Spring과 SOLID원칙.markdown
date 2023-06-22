---
layout: post
title:  "Spring과 SOLID원칙"
date:   2023-06-22 14:42:00 +0900
categories: spring
tags: spring
---

## Spring과 SOLID원칙

## SOLID란 무엇일까?

SOLID란, 객체지향 프로그래밍 및 설계 시 좋은 코드를 위하여 지켜야할 5가지 원칙을 말한다.<br>
해당 원칙은 코드의 유지보수를 쉽게하고, 기능의 확장에 용이하게하는 등 변화에 잘 대응 하고 좋은 설계를 하기 위한 하나의 전략이다.<br>
SOLID라는 개념은 자바나, 스프링에 국한된 개념이 아니지만, Spring과 아주 밀접한 관계가 있기때문에 해당 카테고리에서 다뤄보려고 한다.


### Spring과 SOLID의 관계

Spring은 SOLID의 원칙을 철저하게 지킨 프레임워크라고 할 수 있다.<br>
또한 Spring은 개발자가 SOLID원칙을 준수해서 코드를 작성할 수 있도록, 많은 기능들을 제공하고 이에 맞추어 개발자들은 유지보수에 용이한 프로그램 코드를 짤 수 있게 된다.

### SRP 단일 책임 원칙

* 한 클래스는 하나의 책임만 가져야 한다.

하나의 클래스에 View,Controller,Model의 책임을 모두 가지고 있다면, 어떻게 될까?<br>
모든 기능들이 하나의 클래스에 정의되어 있기 때문에, 만약 해당 코드에 대한 기능이 변경되거나 문제가 생겼을 때 수정이 힘들 것이다.<br>
하지만 하나의 책임이라고 해서, 무조건 적으로 클래스를 나눠서 설계하는 것은 옳지 못하다.<br>
클래스가 가지는 책임의 범위는 어디까지나 개발자의 설계에 따라 달라지는 것이므로,<br> 
여기서 말하는 단일 책임의 원칙은 구현할 서비스에 대해서 적절하게 책임의 범위를 잘 나눈 후 해당 책임만 설정하는 SRP를 잘 지켰다고 할 수 있다.

### OCP 개방/폐쇄 원칙

* 소프트웨어 요소는 확장에는 열려 있고 변경에는 닫혀 있어야한다.

객체지향의 대표 요소중 다형성을 활용하면 이처럼 OCP를 잘 지킬 수 있다.<br>
아래의 코드는 OCP가 제대로 지켜지지 않은 코드의 예시이다.

```java
public class MemberService{
    private MemberRepository memberRepository = new MemoryMemberRepository();
    // private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

MemberService 클라이언트는 적용할 Repository를 매번 일일히 코드를 수정해주어야 하며, 이는 OCP원칙이 어긋난다.<br>
위의 예제에서는 현재 하나의 코드만 수정이 되었지만, 만약 MemberRepository를 참조하는 수많은 클래스가 있다면, 모두 하나하나 고쳐줘야하는 번거로움이 생길 것이다.<br>
위의 코드를 리팩토링 해보자.<br>

```java
public class MemberService{
    private MemberRepository memberRepository = new MemberRepository();
}
```

```java
public class AppConfig{
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }
    
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
```
위 코드를 살펴보면, MemberService는 일일히 memberRepository의 구현객체를 지정해줄 필요없이, AppConfig에서 설정한 구현객체가 자동으로 생성됨을 볼 수있다. <br> 
만약 해당 MemberRepository의 구현객체를 변경해주고 싶다면, AppConfig에 정의된 MemberRepository의 구현체만 수정해주면, MemberRepository를 참조하는 모든 클래스에 MemberRepository의 구현 객체를 변경할 수 있게된다. <br> 
이처럼 OCP개념은 코드의 유지보수와 변경에 유연함이 생기고, 클린한 코드를 구성할 수 있다.


### LSP 리스코프 치환 원칙

* 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야한다.

리스코프 치환 원칙은 개발자가 정의한 상위 클래스 혹은 인터페이스에서 정의한 규약들을 하위 클래스가 상속받았을 때, 해당 규약이 깨지면 안된다는 의미이다.<br>
바꿔말해서 하위 타입은 언제나 상위 타입으로 교체할 수 있어야 한다라는 의미로도 볼 수 있다.<br>
자동차를 예를들어 보자. <br>
자동차는 엑셀을 밟으면, 항상 앞으로 간다. <br> 
물론 제작자가 엑셀을 밟으면 뒤로가는 자동차를 만들 수도 있지만, 이는 LSP를 위반한 것이다.<br>
이유는 상위클래스(인터페이스)에서 자동차는 앞으로 간다고 규약을 지정했기 때문이다. 이처럼, LSP는 컴파일 단계를 넘어서, 상위클래스가 지정한 규약을 반드시 지켜야 한다.
<br>

아래의 코드 예시로 한번 이해해보자.
```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.LinkedList;

public class LSP {
    public void lsp() {
        Collection collection = new ArrayList();
    }

    public void testAdd(Collection collection) {
        collection.add(123);
    }
}
```
collection이은 Collection 타입으로 선언이 되어있지만 실제 구현 객체는 ArrayList로 대체 되었다.<br>
이처럼, ArrayList는 Collection의 정의한 메서드(add)를 지키며, Collection(상위 클래스)로 대체 되더라도 아무 문제가 없다.


### ISP 인터페이스 분리 원칙

* 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 보다 낫다.

이번에도 자동차와 사람을 예를 들어보자.<br>
자동차에는 운전 인터페이스와 정비 인터페이스가 있고, 사람은 운전자 클라이언트와, 정비사 클라이언트가 있다.<br>
이렇게 자동차라는 하나의 인터페이스를 운전과 정비로 분리한다면, 운전자는 정비 인터페이스가 변경되더라도, 전혀 문제가 없다.<br>
반대로 정비사는 운전 인터페이스가 변경되어도 전혀 문제가 없다.<br>
이처럼 인터페이스가 클라이언트에 따라 분리된다면, 인터페이스의 역할이 좀더 명확해지고, 유지보수나 대체 가능성이 높아진다. 


### DIP 의존관계 역전 원칙

* 개발자는 구현체에 의존하는 것이 아닌 추상화에 의존해야한다.

공연을 예를들어 보자.<br>
공연에서의 역할은 언터페이스, 배우는 구현체이다. 공연은 역할에 의존해야지 배우에 의존해선 안된다.<br>
만약 배우가 사정이 생겨서 공연을 하지 못하게 된다면, 다른 배우가 해당 역할을 수행해야 할 것이다. 하지만 배우에게 의존하게 된다면, 해당 공연은 진행이 불가능해 질 것이다. 이처럼 DIP는 구현체(배우)에 의존하기보다 추상클래스(역할)에 의존해야 한다는 뜻이다.

다음 예시 코드를 살펴보자.


```java
public class MemberService{
    private MemberRepository memberRepository = new MemberRepository();
}
```

```java
public class AppConfig{
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }
    
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
```
해당 코드는 이미 OCP에서 한번 나온 코드이다.<br>
MemberService는 MemberRepository라는 인터페이스(추상화)에만 의존함으로써, AppCofig의 설정에 따라서 언제든지 구현체가 변경될 수 있다.<br>
따라서, 역할과 구현을 철저하게 분리하고, 변경에 유연하게 적용할 수 있다.



---

## 정리

* SOLID는 객체지향 프로그래밍 및 설계시 좋은 코드를 위한 5가지 원칙이다.
* 단일 책임 원칙 : 하나의 클래스는 하나의 책임만 진다.
* 개방 폐쇄 원칙 : 확장에는 열려있고, 변경에는 닫혀있어야 한다.
* 리스코프 치환 원칙 : 하위타입은 상위타입의 규약을 지키며 완전히 대체될 수 있어야 한다.
* 인터페이스 분리 원칙 : 특정 클라이언트를 위한 여러개의 인터페이스가 하나의 범용 인터페이스 보다 낫다.
* 의존관계 역전 원칙 : 개발자는 구현체에 의지하는 것이 아닌, 추상화에 의존해야 한다.

실제 개발을 하면서, 프로젝트 일정이나, 다양한 사유로 이러한 원리들이 잘 지켜지지 않았다.<br>
하지만, 항상 해당 원칙들을 준수해야 클린 코드가 나온다는 사실을 기억하고, 지키며 개발하려고 노력하자.😊