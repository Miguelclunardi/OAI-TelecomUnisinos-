# Deployment Tutorial — OpenAirInterface 5G Core Network with UERANSIM

This repository provides instructions for configuring, building, and running the **OpenAirInterface (OAI) 5G Core Network** integrated with **UERANSIM**, aimed at testing connectivity, performance, and network validation.  
This tutorial is recommended for studying the OAI 5G Core and understanding its fundamentals.

---

## 1. Requirements

Before starting, make sure you have completed the **Basic Deployment** of the OAI Core version 1.5:

* [OAI Core Network V1.5](../core/OAI Core Network-V1.5.md)

---

## 2. OAI Core Configuration

### 2.1 AMF Parameters

Access the main configuration file:

```bash
cd oai-cn5g-fed/docker-compose/docker-compose-basic-vpp-nrf.yaml
```

Add the following parameters under the oai-amf block:

```yaml
- INT_ALGO_LIST=["NIA1","NIA2"]
- CIPH_ALGO_LIST=["NEA1","NEA2"]
```

These parameters define the integrity and encryption algorithms allowed for secure communication between network functions.

---

## 3. UERANSIM Installation and Build

Run the following commands:

```bash
git clone -b docker_support https://github.com/orion-belt/UERANSIM.git
cd UERANSIM
docker build --target ueransim --tag ueransim:latest -f docker/Dockerfile.ubuntu.18.04 .
```

---

## 4. Starting the OAI Core

To start: 

```bash
python3 core-network.py --type start-basic-vpp
```

### 4.1 Common Error Fix

**Caso ocorra o erro:**

```
SMF not receiving heartbeats from UPF...
```

Run:

```bash
docker restart oai-smf
python3 core-network.py --type start-basic-vpp
```

### 4.2 Stopping the Core

```bash
python3 core-network.py --type stop-basic-vpp
```

---

## 5. **Inicialização do UERANSIM**

To start:

```bash
docker-compose -f docker-compose-ueransim-vpp.yaml up -d
```

### 5.1 Stopping UERANSIM

```bash
docker-compose -f docker-compose-ueransim-vpp.yaml down
```

---

## 6. Functionality Check

### 6.1 List Active Containers

```bash
docker ps
```

### 6.2 Check UERANSIM Logs

```bash
docker logs ueransim
```

At the end of the log, you should see something similar to:

```
[2025-08-26 19:00:20.176] [nas] [info] PDU Session establishment is successful PSI[1]
[2025-08-26 19:00:20.186] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 12.1.1.3] is up.
```

> **Note:** Write down the IP (`12.1.1.x`) as it will be needed for connectivity testing
---

## 7. Connectivity Tests

### 7.1 UE → Internet Test

Command:

```bash
docker exec ueransim ping -c 3 -I uesimtun0 google.com
```
Successful output example:

```
PING google.com (172.217.18.238) from 12.2.1.2 : 56(84) bytes of data.
64 bytes from par10s10-in-f238.1e100.net (172.217.18.238): icmp_seq=1 ttl=115 time=5.12 ms
64 bytes from par10s10-in-f238.1e100.net (172.217.18.238): icmp_seq=2 ttl=115 time=7.52 ms
64 bytes from par10s10-in-f238.1e100.net (172.217.18.238): icmp_seq=3 ttl=115 time=7.19 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 4ms
rtt min/avg/max/mdev = 5.119/6.606/7.515/1.064 ms
```

### 7.2 Internet → UE Test

> **Note:** Make sure to use the IP shown in the container logs, as it may change each time the system starts.

Command:

```bash
docker exec -it oai-ext-dn ping -c 3 12.2.1.2
```

Successful output example:

```
PING 12.2.1.2 (12.2.1.2) 56(84) bytes of data.
64 bytes from 12.2.1.2: icmp_seq=1 ttl=64 time=0.235 ms
64 bytes from 12.2.1.2: icmp_seq=2 ttl=64 time=0.145 ms
64 bytes from 12.2.1.2: icmp_seq=3 ttl=64 time=0.448 ms

--- 12.1.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2036ms
rtt min/avg/max/mdev = 0.145/0.276/0.448/0.127 ms
```

### 7.3 Advanced Testing

For further testing (speed, stress, etc.), see:

[Documentação Oficial — DEPLOY\_SA5G\_WITH\_UERANSIM](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/v1.5.0/docs/DEPLOY_SA5G_WITH_UERANSIM.md?ref_type=tags)
