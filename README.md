# Canary Link
A complete IoT telemetry system designed for mining environments, consisting of hardware, a gateway, backend cloud services, and a frontend dashboard.  
Developed by [Johann](https://github.com/hager3737).

Each part of the system exists as its own module (ESP32 node, nRF7002DK gateway, Amplify dashboard).  
See individual repositories for more details.

---

## Overview

## Architecture

![Canary Link Architecture](/images/flow.png)

---

### Hardware (ESP32 LoRa Sensor Node)
A low-power underground sensor node that collects temperature readings and transmits them using long-range LoRa.

**Features**
- ESP32 NodeMCU Dev Kit C  
- SX1278 LoRa (433 MHz)  
- LM75 temperature sensor  
- ESP-IDF (FreeRTOS)  
- Periodic temperature measurements  
- LoRa transmission with SX127x driver  
- TX callbacks, FIFO resets, interrupt-driven sending  
- Custom LM75 and LoRa driver integration

**ESP32 terminal**

![Canary Link esp](/images/esp-terminal.png)

**Repository**  
https://github.com/hager3737/esp32-lora-transmitter

---

### Gateway (nRF7002DK LoRa → Wi-Fi → AWS)
A surface-level gateway that receives LoRa packets from the mine and forwards them to AWS IoT Core.

**Features**
- Nordic nRF5340 + nRF7002 Wi-Fi  
- Zephyr RTOS  
- Receives LoRa packets from ESP32 nodes  
- Sends MQTT messages to AWS IoT Core  
- Automatic reconnect and backoff logic  
- mTLS (mutual TLS) with X.509 certificates  
- Secure encrypted MQTT transport

**NRF Terminal**

![Canary Link nrf](/images/nrf-terminal.png)

**Repository**  
https://github.com/hager3737/nrf-aws-mqtt

---

### Backend (AWS Cloud Pipeline)

**Flow**
1. LoRa data arrives at AWS IoT Core  
2. IoT Rule forwards telemetry to Lambda  
3. Lambda validates and enriches telemetry  
4. DynamoDB stores:
   - Devices
   - Telemetry (timestamped data)
5. If temperature > threshold → Lambda sends alert to Discord

**Security**
- IAM role restrictions  
- API Key for GraphQL API  
- Owner-bound authorization  
- Mutual TLS at device → cloud  

---

### Frontend (Next.js + AWS Amplify Dashboard)

A modern dashboard for managing devices and viewing telemetry.

**Features**
- Next.js 14 App Router  
- Auth via AWS Cognito  
- Amplify Gen 2 GraphQL client  
- Temperature chart
- Device registration & deletion  
- Real-time updates

**Data visualization**

![Canary Link Data display](/images/data.png)

![Canary Link discord display](/images/discord-data.png)

**Repository**  
https://github.com/hager3737/amplify-next-template

---

## Future Improvements

Planned upgrades include:

- Downlink commands from dashboard → nodes  
- Over-the-air firmware updates (OTA)  
- Additional sensors (humidity, gas detection)  
- Multi-node mesh support

---

## Author

**Johann Bonde Hager**  
IoT & Embedded Systems Developer  
GitHub: https://github.com/hager3737

---