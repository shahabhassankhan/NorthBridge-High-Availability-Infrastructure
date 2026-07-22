# NorthBridge High Availability Infrastructure

A two-tier, highly available web deployment on Microsoft Azure. Two Windows Server VMs sit behind a Load Balancer, so the site stays up if one server goes down.

**Author:** Shahab Hassan Khan

---

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Azure Services Used](#azure-services-used)
- [Infrastructure Design](#infrastructure-design)
- [Virtual Machines](#virtual-machines)
- [Load Balancer Configuration](#load-balancer-configuration)
- [High Availability Validation](#high-availability-validation)
- [Public IP Design Decision](#public-ip-design-decision)
- [Architecture](#architecture)
- [Skills Demonstrated](#skills-demonstrated)
- [Learning Outcomes](#learning-outcomes)
- [Screenshots](#screenshots)

---

## Overview

This project covers the design, deployment, and testing of a highly available two-tier web infrastructure on Azure. The goal was a hosting setup where the site stays reachable even if one of the web servers goes offline.

The build uses two Windows Server virtual machines running IIS, an Azure Standard Load Balancer, an Availability Set, a virtual network with scoped NSG rules, health probes, and load balancing rules.

## Objectives

- Design a highly available web infrastructure
- Deploy multiple Windows Server virtual machines
- Configure Azure Load Balancer
- Set up health probes
- Configure load balancing rules
- Demonstrate automatic failover
- Apply Azure networking best practices
- Reduce the public attack surface after deployment

## Azure Services Used

- Resource Groups
- Virtual Network (VNet) & Subnets
- Network Security Groups (NSGs)
- Windows Server 2016 Datacenter Virtual Machines
- Availability Set
- Azure Standard Load Balancer
- Public IP Addresses
- Backend Pools
- Health Probes
- Load Balancing Rules
- IIS Web Server

## Infrastructure Design

**Resource Group:** `nbl2-compute-rg` — holds the virtual machines, availability set, and load balancer.

**Virtual Network:** `nbl2-vnet-core`, with a single subnet, `nbl2-web-snet`, hosting both web servers.

**Network Security Group:** inbound rules allow TCP 80 (HTTP) for web traffic and TCP 3389 (RDP) for remote administration.

**Availability Set:** `nbl2-avset`, protecting the VMs against hardware failure and planned maintenance, and increasing overall availability.

## Virtual Machines

Two Windows Server 2016 virtual machines were deployed, each running IIS and hosting an identical website. Each page displays its own server name, so traffic routing through the Load Balancer can be verified directly.

| VM | Role | Page Displays |
|---|---|---|
| VM01 | Web server 1 | NorthBridge High Availability Project — Server: VM01 |
| VM02 | Web server 2 | NorthBridge High Availability Project — Server: VM02 |

## Load Balancer Configuration

**Type:** Azure Standard Load Balancer

**Frontend IP:** public, single entry point for all client traffic

**Backend Pool:** contains VM01 and VM02, the set of servers that receive incoming requests

**Health Probe:**
- Protocol: HTTP
- Port: 80
- Path: `/`
- Purpose: continuously checks whether each server responds. Healthy servers keep receiving traffic, unhealthy ones get removed from rotation automatically.

**Load Balancing Rule:**
- Protocol: TCP
- Frontend Port: 80
- Backend Port: 80
- Session Persistence: None
- Purpose: distributes incoming HTTP traffic across the healthy backend servers.

## High Availability Validation

The site was accessed through the Load Balancer's public IP, with both servers responding correctly.

To confirm failover actually worked:

1. VM01 was stopped (deallocated)
2. The health probe detected the failure within seconds
3. The Load Balancer stopped routing traffic to VM01
4. All traffic shifted to VM02 automatically
5. The site stayed reachable on the same public address throughout, no manual steps involved

This confirmed the setup does what it's meant to do: survive the loss of one server without any downtime.

## Public IP Design Decision

During setup, both VMs had temporary public IPs so they could be reached directly for RDP, IIS installation, and initial testing.

Once the Load Balancer was configured and validated, those temporary public IPs were removed from both VMs. In the final setup, only the Load Balancer is publicly reachable. The two backend servers communicate solely over private IPs inside the virtual network.

This cuts down the public attack surface while keeping the application fully available, in line with standard Azure security practice.

## Architecture

```
Internet
   │
   ▼
Azure Standard Load Balancer
   (Public IP Address)
   │
   ▼
Health Probe (HTTP)
   │
   ▼
Load Balancing Rule
   │
   ▼
Backend Pool (VMs)
   │
   ├───────────────┬───────────────┐
   ▼                               ▼
Windows Server VM01        Windows Server VM02
IIS                        IIS
Private IP                 Private IP
   │                               │
   └───────────────┬───────────────┘
                    ▼
        Azure Virtual Network
           nbl2-web-snet
```

## Skills Demonstrated

Azure infrastructure deployment, virtual networking, subnet design, network security groups, Windows Server administration, IIS configuration, availability sets, Azure Load Balancer, backend pool configuration, health probes, load balancing rules, traffic distribution, automatic failover, infrastructure validation, high availability architecture, network troubleshooting, production infrastructure design, and security best practices.

## Learning Outcomes

Through this project, the following was learned hands-on:

- Designing highly available Azure infrastructure
- Deploying multiple Windows Server virtual machines
- Configuring Azure networking end to end
- Setting up an Azure Standard Load Balancer
- Implementing health probes
- Configuring load balancing rules
- Validating high availability through real failover testing
- Securing backend VMs by removing unnecessary public IP exposure
- Applying production-style Azure architecture principles instead of a single-server setup

## Screenshots

All screenshots are in `/screenshots`.

| # | Screenshot | Description |
|---|---|---|
| 01 | `01_nbl2_resource_groups.jpg` | Resource groups overview |
| 02 | `02_nbl2_virtual_network.jpg` | Virtual network |
| 03 | `03_nbl2_web_nsg.jpg` | Web NSG |
| 04 | `04_nbl2_subnet_association_web_nsg.jpg` | Subnet association with web NSG |
| 05 | `05_nbl2_nsg_inbound_rules.jpg` | NSG inbound rules |
| 06 | `06_nbl2_web_vm01_configuration_avset.jpg` | VM01 configuration, availability set |
| 07 | `07_nbl2_web_vm01_overview_page.jpg` | VM01 overview |
| 08 | `08_nbl2_web_vm02_overview_page.jpg` | VM02 overview |
| 09 | `09_nbl2_web_vms_list.jpg` | Both VMs listed |
| 10 | `10_nbl2_loadbalancer.jpg` | Load Balancer resource |
| 11 | `11_nbl2_loadbalancer_frontend_ip.jpg` | Load Balancer frontend IP |
| 12 | `12_nbl2_loadbalancer_backend_pool.jpg` | Backend pool configuration |
| 13 | `13_nbl2_web_vm01_iis_installation.jpg` | IIS installation on VM01 |
| 14 | `14_nbl2_web_vm02_ip.jpg` | VM02 IP address |
| 15 | `15_nbl2_web_vm02_iis_installation.jpg` | IIS installation on VM02 |
| 16 | `16_nbl2_web_vm01_testing_public_ip.jpg` | Testing VM01 via public IP |
| 17 | `17_nbl2_lb_health_probe.jpg` | Health probe configuration |
| 18 | `18_nbl2_lb_adding_rule.jpg` | Adding the load balancing rule |
| 19 | `19_nbl2_loadbalancer_backend_pool_main.jpg` | Backend pool, main view |
| 20 | `20_nbl2_lb_overview.jpg` | Load Balancer overview |
| 21 | `21_nbl2_lb_verifying.jpg` | Verifying traffic through the Load Balancer |
| 22 | `22_nbl2_stopped_vm01.jpg` | VM01 stopped for failover test |
| 23 | `23_nbl2_vm02_showed_lb_verified.jpg` | VM02 confirmed serving all traffic |
| 24 | `24_nbl2_vm01_public_ip_removed.jpg` | VM01 public IP removed |
| 25 | `25_nbl2_vm02_public_ip_removed.jpg` | VM02 public IP removed |

---

Built and deployed by **Shahab Hassan Khan** as a hands-on Azure high availability project.
