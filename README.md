#Network Information API v3

## Introduction

### Use cases and requirements
A detailed document containing set of use cases and requirements for this API can be found [here](https://github.com/w3c-webmob/netinfo).

### Previous versions

* [Version 1](http://www.w3.org/TR/2011/WD-netinfo-api-20110607/)
* [Version 2](http://www.w3.org/TR/netinfo-api/)

## Why do we need a new version of this API?

TBW...

## API description

### The NetworkInformation interface
The `NetworkInformation` interface is exposed on the `Navigator` object. 
   
    Navigator implements NetworkInformation;

All instances of the `Navigator` type are defined to also implement the `NetworkInformation` interface.

    [NoInterfaceObject]
    interface NetworkInformation {
          readonly attribute Connection connection;
    };

#### Attributes

* **connection** of type [Connection](#the-connection-interface), readonly. The object from which connection information is accessed.

### The Connection interface
The `Connection` interface provides a handle to the device's connection information.

    [NoInterfaceObject]
    interface Connection : EventTarget {
        readonly attribute DOMString type;
        [TreatNonCallableAsNull]
                 attribute EventHandler? onchange;
    };

#### Attributes

* **type** of type DOMString, readonly. Exposes the current connection type. The value returned is one of the following strings, case-sensitively: `unknown`, `ethernet`, `wifi`, `cellular`, `none`.

* **onchange** of type EventHandler, nullable.

When the `Connection` changes, the user agent must queue a task which updates the `Connection` properties and fire a simple event named `change` at the `Connection` object.

When the user goes online or offline, in addition to the `change` event fired on the `Connection` object, the user agent has to fire a simple event named either `online` or `offline` depending on the applicable value, as defined in [HTML5](http://www.w3.org/TR/html5/).

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
