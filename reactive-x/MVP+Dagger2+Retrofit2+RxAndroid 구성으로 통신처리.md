MVP+Dagger2+Retrofit2+RxAndroid 구성으로 통신처리
=============

![badge](https://img.shields.io/badge/manasobi-RxJava-brightgreen.svg?style=flat-square) ![badge](https://img.shields.io/badge/manasobi-RxAndroid-yellowgreen.svg?style=flat-square)

###### Qiita [MuuKojima Edited at 2016-11-27](http://qiita.com/MuuKojima/items/8843c9451339a8b68f22)

기본적으로는 이전에 작성한 MVP의 구성을 적용한 것 뿐이다.
[http://qiita.com/MuuKojima/items/8088b43876dc8d3d1745](http://qiita.com/MuuKojima/items/8088b43876dc8d3d1745)

이번 회의 내용으로서는 USD을 JPY 환율로 바꾸어 수신 TextView에 표시하는 것 뿐이다. 사용하는 API [http://fixer.io/](http://fixer.io/)는 꽤 편히해 보인다.

#### 패키지 구성
MVP의 모델에 해당하는 부분은 `data`로 분리. View와 Presenter는 `exchange`라는 패키지 안에 함께 넣어두었다.

```java
├data├ApiModule
     ├AppComponent
     ├AppModule
     ├CountryCode
     ├ExchangeApi
     ├ExchangeRateResponse
├exchange├ExchangeRateContract
         ├ExchangeRateComponent
         ├ExchangeRateModule
         ├ExchangeRatePresenter
         ├MainActivity
├AppApplicatiinn
├PerActivity
```

#### rootのbuild.gradle
```java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.0'

       // 追加
       classpath 'com.uphyca.gradle:gradle-android-apt-plugin:0.9.4'
    }
}
```

#### appのbuild.gradle
```java
apply plugin: 'com.android.application'
// 追加 ↑の下に入れる事
apply plugin: 'android-apt'

dependencies {  

    //////// 追加 ////////

    // RxAndroid
    compile 'io.reactivex:rxjava:1.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.2'
    compile 'io.reactivex:rxandroid:1.1.0'
    // Dagger
    compile 'com.google.dagger:dagger:2.2'
    apt 'com.google.dagger:dagger-compiler:2.2'
    // Retrofit
    compile 'com.squareup.retrofit2:retrofit:2.0.2'
    // OkHttp
    compile 'com.squareup.okhttp3:okhttp:3.2.0'
    compile 'com.squareup.okhttp3:logging-interceptor:3.2.0'
    // Gson
    compile 'com.google.code.gson:gson:2.6.2'
    compile 'com.squareup.retrofit2:converter-gson:2.0.2'
}
```

#### AndroidManifest.xmlにインターネットパーミッションを1行追加
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="kojimation.com.retrofitsample">

    <!-- 追加 -->
    <uses-permission android:name="android.permission.INTERNET" />

</manifest>
```

#### ExchangeRateApi.java
```java
public interface ExchangeRateApi {
    String URL = "/latest";

    @GET(URL)
    Observable<ExchangeRateResponse> getExchangeRate(
        @Query("base") String base, @Query("symbols") String symbols);
}
```

#### ExchangeRateResponse.java
```java
public class ExchangeRateResponse {
    private String base;
    private String date;
    private CountryCode rates;

    public String getBase() {
        return base;
    }

    public String getDate() {
        return date;
    }

    public CountryCode getRates() {
        return rates;
    }
}
```

#### CountryCode.java
```java
public class CountryCode {
    private float JPY;

    public float getJPY() {
        return JPY;
    }
}
```

#### AppModuleを作成
```java
@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application mApplication) {
        this.mApplication = mApplication;
    }

    @Provides
    @Singleton
    Application provideApplication() {
        return mApplication;
    }
}
```

#### ApiModuleを作成
```java
@Module
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
            new HttpLoggingInterceptor()
                    .setLevel(HttpLoggingInterceptor.Level.BODY));
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

#### AppComponentを作成
```java
@Singleton
@Component(modules = {AppModule.class, ApiModule.class})
public interface AppComponent {
    Retrofit retrofit();
}
```

#### Applicationクラスを継承したAppApplicationを作成
```java
public class AppApplication extends Application {

    private AppComponent mAppComponent;

    @Override
    public void onCreate() {
        super.onCreate();

        mAppComponent = DaggerAppComponent.builder()
                .appModule(new AppModule(this))
                .apiModule(new ApiModule())
                .build();
    }

    public AppComponent getAppComponent() {
        return mAppComponent;
    }
}
```

#### ViewのinterfaceとPresenterのinterfaceまとめるExchangeRateContractを作成
```java
public interface ExchangeRateContract {
    interface View {
        void bindExchangeRate(ExchangeRateResponse exchangeRateResponse);
    }

    interface Presenter {
        void getExchangeRate();
    }
}
```

#### ExchangeRateContract.PresenterをimplementsしたExchangeRatePresenterを作成
```java
public class ExchangeRatePresenter implements ExchangeRateContract.Presenter {
  private Retrofit mRetrofit;
  private ExchangeRateContract.View mView;

  @Inject
  public ExchangeRatePresenter(Retrofit retrofit, 
    ExchangeRateContract.View view) {
      this.mRetrofit = retrofit;
      this.mView = view;
  }

  public void getExchangeRate() {
      mRetrofit.create(ExchangeRateApi.class)
               .getExchangeRate("USD", "JPY")
               .subscribeOn(Schedulers.newThread())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe(new Observer<ExchangeRateResponse>() {
                  @Override
                  public void onCompleted() {
                  }
                  @Override
                  public void onError(Throwable e) {
                      Log.d("通信 -> ", "失敗" + e.toString());
                  }
                  @Override
                  public void onNext(ExchangeRateResponse exchangeRateResponse) {
                      mView.bindExchangeRate(exchangeRateResponse);
                  }
              });
    }
}
```

#### ViewをModuleに登録するためのExchangeRateModuleを作成
```java
@Module
public class ExchangeRateModule {
    private final ExchangeRateContract.View mView;

    public ExchangeRateModule(ExchangeRateContract.View mView) {
        this.mView = mView;
    }

    @Provides
    ExchangeRateContract.View provideExchangeRateView() {
        return mView;
    }
}
```

#### Inject할때 사용하는 커스텀 어노테이션을 작성 (PerActivity)
```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {}
```

#### Activity等でInjectする用のExchangeRateComponentを作成
```java
@PerActivity
@Component(dependencies = AppComponent.class, modules = ExchangeRateModule.class)
public interface ExchangeRateComponent {
    void inject(MainActivity activity);
}
```

#### MainActivityのxmlにTextViewを1つ追加
```xml
<TextView
        android:id="@+id/txt_jpy"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
```

#### MainActivity
```java
public class MainActivity extends AppCompatActivity 
    implements ExchangeRateContract.View {

    @Inject
    ExchangeRatePresenter mPresenter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        DaggerExchangeRateComponent.builder()
            .appComponent(
                ((AppApplication) getApplicationContext()).getAppComponent())
            .exchangeRateModule(new ExchangeRateModule(this))
            .build().inject(this);

        mPresenter.getExchangeRate();
    }

    @Override
    public void bindExchangeRate(ExchangeRateResponse exchangeRateResponse) {
        TextView textView = (TextView) findViewById(R.id.txt_jpy);
        textView.setText("JPY: " + 
            String.valueOf(exchangeRateResponse.getRates().getJPY()));
    }
}
```

### 실행 결과
![alt](https://qiita-image-store.s3.amazonaws.com/0/33553/edec915e-6423-6106-f667-a2fa4dea7d2a.png)

Githubにサンプルを置いておきます<br>
[https://github.com/MuuKojima/MVPDaggerRetrofitRxAndoridSample](https://github.com/MuuKojima/MVPDaggerRetrofitRxAndoridSample)

CleanArchitectureを適用したものを追記しておきます
[http://qiita.com/MuuKojima/items/5d20a1b6b558cb432010](http://qiita.com/MuuKojima/items/5d20a1b6b558cb432010)



