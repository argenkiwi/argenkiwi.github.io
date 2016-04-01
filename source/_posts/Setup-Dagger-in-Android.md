title: Setup Dagger in Android.
date: 2016-04-02 09:54:36
tags:
- Dagger
- Android
---
Nowadays I use [Dagger](http://google.github.io/dagger/), the dependency injection library, in most of my projects. Every time I have to set it up on a new project I struggle to find a quick guide on how to just do that. Therefore, I decide to write one.

Fist, let's update our *project's* `build.gradle` file to include [android-apt](https://bitbucket.org/hvisser/android-apt) as one of its dependencies:
```Java
...
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
        ...
    }
}
...
```

Next, update you *app's* `build.gradle` to include the `android-apt` plugin and the dependecies for _Dagger_ and _JavaX annotations_.

```Java
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

android {
  ...
}

dependencies {
    ...
    compile 'com.google.dagger:dagger:2.1'
    apt 'com.google.dagger:dagger-compiler:2.1'
}
```

*Note:* As I was writing this post I realised I could remove a dependency that was previously needed to make _Dagger_ work on _Android_. If you have trouble compiling you project, try adding the following to your dependencies:

```Java
dependecies {
  ...
  provided 'javax.annotation:javax.annotation-api:1.2'  
}
```

The `@Inject` annotation used by _Dagger_ depends on this library.
