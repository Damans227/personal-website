---
title: "CloudStack Template Download and Console Session Flow"
date: 2025-11-10T14:00:00-05:00
draft: false
tags:
  - cloudstack
  - ssvm
  - cpvm
  - templates
  - console
---

## 1. Template / ISO Download Workflow

When a user registers a new ISO or template via a URL, CloudStack offloads the download job to the **Secondary Storage VM (SSVM)**.

| Step | Action                                          | Component                                |
| ---- | ----------------------------------------------- | ---------------------------------------- |
| 1    | User hosts ISO/template on HTTP server          | External (public web server)             |
| 2    | User registers the URL                          | CloudStack UI / API                      |
| 3    | CloudStack sends download job to SSVM           | Management Server                        |
| 4    | SSVM downloads the file via HTTP/HTTPS          | Secondary Storage VM (SSVM)              |
| 5    | SSVM writes the file to secondary storage (NFS) | `/export/secondary` on Management Server |
| 6    | CloudStack marks the template/ISO as “Ready”    | CloudStack Database                      |

### Textual Flow Diagram

```
+----------------------------+
| CloudStack Management      |
+-------------+--------------+
              |
   Registers Template/ISO URL
              |
              v
     +-----------------------+
     | Secondary Storage VM  |  (SSVM)
     +-----------+-----------+
                 |
         Downloads via HTTP/HTTPS
                 |
                 v
      +------------------------+
      | Secondary Storage (NFS)|
      |  /export/secondary     |
      +------------------------+
                 |
                 v
       Marks Template as READY
```

---

## 2. Console Session Establishment Flow

When a user opens the **VM Console** (via UI or API), CloudStack routes the connection through the **Console Proxy VM (CPVM)**.

| Step | Action                           | Description                                                                                                                         |
| ---- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
|1     | User initiates a console session | The user clicks **“View Console”** in the CloudStack UI — request goes to the **Management Server (CSMan)**.                        |
|2     | Management chooses suitable CPVM | CloudStack selects an active **CPVM** in that zone and generates a **temporary authentication token**.                              |
|3     | Management sends redirection URL | The user’s browser receives a link like: <br>`https://cpvm-zone1.example.com:8443/console?cmd=auth&token=XYZ`                       |
|4     | User resolves CPVM address       | Browser resolves CPVM hostname/IP via DNS or direct IP.                                                                             |
|5     | User connects to CPVM via HTTPS  | Browser opens a secure HTTPS session to the CPVM’s **public interface**.                                                            |
|6     | CPVM validates session token     | CPVM contacts the **Management Server** via the management/control network to verify the token.                                     |
|7     | CPVM connects to VM console      | CPVM connects internally to the **VM’s VNC port** on the hypervisor and streams that session over HTTPS back to the user’s browser. |

### Textual Flow Diagram

```
+--------------------------+
| CloudStack Management    |
| (Generates Token)        |
+-------------+------------+
              |
        HTTPS Redirect
              |
              v
     +----------------------+
     | Console Proxy VM     |  (CPVM)
     +-----------+----------+
                 |
    Validates Token via Mgmt Net
                 |
                 v
     +----------------------+
     | Hypervisor Host      |
     | (VM’s VNC Console)   |
     +----------------------+
                 |
     Streams HTTPS Console back
                 |
                 v
        +----------------+
        | User Browser   |
        +----------------+
```

---

## Summary

- **SSVM** handles all **template, ISO, and snapshot copy/download** jobs to and from secondary storage.  
- **CPVM** handles **console access (VNC)** securely via HTTPS and token validation.

Understanding these flows helps diagnose issues such as:
- Templates stuck in *Downloading* or *Not Ready* states.
- Console sessions that fail to load or timeout.
