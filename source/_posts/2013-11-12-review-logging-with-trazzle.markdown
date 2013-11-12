---
layout: post
title: "Review: Logging with Trazzle"
date: 2010-11-14 17:39
comments: true
categories: ActionScript Logging Review
---

A few weeks ago [I discovered](https://twitter.com/MattesGroeger/status/24742440693) an ActionScript 3.0 Logger called Trazzle. It's available for Mac only and provides a well-arranged, beautiful logging output, a performance monitor, bitmap logging [and much more](http://www.nesium.com/products/trazzle). In order to use this logger, you need to install the [Logger Client](http) and include the [SWC files](http://github.com/nesium/trazzlelib-as3/downloads) within your project. 

<!-- more -->

{% img /images/posts/trazzle.jpg Trazzle %}

All source files are [available on GitHub](http://github.com/nesium/trazzlelib-as3) as well as a [simple example](http://github.com/nesium/trazzle-demo-app).

## Usage

I assume the author tried to build a logger that is very easy to use. Therefor he provides package level functions which allow fast access to the core features. The logger classes behind can also be used. But this should not be part of this post. 

The first step is to initialize to logging framework:

{% codeblock lang:actionscript3 %}
zz_init(stage, "Logger app title");
{% endcodeblock %}

*Please note:* The logging only works if you import the class `TrazzleLogger`. The reason why the demo application works: the `StatusBar` class holds a reference to `TrazzleLogger`. But normal logging will retrieve the reference only dynamically at runtime, so you have to take care of importing it on your own.

After initialization you can trigger the log messages. By default they will be displayed as plain white text. To make use of the different log levels you have to prepend one of the following characters:

{% codeblock lang:actionscript3 %}
log("normal");
log("d debug");
log("i info");
log("n notice");
log("w warning");
log("e error");
log("c critical");
log("f fatal");
{% endcodeblock %}

This syntax is one of the main differences to other logging frameworks. Because of this convention it es very fast to use. You don't have to retrieve a logger instance and define a strong typed log level. Unfortunately the drawback is, you have to know the convention and you are responsible to use it correctly as there is no compile time check.

You can also use the classical `printf` behavior which allows to define a string with placeholders that will be replaced at runtime. Again, in order to use this feature you have to make sure that the method printf is compiled into the application.

{% codeblock lang:actionscript3 %}
logf("There is a difference between %s and %s", "good", "evil");
{% endcodeblock %}

Furthermore with the function `zz_inspect(object)` you should be able to see all the fields and values of an instance. For some reason I didn't get it to work. Please comment below if you found a solution.

## Filtering

In contrast to other logging frameworks it is not possible to configure the logging visibility for certain packages and log levels from within the flash client. Instead the configuration will completely take place in the Trazzle application by using the filters window. Here you can define and combine different rules. Excluding specific packages is not possible. You can save each filter set for later usage.

{% img /images/posts/trazzleFilters.png Trazzle Filters %}

## Performance Monitor

The performance monitor gives you a chronological sequence of the memory consumption and the frames per second (fps). It worked for me but I have not really tested it.

## Conclusion

Using the trazzle logging framework will force you to use the Trazzle application which is available for OS X only. So if you work in larger teams you have to take into consideration that you can not exchange the logging appender easily. 

What I think is a bit strange, is that you have to manually import the TrazzleLogger in order to use it via the package level functions. This makes it difficult to enable/disable the logging on different environments like on debug and release stages. Reading traces is also very exhausting because you have to read small grey text on a black background. And if you copy the text into another editor it is broken by the line numbers. The other log messages are better readable (see first image).

Apart from this cons you get an easy to use logger which has useful additional features like bitmap data output. With this feature I was able to easily find a bitmap that accidentally prevented clicks. What I especially like is the StackTrace which you can see for each log entry. Because the logger is very easy and fast to use you can eventually use him for some special cases only.

One final note: To see the line numbers you have to compile with the compiler flag `-verbose-stacktraces`.
