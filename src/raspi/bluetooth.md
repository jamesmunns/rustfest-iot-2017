# Talking with Bluetooth

Before we get started, you'll need a Tock board. On the bottom, there should be a sticker with a MAC address that looks like `AA:BB:CC:DD:EE:FF`. We'll need this later.

If you're using the default Tock firmware, you'll also need to know the Service and Characteristics you want to interact with. These are probably:

* Service: `00001801-0000-1000-8000-00805f9b34fb`
    * Characteristic: `00002a05-0000-1000-8000-00805f9b34fb`

To start off, we'll be using a crate called `easy-bluez`, which exposes the ability to read and write to Bluetooth Low Energy devices. This crate automatically handles the scanning, connection, and polling of the characteristics of your choosing. Under the hood, `easy-bluez` uses a crate called `blurz` to interact with Linux's `BlueZ` service over `dbus`.

## A quick reminder

There were some other lines in `~/pi-crossenv` that we skipped over, particularly:

```text
export SYSROOT=/raspi_stretch
export RPI_USR_LIB_DIR=${SYSROOT}/usr/lib/arm-linux-gnueabihf
export RPI_LIB_DIR=${SYSROOT}/lib/arm-linux-gnueabihf

export PKG_CONFIG_PATH=${RPI_USR_LIB_DIR}/pkgconfig
export PKG_CONFIG_LIBDIR=${RPI_USR_LIB_DIR}/pkgconfig:/usr/share/pkgconfig
export PKG_CONFIG_SYSROOT_DIR=${SYSROOT}
export PKG_CONFIG_ALLOW_CROSS=1
export RUSTFLAGS="-L ${RPI_USR_LIB_DIR} -L ${RPI_LIB_DIR} -lsystemd"
```

Since `easy-bluez` uses DBus under the hood to talk to BlueZ, it is necessary to dynamically link to `libdbus-1`, and its dependencies. Since we want to make sure that our cross compilation environment matches our target, we've set up a `SYSROOT` at `/raspi_stretch`. This is a simulated version of the root filesystem found on our Raspberry Pis. This was populated using a tool called `multistrap`, which installs debian packages from other architectures to a specified location.

These environment variables instruct Cargo where to look for dynamically linked libraries, which are checked at compile time.

## Set up `easy-bluez`

Lets set up a new project inside the VM

```bash
cd ~
cargo new --bin hello-bluetooth
cd hello-bluetooth
```

Now, lets add `easy-bluez` as a dependency, and make sure everything still builds. Add these lines to your `Cargo.toml`:

```text
[dependencies]
easy-bluez = "0.1"
```

In your `src/main.rs`, add these lines:

```rust
extern crate easy_bluez;
use easy_bluez::EasyBluez;

fn main() {
    println!("Hello, world!");
}
```

Now we're ready to compile! Don't forget to `source ~/pi-crossenv` if you've closed your window. After a bit of compiling, you should see something like this:

```text
...
   Compiling hello-bluetooth v0.1.0 (file:///home/rustfest/hello-bluetooth)
warning: unused import: `easy_bluez::EasyBluez`
 --> src/main.rs:2:5
  |
2 | use easy_bluez::EasyBluez;
  |     ^^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(unused_imports)] on by default
    Finished dev [unoptimized + debuginfo] target(s) in 59.69 secs
rustfest@rustfest-2017:~/hello-bluetooth$
```

If you don't see something like this, ask for help!

## Read from your Hail board

Okay, lets actually read from your device. `easy-bluez` has a method called `poll`, which you can use like this:

```rust
extern crate easy_bluez;
use easy_bluez::EasyBluez;

fn main() {
    let ez = EasyBluez::new().run();

    let rx_poll = ez.poll(
        "CF:75:CE:86:6D:02",                    // MAC Address
        "00000001-c001-de30-cabb-785feabcd123", // Service
        "0000c01d-c001-de30-cabb-785feabcd123", // Characteristic
    ).expect("Bad data!");

    // Blocking wait for a message to be received
    if let Ok(msg) = rx_poll.recv() {
        println!("{:?}", msg);
    }
}
```

`poll` returns a `Result<Receiver<Box<[u8]>>>`, which will provide you with a stream of data obtained from the device, exposed as heap-allocated arrays of bytes.

Try building this code, and running it on your Raspberry Pi. As a note, `sudo` is required to interact with BlueZ, so you will need to run your binary with `sudo ./hello-bluetooth`. Don't forget to use your MAC Address, Service, and Characteristic!

Once you are able to read a single message, check out the documentation for [Receiver<T>](https://doc.rust-lang.org/std/sync/mpsc/struct.Receiver.html), and try processing messages as they arrive.

## Write to your Hail board

`easy-bluez` has another method called `writeable()`, which can be used like this:

```rust
extern crate easy_bluez;
use easy_bluez::EasyBluez;
use std::time::Duration;
use std::thread::sleep;

fn main() {
    let ez = EasyBluez::new().run();

    let tx_write = ez.writeable(
        "CF:75:CE:86:6D:02",                    // MAC Address
        "00000001-c001-de30-cabb-785feabcd123", // Service
        "0000da7a-c001-de30-cabb-785feabcd123", // Characteristic
    ).expect("Bad data!");

    // Send a single message to the device
    tx_write.send(Box::new([0x00])).unwrap();

    // Sleep for a bit to prevent returning before the message is sent
    sleep(Duration::from_secs(1));
}
```

Try building this code, and running on your Raspberry Pi.

## Putting it together

Once you have reading and writing working, try combining them!

For example, you could:

* Write a message every 4th receive
* Send messages at a fixed interval, while processing all incoming messages
