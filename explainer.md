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

### Promise based API that returns an object

An alternate design is for a promise based API that returns an object. AutoplayPolicyType here is an enum of the different policies defined above. In this design, a site would call getAutoplayPolicy the first time to get the object and then the object will be updated. This has the advantage that the browser does not need to determine the autoplay policy until getAutoplayPolicy is called. This design has the advantage that is allows the website to warm up the code path and avoid delays due to asynchronousity if they want to.

```javascript
interface AutoplayPolicy : EventTarget {
  readonly attribute AutoplayPolicyType type;
  attribute EventHandler onchange;
};

partial interface Document {
  readonly attribute Promise<AutoplayPolicy> getAutoplayPolicy;
};

partial interface HTMLMediaElement {
  Promise<boolean> canAutoplay(); 
};
```

The first example from above using this design would be as follows:

```javascript
let policy = await document.getAutoplayPolicy();

if (policy.type == “allowed”) {
  loadUnmutedVideo();
} else {
  loadMutedVideo();
}
```

The second example from above using this design would be as follows:

```javascript
let policy = await document.getAutoplayPolicy();

video.src = “video.webm”;
video.muted = !await video.canAutoplay();
video.play();
```

### Event Listener Issue

We received some feedback that if you have a bunch of videos on the page that are all listening to the autoplay policy which becomes allowed and they all start playing together.

## Considered Alternatives

### Autoplay Permission

We considered having autoplay as a permission. However, autoplay is not something the site should be able to request; instead autoplay should be something the browser applies to a site.
