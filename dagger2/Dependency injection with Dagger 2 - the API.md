Dependency injection with Dagger 2 - the API
=============

![badge](https://img.shields.io/badge/manasobi-dagger2-brightgreen.svg?style=flat-square)

###### [http://blog.naver.com/akindo123/220602731674](http://blog.naver.com/akindo123/220602731674)

이 포스트는 Dagger2를 이용한 Android의 의존성 주입을 보여주는 포스트 시리즈 중 하나이다. 이번에는 Dagger2의 기초와 이 의존성 주입 프레임워크의 전체 API를 살펴보겠다.

# Dagger 2
이전 포스트에서 DI 프레임웍은 가능한 적은 양의 코드로 모든 것이 함께 연결 될 수 있도록 해준다고 이야기했었다. Dagger 2는 우리를 위해 많은 양의 boilerplate 코드를 생성해주는 DI 프레임워크의 예들 하나이다. 그렇다면 왜 그것이 다른 것들보다 최고인가? 지금 그것은 사용자가 손으로 작성한 것처럼 보이는 완전히 추적 가능한 소스 코드를 생성하는 유일한 DI 프레임워크이다. Dagger 2는 다른 것들 보다 덜 동적이다(리플랙션이 전혀 사용되지 않는다). 하지만 생성된 코드의 간결함과 성능은 손으로 작성한 코드와 동일한 수준이다.

## Dagger 2 기초

```java
public @interface Component {
  Class[] modules() default {};
  Class[] dependencies() default {};
}

public @interface Subcomponent {
  Class[] modules() default {};
}

public @interface Module {
  Class[] includes() default {};
}

public @interface Provides {
}

public @interface MapKey {
  boolean unwrapValue() default true;
}

public interface Lazy {
  T get();
}
```

역시 여기에 JSR-330(자바 표준 의존성 주입)에서 정의되었고 Dagger2에서 사용되는 다른 요소들이 있다.

```java
public @interface Inject {
}

public @interface Scope {
}

public @interface Qualifier {
}
```

이제 이것들은 모두 살펴보자.

## @Inject annotation
처음 그리고 DI에서 가장 중요한 것은 @Inject이다. JSR-330 표준의 구성원이며, 의존성 주입 프레임웍에 의해 주입되어야 하는 의존들에 마크한다. Dagger 2에서는 의존들을 주입하는 3가지 다른 방법들이 있다.

#### 생성자 주입
클래스 생성자에 @inject가 사용된다.

```java
public class LoginActivityPresenter {

private LoginActivity loginActivity;
private UserDataStore userDataStore;
private UserManager userManager;

  @Inject
  public LoginActivityPresenter(LoginActivity loginActivity, UserDataStore userDataStore,
    UserManager userManager) {
    this.loginActivity = loginActivity;
    this.userDataStore = userDataStore;
    this.userManager = userManager;
  }
}
```

모든 파라미터들은 의존성 그래프(dependencies graph)에서 나온다. 생성자에서 사용된 @inject 어노테이션은 이 클래스를 의존성 그래프의 일부로 만든다. 이는 그것이 필요할 때마다 주입될 수 있다는 것을 의미한다. 즉

```java
public class LoginActivity extends BaseActivity {

  @Inject
  LoginActivityPresenter presenter;
//...
}
```

이 경우의 클래스 내에는 하나 이상의 생성자에 @Inject 어노테이트 할 수 없는 제약이 있다.

#### 필드 주입 (Fields injection)
다른 옵션은 특정 필드들을 @Inject로 어노테이트 하는 것이다.

```java
public class SplashActivity extends AppCompatActivity {

  @Inject
  LoginActivityPresenter presenter;
  @Inject
  AnalyticsManager analyticsManager;

  @Override
  protected void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    getAppComponent().inject(this);
  }
}
```

하지만 이 경우 주입 절차는 클래스의 어느 부분에서 "직접" 호출되어야 한다.

```java
public class SplashActivity extends AppCompatActivity {
//...
  @Override 
  protected void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    getAppComponent().inject(this); //요청한 의존들은 이 순간 주입된다.
  }
}
```

이 호출 이전에는 우리의 의존들은 null 값이다.
필드 주입의 한계는 그들은 private이 될 수 없다는 것이다. 왜? 짧게 말해, 생성된 코드는 필드들을 명시적으로 호출하여 채우기 때문이다. 다음처럼:

```java
//이 클래스는 Dagger2에 의해 자동적으로 생성되었다.
public final class SplashActivity_MembersInjector implements MembersInjector {
//...
  @Override
  public void injectMembers(SplashActivity splashActivity) {
    if (splashActivity == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    supertypeInjector.injectMembers(splashActivity);
    splashActivity.presenter = presenterProvider.get();
    splashActivity.analyticsManager = analyticsManagerProvider.get();
  }
}
```

#### 메서드 주입 (Methods injection)
@Inject로 의존을 제공하는 마지막 방법은 이 클래스의 public 메서드를 어노테이트하는 것이다.

```java
public class LoginActivityPresenter {

  private LoginActivity loginActivity;

  @Inject 
  public LoginActivityPresenter(LoginActivity loginActivity) {
    this.loginActivity = loginActivity;
  }

  @Inject
  public void enableWatches(Watches watches) {
    //Watches 인스턴스는 완전히 생성된 LoginActivityPresenter를 요구하였다.
    watches.register(this); 
  }
}
```

모든 메서드 파라미터들은 의존성 그래프에서 제공된다. 하지만 왜 우리가 메서드 주입을 필요로 할까? 우리가 주입된 의존에 자신의 클래스 인스턴스를 전달할 수 있게 한다. 메서드 주입은 생성자 호출 뒤 즉시 호출되며, 이는 우리가 완전히 생성된 this를 전달함을 의미한다. (필드 주입의 경우에는 필드 주입을 명시적으로 호출하여 주입이 끝난 뒤 호출 될 거이다. 방금 예를 든 상황에도 여러 활용법이 있을 것이다.) 

## @Module annotation
**@Module**은 Dagger 2 API의 구성원이다. 이 어노테이션은 의존들을 주입하는 클래스를 표시할 때 사용된다 - 이것 덕분에 Dagger는 요청받은 객체가 생성되는 위치를 알게 될 것이다.

```java
@Module
public class GithubApiModule {

  @Provides
  @Singleton
  OkHttpClient provideOkHttpClient() {
    OkHttpClient okHttpClient = new OkHttpClient();
    okHttpClient.setConnectTimeout(60 * 1000, TimeUnit.MILLISECONDS);
    okHttpClient.setReadTimeout(60 * 1000, TimeUnit.MILLISECONDS);
    return okHttpClient;
  }

  @Provides
  @Singleton
  RestAdapter provideRestAdapter(Application application, 
      OkHttpClient okHttpClient) {
    RestAdapter.Builder builder = new RestAdapter.Builder();
    builder.setClient(new OkClient(okHttpClient))
    .setEndpoint(application.getString(R.string.endpoint));
    return builder.build();
  }
}
```

## @Provides annotation
이 어노테이션은 @Module 클래스에서 사용된다. @Provides는 모듈 안의 의존을 반환하는 메서드들을 표시할 것이다.

```java
@Module
public class GithubApiModule {
//...
  @Provides //This annotation means that method below provides dependency
  @Singleton
  RestAdapter provideRestAdapter(Application application, 
      OkHttpClient okHttpClient) {
    RestAdapter.Builder builder = new RestAdapter.Builder();
    builder.setClient(new OkClient(okHttpClient))
    .setEndpoint(application.getString(R.string.endpoint));
    return builder.build();
  }
}
```

## @Component annotation
이 어노테이션은 모든 것이 함께 연결되게 하는 인터페이스를 만들기 위해 사용된다. 이곳에 우리가 의존을 가져오는 모듈들 (또는 다른 Component들)을 명시한다. 

Also here is the place to define which graph dependencies should be visible publicly (can be injected) and where our component can inject objects. 

@Component은 @Module과 @Inject 사이의 다리 같은 것이다. 두 개의 모듈들을 사용하며, GithubClientApplictaion에 의존을 주입할 수 있고, 세 개의 의존을 공개적으로 보이게(visible publicly) 한 @Component에 대한 예제 코드가 여기 있다.

```java
@Singleton
@Component(
  modules = {AppModule.class, GithubApiModule.class}
)
public interface AppComponent {
  void inject(GithubClientApplication githubClientApplication);
  Application getApplication();
  AnalyticsManager getAnalyticsManager();
  UserManager getUserManager();
}
```

@Component 역시 다른 component에 의존할 수 있으며, 명시적인 라이프 사이클을 가진다 (다음 포스트에서 이에 대해 작성할 것이다.)

```java
@ActivityScope
@Component( 
  modules = SplashActivityModule.class,
  dependencies = AppComponent.class
)
public interface SplashActivityComponent {
  SplashActivity inject(SplashActivity splashActivity);
  SplashActivityPresenter presenter();
}

```

## @Scope annotation

```java
@Scope
public @interface ActivityScope {
}
```

JSR-330 표준의 또 다른 구성원이다. Dagger 2에서 @Scope는 커스텀 범위(scope) 어노테이션을 만들 때 사용된다. 요약하면 그것은 의존을 싱글톤과 매우 비슷한 것으로 만든다. 어노테이트된 의존들은 단일 인스턴스이지만 이는 component의 생명 주기(전체 애플리케이션이 아니라)와 관련 있다. 하지만 내가 앞에서 말했듯이 -우리는 범위에 대해서는 다음 포스트에서 다룰 것이다. 지금 말해둘 만한 것은 모든 커스텀 범위는 (코드 관점에서) 동일한 것을 한다는 것이다- 그것은 객체의 단일 인스턴스를 유지한다. 하지만 그래프 구조적 문제를 가능한 한 빨리 잡아내기 위해 도움이 되는 그래프 확인(validation) 과정에서도 사용된다.

## @MapKey
이 어노테이션은 의존들의 컬렉션을 명시하는 데에 사용된다 (우선은 Map과 Set). 예제 코드는 자명할 것이다.

#### Definition
``` java
@MapKey(unwrapValue = true)
@interface TestKey {
  String value();
}
```

#### Providing dependencies
```java
@Provides(type = Type.MAP)
@TestKey("foo")
String provideFooKey() {
  return "foo value";
}

@Provides(type = Type.MAP)
@TestKey("bar")
String provideBarKey() {
  return "bar value";
}
```

#### Usage
```java
@Inject
Map map;

map.toString() // => „{foo=foo value, bar=bar value}”
```

@MapKey 어노테이션은 우선은 단 두 가지 타입의 키(key)만 지원한다 - String과 Enum

## @Qualifier

@Qualifier 어노테이션은 동일한 인터페이스를 가진 의존을 위한 태그(tags)를 만드는데 도움을 준다. 당신이 두 개의 RestAdapter 객체를 필요로 한다고 생각해보라 - 하나는 Github API를 위한 것이고 다른 것은 facebook API를 위한 것이다. Qualifier는 적절한 것을 식별하는데 도움이 될 것이다.

#### Naming dependencies
```java
@Provides
@Singleton
@GithubRestAdapter //Qualifier
RestAdapter provideRestAdapter() {
  return new RestAdapter.Builder()
    .setEndpoint("https://api.github.com")
    .build();
}

@Provides
@Singleton
@FacebookRestAdapter //Qualifier
RestAdapter provideRestAdapter() {
return new RestAdapter.Builder()
.setEndpoint("https://api.facebook.com")
.build();
}
```

#### Injecting dependencies
```java
@Inject
@GithubRestAdapter
RestAdapter githubRestAdapter;

@Inject
@FacebookRestAdapter
RestAdapter facebookRestAdapter;
view raw
```

# App example
이제 실제에서 우리의 지식을 체크할 시간이다. 우리는 Dagger 2로 만들어진 간단한 Github 클라이언트 앱을 구현할 것이다.

## The idea
우리의 Github 클라이언트는 세 개의 액티비티들과 매우 간단한 유즈 케이스를 가지고 있다. 그것의 전체 흐름은 :
- Github 유저 이름을 입력한다.
- 만약 유저가 존재한다면 공개 리파지토리들의 전체 목록을 보여준다.
- 사용자가 리스트 아이템을 누르면 리파지토리 상세를 보여준다.

우리 앱은 다음과 같이 보일 것이다.

![alt](http://postfiles10.naver.net/20160120_73/akindo123_1453215897315C1pNG_PNG/login_activity_diagram.png?type=w773)

내부에서는, DI 관점에서 우리의 앱은 다음과 같이 보일 것이다.

![alt](http://postfiles15.naver.net/20160120_254/akindo123_1453215897493Xr6iy_PNG/local_components.png?type=w773)

간단히 말해서 - 각 액티비티는 자신의 의존성 그래프를 가진다. 각 그래프 (_Component 클래스)는 두 개의 객체를 가진다 - _Presenter와 _Activity. 역시 각 component는 global component에 대한 의존을 가진다 - AppComponent, which contains among others Application, UserManager, RepositoriesManager etc.

![alt](http://postfiles14.naver.net/20160120_29/akindo123_1453215946334Lkf04_PNG/app_component.png?type=w773)

AppComponent에 대해 이야기하자면 - 그것은 두 개의 모듈을 가진다 : AppModule과 GithubApiModule.
GithubApiModule은 이 모듈 안의 다른 의존들에서만 사용되는 OkHttpClient 또는 RestAdapter같은 의존을 제공한다. Dagger 2에서는 어떤 객체가 component의 외부에서 보일 수 있을지를 제어할 수 있다. 이 경우에는 방금 언급한 객체들은 공개하지 않을 것이다. 대신 우리는 UserManager와 RepositoriesManager를 공개한다. 이 객체들만이 우리의 Activity 들에서 사용되기 때문이다. 모두가 void가 아닌 타입을 반환하며 파라미터가 없는 public 메서드에 의해 명시된다.

#### Provision methods
```java
SomeType getSomeType();
Set getSomeTypes();
@PortNumber int getPortNumber();
```

더욱이 우리는 우리가 (멤버-주입을 통해) 의존들을 주입하길 원하는 곳을 명시해야 한다. AppComponent의 경우 어디에서도 주입되지 않는다. 이는 우리가 그것을 단지 scoped component들의 의존으로서만 사용하였기 때문이다. 그리고 그들 각각은 명시된 inject(_Activity activity) 메서드를 가진다. 그리고 여기에 간단한 룰이 있다 - 주입은 단일 파라미터(이는 우리가 우리의 의존을 주입하길 원하는 인스턴스를 명시한다)를 가진 메서드에 의해 명시된다. 여기서 이름은 상관이 없다. 하지만 반환은 void이거나 넘겨주었던 파라미터의 타입이어야 한다.

#### Members-injection methods
```java
SomeType getSomeType();
Provider getSomeTypeProvider();
Lazy getLazySomeType();
```

# Implementation
더 이상 소스 코드를 파고들지 않을 것이다. 대신 [GithubClient](https://github.com/frogermcs/GithubClient) 코드를 클론하고 최신 Android Studio에 import 하라. 어디서 시작을 해야 할지 힌트를 주자면:

## Dagger 2 installation
지금 /app/build.gradle 파일을 체크하라. 우리가 해야 할 것은 Dagger 2 의존들을 추가하고 생성된 코드와 Android Studio IDE를 결속시키기 위해 android-apt 플러그인을 사용하는 것이다.

## AppComponent
GithubClientApplication 클래스에서부터 GithubClient를 탐색하라. 여기서 AppComponent가 생성되고 저장된다. 이것은 모든 단일-인스턴스 객체들은 Application 객체가 존재하는 한 존재할 것이다(그래서 항상).
AppComponent 구현은 Dagger 2에 의해 생성된 코드에서 주입된다 (객체는 builder 패턴에서 생성될 수 있다 : Dagger{ComponentName}.builder()). 그리고 이것은 모든 Component들의 의존들을 놓아야 하는 곳이다(modules과 다른 components)

## Scoped components
지금 어떻게 Activity component가 생성되는지를 확인하기 위해서 SplashActivity부터 탐색하라. 그것은 자신만의 Component(SplashActivityComponent)를 생성하고 모든 @Inject 어노테이션된 의존들(이 경우 SplashActivityPresenter와 AnalyticsManager)을 주입하는 곳인 setupActivityComponent(AppComponent)를 오버라이드 한다.
그리고 여기서 우리는 AppComponent 인스턴스(SplashActivityComponent가 그것에 의존하기 때문이다)와 (Presenter와 Activity instance를 제공하는) SplashActivityModule를 제공한다.
나머지는 당신에게 달려있다. 진지하게 모든 것이 어떻게 맞아떨어지는지 이해하려 해보아라. 그리고 다음 포스트에서 우리는 Dagger 2의 요소들에 대해 더 자세히 (내부에서는 어떻게 동작하는지) 알아보도록 할 것이다.




























