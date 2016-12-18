Dagger2와 상속
=============

![badge](https://img.shields.io/badge/manasobi-dagger2-brightgreen.svg?style=flat-square)

###### [http://blog.naver.com/akindo123/220608253129](http://blog.naver.com/akindo123/220608253129)

## Caee 1. 상속된 클래스의 Field Injection
Dagger 2의 Field Injection는 부모 Field의 의존까지 모두 충족시켜준다.

```java
public class SuperClass {
@Inject
@Getter
LoginApi loginApi;
}

public class SubClass extends SuperClass {
@Inject
@Getter
RestApi restApi;

@Inject
public SubClass() {
} 
}

public class MainActivity {
...
@Inject
SubClass injectedSubclass;

...
@Override
protected void onCreate(Bundle savedInstanceState) {
AppComponent component = DaggerAppComponent.builder().build();
component.inject(this);

LoginApi loginApi = injectedSubclass.getLoginApi();
RestApi restApi = injectedsubclass.getRestApi();
```

## Case 2. Construction Injection with Field Injection
서브 클래스를 Construction Injection한 경우에도 부모 Field의 의존까지 수행한다.

```java
public class SuperClass {
@Inject
@Getter
LoginApi loginApi;
}

public class SubClass extends SuperClass {
@Getter
RestApi restApi;

@Inject
public SubClass(RestApi restApi) {
this.restApi = restApi;
} 
}

public class MainActivity {
...
@Inject
SubClass injectedSubclass;

...
@Override
protected void onCreate(Bundle savedInstanceState) {
AppComponent component = DaggerAppComponent.builder().build();
component.inject(this);

LoginApi loginApi = injectedSubclass.getLoginApi();
RestApi restApi = injectedsubclass.getRestApi();
```

## Case 3. Member-Injection methods로 부모 클래스를 명시적으로 주입
부모 클래스의 자식 클래스의 @Inject 어노테이션된 Field는 Null 상태로 유지된다.

```java
@Component
public interface AppComponent {
void inject(SuperClass);
}

public class SuperClass {
@Inject
@Getter
LoginApi loginApi;

public SuperClass() {
AppComponent component = DaggerAppComponent.builder().build();
component.inject(this);
}
}

public class SubClass extends SuperClass {
@Inject
@Getter
RestApi restApi;

public SubClass() {
} 
}

public class MainActivity {
...
SubClass injectedSubclass;

...
@Override
protected void onCreate(Bundle savedInstanceState) {
SubClass subClass = new SubClass();

LoginApi loginApi = injectedSubclass.getLoginApi();
RestApi restApi = injectedsubclass.getRestApi(); <-- 여기서 NPE 발생
```