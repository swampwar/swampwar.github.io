---
layout: post
title: equals와 hashcode
tags: [JAVA, equals, hashcode]
---

롬복의 @EqualsAndHashCode 애노테이션을 보다가 문득 equals()와 hashCode() 메서드에 대해서 잘 모른다는 사실을 알고 한번 찾아보고 정리한다.

Object 클래스에 정의된 equals(Object obj) 메서드를 보면 '==' 비교결과를 반환한다.

```java
class Object{
    public boolean equals(Object obj) {
        return this == obj;
    }    
}
```

그럼 '==' 비교는 무엇을 뜻하는 것일까? 참조형 변수끼리 '==' 비교를 한다면 '동일' 객체를 참조하고 있는지 확인하는 것이다.  
(논리적으로 같은 객체가 아니다.)

```java
class Game{
    int star;

    public Game(int star) {
        this.star = star;
    }
}
```

```java
Game game1 = new Game(1);
Game game2 = game1;

// '동일' 객체를 참조하므로 game1 == game2
```

```java
Game game1 = new Game(1);
Game game2 = new Game(1);

// 같은 프로퍼티를 가지므로 논리적으로 동일한 객체지만 
// '동일' 객체를 참조하지 않으므로 game1 != game2
```

하지만 개발을 하다보면 '동일'한 객체인지 체크하는 것보다 논리적으로 '동등'한지(모든 프로퍼티가 같은 객체인지) 체크할 상황이 더 많다.  
이런 경우에는 보통 클래스에 `Object.equals(Object obj)`를 재정의한다.

```java
class GameWithEquals{
    int star;

    public GameWithEquals(int star) {
        this.star = star;
    }

    @Override
    public boolean equals(Object game) {
        if(!(game instanceof GameWithEquals)) return false;
        return ((GameWithEquals) game).star == this.star;
    }
}
```

```java
@Test
public void 재정의한_equals_동등비교(){
    GameWithEquals game1 = new GameWithEquals(1);
    GameWithEquals game2 = new GameWithEquals(1);

    assertTrue(game1.equals(game2));
}
```

단순히 '==' 비교를 하던 `Object.equals(Object obj)`를 재정의 했고,
객체간의 동등비교가 필요하면 이 함수를 활용하면 되니 모든 문제가 해결되었다고 생각할 수 있지만
equals()와 짝으로 hashcode()메서드도 재정의 되어야 한다.

왜냐하면 컬렉션들(HashSet, HashMap, HashTable ...)은 객체가 동등한지 비교할 때 equals()와 hashcode()를 둘다 체크하기 때문이다.
예를들어 equals()만 재정의한 경우에는 다음 테스트를 통과하지 못한다.

```java
@Test
public void Map은_키값비교를_equals만으로_하지않는다(){
    HashMap<GameWithEquals, String> map = new HashMap<>();
    GameWithEquals game1 = new GameWithEquals(1);
    map.put(game1, "HI");

    GameWithEquals game2 = new GameWithEquals(1);
    assertTrue(game1.equals(game2)); // True
    assertEquals(map.get(game2), "HI"); // null반환
}
```

컬렉션은 키값 객체를 비교할 때 equals()와 hashcode()를 모두 체크하므로 재정의 하지 않은 hashcode()에 의해 테스트는 실패하게 된다.  
`Object.hashCode()`는 native로 정의되어 있으며 실제 참조하는 객체에 대한 교유값이므로 '==' 비교와 동일하게 *다른객체를 참조한다면 서로 다른값이 반환된다.*
논리적인 비교가 되기 위해서는 equals와 마찬가지로 가진 프로퍼티를 이용하여 hashCode를 반환하도록 오버라이드 한다.

```java
class GameWithEqualsAndHashCode{
    int star;

    public GameWithEqualsAndHashCode(int star) {
        this.star = star;
    }

    @Override
    public boolean equals(Object game) {
        if(!(game instanceof GameWithEqualsAndHashCode)) return false;
        return ((GameWithEqualsAndHashCode) game).star == this.star;
    }

    @Override
    public int hashCode() {
        return Objects.hash(star);
    }
}
```

```java
@Test
public void Map은_키값비교를_equals와_hashCode로_한다(){
    HashMap<GameWithEqualsAndHashCode, String> map = new HashMap<>();
    GameWithEqualsAndHashCode game1 = new GameWithEqualsAndHashCode(1);
    map.put(game1, "HI");

    GameWithEqualsAndHashCode game2 = new GameWithEqualsAndHashCode(1);
    assertTrue(game1.equals(game2)); // True
    assertEquals(map.get(game2), "HI"); // HI반환, True
}
```

정리하면,  
객체간의 논리적인 비교를 위해서는 equals()와 hashCode()를 모두 재정의 해야한다.  
재정의는 개발자가 작성해도 되지만 IDE의 자동완성이나 Lombok(@EqualsAndHashCode)의 도움으로 자동작성 할 수도 있다.
