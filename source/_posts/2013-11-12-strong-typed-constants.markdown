---
layout: post
title: "Strong Typed Constants"
date: 2011-01-02 19:19
comments: true
categories: ActionScript Best-Practice
---

Constants are commonly occurring in the ActionScript core classes. Typical examples are events (`Event.COMPLETE`) or general configurations (`StageAlign.TOP_LEFT`). These constants are typically of type `String` and grouped in one class. 

<!-- more -->

If you want to set the `align` property of the `Stage` for example you have to know where to look up the different possible constants. This can be done by looking up the documentation or by guessing the potential class name (`StageAlign`). IDEs like [FDT](http://fdt.powerflasher.com) provide support for these constants by suggesting them in the auto completion. 

However, the programmer is never enforced to use the existing constants and can assign any other string. This is error prone because the string is eventually misspelled. It is also annoying if you have to look up the documentation because your IDE doesn't suggest them. If you use your own constants in an application or library then you are completely on your own.

Other programming languages like Java provide a special syntax for this problem: `enum`

{% codeblock lang:java %}
public enum Direction {LEFT, RIGHT}
{% endcodeblock %}

This enumerations can then be used strong typed in code. For example a function can define this as type in the signature. Assigning something else than one of the predefined constants (`LEFT` or `RIGHT`) would cause a compiler error:

{% codeblock lang:java %}
public void applyDirection(Direction direction) {
        // use the enum here
}
{% endcodeblock %}

The ActionScript language itself has no support for enumerations. But there is a solution: Instead of using constants of type `String` just use the type of the constants class itself.

{% codeblock lang:actionscript3 %}
public class Direction
{
  public static const LEFT:Direction = new Direction();
  public static const RIGHT:Direction = new Direction();
}
{% endcodeblock %}

A function can then look like this:

{% codeblock lang:actionscript3 %}
public function applyDirection(direction:Direction):void
{
  switch (direction)
  {
    case Direction.LEFT:
      // use direction here...
      break;
    case Direction.RIGHT:
      // use direction here...
      break;
    default:
      throw new IllegalOperationError("Unsupported direction " + direction);
  }
}
{% endcodeblock %}

{% codeblock lang:actionscript3 %}
applyDirection(Direction.LEFT);
{% endcodeblock %}

You would still be able to create new instances of `Direction` and assign them. But the implementation of `applyDirection()` relies on the predefined constants. An error would be thrown if another instance would be assigned. Furthermore the user can see the different possible constants because he knows which specific type is required.

If you want to trace the `Direction` or if you want to store more information this is no problem, too:

{% codeblock lang:actionscript3 %}
public class Direction
{
  public static const LEFT:Direction = new Direction("left");
  public static const RIGHT:Direction = new Direction("right");

  private var direction:String;

  public function Direction(direction:String)
  {
    this.direction = direction;
  }

  public function toString():String
  {
    return direction;
  }
}
{% endcodeblock %}

Right now, this approach is the only way to provide type save enumerations in ActionScript. You should always use them if you write third party libraries. Also in projects with multiple developers this makes sense.

*Update 2011/03/13:* In order to to restrict the enum to the predefined constants you could combine the declaration with a singleton enforcing approach. Just expect an instance of a nested class in the constructor:

{% codeblock lang:actionscript3 %}
package
{
  public class Direction
  {
    public static const LEFT:Direction = new Direction(new EnumEnforcer(), "left");
    public static const RIGHT:Direction = new Direction(new EnumEnforcer(), "right");

    private var direction:String;

    public function Direction(enumEnforcer:EnumEnforcer, direction:String)
    {
      this.direction = direction;
    }

    public function toString():String
    {
      return direction;
    }
  }
}

class EnumEnforcer
{
}
{% endcodeblock %}

Of course this is still not 100% save because you could just pass in `null`. In this case an error could be thrown. But this would happen at runtime only then.

Thanks to *Thijs* for pointing out this problem in the comments and *Peter Höche* for suggesting this solution.
