---
layout: post
title: "Label Validation with FlexUnit"
date: 2011-01-18 22:36
comments: true
categories: ActionScript BestPractice Workflow
---

While working in interdisciplinary teams where graphics are produced by artists and the code comes from the developers, a solid designer-developer work-flow is crucial. At my current project team at [Wooga](http://www.wooga.com) we already established a very good work-flow. The artists can produce graphics and see them in the running application after committing them.

<!-- more -->

## The Problem

In our project we use [SWC asset libraries](http://www.richardleggett.co.uk/blog/index.php/2010/03/08/flash_builder_and_flash_pro_asset_workflows) to have a compile time check and strong typed access to all our graphics. To add logic to `MovieClip` frames you could either put the code directly on the timeline or the artist defines labels which are then utilized from the code. In order to keep the view separated from the code you should always use the label approach. But this is also risky if the artist accidentally deletes or renames a label. The code would then behave unexpectedly without knowing it. The only way to see the problem is to start the application and test all the `MovieClip` logic.

## The First Approach

To protect the labels from unintended changes, our first approach was to check their existence wherever we used them in code. This means we had to provide a separate `Vector` that contained all the expected labels. 

{% codeblock lang:actionscript3 %}
private static var labels:Vector.<String> = new Vector.<String>();
private var view:IconView;

public function IconMediator(view:IconView):void
{
  initializeLabels();
  assertLabelsExist(view);
}

private function initializeLabels():void
{
  if (labels.length == 0)
    labels.push(IconState.ON, IconState.OFF);
}

private function assertLabelsExist(view:ButtonRounded):void
{
  var requiredMatchesRemaining:int = labels.length;

  for each (var requiredLabel:String in labels)
  {
    for each (var label:FrameLabel in view.currentLabels)
    {
      if (requiredLabel == label.name)
        requiredMatchesRemaining--;
    }
  }

  if (requiredMatchesRemaining != 0)
    throw new IllegalArgumentError('IconView requires all labels: ' + labels);
}
{% endcodeblock %}

Beside this we defined the labels itself in an [enumeration class](/strong-typed-constants/) to provide strong typed access.

{% codeblock lang:actionscript3 %}
public class IconState
{
  public static const ON:IconState = new IconState("on");
  public static const OFF:IconState = new IconState("off");

  private var type:String;

  public function IconState(type:String)
  {
    this.type = type;
  }

  public function toString():String
  {
    return type;
  }
}
{% endcodeblock %}

This approach leads to three main problems:

* Depending on when the validation is executed in code, it could still happen that you don't see the problems immediately
* We create a lot of code that is only necessary for validation but not for the application itself
* We create redundancy because we have to maintain the `Vector` of expected labels and the enumeration class (`IconState`). When changing labels it is very likely that you forget to update the Vector.

## The Solution

Thats why we came up with a different approach. We moved the validation into the unit tests. Now the code is separated but still executed because of our integration server (Hudson with <a href="http://www.flexunit.org/" target="_blank">FlexUnit support</a>). The normal application code is now much slimmer and better readable. The artists/developers are automatically notified by mail if their changes break the tests.

To solve point 3 I wrote an assertion method that can reflect on enumeration classes and check all the defined labels on a certain MovieClip. 

{% codeblock lang:actionscript3 %}
public function assertLabelEnum(target:MovieClip, enumClass:Class):void
{
  var classInfo:ClassInfo = ClassInfo.forClass(enumClass);
  var properties:Array = classInfo.getStaticProperties();
  var expectedLabel : String;
  var type:*;
  var labelCounter:int = 0;

  for each (var property : Property in properties)
  {
    type = property.getValue(enumClass);

    if (type is String)
      expectedLabel = String(type);
    else if (type is enumClass)
      type["toString"]();
    else
      continue;

    labelCounter++;
    expectedLabel = type["toString"]();

    assertLabel(expectedLabel, target);
  }

  assertEquals("Amount of expected labels differs from the amount of existing labels", labelCounter, target.currentLabels.length);
}
{% endcodeblock %}

This method internally calls another assertion method `assertLabel()`. This method can also be used independently for testing specific labels without using enumeration classes.

{% codeblock lang:actionscript3 %}
public function assertLabel(target:MovieClip, expectedLabel:String):void
{
  var found:Boolean = false;

  for each (var label:FrameLabel in target.currentLabels)
  {
    if (expectedLabel == label.name)
    {
      found = true;
      break;
    }
  }

  if (!found)
    fail("Expected label [" + expectedLabel + "] not found in " + target);
}
{% endcodeblock %}

*Note:* I used the reflection library from spicelib to retrieve all the static members of the enumeration class. You can [download the library here](http://www.spicefactory.org/parsley/).

And this is how the test method would look like:

{% codeblock lang:actionscript3 %}
[Test]
public function test_icon_frame_labels():void
{
  assertLabelEnum(new IconView(), IconState);
}
{% endcodeblock %}

## Conclusion

We made very good experiences with this approach because unintended changes on the labels no longer lead to awkward behavior in the application itself. And of course the application code can focus on the main logic and is therefor better readable.
