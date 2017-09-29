# MQTT and other IoT Protocols

For lightweight IoT devices, it can be difficult to implement protocols like HTTP commonly used in web applications, due to Memory, CPU, or Code Size limitations. Additionally, common behavior for IoT devices such as asynchronously sending updates or subscribing to updates of incoming data, are not well suited to traditional HTTP or REST APIs.

This has led to the development of protocols such as [MQTT], [CoAP], and [LWM2M], which address the complexity concerns, as well as behavior mismatch concerns. These protocols are primarily designed for Machine to Machine (M2M) communication, as opposed to HTTP, which is typically used for presentation of human readable information.

[MQTT]: https://en.wikipedia.org/wiki/MQTT
[CoAP]: https://en.wikipedia.org/wiki/Constrained_Application_Protocol
[LWM2M]: https://en.wikipedia.org/wiki/OMA_LWM2M

As these protocols have become commonly used by lightweight IoT devices, they have also become commonly used by devices without constraints, for interoperability purposes.

## MQTT

MQTT is an application protocol that operates over TCP/IP, and allows devices to Subscribe and Publish to Topics. Devices act as MQTT Clients, and connect to a central MQTT Broker, which manages connections and dispatches messages to all connected MQTT Clients.

Similarly to HTTP, MQTT can be used directly over TCP, or may be secured using SSL/TLS. When the TCP connection used to connect to the MQTT Broker is secured with SSL/TLS, it is referred to as MQTTS.

### Topics

Topics are represented as UTF-8 strings, and are composed of one or more Topic Levels, separated by the `/` character. Some examples of topics could include:

* `home/james/kitchen/temperature/window`
* `home/james/kitchen/temperature/oven`
* `home/james/kitchen/brightness/window`
* `home/niklas/livingroom/temperature/couch`
* `home/niklas/kitchen/brightness/window`
* `office/geeny/kitchen/temperature/window`
* `office/geeny/kitchen/temperature/table`

### Publishing

When an MQTT client would like to send data, it Publishes the data to a specific topic. The format of the payload, or data, is implementation defined, and is often either binary packed data (for size concerns), or JSON (for ease of use).

MQTT Clients do not talk directly to eachother, instead they send data to the MQTT Broker on a specific topic, which is broadcasted to all Clients which have subscribed to that topic.

For example, James' Kitchen Temperature Sensor might send the following information:

Topic: `home/james/kitchen/temperature/window`

Data:

```JSON
{
    "temperature_celsius": 22.0
}
```

### Subscribing

When an MQTT client would like to receive data, it subscribes to a topic. When a message is sent from the Broker to the Client, it contains the Topic as well as the payload. Topics may be specific, such as `home/james/kitchen/temperature/window`, or may include wildcards.

#### Wildcards

There are two kinds of subscription wildcards, Single Level, and Multi level.

A single level wildcard is represented as a `+` character. The subscription request for `home/+/kitchen/brightness/window` would match:

- `home/james/kitchen/brightness/window`
- `home/niklas/kitchen/brightness/window`

A multi level wildcard is represented as a `#` character. The subscription request for `#/temperature/window` would match:

- `home/james/kitchen/temperature/window`
- `office/geeny/kitchen/temperature/window`

These wildcards can be repeated and combined in a single subscription request. The subscription request for `#/temperature/+` would match all of :

* `home/james/kitchen/temperature/window`
* `home/james/kitchen/temperature/oven`
* `home/niklas/livingroom/temperature/couch`
* `office/geeny/kitchen/temperature/window`
* `office/geeny/kitchen/temperature/table`

### Quality of Service

MQTT allows for varying levels of guarantees regarding delivery of messages. There are three levels that can be chosen:

* QoS Level 0: At Most Once
* QoS Level 1: At Least Once
* QoS Level 2: Exactly Once

As the QoS level rises, so does the amount of overhead. QoS Level 0 message require no acknowledgement from the Broker, QoS Level 1 requires a single acknowledgement from the Broker, and QoS Level 2 requires a four part handshake before a message has completed sending.

For periodic messages, such as a regular temperature reading, it may not be necessary to confirm reception (since additional messages will be sent), so QoS 0 would be a good match. For important messages that do not have side effects if applied multiple times, such as a "stop motor" command, QoS Level 1 would be a good match. For messages that must be received exactly once, such as "increase temperature 5 degrees", QoS 2 should be used.

## CoAP and other technologies

Other protocols have also been created to address many of the same topics addressed by MQTT. For example, CoAP was designed to operate similarly to how REST APIs work, with two major changes:

* UDP is used rather than TCP. This allows for reduced implementation requirements when compared to TCP.
* CoAP introduces a SUBSCRIBE verb, which allows for asynchronus updates
