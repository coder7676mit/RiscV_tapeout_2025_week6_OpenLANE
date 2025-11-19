Report

This document provides a fully detailed explanation of how ESP-IDF and the Espressif fork of QEMU were installed and built from source on an Ubuntu 24.04 system running inside VirtualBox. The explanations cover every step, starting from a basic conceptual level, and include the exact reasons behind each error encountered. The goal is to create a complete, transparent reference that shows how QEMU ESP32 emulation was brought up correctly despite dependency issues and mismatches that are common in newer Linux distributions.

---

1\. Understanding the Goal from Ground Level

An ESP32 is a microcontroller with a custom architecture called Xtensa. It is not compatible with the native x86 processor used by PCs. When building firmware for ESP32 using ESP-IDF, testing that firmware normally requires real hardware. However, for debugging, automation and development purposes, it is extremely useful to emulate the ESP32 on a computer. QEMU is an open-source emulator capable of simulating many CPU architectures. Espressif maintains a modified version of QEMU that adds support for Xtensa-based microcontrollers like ESP32. The official QEMU upstream does not include this Xtensa support.

Therefore, we needed to build Espressif's version of QEMU from source so that compiled ESP32 firmware can run in a virtual environment. This required setting up the entire ESP-IDF toolchain, followed by cloning, configuring and compiling the custom QEMU fork.

Ubuntu 24.04 is a new release, which means several system libraries have been updated to newer versions that are sometimes incompatible with older QEMU forks. Additionally, installing Linux inside a VirtualBox environment introduces extra constraints: graphics libraries, virtualization drivers, networking stack behavior and package versions can all differ from bare-metal Ubuntu.

These environmental differences are the root cause of the build failures we encountered.

---

2\. Installing ESP-IDF on Ubuntu 24.04

Before interacting with QEMU, ESP-IDF must be installed. This includes the Xtensa compilers, Python toolchain, build system and device utilities. Ubuntu 24.04 already ships with Python 3.12, which is compatible with ESP-IDF v6.x.

The following commands were executed:

sudo apt update\
sudo apt install -y git wget flex bison gperf gcc cmake ninja-build ccache dfu-util libusb-1.0-0 python3 python3-pip python3-venv\
mkdir -p ~/esp\
cd ~/esp\
git clone --recursive https://github.com/espressif/esp-idf.git\
cd esp-idf\
./install.sh esp32\
. ./export.sh\
idf.py --version

Everything worked correctly because ESP-IDF supports modern Ubuntu versions well.

---

3\. Cloning the Espressif QEMU Fork

Next, the QEMU fork was cloned:

cd ~\
git clone https://github.com/espressif/qemu.git\
cd qemu\
git submodule update --init --recursive

QEMU requires Meson and Ninja build systems, which already exist on Ubuntu 24.04.

---

4\. First Configuration Attempt and Why It Failed

Inside ~/qemu/build directory:

../configure --target-list=xtensa-softmmu

The configuration succeeded, but the build failed immediately during compilation. The specific error was:

fatal error: libslirp.h: No such file or directory\
fatal error: libslirp-version.h: No such file or directory\
undefined reference to slirp_new\
undefined reference to slirp_state_save\
undefined reference to slirp_add_exec

To understand why this happened, it is important to understand what libslirp is.

Slirp is a user-mode networking library used by QEMU to emulate virtual network interfaces. The Espressif fork of QEMU depends on libslirp, but expects the older 4.4.0 ABI. Ubuntu 24.04 ships a much newer version of libslirp (4.7.0). This newer version changes header file locations, removes some API functions, and reorganizes headers like libslirp-version.h.

QEMU ESP32, being older and not updated to match Ubuntu 24.04's package versions, expected the older API. Ubuntu provided the newer API, which is incompatible. This mismatch is exactly why the compilation failed.

Because this system was inside VirtualBox, networking virtualization modules were also minimal, and slirp integration often behaves differently inside virtualized environments. But the primary issue came from ABI mismatch, not VirtualBox itself.

---

5\. Why Ubuntu 24.04 Caused libslirp Errors

Ubuntu 24.04 (Noble Numbat) includes modernized, security-hardened versions of many libraries. libslirp 4.7 removes several function names that older QEMU versions expect. Header files like libslirp-version.h no longer exist. The compiled library symbols differ from what older QEMU searches for during linking.

Espressif QEMU is based on older QEMU code which predates these changes. Therefore, when QEMU tried to use slirp functions such as:

slirp_new\
slirp_state_save\
slirp_remove_hostxfwd\
slirp_pollfds_poll

Ubuntu's libslirp 4.7 did not export those symbols anymore, resulting in linker errors like:

undefined reference to `slirp_new'

This is a direct result of using a newer Linux distribution than the QEMU fork was tested on.

---

6\. Fixing the libslirp Mismatch

The fix required removing Ubuntu's version:

sudo apt remove -y libslirp-dev

Then building the correct version directly from source:

sudo apt install -y build-essential meson ninja-build\
cd ~\
git clone https://gitlab.freedesktop.org/slirp/libslirp.git\
cd libslirp\
git checkout v4.4.0\
meson setup build\
ninja -C build\
sudo ninja -C build install\
sudo ldconfig

Installing from source puts headers into /usr/local/include/slirp, which overrides the newer version.

Now the header libslirp-version.h exists and symbols like slirp_add_exec are available.

---

7\. Final Configuration and Successful Build

Returning to QEMU:

cd ~/qemu/build\
../configure --target-list=xtensa-softmmu\
ninja -j$(nproc)

This time the compilation completed successfully, confirming that the correct libslirp version fixed all issues.

---

8\. Why VirtualBox Environment Matters

Running Ubuntu inside a VirtualBox virtual machine introduces a few important conditions:

1\. VirtualBox guest additions modify the system networking stack.

2\. VirtualBox uses NAT internally, which interacts with slirp-based implementations.

3\. Some Ubuntu 24.04 packages are optimized differently when running inside virtualization.

4\. Graphics libraries and audio backends are minimal, so QEMU may disable certain GUI subsystems.

However, the core error was not caused by VirtualBox itself. Instead, the underlying cause was that a modern distribution (Ubuntu 24.04) ships newer libraries incompatible with older QEMU branches. VirtualBox simply added another layer where missing features could have appeared if not handled correctly.

---

9\. Complete Command List (Fully Ordered, Copy-Paste Ready)

sudo apt update\
sudo apt install -y git wget flex bison gperf gcc cmake ninja-build ccache dfu-util libusb-1.0-0 python3 python3-pip python3-venv

mkdir -p ~/esp\
cd ~/esp\
git clone --recursive https://github.com/espressif/esp-idf.git\
cd esp-idf\
./install.sh esp32\
. ./export.sh\
idf.py --version

cd ~\
git clone https://github.com/espressif/qemu.git\
cd qemu\
git submodule update --init --recursive

mkdir build\
cd build\
../configure --target-list=xtensa-softmmu

sudo apt install -y libslirp-dev

sudo apt remove -y libslirp-dev\
sudo apt install -y build-essential meson ninja-build

cd ~\
git clone https://gitlab.freedesktop.org/slirp/libslirp.git\
cd libslirp\
git checkout v4.4.0\
meson setup build\
ninja -C build\
sudo ninja -C build install\
sudo ldconfig

cd ~/qemu/build\
../configure --target-list=xtensa-softmmu\
ninja -j$(nproc)\
ls /usr/local/include | grep slirp
