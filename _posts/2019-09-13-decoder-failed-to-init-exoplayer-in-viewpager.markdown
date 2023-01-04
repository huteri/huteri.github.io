---
layout: post
title: "Decoder Failed to Init - Exoplayer & ViewPager"
date: 2019-09-13 13:52:53 +0800
comments: true
categories: exoplayer, tips
---


There is a limit on how many times we can initialize exoplayer in a single device. If we initialize more than it should be, It will cause exoplayer to throw an exception that is not easy to debug and can potentially break the video player in the app.

<!--more-->

It's pretty common to show video player inside viewpager, sometimes we just want to play video in a list or swipeable view like view pager. This process requires us to initialize exoplayer inside child views, which is not a good practice because these views can be initialized multiple times by the operating system.

In my case, every time I swipe or navigate to another screen, a new instance of exoplayer is created and an exception is thrown at the end.

Here is a sample of exceptions thrown by exoplayer.
```
playerFailed [0.58, 1]
    com.google.android.exoplayer2.ExoPlaybackException: com.google.android.exoplayer2.audio.AudioSink$InitializationException: AudioTrack init failed: 0, Config(32000, 12, 32000)
        at com.google.android.exoplayer2.audio.MediaCodecAudioRenderer.processOutputBuffer(MediaCodecAudioRenderer.java:738)
        at com.google.android.exoplayer2.mediacodec.MediaCodecRenderer.drainOutputBuffer(MediaCodecRenderer.java:1507)
        at com.google.android.exoplayer2.mediacodec.MediaCodecRenderer.render(MediaCodecRenderer.java:653)
        at com.google.android.exoplayer2.ExoPlayerImplInternal.doSomeWork(ExoPlayerImplInternal.java:575)
        at com.google.android.exoplayer2.ExoPlayerImplInternal.handleMessage(ExoPlayerImplInternal.java:326)
        at android.os.Handler.dispatchMessage(Handler.java:102)
        at android.os.Looper.loop(Looper.java:164)
        at android.os.HandlerThread.run(HandlerThread.java:65)
     Caused by: com.google.android.exoplayer2.audio.AudioSink$InitializationException: AudioTrack init failed: 0, Config(32000, 12, 32000)
        at com.google.android.exoplayer2.audio.DefaultAudioSink$Configuration.buildAudioTrack(DefaultAudioSink.java:1415)
        at com.google.android.exoplayer2.audio.DefaultAudioSink.initialize(DefaultAudioSink.java:514)
        at com.google.android.exoplayer2.audio.DefaultAudioSink.handleBuffer(DefaultAudioSink.java:595)
        at com.google.android.exoplayer2.audio.MediaCodecAudioRenderer.processOutputBuffer(MediaCodecAudioRenderer.java:732)
        at com.google.android.exoplayer2.mediacodec.MediaCodecRenderer.drainOutputBuffer(MediaCodecRenderer.java:1507) 
        at com.google.android.exoplayer2.mediacodec.MediaCodecRenderer.render(MediaCodecRenderer.java:653) 
        at com.google.android.exoplayer2.ExoPlayerImplInternal.doSomeWork(ExoPlayerImplInternal.java:575) 
        at com.google.android.exoplayer2.ExoPlayerImplInternal.handleMessage(ExoPlayerImplInternal.java:326) 
        at android.os.Handler.dispatchMessage(Handler.java:102) 
        at android.os.Looper.loop(Looper.java:164) 
        at android.os.HandlerThread.run(HandlerThread.java:65) 
```

To solve this, we have to track how many initializations happens and releases. The simple way to do this is to filter the logcat by `ExoPlayerImpl` 

```
Init b27c0c5 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 47cabd7 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init e299218 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release e299218 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init 84cae8b [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 47a1ebd [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 2b65814 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release 2b65814 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init c5b3e6 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init e364140 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init a7adbbe [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release a7adbbe [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init 35bee65 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 25a4fc7 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 39147e8 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release 39147e8 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init 8749a97 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init ad6ca69 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init ef20e73 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release ef20e73 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init f79bc02 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 623a87c [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init eaa033c [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release eaa033c [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init 5fd9dcd [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 99a43ef [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 77a19e0 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release 77a19e0 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init 21ae6b3 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init d8455a5 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Init 971d6c2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
Release 971d6c2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
Init d1895ef [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
```

As we can see from the sample above, exoplayer was initiated more than it released. When this reaches a certain amount, the exception will be thrown.

The solution can be as simple as null checking before initializing it

```
   if(player != null)
      return

   player = ExoPlayer(context)
   player?.initialize()
```

This is what it looks like after null checking

```

I/ExoPlayerImpl: Init cfbe42d [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Init 369091e [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Init 74f26bf [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 74f26bf [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init 3a6dbd2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 3a6dbd2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init 6b25a8d [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 6b25a8d [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init 4dda146 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 4dda146 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init 96c7128 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 96c7128 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init 9068361 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release 9068361 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init a4681ac [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release a4681ac [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init b6c38f2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release b6c38f2 [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init d62768b [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release d62768b [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
I/ExoPlayerImpl: Init e65a93c [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29]
I/ExoPlayerImpl: Release e65a93c [ExoPlayerLib/2.10.4] [taimen, Pixel 2 XL, Google, 29] [goog.exo.core, goog.exo.ui, goog.exo.hls]
```

I hope this is useful and helps someone who has the same problem.
        