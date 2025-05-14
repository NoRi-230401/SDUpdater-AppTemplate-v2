# SDUpdater-AppTemplate v2
**[` 日本語 `](README_jp.md)**

## 1. Introduction

This document explains how to use the M5Stack device application template with SD card updater functionality, provided in `SDUpdater-AppTemplate-v2\src\main.cpp`.

This template allows you to easily load and run new firmware or a menu program (`menu.bin`) stored on an SD card by performing a specific operation when the device starts up.

## 2. Supported Devices

This template supports the following M5Stack devices:

*   M5Stack Cardputer/Cardputer v1.1
*   M5Stack Core2/Core2 for AWS/Core2 v1.1
*   M5Stack CoreS3/CoreS3-SE

## 3. Key Features

*   **Device Initialization**:
    *   Serial communication baud rate setting (115200 bps).
    *   Display initialization (brightness, orientation, background color, text size).
    *   SD card initialization.
        *   Cardputer: Uses SPI2 interface.
        *   Core2/CoreS3: Uses standard SPI interface (GPIO_NUM_4).
    *   Error display and process termination upon SD card initialization failure.
*   **SD Updater Lobby (`SDU_lobby`)**:
    *   Detects if a specific key is pressed during device startup.
        *   **M5Stack Cardputer**: 'a' key
        *   **M5Stack Core2 / CoreS3**: BtnA (the leftmost button)
    *   If the key is pressed, it loads the `/menu.bin` file from the root directory of the SD card, reboots the device, and executes it.
    *   **For M5Stack CoreS3**: Considering the issue where the button might be hard to recognize, it displays the message "Press BtnA to load menu" for 3 seconds after startup, allowing `menu.bin` to be loaded by pressing BtnA during that time.

## 4. How to Use

### 4.1. Preparation

1.  Save the firmware or menu program you want to load as `menu.bin` in the root directory of your SD card.
    *   This file must be in a format compatible with the M5StackUpdater library. Usually, it's a compiled binary file.

### 4.2. How to Launch the SD Updater

1.  **For M5Stack Cardputer**:
    *   Press and hold the 'a' key while powering on the device or pressing the reset button.
2.  **For M5Stack Core2**:
    *   Press and hold BtnA (the leftmost button) while powering on the device or pressing the reset button.
3.  **For M5Stack CoreS3**:
    *   Power on the device or press the reset button.
    *   A green message "Press BtnA to load menu" will be displayed on the screen.
    *   Press BtnA (the leftmost button) within the 3 seconds the message is displayed.

If the above operation is successful, `menu.bin` will be loaded from the SD card, and the device will restart to run the new program.

### 4.3. Normal Boot

If you start the device without performing the key operations described above, after `SDU_lobby()` in the `setup()` function is executed, "Hello World!" will be displayed on the screen by `disp("Hello World!");`. Then, the `loop()` function will run repeatedly (currently, `loop()` is empty in this template).

## 5. Code Overview

*   **`setup()`**:
    *   `m5stack_begin()`: Calls the device initialization process.
    *   `SDU_lobby()`: Calls the SD updater lobby function.
    *   `disp("Hello World!")`: Displays a message during normal boot.
*   **`loop()`**:
    *   This is the main loop. Write your application's main logic here. It is currently empty in this template.
*   **`m5stack_begin()`**:
    *   Uses `M5.config()` to set the serial baud rate, etc.
    *   Display and SD card initialization processes are separated for Cardputer and other devices (Core2, CoreS3).
    *   It attempts to initialize the SD card 10 times, and if it fails, it displays an error message and stops.
*   **`SDU_lobby()`**:
    *   Checks for key input at startup (Cardputer: 'a' key, others: BtnA) depending on the device type.
    *   For CoreS3, it includes a special process to display a message prompting a button press for 3 seconds.
    *   If the key is pressed, it calls `updateFromFS(SD, "/menu.bin")` to load the firmware from the SD card and `ESP.restart()` to reboot the device.
*   **`disp(String msg)`**:
    *   Displays the specified string on the screen. The display object used differs between Cardputer and other devices.

## 6. Customization

*   **Developing Your Own Application**:
    *   Write the main logic for your application in the `loop()` function.
    *   If necessary, you can add initialization code after `disp("Hello World!");` in the `setup()` function.
*   **Changing the File Name to Load**:
    *   By changing the second argument of `updateFromFS(SD, "/menu.bin");` in the `SDU_lobby()` function, you can change the name of the file to be loaded from the SD card. Example: `updateFromFS(SD, "/myapp.bin");`
*   **Changing SD Card Pin Settings (Advanced)**:
    *   You can change the SD card's SPI pin settings and initialization parameters in the `m5stack_begin()` function, but usually, the default settings work fine.

## 7. Important Notes

*   Ensure that the SD card is correctly inserted and formatted.
*   Ensure that the `menu.bin` file exists in the root directory of the SD card.
*   If SD card initialization fails, "ERR: SD begin ..." will be displayed on the screen, and the program will stop. In this case, check the SD card's connection and status.
*   When loading `menu.bin` on M5Stack CoreS3, you need to press BtnA within 3 seconds while the message is displayed.


## 8. Link

[SDUpdater-AppTemplate-v2](https://github.com/NoRi-230401/SDUpdater-AppTemplate-v2)

[SimpleMenu-SDUpdater](https://github.com/NoRi-230401/SimpleMenu-SDUpdater)

[CardputerSimpleLauncher@shikarunochi](https://github.com/shikarunochi/CardputerSimpleLaucher)

[M5Stack-SD-Updater@tobozo](https://github.com/tobozo/M5Stack-SD-Updater)


