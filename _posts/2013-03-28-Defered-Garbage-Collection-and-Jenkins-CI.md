---
layout: post
title:  "Deferred Garbage Collection and Jenkins CI"
date:   2013-05-29 17:48:40
categories: jenkins ci testing optimization
---
After finding the helpful [article][dgc-article] on deferred garbage collection from the
awesome ariejan.net I immediately put it into practice. After all, who can
resist nearly free test optimization! Everything worked perfectly. I was seeing
the test time cut nearly in half score! 

This was going great but almost immediatly I started to see some intermittent
failures on my Jenkins CI server. The problem was very perplexing (although now
painfully obvious), I was getting only an error that looked like: 

{% highlight ruby %}
.................Killed
{% endhighlight %}

It was pretty apparent that the process was being reaped for one reason or
another but I could not out my finder on it since it seemed almost random where
it would be reaped. It was not tied to a specfic test or even an entire batch of
them. It was not untill I remembered the deferred garbage collection. Right then it hit me like a ton of bricks, of course! I am
preventing the garbage collector from cleaning up after my code in memory so it is
destroying my available memory. 

Since the linode I am using for my personal Jenkins CI server has only a poultry
1 GB of memory and 512mb swap this was being consumed in no time where as on my
local machine with a nice stack of 16gb there was no issue. Typically the
testing without defering garbage collection would register only about 200mb of
memory usage, but after defering it was in excess of 1.4GB. Obviously outside
the capacity of my poor linode. 

From here there were a few options I could take: 

- Optimize the garbage collection in hope of keeping under my memory limits but
  still worth defering 
- Increase the available memory on my Linode 
- Ignore defering garbage collection altogether when running tests on Jenkins

Optimizing the garbage collection would not be too difficult as stated in the
article above you can just tweak the threshold and see if the benefits still
fit. Pretty straight forward, but potentially time consuming.

Increasing the abailable memory on my linode was the other option but not one I
wanted to jump into. Vertically scaling the linode server that was moderately used at
best by only myself was a bit unnecessary. Not to mention who wants to pay more
money just to run some tests? But if you are into that sort of thing its an easy
fix.

The route I went was just to not defer the garbage collecting at all. My
rational here is that I can afford to wait a bit longer running tests here - this is
only a CI server after all - and minimizing the memory footprint would help so I
do not have to tweak little settings every time I add some test. 

To accomplish this is incredibly simple. In your spec helper just wrap the
deferred garbage collection script in an if statement where providing an
environment variable will prevent defering the garbage collection. 

{% highlight ruby %}
config.before(:all) do
  DeferredGarbageCollection.start unless ENV["SKIP_DGC"]
end

config.after(:all) do
  DeferredGarbageCollection.reconsider unless ENV["SKIP_DGC"]
end
{% endhighlight %}

then in your CI script instead of 

{% highlight ruby %}
rake rspec 
{% endhighlight %}

just use

{% highlight ruby %}
SKIP_DGC=true rake rspec
{% endhighlight %}

This way you can bask in the speedy glory that you get from running the tests with derfering
on your local machine and all you have to do is throw in a flag on your CI
script to prevent Jenkins from eating itself. 

[dgc-article]: https://ariejan.net/2011/09/24/rspec-speed-up-by-tweaking-ruby-garbage-collection/
