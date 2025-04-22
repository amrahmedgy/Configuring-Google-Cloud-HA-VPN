# 🌐 Google Cloud HA VPN Configuration

This repository demonstrates how to configure **High Availability (HA) VPN** on **Google Cloud Platform (GCP)**. HA VPN offers a 99.99% SLA for secure connectivity between on-premises environments and GCP VPC networks.

---

## 📌 Project Overview

This project simulates a real-world HA VPN setup between:

- A GCP VPC network (`vpc-demo`)
- A simulated on-premises environment (`on-prem`)

**Key Features:**

- Redundant HA VPN gateways
- BGP dynamic routing setup
- Cross-region connectivity
- Automatic failover validation
  
  ![Capture2](https://github.com/user-attachments/assets/89b0d7f2-4336-45f2-8ef7-b29bec18697e)


---

## ✅ Prerequisites

Before deploying, ensure you have:

- A **Google Cloud project** with billing enabled
- `gcloud` CLI installed and authenticated
- Compute Engine API enabled
- Basic knowledge of **VPC**, **VPN**, and **BGP**

---

## 🚀 Quick Start Deployment

To deploy the solution, run the following commands:

```bash
# Make scripts executable
chmod +x scripts/*.sh

# Run deployment scripts in order
./scripts/01-create-vpc-networks.sh
./scripts/02-create-onprem-network.sh
./scripts/03-create-ha-vpn-gateways.sh
./scripts/04-create-vpn-tunnels.sh
./scripts/05-create-bgp-peering.sh
./scripts/06-verify-configuration.sh
```

---

## 🔧 Detailed Implementation Steps

### 1. 🏗️ Network Infrastructure Setup

```bash
# Create VPC networks
gcloud compute networks create vpc-demo --subnet-mode custom
gcloud compute networks subnets create vpc-demo-subnet1 \
  --network vpc-demo --range 10.1.1.0/24 --region us-east4
gcloud compute networks subnets create vpc-demo-subnet2 \
  --network vpc-demo --range 10.2.1.0/24 --region us-east1

# Create on-premises network
gcloud compute networks create on-prem --subnet-mode custom
gcloud compute networks subnets create on-prem-subnet1 \
  --network on-prem --range 192.168.1.0/24 --region us-east4
```

---

### 2. 🌐 HA VPN Gateway Configuration

```bash
# Create HA VPN gateways
gcloud compute vpn-gateways create vpc-demo-vpn-gw1 \
  --network vpc-demo --region us-east4
gcloud compute vpn-gateways create on-prem-vpn-gw1 \
  --network on-prem --region us-east4

# Create Cloud Routers
gcloud compute routers create vpc-demo-router1 \
  --region us-east4 --network vpc-demo --asn 65001
gcloud compute routers create on-prem-router1 \
  --region us-east4 --network on-prem --asn 65002
```

---

### 3. 🔁 VPN Tunnel Creation

```bash
# Create tunnels for vpc-demo
gcloud compute vpn-tunnels create vpc-demo-tunnel0 \
  --peer-gcp-gateway on-prem-vpn-gw1 --region us-east4 \
  --ike-version 2 --shared-secret [YOUR_SECRET] \
  --router vpc-demo-router1 --vpn-gateway vpc-demo-vpn-gw1 --interface 0

gcloud compute vpn-tunnels create vpc-demo-tunnel1 \
  --peer-gcp-gateway on-prem-vpn-gw1 --region us-east4 \
  --ike-version 2 --shared-secret [YOUR_SECRET] \
  --router vpc-demo-router1 --vpn-gateway vpc-demo-vpn-gw1 --interface 1
```

---

### 4. 🔄 BGP Peering Setup

```bash
# Configure BGP interface and peer for tunnel0
gcloud compute routers add-interface vpc-demo-router1 \
  --interface-name if-tunnel0-to-on-prem \
  --ip-address 169.254.0.1 --mask-length 30 \
  --vpn-tunnel vpc-demo-tunnel0 --region us-east4

gcloud compute routers add-bgp-peer vpc-demo-router1 \
  --peer-name bgp-on-prem-tunnel0 \
  --interface if-tunnel0-to-on-prem \
  --peer-ip-address 169.254.0.2 --peer-asn 65002 \
  --region us-east4
```

---

## 🔍 Verification & Testing

```bash
# Check tunnel status
gcloud compute vpn-tunnels list

# SSH to test instance (replace with your instance name)
gcloud compute ssh on-prem-instance1 --zone us-east4-c \
  --command "ping -c 4 10.1.1.2"

# Check Cloud Router status
gcloud compute routers get-status vpc-demo-router1 --region us-east4
```

---

## 📚 Documentation

- [VPN Setup Guide](documentation/vpn-setup-guide.md)
- [BGP Configuration](documentation/bgp-configuration.md)
- [Testing & Troubleshooting](documentation/testing-troubleshooting.md)

---

## 📖 References

- [Google Cloud HA VPN Docs](https://cloud.google.com/network-connectivity/docs/vpn)
- [Cloud Router Guide](https://cloud.google.com/router/docs)
- [BGP Best Practices](https://cloud.google.com/network-connectivity/docs/router/concepts/bgp)

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.
