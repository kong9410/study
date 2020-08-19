# `@VisibleForTesting`

가시성이 테스트를 위해 완화되었음



테스트 코드로 테스트를 하려는 클래스의 private 메소드에 접근할 수 없기에 private 메소드를 테스트할 수 없다. 이 경우 테스트를 하는 방법이 세가지가 있다.

1. 테스트 클래스에서 접근할 수 있게 가시성을 완화한다.
2. 리플렉션을 이용해 접근한다.
3. 테스트를 하지 않는다. public 단위로 테스트를 한다.



1번의 경우는 캡슐화 원칙에 위배되지만 `@VisibleForTesting` 어노테이션을 붙여 테스트 코드에서만 호출 할 수 있도록 나타낼 수 있다.

예제 코드)

```java
class MyService {
    private void process() {
        /* process something... */
    }
}
```

```java
class MyServiceTest {
    @Test
    public void testProcess() {
        MyService myService = new MyService();
        myService.process();
        /* ... */
    }
}
```