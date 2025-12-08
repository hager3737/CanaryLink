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

## Scalability of AWS Components

Canary Link is designed using fully managed AWS services, which makes the system capable of scaling automatically as the number of devices or data volume increases. Below is an overview of how each AWS component contributes to scalability:

### **AWS IoT Core**
- Handles millions of simultaneous MQTT connections without requiring server management.
- Automatically scales with the number of connected devices (ideal for future multi-node mine deployments).
- Supports device provisioning, shadow services, and rules engine — all serverless and scalable.

### **AWS Lambda**
- Fully serverless compute: scales automatically with incoming telemetry events.
- No servers to configure; AWS spawns more Lambda instances automatically during heavy load.
- Pay-per-invocation model ensures cost efficiency, even when scaling to thousands of sensor messages per second.

### **DynamoDB**
- NoSQL database designed for massive throughput with single-digit millisecond latency.
- Automatically increases capacity via **On-Demand mode** if traffic increases.
- Stores telemetry and device metadata without requiring schema migrations.
- Ideal for time-series IoT data because it scales horizontally across partitions.

---

## Alternative Storage Options (Example: Amazon S3)

Even though Canary Link stores live telemetry in DynamoDB, other storage solutions could be used depending on the data retention and analytics requirements. For example:

### **Amazon S3 for cold or long-term storage**
- S3 provides durable object storage and is ideal for archiving large volumes of telemetry.
- IoT Core rules or Lambda could periodically **batch-export** telemetry into S3 for:
  - Data lake usage  
  - Long-term, low-cost storage  
  - Machine learning analytics (via Athena or SageMaker)  
  - CSV/JSON archives for auditors or mine operators  

### **How S3 could integrate with this project**
- Telemetry older than X days could be automatically written to S3.  
- A second Lambda could compress and archive datasets in:
  - **S3 Standard** or  
  - **S3 Glacier Deep Archive** for extra low cost.  
- The Next.js dashboard could display visualizations based on historical files stored in S3.

This demonstrates that the solution is flexible and can evolve depending on the data lifecycle and storage needs.

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