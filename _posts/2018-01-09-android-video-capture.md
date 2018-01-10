---
layout: post
title:  "Android Video Capture
date:   2018-01-09 13:25:00 -0400
categories: android  debugging hacking
---

This is the oddest thing I've run across.  I followed all of the tutorials using Android's Camera Intent for capturing a video and was reusing code that uploads the contents to an S3 bucket (via Carrier Wave in Rails).  When I uploaded the mp4, it would set the Content-Type to video/mp4, I could download and play the file, I could view it in Android chrome and Safari (OS X and iOS).  But Firefox and Chrome on my Mac would not play it.

Firefox at least gave an error:
````
Media resource https://<SNIP>/video.mp4?13134 could not be decoded.
video.mp4
Media resource https://<SNIP>/video.mp4?13134 could not be decoded, error: Error Code: NS_ERROR_DOM_MEDIA_DEMUXER_ERR (0x806e000c)
Details: virtual RefPtr<MP4Demuxer::InitPromise> mozilla::MP4Demuxer::Init(): No MP4 audio () or video () tracks

````
I checked the info from Finder and saw that it was an mp4.  ffmpeg said (via ffmpeg -i bad.mp4)


````
ffmpeg version 3.4.1 Copyright (c) 2000-2017 the FFmpeg developers
  built with Apple LLVM version 9.0.0 (clang-900.0.39.2)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/3.4.1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      55. 78.100 / 55. 78.100
  libavcodec     57.107.100 / 57.107.100
  libavformat    57. 83.100 / 57. 83.100
  libavdevice    57. 10.100 / 57. 10.100
  libavfilter     6.107.100 /  6.107.100
  libavresample   3.  7.  0 /  3.  7.  0
  libswscale      4.  8.100 /  4.  8.100
  libswresample   2.  9.100 /  2.  9.100
  libpostproc    54.  7.100 / 54.  7.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '/tmp/delete-me.mp4':
  Metadata:
    major_brand     : mp42
    minor_version   : 0
    compatible_brands: isommp42
    creation_time   : 2018-01-09T02:27:06.000000Z
    location        : +37.4220-122.0839/
    location-eng    : +37.4220-122.0839/
    com.android.version: 7.0
  Duration: 00:00:01.42, start: 0.000000, bitrate: 2470 kb/s
    Stream #0:0(eng): Video: mpeg4 (Simple Profile) (mp4v / 0x7634706D), yuv420p(tv, smpte170m/smpte170m/bt709), 320x240 [SAR 1:1 DAR 4:3], 167 kb/s, 14.29 fps, 14.83 tbr, 90k tbn, 1k tbc (default)
    Metadata:
      rotate          : 90
      creation_time   : 2018-01-09T02:27:06.000000Z
      handler_name    : VideoHandle
    Side data:
      displaymatrix: rotation of -90.00 degrees
    Stream #0:1(eng): Audio: amr_nb (samr / 0x726D6173), 8000 Hz, mono, flt, 12 kb/s (default)
    Metadata:
      creation_time   : 2018-01-09T02:27:06.000000Z
      handler_name    : SoundHandle
At least one output file must be specified
````

For a "good" file that I captured from an iOS app:
````
ffmpeg version 3.4.1 Copyright (c) 2000-2017 the FFmpeg developers
  built with Apple LLVM version 9.0.0 (clang-900.0.39.2)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/3.4.1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      55. 78.100 / 55. 78.100
  libavcodec     57.107.100 / 57.107.100
  libavformat    57. 83.100 / 57. 83.100
  libavdevice    57. 10.100 / 57. 10.100
  libavfilter     6.107.100 /  6.107.100
  libavresample   3.  7.  0 /  3.  7.  0
  libswscale      4.  8.100 /  4.  8.100
  libswresample   2.  9.100 /  2.  9.100
  libpostproc    54.  7.100 / 54.  7.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '/tmp/good.mp4':
  Metadata:
    major_brand     : qt
    minor_version   : 0
    compatible_brands: qt
    creation_time   : 2017-11-30T20:01:58.000000Z
    com.apple.quicktime.make: Apple
    com.apple.quicktime.model: iPhone 6s
    com.apple.quicktime.software: 11.1.2
    com.apple.quicktime.creationdate: 2017-11-30T13:34:05-0500
  Duration: 00:00:01.94, start: 0.000000, bitrate: 5217 kb/s
    Stream #0:0(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, mono, fltp, 88 kb/s (default)
    Metadata:
      creation_time   : 2017-11-30T20:01:58.000000Z
      handler_name    : Core Media Data Handler
    Stream #0:1(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p(tv, bt709), 1280x720, 5114 kb/s, 29.97 fps, 29.97 tbr, 600 tbn, 1200 tbc (default)
    Metadata:
      rotate          : 90
      creation_time   : 2017-11-30T20:01:58.000000Z
      handler_name    : Core Media Data Handler
      encoder         : H.264
    Side data:
      displaymatrix: rotation of -90.00 degrees
At least one output file must be specified
````

The camera capture code in android is simple:
````java
// capture
  String intentAction = MediaStore.ACTION_VIDEO_CAPTURE;
  Intent cameraIntent = new Intent(intentAction);
  context.startActivityForResult(cameraIntent, cameraMode);
````

The multipart body code is virtually identical to the jpeg upload code
````java
// upload (finalFile comes from from intent.getData)
  RequestBody requestImage = RequestBody.create(MEDIA_TYPE_MP4, finalFile);
  return new MultipartBody.Builder().setType(MultipartBody.FORM).addFormDataPart("comment[doc]", "video.mp4", requestImage).build();

````

I figured out today the reason why it wasn't working.  I was taking videos with the emulator!  The codecs are on the hardware and as far as I can tell, the emulator doesn't have them.  I hope this helps future hackers if anyone runs into this problem!
