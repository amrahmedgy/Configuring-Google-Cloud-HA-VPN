# Google Cloud HA VPN Configuration

This repository demonstrates how to configure High Availability (HA) VPN in Google Cloud Platform. HA VPN is a regional service that provides a 99.99% SLA for establishing secure, private connections between your on-premises network and your Google Cloud VPC network.

![HA VPN Architecture](architecture/ha-vpn-architecture.svg)
<svg viewBox="0 0 800 500" xmlns="http://www.w3.org/2000/svg">
  <!-- Background -->
  <rect width="800" height="500" fill="#f5f5f5"/>
  
  <!-- VPC Demo Network -->
  <rect x="50" y="40" width="300" height="420" rx="10" fill="#e1f5fe" stroke="#0288d1" stroke-width="2"/>
  <text x="200" y="70" font-family="Arial" font-size="18" text-anchor="middle" font-weight="bold">VPC Demo Network</text>
  
  <!-- VPC Demo Subnets -->
  <rect x="70" y="90" width="260" height="100" rx="5" fill="#b3e5fc" stroke="#0288d1" stroke-width="1"/>
  <text x="200" y="110" font-family="Arial" font-size="14" text-anchor="middle">vpc-demo-subnet1</text>
  <text x="200" y="130" font-family="Arial" font-size="12" text-anchor="middle">us-east4 (10.1.1.0/24)</text>
  
  <rect x="70" y="200" width="260" height="100" rx="5" fill="#b3e5fc" stroke="#0288d1" stroke-width="1"/>
  <text x="200" y="220" font-family="Arial" font-size="14" text-anchor="middle">vpc-demo-subnet2</text>
  <text x="200" y="240" font-family="Arial" font-size="12" text-anchor="middle">us-east1 (10.2.1.0/24)</text>
  
  <!-- VPC Demo Instances -->
  <rect x="120" y="140" width="160" height="40" rx="5" fill="#ffffff" stroke="#0288d1" stroke-width="1"/>
  <text x="200" y="165" font-family="Arial" font-size="12" text-anchor="middle">vpc-demo-instance1</text>
  
  <rect x="120" y="250" width="160" height="40" rx="5" fill="#ffffff" stroke="#0288d1" stroke-width="1"/>
  <text x="200" y="275" font-family="Arial" font-size="12" text-anchor="middle">vpc-demo-instance2</text>
  
  <!-- VPC Demo Router -->
  <rect x="120" y="320" width="160" height="40" rx="5" fill="#ffecb3" stroke="#ffa000" stroke-width="1"/>
  <text x="200" y="345" font-family="Arial" font-size="12" text-anchor="middle">vpc-demo-router1 (ASN 65001)</text>
  
  <!-- VPC Demo VPN Gateway -->
  <rect x="120" y="380" width="160" height="60" rx="5" fill="#dcedc8" stroke="#689f38" stroke-width="2"/>
  <text x="200" y="405" font-family="Arial" font-size="14" text-anchor="middle" font-weight="bold">vpc-demo-vpn-gw1</text>
  <text x="200" y="425" font-family="Arial" font-size="10" text-anchor="middle">Interface 0 & Interface 1</text>
  
  <!-- On-Prem Network -->
  <rect x="450" y="40" width="300" height="420" rx="10" fill="#f3e5f5" stroke="#7b1fa2" stroke-width="2"/>
  <text x="600" y="70" font-family="Arial" font-size="18" text-anchor="middle" font-weight="bold">On-Prem Network</text>
  
  <!-- On-Prem Subnet -->
  <rect x="470" y="90" width="260" height="100" rx="5" fill="#e1bee7" stroke="#7b1fa2" stroke-width="1"/>
  <text x="600" y="110" font-family="Arial" font-size="14" text-anchor="middle">on-prem-subnet1</text>
  <text x="600" y="130" font-family="Arial" font-size="12" text-anchor="middle">us-east4 (192.168.1.0/24)</text>
  
  <!-- On-Prem Instance -->
  <rect x="520" y="140" width="160" height="40" rx="5" fill="#ffffff" stroke="#7b1fa2" stroke-width="1"/>
  <text x="600" y="165" font-family="Arial" font-size="12" text-anchor="middle">on-prem-instance1</text>
  
  <!-- On-Prem Router -->
  <rect x="520" y="320" width="160" height="40" rx="5" fill="#ffecb3" stroke="#ffa000" stroke-width="1"/>
  <text x="600" y="345" font-family="Arial" font-size="12" text-anchor="middle">on-prem-router1 (ASN 65002)</text>
  
  <!-- On-Prem VPN Gateway -->
  <rect x="520" y="380" width="160" height="60" rx="5" fill="#dcedc8" stroke="#689f38" stroke-width="2"/>
  <text x="600" y="405" font-family="Arial" font-size="14" text-anchor="middle" font-weight="bold">on-prem-vpn-gw1</text>
  <text x="600" y="425" font-family="Arial" font-size="10" text-anchor="middle">Interface 0 & Interface 1</text>
  
  <!-- VPN Tunnels -->
  <path d="M 200 440 C 200 470, 600 470, 600 440" stroke="#689f38" stroke-width="3" fill="none" stroke-dasharray="5,5"/>
  <path d="M 180 440 C 180 490, 620 490, 620 440" stroke="#689f38" stroke-width="3" fill="none" stroke-dasharray="5,5"/>
  
  <text x="290" y="465" font-family="Arial" font-size="12" text-anchor="middle">Tunnel 0 (BGP 169.254.0.1/30)</text>
  <text x="510" y="485" font-family="Arial" font-size="12" text-anchor="middle">Tunnel 1 (BGP 169.254.1.1/30)</text>
  
  <!-- Global Routing Mode Label -->
  <rect x="70" y="20" width="180" height="20" rx="5" fill="#fff9c4" stroke="#fbc02d" stroke-width="1"/>
  <text x="160" y="35" font-family="Arial" font-size="12" text-anchor="middle">BGP Routing Mode: GLOBAL</text>
  
  <!-- Title -->
  <text x="400" y="20" font-family="Arial" font-size="20" text-anchor="middle" font-weight="bold">Google Cloud HA VPN Architecture</text>
</svg>

## Project Overview

This project simulates a real-world scenario where we:

1. Create a global VPC network (`vpc-demo`) with subnets in multiple regions
2. Set up a simulated on-premises environment using another VPC (`on-prem`)
3. Configure HA VPN gateways in both networks
4. Establish secure tunnels between the networks using BGP for dynamic routing
5. Configure and verify BGP peering for high availability
6. Test connectivity with proper failover

## Prerequisites

- Google Cloud Platform account
- `gcloud` CLI installed and configured
- Basic understanding of networking concepts
- Permissions to create VPC networks, VPN tunnels, and compute instances

## Implementation Steps

### 1. Set Up Network Infrastructure

The scripts in this repository create:
- A global VPC with two custom subnets in different regions
- A simulated on-premises network with its own subnet
- Compute Engine instances in both networks
- Firewall rules to allow traffic between networks

### 2. Configure HA VPN Gateways

We set up HA VPN gateways in both networks with:
- Two interfaces per gateway, each with its own public IP address
- Cloud Routers with BGP configuration
- Multiple VPN tunnels for redundancy

### 3. Establish BGP Peering

BGP peering is configured for dynamic route exchange between networks:
- Custom autonomous system numbers (ASNs) for each network
- Interface configuration for each tunnel
- BGP peering with appropriate IP addresses

### 4. Test High Availability

The setup demonstrates:
- Successful private connectivity between networks
- Automatic failover when a tunnel goes down
- Global routing mode for cross-region communication

## Usage

Each script in the `scripts/` directory can be run sequentially to deploy the complete solution:

```bash
# Make scripts executable
chmod +x scripts/*.sh

# Run scripts in sequence
./scripts/01-create-vpc-networks.sh
./scripts/02-create-onprem-network.sh 
./scripts/03-create-ha-vpn-gateways.sh
./scripts/04-create-vpn-tunnels.sh
./scripts/05-create-bgp-peering.sh
./scripts/06-verify-configuration.sh

Documentation
Detailed documentation is available in the documentation/ directory:

VPN Setup Guide - Detailed steps for setting up HA VPN
BGP Configuration - Guide to configuring BGP peering
Testing Connectivity - Methods to verify the setup
Troubleshooting - Common issues and solutions

References

Google Cloud HA VPN Documentation
Cloud Router Documentation
BGP Path Selection

License
This project is licensed under the MIT License - see the LICENSE file for details.

## Script Examples

Let's create the first two scripts for your project:

### 01-create-vpc-networks.sh

```bash
#!/bin/bash
# Create VPC Demo Network and Subnets

# Exit on error
set -e

echo "Creating VPC network: vpc-demo"
gcloud compute networks create vpc-demo --subnet-mode custom

echo "Creating subnet vpc-demo-subnet1 in us-east4"
gcloud compute networks subnets create vpc-demo-subnet1 \
  --network vpc-demo \
  --range 10.1.1.0/24 \
  --region "us-east4"

echo "Creating subnet vpc-demo-subnet2 in us-east1"
gcloud compute networks subnets create vpc-demo-subnet2 \
  --network vpc-demo \
  --range 10.2.1.0/24 \
  --region us-east1

echo "Creating firewall rule to allow all custom traffic within the network"
gcloud compute firewall-rules create vpc-demo-allow-custom \
  --network vpc-demo \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 10.0.0.0/8

echo "Creating firewall rule to allow SSH, ICMP traffic from anywhere"
gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
  --network vpc-demo \
  --allow tcp:22,icmp

echo "Creating VM instance vpc-demo-instance1 in zone us-east4-b"
gcloud compute instances create vpc-demo-instance1 \
  --machine-type=e2-medium \
  --zone us-east4-b \
  --subnet vpc-demo-subnet1

echo "Creating VM instance vpc-demo-instance2 in zone us-east1-d"
gcloud compute instances create vpc-demo-instance2 \
  --machine-type=e2-medium \
  --zone us-east1-d \
  --subnet vpc-demo-subnet2

echo "VPC Demo Network setup complete!"
02-create-onprem-network.sh
bash#!/bin/bash
# Create On-premises Simulated Network and Subnet

# Exit on error
set -e

echo "Creating VPC network: on-prem"
gcloud compute networks create on-prem --subnet-mode custom

echo "Creating subnet on-prem-subnet1 in us-east4"
gcloud compute networks subnets create on-prem-subnet1 \
  --network on-prem \
  --range 192.168.1.0/24 \
  --region us-east4

echo "Creating firewall rule to allow all custom traffic within the network"
gcloud compute firewall-rules create on-prem-allow-custom \
  --network on-prem \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 192.168.0.0/16

echo "Creating firewall rule to allow SSH, ICMP traffic from anywhere"
gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
  --network on-prem \
  --allow tcp:22,icmp

# Get available zones in us-east4 (avoiding the zone used for vpc-demo-instance1)
ZONES=$(gcloud compute zones list --filter="name ~ ^us-east4- AND name != us-east4-b" --format="value(name)" | head -n 1)
SELECTED_ZONE=${ZONES[0]}

echo "Creating VM instance on-prem-instance1 in zone $SELECTED_ZONE"
gcloud compute instances create on-prem-instance1 \
  --machine-type=e2-medium \
  --zone $SELECTED_ZONE \
  --subnet on-prem-subnet1

echo "On-premises Network setup complete!"
Documentation Example
Let's create the VPN setup guide document:
vpn-setup-guide.md
markdown# Google Cloud HA VPN Setup Guide

This guide provides detailed steps for setting up High Availability (HA) VPN in Google Cloud.

## Prerequisites

Before you begin, make sure you have:
- A Google Cloud account with appropriate permissions
- Two VPC networks (in this guide, we use `vpc-demo` and `on-prem`)
- Basic understanding of BGP routing
- `gcloud` CLI installed and configured

## Step 1: Create HA VPN Gateways

An HA VPN gateway provides two interfaces for redundancy. Each interface gets its own external IP address.

### Create VPN Gateway in vpc-demo Network

```bash
gcloud compute vpn-gateways create vpc-demo-vpn-gw1 \
  --network vpc-demo \
  --region us-east4
Create VPN Gateway in on-prem Network
bashgcloud compute vpn-gateways create on-prem-vpn-gw1 \
  --network on-prem \
  --region us-east4
Step 2: Create Cloud Routers
Cloud Routers are required for dynamic routing with BGP.
Create Router in vpc-demo Network
bashgcloud compute routers create vpc-demo-router1 \
  --region us-east4 \
  --network vpc-demo \
  --asn 65001
Create Router in on-prem Network
bashgcloud compute routers create on-prem-router1 \
  --region us-east4 \
  --network on-prem \
  --asn 65002
Step 3: Create VPN Tunnels
For HA VPN, you need to create two tunnels between the VPN gateways.
Create Tunnels from vpc-demo to on-prem
bash# Create first tunnel from interface 0
gcloud compute vpn-tunnels create vpc-demo-tunnel0 \
  --peer-gcp-gateway on-prem-vpn-gw1 \
  --region us-east4 \
  --ike-version 2 \
  --shared-secret YOUR_SECRET_HERE \
  --router vpc-demo-router1 \
  --vpn-gateway vpc-demo-vpn-gw1 \
  --interface 0

# Create second tunnel from interface 1
gcloud compute vpn-tunnels create vpc-demo-tunnel1 \
  --peer-gcp-gateway on-prem-vpn-gw1 \
  --region us-east4 \
  --ike-version 2 \
  --shared-secret YOUR_SECRET_HERE \
  --router vpc-demo-router1 \
  --vpn-gateway vpc-demo-vpn-gw1 \
  --interface 1
Create Tunnels from on-prem to vpc-demo
bash# Create first tunnel from interface 0
gcloud compute vpn-tunnels create on-prem-tunnel0 \
  --peer-gcp-gateway vpc-demo-vpn-gw1 \
  --region us-east4 \
  --ike-version 2 \
  --shared-secret YOUR_SECRET_HERE \
  --router on-prem-router1 \
  --vpn-gateway on-prem-vpn-gw1 \
  --interface 0

# Create second tunnel from interface 1
gcloud compute vpn-tunnels create on-prem-tunnel1 \
  --peer-gcp-gateway vpc-demo-vpn-gw1 \
  --region us-east4 \
  --ike-version 2 \
  --shared-secret YOUR_SECRET_HERE \
  --router on-prem-router1 \
  --vpn-gateway on-prem-vpn-gw1 \
  --interface 1
Step 4: Configure BGP Peering
Set up BGP peering for each tunnel.
Configure BGP for vpc-demo Tunnels
bash# Configure for tunnel0
gcloud compute routers add-interface vpc-demo-router1 \
  --interface-name if-tunnel0-to-on-prem \
  --ip-address 169.254.0.1 \
  --mask-length 30 \
  --vpn-tunnel vpc-demo-tunnel0 \
  --region us-east4

gcloud compute routers add-bgp-peer vpc-demo-router1 \
  --peer-name bgp-on-prem-tunnel0 \
  --interface if-tunnel0-to-on-prem \
  --peer-ip-address 169.254.0.2 \
  --peer-asn 65002 \
  --region us-east4

# Configure for tunnel1
gcloud compute routers add-interface vpc-demo-router1 \
  --interface-name if-tunnel1-to-on-prem \
  --ip-address 169.254.1.1 \
  --mask-length 30 \
  --vpn-tunnel vpc-demo-tunnel1 \
  --region us-east4

gcloud compute routers add-bgp-peer vpc-demo-router1 \
  --peer-name bgp-on-prem-tunnel1 \
  --interface if-tunnel1-to-on-prem \
  --peer-ip-address 169.254.1.2 \
  --peer-asn 65002 \
  --region us-east4
Configure BGP for on-prem Tunnels
bash# Configure for tunnel0
gcloud compute routers add-interface on-prem-router1 \
  --interface-name if-tunnel0-to-vpc-demo \
  --ip-address 169.254.0.2 \
  --mask-length 30 \
  --vpn-tunnel on-prem-tunnel0 \
  --region us-east4

gcloud compute routers add-bgp-peer on-prem-router1 \
  --peer-name bgp-vpc-demo-tunnel0 \
  --interface if-tunnel0-to-vpc-demo \
  --peer-ip-address 169.254.0.1 \
  --peer-asn 65001 \
  --region us-east4

# Configure for tunnel1
gcloud compute routers add-interface on-prem-router1 \
  --interface-name if-tunnel1-to-vpc-demo \
  --ip-address 169.254.1.2 \
  --mask-length 30 \
  --vpn-tunnel on-prem-tunnel1 \
  --region us-east4

gcloud compute routers add-bgp-peer on-prem-router1 \
  --peer-name bgp-vpc-demo-tunnel1 \
  --interface if-tunnel1-to-vpc-demo \
  --peer-ip-address 169.254.1.1 \
  --peer-asn 65001 \
  --region us-east4
Step 5: Configure Firewall Rules
Configure firewall rules to allow traffic between the networks.
bash# Allow traffic from on-prem to vpc-demo
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
  --network vpc-demo \
  --allow tcp,udp,icmp \
  --source-ranges 192.168.1.0/24

# Allow traffic from vpc-demo to on-prem
gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
  --network on-prem \
  --allow tcp,udp,icmp \
  --source-ranges 10.1.1.0/24,10.2.1.0/24
Step 6: Configure Global Routing
To enable access across regions, update the vpc-demo network to use global routing mode:
bashgcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
Verification
Verify the status of your VPN tunnels:
bashgcloud compute vpn-tunnels list
gcloud compute vpn-tunnels describe vpc-demo-tunnel0 --region us-east4
gcloud compute vpn-tunnels describe vpc-demo-tunnel1 --region us-east4
Test connectivity by SSH'ing into instances and pinging across networks:
bash# From on-prem-instance1, ping vpc-demo-instance1
ping 10.1.1.2

# From on-prem-instance1, ping vpc-demo-instance2 (in different region)
ping 10.2.1.2
Testing Failover
Test high availability by deleting one tunnel and verifying connectivity still works:
bash# Delete one tunnel
gcloud compute vpn-tunnels delete vpc-demo-tunnel0 --region us-east4

# Verify connectivity still works
# The second tunnel should take over automatically
Next Steps

Monitor tunnel status with Cloud Monitoring
Set up alerting for tunnel status changes
Consider implementing Cloud NAT for outbound connectivity from private instances


## Next Steps for Your GitHub Project

1. **Complete the Scripts**: Finish creating all the scripts in the `scripts/` directory based on the lab steps.

2. **Add Screenshots**: Add screenshots of successful connectivity tests and tunnel status verifications.

3. **Create Additional Documentation**: Complete the other documentation files including BGP configuration, troubleshooting guides, etc.

4. **Add a License File**: Include an open-source license like MIT.

5. **Create Issues and Projects**: Set up GitHub Issues for potential improvements.

This project will effectively showcase your hands-on experience with:
- Google Cloud networking concepts
- HA VPN configuration
- BGP routing
- Infrastructure as Code principles
- Network security practices

Would you like me to help you with any specific part of this GitHub project structure in more detail?RetryClaude does not have the ability to run the code it generates yet.Claude can make mistakes. Please double-check responses.
