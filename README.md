# AWS Network Traffic Analysis & Outbound Traffic Blocking (VPC Flow Logs + NACLs)

Cloudbynea Portfolio Project • Incident Response • Network Security • Cost Control

## Overview

This project simulates a real operational scenario where unexpected network charges and unusual outbound traffic were detected in an AWS environment. Using **VPC Flow Logs**, **Amazon S3**, and **Network ACLs (NACLs)**, the goal was to identify the suspicious IP, analyze traffic patterns, and implement a **zero-downtime containment** strategy by blocking outbound traffic at the subnet level.

This case demonstrates practical skills in **cloud forensics, networking, and security hardening**, aligned with roles such as Cloud Support Engineer, Technical Account Manager (TAM), Security Analyst, and FinOps practitioner.

---

## Problem Statement

A customer reported increased data transfer costs and intermittent connections from their web servers to an unknown external IP. Security Groups were permissive, and no network monitoring was enabled, leaving the root cause unclear.

**Objectives:**

- Capture and analyze network traffic  
- Identify suspicious external destinations  
- Block outbound connections from the subnet  
- Validate the enforcement  
- Ensure no workload downtime  

---

## Architecture Summary

The environment consists of:

- A **VPC** with a **public subnet**
- Two **EC2 Web Servers**
- A permissive **Security Group** (allows outbound traffic)
- A **Network ACL (NACL)** controlling subnet-level traffic
- **VPC Flow Logs** stored in an **Amazon S3 bucket**
- A “mysterious” external IP communicating with the instances

Flow Logs enable visibility into traffic, allowing detection of the specific IP that should not be communicating with the workload.

> **Security model:**  
> - Security Groups: stateful, instance-level  
> - Network ACLs: stateless, subnet-level  

---

## Technical Approach

### 1. Enable VPC Flow Logs

VPC Flow Logs were configured on the VPC and sent to an S3 bucket for analysis.

**Key settings:**

- **Resource:** VPC associated with the web servers  
- **Destination:** Amazon S3 bucket  
- **Filter:** `ACCEPT`  
- **Aggregation interval:** 1 minute  

This configuration produced records containing:

- Source and destination IP addresses  
- Ports and protocol  
- Packets and bytes  
- Action (`ACCEPT` or `REJECT`)  
- Log status  

---

### 2. Analyze Flow Logs for Suspicious IPs

Downloaded and inspected `.gz` Flow Log files revealed public IPs generating traffic to/from the EC2 instances.

Example extracted from logs:

```text
version account-id interface-id srcaddr      dstaddr     srcport dstport protocol packets bytes  start       end         action log-status
2       9015...    eni-...     98.87.174.97 10.10.0.20  443     50258   6        6       378    ...         ...         ACCEPT OK
2       9015...    eni-...     10.10.0.20   98.87.174.97 50258  443     6        8       490    ...         ...         ACCEPT OK
