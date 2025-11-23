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
---

# 5. Verification: Confirming All Installations Are Working Correctly

Before proceeding with projects, it is important to verify that ESP-IDF, QEMU, toolchains, Python environment, and supporting libraries are correctly installed.  
Running the following commands ensures the entire development and emulation environment is functional.

---

## Verify ESP-IDF Environment and toolchain
This confirms that the ESP-IDF environment variables are active, the Python environment is detected, and all required tools are located.

```bash
cd ~/esp/esp-idf
source export.sh
idf.py --version

xtensa-esp32-elf-gcc --version
```

## Verifying QEMU environment
``bash
qemu-system-xtensa --version
```

---


# 7.

## Creating a demo project

### Initializing the ESP-IDF Environment Before Creating Projects

Before creating or building any ESP-IDF project, the ESP-IDF environment must be activated. ESP-IDF relies on a Python virtual environment, toolchain binaries, CMake helpers, and internal scripts that are loaded through the `export.sh` file. Without sourcing this script, commands like `idf.py`, the Xtensa toolchain, and ESP-IDF’s QEMU integration will not function correctly.

To initialize the environment, the following command was executed from inside the ESP-IDF installation directory:

```bash
cd ~/esp/esp-idf
source export.sh
```

### Creating the First ESP-IDF Project Inside the Workspace

After initializing the ESP-IDF environment, a dedicated directory was prepared to keep all firmware projects organized. A `projects` folder was created inside the main ESP workspace, and the Hello World project was generated there. These steps ensure a clean structure and prevent mixing project files with ESP-IDF’s internal repository.

The following commands were executed:

```bash
cd ~/esp/projects
mkdir hello
idf.py create-project hello_world
```
Once the project directory was created, the next step was to configure it for the ESP32 architecture. This is required before any build action, as ESP-IDF supports multiple chips and must know which target to compile for. The configuration was applied using:
```bash
cd ~/esp/projects/hello_world
idf.py set-target esp32
```
#### Edit the hello_world.c file which was created

```bash

```

Then build the project using

``bash
idf.py build

//Once this is done succesfully, run the project and view the output on IDF monitor

idf.py qemu monitor
``
----

## ESP-IDF LED Blink Test on QEMU

This section provides the complete sequence of commands and source code required to create, configure, build, and run the ESP-IDF LED Blink project entirely inside the ESP32 QEMU emulator. Terminal commands and source code are separated into proper code blocks to ensure safe usage and clean documentation.

### Step 1: Initialize the ESP-IDF Environment

Before creating any project, the ESP-IDF environment must be activated so that `idf.py`, the Xtensa toolchain, CMake, and Python dependencies are available.

```bash
cd ~/esp/esp-idf
source export.sh
```

### Step 2: Create Project Folder and Generate Blink Template

```bash
cd ~/esp/projects
idf.py create-project blink
cd blink
```
### Step 3: Code the project

ESDP-IDF has standard example codes as .c file for hello world and blink projects. So you can use that one or even code it separately.

Open the .c file and code

```bash
/* Blink Example

   This example code is in the Public Domain (or CC0 licensed, at your option.)

   Unless required by applicable law or agreed to in writing, this
   software is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
   CONDITIONS OF ANY KIND, either express or implied.
*/
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"
#include "led_strip.h"
#include "sdkconfig.h"

static const char *TAG = "example";
#define BLINK_GPIO CONFIG_BLINK_GPIO

static uint8_t s_led_state = 0;

#ifdef CONFIG_BLINK_LED_STRIP
static led_strip_handle_t led_strip;

static void blink_led(void)
{
    if (s_led_state) {
        led_strip_set_pixel(led_strip, 0, 16, 16, 16);
        led_strip_refresh(led_strip);
    } else {
        led_strip_clear(led_strip);
    }
}

static void configure_led(void)
{
    ESP_LOGI(TAG, "Example configured to blink addressable LED!");
    led_strip_config_t strip_config = {
        .strip_gpio_num = BLINK_GPIO,
        .max_leds = 1,
    };
#if CONFIG_BLINK_LED_STRIP_BACKEND_RMT
    led_strip_rmt_config_t rmt_config = {
        .resolution_hz = 10 * 1000 * 1000,
        .flags.with_dma = false,
    };
    ESP_ERROR_CHECK(led_strip_new_rmt_device(&strip_config, &rmt_config, &led_strip));
#elif CONFIG_BLINK_LED_STRIP_BACKEND_SPI
    led_strip_spi_config_t spi_config = {
        .spi_bus = SPI2_HOST,
        .flags.with_dma = true,
    };
    ESP_ERROR_CHECK(led_strip_new_spi_device(&strip_config, &spi_config, &led_strip));
#else
#error "unsupported LED strip backend"
#endif
    led_strip_clear(led_strip);
}

#elif CONFIG_BLINK_LED_GPIO
static void blink_led(void)
{
    gpio_set_level(BLINK_GPIO, s_led_state);
}

static void configure_led(void)
{
    ESP_LOGI(TAG, "Example configured to blink GPIO LED!");
    gpio_reset_pin(BLINK_GPIO);
    gpio_set_direction(BLINK_GPIO, GPIO_MODE_OUTPUT);
}
#else
#error "unsupported LED type"
#endif

void app_main(void)
{
    configure_led();

    while (1) {
        ESP_LOGI(TAG, "Turning the LED %s!", s_led_state == true ? "ON" : "OFF");
        blink_led();
        s_led_state = !s_led_state;
        vTaskDelay(CONFIG_BLINK_PERIOD / portTICK_PERIOD_MS);
    }
}
```
### Step 4: Configure the Project and set the target

Run the configuration menu:

```bash
idf.py menuconfig

Inside menuconfig:

Select LED GPIO pin
Choose blink period
Select LED type (GPIO or LED strip)
Save and exit.

//Then set the target explicitly:

idf.py set-target esp32
```
### Step 5: Build the project and run on QEMU

``bash
idf.py build

//Once this is done succesfully, run the project and view the output on IDF monitor

idf.py qemu monitor
``

----

## ESP32 QEMU Temperature Sensor Example – Complete Steps and Code


### Step 1: Activate ESP-IDF Environment

Before creating the temperature project, the ESP-IDF environment must be loaded:

```bash
cd ~/esp/esp-idf
source export.sh
```

### Step 2: Create Project Directory and Generate Template

A new project directory was created under esp/projects:

``
cd ~/esp/projects
idf.py create-project temp_sensor
cd temp_sensor

``

### Step 3: Add the Temperature Sensor Application Code

ESP-IDF does not generate a temperature example automatically, so a custom file is created.

Create or replace:

main/temp_sensor_main.c

with the following full code:

```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sdkconfig.h"

// Define a tag for logging output
static const char *TAG = "TEMP_SENSOR_QEMU";

// --- Mock Sensor Driver Function ---
// In a real project, this function would handle I/O to a physical sensor.
// In QEMU, we return a simulated, changing value.
float mock_sensor_read_temp_celsius() {
    static float current_temp = 20.0; // Start at 20.0 C
    static int direction = 1;         // 1 for increasing, -1 for decreasing

    // Simulate temperature change
    current_temp += (0.1 * direction);

    // Change direction if we hit a boundary
    if (current_temp >= 30.0) {
        direction = -1;
    } else if (current_temp <= 20.0) {
        direction = 1;
    }

    return current_temp;
}

// --- Main Application Task ---
void temp_task(void *pvParameter) {
    ESP_LOGI(TAG, "Temperature Reader Task Started.");

    while (1) {
        // 1. Read the simulated temperature
        float temp_c = mock_sensor_read_temp_celsius();

        // 2. Print the result using the logging system (visible in QEMU serial output)
        ESP_LOGI(TAG, "Current Temperature: %.2f C", temp_c);

        // 3. Wait for 2 seconds before the next reading (FreeRTOS delay)
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}

// --- ESP-IDF Application Entry Point ---
void app_main(void) {
    // Create the task that will handle the temperature reading
    xTaskCreate(temp_task, "temp_reader", 2048, NULL, 5, NULL);
}

```
---

### Step 4: Configure Target

Before building, select ESP32 as the target:
```
idf.py set-target esp32
```

### Step 5: Build and Run the Temperature Sensor Firmware

Compile the project:

``
idf.py build

//Run the Temperature Sensor Program in QEMU

idf.py qemu monitor
``

On running, QEMU outputs a continuous simulated temperature feed:


----

# 8. 





