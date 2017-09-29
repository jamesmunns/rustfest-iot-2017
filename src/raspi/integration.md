# Putting it together

As we're getting to the end of the guided period, there are a few tasks to try out without much guidance.

Give them a try, and feel free to ask for help if you need it!

## Task One: Transparent Protocol Converter

For our Tock board, we would like to send all information received from the Environmental Sense Characteristic over MQTT. Additionally, we would like to take specific messages received over MQTT, and forward them to the LED control characteristic.

## Task Two: More than one Tock

Repeat the steps of Task One, however support 2 or more Tock boards at the same time. When sending messages over MQTT, make sure that it is possible to determine where the messages came from. When receiving messages from MQTT, make sure that only the correct Tock device receives the message you sent.
