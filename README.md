# media-latency-hint

This is the repository for media-latency-hint. You're welcome to
[[contribute]](CONTRIBUTING.md)!


# How to try it in Chrome
* Available in version 83 or higher.
* Start chrome using the flag --enable-blink-features=MediaLatencyHint to enable the feature. 
* Set <video>.latencyHint as early as possible, even before setting a source (src). Particularly important for hint values that lower the default latency (< 0.2). If you set a source first we will begin prerolling with the default latency, which defeats the purpose.
* <video>.latencyHint will initially default to NaN. In this mode we render with Chrome's default preroll queue sizes (4 frames of video, 200 ms of audio). 
* <video>.latencyHint = 0 is a special value that indicates "bare minimum". This will be 1 frame of video and generally 40 ms of audio. The audio limit may be higher if the OS requires (say you're using a bluetooth headset). 
** Only use the bare minimum if that's truly what you want. If you'd be happy with 100ms of latency, you're much better off specifying that instead (balance latency and risk of decoder underflow). 
** Please let me know what you think of the 40ms lower bound. We may be able to go lower by asking the OS for a smaller buffer (users more battery power). 
* The provided latencyHint will also be clamped internally to a max size. The current limits are 24 frames of video and 3 seconds of audio. These are somewhat arbitrary, but the main concern with video is memory. 
* Once the video is loaded you can use chrome://media-internals to see how the latencyHint is interpreted by the audio and video renders (i.e. was the hint clamped?).
* There is a quirk with clearing the hint. Ideally you would set <video>.latencyHint = NaN (the initial value), but this doesn't work yet. For now you must call <video>.removeAttribute('latencyhint') (all lower case for attribute name) to restore the default buffering mode. 
