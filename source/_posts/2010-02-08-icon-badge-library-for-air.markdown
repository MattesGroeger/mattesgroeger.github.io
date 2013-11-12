---
layout: post
title: "Icon Badge Library for Air"
date: 2010-02-08 14:06
comments: true
categories: [ActionScript, Air, API, Lib]
---

[Adobe Air](http://www.adobe.com/products/air/) is often used to build feed readers or social media clients. This kind of applications can retrieve new data while they are running in the background. In that case it would be great to inform the user about the amount of new items.

<!-- more -->

{% img right /images/posts/mail.png %}

With OS X you can use the dock icon for that purpose. The Cocoa Framework allows to display a user defined text consistently on top of the application dock tile (icon). E.g. the screenshot at the right shows 2 unread mails in the inbox.

{% img left /images/posts/comparision.png Native and emulated badge %}

Unfortunately the Air runtime allows no access to this native functionality. Thats why the idea for this library came up. It tries to emulate the native badge with the possibilities Air provides (see left image). Until now only the OS X badge is supported but a Windows implementation is possible, too. I will demonstrate the extensibility in one of the following blog posts.

## Usage

To show the badge label you can use the static facade `AirIconBadge`. Internally it will create an `IconBadge` appropriate for the current operating system. **Note:** only one implementation for OS X is provided until now! Windows or Linux users won't see a badge label.

With the static property `label` you can assign any string that should be displayed. Thats it.

{% codeblock lang:as3 %}
AirIconBadge.label = "1";
{% endcodeblock %}

If you assign an empty string or `null`, no badge will be displayed. To remove the current badge label you can also call the method `clearLabel()`.

By default the biggest icon defined within the [application descriptor](http://livedocs.adobe.com/flex/3/html/help.html?content=File_formats_1.html) will be loaded and shown. If no icon has been defined or the path is incorrect you must assign a `customIcon` in order to see the label. You can also assign a `customIcon` if you want to replace the default icon temporarily. Removing the `customIcon` will show up the default icon again.

{% codeblock lang:as3 %}
AirIconBadge.customIcon = new CustomIconBitmap();
{% endcodeblock %}

If neither of the icons could be loaded, no badge will be displayed. In this case an error event will be dispatched (`UpdateErrorEvent`). To get status information about the internals you can register for the InformationEvent. Both events will be dispatched by the `IconBadge` witch is statically stored within the AirIconBadge.

{% codeblock lang:as3 %}
AirIconBadge.customIcon = new CustomIconBitmap();

var iconBadge : IconBadge = AirIconBadge.iconBadge;
iconBadge.addEventListener(UpdateErrorEvent.UPDATE_ERROR, handleError);
iconBadge.addEventListener(InformationEvent.INFORMATION, handleInformation);

function handleError(event : UpdateErrorEvent) : void
{
  // do some error handling
}

function handleInformation(event : InformationEvent) : void
{
  trace(event.information.toString());
}
{% endcodeblock %}

## Download & Demo

All sources, binaries and examples are [available for download](http://code.google.com/p/air-icon-badge/downloads/list) under the MIT license. If you have a Mac, you can [download this Air application](http://code.google.com/p/air-icon-badge/downloads/detail?name=air-icon-badge-examples.zip&amp;can=2&amp;q=#makechanges) where the badge shows up on the real application icon. Running this application on windows will have no visual effect.

None Mac users can use the following demo that shows a preview of the dock icon:

<object style="width: 850px; height: 400px;" classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" width="850" height="400" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="src" value="/images/posts/WebBadgeExample.swf" /><embed style="width: 850px; height: 400px;" type="application/x-shockwave-flash" width="850" height="400" src="/images/posts/WebBadgeExample.swf"> </embed></object></br>

## Whats next

The documentation (especially the ASDocs) will be improved as well as the sources. If you have questions or feature requests, please let my know. The next blog posts will give you a deeper insight in the architecture and extensibility of this library.
