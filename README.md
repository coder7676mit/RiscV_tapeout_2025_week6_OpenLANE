# Report

Table of Contents

Introduction
 1.1 FOSSEE and the Open-Source Ecosystem
 1.2 Importance of Open-Source EDA, Embedded Tools, and Emulation
 1.3 Current Trends and Future Direction of Open-Source Hardware Tools

Problem Statement
 2.1 Objective
 2.2 Scope of Work
 2.3 Expected Outcomes

Background and Prerequisites
 3.1 Overview of ESP-IDF
  3.1.1 What is ESP-IDF
  3.1.2 Structure and Components
  3.1.3 Toolchains, Python Environment, and Build System
 3.2 Overview of QEMU for ESP32
  3.2.1 What is QEMU
  3.2.2 How ESP32 Emulation Works
  3.2.3 Limitations of Emulation
 3.3 System Prerequisites
  3.3.1 Linux Environment
  3.3.2 Required Packages (Python, pip, CMake, Ninja, Git, etc.)
  3.3.3 Validating the Environment
 3.4 Yaksh Platform Overview
  3.4.1 What is Yaksh
  3.4.2 Why an ESP32 Emulator is Needed
  3.4.3 Requirements for Automated Code Evaluation

ESP32 Development Environment Setup
 4.1 Setting Up ESP-IDF
 4.2 Installing ESP-IDF Tools
 4.3 Verifying ESP-IDF Installation
 4.4 Cloning and Configuring QEMU for ESP32
 4.5 Building QEMU From Source

Common Errors and Their Resolutions
 5.1 Missing Headers, Libraries, or Dependencies
 5.2 Linking Errors During QEMU Build
 5.3 Issues With ESP-IDF Python Environment
 5.4 How Each Issue Was Identified and Fixed

Verification of Installations
 6.1 Commands to Verify Toolchains
 6.2 Testing QEMU Execution
 6.3 Testing ESP-IDF Build System
 6.4 (Optional) Screenshot references

Running and Demonstrating ESP32 Examples
 7.1 LED Blinking Program
  7.1.1 Code
  7.1.2 Build
  7.1.3 Run on QEMU
  7.1.4 Output Verification
 7.2 Temperature Sensor Example
  7.2.1 Code
  7.2.2 Build
  7.2.3 Run on QEMU
  7.2.4 Output Verification

Integrating the Setup With the Yaksh Platform
 8.1 Automated Code Grading Architecture
 8.2 Using QEMU Execution for Non-Hardware Evaluation
 8.3 Designing Auto-Evaluation Test Cases
 8.4 Example: Auto-grading LED Blink Submission

Future Scope and Project Extensions
 9.1 Enhancing the Emulator Environment
 9.2 Expanding to Other ESP Series
 9.3 Using This Setup for Research and Automation
 9.4 Building Complete CI Pipelines With ESP-IDF + QEMU
 9.5 Potential Student and Academic Projects

1 Introduction
1.1 FOSSEE and the Open-Source Ecosystem

The FOSSEE initiative plays a crucial role in promoting open-source tools for engineering, scientific computing, simulation, and education. The core motivation is to replace costly proprietary software with free, transparent, and modifiable alternatives. This philosophy empowers students and developers to explore internal mechanisms, contribute improvements, and adapt tools to suit academic and research needs.

Open-source toolchains remove licensing restrictions, encourage innovation, and enable institutions to adopt scalable development environments.

1.2 Importance of Open-Source EDA, Embedded Tools, and Emulation

Embedded systems education traditionally depends on proprietary IDEs or physical hardware kits. Open-source frameworks such as ESP-IDF and QEMU allow developers to:

• inspect the internal build system, toolchain, and compilation flow
• emulate hardware to perform testing without requiring physical boards
• automate learning and evaluation systems
• experiment with hardware-level behavior at zero cost

QEMU emulation extends these capabilities by offering virtual hardware platforms where one can test firmware, debug system behavior, and validate embedded programs.

1.3 Current Trends and Future Direction of Open-Source Hardware Tools

The shift toward open hardware is accelerating through efforts like RISC-V, OpenTitan, open FPGA toolchains, and community-maintained development frameworks. Emulation-based testing, CI-driven firmware development, and automated classroom evaluation systems are becoming critical educational tools.

ESP32 emulation using QEMU represents a foundational component of this movement. It enables scalable embedded systems teaching without requiring every learner to have hardware access.

2 Problem Statement
2.1 Objective

The objective is to create a fully functional ESP32 firmware development and emulation environment using:

• ESP-IDF (Espressif development framework)
• QEMU (Espressif-maintained fork with Xtensa support)

The goal includes compiling and running ESP32 firmware examples and validating them using emulation.

2.2 Scope of Work

The scope includes:

• setting up ESP-IDF toolchains
• compiling QEMU for the Xtensa architecture
• resolving build-time issues
• testing two sample ESP32 projects (LED and temperature simulation)
• preparing this setup to work with the Yaksh automated evaluation platform

2.3 Expected Outcomes

Expected deliverables include:

• a clean ESP32-QEMU environment
• demonstrable firmware examples running on the emulator
• a documented process flow for installation and debugging
• instructions for integrating automated grading

3 Background and Prerequisites
3.1 Overview of ESP-IDF
3.1.1 What is ESP-IDF

ESP-IDF is the official development framework for ESP32 and related chips. It provides:

• firmware libraries
• build system based on CMake and Python
• hardware abstraction and APIs
• toolchain installer and environment manager

ESP-IDF supports both real hardware and QEMU-based emulation.

3.1.2 Structure and Components

ESP-IDF consists of:

• components (drivers, WiFi, Bluetooth, peripherals, FreeRTOS, etc.)
• Kconfig-based build options
• a Python-managed environment
• cross-compiler toolchains
• CMake-based build rules

3.1.3 Toolchains, Python Environment, and Build System

ESP-IDF requires:

• Xtensa-ESP32 GCC compiler
• Python 3 + pip environment
• CMake, Ninja, and Git
• IDF-specific tools installed using install.sh

3.2 Overview of QEMU for ESP32
3.2.1 What is QEMU

QEMU is an open-source machine emulator supporting numerous architectures.
Espressif maintains a fork that adds:

• Xtensa CPU support
• ESP32 device emulation (timer, UART, flash, etc.)

3.2.2 How ESP32 Emulation Works

ESP32 binaries compiled using ESP-IDF can be executed inside QEMU using virtual peripherals.
This allows testing:

• bootloader
• partitions
• basic peripherals
• UART output

3.2.3 Limitations of Emulation

Some complex peripherals might not be modeled fully.
However, for basic firmware verification, QEMU is reliable.

3.3 System Prerequisites
3.3.1 Linux Environment

A standard Linux distribution is recommended for building ESP-IDF and QEMU.

3.3.2 Required Packages

Typical prerequisites include:
