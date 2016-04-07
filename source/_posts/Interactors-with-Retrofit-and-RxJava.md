title: 'Android: Interactors with Retrofit and RxJava'
date: 2016-04-07 15:48:51
tags:
- MVP
- Retrofit
- RxJava
---
In a [previous article](http://soflete.github.io/2016/04/01/Android-MVP-with-Dagger/) I described how to structure your application under the _Model-View-Presenter_ architecture applying dependency injection with [Dagger](http://google.github.io/dagger/). I covered how to setup a _View_ and its _Presenter_. Today I would like to share with you how to use an _Interactors_ to communicate the _Presenter_ with the _Model_.

Setup
-----

For this example, I am going to introduce [Retrofit](http://square.github.io/retrofit/), a networking library that makes it easy to define endpoints from which to retrieve data from the network. Also, I will make use of [RxJava](https://github.com/ReactiveX/RxJava) (and [RxAndroid](https://github.com/ReactiveX/RxAndroid)) to handle asynchronous requests. Let’s begin by updating the dependencies in our application's `build.gradle` file.

```Java
dependencies {
    ...
    compile 'com.squareup.retrofit2:retrofit:2.0.1'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.1'
    compile 'com.squareup.retrofit2:converter-gson:2.0.1'
    compile 'io.reactivex:rxandroid:1.1.0'
    compile 'io.reactivex:rxjava:1.1.2'
    ...
}
```

We added an _RxJava_ adapter to enable _Retrofit_ to return `Observable` objects from its _Services_. We included the _GSon_ converter only because the endpoint we are going to use in this example returns responses in the _JSON_ format.

Retrofit Services
-----------------

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
    private String title;

    @SerializedName("published")
    private Date published;

    @SerializedName("link")
    private String url;

    public Date getPublished() { return published; }

    public String getTitle() { return title; }

    public String getUrl() { return url; }
}
```

And another one to represent the response from the service which contain the news feed.

```Java
public class NewsGeonetResponse {

    @SerializedName("feed")
    private NewsStory[] newsStories;

    public NewsStory[] getNewsStories() {
        return newsStories;
    }
}
```
We are now ready to define our _Service_.

```Java
public interface GeonetService {
    @GET("news/geonet")
    void getNews(Callback<NewsGeonetResponse> callback);
}
```

We want `GeonetService` to be available throughout the application. Therefore, it will be provided by our application's _Module_.

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

Finally, we need to expose the service through the application's _Component_.

```Java
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    GeonetService getGeonetService();
}
```

Interactors
-----------

To shield our _Presenter_ from the specifics of _Retrofit_ we are going to create an _Interactor_.

```Java
public class GetNewsInteractor {

    private final GeonetService service;
    private Subscription subscription = Subscriptions.empty();

    public GetNewsInteractor(GeonetService service) {
        this.service = service;
    }

    public void execute(Subscriber<NewsGeonetResponse> subscriber) {
        subscription = service.getNews()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }

    public void cancel() {
        if (!subscription.isUnsubscribed()) {
            subscription.unsubscribe();
        }
    }
}
```

`GetNewsInteractor` executes a request on a separate thread and returns its response into the main thread (or UI thread). It also holds a subscription to the request that can be canceled at any time. We need this _Interactor_ to be available to our _Presenter_, so let's add it to its _Module_.

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

`MainPresenter` is injected with `GetNewsInteractor`. But to be able to inject `GeonetService` to `GetNewsInteractor` we need our _Component_ to declare a dependency on `ApplicationComponent`.

```Java
@ViewScope
@Component(dependencies = ApplicationComponent.class, modules = MainModule.class)
public interface MainComponent {
    void inject(MainFragment fragment);
}
```

Finally, our _Presenter_ needs to implement _RxJava_'s `Subscriber` interface so it can act as a callback for the requests made by `GetNewsInteractor`.

```Java
public class MainPresenter extends Subscriber<NewsGeonetResponse> {

    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }

    @Override
    public void onCompleted() {
    }

    @Override
    public void onError(Throwable e) {
    }

    @Override
    public void onNext(NewsGeonetResponse newsGeonetResponse) {
    }
}
```

In an upcoming example I will explain how I apply _Test Driven Development (TDD)_ to define the behavior of the `MainPresenter` while I write a full set of Unit Tests for it.