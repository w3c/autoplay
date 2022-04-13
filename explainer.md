# Autoplay Detection API

## Problem

If a website plays a video it cannot detect whether it will be allowed to autoplay or whether that autoplay will be blocked until it calls play on the media element.

## Goal

A site should be able to determine whether a video will autoplay or not. Knowing this in advance is important for a site as it may want to adjust the user experience or select alternate content to use instead.

## Proposed API Design

The document has an autoplay policy which can change. There will be an API on the document to get the autoplay policy and another on the media element which will return a boolean as to whether the media will be able to autoplay.

There are three types of autoplay policy:

 * `allowed` - autoplay is allowed
 * `allowed-muted` - muted autoplay is allowed
 * `disallowed` - document level autoplay is not allowed. The site should check on the element whether element level autoplay is allowed

The API is defined as the following:

```javascript
enum AutoplayPolicy {
  "allowed",
  "allowed-muted",
  "disallowed"
};

enum AutoplayPolicyMediaType {
  "mediaelement",
  "audiocontext"
};

partial interface Navigator {
  AutoplayPolicy getAutoplayPolicy(AutoplayPolicyMediaType type);
  AutoplayPolicy getAutoplayPolicy(HTMLMediaElement element);
  AutoplayPolicy getAutoplayPolicy(AudioContext context);
};
```

## Sample Code

### A site would like to change the source based on whether it will autoplay

In this situation a site would like to change the source based on whether it will autoplay. In this example the site will play `video.webm` if it can autoplay unmuted. If it cannot then it will use `video-alt.webm` instead.

```javascript
const policy = navigator.getAutoplayPolicy("mediaelement");

if (policy === "allowed") {
  loadUnmutedVideo();
} else {
  loadMutedVideo();
}
```

### A site would like to change its experience based on whether it will autoplay

In this situation a site would like to change its experience based on whether it will autoplay. In this example if it cannot autoplay unmuted then it will mute the video element.

```javascript
const video = document.getElementById("video");
const policy = navigator.getAutoplayPolicy("mediaelement");

video.src = "video.webm";

if (policy === "allow-muted") {
  video.muted = true;
}

video.play();
```

## Alternatives Considered

### Event Listener

We considered having an event listener to allow sites to listen to changes in autoplay policy. However, this may cause issues where there are multiple event listeners on a page that all start playing when the autoplay policy changes.

### Autoplay Permission

We considered having autoplay as a permission. However, there are a few differences between the autoplay policy and permissions:

 * Autoplay policy is something that is applied to the site from the browser; rather than something that should be requested
 * Autoplay policy has multiple states (e.g. allowed, allowed-muted); rather than a yes or no
 * Autoplay policy is not implemented by user agents as a permission
 * Permissions are async
