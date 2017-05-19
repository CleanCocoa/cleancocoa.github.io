---
title: WebcamSnap Open Source Library Released
created_at: 2017-05-19 15:33:58 +0200
kind: worklog
tags: [ opensource, avfoundation ]
image: blog/20170519153500_webcam-snap.png
url: https://github.com/CleanCocoa/WebcamSnap
comments: on
---

{% include figure.md src="/assets/blog/2017/20170519153500_webcam-snap.png" caption="The picture cropping sheet in action" %}

Another month goes by, another little macOS component was released.

This time, I wanted to have a very simple drop-in view component that takes pictures with the iSight camera of MacBooks or any other connected USB camera device. And if the user wants to crop the picture, I wanted to add that as post-processing.

Thought this would be a common need, but Google wasn't helping much. So I coded this thing myself.

Have a look: **<https://github.com/CleanCocoa/WebcamSnap>**

## How Taking Pictures with AVFoundation Works

Before I got it working, all the AVFoundation code looked pretty scary. But it's simple, really, when you think about the possibilities of audio and video capturing and processing. 

These are the requirements to take a photo with a USB camera using AVFoundation:

1. you need a `AVCaptureSession` that controls the lifetime of the audio and/or video input and/or output;
2. you specify output ports of type `AVCaptureOutput`, like the `AVCaptureStillImageOutput` I use here to grab a single image (instead of video or audio);
3. you specify input ports of type `AVCaptureInput`, like the `AVCaptureDeviceInput` that takes a `AVCaptureDevice`, which in this case is of type `AVMediaTypeVideo`;
4. you add a Core Animation layer (`CALayer`) to a preview view using `AVCaptureVideoPreviewLayer(session:)` so users see what the camera is showing before they hit the "Take Picture" button.

If things work out well, you have a session that can read video data from your camera device and streams the video signal to the preview. You can shoot photos, also known as "request still images" from the video stream, using `captureStillImageAsynchronously` on your `AVCaptureStillImageOutput`.

All of this setup is encapsulated in [the `Webcam` object of my library.](https://github.com/CleanCocoa/WebcamSnap/blob/7a90a386ec2d0f1532a988a57e4dc502ce76c135/WebcamSnap/Webcam.swift) (links to the version of v1.0.0, no the latest, to prevent dead links in the future).

Hope it helps!
