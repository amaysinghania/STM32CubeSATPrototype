# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an STM32H7-based CubeSat prototype project focused on camera integration using the OV5640 camera module. The project implements DCMI (Digital Camera Module Interface) with DMA for frame capture, along with SD card storage via FATFS for image data.

## Hardware Platform

- **MCU**: STM32H753ZITx (ARM Cortex-M7 @ 480MHz)
- **Board**: NUCLEO-H753ZI development board
- **Camera**: OV5640 5MP camera module
- **Storage**: SD card via SPI interface

## Build System

This project uses **STM32CubeIDE** as the primary development environment. The project is configured through STM32CubeMX (.ioc files) and builds using the STM32 HAL framework.

### Key Build Commands
Since this is an STM32CubeIDE project, building is typically done through the IDE. However, if using command line:
- The project uses GCC ARM toolchain
- Linker scripts: `STM32H753ZITX_FLASH.ld` and `STM32H753ZITX_RAM.ld`
- Debug configuration: `CameraTesting Debug.launch`

## Project Architecture

### Directory Structure
- `CameraTesting/` - Main project directory containing all source code
  - `Core/Src/` - Application source files including main.c and camera drivers
  - `Core/Inc/` - Header files and HAL configuration
  - `Drivers/` - STM32 HAL drivers and BSP files
  - `FATFS/` - FAT filesystem implementation for SD card
  - `Middlewares/` - Third-party middleware (FatFs)

### Key Components

#### Camera System
- **OV5640 Driver**: Custom driver implementation in `ov5640.c` and `ov5640.h`
- **DCMI Interface**: Digital Camera Module Interface for high-speed data capture
- **DMA Integration**: DMA1_Stream0 configured for DCMI data transfer
- **I2C Communication**: I2C1 used for camera module configuration (0x3C address)
- **Frame Buffer**: Located in SRAM1 for 640x480 frame storage

#### Peripheral Configuration
- **DCMI**: Configured for JPEG mode with rising edge clock polarity
- **I2C1**: Camera configuration interface (SCL: PB8, SDA: PB9)
- **SPI1**: SD card interface with CS on PB3
- **USART2**: Debug/logging interface
- **GPIO**: Camera control pins (Reset: PF1, Shutdown: PF0, Power: PB0, PB14)

#### Memory Management
- Frame buffer allocated in SRAM1 section for optimal DMA performance
- FATFS configured with tiny mode (_FS_TINY=1) for memory efficiency

### Clock Configuration
- System clock: 480MHz
- AHB: 240MHz
- APB1: 120MHz, APB2: 30MHz
- I2C clock source: HSI (64MHz)
- SPI clock: 64MHz with prescaler for SD card compatibility

## Development Workflow

### Camera Integration
The main focus is OV5640 camera integration with STM32H7. Current implementation includes:
- I2C initialization for camera configuration
- DCMI setup for frame capture
- DMA configuration for efficient data transfer
- Basic frame buffer management

### Pin Assignments (From .ioc Configuration)
- **DCMI Data Lines**: PC6-PC9 (D0-D3), PE4-PE6 (D4-D6), PD3 (D5)
- **DCMI Control**: PA4 (HSYNC), PA6 (PIXCLK), PG9 (VSYNC)
- **I2C1**: PB8 (SCL), PB9 (SDA) with pull-up resistors
- **SPI1**: PA5 (SCK), PB4 (MISO), PB5 (MOSI), PB3 (CS)
- **Camera Control**: PF0 (Shutdown), PF1 (Reset), PB0/PB14 (Power control)

## Development Notes

### Current Status
Based on README.md, the camera drivers are partially implemented but pin configuration for I2C, DCMI, and clock generation needs verification. The OV5640 drivers are present but may need debugging for proper communication.

### Key Areas for Development
1. **Camera Initialization**: Verify I2C communication with OV5640
2. **DCMI Configuration**: Ensure proper sync and data capture
3. **Frame Processing**: Implement JPEG decoding if needed
4. **SD Card Storage**: Integrate captured frames with FATFS
5. **Power Management**: Optimize for battery-powered operation

### Important Considerations
- Frame buffer is allocated in SRAM1 for DMA efficiency
- DCMI is configured for JPEG mode - verify camera output format compatibility
- SD card uses SPI interface - ensure adequate speed for frame storage
- I2C pull-up resistors are configured in hardware