# APN Configuration Tutorial for the UE (COTS 5G)

This tutorial describes the process of configuring the **APN (Access Point Name)** on the **5G phone (COTS UE)** that will be used to connect to the private 5G network implemented with **OAI CN5G** and **OAI gNB**.

---

## 1. Requirements

- A 5G phone compatible with SA (Standalone) networks.  
- A **Sysmocom** SIM card previously programmed (as described in [TutorialSIM_UECots.md](TutorialSIM_UECots.md)).  
- An application that allows **locking the phone to 5G**, such as *Force LTE Only* or *5G Only Mode*.  

---

## 2. Phone Configuration

1. Insert the Sysmocom SIM card into the 5G phone.  
2. Enable **roaming mode**, since the MCC/MNC codes used (001/01) do not belong to a commercial operator.  
3. Open the menu **Mobile Network Settings → Access Point Names (APN)**.  
4. Create a new APN with the following parameters:

| Field | Value |
|-------|-------|
| **Name** | oai |
| **APN** | oai |
| **MCC** | 001 |
| **MNC** | 01 |
| **APN Type** | default,mms,supi,hipri,fota,cbs,xcap |
| **APN Protocol** | IPv4 |
| **Roaming Protocol** | IPv4 |
| **Other fields** | Leave blank |

> ⚠️ **Tip:**  
> If the phone does not allow editing all APN types, keep at least `default` and `hipri`.

---

## 3. Reference to the DNNs configured in the OAI Core

The **DNNs (Data Network Names)** defined in the 5G core are used to configure the phone’s APNs.  
This information can be checked directly in the OAI Core configuration file:
```bash
oai-cn5g/conf/config.yaml
