## OAI-5GCore with nrUE

This tutorial aims to demonstrate the complete process for **configuring and running a functional 5G network**, composed of a **Core Network (CN5G)**, a **gNB**, and a **UE**, using the **OpenAirInterface (OAI)** ecosystem.

The scenario presented uses a **USRP B210** as the base station (**gNB**) and a second computer, also equipped with a USRP, acting as the **UE (User Equipment)**.  
Thus, instead of a commercial device, the UE will be emulated by another host running the **OAI nrUE** software.

Throughout this tutorial, all the necessary steps for setting up the environment will be covered, including the installation of prerequisites, building the OAI modules, and initializing the main components of the 5G network.

---
## System Requirements

To run this complete 5G environment, you will need **two computers** and at least **two USRP radios** (e.g., USRP B210).  
Each machine will play a specific role within the setup, as described below.

### Computer 1 — OAI Core Network (CN5G) + gNB

This computer will be responsible for running both the **Core Network (CN5G)** and the **gNB (Next Generation NodeB)**.

**Recommended specifications:**
- **Operating System:** Ubuntu 24.04 LTS  
- **Processor:** 8 cores x86_64 @ 3.5 GHz  
- **RAM:** 32 GB  
- **Radio Device:** USRP B210 (or equivalent)  

For installation and configuration of the 5G Core, follow the detailed tutorial available at:  
👉 [OAI Core Network — CN5G (Develop)](../core/OAICoreNetwork-CN5G-Develop.md)

---
### Computer 2 — OAI nrUE

The second computer will be used to emulate the **UE (User Equipment)**, also connected to a USRP.  
This machine will simulate the behavior of a real 5G device.

**Recommended specifications:**
- **Operating System:** Ubuntu 24.04 LTS  
- **Processor:** 8 cores x86_64 @ 3.5 GHz  
- **RAM:** 8 GB  
- **Radio Device:** USRP B210 (or equivalent)

---

### UHD Configuration

Both the **Core/gNB** and the **UE** computers must have the **UHD (USRP Hardware Driver)** properly installed and configured.  
The installation and compilation process can be followed in the official guide below:

👉 [UHD Installation Tutorial](../configs/TutorialUHD.md)

---

## Building OAI gNB and OAI nrUE

After installing UHD, proceed to obtain and compile the main components of OpenAirInterface.

### Compiling OAI gNB and OAI nrUE

Download the latest **OpenAirInterface 5G** source code, install the required dependencies, and compile the necessary modules for both the gNB and nrUE on each computer.

```bash
# Obter o código-fonte do OpenAirInterface
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git ~/openairinterface5g
cd ~/openairinterface5g
git checkout develop

# Instalar dependências do OAI
cd ~/openairinterface5g/cmake_targets
./build_oai -I

# Instalar dependências do nrscope
sudo apt install -y libforms-dev libforms-bin

# Compilar o OAI gNB e o nrUE com suporte a USRP
cd ~/openairinterface5g/cmake_targets
./build_oai -w USRP --ninja --nrUE --gNB --build-lib "nrscope" -C
``` 
### Running OAI CN5G

Start the 5G Core Network using the preconfigured Docker Compose setup:
```bash
cd ~/oai-cn5g
docker compose up -d
``` 
### Running OAI gNB

In this setup, the USRP B210 will be used as the radio interface:

```bash 
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --gNBs.[0].min_rxtxtime 6 -E --continuous-tx
```

### Running OAI nrUE

This process must be executed on a **second computer** (different host from the gNB), running **Ubuntu 24.04 LTS** and connected to its own **USRP B210**.
```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ue-fo-compensation -E --uicc0.imsi 001010000000001
```
### Connection Verification

After successfully starting the CN5G, gNB, and UE, you can verify end-to-end connectivity through ping tests.

## Test 1 — Communication Between UE and CN5G

On the UE host, run:
```bash
ping 192.168.70.135 -I oaitun_ue1
```

## Test 2 — Internet Access (Google)

Once communication with the core is confirmed, test external connectivity:
```bash
ping google.com -I oaitun_ue1
```
For more details, check the official documentation:
👉 [https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#5-oai-ue](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_nrUE.md#5-oai-ue)
