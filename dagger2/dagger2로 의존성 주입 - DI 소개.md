dagger2로 의존성 주입 - DI 소개
=============

![badge](https://img.shields.io/badge/manasobi-dagger2-brightgreen.svg?style=flat-square)

###### [http://blog.naver.com/akindo123/220600558841](http://blog.naver.com/akindo123/220600558841)

며칠 전 나(Mirosław Stanek)는 Crawcow의 Tech Space의 Google I/O 2015에서, Dagger 2를 이용한 `의존성 주입`에 대한 프레젠테이션을 했었다. 준비 시간 동안 이야기해야 많을 것들을 한 다스의 슬라이드에 모두 다룰 수는 없다는 것을 알게되었다. 하지만 그건 새로운 포스트 시리즈를 시작하기에 좋은 시작점이 될 것이다. - **안드로이드에서 의존성 주입**

![alt](http://dthumb.phinf.naver.net/?src=%22https%3A%2F%2Fspeakerd.s3.amazonaws.com%2Fpresentations%2F4b55a6f3efac405d9209aa731c9a74c9%2Fslide_0.jpg%22&type=ff500_300)

이 포스트에서 나는 나의 프레젠테이션을 요약하며 살펴 볼 것이다. 단계적이지는 않다 -내 생각에는 이제는 과거와 단절할 시간이며 하지 말아야 할 것들과 사용하지 않을 것들로 돌아가지 않을 것이다. Jake Wharton은 과거(Guice, Dagger1)에 대해 이야기했었고, Gregory Kick도 마찬가지이다. 나 역시 이전의 솔루션에 대해 몇 분간 이야기했었다. 하지만 이제는 지금 우리가 존재하는 곳에서 시작할 시간이다.

## 의존성 주입 (Dependency injection)
 의존성 주입은 객체들을 생성하고 그것들을 필요한 곳에 전달해주는 것이 전부이다. 의존성 주입의 이론에 대해서는 깊이 들어가지 않을 것이다 (이에 대해서는 Wikipedia의 DI definition를 방문하라). 대신 간단한 클래스를 상상해보자: 두 개의 의존관계들 (UserStore와 ApiService)를 가진 UserManager. 의존성 주입이 없으면 이 클래스는 다음과 같이 보일 것이다.

```java
class UserManager {

  private ApiService apiService;
  private UserStore userStore;

//No-args constructor. Dependencies are created inside.
  public UserManager() {
    this.apiService = new ApiSerivce();
    this.userStore = new UserStore();
  }

  void registerUser() {/* */}
}

class RegisterActivity extends Activity {

  private UserManager userManager;

  @Override
  protected void onCreate(Bundle b) {
    super.onCreate(b);
    this.userManager = new UserManager();
  }

  public void onRegisterClick(View v) {
    userManager.registerUser();
  }
}
```
[github 주소](https://gist.github.com/frogermcs/907485126fcb2bda8978#file-usermanagernodi-java)

이 코드에 있는 약간의 문제는 무엇일까? 다음을 상상해보자. 당신은 UserStore 구현을 변경하여 SharedPreferences를 저장 메커니즘으로 사용하고 싶다. 그것은 인스턴스를 만들기 위해 최소한 Context 객체가 필요하므로 생성자를 통해 UserStore에게 전달되어야 한다. 이는 UserManager 클래스 역시 새로운 UserStore 생성자를 다루기 위해 수정되어야 한다는 것을 의미한다. 이제 UserStore를 사용하는 한 다스의 클래스들을 상상하라 - 그들 모두가 수정되어야 한다.

이제 의존성 주입을 사용하는 UserManager를 보자.

![alt](http://postfiles14.naver.net/20160118_125/akindo123_14530504372058djSv_PNG/user_manager_di.png?type=w773)

그것의 의존관계들은 클래스 밖에서 생성되고 제공된다.

```java
class UserManager {

  private ApiService apiService;
  private UserStore userStore;

  //Dependencies are passed as arguments
  public UserManager(ApiService apiService, UserStore userStore) {
    this.apiService = apiService;
    this.userStore = userStore;
  }

  void registerUser() {/* */}
}

class RegisterActivity extends Activity {

  private UserManager userManager;

  @Override
  protected void onCreate(Bundle b) {
    super.onCreate(b);
    ApiService api = ApiService.getInstance();
    UserStore store = UserStore.getInstance();

    this.userManager = new UserManager(api, store);
  }

  public void onRegisterClick(View v) {
    userManager.registerUser();
  }
}
```

이제 동일한 상황에서 `의존들 중 하나의 구현을 변경한다.` UserManager 소스 코드가 수정될 필요가 없다. 그것의 모든 의존들은 외부에서 제공되므로 수정되어야 할 곳은 우리가 UserStore 객체를 생성하는 곳뿐이다.

이제 의존성 주입 사용의 장점은 무엇일까?

#### 생성/사용 분리 (Construction/usage seperation)
우리는 객체들의 인스턴스를 한 번만 생성할 것이다 - 대게 이 객체들이 사용되는 곳과 다른 곳에서. 이 접근법 덕분에 우리의 코드는 훨씬 모듈화된다 - 모든 의존들은 (동일한 인터페이스를 가지고 있는 한) 우리 애플리케이션의 로직에 영향을 주지 않고 쉽게 교체될 수 있다. 

DataUserStore를 SharedPrefsUserStore로 변경하고 싶은가?  Fine, just take care about public API (to be the same as DatabaseUserStore) or just implement the same interface.

#### Unit testing

본래의 유닛 테스트는 클래스가 완전히 격리(isolation) 된 상태에서 테스트될 것으로 가정한다 -그것의 의존에 대한 지식 없이. 실제, 우리의 UserManager 클래스에 대해 우리가 작성해야 할 유닛 테스트의 예제가 여기 있다.

```java
public class UserManagerTests {

UserManager userManager;

@Mock
ApiService apiServiceMock;
@Mock
UserStore userStoreMock;

@Before
public void setUp() {
  MockitoAnnotations.initMocks(this);
  userManager = new UserManager(apiServiceMock, userStoreMock);
}

@After
  public void tearDown() {
}

@Test
public void testSomething() {
  //Test our userManager here - all its dependencies are satisfied
}
```

그리고 이는 DI를 통해서만 가능하다 -UserManager가 UserStore와 ApiManager의 구현들과는 완전히 독립적인 것 덕분이다. 우리는 이들 클래스들의 mock들을 제공할 수 있고 (in short - mocks are classes with the same public API which does nothing in method calls and/or returns values which we expect)  UserManager를 그것의 의존들의 실제 구현과 분리하여 테스트할 수 있다.

#### Independend/concurrent development

코드 모듈화 덕분에 (UserStore는 UserManager와 독립적으로 구현될 수 있다) 프로그래머들 간에 코드를 분리하기가 쉽다. UserStore의 인터페이스만 모두가 알면 된다(특히 UserManager에서 사용하는 UserStore의 public 메서드들). 나머지(구현체, 로직)은 unit test를 통해 테스트될 수 있다.

## Dependency injection frameworks
의존성 주입 패턴은 장점 외에 약간의 단점도 있다. 그중 하나는 거대한 boilerplate이다. MVP(model-view-presenter) 패턴이 구현된 간단한 LoginAcitivity을 상상해보라. 이 클래스는 이렇게 생겼을 것이다.

![alt](http://postfiles2.naver.net/20160118_273/akindo123_1453051317663dgNrl_PNG/login_activity_diagram.png?type=w773)

LoginActivityPresenter 초기화에 대한 코드는 아래와 같을 것이다

```java
public class LoginActivity extends AppCompatActivity {

  LoginActivityPresenter presenter;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    OkHttpClient okHttpClient = new OkHttpClient();
    RestAdapter.Builder builder = new RestAdapter.Builder();
    builder.setClient(new OkClient(okHttpClient));
    RestAdapter restAdapter = builder.build();
    ApiService apiService = restAdapter.create(ApiService.class);
    UserManager userManager = UserManager.getInstance(apiService);

    UserDataStore userDataStore = UserDataStore.getInstance(
      getSharedPreferences("prefs", MODE_PRIVATE)
    );

    //Presenter is initialized here
    presenter = new LoginActivityPresenter(this, userManager, userDataStore);
  }
}
```

친절해 보이지는 않는다. 그렇지 않은가?
그리고 이 문제를 DI 프래임웍들이 해결해준다. 그것들을 사용하는 동일한 코드는 이렇게 생겼을 것이다

```java
public class LoginActivity extends AppCompatActivity {

  @Inject
  LoginActivityPresenter presenter;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //Satisfy all dependencies requested by @Inject annotation
    getDependenciesGraph().inject(this);
  }
}
```

훨씬 간단하지 않는가? 물론 DI 프레임웍은 아무것도 없는 곳에서 객체들을 가져가지 않는다 - 그들은 여전히 우리의 코드 어딘가에서 초기화되고 되어야 한다. 하지만 객체의 생성과 사용은 분리된다 (사실 이것은 DI 패턴의 전제이다). 그리고 DI 프래임웍은 어떻게 모든 것을 함께 연결할지를 감독한다 (어떻게 객체들을 그들이 요청된 곳으로 전달할지)

## To be continued
내가 설명한 모든 것은 Dagger 2(안드로이드와 자바 개발에서 사용할 수 있는 의존성 주입 프레임웍)에 대한 간단한 배경이다. 다음 포스트에서 Dagger 2 API 전체를 살펴 보려 노력할 것이다.  In case you don’t want to wait just try my [Github client example](https://github.com/frogermcs/GithubClient) which is built on top of Dagger 2 and was used with my presentation. Just a hint - @Modules and @Components are the places to construct/provide objects. @Inject are the places where our objects are used.
More detailed description - soon.

원본 :[http://frogermcs.github.io/dependency-injection-with-dagger-2-introdution-to-di/](http://frogermcs.github.io/dependency-injection-with-dagger-2-introdution-to-di/)

















