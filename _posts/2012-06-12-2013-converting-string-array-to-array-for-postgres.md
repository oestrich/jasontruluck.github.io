---
layout: post
title:  "Converting String-Arrays to Arrays in Ruby"
date:   2013-05-29 17:48:40
categories: ruyb array string optimization tricks postgres
---
A quick trick that came to mind recently for converting string arrays to arrays
was something that seems to come up quite a bit. Just to define what I am
talking about, when I say "string-arrays" vs arrays I mean something like this: 

{% highlight ruby %}
"[1,2,3,4]" #<- String that contains an Array (String-Array)
[1,2,3,4]   #<- Normal Array
# Or:
"['j','k','l','m']" #<- String that contains an Array (String-Array)
['j','k','l','m']   #<- Normal Array
# etc....
{% endhighlight %}

The most frequent time I have seen something like this is within something like
a Postgresql HStore hash. Since the hash will not allow storing anything besides
a string storing an array will automatically be converting into an string form
of that array. Of course it could be argued that this is not the most "correct"
way to accomplish your goal and yada yada but thats not what we are here for
today. There are many instances where you may already have this going on or have
your reasons so heres some tips!

- Treat the string as an array for fast conversion. This works well if you know
  the format of the string and know it will be consistent. 

  `"[1,2,3,4]"[1..-2].split(",")`<br/>
  
  This will gather the items within the brackets using the standard method of
  grabbing items from an array using a range `[1..-2]` and then split the
  remaining string which is `1,2,3,4` by a delimeter you speciy (in this case `,`) leaving you with `[1,2,3,4]`

- Another method is using gsub to remove specfic characters than splitting the
  elements like so:

  `"[1,2,3,4]".gsub("[", "").gsub("]", "").split(",")`<br/>

  This will remove the brackets using `gsub` then split the remaining string using
  `split`. This method has the benefit of flexibility over the other method, you
  can convert string arrays of non standard format like `" [1,2,3,3]"` which
  contains a space but at the cost of speed. You could also throw things like
  `.compact` into the mix to remove that space as well but you can decide. 

### Benchmarks

As far as speed is concerned the first method is far faster even on large data
sets. I made a quick script to benchmark the two methods that is available
[here](https://gist.github.com/jasontruluck/5762470). But a glimpse of the
results is as such:


{% highlight ruby %}
# Using "[1,2,3,4]"[1..-2].split(",")
# user     system      total        real
0.000000   0.000000   0.000000 (  0.001216) # 1000 items
0.010000   0.000000   0.010000 (  0.011726) # 10000 items 
0.110000   0.000000   0.110000 (  0.110641) # 100000 items 
1.100000   0.000000   1.100000 (  1.105512) # 1000000 items

# Using "[1,2,3,4]".gsub("[", """]").gsub("", "]").split(
# user     system      total        real
0.010000   0.000000   0.010000 (  0.011399)  # 1000 items
0.110000   0.000000   0.110000 (  0.112650)  # 10000 items
1.090000   0.000000   1.090000 (  1.093740)  # 100000 items
11.720000   0.040000  11.760000 ( 11.754000) # 1000000 items
{% endhighlight %}

As you can see the later increases in time alot as the amout of items
inscreases.

