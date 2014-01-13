#Network Information API v3

## Introduction

### Use cases and requirements
This document attempts to address the [use case and requirements for network information](https://github.com/w3c-webmob/netinfo).

### Previous versions

* [Version 1](http://www.w3.org/TR/2011/WD-netinfo-api-20110607/)
* [Version 2](http://www.w3.org/TR/netinfo-api/)

### What's new in this version?

* Take back the `type` property introduced in [version 1](http://www.w3.org/TR/2011/WD-netinfo-api-20110607) with a different set of possible values.
* Remove the `bandwidth` and `metered` properties introduced in [version 2](http://www.w3.org/TR/netinfo-api/).
* Keep the `onchange` event introduced in [version 2](http://www.w3.org/TR/netinfo-api/).

Go to the [Discussion](#discussion) section for a rationale about the changes introduced in this version.

## Extensions to the Navigator object

``` 
partial interface Navigator {
   readonly attribute Connection connection;
};
```

### The `connection` attribute
The `connection` attribute provides access to information about the network connection the user agent is currently using.

### The `Connection` interface
The `Connection` interface provides a means to access information about the network connection the user agent is currently using.

```
interface Connection : EventTarget {
    readonly attribute ConnectionType type;
             attribute EventHandler onchange;
};
```

### The `type` attribute
The `type` attribute, when getting, returns the type of connection that the user agent is using to communicate with the internet.

When the `Connection` changes, the user agent must queue a task which updates the `Connection` properties and fire a simple event named `change` at the `Connection` object.

## The `ConnectionType` enum

```
enum ConnectionType{ "unknown", "ethernet", "wifi", "cellular", "none"}
```

 * "unknown": the connection type is not known at this time. 
 * "ethernet": the user agent is using an ethernet connection. 
 * "wifi": the user agent is using a Wi-Fi connection. 
 * "cellular": the user agent is using a cellular connection (e.g., EDGE, 3G, 4G, etc.). 
 * "none": the user agent is not using any connection (it's explicitly offline, such as in "airplane" mode). 


## Examples

Based on the use cases detailed at [w3c-webmob/netinfo](https://github.com/w3c-webmob/netinfo).

### Example 1

This basic example shows how a video website can warn the user that watching a video on cellular might cost them money.

```html
<!DOCTYPE>
<html>
  <head>
    <title>ACME video</title>
  </head>
  <body>
    <video src="lolcats.ogg" controls></video>
    <script>
      function warn() {
        alert("This is a free ACME service. However, your operator may " +
              "charge you for the amount of data you use. If you are " +
              "unsure how much data costs on your tariff, please contact " +
              "your network operator.");
      }

      function startLoading() {
        video.setAttribute("src", "lolcats.ogg" );
        video.load();
      }

      var video = document.getElementsByTagName("video")[0];
      if (navigator.connection.type === "cellular") {
        warn();
      } else if (navigator.connection.type !== "none") {
        startLoading();
      }
    </script>
  </body>
</html>
```

### Example 2

This example shows how a mobile web application can apply a user preference about wether a download should happen only over WiFi and how can it warn the user about a download happening over cellular.

```html
<!DOCTYPE>
<html>
  <head>
    <title>ACME downloads over WiFi</title>
    <script>
      var gDownloading = false;

      function downloadsOnlyOverWiFi() {
        // Return true if the user allows downloads only over WiFi
      }

      function doDownload() {
        // Do download.
        gDownloading = true;
      }

      function download() {
        if (navigator.connection.type === "cellular") {
          if (downloadsOnlyOverWiFi()) {
            alert("You are connected to a mobile network and cannot download " +
                  "this file.");
            return;
          }
          alert("You are going to download a file over your mobile network. " +
                "To prevent this in the future, please visit Settings");
        }

        doDownload();
      }

      navigator.connection.addEventListener("change", function() {
        if (gDownloading && navigator.connection.type === "cellular" &&
            downloadsOnlyOverWiFi()) {
          alert("Your phone lost Wi-Fi so we continued your download on your "
                "mobile network. To prevent this in the future, go to Settings");
        }
      });
    </script>
  </head>
  <body>
    <button onclick="download();">Download</button>
  </body>
</html>
```

### Example 3

The example provides a simple metering application that monitors how much time
is the user connected to each type of connection.

```html
<!DOCTYPE>
<html>
  <head>
    <title>ACME connection monitor</title>
  </head>
  <body>
    <script>
      var then = new Date();
      var lastConnectionType = window.navigator.connection.type;

      function addTimeTo(connectionType, time) {
        /* Add `time` to `connectionType` */
      }

      function startMonitoring() {
        window.navigator.connection.addEventListener('change', function () {
          var now = new Date();
          addTimeTo(lastConnectionType, now - then);
          lastConnectionType = window.navigator.connection.type;
          then = now;
        })
      }

      window.onload = startMonitoring;
    </script>
  </body>
</html>
```

## Discussion

### Why not exposing a `bandwidth` property?

* We did not find any valid use case.

  * [Adaptive streaming according to network conditions](http://www.w3.org/community/coremob/wiki/Features/Network_Information_API#Poor_man.27s_adaptive_streaming) is not a use case that can be addressed by the Network Information API. Instead,  [DASH](http://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) and/or the [MediaSource Extensions API](https://dvcs.w3.org/hg/html-media/raw-file/default/media-source/media-source.html) are better suited to address adaptive streaming. Check this [interesting video](http://www.youtube.com/watch?v=UklDSMG9ffU) about this topic.

  * Use cases like the [example 2](http://www.w3.org/TR/netinfo-api/#examples) of the second version of this API, where an image viewer can select a low definition or a high definition image based on the current connection bandwidth, are being addressed by the [Responsive Images Community Group](http://responsiveimages.org/) - see in particular [the `picture` element](http://picture.responsiveimages.org) and [Client Hints](http://tools.ietf.org/html/draft-grigorik-http-client-hints). For performance reasons, the choice of which image to load should be left to the user agent (instead of scripts), based on the UA's knowledge of the screen’s pixel density or whatever other factors it deems relevant to the decision.

* Hard to estimate.

  * The [Outstanding issues section](http://www.w3.org/TR/netinfo-api/#outstanding-issues) of the second version of this API already mentions that _"`bandwidth` may be hard to implement and can be quite power-consuming to keep up-to-date"_ as it may require regularly sending test payloads that can also add extra costs to the user (probably very significant when roaming). _"Its value might be unrelated to the actual connection quality that could be affected by the server"_ and by other consumers of the same connection. It finally suggests a potential solution that could be _"to return non absolute values that couldn't be easily abused and would be more simple to produce for the user agent. For example, having a set of values like `very-slow`, `slow`, `fast` and `very-fast`. Another solution would be to have only values like `very-slow`, `slow` and the empty string"_. But this does not seem to be a good solution, even if the user agent is able to estimate the available bandwidth, the perception of network speed differs between consumers, what is considered fast enough for some network requests could be slow for others.
  * [Mozilla's Android implementation](https://mxr.mozilla.org/mozilla-central/source/mobile/android/base/GeckoNetworkManager.java#72) maps static values based on the connection type, which is basically the same as exposing the network type, and it is quite inaccurate if the intention is to expose the network speed available. The theoretical maximum speed for the current connection type might be far from the real current speed. It does not only depend on the location, time of day, number of active peers but it is also controlled by the network provider. I can be connected via LTE but if I already consumed my monthly data plan, my connection will be really slow. That's why making assumptions about the network speed based on the connection type is not appropriate and should be discouraged. (Note that none of the [use cases](https://github.com/w3c-webmob/netinfo#use-cases-and-requirements) specifically mention this).

### Why not exposing a `metered` property?

* We didn't find any use case that cannot be addressed by exposing only the `type` property.

* The user agent won't be able to know the value of this property without asking to the network provider or to the user about it. For the former, we would require network operators to provide an API which enables user agents to query the user's data plan and its associated costs. Apart from the obvious privacy issues, being a telco employee, I cannot see this happening any time soon. For the latter, it seems that developers can already ask the user if a specific connection type is metered or not and so keep a record of it if needed. So we cannot see any additional value that exposing a `metered` property could add to this API. In fact, existing implementations like [Android](http://androidxref.com/4.3_r2.1/xref/frameworks/support/v4/java/android/support/v4/net/ConnectivityManagerCompat.java#37) or [Firefox](https://mxr.mozilla.org/mozilla-central/source/mobile/android/base/GeckoNetworkManager.java#321) already map the cellular connection type to a metered connection.

### Why did we changed the `type` property values?

* Avoid potential fingerprinting issues.
* Avoid network speed assumptions based on the network connection type.
