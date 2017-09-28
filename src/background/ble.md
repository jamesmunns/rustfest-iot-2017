# Bluetooth Low Energy

Bluetooth Low Energy (BLE) was integrated into the Bluetooth 4.0 specification in 2010. BLE reduced the typical power consumption for devices compared to Bluetooth Classic (Bluetooth 3.x and below), and provided a framework (GATT) that allowed implementation of common functionality, as well as implementation specific functionality.

Bluetooth has not typically been thought of as an IoT-specific technology, however the prevalence of hardware due to commercially successful products, as well as usage in Laptops and Phones, means that the barrier to entry for consumers is reduced. Additionally, Bluetooth 5 (introduced 2016) introduces additional capabilities, including extended range, higher data throughput, and better coexistence (allowing more simultaneous devices), making it a better fit for IoT applications.

## GATT

The Generic Attribute Profile, or GATT, defines a system for Central Clients (e.g. a Mobile Phone or Laptop) to discover and interact with Peripheral Servers (e.g. a Bluetooth Fitness Device).

Servers may provide a set of standard functionality to allow interoperability with many Central devices, as well as expose manufacturer specific functionality.

## Clients and Servers

With BLE, the Server broadcasts information about itself, allowing Clients to discover the Server, as well as what capabilities the Server offers. Clients can choose to connect to the Server, in order to obtain additional information, or to establish a secure communication channel.

## Services and Characteristics

Functionality of BLE are exposed via Characteristics, which are identified with a 128-bit UUID, such as `0000c01d-c001-de30-cabb-785feabcd123`. These Characteristics are grouped into Services, which provide no functionality, but are a collection of related Characteristics. Services are also identified with a UUID, such as `00000001-c001-de30-cabb-785feabcd123`.

An example of this grouping might look like this:

* Bluetooth Server Peripheral
    * Service: `00000001-c001-de30-cabb-785feabcd123`
        * Characteristic: `0000da7a-c001-de30-cabb-785feabcd123`
        * Characteristic: `0000c01d-c001-de30-cabb-785feabcd123`
        * Characteristic: `0000cafe-c001-de30-cabb-785feabcd123`
    * Service: `0f050001-3225-44b1-b97d-d3274acb29de`
        * Characteristic: `0f050002-3225-44b1-b97d-d3274acb29de`
    * Service: `00001801-0000-1000-8000-00805f9b34fb`
        * Characteristic: `00002a05-0000-1000-8000-00805f9b34fb`

When interacting with a BLE Server, all communication is made via Characteristics.

### Standard Services

For common functionality standardized by the Bluetooth Special Interest Group (SIG), Services and Characteristics can be assigned a 16-bit Shorthand ID, which can be used instead of the full 128-bit UUID. The Bluetooth SIG maintains a list of defined [Services] and [Characteristics].

[Services]: https://www.bluetooth.com/specifications/gatt/services
[Characteristics]: https://www.bluetooth.com/specifications/gatt/characteristics

In the example above, the Service `00001801-0000-1000-8000-00805f9b34fb` is defined as a "Generic Attribute" service, which may be shortened as `0x1801`. The Characteristic `00002a05-0000-1000-8000-00805f9b34fb` is defined as "Service Changed", and may be shortened as `0x2a05`.

## Actions on Characteristics

There are specific actions which each Characteristic may or may not support. There are six main interactions:

### Read

A **Read** is requested by the Client, and the Server retrieves the relevant data. Responses are limited to a single packet, typically about 20 bytes of data or less.

### Write

A **Write** sends data to the Client, and the Server accepts the data. Writes are also limited to a single packet, typically about 20 bytes of data or less.

### Long Read

**Long Read**s are a procedure of breaking a larger data transmission into multiple parts, with each part containing data and an offset.

### Long Write

**Long Write**s are a procedure of breaking a larger data transmission into multiple parts, with each part containing data and an offset.

### Notify

Rather than requiring a Client to poll a **Read** characteristic to obtain updated data, Bluetooth allows a Client to request a Server to **Notify** it whenever the data changes. The Bluetooth Server may then send data to all subscribed Clients asynchronously. **Notify** messages are also limited to the 20-byte data limit.

**Notify** is particularly useful for low power devices, as it does not require the Server to be continually listening for requests from the Client.

### Indicate (subscribe w/ ACK)

**Indicate** is similar to **Notify**, however **Indicate** requires the Client to acknowledge the reception of the message.
