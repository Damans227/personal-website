---
title: "Marvin `deployDataCenter.py` Troubleshooting Summary"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

## 1) Marvin `deployDataCenter.py` — Issues & Fixes

### 1.1 Running marvin from the root

**Fix**: Run from repo root:
```bash
cd ~/lab/cloudstack
python3 tools/marvin/marvin/deployDataCenter.py -i setup/dev/advanced.cfg
```

---

### 1.2 DNS: `sim` Not Resolvable

**Symptom**
```
UnknownHostException: sim: Name or service not known
```

**Fix Options**
- Add to `/etc/hosts`:
  ```bash
  echo "127.0.0.1 sim" | sudo tee -a /etc/hosts
  ```
- Or change host URLs in `advanced.cfg` to:
  ```
  http://127.0.0.1/c0/h0
  ```

---

### 1.3 Host Authentication Error (KVM path only)

**Symptom**
```
errorText: [Authentication error with host password]
```

**Fix**
- Ensure `username`/`password` in `advanced.cfg` match real SSH login
- Or prefer **SSH key-based auth**:
  ```bash
  ssh-copy-id darora@127.0.0.1
  ```

---

### 1.4 API Auth (401): “Unable to verify credentials”

**Symptom**
```
errorCode: 401
errorText: unable to verify user credentials and/or request signature
```

**Root Cause**: Missing `apiKey` and `secretKey` in `mgtSvr` block

**Fix**

```json
"mgtSvr": [{
  "mgtSvrIp": "127.0.0.1",
  "port": 8080,
  "useHttps": false,
  "user": "admin",
  "passwd": "password",
  "domain": "ROOT",
  "apiKey": "YOUR_API_KEY",
  "secretKey": "YOUR_SECRET_KEY",
  "certCAPath": "NA",
  "certPath": "NA"
}]
```

---

### 1.5 `NoneType.encode()` During Request Signing

**Symptom**
```
AttributeError: 'NoneType' object has no attribute 'encode'
```
**Trace**
```
hmac.new(self.securityKey.encode('utf-8'), ...)
```

**Root Cause**: `self.securityKey` is `None` because Marvin expected:
```python
mgmt_details.apiKey  # attribute
```
But config passed:
```python
mgmt_details["apiKey"]  # dict access
```

** Fix — Patch CSTestClient**

In `tools/marvin/marvin/cloudstackTestClient.py`, inside `__createApiClient()`:

```python
self.__mgmtDetails is missing apiKey and secretKey
```
---

## 2) Pivot to Simulator (Skip KVM for Now)

### 2.1 Minimal `advanced.cfg` Changes

```json
"hypervisor": "Simulator",
"url": "http://sim/c0/h0",
"username": "sim",
"password": "sim"
```

And add to `/etc/hosts` if needed:

```bash
echo "127.0.0.1 sim" | sudo tee -a /etc/hosts
```

### 2.2 Deploy Again
```bash
python3 tools/marvin/marvin/deployDataCenter.py -i setup/dev/advanced.cfg -d
```

Logs: `/tmp/MarvinLogs/DeployDataCenter__*/runinfo.txt`