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

Go to the [Discussion](https://github.com/w3c-webmob/netinfo#discussion) section of the use case and requirements doc for a rationale about the changes introduced in this version.

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
