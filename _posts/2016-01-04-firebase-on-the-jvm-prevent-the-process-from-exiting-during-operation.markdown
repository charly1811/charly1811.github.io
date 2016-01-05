---
layout: post
title:  "Firebase on the JVM: Prevent the process from exiting during operations"
date:   2016-01-04 22:06:00 -0500
categories: java firebase
---

In a nutshell Firebase is a backend service for mobile app. 
It offers everything you need to create a cloud experience for your app (data storage, user authentication, static hosting, and more)
The firebase team provide SDKs for Android, iOS and the web

The firebase SDKs for Android and JVM works almost exactly the same. 
The only difference is that on JVM API calls are made in daemon threads therefore the main thread of you program will close right after you call the firebase methods. 

[The Firebase documentation][firebase-docs] suggests to a the Semaphore or CountDownLatch object to prevent the process from exiting during operation.
Here is my solution inspired by the suggestion from the Firebase documentation.

### The wrapper class
I create a class contains an instance of the Firebase object for my app and methods for each operation
I might use (getDataSnapshot(), remove(), setValue()).

{% highlight java %}
public class FirebaseOperationManager {
    private firebase = new Firebase("https://myApp.firebaseio.com/");
    private ExecutorService tasksExecutor = Executors.newCachedThreadPool();
    
    public void getDataSnapshot(String path,  OperationResult<DataSnapshot> operationResult)
    public void setValue(String path,  OperationResult<Firebase> operationResult)
    public void remove(String path,  OperationResult<Firebase> operationResult)
    
}
{% endhighlight %}

{% highlight java %}
public interface OperationResult<T> {
    void onResult(T result);
    void onError(FirebaseError firebaseError);
}
{% endhighlight %}


```OperationResult<T>``` is a callback interface to be notified when the operation has been successfully executed or when there is an error.
We also need an ```ExecutorService``` to execute each operation asynchronously

### Structure of the wrapper's method
To properly explain the structure of the wrapper methods here is an example of a method that retrieve the ```DataSnaphost``` of a value
in a Firebase database

{% highlight java %}
public void getDataSnapshot(final String path, final OperationResult<DataSnapshot> operationResult) {
    final CountDownLatch done = new CountDownLatch(1);
    tasksExecutor.submit(new Runnable() {
        @Override
        public void run() {
       // Start the firebase operation
       firebase.child(path).addListenerForSingleValueEvent(new ValueEventListener() {
       @Override
       public void onDataChange(DataSnapshot dataSnapshot) {
           operationResult.onResult(dataSnapshot);
           
           // Resume the worker thread
           done.countDown();
       }

       @Override
       public void onCancelled(FirebaseError firebaseError) {
           operationResult.onError(firebaseError);
           
           // Resume the worker thread
           done.countDown();
       }
        });
       
       // Pause the worker thread
       try {
       done.await();
        } catch (InterruptedException e) {
       e.printStackTrace();
        }
    }
});
{% endhighlight %}

The method flow to retrieve the ```DataSnapshot``` is executed in a worker thread and the this thread is paused
until the operation terminated or when there is an error

Now this is how the method would be used

{% highlight java %}
databaseManager.getDataSnapshot(offersDir, new OperationResult<DataSnapshot>() { 
   @Override 
   public void onResult(DataSnapshot result) { 
      // TODO: Do something with the data
   } 
 
   @Override 
   public void onError(FirebaseError firebaseError) { 
      firebaseError.toException().printStackTrace();
   } 
}); 
{% endhighlight %}

[firebase-website]: https://www.firebase.com/docs/android
[firebase-docs]: https://www.firebase.com/docs/android/guide/saving-data.html#section-other-runtimes
