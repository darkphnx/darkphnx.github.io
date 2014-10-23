---
layout: post_page
title: CoffeeRoutes - Rails named routes in Coffeescript
tags: Ruby, Rails, Coffeescript
---

Fairly frequently I find myself in the posiiton that I need to make an AJAX call to my rails app to send or fetch some piece of data or another. To do this I need to know which URL to hit, and which HTTP verb to use to make the request.

If I'm submitting a form via AJAX, that's no problem at all. Both the url and the method are right there in the form declaration, we can just do.

{% highlight coffee %}
form = $('form.my_form')
$.ajax
  url: form.attr('action')
  type: form.attr('method')
{% endhighlight %}

What about when we're not submitting a form though? What if we want to pull some data from a URL, or submit some data without having a form? There's a few hacks that I've used in the past, none of them are much good.

### The Workarounds
__Hack #1:__ Hard code the URL. This works brilliantly until you change the route, then you have to hunt through your coffeescript to find all of the places you've hardcoded the URL. This can be particularly annoying if you've had to concatenate in some parameters to your URL.

__Hack #2:__ Put a submission URL in a data attribute. In the cases where the URL updates a particular resource, for example getting more information about an item from a list, you can place the URL in a data attribute on that list item.

{% highlight haml %}
%ul.list
  - for item in items
    %li.list__item{:data => {:show_url => item_path(item)}= item.name
{% endhighlight %}

{% highlight coffee %}
$('li.list__item').on 'click', ->
  itemUrl = $(this).data('showUrl')
  $.ajax
    url: itemUrl...
{% endhighlight %}

__Hack #3:__ Global containing URLs. An object placed inline with on your page that contains URLs that you might need on that page request.

{% highlight haml %}
:javascript 
  window.ajaxUrls = {
    "create" : #{items_path}, 
    "poll": #{poll_items_path}
  };
{% endhighlight %}

### The Solution
The solution actually builds on Hack #3, but formalises it in a way that doesn't require hacky inline JS and presents  a nice friendly wrapper and some helper methods.

Rails has already has a nice way to print out your application's routing table, it uses it whenever you type `rake routes` or encounter a routing error. With this in mind I set about finding out how rake routes worked. Working backwards from the rake task I found `ActionDispatch::Routing::RoutesInspector` along with it's friends `ActionDispatch::Routing::HtmlTableFormatter` and `ActionDispatch::Routing::ConsoleFormatter`. Ignoring the warning at the top of the RoutesInspector "# People should not use this class." I set about building a formatter class that would output a nice JSON string of routes.

JSON string produced, now we need to inject thst into our page and build a few helper methods around the data. `coffee_routes.coffee.erb` creates a global CofeeRoutes object and gives it a parameter of `routes` which contains our JSON object.

Next the `path` function was added to CoffeeRoutes to find the named route from the #routes and substitute in any parameter varaibles that could be passed in as an object along with the name of the route. Finally, a set of global helpers is created so that you can just type `poll_item_path({"id" : 5})` for example anywhere in your coffeescript/javascript.

### Usage
1. Include `coffee_routes` in the Gemfile for your Rails app
2. Add `#= require coffee_routes`  to your application.coffee
3. Access your named routes in Coffeescript

{% highlight coffee %}
project_item_path({"project_id" : "project-awesome", "id" : 5})
=> "/projects/project-awesome/items/5"
{% endhighlight %}

### Source
Get the source on [Github](https://github.com/darkphnx/coffee_routes).