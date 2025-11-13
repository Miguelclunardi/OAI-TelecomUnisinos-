# SIM/UICC Programming (Sysmocom + Osmocom / pysim)

**Objective:** this guide explains how to program a reprogrammable SIM/UICC card (e.g., Sysmocom-compatible card) using the Osmocom community tools (`pysim`).  
---

## Introduction

The cards used in this scenario are reprogrammable cards compatible with Osmocom tools. We use the **pysim** package to flash new identities — IMSI, ICCID, keys (K), OPc/OP, SPN, and other parameters — into the UICC/SIM card that will be used by the COTS UE (5G phone).

This document covers the entire process from environment setup to the programming command and basic verification. The examples below assume an Ubuntu environment and a smartcard reader connected to the PC.

---

## Important Warnings

- **Sensitive data:** IMSI, K, OP/OPc, and ICCID are sensitive values. **DO NOT** publish them in public repositories or logs.  
- **Risk:** programming cards may render them unusable if parameters are incorrect. Perform tests on cards intended for development.

---

## Prerequisites

- Computer running **Ubuntu 22.04/24.04 LTS** (or equivalent Debian-based).  
- USB smartcard reader compatible with PC/SC (e.g., ACS ACR122U or similar).  
- Reprogrammable SIM/UICC card (e.g., Sysmocom-compatible card).  
- Root/sudo access to install packages and operate the reader.  
- Internet connection to download packages and clone repositories.

---

## Environment Installation (pysim + dependencies)

Update system packages and install the basic dependencies:

```bash
sudo apt update
sudo apt install -y git pcscd libpcsclite-dev python3 python3-setuptools python3-pip python3-pyscard

git clone https://github.com/osmocom/pysim
cd pysim
sudo apt-get install --no-install-recommends \
    pcscd libpcsclite-dev \
    python3 \
    python3-setuptools \
    python3-pyscard \
    python3-pip
pip3 install -r requirements.txt

```
## Programming Command — Example

Below is an example of the command used to program the card with the `pySim-prog.py` utility. Adjust the parameters (IMSI, ICCID, K, OPc, MCC, MNC, SPN, etc.) according to your case.
```bash
./pySim-prog.py \
  -p 0 \
  -n "TelecomUNISINOS" \
  -a 1234567 \
  -s 8988211000000280969 \
  --mcc=001 \
  --mnc=01 \
  --imsi=001010000000001 \
  -k fec86ba6eb707ed08905757b1bb44b8f \
  --opc=C42449363BBAD02B66D16BC975D77CC1
```
### Meaning of Parameters (Summary)

- `-p 0` — port / reader index (e.g., reader 0).  
- `-n "TelecomUNISINOS"` — **SPN** (*Service Provider Name*) — operator name.  
- `-a 1234567` — administrative parameter used by the utility (ADM).  
- `-s 8988211000000280969` — **ICCID** of the card (card identifier).  
- `--mcc=001` — **MCC** (*Mobile Country Code*).  
- `--mnc=01` — **MNC** (*Mobile Network Code*).  
- `--imsi=001010000000001` — **IMSI** (subscriber identity).  
- `-k <K>` — **K** (subscriber key / *Ki*) used for authentication.  
- `--opc=<OPc>` — **OPc** (or `--op`, depending on the tool) — parameter related to the authentication scheme.  

> **Note:** Depending on the version of `pysim` you are using, the flag for OPc may be `--opc` or `--op`. Check with `./pySim-prog.py --help` to confirm.

