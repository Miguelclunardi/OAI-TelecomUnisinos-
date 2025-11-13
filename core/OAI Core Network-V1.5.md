# OAI Core Network Detailed Setup and Deployment Guide

This repository provides configurations and scripts for setting up and basic-deploying the OpenAirInterface (OAI) 5G Core Network. This guide will walk you through the necessary steps for configuration and deployment, and explain the Docker Compose file used for the core network services.

## 1. Set Up

For this setup, OAI Core Network version `1.5.0` was used.

### 1.1 Core-host Configurations

Before deploying the core network, ensure your core-host machine is configured to allow IP forwarding. This is crucial for network traffic to flow correctly through the Dockerized core network components.
```shell
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
````

  * `sudo sysctl net.ipv4.conf.all.forwarding=1`: Enables IP forwarding on all network interfaces. This allows the host to forward packets between different networks, which is essential for the core network's operation.
  * `sudo iptables -P FORWARD ACCEPT`: Sets the default policy for the FORWARD chain in iptables to ACCEPT. This allows packets to be forwarded by default, preventing them from being dropped by the firewall.

### 1.2 Clone `oai-cn5g-fed`

Clone the official `oai-cn5g-fed` repository and switch to the specified version. This repository contains the necessary Docker Compose files and scripts for deploying the OAI 5G Core Network.

```shell
git clone [https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git)
git checkout v1.5.0
```

### 1.3 gNB-host Configurations

If you are using a separate gNB (Next-Generation NodeB) host, you must configure a static route to the internal Docker network of the OAI Core Network. This enables the gNB to establish a connection with the core network components.

```shell
sudo ip route add route 192.168.70.128/26 via {ip_addr_corehost} dev {interface}

#example
sudo ip route add route 192.168.70.128/26 via 191.4.205.38 dev br01
```

  * `192.168.70.128/26`: This is the subnet used by the `public_net` Docker network, as defined in `docker-compose-basic-nrf.yaml`.
  * `{ip_addr_corehost}`: Replace with the actual IP address of your core-host machine.
  * `{interface}`: Replace with the network interface on the gNB-host that connects to the core-host network.

## 2\. Deploying the Core Network

Navigate to the `docker-compose` directory within your `oai-cn5g-fed` clone and use the `core-networks.py` script to manage the core network deployment.

  * To start the core:
    ```shell
    cd path-to/oai-cn5g-fed/docker-compose
    python3 core-network.py --type start-basic --scenario 1
    ```
  * To stop the core:
    ```shell
    python3 core-network.py --type stop-basic --scenario 1
    ```

## 3\. Understanding the Docker Compose Configuration (`docker-compose-basic-nrf.yaml`)

The `docker-compose-basic-nrf.yaml` file defines the services, networks, and configurations for the OAI 5G Core Network deployment using Docker.

  * **`version: '3.8'`**: Specifies the Docker Compose file format version.

  * **`networks`**: Defines the Docker networks used by the services.

      * **`public_net`**: This is a bridge network named `oai-cn5g`.
          * `subnet: 192.168.70.128/26`: Specifies the IP address range for this network. This is critical for configuring routes on the gNB host.
          * `com.docker.network.bridge.name: "oai-cn5g"`: Assigns a specific name to the underlying bridge interface created by Docker.

  * **`services`**: Defines the individual containers that make up the 5G Core Network. Each service typically runs a specific 5G Network Function (NF).

      * **`mysql`**:

          * **Purpose**: Acts as the database for OAI Core Network components, storing subscriber information and other essential data.
          * **Image**: `mysql:8.0`.
          * **Volumes**: Mounts `oai_db2.sql` to initialize the database and a healthcheck script.
          * **Environment Variables**: Sets `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, and `MYSQL_ROOT_PASSWORD` for database access.
          * **IP Address**: `192.168.70.131` on `public_net`.

      * **`oai-nrf` (Network Repository Function)**:

          * **Purpose**: Facilitates the discovery of network functions (NFs) within the 5G Core Network. Other NFs register with the NRF and discover each other through it.
          * **Image**: `oaisoftwarealliance/oai-nrf:v1.5.0`.
          * **IP Address**: `192.168.70.130` on `public_net`.

      * **`oai-udr` (User Data Repository)**:

          * **Purpose**: Stores subscriber-related data, including subscription information, authentication data, and policy data.
          * **Image**: `oaisoftwarealliance/oai-udr:v1.5.0`.
          * **Dependencies**: Depends on `mysql` and `oai-nrf`.
          * **Environment Variables**: Configured to connect to the MySQL database and register with the NRF.
          * **IP Address**: `192.168.70.136` on `public_net`.

      * **`oai-udm` (User Data Management)**:

          * **Purpose**: Manages user subscription data and handles authentication credentials generation. It interfaces with the UDR.
          * **Image**: `oaisoftwarealliance/oai-udm:v1.5.0`.
          * **Dependencies**: Depends on `oai-udr`.
          * **Environment Variables**: Configured to connect to `oai-udr` and `oai-nrf`.
          * **IP Address**: `192.168.70.137` on `public_net`.

      * **`oai-ausf` (Authentication Server Function)**:

          * **Purpose**: Handles authentication and authorization of user equipment (UE) accessing the 5G network. It interacts with the UDM for subscriber credentials.
          * **Image**: `oaisoftwarealliance/oai-ausf:v1.5.0`.
          * **Dependencies**: Depends on `oai-udm`.
          * **Environment Variables**: Configured to connect to `oai-udm` and `oai-nrf`.
          * **IP Address**: `192.168.70.138` on `public_net`.

      * **`oai-amf` (Access and Mobility Management Function)**:

          * **Purpose**: Manages access control, mobility management, and connection states for UEs. It is the primary contact point for UEs and interacts with various NFs for session management and authentication.
          * **Image**: `oaisoftwarealliance/oai-amf:v1.5.0`.
          * **Dependencies**: Depends on `mysql`, `oai-nrf`, and `oai-ausf`.
          * **Environment Variables**: Configures Mobile Country Code (MCC), Mobile Network Code (MNC), supported Slices (SST/SD), and connections to SMF, NRF, AUSF, and UDM.
          * **IP Address**: `192.168.70.132` on `public_net`.

      * **`oai-smf` (Session Management Function)**:

          * **Purpose**: Manages Packet Flow Descriptions (PFDs), IP address allocation for UEs, and interaction with the User Plane Function (UPF) for traffic routing.
          * **Image**: `oaisoftwarealliance/oai-smf:v1.5.0`.
          * **Dependencies**: Depends on `oai-nrf` and `oai-amf`.
          * **Environment Variables**: Configures DNS servers, connections to AMF, UDM, UPF, and NRF. Also defines Data Network Names (DNNs), Slices (NSSAI), and IP ranges for UE allocation.
          * **IP Address**: `192.168.70.133` on `public_net`.

      * **`oai-spgwu` (User Plane Function - SGW-U/PGW-U)**:

          * **Purpose**: Handles the user plane traffic forwarding between the gNB and the external data network. It is responsible for packet routing, encapsulation, and decapsulation.
          * **Image**: `oaisoftwarealliance/oai-spgwu-tiny:v1.5.0`.
          * **Dependencies**: Depends on `oai-nrf` and `oai-smf`.
          * **Capabilities**: Requires `NET_ADMIN` and `SYS_ADMIN` capabilities for network interface manipulation.
          * **Environment Variables**: Configures network interfaces, UE IP ranges (`12.1.1.0/24`), enables 5G features, and specifies connections to NRF, as well as supported slices and DNNs.
          * **IP Address**: `192.168.70.134` on `public_net`.

      * **`oai-ext-dn` (External Data Network)**:

          * **Purpose**: Simulates an external data network (e.g., the internet) that user traffic is routed to/from. It ensures that traffic from the UE's assigned IP range (`12.1.1.0/24`) is correctly routed via the UPF.
          * **Image**: `oaisoftwarealliance/trf-gen-cn5g:latest`.
          * **Entrypoint**: Sets up a static route to forward traffic destined for the UE IP range (`12.1.1.0/24`) via the `oai-spgwu` (UPF).
          * **IP Address**: `192.168.70.135` on `public_net`.

## 4\. More Information

For more detailed information and advanced deployment scenarios, please refer to the official OAI documentation:

  * [Basic Deployment using Docker Compose](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/master/docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md)
