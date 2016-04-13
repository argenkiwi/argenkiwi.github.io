title: 'Android: Unit Testing Presenters'
date: 2016-04-10 11:15:37
tags:
- Android
- Unit Testing
- TDD
- MVP
---
This is the third post of a series. In the [first post](http://soflete.github.io/2016/04/01/Android-MVP-with-Dagger/) I demonstrate how to setup the _View_ and the _Presenter_ of a _Model-View-Presenter_ architecture using _dependency injection_. In the [second post](http://soflete.github.io/2016/04/07/Interactors-with-Retrofit-and-RxJava/) I describe how to put together an _Interactor_ and inject it into the _Presenter_.

In this post I would like to show you how I apply _TDD (Test Driven Development)_ to define and implement the interactions of the _Presenter_ with the _View_ and the _Interactor_.

Setup
-----

In order to execute Unit Tests in _Android Studio_ you need to make sure you have a `src/test` folder. Also, you need to include dependencies on _JUnit_ and _Mockito_ in your application's `build.gradle` file.

```Java
...

dependencies {
    ...
    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-core:1.10.19'
}
```

The Unit Test
-------------

When you do _TDD_ you start by writing a test and making sure it fails. In this case, we are writing tests for the _Presenter_ we created in the previous articles.

**Note:** If you are using _Android Studio_ you can place your cursor on `MainPresenter`, press `Alt+Enter` and select the `Create Test` option. Make sure to save the test in the `src/test` folder.

```Java
@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {

    private MainPresenter presenter;

    @Mock
    private MainView view;

    @Mock
    private GetNewsInteractor interactor;

    @Before
    public void setUp() throws Exception {
        presenter = new MainPresenter(view, interactor);
    }

    // TODO: Your tests here.
}
```

The `setUp()` method creates an instance of `MainPresenter` before running each test. For that we need to _Mock_ `MainView` and `GetNewsInteractor`. The `@RunWith(MockitoJUnitRunner.class)` annotation makes sure the fields marked with the @Mock annotation are initialised before running each test.

We are now ready to write our first test. We would like to ensure `MainPresenter` attempts to load some news when the view is created.

```Java
...
import static org.mockito.Mockito.verify;

@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...    
    @Test
    public void shouldGetNews(){
        presenter.onViewCreated();
        verify(interactor).execute(presenter);
    }
}
```

`Mockito.verify()` makes sure the methods of a _Mock_ are being executed and that the expected parameters are passed to them. If you try to run this test, you will get a compile time error.

This simple test forces us to do two things:
- Add an `onViewCreated` method to our `MainPresenter`.
- Make `MainPresenter` implement RxJava's `Observer` interface in order to receive events from `GetNewsInteractor`.

```Java
public class MainPresenter implements Observer<NewsGeonetResponse> {

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

    public void onViewCreated() {

    }
}
```

If we run the test again this time, it will tell us `interactor.execute(presenter)` hasn't been executed. This is great, we've just applied the first step of _TDD_: we made our test fail. Let's move on to getting our test to pass by implementing the `onViewCreated` method of `MainPresenter`.

```Java
public class MainPresenter implements Observer<NewsGeonetResponse> {
    ...
    public void onViewCreated() {
        interactor.execute(this);
    }
}
```

If everything went as expected, our test should now pass. Easy, right? Now we want to make sure we show the news when they finish loading.

```Java
...
import static org.mockito.Mockito.when;

@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...
    @Mock
    private NewsGeonetResponse response;

    ...
    @Test
    public void shouldShowNews(){
        NewsStory[] newsStories = new NewsStory[]{};
        when(response.getNewsStories()).thenReturn(newsStories);
        presenter.onNext(response);
        verify(view).showNews(newsStories);
    }
}
```

In this case, I chose to mock the response. This way I can specify what the `getNewsStories()` method returns by using `Mockito.when()`. Again, if we run our test now it will fail because we still haven't implemented the `showNews` method. `MainView` should look as follows:

```Java
public interface MainView {
    void showNews(NewsStory[] newsStories);
}
```

Make sure `MainFragment` implements the new method of the `MainView` interface and run the test again. It should still fail. It is time to update `MainPresenter` to make the test pass.

```Java
public class MainPresenter implements Observer<NewsGeonetResponse> {
    ...
    @Override
    public void onNext(NewsGeonetResponse newsGeonetResponse) {
        view.showNews(newsGeonetResponse.getNewsStories());
    }
}
```

The test should pass now. If so, well done!

If you got this far, I can safely assume you are a smart reader and you already understood how the process works. So I'll skip the explanations and write a few more tests.

```Java
@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...
    @Mock
    private Throwable error;

    ...
    @Test
    public void shouldCancel() {
        presenter.onStop();
        verify(interactor).cancel();
    }

    @Test
    public void shouldHideLoading(){
        presenter.onCompleted();
        verify(view).hideLoading();
    }

    @Test
    public void shouldShowError(){
        presenter.onError(error);
        verify(view).showError();
    }

    @Test
    public void shouldShowLoading(){
        presenter.onViewCreated();
        verify(view).showLoading();
    }
}
```

I want a loading dialog to show when the view is created. Also, I want to hide it when we finish loading the news. Finally, if there is an error, I want to display an error message. If you apply the steps we described before, `MainView` should now look as follows.

```Java
public interface MainView {
    void hideLoading();

    void showError();

    void showLoading();

    void showNews(NewsStory[] newsStories);
}
```

If you managed to make your tests pass, `MainPresenter` should be similar to this:

```Java
public class MainPresenter implements Observer<NewsGeonetResponse> {

    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }

    @Override
    public void onCompleted() {
        view.hideLoading();
    }

    @Override
    public void onError(Throwable e) {
        view.showError();
    }

    @Override
    public void onNext(NewsGeonetResponse newsGeonetResponse) {
        view.showNews(newsGeonetResponse.getNewsStories());
    }

    public void onStart() {
        view.showLoading();
        interactor.execute(this);
    }

    public void onStop() {
        interactor.cancel();
    }
}
```

Finally, `MainFragment` should also have some new empty methods.

```Java
public class MainFragment extends Fragment implements MainView {

    @Inject
    MainPresenter presenter;

    ...
    @Override
    public void hideLoading() {

    }

    @Override
    public void showError() {

    }

    @Override
    public void showLoading() {

    }

    @Override
    public void showNews(NewsStory[] newsStories) {

    }
}
```

Conclusion
----------

By writing _Unit Tests_ for our _Presenter_ first, we can test and implement the behaviour of our application without writing a single line of UI code. This allows us to focus on one aspect of our application at a time. Also, our feedback loop shortens as we don't need to deploy our application to a device or an emulator to verify it is doing the right thing. Implementing the methods in our _View_ should be trivial, given each of them fulfills a very specific purpose as a byproduct of applying both _TDD_ and _MVP_.

This concludes my series of articles on the _Mode-View-Presenter_ architecture and _Test Driven Development_ applied to _Android_. I hope some of these ideas help you make your development process more enjoyable.
