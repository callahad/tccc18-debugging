# Debugging Mobile Web Performance

_Dan Callahan · [@callahad](https://twitter.com/callahad)_

[github.com/callahad/tccc18-debugging](https://github.com/callahad/tccc18-debugging/)

***

![](img/firefox_logo-wordmark-vert_RGB.png)
<!-- .element: style="max-width:50%" -->

***

## The Plan

1. Remote Debugging
2. Local Debugging
3. Performance
    1. Network
    2. Framerate
    3. Memory

---

## Remote Debugging

_With Real Hardware!_

***

### Native Tools

|                        | Windows | Mac     | Linux   |
| ---------------------- | ------- | ------- | ------- |
| __(Android) Chrome__   | Chrome  | Chrome  | Chrome  |
| __(Android) Firefox__  | Firefox | Firefox | Firefox |
| __(iOS) Safari__       |         | Safari  |         |
| __(iOS) Chrome__       |         |         |         |

***

### [Valence](https://developer.mozilla.org/docs/Tools/Valence)

Included with [Firefox Developer Edition](https://firefox.com/developer)

|                        | Windows | Mac     | Linux   |
| ---------------------- | ------- | ------- | ------- |
| __(Android) Chrome__   | Firefox | Firefox | Firefox |
| __(Android) Firefox__  | Firefox | Firefox | Firefox |
| __(iOS) Safari__       | Firefox | Firefox | Firefox |
| __(iOS) Chrome__       |         |         |         |

***

### DEMO

_(Connecting to various browsers)_

***

### [Weinre](http://people.apache.org/~pmuellr/weinre/docs/latest/)

"<u>We</u>b <u>in</u>spector <u>re</u>mote"

_Fallback for iOS browsers / webviews, Mobile IE, etc._

***

### Device Setup

- Android:

    1. Settings → About Phone → Build Number (Tap 7x)
    2. Settings → Developer Options → USB Debugging
    3. (Firefox) → Settings → DevTools → Remote Debugging

- iOS:

    1. Settings → Safari → Advanced → Web Inspector

***

### Computer Setup

- Android on Windows:

    1. Install [OEM USB Driver](http://developer.android.com/tools/extras/oem-usb.html)
    2. Connect Phone

- iOS on Mac OS X:

    1. Safari → Prefs → Advanced → Show Develop Menu
    2. Connect Phone

---

## Local Debugging

_Without Real Hardware!_

***

### Simulators / Emulators

Awesome, but cumbersome.

_"Can't I just use my desktop browser?"_

***

### DEMO

_(Touch Events)_

***

### Mobile vs. Desktop

| Good / Similar | Bad / Different |
| -------------- | --------------- |
| Same Engines   | Worse Network   |
| No Old IE      | Slower CPU      |
| Hardware Accel | Less RAM        |

***

### ORLY?

![](img/craptops.jpg)

***

> You __are not__ exempt from mobile testing.
>
> Or cross-browser testing.

***

### iOS WebViews were extra slow

_([They aren't anymore](http://developer.telerik.com/featured/why-ios-8s-wkwebview-is-a-big-deal-for-hybrid-development/).)_

* UIWebView doesn't use Safari's JIT.
* WKWebView (iOS 8) does.

---

## Measuring Performance

_You can't debug what you can't measure_

***

### Areas of Measurement

1. Network Performance
2. Frames Per Second
3. Memory Usage

---

## Network Performance

_Can you hear me now?_

***

### What do we care about?

1. Time until something appears.
2. Time until the page is interactive.

_Hack the first to get better __perceived speed__._

Note: Script tag at end of body, cache headers, etc.

***

![](img/facebook-shell.png)
<!-- .element: style="max-width:90%" -->

***

### Network Perf Tools

1. Get to know your Network Panel
2. [webpagetest.org](http://webpagetest.org)
3. [developers.google.com/speed/pagespeed/insights/](https://developers.google.com/speed/pagespeed/insights/)

***

![](img/netperf-chrome.png)

***

![](img/netperf-webpagetest.png)

***

![](img/netperf-pagespeed.png)

***

### Simulating Bad Connections

* [Clumsy](https://jagt.github.io/clumsy/)
* [Network Link Conditioner](http://nshipster.com/network-link-conditioner/)
* [Chrome DevTools](https://developer.chrome.com/devtools/docs/device-mode#network-conditions)

***

### Proxies

* [Fiddler](http://www.telerik.com/fiddler)
* [Charles](http://www.charlesproxy.com/)
* [mitmproxy](http://mitmproxy.org/)

***

### Obsolete Practices

1. Domain sharding
2. Concatenating sources
3. Combining images into sprites

Thanks, SPDY and HTTP/2!

***

### Still Good

1. Minify + Gzip
2. Compress / Remove Images
3. Serve appropriate images

Note: All about reducing page weight

---

## Framerate Performance

_60 FPS means 16 ms per frame_

***

### Rendering Stages

1. Recalculate Styles
2. Reflow / Layout
3. Paint
4. Composite

See http://csstriggers.com/

***

![](img/compositing-aerotwist.jpg)

_Illustration by Paul Lewis, <br> from [Pixels are Expensive](http://aerotwist.com/blog/pixels-are-expensive/)._

***

### Creating layers

```css
.foo {
    will-change: transform, opacity; /* New goodness */
    transform: translateZ(0); /* Old hack for Safari */
}
```

***

### Trivia: Selector Performance

No. | Type       | Example
--- | ---------- | -------
1.  | Identifier | `#id`
2.  | Class      | `.class`
3.  | Tag        | `div`
4.  | Sibling    | `a + i`
5.  | Descendant | `ul > li`
6.  | Universal  | `*`
7.  | Attribute  | `input[type="text"]`
8.  | Pseudo     | `a:hover`

Note: Right to left matching! Also, adding / removing classes is fast.

***

### Layout Thrashing, Part 1

```javascript
var h1 = element1.clientHeight;          // Read
element1.style.height = (h1 * 2) + 'px'; // Write (invalidates)

var h2 = element2.clientHeight;          // Read (forces reflow)
element2.style.height = (h2 * 2) + 'px'; // Write (invalidates)

var h3 = element3.clientHeight;          // Read (forces reflow)
element3.style.height = (h3 * 2) + 'px'; // Write (invalidates)

// Document reflows at end of frame
```

_From Wilson Page's [blog](http://wilsonpage.co.uk/preventing-layout-thrashing/)_

***

### Layout Thrashing, Part 2

```javascript
// Read
var h1 = element1.clientHeight;
var h2 = element2.clientHeight;
var h3 = element3.clientHeight;

// Write (invalidates layout)
element1.style.height = (h1 * 2) + 'px';
element2.style.height = (h2 * 2) + 'px';
element3.style.height = (h3 * 2) + 'px';

// Document reflows at end of frame
```

_From Wilson Page's [blog](http://wilsonpage.co.uk/preventing-layout-thrashing/)_

***

![](img/layout-thrashing.png)

***

### Need even more? Virtualize.

- [React](https://facebook.github.io/react/)
- [Mithril](https://lhorie.github.io/mithril/)
- [virtual-dom](https://github.com/Matt-Esch/virtual-dom)

***

### Animations

[YouMightNotNeedjQuery.com](http://youmightnotneedjquery.com/)

---

## Memory Performance

_Let's just do some demos here, too._

---

## Thanks!

_Dan Callahan · [@callahad](https://twitter.com/callahad)_

[github.com/callahad/tccc18-debugging](https://github.com/callahad/tccc18-debugging/)
