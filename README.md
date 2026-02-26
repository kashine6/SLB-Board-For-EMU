# SLB-Board-For-EMU


## Table of Contents

- [Introduction](#introduction)
- [1. PIN Diagram](#1-pin-diagram)
- [2. EMU Wiring Diagram](#2-emu-wiring-diagram)
- [3. EMU Configuration](#3-emu-configuration)
- [4. Flashing Guide](#4-flashing-guide)
  - [4.1. Flashing Katapult (Optional)](#41-flashing-katapult-optional)
  - [4.2. Compiling Klipper Firmware](#42-compiling-klipper-firmware)
  - [4.3. Updating Firmware via Katapult](#43-updating-firmware-via-katapult)
  - [4.4. Updating Firmware via DFU](#44-updating-firmware-via-dfu)

---

## Introduction
**SLB (Solo Lane Board) ** is a control board designed specifically for **EMU**-style MMU systems, where each lane is driven by an independent controller board.  

No WAGO connectors and no LED daisy-chaining. Supports power and CANBUS cascading, and is compatible with the EMU_Hatch_Board, greatly simplifying EMU wiring.



This motherboard is factory-produced using high-quality electronic components with SMT assembly, ensuring reliable and consistent quality.

<img src="Assets/1-4.png" width="100%"/>

**It has the following features:**

- **Main Chip**: STM32G0B1CBT6 MCU
- **1 x Onboard TMC2209 stepper driver** operating in UART mode, with 5V logic circuitry.
- **1x 8-pin XH2.54 connector** is provided for direct connection to the **EMU Hatch Board**.
- **Dual power connectors and dual CAN interfaces** for convenient daisy-chaining.
- **2 × Neopixel RGB interfaces**, one of which is integrated into the 8-pin XH2.54 connector.
- **6 × I/O connectors**, two of which are integrated into the 8-pin XH2.54 connector.
- **3 × I2C interfaces**, one of which is integrated into the 8-pin XH2.54 connector.
- **2 × 24V fans** with adjustable speed control
- **1 × PTC heating interface**, 5A max
- **1 × NTC temperature sensor** interface for temperature monitoring
- **1 x Hall interfaces** for PSF Sensor
- **1 x 5V Brushed DC motor drivers** with forward/reverse support, En pin, and sleep mode
- **1 x SPI interfaces**

  

## 1. PIN Diagram

<img src="Assets/1-2.jpg" width="100%"/>




## 2. EMU Wiring Diagram

<img src="Assets/1-7.jpg" width="100%"/>

<img src="Assets/1-6.png" width="50%"/>

To be defined






## 3. EMU Configuration

To be defined



## 4. Flashing Guide

### 4.1 Flashing Katapult (Optional)

Users who only use a CAN connection and want the convenience of quick firmware updates by pressing the Reset button can choose to flash Katapult (formerly known as Canboot).

Katapult is not mandatory, regardless of whether you use a CAN or USB connection.

```bash
# Go to the home directory
cd ~
 
# Download katapult (skip if you already have it)
git clone https://github.com/Arksine/katapult
 
# Enter the katapult directory
cd katapult
```




![img](Assets/3.png)






```bash
# Clean the previous build
make clean
 
# Modify the build configuration, fill in the parameters as shown in the image below
make menuconfig
 
# After verifying the settings, press q to exit, select y, then type make to start compiling
make
```


![img](Assets/4.png)

> [!TIP]
> The speed can be set according to your needs, 1000000 is recommended.




![img](Assets/5.png)



![img](Assets/6.png)


After compilation, katapult.bin will be generated in the ~/katapult/out folder.



**Entering DFU Mode**

Use a jumper to short the 5V USB pins. After shorting, you can power the board via USB for flashing. 

> [!TIP]
> Remember to remove the jumper after flashing is complete.  

If you are using 24V to power the board, you do not need to short the jumper, but you can only use Method 2 below to enter DFU mode.

<img src="Assets/7.png" width="100%"/>

- **Method 1 to enter DFU:** With the board completely powered off, press and hold the Boot button, connect the board to the host computer with a Type-C cable, then release the Boot button.

- **Method 2 to enter DFU:** If the board is powered on, connect it to the host computer with a Type-C cable, then press and hold the Boot button, press and release the Reset button, release the Reset button, and finally release the Boot button.

Then use the lsusb command to check if a DFU device is present.



```bash
lsusb
```


![img](Assets/8.png)




Use the following command to flash the board:

```bash
make flash FLASH_DEVICE=0483:df11
```


![img](Assets/9.png)




After flashing is complete, please unplug the USB cable and connect the CAN signal cable and 24V power supply.

> [!NOTE]
> 1: Remember to remove the 5V USB jumper.  
> 2: If you have multiple CAN devices, you only need to short the 120-ohm resistor on one device. It is best to short the 120-ohm resistor on the mainboard at the end of the chain.  
> 3: Pay close attention to the wiring order of 24V, GND, CAN-H, and CAN-L. Double-check carefully. Incorrect wiring may cause the mainboard to burn out, and you will be responsible for the consequences.  

<img src="Assets/10.png" width="100%"/>





Use the following command to check the CAN UUID. 
> [!TIP]
> You may need to quickly double-press the reset button to enter the bootloader.

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```


![img](Assets/11.png)




### 4.2 Compiling Klipper Firmware

SSH into your host machine.

```bash
# cd to the klipper directory
cd ~/klipper
 
# Clean the previous build
make clean
 
# Configure the build parameters [refer to the image below]
make menuconfig
```


![img](Assets/12.png)




Set the build parameters according to the protocol you need to use.


![img](Assets/13.png)



```bash
# If not using Katapult
Bootloader offset (No bootloader) --->
 
# If using Katapult
Bootloader offset (8KiB bootloader) --->
 
# If using USB communication via Type-C
Communication interface (USB (on PA11/PA12)) --->
 
# If using CANBus communication
Communication interface (CAN bus (on PD0/PD1)) --->
 (1000000) CAN bus speed
```

Press `q` to exit, and `y` to save.


![img](Assets/14.png)




Enter the make command to start compiling.

```bash
make
```


![img](Assets/15.png)

![img](Assets/16.png)




### 4.3 Updating Firmware via Katapult

Connect the CAN signal cable and 24V power supply.

> [!NOTE]
> 1: Remember to remove the 5V USB jumper.  
> 2: If you have multiple CAN devices, you only need to short the 120-ohm resistor on one device.  
> 3: Pay close attention to the wiring order of 24V, GND, CAN-H, and CAN-L. Double-check carefully. Incorrect wiring may cause the mainboard to burn out, and you will be responsible for the consequences.  

<img src="Assets/10.png" width="100%"/>





Then use the following command to check the UUID:

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```


![img](Assets/18.png)




Use the following command to flash in CAN mode:

```bash
cd ~/katapult/scripts
 
python3 flashtool.py -i can0 -f ~/klipper/out/klipper.bin -u your_uuid_here
```


![img](Assets/19.png)




Then continue to use the previous command to check the CAN information.


![img](Assets/20.png)



> [!TIP]
> To upgrade the CAN firmware in the future, repeat this process. When compiling the Katapult firmware earlier, we set it to enter Katapult mode by double-pressing the reset button. 
> In practice, you can also directly enter the flashing command, and the board will automatically restart and enter Katapult.  

### 4.4 Updating Firmware via DFU

1: Enter DFU mode.

Use a jumper to short the 5V USB pins. After shorting, you can power the board via USB for flashing. 

> [!TIP]
> Remember to remove the jumper after flashing is complete.

If you are using 24V to power the board, you do not need to short the jumper, but you can only use Method 2 below to enter DFU mode.

<img src="Assets/21.png" width="100%"/>




- Method 1 to enter DFU: With the board completely powered off, press and hold the Boot button, connect the board to the host computer with a Type-C cable, then release the Boot button.

- Method 2 to enter DFU: If the board is powered on, connect it to the host computer with a Type-C cable, then press and hold the Boot button, press and release the Reset button, release the Reset button, and finally release the Boot button.

2: In SSH, type lsusb to check if a DFU device appears. Normally, a DFU device should appear.

If no DFU device appears, repeat the operation.


![img](Assets/22.png)



3: In SSH, enter:

```bash
# Make sure you are in the klipper directory
cd ~/klipper
 
# 0483:df11 is the ID found with lsusb earlier
# If it says make not found, install it with sudo apt install dfu-util -y
 
make flash FLASH_DEVICE=0483:df11
```

The flashing process will then begin [you may need to enter your password]. Wait for the progress bar to complete. When "successfully" appears, it means the flashing was successful.

You can ignore any errors that appear after "successfully".


![img](Assets/23.png)


Flashing is complete.

4: **Get the serial ID**

Unplug and replug the USB data cable, use lsusb to check if there is an stm32f446 device.

Use the following command to check the device's serial ID:

```bash
ls /dev/serial/by-id/
```


![img](Assets/24.png)




The serial ID for this device is:

```bash
serial: /dev/serial/by-id/usb-Klipper_stm32g0b1xx_20003A00025030544E383020-if00
```

> [!TIP]
> For later use, please remove the 5V USB jumper and connect 24V power to the CAN interface.


