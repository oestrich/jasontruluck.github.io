---
layout: post
title:  "(Opinonated) Jenkins CI Installation Script"
date:   2013-07-13 15:48:40
categories: ruby jenkins ci automation
---
Recently I had to reinstall our setup for our Jenkins CI box and it made me
realize how much I did not want to have to go through this again. Don't get me
wrong, I think Jenkins is awesome and overall pretty easy to get up and running
but hanging around for a few hours installing and configuring packages that we
need to get our basic environment up and running is not my idea of a fun day.
After all, we would all rather be coding right?!

So, of course my first thought was to automate it. Personally I love automation
and you should too, its the best thing since recursion! Anyways... this is a
brief overview of the script I slapped together and I hope it can help others
out there. It is of course pretty opinionated as far as frameworks and
technologies go but I tried to make it fairly flexible to accomodate different
things, feel free to fork away.

###What it does:
####Installs
* Jenkins CI
* RVM
* Ruby (defaults to 1.9.3 and 2.0.0)
* Postgresql 9.2
* Phantom JS 1.9.0
* Redis 2.4.16
* Jenkins CI Plugins (Git, Git Client, Github API, Github OAuth)
####Sets up basic security
* adds a deploy user
* sets up SSH key pair for authentication
* disables SSH password authentication
* disables root login
* creates firewalls settings in IpTable (Blocks all ports but 80. 8080, and 22)
* installs fail2ban with default configuration

The script tries to be minimally interactive but since somethings are better
with user input you will have to pay attention for things like adding users.
To get everything going just ssh into your box and do the following:

{% highlight ruby %}
  vim setup.sh
{% endhighlight %}

paste the script from the gist

{% highlight ruby %}
  chmod +x setup.sh && ./setup.sh
{% endhighlight %}

Now sit back and relax!

### Where I use this
I have only tested this on our enivronment which is a Ubunutu 12.04LTS install
on a Linode VPS. I would love to hear some feedback on what other installs this
works on. 

### Paydirt
[Grab the code over on Gists](https://gist.github.com/jasontruluck/5991940)

