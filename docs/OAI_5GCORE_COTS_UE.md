## OAI-5GCORE with COTS_UE

This tutorial describes how to configure and run a **monolithic 5G network** using the **OpenAirInterface (OAI)** components — specifically the **OAI CN5G** (core) and the **OAI gNB** — on a **single computer**.  

The goal is to demonstrate the complete operation of a 5G Standalone (SA) network by connecting it to a **COTS UE (Commercial Off-The-Shelf User Equipment)**, i.e., a **commercial 5G-compatible smartphone**.  

The tutorial covers the entire process from environment preparation and component compilation to connectivity testing with a real 5G device.

---

## System Requirements

Before starting, make sure your environment meets the minimum hardware and software requirements to run a monolithic 5G network with OpenAirInterface.

### Recommended Minimum Hardware

- **Single computer** running both OAI CN5G (core) and OAI gNB:  
  - **Operating system:** Ubuntu 24.04 LTS  
  - **CPU:** 8 cores x86_64 @ 3.5 GHz  
  - **RAM:** 32 GB  
  - **USRP Interface:** B210, N300, or X300  
  - Properly identify and configure the USRP network interface in the gNB configuration file.  
  - **User Equipment (UE):** in this setup, a **commercial 5G smartphone (COTS UE)** equipped with a **Sysmocom reprogrammable SIM card** was used.

### Dependencies and Reference Tutorials

This tutorial assumes you have already completed the basic installation and configuration steps of the following components:

- **UHD (USRP Hardware Driver)**  
  Full guide available at:  
  [Tutorial UHD - OAI Unisinos](../configs/TutorialUHD.md)

- **OAI Core Network (CN5G)**  
  Installation and configuration guide available at:  
  [OAI Core Network - CN5G Develop](../core/OAICoreNetwork-CN5G-Develop.md)

These tutorials provide the necessary foundation to prepare the environment before running the gNB and connecting the COTS UE.

---

## OAI gNB Compilation

In this step, you will **download the OpenAirInterface 5G (OAI)** source code and **compile the gNB (gNodeB)** with USRP hardware support.

### Compilation Steps:

Clone the official OAI repository:

```bash
# Get OAI source code
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop
# Install OAI dependencies
cd ~/openairinterface5g/cmake_targets
./build_oai -I
# Compile OAI gNB
cd ~/openairinterface5g/cmake_targets
./build_oai -w USRP --ninja --gNB -C
```
---

## UE (User Equipment) Configuration

Before running the 5G Core (OAI CN5G) and the gNB, you need to configure the UE (User Equipment), that is, the 5G smartphone that will connect to the network.

This step is essential to ensure that the device can authenticate correctly and establish communication with the 5G network.
You will need to configure:

- The reprogrammable Sysmocom SIM card, with the correct authentication parameters;

- The APN (Access Point Name) on the phone, so the data connection routes properly to the configured 5G Core.

The next sections describe the complete UE configuration process.

## SIM Card Programming (Sysmocom)

Before running the 5G network, you must program the SIM card that will be used by the 5G device (COTS UE).

In this setup, a 5G smartphone with a Sysmocom reprogrammable SIM card was used.
The SIM programming process is essential to define parameters such as IMSI, MCC, MNC, and authentication keys (K and OPC), ensuring proper authentication of the device in the network.

The full step-by-step guide is available below:

**SIM Programming Tutorial (Sysmocom)**  
👉 [TutorialSIM_UECots.md](../configs/TutorialSIM_UECots.md)

> ⚠️ **Importante:** Certifique-se de realizar a programação do SIM Card antes de iniciar o processo de conexão do celular à rede 5G, pois o chip precisa estar configurado com as credenciais corretas do OAI Core.

## Managing UE Records in the Database

Make sure to program the SIM card before attempting to connect the phone to the 5G network, as the SIM must be configured with the correct **OAI Core credentials**.

You can choose between:

1. **Manually adding new UE records to the database, or**  
2. **Using an existing pre-configured record.**.

To modify these settings, open the SQL file responsible for the network data:
```bash
oai-cn5g/database/oai_db.sql
```
Inside this file, locate the sections related to user authentication and session management:

- `-- Dumping data for table \`SessionManagementSubscriptionData\``
- `-- Dumping data for table \`AuthenticationSubscription\``

These tables store subscription details, IMSI, authentication keys (K and OPC), and parameters needed for UE authentication.

> ⚠️ **TIP:** Make sure that the values programmed in the Sysmocom SIM (IMSI, MCC, MNC, K, and OPC) match those stored in the database.  
> Otherwise, the device will fail to authenticate with the 5G Core.

## APN Configuration

Before connecting the phone to the 5G network, it is necessary to configure the **APN (Access Point Name)** to ensure that the data traffic is properly routed to the 5G Core (OAI CN5G).

The complete step-by-step guide for this configuration is available in the tutorial below:

📘 **5G Phone APN Configuration Tutorial**  
👉 [TutorialAPN.md](../configs/TutorialAPN.md)

> ⚠️ **Important:**  
> Make sure the APN uses the same **MCC**, **MNC**, and **DNN** configured in the Sysmocom SIM card and in the `config.yaml` file of the OAI Core.  
> These values must match for the UE to successfully authenticate with the 5G network.

---

## Run OAI CN5G and OAI gNB

After configuring the **UE**, we can proceed to run the **Core 5G (OAI CN5G)** and the **gNB**.

---

### Run OAI CN5G

To start the 5G Core:

```bash
cd ~/oai-cn5g
docker compose up -d
```
### Run OAI gNB
```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf -E --continuous-tx
```

## UE Connection to the 5G Network

After configuring the **UE** (SIM and APN) and starting the **5G Core (OAI CN5G)** and the **gNB**, the next step is to connect the mobile device to the 5G network.

### Connection Steps:

1. On the mobile device, **manually search for available networks**.  
2. Select the network corresponding to the configured **MCC and MNC** combination (for example, MCC=001 and MNC=01).  

> ⚠️ **Important:**  
> Make sure the mobile device is forced to connect **exclusively to the 5G network**, otherwise it may attempt to connect to nearby 4G or 3G networks.

### Monitoring the Connection

You can monitor the UE registration and connection traffic through the **gNB and Core logs**.

#### Logs do Core

#### Core Logs

To access the logs of a Core Docker container:

```bash
docker ps # To list the container names
docker logs <container_name> -f
```
It is recommended to check the AMF logs for debugging purposes:

```bash
docker logs oai-amf -f
```
#### gNB Losg

The gNB logs can be viewed directly in the terminal where the nr-softmodem was executed.
These logs allow you to monitor the UE registration process and identify possible authentication or configuration issues.

### UE Connection Test

After the UE successfully registers on the 5G network, you can test the connectivity:

1. **Access the Internet** directly from the mobile device (UE), by browsing or using data-dependent apps.
2. **Test connectivity via ping** from the UE’s host, using the **IP assigned by the Core**.

Example of a ping test from the UE:

```bash
docker exec -it oai-ext-dn ping 12.0.0.2
```
---

### Stability Notes

> ⚠️ **Warning:**  
> The system implemented with **OAI CN5G, gNB, and COTS UE** is still experimental and may present instabilities.  
> Even when following all the steps correctly, the network **may not function properly in some attempts**.

#### Troubleshooting Procedure

If a connection or UE registration failure occurs:

1. **Restart the entire system**, including:  
   - The **5G Core (OAI CN5G)**  
   - The **gNB**  
   - The **UE (mobile device)**  
2. Double-check the SIM and APN configurations before trying to reconnect.

> 🔄 **Tip:**  
> Multiple restarts may be required until the UE successfully registers on the 5G network.  
> Always monitor the logs carefully.

