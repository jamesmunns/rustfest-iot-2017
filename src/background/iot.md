# The Internet of Things

## What is the Internet of Things?

Generally, the Internet of Things covers devices that are now capable of communicating with other systems, either locally (like Bluetooth), or globally, via the Internet. People want to control items in the physical world similarly to how they are able to control and interconnect software

### Where did it come from?

IoT has risen from two main sources:

1. Industrial machinery that was "smart" before, but had little or no device to device connectivity
2.  Consumer devices, which are now cheap enough or resource efficient enough to include a networked component

As components used in devices like wireless routers and mobile phones have become more available, as well as cheaper, hardware developers have begun using them in other products, including IoT devices.

Around the same time, the maker movement, especially with the introduction of platforms like Arduino and the Raspberry Pi, have made embedded development much more accessible, both from a price perspective, as well as learning curve, and available resources.

We've now ended up with a new wave of embedded systems developers who are interested in the Internet of Things.

## Why is Rust a good match for IoT?

Over the last 20 years or so, we've gotten *okay* at connecting computers to the Internet, as well as keeping them decently secure. This has happened through:

* Use of well known, actively developed components
* Healthy respect for security updates and patches
* Multiple layers of security, including isolation
* Sane defaults
* Memory safe languages (usually garbage collected)
* Willingness to trade some performance for better security
* Improved security implementation in hardware and software

However on IoT devices, especially low end or battery powered devices, most developers are forced to minimize overhead. Many embedded systems and software were never originally designed to be Internet connected, let alone publicly available.

Rust allows us to move lots of the safety checks from runtime to compile time, allowing us to avoid some of the most common security holes such as buffer overflows, use after frees, etc. that are very common in protocol and parser implementations.

Additionally, Rust ships with Cargo, the default build tool and package manager for Rust development, which allows easier reuse of code.

Because Rust is based on LLVM, we have either full or partial support for some of the most common hardware used in IoT today, including:

- ARM Cortex-A
- ARM Cortex-M
- MIPS
- x86[-64]

## Architecture

<p align="center">
<img title="The Internet of Things, a basic diagram" src="assets/iot.svg">
</p>

Since IoT covers such a wide range of topics and devices, different approaches are needed to solve different problems

### Direct Nodes

Direct Nodes are devices that are capable of connecting directly to the Internet, typically over WiFi, GSM/Cell, or Ethernet. Since these communication methods are relatively high-power, direct nodes tend to either be wall-powered, or if battery powered, they connect infrequently to save battery life.

These could include devices running Linux, like a Raspberry Pi, or high end microcontroller systems, like the ESP32.

Speaking a full "Internet Protocol", including components like SSL, IPv6, and more requires computational and memory resources that many device can't handle

Examples:

- Amazon Alexa
- Smart TVs and other Appliances
- Nest

### Indirect Nodes

For devices where power or capabilities prevent direct Internet access, the next most common kind of IoT device connects to the Internet with assistance from a more powerful device.

This could mean the device is equipped to use a low power interface, such as Zigbee, Bluetooth, 6LoWPan, or Thread, which are designed to be power efficient, and were designed with low resource devices in mind.

For protocols like 6LoWPan or Thread, a simple "border router" which translates transparently from the low power protocol to full IP can be used. For protocols which are significantly different from IP, a Smart Hub/Gateway is typically used. Gateways typically require more "business logic" to determine how to correctly route messages.

Examples:

- Bluetooth Fitness Trackers
- Smart Light Bulbs
- Temperature Sensors

### Border Routers and Gateways

As mentioned above, Border Routers and Gateways are used to assist connectivity of low power devices. Both of these devices share the same typical design:

* Wall powered
* Ethernet, Cell, or WiFi connectivity
* Low-power radios/interfaces (LPWAN, PAN) for communication with Indirect Nodes

For border routers, little additional functionality may be necessary, other than converting from 6LoWPan to IPv6. These device function similarly to WiFi routers: transparently connecting the devices to the Internet.

For gateways, it is typically necessary to understand the devices that will be connected. This could be as simple as mapping local protocol constructs like Services and Characteristics to some kind of IP construct, such as URIs/endpoints.

Both of these devices may also be a "Direct Node" themselves, while also providing Border Router or Gateway functionality.

- "Smart Hub"/"Smart Gateway" devices
- Thermostat that communicates with Temperature Sensors
- Raspberry Pi running [Home Assistant](https://home-assistant.io/)

### The Cloud

* A central point for all devices to connect
* Enables additional functionality, such as maintaining history, connecting dissimilar devices to each other

Typically speaks device-friendly protocols such as MQTT, CoAP, LWM2M, etc., as well as protocols useful for browsers and apps, such as REST, Websockets, etc.
