---
layout: post
title:  "How to tint any Drawable on Android"
date:   2015-10-29 22:06:00
categories: android
---

Android lollipop added new features related to drawables. 
One of this capabilities is to “tint” any drawable using the Drawable.setTint() method. 
Unfortunately this method is only available for Android Lollipop and above.
After many research on the Internet I finally found the solution to tint any drawables on any API level.
The Android documentation hide many secrets and I think developers (Me particularly) should take the time to read it a little bit.


The hidden secret I am talking about is the [DrawableCompat][drawable-compat-doc] class. 
To tint drawables using this class you can use the following code

{% highlight java %}
Drawable mDrawable = DrawableCompat.wrap(getMyDrawable()); 
DrawableCompat.setTint(mDrawable,mColor); 
useMyDrawable(mDrawable)
{% endhighlight %}

First we wrap the drawable using DrawableCompat.wrap() to make it “tintable”. 
Then we call the DrawableCompat.setTint() to change the color of the drawable

[drawable-compat-doc]: https://developer.android.com/reference/android/support/v4/graphics/drawable/DrawableCompat.html