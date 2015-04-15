---
layout: post_page
title: "Managing Threads with Queue"
tags: Ruby, Threading, Queue
---

One of my favourite utilities in the Ruby standard library is the `Queue` class. Queue provides you with a delightfully simple way to synchronise communication between threads. Typically, Queue is used to manage the work assigned to a thread pool. We'll look at some of the ways it assists us in this endeavour.  

Queue has two main methods in it's API, `push` and `pop`. These tow methods do exactly what you'd expect, push adds an item to the queue, and pop retrieves an item. By default, if you call `pop` in your thread and the queue is empty, it will block until an item becomes available. Multiple threads will be able to pop items off of the queue in a safe manner. Let's have a look in a simple example:

{% highlight ruby %}
queue = Queue.new
threads = []

# Start 2 threads to pop work off of the queue and print it, then sleep 3 seconds for effect
2.times do
  threads << Thread.new do
    loop do
      queue_item = queue.pop
      puts queue_item.inspect
      sleep 3
    end
  end
end

# Add 5 items to the queue to be processed by the threads above
5.times do |n|
  queue.push "Item #{n}"
end
{% endhighlight %}

Running this example you'll see that the threads pick up an item each off of the queue, print it off and then sleep for 3 seconds before picking up the next item from the queue. Once the queue is empty, they'll block on queue.pop until a new item becomes available for processing.

Queue also comes with a couple of extra handy methods to help you manage the workload for your threads. `#length` tells you how many items are currently queued, `#num_waiting` lets you know how many threads are currently blocking on pop, waiting for input and `#clear` allows you to remove any items from the queue currently waiting to be processed. In particular, you can use the last 2 methods to build a clean shutdown into your script, so that all of your threads exit gracefully once they've finished their current work.

{% highlight ruby %}
trap "INT" do
  # Remove any queued jobs from the list
  queue.clear
  
  # Wait until any currently running threads have finished their current work and returned to queue.pop
  while queue.num_waiting < threads.count
    pending_threads = threads.count - queue.num_waiting
    puts "Waiting for #{pending_threads} threads to finish"
    sleep 1
  end
  
  # Kill off each thread now that they're idle and exit
  threads.each(&:exit)
  Process.exit(0)
end
{% endhighlight %}

We utilise the Queue class in a couple of places throughout our apps, we use it in Deploy to send files to multiple servers simultaneously and it's used in some internal scripts where parallel processing is advantageous.
