# GCP Cloud VPC

## Overview

Google Cloud VPC (Virtual Private Cloud) provides a managed networking functionality for your Cloud Platform resources. VPC networks connect your Google Cloud resources to each other and to the internet.

## Key Concepts

- **VPC Network**: A global resource that consists of a list of regional virtual subnetworks (subnets) in data centers, all connected by a global wide area network (WAN).
- **Subnets**: Regional resources. Each subnet defines a range of IP addresses.
- **Firewall Rules**: Control traffic to and from your VM instances.
- **Routes**: Define paths for traffic leaving a VPC network.
- **VPC Peering**: Connects two VPC networks so that workloads in different VPC networks can communicate internally.
- **Cloud VPN / Cloud Interconnect**: Connects your on-premises network to your GCP VPC.

## VPC Network Types

| Type | Description |
|------|-------------|
| Auto mode | Automatically creates subnets in each region with predefined IP ranges |
| Custom mode | Gives full control over subnet creation and IP ranges |

## Creating a VPC Network

### Using gcloud CLI

```bash
# Create a custom VPC network
gcloud compute networks create my-vpc \
    --subnet-mode=custom \
    --bgp-routing-mode=regional

# Add a subnet to the VPC
gcloud compute networks subnets create my-subnet \
    --network=my-vpc \
    --region=us-central1 \
    --range=10.0.0.0/24
```

### Using the Console

1. Navigate to **VPC network** in the Google Cloud Console.
2. Click **Create VPC Network**.
3. Provide a name and select **Custom** subnet mode.
4. Add subnets with the desired region and IP range.
5. Click **Create**.

## Firewall Rules

```bash
# Allow SSH (port 22) from anywhere
gcloud compute firewall-rules create allow-ssh \
    --network=my-vpc \
    --allow=tcp:22 \
    --source-ranges=0.0.0.0/0

# Allow internal traffic between VMs on the same network
gcloud compute firewall-rules create allow-internal \
    --network=my-vpc \
    --allow=tcp,udp,icmp \
    --source-ranges=10.0.0.0/24
```

## VPC Peering

```bash
# Create a peering connection from network-a to network-b
gcloud compute networks peerings create peer-a-to-b \
    --network=network-a \
    --peer-project=my-project \
    --peer-network=network-b

# Create the reverse peering connection
gcloud compute networks peerings create peer-b-to-a \
    --network=network-b \
    --peer-project=my-project \
    --peer-network=network-a
```

## Best Practices

- Use **custom mode** VPC networks for production environments to maintain full control over IP ranges.
- Apply **least-privilege firewall rules** — only open ports that are required.
- Use **Private Google Access** to allow VMs without external IPs to reach Google APIs.
- Enable **VPC Flow Logs** for network monitoring and security analysis.
- Use **Shared VPC** to centrally manage network resources across multiple projects.

## Video Tutorial

<iframe width="560" height="315" src="https://www.youtube.com/embed/cF2lfLlfdko?si=RuP8kUft7pjQtbga" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Further Reading

- [GCP VPC Documentation](https://cloud.google.com/vpc/docs)
- [VPC Network Overview](https://cloud.google.com/vpc/docs/vpc)
- [Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls)
- [VPC Peering](https://cloud.google.com/vpc/docs/vpc-peering)
