---
layout: post
title: "Android: Unit Testing Presenters"
date: 2017-07-01 04:53:37 +0000
tags: [Android, MVP, Unit Testing, TDD]
---

This is the third post of a series. In the [first post]({% post_url 2017-07-01-Android-MVP-with-Dagger %}) I demonstrate how to setup the *View* and the *Presenter* of a *Model-View-Presenter* architecture using *dependency injection*. In the [second post]({% post_url 2017-07-01-Interactors-with-Retrofit-and-RxJava %}) I describe how to put together an *Interactor* and inject it into the *Presenter*.

In this post I would like to show you how I apply *TDD (Test Driven Development)* to define and implement the interactions of the *Presenter* with the *View* and the *Interactor*.

## Setup

In order to execute Unit Tests in *Android Studio* you need to make sure you have a `src/test` folder. Also, you need to include dependencies on *JUnit* and *Mockito* in your application’s `build.gradle` file.

```java
dependencies {
    ...
    testCompile 'junit:junit:4.12'                      
    testCompile 'org.mockito:mockito-core:2.8.47' 
}
```

## The Unit Test

When you do *TDD* you start by writing a test and making sure it fails. In this case, we are writing tests for the *Presenter* we created in the previous articles.

**Note:** If you are using *Android Studio* you can place your cursor on `MainPresenter`, press `Alt+Enter` and select the `Create Test` option. Make sure to save the test in the `src/test` folder.

```java
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

The `setUp()` method creates an instance of `MainPresenter` before running each test. For that we need to *Mock* `MainView` and `GetNewsInteractor`. The `@RunWith(MockitoJUnitRunner.class)` annotation makes sure the fields marked with the @Mock annotation are initialised before running each test.

We are now ready to write our first test. We would like to ensure `MainPresenter` attempts to load some news when the view is created.

```java
...
import static org.mockito.Mockito.verify;

@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...    
    @Test
    public void shouldGetNews(){
        presenter.onViewCreated();
        verify(interactor).execute(
                ArgumentMatchers.<Consumer<NewsGeonetResponse>>any(),
                ArgumentMatchers.<Consumer<Throwable>>any()
        );
    }
}
```

`Mockito.verify()` makes sure the methods of a *Mock* are being executed and that the expected parameters are passed to them. If you try to run this test, you will get a compile time error.

By writing this test we realise we need to add an `onViewCreated` method to our `MainPresenter`.

```java
public class MainPresenter {
    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }

    public void onViewCreated() {

    }
}
```

If we run the test again this time, it will tell us `interactor.execute(presenter)` hasn’t been executed. This is great, we’ve just applied the first step of *TDD*: we made our test fail. Let’s move on to getting our test to pass by implementing the `onViewCreated` method of `MainPresenter`.

```java
public class MainPresenter {
    ...
    public void onViewCreated() {
        interactor.execute(new Consumer<NewsGeonetResponse>() {
            @Override
            public void accept(@NonNull NewsGeonetResponse newsGeonetResponse) throws Exception {
                // TODO Show news.
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                // TODO Handle error.
            }
        });
    }
}
```

If everything went as expected, our test should now pass. Easy, right? Now we want to make sure we show the news when they finish loading.

```java
@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...
    @Captor
    private ArgumentCaptor<Consumer<NewsGeonetResponse>> newsConsumerCaptor;

    ...
    @Test
    public void shouldShowNews() throws Exception {
        NewsStory[] stories = new NewsStory[]{};
        NewsGeonetResponse response = new NewsGeonetResponse(stories);

        presenter.onViewCreated();
        verify(interactor).execute(
                newsConsumerCaptor.capture(),
                ArgumentMatchers.<Consumer<Throwable>>any()
        );

        newsConsumerCaptor.getValue().accept(response);
        verify(view).showNews(stories);
    }
}
```

Again, if we run our test now it will fail because we still haven’t implemented the `showNews` method. `MainView` should look as follows:

```java
public interface MainView {
    void showNews(NewsStory[] newsStories);
}
```

Make sure `MainFragment` implements the new method of the `MainView` interface and run the test again. It should still fail. It is time to update `MainPresenter` to make the test pass.

```java
public class MainPresenter implements Observer<NewsGeonetResponse> {
    ...
    public void onViewCreated() {
        interactor.execute(new Consumer<NewsGeonetResponse>() {
            @Override
            public void accept(@NonNull NewsGeonetResponse newsGeonetResponse) throws Exception {
                view.showNews(newsGeonetResponse.getNewsStories());
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                // TODO Handle error.
            }
        });
    }
}
```

The test should pass now. If so, well done!

If you got this far, I can safely assume you are a smart reader and you already understood how the process works. So I’ll skip the explanations and write a few more tests.

```java
@RunWith(MockitoJUnitRunner.class)
public class MainPresenterTest {
    ...
    @Captor
    private ArgumentCaptor<Consumer<Throwable>> throwableConsumerCaptor;

    ...
    @Test
    public void shouldShowError() throws Exception {
        presenter.onViewCreated();
        verify(interactor).execute(
                ArgumentMatchers.<Consumer<NewsGeonetResponse>>any(),
                throwableConsumerCaptor.capture()
        );

        throwableConsumerCaptor.getValue().accept(new Throwable());
        verify(view).showError();
    }
}
```

Iff there is an error, I want to display an error message. Also, when the view is about to be destroyed, I want to cancel the request in case it is still pending. If you apply the steps we described before, `MainView` should now look as follows.

```java
public interface MainView {
    void hideLoading();

    void showError();
}
```

If you managed to make your tests pass, `MainPresenter` should be resemble the following:

```java
public class MainPresenter {
    private final MainView view;
    private final GetNewsInteractor interactor;

    public MainPresenter(MainView view, GetNewsInteractor interactor) {
        this.view = view;
        this.interactor = interactor;
    }

    public void onViewCreated() {
        interactor.execute(new Consumer<NewsGeonetResponse>() {
            @Override
            public void accept(@NonNull NewsGeonetResponse newsGeonetResponse) throws Exception {
                view.showNews(newsGeonetResponse.getNewsStories());
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(@NonNull Throwable throwable) throws Exception {
                view.showError();
            }
        });
    }
}
```

Finally, `MainFragment` should also have a new empty method.

```java
public class MainFragment extends Fragment implements MainView {

    @Inject
    MainPresenter presenter;

    @Override
    public void onAttach(Context context) {
        AndroidSupportInjection.inject(this);
        super.onAttach(context);
    }

    @Override
    public void showNews(NewsStory[] stories) {

    }

    @Override
    public void showError() {

    }
}
```

## Conclusion

By writing *Unit Tests* for our *Presenter* first, we can test and implement the behaviour of our application without writing a single line of UI code. This allows us to focus on one aspect of our application at a time. Also, our feedback loop shortens as we don’t need to deploy our application to a device or an emulator to verify it is doing the right thing. Implementing the methods in our *View* should be trivial, given each of them fulfills a very specific purpose as a byproduct of applying both *TDD* and *MVP*.

This concludes my series of articles on the *Mode-View-Presenter* architecture and *Test Driven Development* applied to *Android*. I hope some of these ideas help you make your development process more enjoyable.
