title: 'Android: Interactors with Retrofit and RxJava'
date: 2017-07-01 15:24:55
tags:
- MVP
- Retrofit
- RxJava
---
In a [previous article](http://soflete.github.io/2017/07/01/Android-MVP-with-Dagger/) I described how to structure your application under the _Model-View-Presenter_ architecture applying dependency injection with [Dagger](http://google.github.io/dagger/). I covered how to setup a _View_ and its _Presenter_. Today I would like to share with you how to use an _Interactor_ to communicate the _Presenter_ with the _Model_.

Setup
-----

For this example, I am going to introduce [Retrofit](http://square.github.io/retrofit/), a networking library that makes it easy to define endpoints from which to retrieve data from the network. Also, I will make use of [RxJava](https://github.com/ReactiveX/RxJava) (and [RxAndroid](https://github.com/ReactiveX/RxAndroid)) to handle asynchronous requests. Let’s begin by updating the dependencies in our application's `build.gradle` file.

```Java
dependencies {
    ...
    // Retrofit
    compile 'com.squareup.retrofit2:retrofit:2.3.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.3.0'
    compile 'com.squareup.retrofit2:converter-gson:2.3.0'

    // RxAndroid and RxJava
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    compile 'io.reactivex.rxjava2:rxjava:2.1.1'
}
```

We added an _RxJava_ adapter to enable _Retrofit_ to return `Single` objects from its _Services_. We included the _Gson_ converter only because the endpoint we are going to use in this example returns responses in the _JSON_ format.

The Retrofit Service
--------------------

Let's say we want to list the latest news from [Geonet](http://www.geonet.org.nz/), a geological hazard monitoring system from New Zealand. The response we get from http://api.geonet.org.nz/news/geonet looks similar to this:

```JSON
{
  "feed": [
    {
      "title": "February 2016 landslides",
      "published": "2016-03-22T02:38:15Z",
      "link": "http://info.geonet.org.nz/display/slide/2016/03/22/February+2016+landslides",
      "mlink": "http://info.geonet.org.nz/m/view-rendered-page.action?abstractPageId=17039668"
    },
    ...
  ]
}
```

So let's create a class to represent a _News Story_.

```Java
public class NewsStory {

    @SerializedName("title")
    private final String title;

    @SerializedName("published")
    private final Date published;

    @SerializedName("mlink")
    private final String url;

    public NewsStory(String title, Date published, String url) {
        this.title = title;
        this.published = published;
        this.url = url;
    }

    public String getTitle() {
        return title;
    }

    public Date getPublished() {
        return published;
    }

    public String getUrl() {
        return url;
    }
}
```

And another one to represent the response from the service which contains the news feed.

```Java
public class NewsGeonetResponse {

    @SerializedName("feed")
    private final NewsStory[] newsStories;

    public NewsGeonetResponse(NewsStory[] newsStories) {
        this.newsStories = newsStories;
    }

    public NewsStory[] getNewsStories() {
        return newsStories;
    }
}
```

We are now ready to define our _Service_.

```Java
public interface GeonetService {

    @GET("news/geonet")
    Single<NewsGeonetResponse> getNews();
}
```

The Interactor
--------------

To shield our _Presenter_ from the specifics of _Retrofit_ we are going to create an _Interactor_.

```Java
public class GetNewsInteractor {
    private final GeonetService service;
    private Disposable disposable;

    public GetNewsInteractor(GeonetService service) {
        this.service = service;
    }

    public void execute(
            Consumer<? super NewsGeonetResponse> onSuccess,
            Consumer<? super Throwable> onError
    ) {
        disposable = service.getNews()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(onSuccess, onError);
    }

    public void cancel() {
        if (disposable != null) disposable.dispose();
    }
}
```

`GetNewsInteractor` executes a request on a separate thread and returns its response into the main thread (or UI thread). It also holds a subscription to the request that can be canceled at any time.

Let's add a reference to `GetNewsInteractor` into our _Presenter_.

```Java
public class MainPresenter {
    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }
}
```

Dependency Injection
--------------------

`GetNewsInteractor` needs to be provided to `MainPresenter` by its _Module_.

```Java
@Module
public class MainModule {

    private final MainView view;

    public MainModule(MainView view) {
        this.view = view;
    }

    @Provides
    public GetNewsInteractor provideInteractor(GeonetService service){
        return new GetNewsInteractor(service);
    }

    @Provides
    public MainPresenter providePresenter(GetNewsInteractor interactor) {
        return new MainPresenter(view, interactor);
    }
}
```

`MainPresenter` is injected with `GetNewsInteractor`. But to be able to inject `GeonetService` to `GetNewsInteractor` we need to convert `MainComponent` into a _Subcomponent_ of `ApplicationComponent`.

```Java
@Subcomponent (modules = MainModule.class)
public interface MainComponent extends AndroidInjector<MainFragment> {

    @Subcomponent.Builder
    abstract class Builder extends AndroidInjector.Builder<MainFragment> {
    }
}
```
We need to provide an injector for our _Fragment_. Let's create a module for that which `ApplicationComponent` can use:

```Java
@Module(subcomponents = MainComponent.class)
public abstract class InjectorsModule {

    @Binds
    @IntoMap
    @FragmentKey(MainFragment.class)
    abstract AndroidInjector.Factory<? extends Fragment> mainFragmentInjector(MainComponent.Builder builder);
}
```

Also, we want `GeonetService` to be available throughout the application. Let's create a Module to provide it.

```Java
@Module
public class ApplicationModule {

    @Provides
    public Gson provideGson(){
        return new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ssz").create();
    }

    @Provides
    public Retrofit provideRetrofit(Gson gson){
        return new Retrofit.Builder()
                .baseUrl("http://api.geonet.org.nz/")
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create(gson))
                .build();
    }

    @Provides
    public GeonetService provideGeonetService(Retrofit retrofit){
        return retrofit.create(GeonetService.class);
    }
}
```

Next, we need to expose the injector and the service through the application's _Component_.

```Java
@Singleton
@Component(modules = {
        ApplicationModule.class,
        AndroidInjectionModule.class,
        InjectorsModule.class
})
public interface ApplicationComponent extends AndroidInjector<MyApplication> {

    @Component.Builder
    abstract class Builder extends AndroidInjector.Builder<MyApplication> {
    }
}
```

In order to make the `ApplicationComponent` available throughout our application, we need to build it when the Application is created.

```Java
public class MyApplication extends Application implements HasSupportFragmentInjector {

    @Inject
    DispatchingAndroidInjector<Fragment> fragmentDispatchingAndroidInjector;

    @Override
    public void onCreate() {
        super.onCreate();
        DaggerApplicationComponent.builder().create(this).inject(this);
    }

    @Override
    public AndroidInjector<Fragment> supportFragmentInjector() {
        return fragmentDispatchingAndroidInjector;
    }
}
```

Don't forget to add `MyApplication` as the `name` attribute of the `application` element in `AndroidManifest.xml`.

Finally, we need to update `MainFragment` so it is injected by the appropriate injector.

```Java
public class MainFragment extends Fragment implements MainView {
    
    @Inject
    MainPresenter presenter;

    @Override
    public void onAttach(Context context) {
        AndroidSupportInjection.inject(this);
        super.onAttach(context);
    }
}
```

At this stage, `MainPresenter` can communicate with `MainView` and `GetNewsInteractor`. In an [upcoming example](http://soflete.github.io/2017/07/01/Android-Unit-Testing-Presenters/) I will explain how I apply _Test Driven Development (TDD)_ to define the behavior of the `MainPresenter` while I write a full set of Unit Tests for it.
