---
title: Our video recording setup
draft: true
---

We bought a Pyle [INSERT THE THING] with two headest and two lavalier
microphones. Cost 180$ CAD, but I wanted to fiddle with it anyway.

YouTube Live lists the supported streaming encoders here:

  https://support.google.com/youtube/answer/2907883?hl=en&pageId=113024377635277809206

Being a fan of open source (and being on Linux most of the time), I
spotted OBS there in that list as an open source method for streaming
content to YouTube Live.

I was looking at this on February 7th and noticed Open Broadcasting Software released a new version on Feb 4th !  Sounded like I should try it today.  You can download it from:

  https://obsproject.com/download#mp

OBS has great docs over here:

  http://jp9000.github.io/OBS/

After trying out live streaming to YouTube, loading local videos,
streaming my desktop, It works really great (tried here under Linux).

The YouTube Live features are also pretty impressive, you can start
streaming with OBS and your YouTube Live channel starts streaming.  Or
you can schedule an event, and then setup multiple cameras to stream
their content to YouTube, and use the web-based "Live Control Room" to
move from one to another.

To configure client's x11grab (Linux), dshow (Windows) or avfoundation (Mac) for streaming, see:

  https://trac.ffmpeg.org/wiki/Capture/Desktop

along with

  https://trac.ffmpeg.org/wiki/EncodingForStreamingSites


### Capturing the presenters' desktops and slides.

I thought I'd need an RTMP server.. instructions ot use nginx as an RTMP server here https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/ but I wanted to use Go, right ?

As I went by, I discovered https://github.com/nllptr/zazen and a bunch of references to SRS (Simple-RTMP-Streaming) servers. I had a hard time finding which one I could install because of lightweight README.md files (always treat your README.md with great care!)

I finally found https://github.com/ossrs/go-oryx which is an SRS implementation in Go, by those who originally did it in C++.


### OBS Remote

For Windows users, there is also a OBS remote control (browser based) here:

  http://client.obsremote.com/
