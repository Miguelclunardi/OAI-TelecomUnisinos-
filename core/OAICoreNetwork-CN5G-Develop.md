# Tutorial: Configuring a 5G Core with OpenAirInterface (OAI)

This tutorial describes how to configure and run an *OAI* 5G Core.  

---

## Minimum Hardware Requirements

To run **OAI CN5G** and **OAI gNB**, the following setup is recommended:

- **Operating System:** Ubuntu 24.04 LTS  
- **CPU:** 8 cores x86_64 @ 3.5 GHz  
- **RAM:** 32 GB  
- **Hardware:** Laptop, desktop, or server  

---

## 1. OAI CN5G

### 1.1 OAI CN5G Prerequisites

Install the required packages:

```bash
sudo apt install -y git net-tools putty

# Official Docker repository
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker packages
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add the current user to the docker group
sudo usermod -a -G docker $(whoami)

reboot
```
### OAI 1.2 CN5G Configuration Files

```bash
wget -O ~/oai-cn5g.zip https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g
unzip ~/oai-cn5g.zip
mv ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
```
### 1.3 Download OAI CN5G Docker Images

```bash
cd ~/oai-cn5g
docker compose up -d
```
---

## 2.0 Running OAI CN5G

### 2.1 Start OAI CN5G
```bash
cd ~/oai-cn5g
docker compose pull
```

### 2.2 Stop OAI CN5G
```bash
cd ~/oai-cn5g
docker compose down
```
---
## 3. Testing OAI CN5G

### 3.1 Check running containers:
```bash
sudo docker ps
```

### 3.2 Check logs:
```bash
sudo docker logs (nome do container) -f
```
Exemplo: 
```bash
sudo docker logs oai-amf -f
```
