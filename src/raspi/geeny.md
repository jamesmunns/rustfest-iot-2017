# Talking to the Cloud

It's time to start talking with the Internet! For this exercise, we will be using a publicly available MQTT Broker provided by Hive MQ to demonstrate some basic functionality. We'll be using a library called [rumqtt] to communicate as an MQTT Client.

[rumqtt]: https://docs.rs/rumqtt/0.10.0/rumqtt/

Lets make a new project, and add `rumqtt` to it. In your Virtual Machine:

```bash
cargo new --bin hello-mqtt
```

In your Cargo.toml:

```
[dependencies]
rumqtt = "0.10"
```

In your `src/main.rs`

```
extern crate rumqtt;
```

## Interact over MQTT

For easy debugging, we'll use HiveMQ's [Live Dashboard]. You can open a browser window here, and press "Connect". You will be able to add subscription topics, as well as send messages to topics.

[Live Dashboard]: http://www.hivemq.com/demos/websocket-client/

You will be able to connect to the same server using `rumqtt`, and see all messages sent back and forth

> WARNING: This is a publically used server, and is typically very busy with people sending all kinds of junk (and non-junk) data. Be careful when subscribing to wildcard topics with either `+` or `#`. I suggest using a path something like:
>
> `rustfest/YOUR_NAME/foo/bar`

## Sending Messages

We'll start off with sending messages. Here is an example of connecting to the broker mentioned above, and sending a single message. You should see the message appear in the [Live Dashboard] window.

```rust
extern crate rumqtt;
use std::time::Duration;
use std::thread::sleep;
fn main() {

    let opts = rumqtt::MqttOptions::new()
        .set_broker("broker.mqttdashboard.com:1883")
        .set_client_id("demo-mqtt")
        .set_keep_alive(5)
        .set_reconnect(10);

    let mut client = rumqtt::MqttClient::start(opts, None).unwrap();

    client
        .publish(
            "rustfest/james/demo",
            rumqtt::QoS::Level0,
            String::from("Hello, MQTT!").into_bytes(),
        ).unwrap();

    // Give time for the message to be sent
    sleep(Duration::from_secs(3));
}
```

## Receiving Messages

Here is an example which allows you to listen incoming messages for 30 seconds. After starting this client, you should be able to send messages from the [Live Dashboard], and have them appear on your client.

```rust
extern crate rumqtt;

use std::time::Duration;
use std::thread::sleep;

fn main() {
    // Configure the client
    let opts = rumqtt::MqttOptions::new()
        .set_broker("broker.mqttdashboard.com:1883")
        .set_client_id("james-demo-mqtt")
        .set_keep_alive(5)
        .set_reconnect(10);

    // Provide a callback for all incoming messages
    let msg_handler = rumqtt::MqttCallback::new().on_message(|message| {
        println!("- Topic: {}\n  Message: {}",
                message.topic.to_string(),
                String::from_utf8_lossy(message.payload.as_ref())
        );
    });

    // Start the client
    let mut client = rumqtt::MqttClient::start(opts, Some(msg_handler)).unwrap();

    // Subscribe to topics
    client.subscribe(vec![("rustfest/#", rumqtt::QoS::Level0)]).unwrap();

    // Give time for the message(s) to arrive
    sleep(Duration::from_secs(30));
}
```

## A quick note about security

In this demo we are using MQTT, rather than MQTTS. This is mainly due to the fact that the demonstration broker we are using does not support MQTTS.

It is always recommended to use SSL when possible, otherwise, all data and login credentials are sent and received in plain text. When using SSL, you can make the following changes to the client configuration:

```rust
let opts = rumqtt::MqttOptions::new()

    // Port 8883 is typically MQTTS
    .set_broker("broker.mqttdashboard.com:8883")

    // What certificates to validate the server with
    .set_ca("/etc/ssl/certs/ca-certificates.crt")

    // Force verification of the certificates
    .set_should_verify_ca(true)

    // Same as above
    .set_client_id("demo-mqtt")
    .set_keep_alive(5)
    .set_reconnect(10);
```

## Put it together

Try creating a client that responds to all incoming messages on the same topic.

If you have time, try sending and receiving data formatted as JSON using [Serde JSON](https://github.com/serde-rs/json)
