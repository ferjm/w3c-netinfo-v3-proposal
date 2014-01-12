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

    partial interface Navigator {
          readonly attribute Connection connection;
    };

### The `connection` attribute

* **connection** of type [Connection](#the-connection-interface), readonly. The object from which connection information is accessed.

### The Connection interface
The `Connection` interface provides a handle to the device's connection information.

    interface Connection : EventTarget {
        readonly attribute ConnectionType type;
                 attribute EventHandler onchange;
    };
    
    enum ConnectionType{ "unknown", "ethernet", "wifi", "cellular", "none"}

### The `type` attribute

* **type**. returns the type of connection that the user agent is using to communicate with the internet.

When the `Connection` changes, the user agent must queue a task which updates the `Connection` properties and fire a simple event named `change` at the `Connection` object.

## Examples

Based on the use cases detailed at [w3c-webmob/netinfo](https://github.com/w3c-webmob/netinfo).

### Example 1

This basic example shows how a video website can warn the user that watching a video on cellular might cost them money.

    <!DOCTYPE>
    <html>
      <head>
        <title>ACME video</title>
      </head>
      <body>
        <video src="lolcats.ogg" controls></video>
        <script>
          function pauseAndWarn() {
            video.pause();
            alert("This is a free ACME service. However, your operator may " +
                  "charge you for the amount of data you use. If you are " +
                  "unsure how much data costs on your tariff, please contact " +
                  "your network operator.");
          }

          var video = document.getElementsByTagName("video")[0];
          video.addEventListener("playing", function() {
            if (navigator.connection.type === "cellular") {
              pauseAndWarn();
            }
          });

          navigator.connection.addEventListener("change", function() {
            if (navigator.connection.type === "cellular" &&
                !video.paused && !video.ended && video.currentTime > 0) {
              pauseAndWarn();
            }
          });
        </script>
      </body>
    </html>

### Example 2

This example shows how a mobile web application can apply a user preference about wether a download should happen only over WiFi and how can it warn the user about a download happening over cellular.

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

### Example 3

This example shows how a web application can advise the user to activate Wi-Fi to improve location accuracy.

    <!DOCTYPE>
    <html>
      <head>
        <title>ACME maps</title>
      </head>
      <body>
        <script>
          function onPositionSuccess(position) {
            // Do map stuff.
          }

          function onPositionError() {
            // Show error.
          }

          function geoFindMe() {
            if (navigator.connection.type !== "wifi") {
              alert("Turning on Wi-Fi will improve location accuracy");
            }

            navigator.geolocation.getCurrentPosition(onPositionSuccess,
                                                     onPositionError);
          }

          window.onload = geoFindMe;
        </script>
      </body>
    </html>

## Discussion

### Why not exposing a `bandwidth` property?

* We did not find any valid use case.

  * [**Adaptive streaming according to network conditions**](http://www.w3.org/community/coremob/wiki/Features/Network_Information_API#Poor_man.27s_adaptive_streaming) does not seem to be an use case solvable by the Network Information API. In fact, it seems that [DASH](http://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) and/or the [MediaSource Extensions API](https://dvcs.w3.org/hg/html-media/raw-file/default/media-source/media-source.html) are already taking care of this use case in a different way.

  * Use cases like the [example 2](http://www.w3.org/TR/netinfo-api/#examples) of the second version of this API where an image viewer can select a low definition or a high definition image based on the current connection bandwidth are being solved by the [Responsive Images WG](http://responsiveimages.org/) and it seems that the choice of which image to load should be left to the user agent, based on its knowledge of the screen’s pixel density, the device’s bandwidth, and whatever other factors it deems relevant to the decision.

* Not possible to estimate... TBD

### Why not exposing a `metered` property?

TBD

### Why did we changed the `type` property values?

* Avoid potential fingerprinting issues.

TBD
