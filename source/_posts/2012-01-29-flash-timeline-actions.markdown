---
layout: post
title: "Flash Timeline Actions"
date: 2012-01-29 17:12
comments: true
categories: ActionScript BestPractice Workflow Timeline
---

A proper designer-developer workflow is essential, especially in bigger teams. By separating logic and view (e.g. via [MVC Design Pattern](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)) we try to avoid dependencies between artists and programmers. Less coupling increases the speed of development and design.

<!-- more -->

Sometimes it is hard to avoid this coupling, especially if it comes to optimization. Let's imagine an artist who wants to have sound within his timeline animation. He could just embed the sound and put it in the appropriate timeline frame. But if he then wants to use the same sound in another FLA it gets complicated. We don't want the same sound to be loaded twice.

This post describes a pattern for triggering more sophisticated logic from the Flash timeline so artists can easily make use of it. For easier understanding I will stick to the sound example. But keep in mind that this pattern can be applied to other problems as well. I will give an example at the end of this post.

Please also note that this post focuses on timeline related actions. If you want to trigger sounds based on mouse clicks or just playing background sounds, you are perfectly fine doing this purely by code.

### Alternative Approaches

Before discussing the actual solution in detail, I would like to briefly mention two alternatives and their particular drawbacks.

I already mentioned one solution: Embedding sounds within their respective `FLA` files. But using the same sound in other `FLAs` would cause it to be loaded multiple times. The browser can't cache them either because they are embedded in different `SWF` files.

Another solution would be to trigger sounds programmatically by using frame labels. This means that your code would have to observe the timeline for execution of certain labels. You have to use some kind of configuration that tells the code which sound to play on which MovieClip and label. Parsing every MovieClip wouldn't be a good idea in terms of performance.

Both alternatives have a drawback either for artists or programmers. Wouldn't it be cool if we could trigger our logic from the timeline instead of heaving the logic observing it? Thats where 'Timeline Actions' enter the stage.

## Timeline Actions

This illustration shows the relevant parts that I'll explain in the following sections.

{% img center /images/posts/Graph.png Timeline Actions Overview %}

### The basic concept

In the above image you can see two example asset files (red) which will be maintained by the artists. The sound logic (blue) is a separate `SWF` or part of the application maintained by the programmers. It's responsible for loading, caching and playing the sounds. The goal is, that artists can trigger any sound by putting the following frame script into the timeline:

{% codeblock lang:actionscript3 %}
playSound("mySound");
{% endcodeblock %}

*Note*: Because the `playSound()` is a top-level function it requires no import statement.

The sound logic should receive the string `"mySound"` and select the appropriate MP3 for playback. But how do you communicate from the assets (red) to the sound logic (blue)? It's not a good idea to compile the sound logic into each asset. Thats why the first step is, to put just the communication logic into a very small library called "`timeline-sound.swc`" (green).

This library contains the `playSound()` function as well as a static `SoundDelegate` class. The `SoundDelegate` is the connection between `playSound()` and sound logic. The next section "Technical implementation" will give you more insight into the `SWC` contents.

{% img center /images/posts/ActionscriptSettings.png Advanced ActionScript 3.0 Settings %}

You can reference the `timeline-sound.swc` in each `FLA` now. Therefore open your `FLA` and navigate to `File -&gt; ActionScript Settings...` Here go to the second tab "`Library path`" and select the "`Browse to SWC file`" button (1). After choosing the `timeline-sound.swc` it shows up in the panel underneath (2). Now you can use the `playSound()` function as frame script without getting a compiler error. At the same time your sound logic keeps separate.

### Technical implementation

This section will explain how top-level function, delegate and sound logic get wired together. Therefore lets see what the contents of the `timeline-sound.swc` are. You can [browse and download the source code from GitHub](https://github.com/MattesGroeger/as3-timeline-sound) or just read on.

{% img center /images/posts/TimelineSoundContents.png Contents of &#39;timeline-sound.swc&#39; %}

The `SoundDelegate` contains only static methods. Therefore it can be reached from anywhere in the application (including the top-level function `playSound()`). It's implementation is rather simple. It basically just holds a reference to another instance that it delegates to (line 17). Again, for encapsulation reasons this is implemented as an interface `ISoundAdapter` (line 3).

{% codeblock lang:actionscript3 %}
public class SoundDelegate
{
  private static var adapter:ISoundAdapter;

  public static function init(adapter:ISoundAdapter = null):void
  {
    SoundDelegate.adapter = (adapter != null) ? adapter : new NullSoundAdapter();
  }

  public static function get initialized():Boolean
  {
    return adapter != null;
  }

  public static function playSound(key:String):void
  {
    adapter.playSound(key);
  }
}
{% endcodeblock %}

You can also see an `init()` function (line 5) which is used to set the proper <strong>`ISoundAdapter`</strong> implementation. The `ISoundAdapter` defines just one method:

{% codeblock lang:actionscript3 %}
public interface ISoundAdapter
{
  function playSound(key:String):void;
}
{% endcodeblock %}

In line 7 of the `SoundDelegate` listing you will also notice that a `NullSoundAdapter` is assigned in case no instance gets passed to this method. This null-implementation silently ignores every call to `playSound()`. And this is how the <strong>`playSound()`</strong> top-level function looks like:

{% codeblock lang:actionscript3 %}
public function playSound(title:String):void
{
  if (!SoundDelegate.initialized)
    SoundDelegate.init();

  SoundDelegate.playSound(title);
}
{% endcodeblock %}

The function makes sure that the `SoundDelegate` class gets initialized. Because it will create the `NullSoundAdapter` by default, it is not very useful yet. Thats where the actual sound logic comes in. Whenever the logic is ready it calls the `SoundDelegate` and registers itself as `SoundAdapter` (line 6):

{% codeblock lang:actionscript3 %}
public class SoundLogic implements ISoundAdapter
{
  public function SoundLogic()
  {
    // Do this after sounds are loaded
    SoundDelegate.init(this);
  }

  public function playSound(key:String):void
  {
    // play the sound for 'key'
  }
}
{% endcodeblock %}

The `SoundLogic` class implements the `ISoundAdapter` interface. Now whenever someone calls `playSound("foo")` the method `SoundLogic.playSound()` will be called with the string "`foo`". Here you can now put all the logic that is necessary to play the sound for string "`foo`".

### Adding new sounds

Well you could now argue that, with this solution, adding new sounds still requires programmatic effort. In [Magic Land](http://apps.facebook.com/magicland/) (the game I worked on [at wooga](http://www.wooga.com/)) we also found a proper solution for this. Before I can explain it you have to know that in our game the sound logic is encapsulated within a module (`SWF`). This module is loaded lazy because it is less important than other parts of the game. A user can already play our game while the sounds are still loading.

The idea is to embed all sounds that could be triggered from the timeline in a single `FLA`, so the artists can maintain it. Sounds have to be exported for ActionScript, where the linkage name matches the string that will be later used for the `playSound()` method parameter. In order to make the sounds available to the sound logic we chose to compile them into the sound module itself. This way we get good compression and have no additional loading effort. Note however that this approach wouldn't make sense for large sound files.

{% /images/posts center http://blog.mattes-groeger.de/wp-content/uploads/2012/01/sound-library.png sounds.fla %}

The question is now, how do we get new sounds automatically compiled into the sound module? What we did, we created a Sprite that contains all sounds. The `FLA` file then gets compiled into a `SWC` library and is part of the sound module classpath. To enforce the inclusion of the sound definitions the exported `SoundInclusionContainer` class is once referenced in the module:

{% codeblock lang:actionscript3 %}
// Sound module constructor
public function SoundModule()
{
  SoundInclusionContainer; // sound embedding
}
{% endcodeblock %}

This way, whenever the sound module gets compiled, all linked sounds within the container will be made available for the `playSound()` method call. Now the complete flow of adding and using timeline sounds is independent of the programmers while providing an optimized sound loading experience.

## Conclusion

This post showed how artists can easily link timeline animations to a more complex logic. As this post is more about the general idea I didn't explain how the sound logic itself looks like.

I found another use cases for this pattern in our game [Magic Land](http://apps.facebook.com/magicland/). We use it for displaying visual effects. Similar to sounds they are part of a separate module that is lazy loaded. On low performance computers we could disable the effects completely.

Finally, lets have a look on the pros and cons of this pattern.

### Disadvantages

The artists won't be able to hear the sound wile testing their sole assets. They have to start the game including the sound logic in order to hear something. In [Magic Land](http://apps.facebook.com/magicland/) the artists are able to build the game locally on their computer (probably a topic for another blog post). So they can hear how it feels in the actual game.

Another drawback is that each asset `FLA` that makes use of the timeline actions has to link the `timeline-sound.swc`, which adds 2 Kb. A solution would be to exclude the library for compiling and enforce it to compile in the main application. But then running the asset `SWF` outside the final application/game throws errors and timeline animations won't be visible anymore.

### Advantages

On the positive side programmers get less distracted by sound issues because artists can do everything on their own. They are free to add the `playSound()` call in any asset file, as long as they link the `timeline-sound.swc` into the `FLA` file.

At the same time the sounds itself can be loaded after more important stuff has been done loading. This leads to a much better user experience. Also imagine a user turned off sounds. Now you can just not load the sounds at all and the `playSound()` call will be silently ignored (null-implementation of `ISoundAdapter`).

### Sources

You can find all source files for the timeline-sound library in [this GitHub repository](https://github.com/MattesGroeger/as3-timeline-sound). If you have any questions feel free to comment below. Enjoy!
