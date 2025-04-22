Here's a refined and organized GitHub README template with copy-paste friendly code blocks:

```markdown
# Google Cloud HA VPN Configuration

This repository demonstrates how to configure High Availability (HA) VPN in Google Cloud Platform (GCP). HA VPN provides a 99.99% SLA for secure connections between on-premises networks and GCP VPC networks.

## Project Overview

This project simulates a real-world HA VPN setup between:
- A global GCP VPC network (`vpc-demo`)
- A simulated on-premises environment (`on-prem`)

Key features demonstrated:
- HA VPN gateways with redundant interfaces
- BGP dynamic routing configuration
- Cross-region connectivity
- Automatic failover testing

## Prerequisites

- Google Cloud account with billing enabled
- `gcloud` CLI installed and authenticated
- Project with Compute Engine API enabled
- Basic networking knowledge (VPC, VPN, BGP concepts)

## Quick Start Deployment

Copy and paste these commands to deploy the full solution:

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

## Detailed Implementation Steps

### 1. Network Infrastructure Setup

```bash
# Create VPC networks and subnets
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

### 2. HA VPN Gateway Configuration

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

### 3. VPN Tunnel Creation

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

### 4. BGP Peering Setup

```bash
# Configure BGP for vpc-demo tunnels
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

## Verification Steps

```bash
# Check tunnel status
gcloud compute vpn-tunnels list

# Test connectivity
gcloud compute ssh on-prem-instance1 --zone us-east4-c \
  --command "ping -c 4 10.1.1.2"

# Verify BGP routes
gcloud compute routers get-status vpc-demo-router1 --region us-east4
```

## Documentation

- [VPN Setup Guide](documentation/vpn-setup-guide.md)
- [BGP Configuration](documentation/bgp-configuration.md)
- [Testing & Troubleshooting](documentation/testing-troubleshooting.md)

## References

- [Google Cloud HA VPN Documentation](https://cloud.google.com/network-connectivity/docs/vpn)
- [Cloud Router Documentation](https://cloud.google.com/router/docs)
- [BGP Best Practices](https://cloud.google.com/network-connectivity/docs/router/concepts/bgp)

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.
```

Key improvements made:
1. Organized into clear sections with logical flow
2. Added copy-paste friendly code blocks for easy deployment
3. Standardized command formatting
4. Included verification steps
5. Added links to documentation
6. Simplified prerequisite section
7. Made the quick start section more prominent

You can copy this entire markdown content directly into your GitHub README.md file. The code blocks will render properly on GitHub.
