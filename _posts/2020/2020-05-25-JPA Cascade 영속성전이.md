---
layout: post
title: JPA Cascade 영속성전이
tags: [JPA, Cascade, 영속성전이, EntityManager]
---

### JPA Cascade

`Cascade`의 사전적인 의미는 종속, 전이라는 뜻이다.  
보통 부모-자식 관계의 엔티티에서 부모쪽의 상태 변경이 자식 쪽으로 전이되도록 한다. 
즉 부모객체만 저장 또는 삭제 했을 때에도 연관된 자식객체가 같이 저장이 되거나 삭제되는 것이다.

```java
@Getter
@Setter
@Entity
public class Parent {
    @Id @GeneratedValue
    Long id;
    String name;

    @OneToMany(mappedBy = "parent")
    List<Child> children = new ArrayList<>();

    public void addChild(Child child){
        this.children.add(child);
        child.setParent(this);
    }
}

@Getter
@Setter
@Entity
public class Child {
    @Id @GeneratedValue
    Long id;
    String name;

    @ManyToOne
    Parent parent;
}
```

양방향으로 관계가 정의된 Parent 엔티티와 Child 엔티티가 있다. 만일 다음과 같이 객체를 맵핑해준 후에 parent만 저장한다면 어떻게 될까? 당연히 parent만 저장했으므로 child는 저장되지 않는다. (cascade의 디폴트 설정은 '어떠한 상태도 전이하지 않음' 이다.)

```java
@Component
public class CascadeTestRunner implements ApplicationRunner {
    @PersistenceContext
    EntityManager entityManager;

    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Parent parent = new Parent();
        parent.setName("부모님");

        Child childA = new Child();
        childA.setName("자식A");
        Child childB = new Child();
        childB.setName("자식B");
        
        parent.addChild(childA);
        parent.addChild(childB);

        entityManager.persist(parent); // 부모만 저장된다.
    }
}
```

같은 코드로 child까지 저장되게 하기 위해서는 Parent 클래스에서 관계설정을 변경한다. Parent가 저장(PERSIST)될 때 관계가 맵핑된 Child도 같이 저장(PERSIST)해 달라는 의미이다.

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
List<Child> children = new ArrayList<>();
```

cascade가 가질 수 있는 속성은 CascadeType enum으로 정의되어 있으며 저장할 때 사용할 CascadeType.PERSIST 외에 REMOVE, MERGE 등이 있지만 보통 CascadeType.ALL로 퉁쳐서 사용한다고 한다. (REMOVE의 경우 Parent가 삭제될 때 연관된 Child도 모두 같이 삭제되는 것으로 명칭을 통해 추측해볼 수 있다.)

## 고아객체

추가로 부모가 가진 List<Child> children 중 한 요소를 삭제하고 싶으면 어떻게 해야할까?  
부모의 children.remove(i)로 특정 요소를 삭제 후 저장한다고 자식의 요소가 DB에서 delete되지는 않는다.

부모가 가진 자식객체중 참조가 끊어진 객체를 고아객체라 하며, 고아객체를 자동으로 delete하게 해주는 옵션을 `orphanRemoval = true` 로 지정할 수 있다.

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
List<Child> children = new ArrayList<>();
```

```java
@Component
public class CascadeTestRunnerB implements ApplicationRunner {
    @PersistenceContext
    EntityManager entityManager;

    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Parent parent = entityManager.find(Parent.class, 1L);
        parent.getChildren().remove(1);
    }
}
```

위 코드에서는 EntityManager로 find() 한 시점에서 parent 객체에 대한 상태를 관리하기 시작한다. 이후 children의 한 요소를 remove할 때 고아객체가 만들어지면서 `orphanRemoval = true` 가 지정되어 있기 때문에 트랜잭션이 끝나는 시점에 해당 고아객체의 delete가 수행된다.