---
title: "Creating a Virtual Machine in Apache CloudStack (via CloudMonkey)"
date: 2025-11-05T18:00:00Z
draft: false
tags:
  - CloudStack
  - CloudMonkey
  - Virtualization
  - Infrastructure
---

## Overview

1. Create a network  
2. Generate an SSH keypair  
3. Deploy a VM  
4. Associate a public IP  
5. Access the VM via SSH  

---

## List Available Templates

Templates define the base OS image your VM will use.

```bash
cmk list templates templatefilter=all filter=id,name
```

Example output:
```
id                                   name
-----------------------------------  ------------------------------
b9d4cf39-88dc-4092-b972-9114d315bdc0 Ubuntu 24.04
```

---

## List Network Offerings

Network offerings define connectivity features like NAT, DHCP, and firewall.

```bash
cmk list networkofferings filter=id,name,guestiptype,traffictype
```

Choose:
```
DefaultIsolatedNetworkOfferingWithSourceNatService
```

---

## Create a Network

Create your isolated network.  
This triggers deployment of a **Virtual Router (VR)** that provides NAT, DHCP, and DNS.

```bash
cmk create network   name=myisolatednet   displaytext="My Isolated Network"   networkofferingid=<network_offering_id>   zoneid=<zone_id>
```

---

## Create an SSH Keypair

This keypair will allow secure access to your VM.

```bash
cmk create sshkeypair name=mykeypair | jq -r '.keypair.privatekey' | sed 's/\\n/\n/g' > ~/mykeypair.pem
chmod 600 ~/mykeypair.pem
```

---

## Deploy the Virtual Machine

Deploy a VM using your template, network, and keypair.

```bash
cmk deploy virtualmachine   zoneid=<zone_id>   serviceofferingid=<service_offering_id>   templateid=<template_id>   networkids=<network_id>   keypair=mykeypair   name=ubuntu-keyed   displayname="Ubuntu 24.04 with Keypair"
```

---

## Associate a Public IP

Allocate a public IP address to your isolated network.

```bash
cmk associate ipaddress networkid=<network_id>
```

This public IP will be attached to your VR (Virtual Router).

---

## Create a Port Forwarding Rule

Forward incoming traffic on port 22 (SSH) to your VM.

```bash
cmk create portforwardingrule   ipaddressid=<public_ip_id>   privateport=22   publicport=22   protocol=tcp   virtualmachineid=<vm_id>
```

---

## SSH Into Your VM

Now you can connect to your VM securely.

```bash
ssh -i ~/mykeypair.pem ubuntu@<public_ip>
```

---

## Summary

| Step | Action | Command |
|------|--------|----------|
| 1 | List templates | `list templates` |
| 2 | List offerings | `list networkofferings` |
| 3 | Create network | `create network ...` |
| 4 | Create keypair | `create sshkeypair ...` |
| 5 | Deploy VM | `deploy virtualmachine ...` |
| 6 | Associate IP | `associate ipaddress ...` |
| 7 | Port forward | `create portforwardingrule ...` |
| 8 | SSH login | `ssh -i mykeypair.pem ubuntu@<ip>` |
