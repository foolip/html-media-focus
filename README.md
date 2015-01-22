## HTML Media Focus

### Introduction

This proposal enables web pages to request media focus for any HTML media and for user agents to then _reflect_ the state and events of that media between the page and any hardware and software based media control interfaces available to a user.

*Media focus* allows us to use hardware and software-based media controls to control and interact with ongoing web-based media. Examples of such media controls include, but are not limited to: headphone buttons, homescreen buttons and remote control buttons.

For the purpose of setting _media focus_ we define the following:

- a new empty content attribute on `[<video>](https://html.spec.whatwg.org/multipage/embedded-content.html#the-video-element)` and `[<audio>](https://html.spec.whatwg.org/multipage/embedded-content.html#the-audio-element)` elements called `focusable`, and;
- a reflected attribute for the new content attribute (above) on `[HTMLMediaElement](https://html.spec.whatwg.org/multipage/embedded-content.html#htmlmediaelement)` called `focusable`.

### Example Usage

A media element can request media focus with the `focusable` content attribute:

``` html
<video src="myvideo" autoplay controls focusable>
<audio src="audio.mp3" focusable>
```

A media element can also request media focus with the reflected `focusable` attribute:

``` html
<script>
  var myAudio = document.createElement('audio');
  myAudio.src = "audio.mp3";
  myAudio.focusable = true;
  // myAudio.outerHTML === "<audio src="audio.mp3" focusable=""></audio>"
</script>
```

Any HTML media element that has a `focusable` content attribute is called a *focusable media element*.

Whenever a `playing` event is fired toward a _focusable media element_ it obtains _media focus_ and is now the *focused media element*. Only one _focusable media element_ can hold _media focus_ at a time. If another _focusable media element_ currently has _media focus_ then the user agent actively pauses that other _focusable media element_ before passing _media focus_ to the new element.

Any _focusable media element_ can re-gain _media focus_ whenever a `playing` event is fired toward that element (i.e. whenever `.pause()` or `.play()` are called by the developer via JavaScript or by the user via HTML media controls against that _focusable media element_).

### IDL

HTML Media Focus can be described more formally as follows:

``` WebIDL
partial interface [HTMLMediaElement](https://html.spec.whatwg.org/multipage/embedded-content.html#htmlmediaelement) {
  attribute boolean focusable;
}

partial interface [MediaController](https://html.spec.whatwg.org/multipage/embedded-content.html#mediacontroller) {
  attribute boolean focusable;
}
```

By default, `focusable` is always initially set to `false`.

Additional [<video> content attributes](https://html.spec.whatwg.org/multipage/embedded-content.html#the-video-element):
  `focusable` - Whether to provide _media focus_ to this media resource on play

Additional [<audio> content attributes](https://html.spec.whatwg.org/multipage/embedded-content.html#the-audio-element):
  `focusable` - Whether to provide _media focus_ to this media resource on play

### Design FAQ

#### Why is the scope of media focus on &lt;audio&gt; and &lt;video&gt; elements?

Different OS platforms introduce different requirements to allow applications to gain media focus. Having studied the different approaches, scoping on &lt;audio&gt; and &lt;video&gt; elements allows media focus to be applied in a fully cross-platform way. It also ensures that _media focus_ is only provided when media actually begins playing in the user agent and it provides an excellent foundation in which we can reflect media control interface key presses against HTML media content and vice versa (see next question for more details).

#### How does this enable interaction with hardware and software media control interfaces?

We can reflect a _focused media element_'s state toward hardware and software based media control interfaces using logical mappings between the in-page `focused media element` and any available media control interface keys.

When a user presses a button on any media control interface, its logical meaning can be supplied to the `focused media element` via the existing infrastructure of HTMLMediaElement. For example, when a user clicks 'Play / Pause' on their headphone cord the user agent can then apply either a `play()` or `pause()` action, as appropriate, directly to the `focused media element`.

_Media focus_ enables user agents to relay standard media control interface key presses as [standard media events](https://html.spec.whatwg.org/multipage/embedded-content.html#mediaevents) toward the `focused media element` as follows:

- Play
- Pause
- Seek forward
- Seek backward
- Volume

Similarly, any changes to the in-page `focused media element` state can be reflected back to media control interfaces, such as [duration](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-media-duration) and [current time](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-media-currenttime). For example, if a user pauses the `focused media element` via any in-page media controls interface then this state can also be automatically reflected in all attached media control interfaces.

#### What about the forward and backward media control interface keys?

Many types of media control interfaces allow users to skip forward and backward to previous and next tracks in a 'playlist' like way. HTML currently avoids the next to statefully create and consume playlist media declaratively. Instead, such playlist-like functionality can be implemented by web applications in JavaScript.

To enable playlist-like media control interface keys to be fired toward HTMLMediaElement's we thus propose only firing `previous` and `next` events at the current `focused media element`. When a user presses the 'next' button in their media control interface we would thus fire a `next` event directly at the `focused media element` and the web page can either handle that event or not.

The addition of these two events allows us to relay standard previous/next media control interface key presses toward the `focused media element` as follows:

- Skip forward
- Skip backward

When we fire these events on the `focused media element` it enables that element, through any appropriate media control interface, to skip forward and backward between tracks without playlist-like functionality needing to be pre-arranged ahead of time or declared in HTML.

If another `focusable media element` gains _media focus_ then future `next` and `previous` events will be fired only toward this object when it is the `focused media element`. Thus, we can enable full media controls to be used within web applications with the addition of these two events.

#### What about displaying media information in media control interfaces?

Some media control interfaces, such as home screen controls, allow a title and an icon of the currently playing media to be displayed to the user. There is currently no way to provide this type of media information toward a media control interface.

With the following additions, each with a reflected &lt;audio&gt; and &lt;video&gt; content attribute so they can also be declared in HTML, we can enable the display of media information in media control interfaces as follows:

``` WebIDL
partial interface HTMLMediaElement {
  attribute DOMString? title;
  attribute USVString? icon;
}
```

With these additons, it would be possible for a media control interface to display simple metadata for the currently playing media for display in its interface. If title or icon metadata is not provided then suitable defaults could be provided (e.g. the page's title and/or the page's favicon).

#### What about the Web Audio API?

Firstly we should clearly state that there will be legitimate cases when media mixed through the Web Audio API should also be able to obtain _media focus_. The current blocker on enabling Web Audio API-generated media to hold 'audio focus' is the lack of suitable reflection between hardware and software based media control interfaces and Web Audio API objects.

Creating a new `AudioContext` object and then allowing that object to obtain `media focus` introduces a disconnect between the controls available in any typical media controls interface and the methods and events available to an `AudioContext` object.

At the current time, web developers are required to implement their own media interfaces for Web Audio API-generated content to introduce concepts like Play, Pause, Media Seeking and Media Skipping. It would be logical if web developers could re-use the machinery of HTMLMediaElement for this purpose.

If/when we are able to import Web Audio API-generated content in to an HTMLMediaElement object then we would be able to supply _media focus_ to this content in the same way as described in this document. Until that happens, supplying _media focus_ to Web Audio API generated content remains out-of-scope of this proposal.

### Demo

This repository contains a simple web-based demo that demonstrates how `focusable` should work and be enforced by user agents. You can view this demo [here](https://richtr.github.io/html-media-focus/).

### Feedback

If you have any questions or feedback on this proposal please [file an issue](https://github.com/richtr/html-media-focus/issues) against this repository and we can discuss further.
