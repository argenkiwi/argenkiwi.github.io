title: 'Android: MVP with Dagger'
date: 2017-07-01 14:44:01
tags:
- Android
- MVP
- Dagger
---
The _Model-View-Presenter_ pattern for software architecture helps to separate concerns in an application. In this article I intend to show you how I apply this pattern by using [Dagger](http://google.github.io/dagger/) for dependency injection.

First, we need to add the Dagger dependencies to our project:
```Gradle
dependencies {
    ...

    // Dagger
    compile 'com.google.dagger:dagger:2.11'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.11'

    // Dagger Android
    compile 'com.google.dagger:dagger-android:2.11'
    compile 'com.google.dagger:dagger-android-support:2.11'
    annotationProcessor 'com.google.dagger:dagger-android-processor:2.11'
}
```
Let's now create a an empty interface to define our View:
```Java
public interface MainView {

}
```
Next, let's create a concrete implementation of our Presenter which will hold a reference to the View:

```Java
public class MainPresenter {

    private final MainView view;

    public MainPresenter(MainView view) {
        this.view = view;
    }
}
```
In order to allow Dagger to create an instance of the Presenter which uses the Fragment as its View, we need to create an abstract module:
```Java
@Module
public abstract class MainModule {

    @Binds
    public abstract MainView mainView(MainFragment fragment);

    @Provides
    public static MainPresenter mainPresenter(MainView view){
        return new MainPresenter(view);
    }
}
```
Let's create a Component that references our Module and is able to inject our Fragment:
```Java
@Component(modules = {
        AndroidSupportInjectionModule.class,
        MainModule.class
})
public interface MainComponent extends AndroidInjector<MainFragment> {
    
    @Component.Builder
    abstract class Builder extends AndroidInjector.Builder<MainFragment> {
    }
}
```
Finally, we need to build our component and inject our Fragment:
```Java
public class MainFragment extends Fragment implements MainView {

    @Inject
    MainPresenter presenter;

    @Override
    public void onAttach(Context context) {
        // Before adding the following line, build your project.
        DaggerMainComponent.builder().create(this).inject(this);
        super.onAttach(context);
    }
}
```

Conclusion
----------

Now `MainFragment` is ready to delegate all its events to `MainPresenter` and rely on it to retrieve data from the _Model_. The _Presenter_ will make any decisions on how and when to update the View.

In my [next article](http://soflete.github.io/2016/04/07/Interactors-with-Retrofit-and-RxJava/) I explain how to create an _Interactor_ to retrieve data from an API and pass it on to the _Presenter_.
