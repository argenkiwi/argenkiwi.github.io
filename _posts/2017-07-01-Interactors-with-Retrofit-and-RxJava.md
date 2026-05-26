---
layout: post
title: "Android: Interactors with Retrofit and RxJava"
date: 2017-07-01 03:24:55 +0000
tags: [MVP, Retrofit, RxJava]
categories: [Android]
---

In a [previous article]({% post_url 2017-07-01-Android-MVP-with-Dagger %}) I described how to structure your application under the *Model-View-Presenter* architecture applying dependency injection with [Dagger](https://google.github.io/dagger/). I covered how to setup a *View* and its *Presenter*. Today I would like to share with you how to use an *Interactor* to communicate the *Presenter* with the *Model*.

## Setup

For this example, I am going to introduce [Retrofit](https://square.github.io/retrofit/), a networking library that makes it easy to define endpoints from which to retrieve data from the network. Also, I will make use of [RxJava](https://github.com/ReactiveX/RxJava) (and [RxAndroid](https://github.com/ReactiveX/RxAndroid)) to handle asynchronous requests. Let’s begin by updating the dependencies in our application’s `build.gradle` file.

```java
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

We added an *RxJava* adapter to enable *Retrofit* to return `Single` objects from its *Services*. We included the *Gson* converter only because the endpoint we are going to use in this example returns responses in the *JSON* format.

## The Retrofit Service

Let’s say we want to list the latest news from [Geonet](https://www.geonet.org.nz/), a geological hazard monitoring system from New Zealand. The response we get from [https://api.geonet.org.nz/news/geonet](https://api.geonet.org.nz/news/geonet) looks similar to this:

```json
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

So let’s create a class to represent a *News Story*.

```java
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

```java
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

We are now ready to define our *Service*.

```java
public interface GeonetService {

    @GET("news/geonet")
    Single<NewsGeonetResponse> getNews();
}
```

## The Interactor

To shield our *Presenter* from the specifics of *Retrofit* we are going to create an *Interactor*.

```java
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
        if (disposable != null && !disposable.isDisposed()) {
            disposable.dispose();
        }
    }
}
```

Let’s add a reference to `GetNewsInteractor` into our *Presenter*.

```java
public class MainPresenter {
    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }
}
```

## Dependency Injection

Start by letting `GetNewsInteractor` be provided to `MainPresenter` by its *Module*.

```java
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
    public MainPresenter providePresenter(GetNewsInteractor interactor){
        return new MainPresenter(view, interactor);
    }
}
```

Then, create a module which provides `GeonetService` and its dependencies:

```java
@Module
public abstract class ServicesModule {

    @Singleton
    @Provides
    public static Gson provideGson(){
        return new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ssz").create();
    }

    @Singleton
    @Provides
    public static Retrofit provideRetrofit(Gson gson){
        return new Retrofit.Builder()
                .baseUrl("http://api.geonet.org.nz/")
                .addConverterFactory(GsonConverterFactory.create(gson))
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();
    }

    @Singleton
    @Provides
    public static GeonetService provideGeonetService(Retrofit retrofit){
        return retrofit.create(GeonetService.class);
    }
}
```

To be able to inject `GeonetService` to `GetNewsInteractor` we need to convert `MainComponent` into a *Subcomponent* of `ApplicationComponent`.

```java
@Subcomponent (modules = MainModule.class)
public interface MainComponent extends AndroidInjector<MainFragment> {

    @Subcomponent.Builder
    abstract class Builder extends AndroidInjector.Builder<MainFragment> {
    }
}
```

We have to provide an *Injector* for our *Fragment*. Let’s create a module which `ApplicationComponent` can reference:

```java
@Module(subcomponents = MainComponent.class)
public abstract class InjectorsModule {

    @Binds
    @IntoMap
    @FragmentKey(MainFragment.class)
    abstract AndroidInjector.Factory<? extends Fragment> mainFragmentInjector(MainComponent.Builder builder);
}
```

Now we can expose the *Injector* and the *Service* through `ApplicationComponent`.

```java
@Singleton
@Component(modules = {
        AndroidInjectionModule.class,
        InjectorsModule.class,
        ServicesModule.class
})
public interface ApplicationComponent extends AndroidInjector<MyApplication> {

    @Component.Builder
    abstract class Builder extends AndroidInjector.Builder<MyApplication> {
    }
}
```

In order to make `ApplicationComponent` available from our *Application* instance, we need to build it when our *Application* is created.

```java
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

Don’t forget to add `MyApplication` as the `name` attribute of the `application` element in `AndroidManifest.xml`.

Finally, we need to update `MainFragment` so it is injected by the appropriate injector.

```java
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

At this stage, `MainPresenter` can communicate with `MainView` and `GetNewsInteractor`. In an [upcoming example]({% post_url 2017-07-01-Android-Unit-Testing-Presenters %}) I will explain how I apply *Test Driven Development (TDD)* to define the behavior of the `MainPresenter` while I write a full set of Unit Tests for it.
