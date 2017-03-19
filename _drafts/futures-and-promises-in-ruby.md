---
title:  "Future and Promises in Ruby"
date:   2017-03-18 14:32:01 IST
tags: ruby concurrent-ruby promises future
---

1. problem ?
2. promises to rescue !
3. example

Consider an application which needs to process nested background task. By nested
I meant if a background task is complete then another task should start
or some final operation once the task is complete. With bare threads library, there is no cleaner way to achieve this.

Promises framework provides a way to run nested background task asynchronously. Once the first task is complete, another executes
without blocking the main thread. There is also a callback for exception handling in the tasks being executed.

{% highlight ruby %}
Concurrent::Promises.future { 'do something' }
.then { 'do something else' }
.rescue { |error| handle_exception error }
{% endhighlight %}

Lets assume we are listening delivery complete event of all kinds of delivery of
goods.

{% highlight ruby %}
module ProcessEvent
    def self.process event
        # something
    end
end

module ProcessDeliveryCompleteEvent < ProcessEvent
end
{% endhighlight %}

Abstracting the receiving part from processing event, lets create another module
for handling the event when it is received by the poller.

{% highlight ruby %}
module EventHandler
    def self.handle event
        # after some processing on event
        ProcessDeliveryCompleteEvent.process event
    end
end
{% endhighlight %}

Now we start polling in background using Promises
{% highlight ruby %}
Concurrent::Promises.future do
    Provider.poll do |event|
        Concurrent::Promises.future{ EventHandler.handle event }.
        then { log(post_event_processing) }.
        rescue { |error| log(error) }
    end
end.rescue { |error| log(not_listening_to_event_anymore) }
{% endhighlight %}
