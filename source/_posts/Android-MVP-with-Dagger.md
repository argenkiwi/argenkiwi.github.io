title: 'Android: MVP with Dagger'
date: 2016-04-01 08:00:01
tags:
- Android
- MVP
- Dagger
---
The _Model-View-Presenter_ pattern for software architecture helps separate concerns in an application. I would like to share with you how I apply this pattern by using [Dagger](http://google.github.io/dagger/) for dependency injection.

Setup
-----

To learn how to include _Dagger_ into your project's dependencies, click [here](http://soflete.github.io/2016/04/02/Setup-Dagger-in-Android/).

Dagger components and modules
-----------------------------

Let's say we have just created a new Android project in which we have a `MainActivity` and we created a `MainFragment` (empty for now) to contain our UI.

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

Let's begin by creating a and empty _Presenter_ class.

```Java
public class MainPresenter {
}
```

We will need a _Module_ to provide an instance of `MainPresenter`.

```Java
@Module
public class MainModule {

    @Provides
    public MainPresenter providePresenter(){
        return new MainPresenter();
    }
}
```

Finally, we'll also need to create a _Component_ to inject the `MainPresenter`, provided by `MainModule`, into our `MainFragment`, which, as you will see next, is going to be our _View_.

```Java
@Component(modules = MainModule.class)
public interface MainComponent {
    void inject(MainFragment fragment);
}
```

Fragments as Views
------------------

We want to shield our _Presenter_ from knowing the concrete implementation of the _View_. Therefore, we will create an interface.

```Java
public interface MainView {
}
```

Let's update `MainPresenter` to keep a reference to `MainView`.

```Java
public class MainPresenter {

    private final MainView view;

    public MainPresenter(MainView view) {
        this.view = view;
    }
}
```

We now need to update `MainModule` to inject `MainView` into `MainPresenter`.

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

Finally, we will make `MainFragment` implement `MainView`. Let's build our project at this point in order to allow _Dagger_ to generate the concrete implementation of `MainComponent`.

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
- We have added a `presenter` field with the `@Inject` annotation on top to let _Dagger_ know `MainPresenter` needs to be injected.
- We create an instance of `MainModule` and pass the _View_ as a parameter in its constructor.
- We build `DaggerMainComponent` and inject `MainFragment` with its dependencies (`MainPresenter` in this case).

Conclusion
----------

Now the `MainFragment` is ready to delegate all its events into the `MainPresenter` and rely on it to retrieve data from the _Model_. The _Presenter_ will make any decisions on how and when to update the View.

In my [next article](http://soflete.github.io/2016/04/07/Interactors-with-Retrofit-and-RxJava/) I explain how to create an _Interactor_ to retrieve data from an API and pass it on to the _Presenter_.
