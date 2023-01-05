---
layout: post
title: "Simple Step to Track and Improve Performance in Android"
date: 2016-10-06 17:02:28 +0700
comments: true
categories: performance
---

It's been a while since I wrote my last article, today I would like to share more about something. Let's talk about performance. I will give a little sample how we can improve performance of our app.

Apps with good performance should not be lagging. By not lagging, it means that the apps load the UI within 16ms/frame or 60 frames/second. The question is how do we achieve it?

<!--more-->

There are some possible reasons why we can't load the ui within 16ms/frame such as layout/measuring time, overdraw, garbage collector, etc.

### Step 1: Identify the cause
There are some tools to know how our app performs and whether our apps can load the ui 60frames/second. 

The first tool is profile gpu rendering. We can enable this from developer options on our device. 

![](/images/post/enable_profile_gpu_rendering.png)

![](/images/post/gpu_performance.png)

The green line indicate position for 16ms, we should not draw the ui above this line to achieve smooth and lag free app.

Another tool I like is `systrace`, I like it because it gives you more accurate and more explainable data. To use it, we only need to run this simple command

```
$ python ~/Library/Android/sdk/platform-tools/systrace/systrace.py --time=10 -o ~/Desktop/perform.html sched gfx view wm
```

Please note that it uses python, so make sure you have python installed, basically we only need to run python script inside android sdk folder, and then we set where we want to put the result (in this case is `~/Desktop`)

I also set the duration for the data which is 10 seconds. While running this command, we just have to interact with the app that we want to trace.

As an example, while tracing I am doing these actions

{% youtube Fx6KR38JEzs %}

The result is like this

![](/images/post/performance_before.gif)

From this systrace report, focus on F letter at the bottom, these are frames. Green background means the frame was drew below 16ms. The other colours mean otherwise. So our goal here is to make all frames green.

If we look on one of the frame in orange colour, we can see the reason below it.

![](/images/post/performance_explanation.png)

This is the reason why I love systrace. So, we have problem with long `view#draw()`. The most common cause of this issue is overdraw.

### Step 2: Fixing overdraw
Now, let's see how we can fix overdraw. Overdraw is basically we draw the views more than we need to, so we have to find what views that do not need to be drawn, and remove it. You can read more about overdraw [here](https://developer.android.com/studio/profile/dev-options-overdraw.html)

Enable overdraw tracing is simple, just visit developer options then enable Debug GPU overdraw

![](/images/post/overdraw.png)

Red means the view has been drawn more than 3 times.

**How do we find the culprit?**

There is new tool in Android Studio 2.2 called Layout Inspector, it's super handy tool to investigate overdraw beside the old hierarchy layout.

![](/images/post/layout_inspector.png)

We only need to look on the `bg` attribute, as in the screenshot, the LinearLayout has a background, but also the `ViewPager`, `RelativeLayout` and the `RecyclerView` of its child.

So, if we have background in `RecyclerView` why do we need background in `LinearLayout` since the background will not be visible to user?

This is where we have to fix it. Sometimes engineers do not really care about overdraw, and they just put background everywhere, even it's not visible. This is causing android has to draw something useless. 

And also please note, that android by default will provide background to each activity, if we can use default one it will be better and we don't need to overdraw it.

To solve this, I just have to remove unnecessary background

![](/images/post/remove_background.png)

### Step 3: Evaluate
Let's look again on the result.

![](/images/post/overdraw-fix.png)

It's better, and let's look on the systrace report

![](/images/post/performance_after.gif)

Hope it helps. 

