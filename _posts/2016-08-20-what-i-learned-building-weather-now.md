---
layout: post
title: "What I learned when building Weather Now"
date: 2016-08-20 13:45:00
categories: android
description: Lorem Ipsum
---

My Android development flow was really random when I released [My first Android app][simple-currency-converter].
The app was build to just work and wasn't testable. I wanted to learn a better way to build Android app using 
the new Android libraries and tools so I decided to build [Weather Now] [weather-now-google-play] 
a simple weather app that connect to Forecast.io and display the weather to the user. I am going to share all the
resources I used to build this app.

#### Libraries I used
* [Retrofit][retrofit-website]: Networking library that parses HTTP Response and return Java Objects
* [Rx Java][rx-java-website]: An asynchronous programming library.
* [Gson][gson-website]: Library to serialize / deserialize java objects using Json

#### Tips and Trics

__Tips.get(0) == Test your code__

Testing your code is very imporant because it helps you find bugs early in the development process. Testing your code
also allows you to be sure that your code works even if you refractor it in some way. I used [this article][rebecca-franks-testing-article] by Android GDE
[Rebecca Franks][rebecca-franks-website] as an inspiration


__Tips.get(1) == Use a pattern__

It doesn't matter if you use [MVVM][mvvm-wikipedia], [MVC][mvc-wikipedia] or MVCXMVVM. Using a development pattern will help you 
keep your code clean, maintanable and easy to test.

__Tips.get(2) == Use Loaders to execute business logic and display the result on UI__

Android has this very nice class called [Loader][loader-javadoc]. A Loader is basically a class that loads data asynchronously
and returns the result to the Main Thread. You can use loaders to load some data and send it to the UI when it is ready. 
The cool thing about loaders is that they survive activity lifecycle so even if your activity is recreated it is going to listen
for changes from the previous loader you created. You can get more info on loader by reading [this very nice article][loaders-medium]
by Ian Lake. I am also going to write an article showing an __example of using Loaders with RxJava__

__You can download Weather Now on the [Google Play Store][weather-now-google-play] and feel free to leave your in the comments section here !__

[mvc-wikipedia]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[mvvm-wikipedia]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel
[loader-javadoc]: https://developer.android.com/reference/android/content/Loader.html
[loaders-medium]: https://medium.com/google-developers/making-loading-data-on-android-lifecycle-aware-897e12760832
[retrofit-website]:http://square.github.io/retrofit
[android-dev-testing]: https://developer.android.com/training/testing/start/index.html
[rx-java-website]: https://github.com/ReactiveX/RxJava
[gson-website]:https://github.com/google/gson
[jsonschema2pojo_org]: http://jsonschema2pojo.org
[weather-now-google-play]: https://play.google.com/store/apps/details?id=io.github.charly1811.weathernow&hl=en
[simple-currency-converter]: https://play.google.com/store/apps/details?id=cf.charly1811.android.simplecurrencyconverter&hl=en
[rebecca-franks-testing-article]: https://riggaroo.co.za/introduction-automated-android-testing
[rebecca-franks-website]: https://riggaroo.co.za/female-android-developer