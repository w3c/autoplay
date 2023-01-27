# Autoplay Policy Detection

This repository contains the Autoplay Policy Detection specification that is being worked on in the W3C [Media Working Group](https://www.w3.org/media-wg/).

## Introduction

Most user agents have their own mechanisms to block autoplaying media, and those mechanisms are implementation-specific. Web developers need to have a way to detect if autoplaying media is allowed or not in order to make actions, such as selecting alternate content or improving the user experience while media is not allowed to autoplay. For instance, if a user agent only blocks audible autoplay, then web developers can replace audible media with inaudible media to keep media playing, instead of showing a blocked media which looks like a still image to users. If the user agent does not allow any autoplay media, then web developers could stop loading media resources and related tasks to save the bandwidth and CPU usage for users.

Currently, this specification only handles HTMLMediaElement (video and audio) and Web Audio API. This specification does not handle Web Speech API and animated image (GIF animation).

## API

Autoplay detection can be performed through the Navigator object. The result can either allow authors to know if media, which have the same type of the given media type and exist in the document contained in the Window object associated with the queried Navigator object, are allowed to autoplay, or to know if a specific element is allowed to autoplay.

```
enum AutoplayPolicy {
  "allowed",
  "allowed-muted",
  "disallowed"
};

enum AutoplayPolicyMediaType { "mediaelement", "audiocontext" };

[Exposed=Window]
partial interface Navigator {
  AutoplayPolicy getAutoplayPolicy(AutoplayPolicyMediaType type);
  AutoplayPolicy getAutoplayPolicy(HTMLMediaElement element);
  AutoplayPolicy getAutoplayPolicy(AudioContext context);
};
```

## Examples

An example for checking whether authors can autoplay a media element.

```
if (navigator.getAutoplayPolicy("mediaelement") === "allowed") {
  // Create and play a new media element.
} else if (navigator.getAutoplayPolicy("mediaelement") === "allowed-muted") {
  // Create a new media element, and play it in muted.
} else {
  // Autoplay is disallowed, maybe show a poster instead.
}
```

An example for checking whether authors can start an audio context. Web Audio uses sticky activation to determine if AudioContext can be allowed to start.

```
if (navigator.getAutoplayPolicy("audiocontext") === "allowed") {
  let ac = new AudioContext();
  ac.onstatechange = function() {
    if (ac.state === "running") {
      // Start running audio app.
    }
  }
} else {
  // Audio context is not allowed to start. Display a bit of UI to ask
  // users to start the audio app. Audio starts via calling ac.resume()
  // from a handler, and 'onstatechange' allows knowing when the audio
  // stack is ready.
}
```

An example of querying by a specific media element.

```
function handlePlaySucceeded() {
  // Update the control UI to playing.
}
function handlePlayFailed() {
  // Show a button to allow users to explicitly start the video and
  // display an image element as poster to replace the video.
}

let video = document.getElementById("video");
switch (navigator.getAutoplayPolicy(video)) {
  case "allowed":
    video.src = "video.webm";
    video.play().then(handlePlaySucceeded, handlePlayFailed);
    break;
  case "allowed-muted":
    video.src = "video.webm";
    video.muted = true;
    video.play().then(handlePlaySucceeded, handlePlayFailed);
    break;
  default:
    // Autoplay is not allowed, no need to download the resource.
    handlePlayFailed();
    break;
}
```

An example of querying by a specific audio context. Web Audio uses sticky activation to determine if AudioContext can be allowed to start.

```
let ac = new AudioContext();
if (navigator.getAutoplayPolicy(ac) === "allowed") {
  ac.onstatechange = function() {
    if (ac.state === "running") {
      // Start running audio app.
    }
  }
} else {
  // Display a bit of UI to ask users to start the audio app.
  // Audio starts via calling ac.resume() from a handler, and
  // 'onstatechange' allows knowing when the audio stack is ready.
}
```
