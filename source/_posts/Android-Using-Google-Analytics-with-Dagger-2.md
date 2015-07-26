title: 'Android: Using Google Analytics with Dagger 2'
date: 2015-07-26 07:53:25
tags:
- Android
- Dagger2
- GoogleAnalytics
---
I have been teaching myself how to use [Dagger 2](http://google.github.io/dagger/) for Android development. I found out that the best place to start is by letting this library manage the singletons of your application. As an example, I would like to show you how to add Google Analytics using Dagger 2.

Setup
------------

Lets modify our application's build.gradle file in order to include Dagger 2 and Google Analytics in our project:

```Gradle
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

android {
    ...
}

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

dependencies {
    ...
    compile 'com.google.android.gms:play-services:7.5.0'
    compile 'com.google.dagger:dagger:2.0'
    provided 'org.glassfish:javax.annotation:10.0-b28'
    apt 'com.google.dagger:dagger-compiler:2.0'
}
```

Make sure to apply the adt plugin and declare its dependencies. Also include the dependencies for Google Play Services and Dagger 2.

You will also need to create a configuration XML file for Google Analytics (in our example we will use app_tracker.xml). For more information on how to setup Google Analytics, check steps 1 and 3 of the official instructions in the [Analytics SDK for Android](https://developers.google.com/analytics/devguides/collection/android/v4/) site.

Implementation
--------------

Now that both Google Analytics and Dagger 2 are available to be used in our Android project, we are going to create a `@Module` for our application:

```Java
@Module
public class ApplicationModule {

    private final MyApplication mApplication;

    public ApplicationModule(MyApplication application) {
        mApplication = application;
    }

    @Provides
    @Named("app")
    Tracker provideTracker() {
        return GoogleAnalytics.getInstance(mApplication).newTracker(R.xml.app_tracker);
    }
}
```

Our Application Module holds a reference to our Application class passed to its constructor. You may have noticed I added the `@Named` annotation to the `provideTracker()` method. This is how Dagger 2 distinguishes between different instances of the same class, which could be very useful, for instance, if you needed to have multiple GA trackers with different configurations.

Next, let's create our `ApplicationComponent` and make it available to our activities through a getter method:

```Java
public class MyApplication extends Application {

    private ApplicationComponent mApplicationComponent;

    @Override
    public void onCreate() {
        super.onCreate();
        mApplicationComponent = DaggerMyApplication_ApplicationComponent.builder()
                .applicationModule(new ApplicationModule(this))
                .build();
    }

    public ApplicationComponent getApplicationComponent() {
        return mApplicationComponent;
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
                .applicationComponent(((MyApplication) getApplication())
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

We defined `MainComponent` which depends on `ApplicationComponent` and therefore has access to the `Tracker` named "app".
The `inject(MainActivity mainActivity)` method injects `Tracker` (and every other injectable if there was any) to the Activity. Right after invoking it, we are ready to use `Tracker` (in the example above, we set the Screen Name).

You probably noticed the `@ActivityScope` annotation on top of the `MainComponent` interface. In Dagger 2 you can define what is called a custom scope. Scopes tell Dagger 2 what the lifetime of a certain component should be. In the case of `ApplicationComponent` the scope is `@Singleton`. Here is how we define our custom scope:

```Java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityScope {
}
```

Conclusion
----------

All we have to do from now on is inject `Tracker` in our Activities and make use of it. We can also let the `ApplicationModule` provide other types of singletons (e.g.: `SharedPreferences`, `SQLiteOpenHelper`, `SimpleDateFormat`, `Gson`, etc.) and simply inject them where we need them. Hopefully, Dagger 2 will make your code more readable and the lifetime of your objects more predictable.

Of course this is just a small demonstration of what you can do with dependency injection. I hope this example serves as a starting point for those of you new to Dagger 2.
