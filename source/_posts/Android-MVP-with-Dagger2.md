title: 'Android: MVP with Dagger2'
date: 2016-04-01 08:00:01
tags:
- Android
- MVP
- Dagger2
---
The Model-View-Presenter pattern for software architecture helps me to separate concerns in my application. I would like to share with you how I structure my code using dependency injection.

Setup
-----

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
    compile 'com.google.dagger:dagger:2.1'
    provided 'org.glassfish:javax.annotation:10.0-b28'
    apt 'com.google.dagger:dagger-compiler:2.1'
}
```

Make sure to apply the adt plugin and declare its dependencies. Also include the dependencies for Google Play Services and Dagger 2.

Dagger components and modules
-----------------------------

Let's say we have just created a new Android project in which we have a `MainActivity` and we created a `MainFragment` to contain our UI.

```Java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if(savedInstanceState == null){
            getFragmentManager().beginTransaction()
                    .add(android.R.id.content, new MainFragment())
                    .commit();
        }
    }
}
```

Next, we will create a component to hold references to modules and other components we will use in our `MainFragment`.

```Java
@Component(modules = MainModule.class)
public interface MainComponent {
    void inject(MainFragment fragment);
}
```

For the moment `MainModule` only holds a reference to `MainModule` (which we will create soon) and injects the MainFragment with the objects it needs, like the `MainPresenter` the `MainModule` will provide:

```Java
@Module
public class MainModule {
    @Provides
    public MainPresenter providePresenter(){
        return new MainPresenter();
    }
}
```
Fragments as Views
------------------

Lets build our project in order to allow Dagger to generate the appropriate code to inject the Presenter into our View: the `MainFragment`.

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
}
```

A few things to note here:
- Our fragment implements the `MainView` interface, which has no methods yet.
- We added a field to our fragment representing our presenter and added an `@Inject` annotation on top to let Dagger know the Presenter needs to be injected.
- Finally, we make a call to the generated `DaggerMainComponent` class builder and create an instance of the `MainModule` passing the `MainFragment` as a `MainView`.

The `MainModule` now looks as follows:

```Java
@Module
public class MainModule {
    private final MainView view;

    public MainModule(MainView view) {
        this.view = view;
    }

    @Provides
    public MainPresenter providePresenter() {
        return new MainPresenter(view);
    }
}
```

Conclusion
----------

Now the `MainFragment` is ready to delegate all its events into the `MainPresenter` and let it retrieve data from the Model and make any decisions on how and when to update the View. In an upcoming example I will show you how I make use of Test Driven Development to define the use cases handled by the `MainPresenter` while I write a full set of Unit Tests for it.
