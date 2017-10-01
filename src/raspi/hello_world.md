# Hello, World!

## A little setup

> Having trouble with the setup? Don't hesitate to ask for help!

Open up the VM, make yourself comfortable. Sorry for the `en-us` environment and keyboard.

You can choose to either interact with the graphic environment, or you can SSH into the virtual machine. If you want to SSH, don't forget to foreward the port from host to the virtual machine. SSH is served up on the VM's port 22.

* Username: `rustfest`
* Password: `rustfest`

Now would also be a good time to add your SSH public key to the VM's `~/.ssh/authorized_keys`, and add an entry into your SSH config `~/.ssh/config` that looks like this:

```text
host rustfest
    Hostname localhost
    User rustfest
    IdentityFile /home/$YOU/.ssh/id_rsa
    Port $MAPPED_PORT
```

This will save you some typing, but is completely optional.

## Hello, Host!

Lets make a new project

```bash
cargo new --bin hello-raspberry
cd hello-raspberry
```

Now lets compile and run it here

```bash
cargo build
cargo run
```

You should get something like this:

```text
rustfest@rustfest-2017:~/hello-raspberry$ cargo build
   Compiling hello-raspberry v0.1.0 (file:///home/rustfest/hello-raspberry)
    Finished dev [unoptimized + debuginfo] target(s) in 0.74 secs

rustfest@rustfest-2017:~/hello-raspberry$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello-raspberry`
Hello, world!
```

Lets look a bit closer at what we've done. We can use the `file` command to see what type of executable we've generated, and the `ldd` command to see what dynamic libraries we link to:

```bash
file target/debug/hello-raspberry
ldd target/debug/hello-raspberry
```

## Hello, Target!

Now lets build for the Raspberry Pi. Remember, the target triple we want here is `armv7-unknown-linux-gnueabihf`. This is normally done like this:

```bash
cargo build --target armv7-unknown-linux-gnueabihf
```

Give it a try!

Back yet?

Yeah, it didn't work. We haven't given Cargo enough information. Lets check out the file `~/pi-crossenv`. Print it out with this command:

```bash
cat ~/pi-crossenv
```

These are the important lines (for now):

```text
export CARGO_TARGET_ARMV7_UNKNOWN_LINUX_GNUEABIHF_LINKER=arm-linux-gnueabihf-gcc
export CC_armv7_unknown_linux_gnueabihf=arm-linux-gnueabihf-gcc
export CXX_armv7_unknown_linux_gnueabihf=arm-linux-gnueabihf-g++
```

These lines tell Cargo what C Compiler, C++ Compiler, and Linker to use when compiling for the `armv7-unknown-linux-gnueabihf` target.

Lets add these variables to our environment with the `source` command.

```bash
source ~/pi-crossenv
```

Now we can retry cross compiling.

```bash
cargo build --target armv7-unknown-linux-gnueabihf
```

This time you should see something like this:

```text
rustfest@rustfest-2017:~/hello-raspberry$ cargo build --target armv7-unknown-linux-gnueabihf
   Compiling hello-raspberry v0.1.0 (file:///home/rustfest/hello-raspberry)
    Finished dev [unoptimized + debuginfo] target(s) in 0.70 secs
```

Lets check out the generated binary with `file`:

```bash
file target/armv7-unknown-linux-gnueabihf/debug/hello-raspberry
```

How does that look compared to our other binary?

## Hello, Raspberry

Now, lets log on to your Raspberry Pi. There are only 5 of them, so you might have to share.

You should have gotten the IP address of your Raspberry Pi on a post-it note. Lets login, and create a directory for you to work in.

* Username: `pi`
* Password: `internetofthings`

```bash
ssh pi@<IP ADDRESS>

mkdir <YOUR NAME>
```

We are going to need the IP address a lot, so let's create an environment variable.

```bash
export IP=<IP ADDRESS>
```

This allows us to substitute the `$IP` everytime we need to provide the IP address.

Now you should be able to send the file to your Raspberry Pi. From the VM:

```bash
scp \
    target/armv7-unknown-linux-gnueabihf/debug/hello-raspberry \
    pi@$PI:/home/pi/<YOUR NAME>
```

Finally, we should be able to log in, and run our application!

```bash
ssh pi@$PI
cd <YOUR NAME>
./hello-raspberry
```

If it all worked, you should see something like this:

```text
pi@raspberrypi:~/james $ ./hello-raspberry
Hello, world!
```
