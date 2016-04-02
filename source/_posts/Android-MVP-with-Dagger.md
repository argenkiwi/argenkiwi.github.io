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

Next, we will create a _Component_ to hold references to modules and other components we will use in our `MainFragment`.

```Java
@Component(modules = MainModule.class)
public interface MainComponent {
    void inject(MainFragment fragment);
}
```

For the moment `MainComponent` holds a reference to `MainModule` (which we will create soon) and injects `MainFragment` with the objects it needs.

```Java
@Module
public class MainModule {
    @Provides
    public MainPresenter providePresenter(){
        return new MainPresenter();
    }
}
```

The `MainModule` provides the View with its _Presenter_.

Fragments as Views
------------------

Lets build our project in order to allow _Dagger_ to generate the appropriate code to inject the _Presenter_ into our _View_: the `MainFragment`.

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
- Our _Fragment_ implements the `MainView` interface.
- We have added a field to our _View_ representing our _Presenter_ and added an `@Inject` annotation on top to let _Dagger_ know `MainPresenter` needs to be injected.
- Finally, we make a call to the generated `DaggerMainComponent` builder and create an instance of `MainModule` passing `MainFragment` as a _View_.

```Java
public interface MainView {
}
```

`MainView` will define the interactions of `MainPresenter` with, in this case, `MainFragment`. `MainModule` now looks as follows:

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

The _Presenter_ should keep a reference to the _View_. `MainPresenter` should currently look like this:

```Java
public class MainPresenter {
    private final MainView view;

    public MainPresenter(MainView view) {
        this.view = view;
    }
}
```

Conclusion
----------

Now the `MainFragment` is ready to delegate all its events into the `MainPresenter` and rely on it to retrieve data from the _Model_. The _Presenter_ will make any decisions on how and when to update the View.

In an upcoming example I will show you how I make use of _TDD (Test Driven Development)_ to define the behavior of the `MainPresenter` while I write a full set of Unit Tests for it.
