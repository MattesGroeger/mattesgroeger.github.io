---
layout: post
title: "Icon Badge for Windows"
date: 2010-03-14 22:54
comments: true
categories: [ActionScript, Air, Api, Lib]
---

{% img left /images/posts/WindowsSystemTray.jpg %}

Until now the [Air Icon Badge library](/blog/2010/02/08/icon-badge-library-for-air/) implementation only supported OS X dock icons. Nevertheless it is easy to extend the implementation for utilizing it as Windows system tray icon. In this article I will demonstrate how this could be realized. And this is how it will look like on Windows XP (Windows Vista and 7 will look the same).

<!-- more -->

### Implementation

The first class that I create is the `SystemTrayIconBuilder` (it must implement `IconBuilder` interface). This class is responsible for composing the system tray icon. It also sets the general icon size to 16 by 16 pixels.

{% codeblock lang:as3 %}
public class SystemTrayIconBuilder extends AbstractIconBuilder implements IconBuilder
{
  public override function createNewIcon(width : int, height : int) : void
  {
    super.createNewIcon(16, 16);
  }

  public override function addBackground( background : Bitmap) : void
  {
    background.width = background.height = 16;

    container.addChild(background);
  }

  public override function addBadge(label : String) : void
  {
    var badge : TrayIconBadge = new TrayIconBadge();
    badge.label.text = label;

    container.addChild(badge);
  }
}
{% endcodeblock %}

Please note the the `TrayIconBadge` class used in this example (line 17) is a MovieClip designed and exported (swc) with the Flash IDE. I used a pixel font for better readability. Because of the small icon size the visible letter count is also limited.

The next step is to create a custom `SystemTrayIconBadge` which has to implement the `IconBadge` interface. To avoid redundancies I extend the `DockIconBadge` class and override the factory method "`createIconBuilder()`".

{% codeblock lang:as3 %}
public class SystemTrayIconBadge extends DockIconBadge
{
  protected override function createIconBuilder() : IconBuilder
  {
    return new SystemTrayIconBuilder();
  }
}
{% endcodeblock %}

The last step is to build the `CrossPlatformIconBadgeFactory` which implements `IconBadgeFactory`. It is responsible for detecting the users operating system and returning the corresponding `IconBadge`. Again I extend the existing factory class to avoid redundancies:

{% codeblock lang:as3 %}
public class CrossPlatformIconBadgeFactory extends
AirIconBadgeFactroy
{
  public override function create() : IconBadge
  {
    if (NativeApplication.supportsSystemTrayIcon)
      return new SystemTrayIconBadge();

      return super.create();
  }
}
{% endcodeblock %}

Finally for using the new factory it has to be assigned to the `AirIconBadge` class.

{% codeblock lang:as3 %}
AirIconBadge.factory = new CrossPlatformIconBadgeFactory();
AirIconBadge.label = "1";
{% endcodeblock %}

### Summary

As you can see, it is very little effort to add the windows system tray icon capability. You are free to (re)use this example. [Binaries](http://code.google.com/p/air-icon-badge/downloads/list) and [sources](http://code.google.com/p/air-icon-badge/source/browse/#svn/air-icon-badge-examples/trunk/dev/src/de/mgroeger/air/icon/example/windows) can be downloaded from the [google code project site](http://code.google.com/p/air-icon-badge/).

* [Example Download](http://code.google.com/p/air-icon-badge/downloads/detail?name=air-icon-badge-examples.zip&amp;can=2&amp;q=#makechanges) (air/swf)
* [Browse Example Sources](http://code.google.com/p/air-icon-badge/source/browse/#svn/air-icon-badge-examples/trunk/dev/src/)
* [Library Download](http://code.google.com/p/air-icon-badge/downloads/detail?name=air-icon-badge-0.1.zip)
