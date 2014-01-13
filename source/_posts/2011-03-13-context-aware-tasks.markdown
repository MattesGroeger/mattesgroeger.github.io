---
layout: post
title: "Context-Aware Tasks"
date: 2011-03-13 20:03
comments: true
categories: ActionScript BestPractice Parsley
---

This article shows a useful way on how to combine the [Spicelib](http://www.spicefactory.org/parsley/download.php) [Task Framework](http://www.spicefactory.org/parsley/docs/2.3/manual/task.php#intro) with [Parsley](http://www.spicefactory.org/parsley/download.php). This way you can access everything from the context during the execution of a Task...

<!-- more -->

{% blockquote Parsley Documentation http://www.spicefactory.org/parsley/docs/3.3/manual/task.php#intro %}
The Task Framework is a general abstraction for asynchronous operations. It allows nesting / grouping of Tasks in TaskGroup instances for concurrent or sequential execution
{% endblockquote %}

One way to access the `Context` from within a `Task` would be to define it during the <a href="http://www.spicefactory.org/parsley/docs/2.3/manual/config.php" target="_blank">Parsley Configuration</a> (`Context`). But this would mean, that the `Task` will not be removed until the context gets destroyed. Because a `Task` is only active during a limited time, this doesn't make much sense.

Another approach would be, not to define the `Task` in the `Context` but passing in all the necessary dependencies within the constructor. Depending on the amount of required dependencies, the constructor can become quite big. Also `Parsley` features like <a href="http://www.spicefactory.org/parsley/docs/2.3/manual/messaging.php#intro" target="_blank">Messaging</a> are not directly supported anymore.

## Solution

That's why this solution utilizes the <a href="http://www.spicefactory.org/parsley/docs/2.3/manual/lifecycle.php#dynamic" target="_blank">Dynamic Object</a> feature of Parsley. It allows to add any instance to the `Context` during runtime and also removing it again, if not required anymore. For a `Task` this means that it needs to be added before the `start()` is called, and removed after the `complete()` has been triggered.

This adding/removing happens through the `Context` interface of Parsley. It should happen automatically in the `Task` to reduce the amount of code to write. In order to not have to pass the `Context` to each `Task` it could just be set once in the parent `TaskGroup`. The child `Task` then accesses it via the `parent` getter:

{% codeblock lang:actionscript3 %}
public class SequentialContextTaskGroup extends SequentialTaskGroup implements IContextProvider
{
  private var _context:Context;

  public function SequentialContextTaskGroup(context:Context, name:String = null)
  {
    _context = context;

    super(name);
  }

  public function get context():Context
  {
    return _context;
  }
}
{% endcodeblock %}

And this is how the `IContextProvider` interface looks like:

{% codeblock lang:actionscript3 %}
public interface IContextProvider
{
  function get context():Context;
}
{% endcodeblock %}

The following `AbstractContextTask` is then responsible for adding itself to the `Context` as soon as it is started. It also removes itself as soon as the `complete()`, `cancel()` or `skip()` has been called. Furthermore it provides a `doStartContext()` method (read more in the next paragraph).

{% codeblock lang:actionscript3 %}
public class AbstractContextTask extends Task
{
  private var _dynamicObject:DynamicObject;

  protected override function doStart():void
  {
    _dynamicObject = IContextProvider(parent).context.addDynamicObject(this);

    doStartContext();
    super.doStart();
  }

  protected function doStartContext():void
  {
    /* base implementation does nothing */
  }

  protected override function doCancel():void
  {
    cleanContext();
    super.doCancel();
  }

  protected override function doSkip():void
  {
    cleanContext();
    super.doSkip();
  }

  protected override function complete():Boolean
  {
    cleanContext();
    return super.complete();
  }

  private function cleanContext():void
  {
    _dynamicObject.remove();
  }
}
{% endcodeblock %}

The following class is a concrete implementation of `AbstractContextTask`. That's why it gets the `HintComponent` injected (line 4). Please also note that this `Task` overrides the method `doStartContext()` (line 13). This way we ensure that the class already has been added to the `Context`.

{% codeblock lang:actionscript3 %}
public class ShowHintTask extends AbstractContextTask
{
  [Inject]
  public var hint:HintComponent;

  private var type:HintType;

  public function ShowHintTask(type:HintType)
  {
    this.type = type;
  }

  protected override function doStartContext():void
  {
    hint.showHint(type);

    complete();
  }
}
{% endcodeblock %}

The following code shows how everything can be glued together. Note, that you don't have to pass the `Context` to each task.

{% codeblock lang:actionscript3 %}
public class TaskController
{
  [Inject]
  public var context:Context;

  [Init]
  public function initialize():void
  {
    var taskGroup:SequentialContextTaskGroup = new SequentialContextTaskGroup(context);

    taskGroup.addTask(new ShowHintTask(HintType.ATTENTION));
    taskGroup.addTask(new PointToTask(new Point(3, 5)));
    // add more tasks here...

    taskGroup.start();
  }
}
{% endcodeblock %}

## Summary

This approach of `Context`-aware `Tasks` ensures that you don't have a lot of `Tasks` at the same time in the `Context`. The adding and removing from the `Context` happens in an encapsulated class. The `Task` itself can directly access injected objects during execution time.

## Links

* [Strong typed constants](http://mattesgroeger.github.io/blog/2011/01/02/strong-typed-constants/)</li>
* [Task framework documentation](http://www.spicefactory.org/parsley/docs/2.3/manual/task.php)</li>
* [Parsley/Spicelib download](http://www.spicefactory.org/parsley/download.php)</li>
