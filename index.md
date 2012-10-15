---
layout: default
title: "leviwilson.github.com"
---

# Welcome
Thank you for visiting @leviwilson's Github page.  Below are some projects that are being actively worked on.  Please fork me if anything interests you...I would totally fork you.

# Projects

## brazenhead
[brazenhead](https://github.com/leandog/brazenhead) is a low-level driver for Android applications.  [brazenhead](https://github.com/leandog/brazenhead) allows you to instrument any application using a very simple json API with very little setup.

### Getting Started

{% highlight bash %}
$ gem install brazenhead
{% endhighlight %}
After this, the only thing that [brazenhead](https://github.com/leandog/brazenhead) requires is to know some information about the application you are trying to instrument.

{% highlight ruby %}
# tell brazenhead where your application is
server = Brazenhead::Server.new "path/to/your/app.apk"

# start the activity that you'd like to instrument
server.start "SomeActivity"
{% endhighlight %}

## gametel
[gametel](https://github.com/leandog/gametel) is a higher-level abstraction around android cucumber drivers (in particular, [brazenhead](https://github.com/leandog/brazenhead)).  gametel provides for a page-object pattern around lower-level cucumber drivers for Android.  This allows for you to write very simple abstractions to instrument your Android applications.

### Defining a screen
To define a screen, simply use some of [gametel's](https://github.com/leandog/gametel) default accessors to define the controls that you have on your activity.  Here is an example of a login screen page's definition.

{% highlight ruby %}
class LoginPage
  include Gametel

  text(:username, :index => 0)
  text(:password, :index => 1)
  button(:login, :text => 'Login')
end
{% endhighlight %}

That's it!  The only thing left is to utilize your newly defined page-object in your step definitions.

{% highlight ruby %}
on(LoginPage) do |page|
  page.username = 'levi'
  page.password = 'secret'
  page.login
end
{% endhighlight %}
