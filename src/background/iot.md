# The Internet of Things

## What is the Internet of Things?

* Generally, it covers the ability for devices that are now capable of communicating with other systems, either locally (like bluetooth), or globally, via the internet
* People want to control items in the physical world similarly to how they are able to control and interconnect software
* It has generally risen from two sources:
    * Industrial machinery that was "smart" before, but had little or no device to device connectivity
    * Consumer devices, which are now cheap enough to include a networked component
* Things have gotten cheaper due to the commercialization of other items, such as wireless routers and mobile phones, which have pushed down the price for components, and the availability of affordable components
* Around the same time, the maker movement, especially with the introduction of platforms like Arduino and the Raspberry Pi, have made embedded development much more accessable, both from a price perspective, as well as learning curve, and available resources

* We've now ended up with a new wave of embedded systems developers who are interested in the Internet of Things.

<p align="center">
<img title="The Internet of Things, a basic diagram" src="assets/iot.svg">
</p>

## Why is Rust a good match for IoT?

* Over the last 20 years or so, we've gotten okay at connecting computers to the internet, as well as keeping them decently secure
    * Multiple layers of security
    * Good defaults
    * Memory safe languages (ususally garbage collected)
    * willing to trade some performance for better security
    * improved security implementation in hardware and software
* However on IoT devices, especially low end or battery powered devices, most developers are forced to minimize overhead
* Many embedded systems and software were never designed to be internet connected, let alone publically available

Rust allows us to move lots of the safety checks from runtime to compile time, allowing us to avoid some of the most common security holes such as buffer overflows, use after frees, etc that are very common in protocol and parser implementations.

Because Rust is based on LLVM, we have either full or partial support for some of the most common hardware used in IoT today, including:

- ARM Cortex-A
- ARM Cortex-M
- MIPS
- x86[-64]

## Architecture

Since IoT covers such a wide range of topics and devices, different approaches are needed to solve different problems

### Direct Nodes

Direct Nodes are devices that are capable of connecting directly to the internet, typically over WiFi, GSM/Cell, or Ethernet. Since these communication methods are relatively high-power, direct nodes tend to either be wall-powered, or if battery powered, they connect infrequently to save battery life.

These could include devices running linux, like a Raspberry Pi, or high end microcontroller systems, like the ESP32.

Speaking a full "internet protocol", including components like SSL, IPv6, and more requires computational and memory resources that many device can't handle

### Indirect Nodes

For devices where power or capabilities prevent direct internet access, the next most common kind of IoT device connects to the internet with assistance from a more powerful device.

This could mean the device is equipped to use a low power interface, such as Zigbee, Bluetooth, 6lowpan, or Thread, which are designed to be power efficient, and were designed with low resource devices in mind.

For protocols like 6lowpan or Thread, a simple "border router" which translates transparently from the low power protocol to full IP can be used. For protocols which are significantly different from IP, a Smart Hub/Gateway is typically used. Gateways typically require more "business logic" to determine how to correctly route messages.

### Border Routers and Gateways

As mentioned above, Border Routers and Gateways are used to assist connectivity of low power devices. Both of these devices share the same typical design:

* Wall powered
* Ethernet, Cell, or WiFi connectivity
* low-power radios/interfaces for communication with Indirect Nodes

For border routers, little additional functionality may be necessary, other than converting from 6lowpan to IPv6. These device function similarly to WiFi routers: transparently connecting the devices to the internet.

For gateways, it is typically necessary to understand the devices that will be connected. This could be as simple as mapping local protocol constructs like Services and Characteristics to some kind of IP construct, such as URIs/endpoints

### The Cloud

* A central point for all devices to connect
* Enables additional functionality, such as maintaining history, connecting dissimilar devices to eachother

Typically speaks device-friendly protocols such as MQTT, CoAP, LWM2M, etc., as well as protocols useful for browsers and apps, such as REST, Websockets, etc.
