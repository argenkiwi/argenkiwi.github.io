title: Setup Dagger in Android.
date: 2016-04-02 09:54:36
tags:
- Dagger
- Android
---
Nowadays I use Dagger, the dependency injection library originally created by Square and later adopted by Google, in most of my projects. Every time I have to set it up on a new project I struggle to find a quick guide to remind me of the minimun number of steps I need to follow to get it running. Therefore, I decide to write one.

Fist, let's update our project's `build.gradle` file to include `android-apt` as one of its dependencies:
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

Next, update you app's `build.gradle` to include the `android-apt` plugin and the dependecies for _Dagger_ and _JavaX annotations_.

```Java
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

android {
  ...
}

dependencies {
    ...
    compile 'com.google.dagger:dagger:2.1'
    provided 'org.glassfish:javax.annotation:10.0-b28'
    apt 'com.google.dagger:dagger-compiler:2.1'
}
```
