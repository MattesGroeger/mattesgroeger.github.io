---
layout: post
title: "Short-Lived Commands with Parsley"
date: 2010-10-31 21:24
comments: true
categories: as3 parsley
---

This week we had to decide on a [MVC Framework](http://en.wikipedia.org/wiki/Model%E2%80%93View%E2%80%93Controller) for our next game at [Wooga](http://www.wooga.com). In the end the options were [Robotlegs](http://www.robotlegs.org/) and [Parsley](http://www.spicefactory.org/parsley/). Both of them had advantages and disadvantages. One aspect we came across was the support of `Short-lived Command Objects`.

<!-- more -->

These commands hold no state and will be garbage collected after execution. In Robotlegs they are supported via the [`ICommandMap`](http://api.robotlegs.org/org/robotlegs/core/ICommandMap.html) interface. Parsley also provides an implementation of this pattern called [`DynamicCommand`](http://www.spicefactory.org/parsley/docs/2.3/manual/messaging.php#command_objects). In contrast to Robotlegs, Parsley provides 4 different ways to build and configure a context:

* [MXML](http://www.spicefactory.org/parsley/docs/2.3/manual/config.php#mxml): DynamicCommand Tag. Only for Flex projects.
* [XML](http://www.spicefactory.org/parsley/docs/2.3/manual/config.php#xml): DynamicCommand Node. No compiler check on types possible.
* [ActionScript](http://www.spicefactory.org/parsley/docs/2.3/manual/config.php#as3): No way to configure dynamic Commands.
* [Configuration DSL](http://www.spicefactory.org/parsley/docs/2.3/manual/config.php#dsl): Programmatic configuration of Dynamic Commands.

Because our game should not use the Flex framework, the last option is our only choice if we want to have strongly typed command mappings like in Robotlegs.

## DynamicCommands via Configuration DSL

Because a documentation is not available for this case it was a bit tricky to find the solution (thanks for the hint, Jens). This was the first result:

{% codeblock lang:actionscript3 %}
var contextBuilder:ContextBuilder = ContextBuilder.newSetup()
  .viewRoot(this)
  .newBuilder();

var targetDef:DynamicObjectDefinition = contextBuilder
  .objectDefinition()
  .forClass(CommandType)
  .asDynamicObject()
  .build();

DynamicCommandBuilder
  .newBuilder(targetDef)
  .builder
  .messageType(MessageType)
  .stateful(false)
  .build();

contextBuilder.config(ActionScriptConfig.forClass(MainConfig));
contextBuilder.build();
{% endcodeblock %}

Note that you can not use the `ActionScriptContexBuilder.build()` notation anymore. Instead you have to configure the whole context via the DSL. In line 18, the actual configuration (`MainConfig`) which contains all object definitions is passed in.

Because this configuration is very hard to read and redundant if you want to map several Commands, I encapsulated the mapping to a separate class `CommandMap`. Now the configuration looks like this:

{% codeblock lang:actionscript3 %}
var contextBuilder:ContextBuilder = ContextBuilder.newSetup()
  .viewRoot(this)
  .newBuilder();

var commandMap:CommandMap = new CommandMap(contextBuilder);
commandMap.register(LoginCommand, LoginRequest);
commandMap.register(LogoutCommand, LogoutRequest);

contextBuilder.config(ActionScriptConfig.forClass(MainConfig));
contextBuilder.build();
{% endcodeblock %}

Until now I found no way to map or un-map Commands after the context has been built. The only way would be to destroy the context they are registered in.

To get a better impression how short-lived commands are used in Parsley, I implemented a small example, which you can [access on GitHub](http://github.com/MattesGroeger/as3-parsley-example). The CommandMap class can be [found here](http://github.com/MattesGroeger/as3-parsley-example/blob/master/src/de/mattesgroeger/parsley/core/CommandMap.as).

And this is how the login example looks like. Invalid credentials lead to an error pop up. Valid login works with *admin* and password *test*:

<object style="width: 850px; height: 250px;" width="850" height="250" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="src" value="/images/posts/ParsleyExample1.swf" /><embed style="width: 850px; height: 250px;" width="850" height="250" type="application/x-shockwave-flash" src="/images/posts/ParsleyExample1.swf" /> </object>

## Summary

The Robotlegs implementation is very straight forward. Commands can be mapped and un-mapped at any time. This is not possible in Parsley. But you have other features that give maybe similar results (for example <a href="http://www.spicefactory.org/parsley/docs/2.3/manual/messaging.php#interceptors" target="_blank">MessageInterceptors</a>).

Because Parsley reflects on the types you can pass data directly to the execute method. This way, each concrete execute method can receive strong typed parameters (see line 9). No framework class has to be extended for custom Commands. This is how a command can look like:

{% codeblock lang:actionscript3 %}
public class LogoutCommand
{
  [Inject]
  public var service:ISessionService;

  [MessageDispatcher]
  public var dispatcher:Function;

  public function execute(message:LogoutRequest) : ServiceRequest
  {
    return service.logout(message.sessionId);
  }

  public function result(success:Boolean) : void
  {
    dispatcher(new LogoutSuccess(success));
  }
}
{% endcodeblock %}

I also like the native support of synchronous and asynchronous Commands in Parsley. No special configuration is necessary, because Parsley reflects on the return type of the `execute()` method (line 9, `ServiceRequest`).

## Links:

* [Browse the example sources on GitHub](http://github.com/MattesGroeger/as3-parsley-example)
* [Readme of the example](http://github.com/MattesGroeger/as3-parsley-example/blob/master/README.md)
* [CommandMap source](http://github.com/MattesGroeger/as3-parsley-example/blob/master/src/de/mattesgroeger/parsley/core/CommandMap.as)
