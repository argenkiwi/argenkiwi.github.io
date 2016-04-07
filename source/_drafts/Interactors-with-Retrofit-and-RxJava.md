title: 'Android: Interactors with Retrofit and RxJava'
date: 2016-04-07 15:48:51
tags:
- MVP
- Retrofit
- RxJava
---
In a previous article I described how to structure your application under the Model-View-Presenter architecture applying dependency injection with Dagger. I covered how to setup a _View_ and its _Presenter_. Today I would like to share with you how my _Presenter_ 'talks' to the _Model_ using _Interactors_.

Setup
-----
For this example, I am going to introduce Retrofit, a networking library that makes it easy to define endpoints from which to retrieve data from the network. Also, I will make use of _RxJava_ (and _RxAndroid_) to handle asynchronous requests. Let’s update the dependencies in the our application's `build.gradle` file.

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

We added an _RxJava_ adapter to enable _Retrofit_ to return `Observable` objects from its _Services_. We included a _GSon_ converter only because the endpoint we are going to use in this example uses the _JSON_ format for its responses.

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

We need to create _Model_ objects to represent the response and the _News Stories_ in the feed.

Let's create a _Service_ for it.

```Java
public interface GeonetService {
    @GET("news/geonet")
    void getNews(Callback<NewsGeonetResponse> callback);
}
```
