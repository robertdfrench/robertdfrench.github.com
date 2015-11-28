---
layout: post
title:  "Speed Up That Nokogiri Build"
date:   2015-11-27 10:53:24
categories: nokogiri jenkins rails bundler gem
---
An important part of continuous integration is making sure that you can rebuild your environment from scratch. Unfortunately for most Rails apps, running `bundle install` can cause your CI job to hang for several minutes, and [nokogiri](http://www.nokogiri.org/) is usually the culprit.

### Don't rebuild LibXML from scratch
The reason it takes so long to install nokogiri is that, by default, it will try to build its own copies of libxml and libxslt, which are huge. That definitely makes initial adoption easier, but now we're stuck with it, so let's speed things up a bit.

Fortunately, the nokogiri gem takes a [build option](http://www.nokogiri.org/tutorials/installing_nokogiri.html#using_your_system_libraries) that tells it to look for system-installed xml and xslt libraries. To get Bundler to use it, we create an entry in the local [bundler config](http://bundler.io/v1.6/man/bundle-config.1.html) and then install it like so:

{% highlight bash %}
bundle config build.nokogiri --use-system-libraries
bundle install
{% endhighlight %}

### Clean up after yaself
The trouble with this approach is that the bundle config is not ephemeral, and if you are running multiple builds on the same Jenkins server, the config will persist and affect other branches. I recommend sticking the following in your `test.sh` right after your gems are installed:

{% highlight bash %}
bundle config --delete build.nokogiri
{% endhighlight %}

Doing this for any specific gem build settings after you run `bundle install` will make sure that you do not get screwy environment behavior when running future builds.
