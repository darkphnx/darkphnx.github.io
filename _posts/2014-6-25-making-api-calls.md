---
layout: post_page
title: Making API calls from your Android activity
tags: Android, AsyncTask, Retrofit, GSON
---

In the [last post]({%post_url 2014-06-24-consuming-apis-with-android %}) we looked at how to fetch data from a remote API and turn that into some Java objects that we can use. In this post we'll be looking at how you can use these API calls from our actvity.

An activity should have been created when you started your project. If not, ask your IDE to create you a new Activity. Open it up and you should have something that looks like this:

{% highlight java %}
public class Output extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_output);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.output, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();
        if (id == R.id.action_settings) {
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
 {% endhighlight %}
    
You can ignore `onCreateOptionsMenu` and `onOptionsItemSelected`. They're created automatically with the object.

The first thing we want to do is create an instance of our API class and store it as a member variable so that it can be accessed by the methods in our class. Modify your code to do this:

{% highlight java %}
public class Output extends Activity {
	// Our API client
	protected ApiClient.SirportlyApiInterface mApiClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_output);
        
        // Use our API client builder to get us a new instance of the API client
        mApiClient = ApiClient.getSirportlyApiClient();
    }
    ...
}
{% endhighlight %}

Notice that we've called our variable `mApiClient`. This is a convention used by java programmers to indicate that a variable is a _member_ variable. You might think that you can go ahead and start making API calls now in your `onCreate` method, however this will raise a `NetworkOnMainThreadException` exception. Android doesn't allow you to make blocking network calls from the UI thread, since this makes the user interface unresponsive. Instead, we must make our calls on another thread. Enter the [AsynTask](http://developer.android.com/reference/android/os/AsyncTask.html) class.

The `AsyncTask` class allows us to run tasks in a background thread. There are three methods that we can override in AsyncTask. The first method is the task to be performed and is called `doInBackground`. The second is `onProgressUpdate` and can update the UI thread when progress is made. The final method is called `onPostExecute` and will receive the return value of `doInBackground`. Let's define our task now. Place this inside your Activity class.

{% highlight java %}
public class Output extends Activity {
    ...
    private class GetFiltersTask extends AsyncTask<Void, Void, List<Filter>>{
        protected List<Filter> doInBackground(Void... params){
            List<Filters> filters = mApiClient.getFilters();
            return filters;
        }

        protected void onPostExecute(List<Filter> filters){
            String logMsg = "Got " + filters.size() + " filters back from the API";
            Log.d("ApiResults", logMsg);
        }
    }
}
{% endhighlight %}

You'll notice that AsyncTask has three types defined in the extends statement. These represent the types of data your task will be working with. The first is the type of data you'll be giving to `doInBackground` as a parameter. The second is the type of data that you'll be sending to `onProgressUpdate`, the third type is the data you're returning from `doInBackground` and feeding into `onPostExecute`. In this case we're not passing in any parameters, so we mark this as `Void`, and we aren't going to keep track of the progress, sice it probably won't take very long, so we'll mark this as `Void` too. Finally, our return type from `doInBackground` will be a `List<Filter>`, so we'll add this as our last type.

Our `doInBackground` method will now go ahead and call the API in a background thread using the client we defined earlier. When it's completed it'll pass it's results to `onPostExecute`. All this does at the moment is count up the number of items we got from our list and logs them to the Logcat.

Now that everything is defined, we're ready to make our API first API call. Head back to your `onCreate` method and add the following line to the bottom:

{% highlight java %}
public class Output extends Activity {
	// Our API client
	protected ApiClient.SirportlyApiInterface mApiClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_output);
        
        // Use our API client builder to get us a new instance of the API client
        mApiClient = ApiClient.getSirportlyApiClient();
        
        // Call our SyncTask to fetch a list of filters in the background
        new GetFiltersTask().execute();
    }
    ...
}
{% endhighlight %}

Fire up your app in the Android emulator, or on your own phone and you shouldn't see anything happen. However, if you check your logs you should see the message "Got n filters back from the API". Success!

In the next post we'll look at how we can use the returned list of filters and display them in a ListView.