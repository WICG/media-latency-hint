
# Media latencyHint explained

This explainer describes a new `latencyHint` attribute to be added to [HTMLMediaElement](https://html.spec.whatwg.org/multipage/media.html#htmlmediaelement). The document explains the goals and non-goals of the API and in general helps understand the thought process behind the design. The API shape in the document is mostly for information and might not be final.

## Objective
To give web authors influence over how much decoded output must be buffered for a user agent to begin playback or resume following a seek.

## Background

All user agents delay starting/resuming playback until some minimum number of frames are decoded and ready for rendering. This buffer provides an important cushion against playback stalls that might otherwise be caused by intermittent decoder slowness. It also adds a small amount of latency to start playback or resume after seeking.

Today the buffer thresholds are a UA-specific implementation detail. For example, Chromium defaults to require 200 milliseconds of decoded audio and 3 decoded video frames. In the absence of any site-provided hint, UAs will try to balance smoothness and latency, but generally favor smoothness as non-real-time video use-cases (movies, tv, vlogging, ...) are the most popular types of content.

Chromium currently offers an unspecified and undocumented "low delay" mode, which reduces the video threshold from 3 frames to 1. This is triggered by videos who's metadata does not specify a duration. Unfortunately, this sort of video is common on the web and the mode is most often triggered unintentionally by web authors who are unaware of the side effects.

## Use cases

The most obvious use cases would request lower latency. For certain applications, prioritizing smoothness over latency makes the wrong trade off. Examples include interactive video applications like cloud gaming. When a user moves their mouse to control the camera or presses a button to jump over an obstacle, even sub-second latency can make the game unplayable. A similar use case would be remote desktop streaming.

In the other direction, some sites may prefer latency to be higher than the UA default. For example, the non-real-time video use-cases mentioned earlier (movies, tv, vlogging, ...) may choose values like 300 or 500 msec to reduce the chance of decoder stalls.

## Shape and Semantics

We propose latencyHint as a `double`, with units being (partial) seconds.

```JavaScript
partial interface HTMLMediaElement {
    attribute double latencyHint;
};
```

An earlier proposal considered coarse enum categories (e.g. "default" and "minimum"). This is less powerful for site authors and more difficult for implementers to get right. Any reduction in decoded frame buffering will increase risk of decoder underflow. A category like "minimum" forces implementers to choose how close to zero is still usable with only a coarse notion of the site behavior.  Instead, letting sites choose a the target value avoids this issue lets sites make site-specific decisions.

The proposal is to use a double with units in seconds. The granularity of seconds is larger than we need (no one is expected to want several seconds of latency), but this is the time-unit preferred  in HTMLMediaElement and the double type can easily be used to express fractions of a second with sufficient precision.

**The provided value is a only hint**. There are limits on both the low and high ends to what a UA can reasonably offer.
* On the low end, the UA is likely constrained by the platform (e.g. the OS dependent size of the audio rendering buffer). A value of zero will be taken to mean "provide the absolute minimum latency".
* On the high end the UA is constrained by available memory. The effective max value will depend on the video resolution and framerate.


## Usage

Sites are encouraged to set the hint as early as possible, even before setting the `src` attribute. This offers UAs the biggest opportunity to minimize buffering as they setup the media pipeline. For example, UAs may configure the OS audio rendering buffer to optimize for the target latency in the initial stages of pipeline setup without the possibility of re-initializing later on.

### Example
```JavaScript
// Create the element, set the hint to indicate 75 msec of decoded buffer latency.
let videoElement = document.createElement('video');
videoElement.latencyHint = 0.075;

// Now setup playback (MSE, file, stream, ...) just as you do today.
videoElement.src = ...
```
