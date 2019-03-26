# Autoplay Detection API

## Problem
If a website plays a video it cannot detect whether it will be allowed to autoplay or whether that autoplay will be blocked until it calls play on the media element.

## Goal
A site should be able to determine whether a video will autoplay or not. Knowing this in advance is important for a site as they may want to adjust their user experience or select alternate content to use instead.

## Proposed API Design
The document has an autoplay policy which can change. There will be an API on the document to get the autoplay policy and another on the media element which will return a boolean as to whether the media will be able to autoplay.

There are three types of autoplay policy:

 * `allowed` - autoplay is allowed
 * `allowed-muted` - muted autoplay is allowed
 * `disallowed` - document level autoplay is not allowed. The site should check on the element whether element level autoplay is allowed

The API is defined as the following:

```javascript
enum AutoplayPolicy {
  “allowed”,
  “allowed-muted”,
  “disallowed”,
  “unknown”
};

partial interface Document {
  Promise<AutoplayPolicy> getAutoplayPolicy();
};

partial interface HTMLMediaElement {
  Promise<boolean> canAutoplay(); 
};
```

The API should be async because determining whether a site can autoplay can require cross-process communications to gather data such as website settings or user preferences. In this design the browser does not need to calculate the autoplay policy until getAutoplayPolicy is called. Being async also does not limit browser implementations of what determines autoplay policy.

## Sample Code

### A site would like to change the source based on whether it will autoplay

In this situation a site would like to change the source based on whether it will autoplay. In this example the site will play `video.webm` if it can autoplay unmuted. If it cannot then it will use `video-alt.webm` instead.

```javascript
let policy = await document.getAutoplayPolicy();

if (policy == “allowed”) {
  loadUnmutedVideo();
} else {
  loadMutedVideo();
}
```

### A site would like to change its experience based on whether it will autoplay

In this situation a site would like to change its experience based on whether it will autoplay. In this example if it cannot autoplay unmuted then it will mute the video element.

```javascript
let policy = await document.getAutoplayPolicy();

video.src = “video.webm”;
video.muted = !await video.canAutoplay();
video.play();
```

## Open questions

### Sync vs Async

There is a strong disagreement about whether this API should be synchronous instead (the sync design is supported by both Apple and Mozilla). However, it would require all browsers to always be able to answer whether a website can autoplay at load time; regardless of their autoplay policies.

The synchronous API would be designed like this:

```javascript
partial interface Document {
  readonly attribute AutoplayPolicy autoplayPolicy;
  attribute EventHandler onautoplaypolicychange;
};

partial interface HTMLMediaElement {
  readonly boolean canAutoplay; 
};
```

The first example from above using this design would be as follows:

```javascript
if (document.autoplayPolicy == “allowed”) {
  loadUnmutedVideo();
} else {
  loadMutedVideo();
}
```

The second example from above using this design would be as follows:

```javascript
video.src = “video.webm”;
video.muted = !video.canAutoplay;
video.play();
```

### Event Listener Issue

We considered having an event listener to allow sites to listen to changes in autoplay policy. However, this may cause issues where there are multiple event listeners on a page that all start playing when the autoplay policy changes.

### Autoplay Permission

We considered having autoplay as a permission. However, there are a few differences between the autoplay and permissions:

 * Autoplay is something that is applied to the site from the browser; rather than something that should be requested
 * Autoplay has multiple states (e.g. allowed, allowed-muted); rather than a yes or no
 * Autoplay is not implemented by the user agents as a permission
 * Permissions are async
