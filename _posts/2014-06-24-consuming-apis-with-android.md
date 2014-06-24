---
layout: post_page
title: Consuming APIs on Android using Retrofit and GSON
tags: Android
---

Many apps will at some point need to interact with a web API, coming from a Ruby/Rails world, a lot of the apps I write will be interacting with my own APIs. I've written this post as a guide to help myself remember the process, and hopefully help someone else out along the way.

Although the included HTTP client isn't bad, I've chosen the excellent [Retrofit](http://square.github.io/retrofit/) library for making my calls. Retrofit handles the connection and request, and passes the returned data to [GSON](https://code.google.com/p/google-gson/) for parsing and turning into Java objects.

The API I've been interacting with is for [Sirportly](http://www.sirportly.com) which can involve some pretty deeply nested data structures, but I'll try to use simplified examples.

I've split my application into a couple of packages, `api` and `models`. The `api` package will contain the code which interacts with the webservice, while the `models` package will contain Java representations of your data. Let's start with the `api` package. You should also have an activity that was generated when you created your project in Android Studio or Eclipse, we'll use that activity for testing our calls.

Create a new class in the `api` package. We'll call it `ApiClient`.  This class is going to create us a service which we can use to make API calls.

{% highlight java linenum %}

public class ApiClient {
    public static final String TAG = ApiClient.class.getSimpleName();
    private static SirportlyApiInterface sSirportlyApiService;

    public static SirportlyApiInterface getSirportlyApiClient(){
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.sirportly.com/api/v2/")
            .setLogLevel(RestAdapter.LogLevel.FULL)
            .setLog(new RestAdapter.Log() {
                @Override
                public void log(String msg) {
                    Log.d(TAG, msg);
                }
            })
            .build();
      }
}

{% endhighlight %}