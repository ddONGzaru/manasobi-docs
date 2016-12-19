CleanArchitecture+ModelView(MVVM)+DataBinding에 대한 고찰
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [MuuKojima Edited at 2016-11-27](http://qiita.com/MuuKojima/items/5d20a1b6b558cb432010)

기본적으로는 저번에 작성한 글에 CleanArchitecture와 ModelView의 구성을 적용해보았다.

> MVP + Dagger2 + Retrofit2 + RxAndroidの構成で通信を試してみる
> [http://qiita.com/MuuKojima/items/8843c9451339a8b68f22](http://qiita.com/MuuKojima/items/8843c9451339a8b68f22)

이번 회 내용은 달러를 원화로 환율에 맞게 바꾸어 TextView에 뿌려주는 것이 전부이다.

> 使用API: [http://fixer.io/](http://fixer.io/)

실행결과

![alt](https://qiita-image-store.s3.amazonaws.com/0/33553/21fd96ad-2332-ab3f-3357-89002e0564f1.png)

아래와 같은 데이터 1건이 요청에 의해 반환된다.
```json
{
base: "USD",
date: "2016-11-21",
rates: {
        JPY: 110.61
    }
}
```

#### 패키지 구성
|패키지명|설명|
|---|---|
|data|모델 부분을 구성|
|ui|activity, view, presenter, viewModel등으로 구성|
|di|dagger에 필요한 module, component등으로 구성|
|binding|DataBinding에서 사용하는 CallBack등으로 구성|

`패키지 구성은 레이어로 구분짓지 않고, 특징으로 구분지음.`

API로부터의 반환 값을 한행 표시하는것 뿐인데, 굉장히 많은 클라스가 필요하다. 어떻든지 클래스 수가 많이 필요한 것이 CleanArchitecture의 demerit라고 할 수 있겠다.

```java
|- binding
|   |- OnFieldChangedCallback
|   
|- data
|   |- exchange
|   |   |- repository
|   |   |   |- ExchangeRateApi
|   |   |   |- ExchangeRateRepositoryImpl
|   |   |   |- ExchangeRepositoryModule
|   |   |   |
|   |   |- usecase
|   |   |   |- GetExchangeRate
|   |   |   |
|   |   |- CountryCode
|   |   |- ExchangeRateRepository
|   |   |- ExchangeRateResponse
|   |   
|   |- PostExecutionThread
|   |- UseCase
|   
|- di
|   |- ApiModule
|   |- AppComponent
|   |- JobExecutor
|   |- PerActivity
|   |- RepositoryModule
|   |- UIThread
|   
|- ui
|   |- exchange
|       |-ExchangeRateActivity
|       |-ExchangeRateComponent
|       |-ExchangeRatePresenter
|       |-ExchangeRateView
|       |-ExchangeRateViewModel
|
|- AppApplication
```

## CleanArchitecture 기본 룰(한방향에서의 액세스)

![alt](https://qiita-image-store.s3.amazonaws.com/0/33553/74d18a4b-0cbf-daf6-bbe3-cf1408c5ae5a.png)

※참조 [http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/)

이 룰에 의해 이번에는 아래의 순으로 액세스해 나간다.
```
ExchangeRateActivity(View)
↓
ExchangeRatePresenter(Presenter)
↓
GetExchangeRate(UseCase)
↓
ExchangeRateRepository(Repository)
↓
ExchangeRateApi(Cloud)
```

DB(Disk)는 이번에 사용하지않지만, Repository에 추가하면 특별히 수정을 거치지 않고도 사용할 수 있고, API에서 취득한 값을 사용할지, DB로 부터 취득한 값을 사용할지는 상위 클래스들은 모른체 추상화 시킬 수 있다.

## Repository
`exchange` 패키지 내에 `repository` 패키지를 작성. `repository`는 데이터의 CRUD 조작을 제공하는 클래스.

#### ExchangeRateApi
```java
public interface ExchangeRateApi {
    String URL = "/latest";

    @GET(URL)
    Observable<ExchangeRateResponse> getExchangeRate(
        @Query("base") String base, @Query("symbols") String symbols);
}
```

`ExchangeRateApi`를 사용하기 위한 인터페이스 `ExchangeRateRepository`를 작성.
```java
public interface ExchangeRateRepository {
    /**
     * 為替レートを取得
     *
     * @param base
     * @param symbols
     * @return
     */
    Observable<ExchangeRateResponse> getExchangeRate(String base, String symbols);
}
```

`ExchangeRateApi`를 실제 사용하는 구현 클래스의 `ExchangeRateRepositoryImpl`을 작성. 이번에는 API 통신이지만, DB로부터 값을 취득하는 경우에도 여기에 구현.

#### ExchangeRateRepositoryImpl
```java
@Singleton
/* package */ class ExchangeRateRepositoryImpl implements ExchangeRateRepository {

    private final ExchangeRateApi mExchangeRateApi;

    @Inject
    public ExchangeRateRepositoryImpl(ExchangeRateApi mExchangeRateApi) {
        this.mExchangeRateApi = mExchangeRateApi;
    }

    @Override
    public Observable<ExchangeRateResponse> getExchangeRate(String base, String symbols) {
        return mExchangeRateApi.getExchangeRate(base, symbols);
    }
}
```

`ExchangeRateRepository`를 제공하는 `ExchangeRepositoryModule`을 작성. Module 클래스는 @Module을 붙이고 어미에 Module를 붙이는 것이 관례이다. 

#### ExchangeRepositoryModule
```java
@Module
public class ExchangeRepositoryModule {
    @Singleton
    @Provides
    public ExchangeRateRepository provideExchangeRateRepository(
        ExchangeRateRepositoryImpl exchangeRateRepository) {
        return exchangeRateRepository;
    }

    @Provides
    @Singleton
    public ExchangeRateApi provideExchangeRateApi(Retrofit retrofit) {
        return retrofit.create(ExchangeRateApi.class);
    }
}
```

Repository를 사용하기 위한 UseCase의 기저 클래스 `UseCase`를 작성.<br>
`subscribeOn()` 메인 이외의 쓰레드에서 실행<br>
`observeOn()` 메인 쓰레드를 지정<br>
이렇게 하는 것으로, 백그라운드 쓰레드에서 처리한 것을 메인 쓰레드에서 수신하는 것이 가능하다.

#### UseCase
```java
public abstract class UseCase<T> {
    protected final Executor threadExecutor;
    protected final PostExecutionThread postExecutionThread;

    protected UseCase(Executor threadExecutor, 
        PostExecutionThread postExecutionThread) {
        this.threadExecutor = threadExecutor;
        this.postExecutionThread = postExecutionThread;
    }

    protected Observable<T> bindUIThread(Observable<T> useCaseeObservable) {
        return useCaseeObservable
                .subscribeOn(Schedulers.from(threadExecutor))
                .observeOn(postExecutionThread.getScheduler());
    }
}
```

Schedule의 interface를 작성.

#### PostExecutionThread
```java
public interface PostExecutionThread {
    Scheduler getScheduler();
}
```

## UseCase
`exchange` 패키지 내에 `usecase` 패키지를 작성. `usecase`에는 비지니스 로직을 넣어둔다.<br>
`ExchangeRateRepository`을 사용하기 위한 유스케이스 `GetExchangeRate`을 추가.

여기까지해서 `data` 패키지의 소스들은 전부 완성.

#### GetExchangeRate
```java
public class GetExchangeRate extends UseCase<ExchangeRateResponse> {

    private final ExchangeRateRepository mExchangeRateRepository;

    @Inject
    protected GetExchangeRate(Executor threadExecutor, 
        PostExecutionThread postExecutionThread, 
        ExchangeRateRepository exchangeRateRepository) {
        super(threadExecutor, postExecutionThread);
        this.mExchangeRateRepository = exchangeRateRepository;
    }

    public Observable<ExchangeRateResponse> execute(
        String base, String symbols) {
        return bindUIThread(
            mExchangeRateRepository.getExchangeRate(base, symbols));
    }
}
```

통신결과를 메인 쓰레드에서 수신할 수 있도록 di 패키지를 작성하고 UIThread를 작성.

#### UIThread
```java
@Singleton
/* package */ class UIThread implements PostExecutionThread {

    @Inject
    public UIThread() {
    }

    @Override
    public Scheduler getScheduler() {
        // RxのメソッドでMainスレッドを取得することができる
        return AndroidSchedulers.mainThread();
    }
}
```

`Executor`를 계승한 `JobExecutor`를 작성. 아래 URL의 Executor 클래스를 그대로 복붙해서 사용했다.<br>
[https://github.com/android10/Android-CleanArchitecture/blob/master/data/src/main/java/com/fernandocejas/android10/sample/data/executor/JobExecutor.java](https://github.com/android10/Android-CleanArchitecture/blob/master/data/src/main/java/com/fernandocejas/android10/sample/data/executor/JobExecutor.java)

#### JobExecutor
```java
@Singleton
public class JobExecutor implements Executor {

    private static final int INITIAL_POOL_SIZE = 3;
    private static final int MAX_POOL_SIZE = 5;

    // Sets the amount of time an idle thread waits before terminating
    private static final int KEEP_ALIVE_TIME = 10;

    // Sets the Time Unit to seconds
    private static final TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;

    private final BlockingQueue<Runnable> workQueue;

    private final ThreadPoolExecutor threadPoolExecutor;

    private final ThreadFactory threadFactory;

    @Inject
    public JobExecutor() {
        this.workQueue = new LinkedBlockingQueue<>();
        this.threadFactory = new JobThreadFactory();
        this.threadPoolExecutor = 
            new ThreadPoolExecutor(INITIAL_POOL_SIZE, MAX_POOL_SIZE,
                KEEP_ALIVE_TIME, KEEP_ALIVE_TIME_UNIT, 
                this.workQueue, this.threadFactory);
    }

    @Override public void execute(Runnable runnable) {
        if (runnable == null) {
            throw new IllegalArgumentException("Runnable to execute cannot be null");
        }
        this.threadPoolExecutor.execute(runnable);
    }

    private static class JobThreadFactory implements ThreadFactory {
        private static final String THREAD_NAME = "android_";
        private int counter = 0;

        @Override public Thread newThread(Runnable runnable) {
            return new Thread(runnable, THREAD_NAME + counter++);
        }
    }
}
```

`ExchangeRepositoryModule`을 제공하는 `RepositoryModule`을 작성

#### RepositoryModule
```java
@Module(includes = {ExchangeRepositoryModule.class})
public class RepositoryModule {

    @Provides
    @Singleton
    public PostExecutionThread providePostExecutionThread(UIThread uiThread) {
        return uiThread;
    }

    @Provides
    @Singleton
    public Executor provideExecutor(JobExecutor jobExecutor) {
        return jobExecutor;
    }
}
```

`RepositoryModule`을 사용해서 통신하는 `ApiModule`을 작성

#### ApiModule
```java
@Module(includes = {RepositoryModule.class})
public class ApiModule {

    @Provides
    @Singleton
    Gson provideGson() {
        GsonBuilder gsonBuilder = new GsonBuilder();
        return gsonBuilder.create();
    }

    @Provides
    @Singleton
    OkHttpClient provideOkhttpClient() {
        OkHttpClient.Builder client = new OkHttpClient.Builder();
        client.addInterceptor(
            new HttpLoggingInterceptor().setLevel(
                    HttpLoggingInterceptor.Level.BODY));
        return client.build();
    }

    @Provides
    @Singleton
    Retrofit provideRetrofit(Gson gson, OkHttpClient okHttpClient) {
        return new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                // ベースのURLの設定
                .baseUrl("http://api.fixer.io")
                .client(okHttpClient)
                .build();
    }
}
```

모듈과 인터페이스를 합성한 `AppComponent` 작성.

#### AppComponent
```java
@Singleton
@Component(modules = {ApiModule.class})
public interface AppComponent {

    ExchangeRateRepository exchangeRateRepository();

    Executor executor();

    PostExecutionThread postExecutionThread();
}
```

Inject할 때 사용하는 커스텀 어노테이션을 작성. 스코프는 Activity가 살아있을때까지로 함.

#### PerActivity
```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {}
```

`Application`클래스를 상속한 `AppApplication`을 작성.<br>
`Activity`, `Fragment`등에서 `getAppComponet()`를 통해 `AppComponent`를 취득해, Inject함.

#### AppApplication
```java
public class AppApplication extends Application {

    private AppComponent mAppComponent;

    @Override
    public void onCreate() {
        super.onCreate();

        mAppComponent = DaggerAppComponent.builder().build();
    }

    public AppComponent getAppComponent() {
        return mAppComponent;
    }
}
```









