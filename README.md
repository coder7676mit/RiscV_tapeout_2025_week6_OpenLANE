# Report

## Table of Contents  

#### 1. Introduction  
 1.1 FOSSEE and the Role of Open-Source Tools  
 1.2 Open-Source Ecosystem for Current and Future Engineering Education
   
#### 2. Problem Statement  
 2.1 Objective  
 2.2 Scope of Work
   
#### 3. ESP-IDF and QEMU Fundamentals  
 3.1 Overview of ESP-IDF  
 3.2 Components and Capabilities of ESP-IDF  
 3.3 Overview of QEMU and ESP32 Emulation  
 3.4 Yaksh Platform Requirements and Prerequisite Checks
   
#### 4. ESP32 Development Environment Setup  
 4.1 Installing ESP-IDF  
 4.2 Installing ESP-IDF Tools  
 4.3 Building QEMU for ESP32 from Source
   
5. Verification of Installation  
 5.1 Confirming ESP-IDF Installation  
 5.2 Confirming QEMU Installation  
 5.3 Running Smoke Tests

6. Compilation Issues and Their Resolution  
 6.1 Environment-Related Errors  
 6.2 Missing Libraries and Header Files  
 6.3 Reconfiguration and Successful Build
   
7. Running LED and Temperature Sensor Programs  
 7.1 Building and Running the LED Example  
 7.2 Building and Running the Temperature Sensor Example  
 7.3 Output Verification
   
8. Integration with the Yaksh Automated Evaluation System  
 8.1 How QEMU Enables Automated Grading  
 8.2 Workflow for Evaluating ESP32 Assignments
   
9. Future Scope  
 9.1 Advanced Experiments and Projects  
 9.2 Research and Development Opportunities in Open-Source Toolchains  


# 1 Introduction



The work described in this document adopts that same philosophy. It focuses on constructing a fully open and inspectable environment to emulate ESP32 microcontrollers using QEMU and ESP-IDF. By building QEMU from source and integrating it with an ESP32-ready firmware toolchain, the environment supports development, debugging, automated code evaluation through platforms like Yaksh, and experimentation without requiring physical hardware.


## 2 Problem Statement

### 2.1 Objective

The objective is to set up a complete emulation environment for ESP32 microcontrollers using QEMU and the ESP-IDF toolchain. The environment must be fully functional, capable of compiling, flashing, and executing ESP32 programs entirely in software. It should support demonstration programs such as an LED blink example and a basic temperature sensor simulation executed inside QEMU.The scope includes installing ESP-IDF, fetching all required tools, building QEMU with Xtensa support, resolving compilation errors, and ensuring successful execution of emulated firmware. The environment must also integrate with the Yaksh evaluation platform so that ESP32 assignments can be evaluated automatically through the emulator without requiring physical boards. The final setup should be reliable enough to support education, programming labs, and experiment-driven learning.

## 3 ESP-IDF and QEMU Fundamentals

### 3.1 Overview of ESP-IDF

The ESP-IDF (Espressif IoT Development Framework) is the official software development kit for ESP32-series microcontrollers. It provides the full stack required to build embedded applications, including the FreeRTOS kernel, peripheral drivers, Wi-Fi and Bluetooth stacks, OTA support, flash memory handling, build scripts, Python-based tooling, and a complete project structure. ESP-IDF abstracts complex hardware features so that developers can focus on application logic while still having low-level access whenever needed. 
ESP-IDF includes a build system based on CMake and Python, with integrated configuration tools, device-specific compilers, linker scripts, and IDF-monitor utilities. The framework provides APIs for GPIO, ADC, timers, UART, SPI, I2C, PWM, tasks, queues, and other RTOS features. It also contains example projects for LEDs, sensors, networking, and peripheral testing. These components form the foundation for building and running ESP32 firmware both on real hardware and in an emulation environment such as QEMU.

### 3.2 Overview of QEMU and ESP32 Emulation

QEMU is a machine emulator capable of running full system images for various CPU architectures through dynamic binary translation. Espressif maintains a customized fork of QEMU that supports the Xtensa instruction set used by ESP32 chips. This modified version emulates flash memory, interrupts, peripherals, memory-mapped I/O, timers, and general CPU behavior. Building QEMU for ESP32 enables firmware execution without a physical development board, making it useful for continuous integration, automated testing, and code evaluation systems.

### 3.4 Yaksh Platform Requirements and Prerequisite Checks

Yaksh is an automated evaluation platform used in academic institutions for programming and system-level assignments. To support ESP32-based experiments, Yaksh requires a complete toolchain that can compile programs and run them in an emulator.

The required components for a functional environment include:
A functional Linux environment  
Python 3 with pip  
CMake, Ninja, GCC, Git, and other compilation tools  
Required development libraries such as libusb, libslirp, zlib, and ncurses  
A complete ESP-IDF installation and toolchain  
A working build of QEMU with Xtensa support  

All prerequisite checks must be completed before installation to prevent failures during the ESP-IDF or QEMU setup stages.

## 4. 

### System Dependency Installation

A complete ESP32 emulation workflow depends on several foundational system packages. These include compiler toolchains, Python runtimes, USB communication libraries, and build utilities required by ESP-IDF and QEMU. Ensuring that these dependencies are correctly installed is the first preparation step before configuring ESP-IDF or compiling QEMU.

### Initial Dependency Installation Attempt

The setup began with the following command:

```bash
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```
This command installs Python 3, pip, venv, CMake, Ninja, cryptographic libraries, USB libraries, and other tools essential for the ESP32 development environment. Some packages needed for QEMU compilation were still missing.

### 4.2 Updating Package Index and Installing Additional Dependencies

Before installing the remaining packages, the system package index must be updated to ensure all installations use the latest repository metadata. This avoids version conflicts, unavailable packages, or outdated dependency chains during ESP-IDF and QEMU setup.

The following command updates the package lists and installs another set of required utilities. These include core build tools, USB communication libraries, and other foundational components used during ESP-IDF compilation and QEMU execution:

```bash
sudo apt install -y git wget flex bison gperf python3 python3-pip cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0 libusb-1.0-0-dev make gcc
```
This command completes the foundational setup on a Linux system. After this installation, the machine is ready to proceed with configuring the ESP-IDF environment and compiling QEMU for ESP32 emulation.

### Creating Workspace and Cloning ESP-IDF

The ESP-IDF framework must be placed inside a dedicated workspace directory before installation. A new directory named `esp` was created in the home folder to contain all ESP32 development resources. After entering this directory, the official ESP-IDF repository was cloned using a recursive clone to ensure that all required submodules are included.

```bash
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
```

### Installing ESP-IDF Tools for ESP32

After cloning the ESP-IDF repository, the next step is to install all required toolchains, Python packages, and build utilities specific to the ESP32 target. ESP-IDF provides an automated installation script that configures the environment and downloads all dependencies.

The following command was executed from inside the cloned `esp-idf` directory:

```bash
cd ~/esp/esp-idf
./install.sh esp32
```
Once the ESP-IDF tool installation is complete, the environment must be initialized so that all ESP-IDF tools, Python packages, and Xtensa toolchains are available in the current terminal session.

This is done by sourcing the `export.sh` script inside the `esp-idf` directory.

```bash
cd ~/esp/esp-idf
source export.sh
```

### Cloning the Espressif QEMU Repository

With ESP-IDF configured, the next major step is to obtain the ESP32-supported QEMU emulator. Espressif maintains a custom fork of QEMU that includes Xtensa CPU support and ESP32 peripheral emulation. This fork is required because the upstream QEMU does not fully support ESP32 hardware components.

To download the repository, the following commands were executed from the home directory:

```bash
cd ~
git clone https://github.com/espressif/qemu.git
```

### Configuring QEMU for ESP32 (xtensa-softmmu Target)

After cloning the Espressif QEMU fork, the next step is to configure the build system. QEMU uses a configuration script that prepares all compilation settings based on the selected architecture and features.

For ESP32 emulation, QEMU must be built only for the `xtensa-softmmu` target, since this corresponds to the Xtensa LX6 CPU used in ESP32 chips.

The following commands were executed:

```bash
cd ~/qemu
./configure --target-list=xtensa-softmmu
```
### Building QEMU and Fixing libslirp Errors

Once the configuration step finished, the next stage was compiling QEMU using Ninja. The standard build command was executed:

```bash
cd ~/qemu/build
ninja -j$(nproc)
```

During the build process, the compilation failed with multiple errors related to missing symbols and headers from the libslirp library. The errors looked like:

```bash
fatal error: libslirp.h: No such file or directory
undefined reference to `slirp_new'
undefined reference to `slirp_state_load'
undefined reference to `slirp_state_save'
```

### Installing libslirp from Source (Alternative Fix)

In some setups, installing `libslirp-dev` from the package manager may still not provide the correct version required by the Espressif QEMU fork. To guarantee compatibility, the library can be built directly from source. This ensures that QEMU links against the correct headers and provides the full Slirp networking stack expected by ESP32 emulation.

The following prerequisite packages were installed to support building `libslirp`:

```bash
sudo apt install -y build-essential meson ninja-build pkg-config
```
The libslirp repository was then cloned:

```bash
cd ~
git clone https://gitlab.freedesktop.org/slirp/libslirp.git
```

### Then configure xtensa-softmmu and build qemu

```bash
cd ~/qemu
./configure --target-list=xtensa-softmmu

cd ~/qemu/build
ninja -j$(nproc)
```






