---
platform: raspbian
services: iot-hub, stream-analytics, event-hubs, web-apps, documentdb, storage-accounts
language: c
---

# Connect Raspberry Pi to Azure IoT Hub in the cloud

In this tutorial, you begin by learning the basics of working with Raspberry Pi that's running Raspbian. You then learn how to seamlessly connect your devices to the cloud by using Azure IoT Hub. For Windows 10 IoT Core samples, go to the [Windows Dev Center](http://www.windowsondevices.com/).

## What you do

* Setup Raspberry Pi.
* Create an IoT hub.
* Register a device for Pi in your IoT hub.
* Run a sample application on Pi to send sensor data to your IoT hub.

Connect Raspberry Pi to an IoT hub that you create. Then you run a sample application on Pi to collect temperature and humidity data from a BME280 sensor. Finally, you send the sensor data to your IoT hub.

## What you learn

* How to create an Azure IoT hub and get your new device connection string.
* How to connect Pi with a BME280 sensor.
* How to collect sensor data by running a sample application on Pi.
* How to send sensor data to your IoT hub.

## What you need

![What you need](media/iot-hub-raspberry-pi-kit-c-get-started/0_starter_kit.jpg)

* The Raspberry Pi 2 or Raspberry Pi 3 board.
* An active Azure subscription. If you don't have an Azure account, [create a free Azure trial account](https://azure.microsoft.com/free/) in just a few minutes.
* A monitor, a USB keyboard, and mouse that connect to Pi.
* A Mac or a PC that is running Windows or Linux.
* An Internet connection.
* A 16 GB or above microSD card.
* A USB-SD adapter or microSD card to burn the operating system image onto the microSD card.
* A 5-volt 2-amp power supply with the 6-foot micro USB cable.

The following items are optional:

* An assembled Adafruit BME280 temperature, pressure, and humidity sensor.
* A breadboard.
* 6 F/M jumper wires.
* A diffused 10-mm LED.


> **NOTE:** 
These items are optional because the code sample support simulated sensor data.


## Setup Raspberry Pi

### Install the Raspbian operating system for Pi

Prepare the microSD card for installation of the Raspbian image.

1. Download Raspbian.
   -  [Download Raspbian Jessie with Pixel](https://www.raspberrypi.org/downloads/raspbian/) (the .zip file).
   -  Extract the Raspbian image to a folder on your computer.
2. Install Raspbian to the microSD card.
   -  [Download and install the Etcher SD card burner utility](https://etcher.io/).
   -  Run Etcher and select the Raspbian image that you extracted in step 1.
   -  Select the microSD card drive. Note that Etcher may have already selected the correct drive.
   -  Click Flash to install Raspbian to the microSD card.
   -  Remove the microSD card from your computer when installation is complete. It's safe to remove the microSD card directly because Etcher automatically ejects or unmounts the microSD card upon completion.
   -  Insert the microSD card into Pi.

### Enable SSH and SPI

1. Connect Pi to the monitor, keyboard and mouse, start Pi and then log in Raspbian by using `pi` as the user name and `raspberry` as the password.
2. Click the Raspberry icon > **Preferences** > **Raspberry Pi Configuration**.

   ![The Raspbian Preferences menu](media/iot-hub-raspberry-pi-kit-c-get-started/1_raspbian-preferences-menu.png)

3. On the **Interfaces** tab, set **SPI** and **SSH** to **Enable**, and then click **OK**.

   ![Enable SPI and SSH on Raspberry Pi](media/iot-hub-raspberry-pi-kit-c-get-started/2_enable-spi-ssh-on-raspberry-pi.png)

> **NOTE:** 
To enable SSH and SPI, you can find more reference documents on [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) and [RASPI-CONFIG](https://www.raspberrypi.org/documentation/configuration/raspi-config.md).

### Connect the sensor to Pi

Use the breadboard and jumper wires to connect an LED and a BME280 to Pi as follows. If you don’t have the sensor, skip this section.

![The Raspberry Pi and sensor connection](media/iot-hub-raspberry-pi-kit-c-get-started/3_raspberry-pi-sensor-connection.png)

For sensor pins, use the following wiring:

| Start (Sensor & LED)     | End (Board)            | Cable Color   |
| -----------------------  | ---------------------- | ------------: |
| LED VDD (Pin 5G)         | GPIO 4 (Pin 7)         | White cable   |
| LED GND (Pin 6G)         | GND (Pin 6)            | Black cable   |
| VDD (Pin 18F)            | 3.3V PWR (Pin 17)      | White cable   |
| GND (Pin 20F)            | GND (Pin 20)           | Black cable   |
| SCK (Pin 21F)            | SPI0 SCLK (Pin 23)     | Orange cable  |
| SDO (Pin 22F)            | SPI0 MISO (Pin 21)     | Yellow cable  |
| SDI (Pin 23F)            | SPI0 MOSI (Pin 19)     | Green cable   |
| CS (Pin 24F)             | SPI0 CS (Pin 24)       | Blue cable    |

Click to view [Raspberry Pi 2 & 3 Pin mappings](https://developer.microsoft.com/windows/iot/docs/pinmappingsrpi) for your reference.

After you've successfully connected BME280 to your Raspberry Pi, it should be like below image.

![Connected Pi and BME280](media/iot-hub-raspberry-pi-kit-c-get-started/4_connected-pi.jpg)

Turn on Pi by using the micro USB cable and the power supply. Use the Ethernet cable to connect Pi to your wired network or follow the [instructions from the Raspberry Pi Foundation](https://www.raspberrypi.org/learning/software-guide/wifi/) to connect Pi to your wireless network.

![Connected to wired network](media/iot-hub-raspberry-pi-kit-c-get-started/5_power-on-pi.jpg)


## Run a sample application on Pi

### Install the prerequisite packages

1. Use one of the following SSH clients from your host computer to connect to your Raspberry Pi.
    - [PuTTY](http://www.putty.org/) for Windows.
    - The built-in SSH client on Ubuntu or macOS.

2. Install the prerequisite packages for the Microsoft Azure IoT Device SDK for C and Cmake by running the following commands:

   ```bash
   grep -q -F 'deb http://ppa.launchpad.net/aziotsdklinux/ppa-azureiot/ubuntu vivid main' /etc/apt/sources.list || sudo sh -c "echo 'deb http://ppa.launchpad.net/aziotsdklinux/ppa-azureiot/ubuntu vivid main' >> /etc/apt/sources.list"
   grep -q -F 'deb-src http://ppa.launchpad.net/aziotsdklinux/ppa-azureiot/ubuntu vivid main' /etc/apt/sources.list || sudo sh -c "echo 'deb-src http://ppa.launchpad.net/aziotsdklinux/ppa-azureiot/ubuntu vivid main' >> /etc/apt/sources.list"
   sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FDA6A393E4C2257F
   sudo apt-get update
   sudo apt-get install -y azure-iot-sdk-c-dev cmake
   ```


### Configure the sample application

1. Clone the sample application by running the following command:

   ```bash
   git clone https://github.com/Azure-Samples/iot-hub-c-raspberrypi-client-app
   ```
2. Open the config file by running the following commands:

   ```bash
   cd iot-hub-c-raspberry-pi-clientapp
   nano config.json
   ```

   ![Config file](media/iot-hub-raspberry-pi-kit-c-get-started/6_config-file.png)

   There are two macros in this file you can configurate. The first one is `INTERVAL`, which defines the time interval between two messages that send to cloud. The second one `SIMULATED_DATA`,which is a Boolean value for whether to use simulated sensor data or not.

   If you **don't have the sensor**, set the `SIMULATED_DATA` value to `1` to make the sample application create and use simulated sensor data.

3. Save and exit by pressing Control-O > Enter > Control-X.

### Build and run the sample application

1. Build the sample application by running the following command:

   ```bash
   cmake . && make
   ```
   ![Build output](media/iot-hub-raspberry-pi-kit-c-get-started/7_build-output.png)

2. Run the sample application by running the following command:

   ```bash
   sudo ./app '<device connection string>'
   ```

   > **NOTE:** 
   Make sure you copy-paste the device connection string into the single quotes.


You should see the following output that shows the sensor data and the messages that are sent to your IoT hub.

![Output - sensor data sent from Raspberry Pi to your IoT hub](media/iot-hub-raspberry-pi-kit-c-get-started/8_run-output.png)

## Next steps

You’ve run a sample application to collect sensor data and send it to your IoT hub.

You have now learned how to run a sample application that collects sensor data and sends it to your IoT hub. To explore how to store, analyze and visualize the data from this application in Azure using a variety of different services, please click on the following lessons:

-   [Manage cloud device messaging with iothub-explorer]
-   [Save IoT Hub messages to Azure data storage]
-   [Use Power BI to visualize real-time sensor data from Azure IoT Hub]
-   [Use Azure Web Apps to visualize real-time sensor data from Azure IoT Hub]
-   [Weather forecast using the sensor data from your IoT hub in Azure Machine Learning]
-   [Remote monitoring and notifications with Logic Apps]   

[Manage cloud device messaging with iothub-explorer]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-explorer-cloud-device-messaging
[Save IoT Hub messages to Azure data storage]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-store-data-in-azure-table-storage
[Use Power BI to visualize real-time sensor data from Azure IoT Hub]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-live-data-visualization-in-power-bi
[Use Azure Web Apps to visualize real-time sensor data from Azure IoT Hub]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-live-data-visualization-in-web-apps
[Weather forecast using the sensor data from your IoT hub in Azure Machine Learning]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-weather-forecast-machine-learning
[Remote monitoring and notifications with Logic Apps]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps