---
layout: post
title: "Beware of Setting Image Resource for Preview in XML for Android"
date: 2015-07-11 13:31:54 +0700
comments: true
categories: Tips
---

For the past hours, I had been struggling to find the cause of `OutofMemoryException`. I know It’s really common issue with image in Android. I used to have this issue in my first 6 months of learning Android and I thought I had ate enough but guess what, seems like it’s not happenning today.

<!--more-->

Sometimes, when we write a layout file, we want to preview the result and if the layout has an ImageView then we also want to see the image by just giving the image resources in xml. Beware of that, by the time you set image resource in image view for only preview purpose, don’t forget to remove it. If we forget to remove it, then in the runtime the image will be loaded and suck the app memory.

It would seem fine if the image is only loaded once but what if the sample image is on item layout, and we use the layout in recycler view (listview)? the sample image will be loaded in every single item you have in the list. Just multiply it with the size of the image, that is how the app becomes a memory sucker.

I made this mistake, and fortunately I used a really big image, so the exception was always raised.

Here is the example of pre set image resources

``` xml
 <ImageView
  android:id="@+id/iv_preview"
  android:layout_width="match_parent"
  android:layout_height="200dp"
  android:layout_below="@+id/tv_title"
  android:adjustViewBounds="true"
  android:scaleType="centerCrop"
  android:src="@drawable/budget_preview"
/>
```

`budget_preview.png` is a really big image. With default activity, my app only uses around 20mb of memory,

{% picture /images/post/ram_usage_1.png %}

but after open the activity with only 3 items with that layout, it jumped into 90mb, scroll a little bit, OOM was raised.

{% picture /images/post/ram_usage_2.png %}

So don’t forget to remove android:src after you finish preview, and you’ll probably save a good memory
Hope it’s usefull.

####Protip from +Christophe Beyls

Use tools:src instead of android:src to get the preview image in Android Studio with zero impact on runtime http://tools.android.com/tips/layout-designtime-attributes


