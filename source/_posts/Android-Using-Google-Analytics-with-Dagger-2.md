title: 'Android: Using Google Analytics with Dagger 2'
date: 2015-07-26 07:53:25
comments: true
tags:
- Android
- Dagger
- GoogleAnalytics
---
I have been teaching myself how to use [Dagger](http://google.github.io/dagger/) for _Android_ development. I found out that the best place to start is by letting this library manage the singletons of your application. As an example, I would like to show you how to inject _Google Analytics_ using _Dagger_.

Setup
-----

Make sure to [add the Dagger dependencies to your project](http://soflete.github.io/2016/04/02/Setup-Dagger-in-Android/) first. Then, let's modify our application's `build.gradle` file to include _Google Analytics_ into our project:

```Gradle
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

android {
    ...
}

dependencies {
    ...
    compile 'com.google.android.gms:play-services:8.4.0'
}
```

You will also need to create a configuration XML file for _Google Analytics_ (in our example we will use app_tracker.xml). For more information on how to setup Google Analytics, check steps 1 and 3 of the official instructions in the [Analytics SDK for Android](https://developers.google.com/analytics/devguides/collection/android/v4/) site.

Implementation
--------------

Now that both _Google Analytics_ and _Dagger_ are available to be used in our Android project, we are going to create a `@Module` for our application:

```Java
@Module
public class ApplicationModule {

    private final MyApplication application;

    public ApplicationModule(MyApplication application) {
        this.application = application;
    }

    @Provides
    @Named("app")
    Tracker provideTracker() {
        return GoogleAnalytics.getInstance(application).newTracker(R.xml.app_tracker);
    }
}
```

Our Application Module holds a reference to `MyApplication`. You may have noticed I added the `@Named` annotation to the `provideTracker()` method. This is how _Dagger_ distinguishes between different instances of the same class, which could be very useful if you needed to have multiple _Google Analytics_ trackers with different configurations.

Next, let's create our `ApplicationComponent` and make it available to our activities through a getter method:

```Java
public class MyApplication extends Application {

    private ApplicationComponent component;

    public static MyApplication get(Context context) {
        return (MyApplication) context.getApplicationContext();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        this.component = DaggerMyApplication_ApplicationComponent.builder()
                .applicationModule(new ApplicationModule(this))
                .build();
    }

    public ApplicationComponent getApplicationComponent() {
        return this.component;
    }

    @Singleton
    @Component(modules = ApplicationModule.class)
    public interface ApplicationComponent {

        @Named("app")
        Tracker getTracker();
    }
}
```

As you can see, `ApplicationComponent` holds a reference to `ApplicationModule`. We instantiate the `ApplicationModule` passing a reference to `MyApplication` as a parameter. The `getTracker()` method makes `Tracker` available to whatever component depends on `ApplicationComponent`. As an example, lets create an activity with a component of its own:

```Java
public class MainActivity extends AppCompatActivity {

    @Inject
    @Named("app")
    Tracker mTracker;

    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        DaggerMainActivity_MainComponent.builder()
                .applicationComponent(MyApplication.get(this)
                        .getApplicationComponent())
                .build()
                .inject(this);

        mTracker.setScreenName("main screen");

        setContentView(R.layout.activity_main);

        ...
    }

    ...

    @ActivityScope
    @Component(dependencies = MyApplication.ApplicationComponent.class)
    public interface MainComponent {
        void inject(MainActivity mainActivity);
    }
}
```

We defined `MainComponent` which depends on `ApplicationComponent` and therefore has access to the `Tracker` named `app`.
The `inject(MainActivity mainActivity)` method injects `Tracker` (and every other injectable object if there was any) to the _Activity_. Right after invoking it, we are ready to use `Tracker` (in the example above, we set the screen name).

You probably noticed the `@ActivityScope` annotation on top of `MainComponent`. In _Dagger_ you can define what is called a custom scope. Scopes tell _Dagger_ what the lifetime of a certain component should be. In the case of `ApplicationComponent` the scope is `@Singleton`. Here is how we define our custom scope:

```Java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope {
}
```

Conclusion
----------

All we have to do from now on is inject `Tracker` in our activities to be able to use it. We can also let `ApplicationModule` provide other types of singletons (e.g.: `SharedPreferences`, `SQLiteOpenHelper`, `SimpleDateFormat`, `Gson`, etc.) and simply inject them where we need them. Hopefully, _Dagger_ will make your code more readable and the lifetime of your objects more predictable.

Of course this is just a small demonstration of what you can do with dependency injection. I hope this example serves as a starting point for those of you new to _Dagger_.
