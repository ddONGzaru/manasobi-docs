Dependency injection with Dagger 2 - Custom scopes
=============

![badge](https://img.shields.io/badge/manasobi-dagger2-brightgreen.svg?style=flat-square)

###### [http://blog.naver.com/akindo123/220603377951](http://blog.naver.com/akindo123/220603377951)

이 포스트는 Dagger2를 이용한 Android의 의존성 주입을 보여주는 포스트 시리즈 중 하나이다. 이번에는 의존성 주입 초보자들에게 약간 의문적인 custom scope에 대한 시간을 보내도록 하겠다.

## Scope가 우리에게 제공하는 것은 무엇인가?
거의 모든 프로젝트는 싱글톤을 사용한다 - 예를 들면 API 클라이언트, database helpers, analytics manager등에서. 

하지만 우리는 인스턴스화 과정에 대해 다루지 않으므로 (의존성 주입 프레임워크이니까), 어떻게 우리 코드내의 저 객체들을 획득할지에 대해서는 생각하지 않을 것이다. 그 대신 @Inject 어노테이션이 바람직한 인스턴스들을 우리에게 제공할 것이다. 
Dagger 2 scope 메커니즘은 scope가 존재하는 동안 클래스의 단일 인스턴스를 유지한다. 

실전에서는 @ApplicationScope로 scope된 인스턴스들은 Applicatoin 객체가 살아있는 한 존재한다. @ActivityScope는 Activity가 존재하는 동안 참조를 유지한다(예를 들면 우리는 어떤 클래스의 동일한 인스턴스를 이 Activity안에 존재하는 모든 프래그먼트에서 공유할 수 있다.)
즉, scope는 scope 자신이 살아있는 동안 존재하는 "지역적 싱글톤"을 우리에게 제공한다.
확실하게 하자면 - @ActivityScope나 @ApplictionScope 어노테이션은 Dagger2에서 기본적으로 제공되지는 않는다. 그것은 가장 흔하게 사용되는 custom scope이다. 오직 @Singleton scope만이 기본적으로 사용가능하다 (Java 자체에서 제공된다)

## Scopes - 실전 예제
Dagger2의 scopes를 더 이해를 하기 위해, 실전 예제를 훑어보자. Application/Activity scope보다 더 복잡한 것을 구현할 것이다. 이를 위해 우리는 이전 포스트의 [GithubClient](https://github.com/frogermcs/GithubClient) 예제를 볼 것이다. 

우리의 앱은 세 가지 scope를 가지고 있을 것이다.

`@Sigleton`  애플리케이션 scope<br>
`@UserScope` 선택된 사용자와 관련된 클래스들의 scope (실제 앱에서는 그것은 로그인된 유저가 될 수도 있다)<br>
`@ActivityScope` Activity가 존재하는 동안 살아있는 인스턴스의 scope (우리의 경우에는 presenter들)

소개된 @UserScope이 이번 솔루션과 지난 포스트와의 주된 차별점이다. 사용자 경험 측면에서는 그것은 아무것도 제공하지 않는다. 
하지만 아키텍처적 관전에서 그것은 User 인스턴스를 의도된 파라미터로서 전달하지 않고도 우리에게 제공하게 한다. 

메서드들의 파라미터에서 사용자 데이터를 요구하는 클래스들 역시 (이 경우 RepositoriesManager) User 인스턴스를 생성자의 파라미터로 받을 수 있다(의존성 그래프를 통해 제공받을 것이다). 그리고 앱 실행 시간에 생성하는 대신 요구 받았을 때 (on demand) 초기화 될 것이다. 이것은 RepositoriesManager가 GitHub API를 통해 사용자를 받았을 때 생성된다는 것을 의미한다(RepositoriesListActivity가 존재하기 조금 전에).

이것은 우리 앱에서 존재하는 scope들과 컴포넌트들에 대한 간단한 시각화이다.

![alt](http://frogermcs.github.io/images/15/dagger-scopes.png)

싱글톤 (Application scope)는 가장 오래 살아있는 scope(실제로는 application존재하는 한)이다. Application scope의 서브 scope인 UserScope는 그것의 객체에 대한 접근권을 가진다(우리는 언제나 부모 scope에서 객체를 획득할 수 있다). ActivityScope도 동일하다(Activity 객체가 존재하는 한 존재한다.)- 그것은  객체들을 UserScope와 ApplicationScope에서 받을 수 있다.

## Example scopes lifecycle
우리 앱의 scope 라이프 사이클 예제가 여기 있다.

![alt](http://frogermcs.github.io/images/15/scopes-lifecycle.png)

싱글톤은 앱 실행 시점부터 모든 시간을 살아 있는 반면 UserScope는 우리가 Github API(실제 앱에서는 사용자가 로그인 한 뒤)를 통해 User 인스턴스를 받았을 때 생성되고 우리가 SplashActivity로 복귀했을 때 파괴된다 (실제 앱에서는 사용자가 로그아웃 한 뒤). 새로운 사용자를 선택하면 또다른 UserScope가 생성된다.
각 ActivityScope는 그것의 Acitivity 객체가 존재하는 동안 살아있는다.

## Implementation
Dagger2 scope 구현은 적절한 Component 설정을 통해 결정된다. 일반적으로 이를 위해서는 @Subcomponent 어노테이션 또는 Component 의존을 통한 두 가지 방법이 있다.
이들간의 주요 차이점은 객체 그래프 공유이다. `Subcomponent들은 그들 부모들의 전체 객체 그래프에 접근`할 수 있다. 반면 `Component 의존은 Component 인터페이스에 노출된 것들만 접근` 가능하게 한다. 

먼저 @Subcopmonent 어노테이션을 통한 방법을 보자. 만약 If you used Dagger 1 before it’s almost the same as creating a subgraphs from ObjectGraph. Moreover we’ll use similar naming convention for methods which create a subgraphs (but it’s not mandatory).

대신 두 개의 메소드를 추가하였다.
- UserComponent plus(UserModule userModule);
- SplashActivityComponent plus(SplashActivityModule splashActivityModule);

이것들은 우리의 AppComponent에 두 개의 subcomponent들을 만들 수 있다는 것을 의미한다 : UserComponent와 SplashActivityComponent. 그들은 AppComponent의 subcomponent이기 때문에 둘 다 AppModule과 GithubApiModule에 의해 제공되는 인스턴스들에 대한 접근을 가질 것이다.

이 메소드에 대한 명명 규칙은 : 반환 타입은 subcomponent 클래스, 임의의 메서드 이름, 파라미터들은 subcomponent가 필요로 하는 모듈들이다.

UserComponent가 (plus() 메서드의 파라미터로 전달되는) 다른 모듈을 필요로 하는 것을 볼 수 있다. 이렇게 하여 우리는 A새로운 모듈에 의해 제공되는 추가적 객체들로 AppComponent 그래프를 확장하게 된다. UserComponent 클래스는 아래와 같다.

```Java
@UserScope
@Subcomponent(
  modules = {UserModule.class}
)
public interface UserComponent {
  RepositoriesListActivityComponent plus(
      RepositoriesListActivityModule repositoriesListActivityModule);

  RepositoryDetailsActivityComponent plus(
      RepositoryDetailsActivityModule repositoryDetailsActivityModule);
}
```

물론 @UserScope 어노테이션은 우리가 만든 것이다.

```Java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface UserScope {
```

UserComponent에서 우리는 다른 두 subcomponent들을 만들 수 있다.<br>
RespositorieslistActivityComponent와 RepositoryDetailsActivityComponent.

그리고 scope 작업에서 가장 중요한 일이 여기서 일어난다. ApplicationComponent를 상속한 UserComponent에서 얻어지는 모든 인스턴스들은 (Application scope에서) 여전히 싱글톤이다. 하지만 (UserComponent의 일부인) UserModule에서 제공되는 것들은 UserComponent instance가 존재하는 동안만 살아있는 "지역적 싱글톤"이 될 것이다.
그래서 다음 호출로 우리가 다른 UserComponent 인스턴스를 만들 때마다 

`UserComponent appComponent = appComponent.plus(new UserModule(user))`

UserModule에서 획득되는 객체들은 다른 인스턴스들이다.

하지만 중요한 점은 이것이다 - 우리는 UserComponent의 생명주기에 대한 책임을 가진다. 그래서 우리는 그것의 초기화와 해제에 대해 신경써야 한다. 나는 우리 앱의 예제에서 이를 위한 두 개의 추가 메서드를 만들었다.

```Java
public class GithubClientApplication extends Application {

  private AppComponent appComponent;
  private UserComponent userComponent;
  //...
  public UserComponent createUserComponent(User user) {
    userComponent = appComponent.plus(new UserModule(user));
    return userComponent;
  }

  public void releaseUserComponent() {
    userComponent = null;
  }
  //...
}
```

createUserComponent()는 우리가 (SplashActivity에서) Github API를 통해 User 객체를 얻었을 때 호출된다. 그리고 우리가 RepositoiresListActivity에서 나올 때 releaseUserComponent()가 호출된다(이 시점에서 우리는 user scope가 전혀 필요가 없다).

## Scopes in Dagger 2 - under the hood
It’s always good to have a look under the hood how things work. Actually in this case to be sure that there is no magic in Dagger’s scopes mechanism.
We’ll start our investigation from UserModule.provideRepositoriesManager() method. It provides RepositoriesManagerinstance which should be scoped in @UserScope. Let’s check where this method is called (line 8).

```Java
@Generated("dagger.internal.codegen.ComponentProcessor")
public final class UserModule_ProvideRepositoriesManagerFactory 
  implements Factory<RepositoriesManager> {
  //...

  @Override
  public RepositoriesManager get() { 
    RepositoriesManager provided = module.provideRepositoriesManager(
        userProvider.get(), githubApiServiceProvider.get());
    if (provided == null) {
      throw new NullPointerException(
          "Cannot return null from a non-@Nullable @Provides method");
    }
    return provided;
  }

  public static Factory<RepositoriesManager> create(
      UserModule module, Provider<User> userProvider, 
      Provider<GithubApiService> githubApiServiceProvider) { 
    return new UserModule_ProvideRepositoriesManagerFactory(
        module, userProvider, githubApiServiceProvider);
  }
}
```

UserModule_ProvideRepositoriesManagerFactory is a just implementation of Factory pattern which getsRepositoriesManager instance from UserModule. We should dig further.
UserModule_ProvideRepositoriesManagerFactory is used in UserComponentImpl - implementation of our component (line 15).

```java
private final class UserComponentImpl implements UserComponent {
//...
  private UserComponentImpl(UserModule userModule) {
    if (userModule == null) {
      throw new NullPointerException();
    }
    this.userModule = userModule;
    initialize();
  }

  private void initialize() {
    this.provideUserProvider = 
      ScopedProvider.create(UserModule_ProvideUserFactory.create(userModule));
    this.provideRepositoriesManagerProvider = 
    ScopedProvider.create(
        UserModule_ProvideRepositoriesManagerFactory.create(
            userModule, provideUserProvider, 
            DaggerAppComponent.this.provideGithubApiServiceProvider));
  }
//...
}
```

provideRepositoriesManagerProvider object is responsible for providing RepositoriesManager instance every time when we request for it. As we can see provider is implemented by ScopedProvider. Take a look at a part of its source code:

```java
public final class ScopedProvider<T> implements Provider<T> {
//...
  private ScopedProvider(Factory<T> factory) {
    assert factory != null;
    this.factory = factory;
  }
  
  // cast only happens when result comes from the factory
  @SuppressWarnings("unchecked") 
  @Override
  public T get() {
    // double-check idiom from EJ2: Item 71
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          instance = result = factory.get();
        }
      }
    }
    return (T) result;
  }
//...
}
```

Could it be more simple? At the first call ScopedProvider takes instance from factory (UserModule_ProvideRepositoriesManagerFactory in our case) and stores this objet like Singleton pattern. And our scoped provider is just a field in UserComponentImpl so in short it means that ScopedProvider returns single instance as long as its Component exists.
Here you can find full implementation of ScopedProvider.
And that’s it. We’ve just figured out how scopes work under the hood in Dagger 2. And now we know that they are not connected with scopes annotations in any way. Custom annotations give us only a simple way to code validation in compile time and mark classes as single/non-single instances. All scoping stuff is connected to Component’s lifecycle.
And that’s all for today. I hope that scopes will become even simpler to use from now. Thanks for reading!

## Source code
Full source code of described project is available on [Github repository](https://github.com/frogermcs/GithubClient).


































