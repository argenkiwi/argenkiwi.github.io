title: 'Android: Unit Testing Presenters'
date: 2016-04-05 18:52:37
tags:
- Android
- Unit Testing
- MVP
- Dagger
- Retrofit
---
In a previous article I explained how to structure your application code using the _Model-View-Presenter_ architecture pattern with the help of _Dagger_ for dependency injection.

In this post I would like to show you how I do _TDD (Test Driven Development)_ to write the code of the _Presenter_ of a _View_.

Setup
-----

For this example I am going to introduce _Retrofit_, a networking library that makes it easier to define endpoints from which we will retrieve data we will use in our application. Let's update the dependencies in the `build.gradle` file of our application.

```Java
...

dependencies {
    ...
    compile 'com.squareup.retrofit2:retrofit:2.0.1'1
    compile 'com.squareup.retrofit2:converter-gson:2.0.1'
}
```

Next, we will create and interface to represent the service we will request data from. In this case, we are going to load some news stories from Geonet, a geological hazard monitoring system from New Zealand.

```Java
public interface GeonetService {
    @GET("news/geonet")
    void getNews(Callback<NewsGeonetResponse> callback);
}
```

The Application Component
-------------------------

Now we need to create an instance of the service to use in our presenter. We are going to create an `ApplicationComponent` which will expose this service and other _Singletons_ to be used across the application.

```Java
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    GeonetService getGeonetService();
}
```

The `ApplicationModule` is going to provide an instance of `GeonetService`.

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
                .addConverterFactory(GsonConverterFactory.create(gson))
                .build();
    }

    @Provides
    public GeonetService provideGeonetService(Retrofit retrofit){
        return retrofit.create(GeonetService.class);
    }
}
```

Finally, we need our `MainComponent` to expose this service to our `MainModule` as well.

```Java
@ViewScope
@Component(dependencies = ApplicationComponent.class, modules = MainModule.class)
public interface MainComponent {
    void inject(MainFragment fragment);
}
```

We can now inject the service into our `MainPresenter`.

```Java
@Module
public class MainModule {
    private final MainView view;

    public MainModule(MainView view) {
        this.view = view;
    }

    @Provides
    public MainPresenter providePresenter(GeonetService service) {
        return new MainPresenter(view, service);
    }
}
```

Test Driven Development and Presenters
--------------------------------------

Our _Presenter_ is responsible of retrieving the data displayed in our _View_. In this case, we expect it to request for _News Stories_ as soon as our view is created.

```Java
public class MainFragment extends Fragment implements MainView {

    @Inject
    MainPresenter presenter;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        DaggerMainComponent.builder()
                .mainModule(new MainModule(this))
                .build().inject(this);
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        presenter.onViewCreated();
    }
}
```

We want to make sure `MainPresenter` calls the _getNews()_ method of `GeonetService` when its _onViewCreated()_ method is executed. Let's create a Unit Test for our presenter to verify that.

```Java
```
