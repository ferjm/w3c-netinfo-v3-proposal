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

* **type** of type DOMString, readonly. Exposes the current connection type. The value returned is one of the following strings, case-sensitively: `unknown`, `ethernet`, `wifi`, `2g`, `3g`, `4g`, `none`.

* **onchange** of type EventHandler, nullable.

When the `Connection` changes, the user agent must queue a task which updates the `Connection` properties and fire a simple event named `change` at the `Connection` object.

When the user goes online or offline, in addition to the `change` event fired on the `Connection` object, the user agent has to fire a simple event named either `online` or `offline` depending on the applicable value, as defined in [HTML5](http://www.w3.org/TR/html5/).

## Examples

TBW...
