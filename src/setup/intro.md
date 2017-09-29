# Development Environment Setup

> NOTE: These steps are only necessary if you are not using the provided VM.
>
> These were all steps necessary for setting up the Xubuntu 16.04 VM.

```bash
# Add repository for tock compiler
sudo add-apt-repository ppa:team-gcc-arm-embedded/ppa
sudo apt-get update

# Install all dependencies
sudo apt-get install -y \
    ca-certificates \
    cmake \
    curl \
    g++-arm-linux-gnueabihf \
    gcc \
    gcc-arm-embedded \
    gedit \
    git \
    libc6-dev \
    libc6-dev-armhf-cross \
    make pkg-config \
    mosquitto-client \
    multistrap \
    python3 \
    python3-pip \
    wget

# Patch multistrap
sudo sed -i "s/\$forceyes//g" /usr/sbin/multistrap

# Provide python
sudo ln -s /usr/bin/python3 /usr/bin/python

# Populate multistrap configuration

cat << EOF > ./multistrap.conf
[General]
arch=armhf
noauth=true
unpack=true
debootstrap=Debian
aptsources=Debian

[Debian]
source=http://ftp.debian.org/debian
suite=stable
packages=libdbus-1-dev libssl-dev openssl

EOF

# Perform multistrap setup
sudo multistrap -a armhf -d /raspi_stretch -f ./multistrap.conf

# Link a few dependencies to match compilation expectations
sudo ln -s /raspi_stretch/lib/arm-linux-gnueabihf/libsystemd.so.0 /raspi_stretch/lib/arm-linux-gnueabihf/libsystemd.so
sudo ln -s /raspi_stretch/usr/include/arm-linux-gnueabihf/openssl/opensslconf.h /raspi_stretch/usr/include/openssl/opensslconf.h

# Allow the SYSROOT to be seen/owned by the main user
sudo chown -R rustfest /raspi_stretch/

# Setup file used when cross compiling to the Raspberry Pi

cat << EOF > ~/pi-crossenv
# This is where we have installed our fake raspberry pi debian system
export SYSROOT=/raspi_stretch
export RPI_USR_LIB_DIR=${SYSROOT}/usr/lib/arm-linux-gnueabihf
export RPI_LIB_DIR=${SYSROOT}/lib/arm-linux-gnueabihf

# Inform Cargo and Rustc that we are cross compiling, as well as some configuration
# specific to our current use cases
export CARGO_TARGET_ARMV7_UNKNOWN_LINUX_GNUEABIHF_LINKER=arm-linux-gnueabihf-gcc
export CC_armv7_unknown_linux_gnueabihf=arm-linux-gnueabihf-gcc
export CXX_armv7_unknown_linux_gnueabihf=arm-linux-gnueabihf-g++
export OPENSSL_INCLUDE_DIR=/raspi_stretch/usr/include
export OPENSSL_LIB_DIR=/raspi_stretch/usr/lib/arm-linux-gnueabihf
export OPENSSL_DIR=/raspi_stretch/usr/lib/ssl
export PKG_CONFIG_PATH=${RPI_USR_LIB_DIR}/pkgconfig
export PKG_CONFIG_LIBDIR=${RPI_USR_LIB_DIR}/pkgconfig:/usr/share/pkgconfig
export PKG_CONFIG_SYSROOT_DIR=${SYSROOT}
export PKG_CONFIG_ALLOW_CROSS=1
export RUSTFLAGS="-L ${RPI_USR_LIB_DIR} -L ${RPI_LIB_DIR} -lsystemd"

EOF

# Install rust and friends
curl https://sh.rustup.rs -sSf | sh
rustup component add rust-src
rustup install nightly-2017-09-20
rustup target add armv7-unknown-linux-gnueabihf
cargo install xargo
```
