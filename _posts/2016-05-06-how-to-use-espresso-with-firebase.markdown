---
layout: post
title: "How to use Espresso with Firebase"
date: 2016-05-06 18:19:55
categories: testing espresso
---

I recently started using Espresso, [A UI testing library by Google][espresso-website] to write automated tests 
for my apps and I have to say that even though writing tests was a tedious task at the beginning, when I started to
understand how it works the time I spent writing those tests helped me avoid hours of manual testing.

In this article with are going to write tests for an app using Firebase.
If you don't know what Firebase is, Firebase is a service that provide a backend
for app developers. They offer a database for app data, user authentication, static
hosting and many [more][firebase-website].

### Setting up our project

I will only show the bare minimum to get Espresso and Firebase up and running. You can find more details on [The Espresso Website][espresso-website]
and [Firebase Website][how-to-setup-firebase]

* __Add espresso to your project's gradle dependencies:__
{% highlight groovy %}
android {
    ...
    dependencies {
        // JUnit4
        testCompile 'junit:junit:4.12'
        
        // Espresso Core library
        androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
        
        // Espresso Idling library (Used to make Espresso "Wait" for custom events"
        androidTestCompile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'
        
        // Android Support Annotations
        androidTestCompile 'com.android.support:support-annotations:23.+'
        
        // The Android Test Runner
        androidTestCompile 'com.android.support.test:runner:0.5'
    }
    ...
}
{% endhighlight %}

* __Set the Android JUnit4 Test Runner as your Test Instrumentation Runner__
{% highlight groovy %}
android {
    ...
    defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    ...
}
{% endhighlight %}

* __Add the Firebase library to our project's dependencies__

{% highlight groovy %}
android {
    ...
    dependencies {
        compile 'com.firebase:firebase-client-android:2.5.2'
    }
    ...
}
{% endhighlight %}

__Setup Firebase in your Application class__

{% highlight java %}
public class MyApp extends Application {

    private static MyApp instance;

    @Override
    public void onCreate() {
        super.onCreate();
        instance = this;
        
       Firebase.setAndroidContext(this);
    }
}
{% endhighlight %}

### Writing our test class

Here is the source code for our test
{% highlight java %}
@RunWith(AndroidJUnit4.class)
public class FirebaseTest {

    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<MainActivity>(MainActivity.class);

    @Test
    public void testFirebase() {

        final FirebaseOperationIdlingResource pushIdlingResource = new FirebaseOperationIdlingResource();
        Espresso.registerIdlingResources(pushIdlingResource);

        final String item = "Hello World";

        Firebase firebase = new Firebase("https://myapp.firebaseio.com");
        firebase.push().setValue(item, new Firebase.CompletionListener() {
            @Override
            public void onComplete(FirebaseError firebaseError, Firebase itemRef) {
                itemRef.addListenerForSingleValueEvent(new ValueEventListener() {
                    @Override
                    public void onDataChange(DataSnapshot dataSnapshot) {
                        pushIdlingResource.onOperationEnded();
                        assertEquals(item, dataSnapshot.getValue(String.class));
                    }

                    @Override
                    public void onCancelled(FirebaseError firebaseError) {
                        firebaseError.toException().printStackTrace();
                        pushIdlingResource.onOperationEnded();
                    }
                });
            }

        });
        pushIdlingResource.onOperationStarted();

        Espresso.unregisterIdlingResources(pushIdlingResource);
    }

}
{% endhighlight %}

And now the code for our FirebaseOperationIdlingResource
{% highlight java %}
public class FirebaseOperationIdlingResource implements IdlingResource {

    private boolean idleNow = true;
    private ResourceCallback callback;

    @Override
    public String getName() {
        return "FirebaseOperationIdlingResource";
    }

    public void onOperationStarted() {
        idleNow = false;
    }

    public void onOperationEnded() {
        idleNow = true;
        if (callback != null) {
            callback.onTransitionToIdle();
        }
    }

    @Override
    public boolean isIdleNow() {
        return idleNow;
    }

    @Override
    public void registerIdleTransitionCallback(ResourceCallback callback) {
        this.callback = callback;
    }
}
{% endhighlight %}

Now let's break down what is actually happening

During ```testFirebase()``` we first initialize a ```FirebaseOperationIdlingResource```
{% highlight java %}
final FirebaseOperationIdlingResource pushIdlingResource = new FirebaseOperationIdlingResource();
Espresso.registerIdlingResources(pushIdlingResource);
{% endhighlight %}

```FirebaseOperationIdlingResource``` is a subclass of [```IdlingResource```][idling-resource-reference].
By default Espresso executes all operations in the UI Thread and synchronize operations using ```AsyncTasks```.
Espresso does not know how to include custom tasks inside its work so queue so an ```IdlingResource``` tells Espresso when
to "wait" for an operation to be executed and when to "resume" when the operation is done. Espresso will call
```IdlingResource.isIdleNow()``` to know when to resume and when to pause. We will use ```onOperationStarted()```
and ```onOperationEnded()``` to update the value returned ```isIdleNow()```

{% highlight java %}
public void onOperationStarted() {
    idleNow = false;
}

public void onOperationEnded() {
    idleNow = true;
    if (callback != null) {
        callback.onTransitionToIdle();
}
   
@Override
public boolean isIdleNow() {
   return idleNow;
}
{% endhighlight %}

The next step is to push the data to the Firebase Server
{% highlight java %}
firebase.push().setValue(item, new Firebase.CompletionListener(){...})
pushIdlingResource.onOperationStarted();
{% endhighlight %}

We call ```onOperationStarted()``` right after sending the data to tell Espresso to wait for our network operation to complete. Without this call
Espresso would exit right away since no test would have been executed

Then we call ```onOperationEnded()``` inside our ```Firebase.CompletionListener``` instance before calling ```assertEquals()```;

{% highlight java %}
itemRef.addListenerForSingleValueEvent(new ValueEventListener() {
 @Override
 public void onDataChange(DataSnapshot dataSnapshot) {
  pushIdlingResource.onOperationEnded();
  assertEquals(item, dataSnapshot.getValue(String.class));
 }

 @Override
 public void onCancelled(FirebaseError firebaseError) {
  firebaseError.toException().printStackTrace();
  pushIdlingResource.onOperationEnded();
 }
});
{% endhighlight %}

Calling ```onOperationEnded();``` before calling ```assertEquals()``` will resume the test execution queue.

The final step is to unregister the ```IdlingResource``` when we are done with it
{% highlight java %}
Espresso.unregisterIdlingResources(pushIdlingResource);
{% endhighlight %}

Even though ```unregisterIdlingResources()``` is called below ```onOperationStarted()``` it will not be executed after that method.

You can find a the full source code on [Github][sample-source-code]

### Useful Links
* [Espresso Website][espresso-website]
* [Firebase Website][firebase-website]

[espresso-website]: https://google.github.io/android-testing-support-library/docs/espresso/index.html
[firebase-website]: https://www.firebase.com/
[idling-resource-reference]: http://developer.android.com/reference/android/support/test/espresso/IdlingResource.html
[sample-source-code]: https://github.com/charly1811/sample-espresso-firebase
[how-to-setup-firebase]: https://www.firebase.com/docs/android/quickstart.html