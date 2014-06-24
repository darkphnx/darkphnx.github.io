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

{% highlight java %}
public class ApiClient {
    public static SirportlyApiInterface getSirportlyApiClient(){
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.sirportly.com/api/v2/")
            .build();
      }
}
{% endhighlight %}

What we've just done is written a static method (class method for you Rubyists reading) that creates a [RestAdapter](http://square.github.io/retrofit/javadoc/retrofit/RestAdapter.html) and associates it with the base URL of our API.

Next we need to define what calls our API will be capable of making. We do this by defining an [Interface](http://docs.oracle.com/javase/tutorial/java/concepts/interface.html) with few special [annotations](http://docs.oracle.com/javase/tutorial/java/annotations/)  that describe our call. We'll define this interface internally within our `ApiClient` class, since we don't need it anywhere else.

{% highlight java %}
public class ApiClient {
    ...
    public interface SirportlyApiInterface {
        @GET("/objects/filters")
        List<Filters> getFilters();
    }
}
{% endhighlight %}

Here we've created our interface and described a GET request to `/objects/filters`. The second line in the interface describes name of the mehod that we'll be able to call for this request (`getFilters()`) and the type of data it will return (`List<Filters`).

Now we have our `RestAdapter` ready to make calls, and some calls defined in our `SirportlyApiInterface` we can join the two together. Modify your `getSirportlyApiClient()` method to look like the following:

{% highlight java %}
public class ApiClient {
    public static SirportlyApiInterface getSirportlyApiClient(){
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.sirportly.com/api/v2/")
            .build();
      }
      sirportlyApiService = restAdapter.create(SirportlyApiInterface.class);
      return sirportlyApiService;
}
{% endhighlight %}

This will return a new sirportlyApiService every time, which is not very efficient. Lets cache one client with a few small modifications. The addition of a class variable, and an if block to only create the service if it isn't already defined.

{% highlight java %}
public class ApiClient {
    // Keep a copy of our API service cached
    private static SirportlyApiInterface sirpirtlyApiService;
     
    public static SirportlyApiInterface getSirportlyApiClient(){
        // Check to see if a sirportlyApiInterface is already instaciated
        if(sirporltyApiService == null){
            RestAdapter restAdapter = new RestAdapter.Builder()
                .setEndpoint("https://api.sirportly.com/api/v2/")
                .build();
            sirportlyApiService = restAdapter.create(SirportlyApiInterface.class);
        }
        return sirportlyApiService;
    }
}
{% endhighlight %}

Now we have an object we can use to make the API calls that we've defined, we need somewhere to put the results. You'll have noticed in our interface that `getFilters()` returns an array (List) of Filter objects. We're going to want to define a `Filter` class which defines the attributes that will be returned from our API call.

In the case of a pretty simple object you should be able to craft this object by hand. However, if you have particularly complex set of return data, with lots of attributes and other nested objects I'd recommend using [jsonschema2pojo](http://www.jsonschema2pojo.org/) to convert your JSON schema or data into a Java object.  Ensure that annotation style is set to none, and that "Use Primitive Types" is checked. 

I used jsonschema2pojo to build my Filter class, since it's a pretty complex object. However, I'll present a simplified version below. Download your generated class files, or create a new class called Filter in your models package.

If you find your generated models are causing import errors, it's probably down to `javax.annotation.Generated` not being included with Android. It's no problem, just remove these tow files from the top of your model class.

{% highlight java %}
import javax.annotation.Generated;
@Generated("org.jsonschema2pojo")
{% endhighlight %}

Let's have a look at a simplified version of the Filter class that I created. It's a pretty simple job, it's only job is to store the information that comes back from our API call. We'll just need to define some member variables, and create some getters and setters.

{% highlight java %}
public class Filter {
    private int id;
    private String name;
    private List<String> cols = new ArrayList<String>();

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<String> getCols() {
        return cols;
    }

    public void setCols(List<String> cols) {
        this.cols = cols;
    }
}
{% endhighlight %}

In reality, my filter class has nearer 10 attributes along with their associated getters and setters, but we'll go with 3 to keep things simple.

Most of this should be pretty straightforward, the interesting attribute here is `cols`. We've defined it as `List<String>`, our JSON structure for cols gives us an array of column names. With the type defined as List<String> GSON will automatically convert that array into the correct type.

All the pieces are now in place to start using our classes to fetch data from the API. I know this may have felt like a lot of work for making a simple API call. But doing this work up front will allow us to decouple our activities from the dirty business of fetching and parsing data and just focus on displaying it to the user.

In the next post I'll show you how to use `sirpirtlyApiService` in your Activities and display a simple list view.