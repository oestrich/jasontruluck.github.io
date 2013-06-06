---
layout: post
title:  "Conditional Exclusion of Tests in Rspec"
date:   2013-06-06 17:48:40
categories: testing rspec tricks
---
[Rspec](https://github.com/dchelimsky/rspec) is a great tool that offers an awesome DSL for testing. I use it very
frequently in both projects at work and personal projects. This is a handy trick
that I picked up to conditionally exclude tests in Rspec. 

Rspec provides alot of awesome filters that allow control over how and when
tests are run namely the `filter_run_excluding` configuration that will allow
you to exclude certain tests from the current run if they have a tag set on
them. For example, given this spec_helper:

{% highlight ruby %}
# spec/spec_helper.rb
RSpec.configure do |config|
  config.filter_run_excluding :type => 'feature'
end
{% endhighlight %}

you can exclude a certain test by including the tag `feature` like so:

{% highlight ruby %}
describe 'some slow test', :type => :feature do
  # Some test content
end
{% endhighlight %}

this will allow us to skip this test, which is great if the test is slow and
unnecessary to run every time. The down side to this approach is that it
requires that when you want to run your entire suite you will have to remove
these filters to run them. A simple way to get a round this is like so: 

{% highlight ruby %}
# spec/spec_helper.rb
RSpec.configure do |config|
  config.filter_run_excluding :type => 'feature' unless ENV[ALLOW_FEATURE]
end
{% endhighlight %}

the environment variable allows you toggle when to use this exclusion filter. So
executing the test run like so:

{% highlight ruby %}
ALLOW_FEATURE=true rspec
{% endhighlight %}

Will allow you to run your entire suite including the tests tagged with
`feature`. This is really useful on CI servers where running all the tests is
important. Check out more on Rspec exclusion filters on their [relish docs](https://www.relishapp.com/rspec/rspec-core/v/2-14/docs/filtering/exclusion-filters). Happy coding!
