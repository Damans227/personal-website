---
title: "CloudStack KVM Host Add Failure - Part 1"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

# CloudStack KVM Host Add Failure - Part 1

Error encountered while attempting to add a KVM host to CloudStack:

> `Failed to setup keystore on the KVM host: 192.168.122.10`

Stack trace from management-server.log: 


```java
2025-10-03 22:36:14,828 DEBUG [c.c.h.k.d.KvmServerDiscoverer] (qtp1438988851-25:[ctx-99e0fc72, ctx-e969ec93]) (logid:a67f40bb)  
can't setup agent, due to com.cloud.utils.exception.CloudRuntimeException: Failed to setup keystore on the KVM host: 192.168.122.10 
- Failed to setup keystore on the KVM host: 192.168.122.10 
com.cloud.utils.exception.CloudRuntimeException: Failed to setup keystore on the KVM host: 192.168.122.10

        at com.cloud.hypervisor.kvm.discoverer.LibvirtServerDiscoverer.setupAgentSecurity(LibvirtServerDiscoverer.java:196)
        at com.cloud.hypervisor.kvm.discoverer.LibvirtServerDiscoverer.find(LibvirtServerDiscoverer.java:339)
        at com.cloud.resource.ResourceManagerImpl.discoverHostsFull(ResourceManagerImpl.java:873)
        at com.cloud.resource.ResourceManagerImpl.discoverHosts(ResourceManagerImpl.java:717)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:569)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:215)
        at jdk.proxy3/jdk.proxy3.$Proxy235.discoverHosts(Unknown Source)
```

---

## Problem

When adding the host at `192.168.122.10`, CloudStack failed with:

```
Could not add host due to:
can't setup agent, due to com.cloud.utils.exception.CloudRuntimeException: Failed to setup keystore on the KVM host: 192.168.122.10
```

This means the CloudStack management server could **not SSH and run `sudo keytool`** (or related commands) on the KVM host.

---

## Root Cause

The non-sudo user, in this case,  `darora` was being used to SSH into the KVM host. It ofcourse did **not have passwordless sudo** permissions for `keytool`, `mkdir`, `cp`, etc. — commands needed by the CloudStack agent setup.

CloudStack does:

- SSH into the KVM host
- Runs `sudo /usr/bin/keytool` and other setup commands
- Fails if password is prompted or permission is denied

---

## Fix: Sudoers Permissions for darora

On the KVM host (`192.168.122.10`), edit the sudoers file:

```bash
sudo visudo -f /etc/sudoers.d/cloudstack
```

Ensure this block exists:

```bash
Cmnd_Alias CLOUDSTACK = /bin/mkdir, /bin/mount, /bin/umount, /bin/cp, /bin/chmod, /usr/bin/keytool, /bin/keytool, /bin/touch, /bin/find, /bin/df, /bin/ls, /bin/qemu-img

Defaults:darora !requiretty
darora ALL=(ALL) NOPASSWD: CLOUDSTACK
```

---

## Validate It Works

From your CloudStack management server, run:

```bash
ssh darora@192.168.122.10 "sudo /usr/bin/keytool -help"
```

Expected: It prints keytool help without prompting for a password.

If it fails, test with:

```bash
ssh darora@192.168.122.10 "sudo -n /usr/bin/keytool -help"
```

If you see `sudo: a password is required`, the sudoers config isn’t active or is blocked by `requiretty`.

---

## Final Steps

1. Confirm `ssh darora@192.168.122.10 "sudo keytool -help"` works
2. Retry adding the KVM host in CloudStack UI
3. Watch the log:

```bash
tail -f /var/log/cloudstack/management/management-server.log
```

Look for:

```
Successfully added host
```

---

## Summary

Once sudo is passwordless for the commands CloudStack needs, host addition will succeed and the agent will connect.

---