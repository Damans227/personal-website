---
title: "Apache CloudStack, hands on: the full video series"
date: 2026-11-02T02:00:00Z
draft: false
tags:
  - CloudStack
  - Cloud
  - Infrastructure
---

There are good CloudStack introductions and install videos out there. I wanted one series that follows a single cloud the whole way, from the first install to a production setup for customers, with every click and command shown. So I made it, free. I hope the CloudStack community finds it useful. Every script used in the videos is below.

## Scripts

## Video 05a: MacVisor

### video-05a-mvz-commands

The same VMs from the terminal with the `mvz` command, then a look inside the VM. Chapter "From the terminal", 2:08.

```bash
mvz list
mvz ip cloudstack
mvz shell cloudstack
hostname; uname -m
```

## Video 05b: Installing CloudStack on a Mac

### video-05b-log-in-and-check

Log in to the VM, become root, and check the architecture and network card. Chapter "Logging in", 1:27.

```bash
ssh YOUR_USER@192.168.64.22
sudo -i
uname -m
ip -br a
```

### video-05b-netplan-cloudbr0

Writes the Netplan file for the `cloudbr0` bridge. The first address is the VM's own, so use yours. Chapter "The bridge", 1:49.

```bash
cat > /etc/netplan/60-cloudbr0.yaml <<EOF
network:
  version: 2
  ethernets:
    enp0s1: {dhcp4: false, dhcp6: false}
  bridges:
    cloudbr0:
      interfaces: [enp0s1]
      addresses: [192.168.64.22/24, 172.16.10.1/24]
      routes: [{to: default, via: 192.168.64.1}]
      nameservers: {addresses: [192.168.64.1]}
      parameters: {stp: false, forward-delay: 0}
EOF
```

### video-05b-apply-bridge

Removes the old Netplan file, applies the new one, and checks the bridge. Chapter "The bridge", 1:49.

```bash
rm -f /etc/netplan/50-cloud-init.yaml
chmod 600 /etc/netplan/*
netplan apply
ip -br a show cloudbr0
```

### video-05b-forwarding-and-nat

Lets the second address on the bridge reach the outside through this VM. Chapter "Why a second address", 3:08, into "The packages".

```bash
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -s 172.16.10.0/24 ! -d 172.16.10.0/24 -j MASQUERADE
iptables -P FORWARD ACCEPT
```

### video-05b-install-packages

Adds the CloudStack package repository and installs everything on one VM. Chapter "The packages", 3:31.

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.cloudstack.org/release.asc | gpg --dearmor > /etc/apt/keyrings/cloudstack.gpg
echo "deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] https://download.cloudstack.org/ubuntu noble 4.22" > /etc/apt/sources.list.d/cloudstack.list
apt-get update -qq
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq cloudstack-management cloudstack-agent mysql-server nfs-kernel-server qemu-kvm libvirt-daemon-system >/dev/null && echo INSTALLED
```

### video-05b-mysql-cloudstack-cnf

The MySQL settings CloudStack's install guide asks for, in their own file. Chapter "Database, storage and server", 4:53.

```bash
cat > /etc/mysql/mysql.conf.d/cloudstack.cnf <<EOF
[mysqld]
server_id=1
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log_bin=mysql-bin
binlog_format=ROW
EOF
```

### video-05b-mysql-root-password

Restarts MySQL and gives its root user the demo password. Chapter "Database, storage and server", 4:53.

```bash
systemctl restart mysql
mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';"
```

### video-05b-nfs-exports

Shares primary and secondary storage from this VM over NFS. Chapter "Database, storage and server", 4:53.

```bash
mkdir -p /export/primary /export/secondary
echo '/export *(rw,async,no_root_squash,no_subtree_check)' >> /etc/exports
exportfs -a
```

### video-05b-database-and-server

Creates the CloudStack database and starts the management server. Chapter "Database, storage and server", 4:53.

```bash
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:password -i 192.168.64.22 | tail -1; echo DBDONE
cloudstack-setup-management | tail -3
```

### video-05b-root-over-ssh

Lets CloudStack log in to this host as root with a password, which it needs to add the host. Just before "First login", 6:55.

```bash
echo root:password | chpasswd
echo 'PasswordAuthentication yes' > /etc/ssh/sshd_config.d/00-lab.conf
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart ssh
```

### video-05b-agent-properties

Adds the agent settings an ARM host needs, then restarts the agent. Chapter "The agent settings", 8:48.

```bash
echo host.cpu.manual.speed.mhz=2000 >> /etc/cloudstack/agent/agent.properties
echo guest.cpu.arch=aarch64 >> /etc/cloudstack/agent/agent.properties
echo guest.cpu.mode=host-passthrough >> /etc/cloudstack/agent/agent.properties
systemctl restart cloudstack-agent
```

### video-05b-instance-key

Makes the SSH key used to log in to the instance, and prints the public half. Chapter "A real instance", 10:54.

```bash
ssh-keygen -t ed25519 -N "" -f ~/k -q; cat ~/k.pub
```

### video-05b-instance-login

Logs in through the public address and port forward, then checks the architecture and internet access. PUBLIC_IP is your network's source NAT address. Chapter "Logging in", 12:01.

```bash
ssh -b 192.168.64.22 -i ~/k -p 2222 -o StrictHostKeyChecking=accept-new ubuntu@PUBLIC_IP
uname -m
curl -s -o /dev/null -w "%{http_code}\n" https://download.cloudstack.org/
```

## Video 11: acme on the internet

### video-11-ssh-keygen

Makes an SSH key on your own machine. Chapter "acme-key and acme-web-init", 6:56.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/acme-key
```

### video-11-acme-web-init

User data that writes a small web page and serves it on port 80. Chapter "acme-key and acme-web-init", 6:56.

```yaml
#cloud-config
write_files:
  - path: /etc/systemd/system/acme-web.service
    content: |
      [Unit]
      Description=acme web page
      After=network-online.target
      [Service]
      ExecStartPre=/bin/sh -c 'mkdir -p /srv/www && echo "<h1>Hello from $(hostname)</h1>" > /srv/www/index.html'
      ExecStart=/usr/bin/python3 -m http.server 80 --directory /srv/www
      Restart=always
      [Install]
      WantedBy=multi-user.target
runcmd:
  - systemctl daemon-reload
  - systemctl enable --now acme-web
```

## Video 12: acme's tiers, and a second site

### video-12-acme-dr-init

User data that sets a password, so the second site's server can test the tunnel from its console. Between chapters "Site-to-site VPN", 7:27, and "VPN gateway and customer gateway", 9:51.

```yaml
#cloud-config
password: nimbusdr1
chpasswd:
  expire: false
```

## Video 13: Shared networks and autoscale

### video-13-acme-web-busy

The web page user data, plus a line that keeps the CPU busy for fifteen minutes to trigger scale up. Chapter "priya sets up the group", 7:11.

```yaml
#cloud-config
write_files:
  - path: /etc/systemd/system/acme-web.service
    content: |
      [Unit]
      Description=acme web page
      After=network-online.target
      [Service]
      ExecStartPre=/bin/sh -c 'mkdir -p /srv/www && echo "<h1 style=font-family:sans-serif;font-size:72px;text-align:center;margin-top:35vh>Hello from $(hostname)</h1>" > /srv/www/index.html'
      ExecStart=/usr/bin/python3 -m http.server 80 --directory /srv/www
      Restart=always
      [Install]
      WantedBy=multi-user.target
runcmd:
  - systemctl daemon-reload
  - systemctl enable --now acme-web
  - nohup timeout 900 sh -c "while :; do :; done" >/dev/null 2>&1 &
```

## Video 14: Troubleshooting and maintenance

### video-14-find-job-in-log

Finds one job's log ID, then that job's error. LOGID is the value the first grep prints. Chapters "Finding one job in the log", 2:20, and "Log ID, then the reason", 2:46.

```bash
cd /var/log/cloudstack/management
grep -a acme-report-1 management-server.log | grep -a DeployVMCmd | grep -o 'logid:[0-9a-f]*' | tail -1
grep -a LOGID management-server.log | grep -a ERROR | head -1 | cut -d')' -f3-
```

### video-14-agent-stop-and-start

Stops the agent on a KVM host, reads its log, and starts it again. Chapters "kvm1 disconnected", 4:42, and "The fix", 5:24.

```bash
systemctl stop cloudstack-agent
systemctl is-active cloudstack-agent
tail -2 /var/log/cloudstack/agent/agent.log | cut -c1-90
systemctl start cloudstack-agent
```

### video-14-maintenance-failure-reason

Finds why maintenance mode failed. Chapter "It failed: why", 7:25.

```bash
cd /var/log/cloudstack/management
grep -a 'Unable to find a server' management-server.log | tail -1 | cut -d')' -f3- | cut -c1-130
```

## Video 15: A second data center

### video-15-acme-web-init

The same user data as video 11, used for the server in zone-fra1. Chapter "A group with no rules", 4:30.

```yaml
#cloud-config
write_files:
  - path: /etc/systemd/system/acme-web.service
    content: |
      [Unit]
      Description=acme web page
      After=network-online.target
      [Service]
      ExecStartPre=/bin/sh -c 'mkdir -p /srv/www && echo "<h1>Hello from $(hostname)</h1>" > /srv/www/index.html'
      ExecStart=/usr/bin/python3 -m http.server 80 --directory /srv/www
      Restart=always
      [Install]
      WantedBy=multi-user.target
runcmd:
  - systemctl daemon-reload
  - systemctl enable --now acme-web
```

## Video 16: HostPro wants single sign-on

### video-16-podman-run-ldap

Starts a small test directory on a spare machine. For learning, not production. Chapter "A test directory", 1:24.

```bash
podman run -d --name hostpro-ldap -p 389:389 -e LDAP_ORGANISATION=HostPro -e LDAP_DOMAIN=hostpro.example -e LDAP_ADMIN_PASSWORD=dirsecret1 docker.io/osixia/openldap:1.5.0
```

### video-16-hostpro-users-ldif

The people to load into the directory. On camera it is typed into a file called `hostpro-users.ldif`. Chapter "A test directory", 1:24.

```ldif
dn: ou=people,dc=hostpro,dc=example
objectClass: organizationalUnit
ou: people

dn: uid=maria,ou=people,dc=hostpro,dc=example
objectClass: inetOrgPerson
uid: maria
cn: Maria Lopez
givenName: Maria
sn: Lopez
mail: maria@hostpro.example
userPassword: dirpass1

dn: uid=tom,ou=people,dc=hostpro,dc=example
objectClass: inetOrgPerson
uid: tom
cn: Tom Becker
givenName: Tom
sn: Becker
mail: tom@hostpro.example
userPassword: dirpass2
```

### video-16-load-and-check-ldif

Copies the file into the container, loads it, and searches for the result. Chapter "A test directory", 1:24.

```bash
podman cp hostpro-users.ldif hostpro-ldap:/tmp/
podman exec hostpro-ldap ldapadd -x -D cn=admin,dc=hostpro,dc=example -w dirsecret1 -f /tmp/hostpro-users.ldif
podman exec hostpro-ldap ldapsearch -x -LLL -D cn=admin,dc=hostpro,dc=example -w dirsecret1 -b ou=people,dc=hostpro,dc=example uid mail
```

### video-16-change-ldap-password

Changes maria's password in the directory, then asks about the old one. Chapter "Changing the password in the directory", 5:53.

```bash
podman exec hostpro-ldap ldappasswd -x -D cn=admin,dc=hostpro,dc=example -w dirsecret1 -s dirpass9 uid=maria,ou=people,dc=hostpro,dc=example
podman exec hostpro-ldap ldapwhoami -x -D uid=maria,ou=people,dc=hostpro,dc=example -w dirpass1
```

### video-16-console-values

The values typed into the console for LDAP and two-factor. Chapters "Connecting HostPro to it", 2:44, "Linking the domain", 4:18, and "Turning it on", 7:18.

```text
LDAP connection: the directory's address, port 389, domain HostPro
ldap.basedn: dc=hostpro,dc=example
ldap.bind.principal: cn=admin,dc=hostpro,dc=example
ldap.bind.password: dirsecret1
Link domain to LDAP: type OU, LDAP domain ou=people,dc=hostpro,dc=example, account type 0
enable.user.2fa: on, for HostPro
```

### video-16-oathtool

Makes a two-factor code without a phone. YOUR_SETUP_KEY is the key the console gives you. Chapter "Turning it on", 7:18.

```bash
oathtool --totp -b YOUR_SETUP_KEY
```

## Video 17: Bill HostPro for the month

### video-17-usage-server-commands

Installs the usage server, starts it at boot, then checks and restarts it. Chapters "Installing the usage server", 1:41, and "Checking it's running", 2:04. The docs page shows the older chkconfig and update-rc.d commands; `systemctl enable` does the same job.

```bash
# Red Hat systems, then Ubuntu
yum install cloudstack-usage
apt install cloudstack-usage

# Start it, and set it to start at boot
service cloudstack-usage start
systemctl enable cloudstack-usage

# Check it, restart it
systemctl status cloudstack-usage --no-pager -n0 | head -3
systemctl restart cloudstack-usage
```

### video-17-usage-aggregation-setting

The setting changed just for the video, then put back. Chapter "A setting, just for this video", 3:38.

```text
usage.stats.job.aggregation.range = 5     (for the demo)
usage.stats.job.aggregation.range = 1440  (the default, one full day)
```

### video-17-bill-py

Adds up running hours per account from a usage CSV and prices them. Run it with `python3 bill.py usage-records.csv`. Chapter "A small billing script", 5:58.

```python
import csv, sys, collections
PRICE = 0.02  # per running hour, made up
hours = collections.Counter()
for row in csv.DictReader(open(sys.argv[1])):
    if row['usagetype'] == '1':  # running instance
        hours[row['account']] += float(row['rawusage'])
for account, h in sorted(hours.items()):
    print(f"{account:12} {h:8.2f} hours  {h * PRICE:6.2f}")
```

## Video 18: dataco wants backups

### video-18-nfs-share

Shares a folder over NFS from a Red Hat style machine with nfs-utils installed. Use your own hosts' network, and open the NFS ports if your firewall is on. Chapter "Sharing a folder over NFS", 2:44.

```bash
mkdir -p /export/backups
echo '/export/backups YOUR_HOSTS_CIDR(rw,sync,no_root_squash)' >> /etc/exports
cat /etc/exports
systemctl enable --now nfs-server
exportfs -v
```

### video-18-list-backup-files

Lists the backup files on the NFS machine. Chapter "What landed on the NAS", 8:02.

```bash
find /export/backups -type f -exec ls -lh {} +
```

### video-18-trial-server-commands

Run inside the trial server, from its console, to make a file and later lose it. Chapters "Something worth saving", 6:27, and "Losing the file", 9:05.

```bash
echo order-1001 > orders.txt
cat orders.txt
sync
rm -f orders.txt
```

### video-18-settings-and-form-values

The settings and form values used. Chapters "Picking the plugin", 3:35, to "A schedule", 8:25.

```text
backup.framework.enabled: true, both globally and on the zone (false by default)
backup.framework.provider.plugin: nas (dummy by default)

Repository: name nimbus-nas, address <your NFS machine>:/export/backups, type nfs
Offering: name nimbus-nas-offering, External Id: pick nimbus-nas from the list, Allow User driven backups on, Public on
Schedule: daily, 02:00, keep 7, Europe/London
```

## Video 19: dataco wants Kubernetes

### video-19-copy-kubeconfig

Copies the downloaded config to a Linux machine that can reach the cluster's public address. Replace MACHINE with yours. Chapter "dan's kubeconfig", 5:36.

```bash
scp kube.conf root@MACHINE:
ssh root@MACHINE
```

### video-19-get-kubectl

Gets kubectl and points it at the config file. Use the Linux link on your cluster's Access tab, and replace VERSION with yours. Chapter "kubectl and three copies of nginx", 6:13.

```bash
curl -LO https://dl.k8s.io/release/vVERSION/bin/linux/amd64/kubectl
install kubectl /usr/local/bin/
export KUBECONFIG=~/kube.conf
```

### video-19-three-copies-of-nginx

Looks at the cluster and starts three copies of nginx. Chapter "kubectl and three copies of nginx", 6:13.

```bash
kubectl get nodes
kubectl create deployment web --image=nginx --replicas=3
kubectl rollout status deployment/web
kubectl get pods -o custom-columns=POD:.metadata.name,NODE:.spec.nodeName
```

### video-19-six-copies

After scaling the cluster, asks for six copies. Chapter "More pods, and a load balancer", 8:07.

```bash
kubectl scale deployment web --replicas=6
kubectl rollout status deployment/web
```

### video-19-public-address

Puts the deployment on a public address through a CloudStack load balancer. Chapter "More pods, and a load balancer", 8:07.

```bash
kubectl expose deployment web --type=LoadBalancer --port=80
kubectl get service web
```

### video-19-settings

The settings looked at. Chapters "The switch", 2:23, and "The endpoint URL", 2:37.

```text
cloud.kubernetes.service.enabled (true on our lab. If yours is false, set it to true and restart the management server.)
endpoint.url (the address of your management server's API, as cluster nodes can reach it. The button in the console writes the address your browser is using, so check that the nodes can reach it too.)
```

## Video 20: Bring servers in

### video-20-install-virt-tools

Installs the tools the next script needs on the old KVM machine. Chapter "The old machine", 1:47.

```bash
dnf install -y -q virt-install libguestfs-tools-c
```

### video-20-old-server-sh

Builds the small test server on a plain KVM machine. Run it with `bash old-server.sh`. Chapter "The old machine", 1:47.

```bash
#!/bin/bash
# One small server, on a plain KVM machine.
IMG=/var/lib/libvirt/images/acme-legacy.qcow2
curl -sSL -o $IMG https://download.cloudstack.org/templates/cloud-images/ubuntu/ubuntu-22.04-server-cloudimg-amd64.img
export LIBGUESTFS_BACKEND=direct
virt-customize -q -a $IMG --hostname acme-legacy --root-password password:nimbus1 \
  --run-command 'touch /etc/cloud/cloud-init.disabled' \
  --run-command 'printf "network:\n  version: 2\n  ethernets:\n    all:\n      match: {name: \"en*\"}\n      dhcp4: true\n" > /etc/netplan/50-all.yaml' \
  --run-command 'echo "acme-ops orders" > /root/orders.txt'
virt-install --name acme-legacy --memory 1024 --vcpus 1 --import --disk $IMG --network bridge=cloudbr0 --osinfo ubuntu22.04 --graphics vnc --noautoconsole
```

### video-20-libvirt-tcp-port

Opens libvirt's TCP port. With `auth_tcp = "none"`, anyone reaching that port gets in, so use this on a closed network only, and check the docs for your own distribution. Chapter "Opening libvirt's TCP port", 4:00.

```bash
grep -E '^(listen_tcp|auth_tcp)' /etc/libvirt/libvirtd.conf
sed -i 's/^listen_tcp=0/listen_tcp=1/' /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
ss -ltn | grep 16509
```

### video-20-stop-the-server

Only stopped servers can be imported, so this shuts it down and checks. Chapter "Only stopped servers", 4:33.

```bash
virsh shutdown acme-legacy
virsh list --all
```

### video-20-temp-copy-cleanup

Finds and deletes the converted copy the import leaves behind. Chapter "What stays behind", 7:09.

```bash
find /tmp -maxdepth 1 -type f -size +100M -exec ls -lh {} +
find /tmp -maxdepth 1 -type f -size +100M -delete
```

### video-20-disk-files

Makes the second server's disk and an empty data disk on primary storage, for the file and volume imports. Chapter "Getting the files onto primary storage", 8:05.

```bash
cd /var/lib/libvirt/images
cp acme-legacy.qcow2 acme-reports.qcow2
qemu-img create -f qcow2 acme-data.qcow2 2G
ls -lh
```

## Video 21: New features tour

### video-21-minio-test-server

The small test MinIO server, done off camera and built from source. Check MinIO's own docs for what's current. The two values are the Access Key and Secret Key in the Add Object Storage form. Mentioned at chapter "Adding an object store", 0:42.

```bash
go install github.com/minio/minio@RELEASE.2025-04-22T22-12-26Z
mkdir -p /srv/minio
MINIO_ROOT_USER=nimbusadmin MINIO_ROOT_PASSWORD=<a password> minio server /srv/minio --address :9000 --console-address :9001
```

### video-21-mount-shared-file-system

Mounts the share on an instance and writes a file. Chapter "Mounting it", 5:55.

```bash
mkdir -p /files
mount -t nfs <address from the Access tab>:/export /files
df -h /files
echo written by acme-web-1 > /files/hello.txt
cat /files/hello.txt
```

### video-21-webhook-settings

The two settings changed for the demo, then put back. Chapters "Two settings, just for this video", 9:39, and "Putting it back", 12:06.

```text
webhook.delivery.allow.http (false by default)
webhook.delivery.blocklist (we took out 10.0.0.0/8)
```

### video-21-hook-py

A tiny server that prints each webhook event. Run it with `python3 hook.py`. Chapter "A small receiver", 8:16.

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json

class Hook(BaseHTTPRequestHandler):
    def do_POST(self):
        size = int(self.headers['Content-Length'])
        event = json.loads(self.rfile.read(size))
        print(self.headers['X-CS-Event'], event['status'], event['description'], flush=True)
        self.send_response(200)
        self.end_headers()
    def log_message(self, *args):
        pass

HTTPServer(('', 8088), Hook).serve_forever()
```

## Video 22: The API

### video-22-unsigned-call

A call with no signature, which gets refused. Chapter "A call with no signature", 2:54.

```bash
curl -s 'http://MANAGEMENT_SERVER_IP:8080/client/api?command=listZones&response=json'; echo
```

### video-22-api-py

Signs an API call and prints the reply. Chapter "A script that signs", 3:10.

```python
import base64, hashlib, hmac, json, os, sys, urllib.error, urllib.parse, urllib.request
params = dict(arg.split('=', 1) for arg in sys.argv[1:])
params['apikey'] = os.environ['CS_KEY']
params['response'] = 'json'
quote = lambda v: urllib.parse.quote(v, safe='')
query = '&'.join(k + '=' + quote(v) for k, v in sorted(params.items()))
secret = os.environ['CS_SECRET'].encode()
digest = hmac.new(secret, query.lower().encode(), hashlib.sha1).digest()
signature = quote(base64.b64encode(digest).decode())
try:
    url = os.environ['CS_URL'] + '?' + query + '&signature=' + signature
    reply = urllib.request.urlopen(url)
except urllib.error.HTTPError as error:
    reply = error
print(json.dumps(json.load(reply), indent=1))
```

### video-22-api-environment

The three values the script reads, then the first signed call. Chapter "The first signed call", 3:50.

```bash
export CS_URL=http://MANAGEMENT_SERVER_IP:8080/client/api
export CS_KEY=YOUR_API_KEY
export CS_SECRET=YOUR_SECRET_KEY
python3 api.py command=listZones | head -24
```

### video-22-look-up-ids

Looks up the IDs the ticket needs. Chapter "Looking up the IDs", 4:00.

```bash
ZONE=$(python3 api.py command=listZones name=zone-lon1 | jq -r '.listzonesresponse.zone[0].id')
echo $ZONE
DOM=$(python3 api.py command=listDomains name=HostPro listall=true | jq -r '.listdomainsresponse.domain[0].id')
SIZE=$(python3 api.py command=listServiceOfferings name=nimbus-small | jq -r '.listserviceofferingsresponse.serviceoffering[0].id')
IMG=$(python3 api.py command=listTemplates templatefilter=all listall=true name=acme-ubuntu-22.04 | jq -r '.listtemplatesresponse.template[0].id')
NETO=$(python3 api.py command=listNetworkOfferings name=DefaultIsolatedNetworkOfferingWithSourceNatService | jq -r '.listnetworkofferingsresponse.networkoffering[0].id')
UD=$(python3 api.py command=listUserData name=acme-web-init listall=true | jq -r '.listuserdataresponse.userdata[0].id')
OWNER="account=acme-dev domainid=$DOM"
```

### video-22-network-and-instance

Creates the network, deploys the instance, and follows the async job. Chapters "Create the network", 4:51, to "Deploy, and follow the job", 5:29.

```bash
python3 api.py command=createNetwork name=acme-stage-net displaytext=acme-stage-net zoneid=$ZONE networkofferingid=$NETO gateway=10.3.0.1 netmask=255.255.255.0 $OWNER > net.json
jq -r '.createnetworkresponse.network | .name + " " + .state' net.json
NET=$(jq -r '.createnetworkresponse.network.id' net.json)
python3 api.py command=deployVirtualMachine name=acme-stage-1 zoneid=$ZONE templateid=$IMG serviceofferingid=$SIZE networkids=$NET keypairs=acme-key userdataid=$UD $OWNER > vm.json; cat vm.json
JOB=$(jq -r '.deployvirtualmachineresponse.jobid' vm.json)
VM=$(jq -r '.deployvirtualmachineresponse.id' vm.json)
python3 api.py command=queryAsyncJobResult jobid=$JOB | jq '.queryasyncjobresultresponse | {jobstatus}'
python3 api.py command=queryAsyncJobResult jobid=$JOB | jq '.queryasyncjobresultresponse | {jobstatus, name: .jobresult.virtualmachine.name, state: .jobresult.virtualmachine.state, address: .jobresult.virtualmachine.nic[0].ipaddress}'
```

### video-22-address-firewall-port-forward

Gets a public address, opens port 80, forwards it to the instance, and loads the page. Chapters "Public address", 6:14, to "Is it on the internet?", 7:04.

```bash
python3 api.py command=associateIpAddress networkid=$NET > ip.json; cat ip.json
IPID=$(jq -r '.associateipaddressresponse.id' ip.json)
IP=$(python3 api.py command=listPublicIpAddresses id=$IPID listall=true | jq -r '.listpublicipaddressesresponse.publicipaddress[0].ipaddress')
echo $IP
python3 api.py command=createFirewallRule ipaddressid=$IPID protocol=tcp startport=80 endport=80 cidrlist=0.0.0.0/0 | jq -c .
python3 api.py command=createPortForwardingRule ipaddressid=$IPID protocol=tcp publicport=80 privateport=80 virtualmachineid=$VM openfirewall=false | jq -c .
python3 api.py command=listPortForwardingRules ipaddressid=$IPID listall=true | jq '.listportforwardingrulesresponse.portforwardingrule[0] | {state, publicport, privateport, virtualmachinename}'
curl -s http://$IP/; echo
```

## Video 23: CloudMonkey

### video-23-install-cmk

Installs CloudMonkey on the management server. Chapter "Installing it", 1:09.

```bash
curl -sL -o /usr/local/bin/cmk https://github.com/apache/cloudstack-cloudmonkey/releases/latest/download/cmk.linux.x86-64
chmod +x /usr/local/bin/cmk
```

### video-23-cmk-profile

Typed at the `cmk` prompt: a profile with your address and keys, then sync. Chapter "A profile, and sync", 1:22.

```text
set profile nimbus
set url http://MANAGEMENT_SERVER_IP:8080/client/api
set apikey YOUR_API_KEY
set secretkey YOUR_SECRET_KEY
sync
set output table
```

### video-23-ticket-by-hand

The ticket at the `cmk` prompt. On camera, tab fills most of these IDs. The CAPS names stand for your own IDs. Chapters "The network", 3:08, to "Address, firewall, port forwarding", 5:02.

```text
list networkofferings name=DefaultIsolatedNetworkOfferingWithSourceNatService filter=id
create network name=acme-stage-net displaytext=acme-stage-net zoneid=ZONE_ID networkofferingid=NETWORK_OFFERING_ID gateway=10.3.0.1 netmask=255.255.255.0 account=acme-dev domainid=DOMAIN_ID filter=name,state,id
list templates templatefilter=all listall=true name=acme-ubuntu-22.04 filter=name,id
list userdata listall=true name=acme-web-init filter=name,id
deploy virtualmachine name=acme-stage-1 zoneid=ZONE_ID templateid=TEMPLATE_ID serviceofferingid=SERVICE_OFFERING_ID networkids=NETWORK_ID keypairs=acme-key userdataid=USERDATA_ID account=acme-dev domainid=DOMAIN_ID filter=name,state,id
associate ipaddress networkid=NETWORK_ID filter=ipaddress,id
create firewallrule ipaddressid=IP_ID protocol=tcp startport=80 endport=80 cidrlist=0.0.0.0/0 filter=state
create portforwardingrule ipaddressid=IP_ID protocol=tcp publicport=80 privateport=80 virtualmachineid=VM_ID openfirewall=false filter=state
```

### video-23-acme-stage-sh

The whole ticket as a script. `./acme-stage.sh down` removes it, and any other argument builds it. Chapters "The ticket as a script", 6:15, and "Running it, down and up", 7:09.

```bash
#!/bin/bash
set -eo pipefail
cm() { cmk -o json "$@"; }
quiet() { local out; out=$(cm "$@") || { echo "$out"; return 1; }; }
idof() { local id; id=$(cm "$@" filter=id | jq -r '[.[] | arrays][0][0].id // empty')
  [ -n "$id" ] && echo "$id" || { echo "not found: $*" >&2; return 1; }; }
if [ "$1" = down ]; then
  NET=$(idof list networks listall=true name=acme-stage-net)
  VM=$(idof list virtualmachines listall=true name=acme-stage-1)
  quiet destroy virtualmachine id=$VM expunge=true
  IP=$(cm list publicipaddresses associatednetworkid=$NET listall=true \
    issourcenat=false | jq -r '.publicipaddress[0].id')
  quiet disassociate ipaddress id=$IP
  quiet delete network id=$NET
  echo "acme-stage removed"; exit
fi
ZONE=$(idof list zones name=zone-lon1)
DOM=$(idof list domains listall=true name=HostPro)
OWNER="account=acme-dev domainid=$DOM"
NETO=$(idof list networkofferings name=DefaultIsolatedNetworkOfferingWithSourceNatService)
IMG=$(idof list templates templatefilter=all listall=true name=acme-ubuntu-22.04)
SIZE=$(idof list serviceofferings name=nimbus-small)
INIT=$(idof list userdata listall=true name=acme-web-init)
NET=$(cm create network name=acme-stage-net displaytext=acme-stage-net zoneid=$ZONE \
  networkofferingid=$NETO gateway=10.3.0.1 netmask=255.255.255.0 $OWNER \
  | jq -r '.network.id')
echo "network $NET"
VM=$(cm deploy virtualmachine name=acme-stage-1 zoneid=$ZONE templateid=$IMG \
  serviceofferingid=$SIZE networkids=$NET keypairs=acme-key userdataid=$INIT $OWNER \
  | jq -r '.virtualmachine.id')
echo "instance $VM"
PUB=$(cm associate ipaddress networkid=$NET)
IPID=$(echo "$PUB" | jq -r '.ipaddress.id')
IP=$(echo "$PUB" | jq -r '.ipaddress.ipaddress')
quiet create firewallrule ipaddressid=$IPID protocol=tcp startport=80 endport=80 \
  cidrlist=0.0.0.0/0
quiet create portforwardingrule ipaddressid=$IPID protocol=tcp publicport=80 \
  privateport=80 virtualmachineid=$VM openfirewall=false
echo "acme-stage is at http://$IP/"
```

### video-23-find-a-command

Prints what each create command requires. Chapter "Finding a command you've never used", 7:41.

```bash
cmk create vpc -h | head -3
for t in vpc networkacllist networkacl loadbalancer loadbalancerrule affinitygroup snapshot autoscalevmgroup vpngateway vpnconnection; do echo "$t: $(cmk create $t -h | grep -m1 Required)"; done
```

## Video 24: dataco goes to production

### video-24-cmk-profile

Installs CloudMonkey, then at the `cmk` prompt sets dan's profile. Chapter "CloudMonkey on this lab", 3:06.

```bash
curl -sL -o /usr/local/bin/cmk https://github.com/apache/cloudstack-cloudmonkey/releases/latest/download/cmk.linux.x86-64
chmod +x /usr/local/bin/cmk
cmk
set profile dataco
set url http://MANAGEMENT_SERVER_IP:8080/client/api
set apikey YOUR_API_KEY
set secretkey YOUR_SECRET_KEY
sync
set output table
list zones filter=name,id
```

### video-24-vpc-and-web-tier

At the `cmk` prompt: the VPC, its first ACL rules, and the web tier. The CAPS names stand for your own IDs. Chapters "The VPC, by asking the tool", 3:52, and "Rules and the first tier", 4:38.

```text
list vpcofferings filter=name,id
create vpc name=dataco-vpc displaytext=dataco-vpc cidr=10.30.0.0/16 zoneid=ZONE_ID vpcofferingid=VPC_OFFERING_ID filter=name,cidr,state
create networkacllist name=dataco-web-acl description=dataco-web-acl vpcid=VPC_ID filter=name,id
create networkacl aclid=WEB_ACL_ID number=1 action=Allow protocol=tcp startport=80 endport=80 traffictype=Ingress cidrlist=0.0.0.0/0 filter=number,cidrlist,startport,state
create networkacl aclid=WEB_ACL_ID number=2 action=Allow protocol=tcp startport=22 endport=22 traffictype=Ingress cidrlist=MANAGEMENT_SERVER_IP/32 filter=number,cidrlist,startport,state
list networkofferings forvpc=true filter=name,id
create network name=dataco-web displaytext=dataco-web zoneid=ZONE_ID networkofferingid=TIER_OFFERING_ID gateway=10.30.1.1 netmask=255.255.255.0 vpcid=VPC_ID aclid=WEB_ACL_ID filter=name,cidr,state
```

### video-24-dataco-tiers-sh

Builds the app and database tiers with their ACLs. Run it with `chmod +x dataco-tiers.sh` then `./dataco-tiers.sh`. Chapter "The other tiers, as a script", 5:33.

```bash
#!/bin/bash
set -eo pipefail
cm() { cmk -o json "$@"; }
idof() { local id; id=$(cm "$@" filter=id | jq -r '[.[] | arrays][0][0].id // empty')
  [ -n "$id" ] && echo "$id" || { echo "not found: $*" >&2; return 1; }; }
VPC=$(idof list vpcs name=dataco-vpc)
ZONE=$(idof list zones name=zone-lon1)
PLAIN=$(idof list networkofferings name=DefaultIsolatedNetworkOfferingForVpcNetworks)
WITHLB=$(idof list networkofferings name=DefaultIsolatedNetworkOfferingForVpcNetworksWithInternalLB)
acl() { cm create networkacllist name=$1 description=$1 vpcid=$VPC | jq -r '.networkacllist.id'; }
allow() { cm create networkacl aclid=$1 number=$2 action=Allow protocol=tcp startport=$3 endport=$3 \
  traffictype=Ingress cidrlist=$4 >/dev/null; }
tier() { cm create network name=$1 displaytext=$1 zoneid=$ZONE networkofferingid=$2 gateway=$3 \
  netmask=255.255.255.0 vpcid=$VPC aclid=$4 >/dev/null; echo "tier $1 is ready"; }
# App: port 8080 from the web tier, and ssh from the web tier.
APP=$(acl dataco-app-acl)
allow $APP 1 8080 10.30.1.0/24
allow $APP 2 22 10.30.1.0/24
tier dataco-app $WITHLB 10.30.2.1 $APP
# Database: port 3306 from the app tier only, and ssh from the web tier.
DB=$(acl dataco-db-acl)
allow $DB 1 3306 10.30.2.0/24
allow $DB 2 22 10.30.1.0/24
tier dataco-db $PLAIN 10.30.3.1 $DB
```

### video-24-dataco-init-yaml

One user data file for every server. It picks a port from the server's name. Chapter "Key, user data, anti-affinity", 6:32.

```yaml
#cloud-config
write_files:
  - path: /usr/local/bin/dataco-page
    permissions: '0755'
    content: |
      #!/bin/sh
      case "$(hostname)" in
        dataco-app-*) PORT=8080 ;;
        dataco-db-*)  PORT=3306 ;;
        *)     PORT=80 ;;
      esac
      mkdir -p /srv/www
      echo "<h1>Hello from $(hostname)</h1>" > /srv/www/index.html
      exec python3 -m http.server $PORT --directory /srv/www
  - path: /etc/systemd/system/dataco-page.service
    content: |
      [Service]
      ExecStart=/usr/local/bin/dataco-page
      Restart=always
      [Install]
      WantedBy=multi-user.target
runcmd:
  - systemctl enable --now dataco-page
```

### video-24-key-user-data-affinity

Makes a key, registers the key and the user data, and creates the anti-affinity group. Chapter "Key, user data, anti-affinity", 6:32.

```bash
ssh-keygen -q -t ed25519 -N '' -C dataco-ops -f ~/.ssh/id_ed25519
echo "StrictHostKeyChecking accept-new" >> ~/.ssh/config
cmk register sshkeypair name=dataco-key publickey="$(cat ~/.ssh/id_ed25519.pub)" filter=name
cmk register userdata name=dataco-init userdata="$(base64 -w0 dataco-init.yaml)" filter=name,id
cmk create affinitygroup -h | head -3
cmk create affinitygroup name=dataco-web-spread type="host anti-affinity" filter=name,type
```

### video-24-first-web-server

At the `cmk` prompt: the first web server. Chapter "The first web server", 7:44.

```text
list templates templatefilter=self filter=name,id
deploy virtualmachine name=dataco-web-1 zoneid=ZONE_ID templateid=TEMPLATE_ID serviceofferingid=SERVICE_OFFERING_ID networkids=WEB_NETWORK_ID keypairs=dataco-key userdataid=USERDATA_ID affinitygroupnames=dataco-web-spread filter=name,state
```

### video-24-dataco-servers-sh

Deploys the other four servers. Run it with `chmod +x dataco-servers.sh` then `time ./dataco-servers.sh`. Chapter "The other servers, as a script", 8:19.

```bash
#!/bin/bash
set -eo pipefail
cm() { cmk -o json "$@"; }
idof() { local id; id=$(cm "$@" filter=id | jq -r '[.[] | arrays][0][0].id // empty')
  [ -n "$id" ] && echo "$id" || { echo "not found: $*" >&2; return 1; }; }
ZONE=$(idof list zones name=zone-lon1)
IMG=$(idof list templates templatefilter=self name=dataco-ubuntu-22.04)
SIZE=$(idof list serviceofferings name=nimbus-small)
INIT=$(idof list userdata name=dataco-init)
server() { local net; net=$(idof list networks name=$2)
  cm deploy virtualmachine name=$1 zoneid=$ZONE templateid=$IMG serviceofferingid=$SIZE \
    networkids=$net keypairs=dataco-key userdataid=$INIT $3 >/dev/null; echo "$1 is running"; }
server dataco-web-2 dataco-web affinitygroupnames=dataco-web-spread
server dataco-app-1 dataco-app
server dataco-app-2 dataco-app
server dataco-db-1 dataco-db
```

### video-24-public-load-balancer

At the `cmk` prompt: a public address, a load balancer rule, and both web servers behind it. Chapter "The public load balancer", 9:09.

```text
list vpcs filter=name,id
associate ipaddress vpcid=VPC_ID filter=ipaddress,id
create loadbalancerrule name=dataco-web-lb publicipid=LB_IP_ID publicport=80 privateport=80 algorithm=roundrobin networkid=WEB_NETWORK_ID openfirewall=false filter=name,publicip,state,id
assign toloadbalancerrule id=LB_RULE_ID virtualmachineids=WEB_1_ID,WEB_2_ID
```

### video-24-internal-load-balancer

At the `cmk` prompt: the internal load balancer for the app tier, and a port forward for SSH into web-1. Chapter "Internal load balancers", 9:53.

```text
create loadbalancer name=dataco-app-lb scheme=Internal algorithm=roundrobin sourceport=8080 instanceport=8080 networkid=APP_NETWORK_ID sourceipaddressnetworkid=APP_NETWORK_ID filter=name,sourceipaddress,id
assign toloadbalancerrule id=INTERNAL_LB_ID virtualmachineids=APP_1_ID,APP_2_ID
associate ipaddress vpcid=VPC_ID filter=ipaddress,id
create portforwardingrule ipaddressid=SSH_IP_ID protocol=tcp publicport=22 privateport=22 virtualmachineid=WEB_1_ID networkid=WEB_NETWORK_ID openfirewall=false filter=publicip,publicport,state
```

### video-24-test-the-rules

Tests the load balancer and ACLs from inside. The CAPS names stand for your own addresses. Chapter "Testing the rules", 11:01.

```bash
until ssh -o ConnectTimeout=5 ubuntu@WEB_1_PUBLIC_IP true 2>/dev/null; do sleep 5; done; ssh ubuntu@WEB_1_PUBLIC_IP hostname
ssh ubuntu@WEB_1_PUBLIC_IP curl -s http://INTERNAL_LB_IP:8080/
ssh ubuntu@WEB_1_PUBLIC_IP curl -s http://INTERNAL_LB_IP:8080/
ssh ubuntu@WEB_1_PUBLIC_IP curl -s -m 5 http://DB_IP:3306/; echo "curl said $?"
ssh -J ubuntu@WEB_1_PUBLIC_IP ubuntu@APP_IP curl -s http://DB_IP:3306/
```

### video-24-database-disk-and-snapshots

A data disk for the database server, checked from inside, with a snapshot policy and a first snapshot. The first two lines are at the `cmk` prompt. Chapter "The database's disk and snapshots", 12:01.

```bash
create volume name=dataco-db-data zoneid=ZONE_ID diskofferingid=DISK_OFFERING_ID filter=name,state,id
attach volume id=VOLUME_ID virtualmachineid=DB_1_ID filter=name,vmname,state
ssh -J ubuntu@WEB_1_PUBLIC_IP ubuntu@DB_IP lsblk -dno NAME,SIZE
cmk create snapshotpolicy -h | head -3
cmk create snapshotpolicy volumeid=VOLUME_ID intervaltype=daily schedule=00:02 maxsnaps=7 timezone=UTC filter=intervaltype,schedule,maxsnaps
cmk create snapshot volumeid=VOLUME_ID name=dataco-db-first filter=name,state
```

### video-24-dataco-office-sh

A second VPC stands in for the office, joined by a site-to-site VPN. Set your own shared secret. Run it with `chmod +x dataco-office.sh` then `time ./dataco-office.sh`. Chapter "The office link", 13:07.

```bash
#!/bin/bash
set -eo pipefail
cm() { cmk -o json "$@"; }
idof() { local id; id=$(cm "$@" filter=id | jq -r '[.[] | arrays][0][0].id // empty')
  [ -n "$id" ] && echo "$id" || { echo "not found: $*" >&2; return 1; }; }
ZONE=$(idof list zones name=zone-lon1)
OFFER=$(idof list vpcofferings name="Default VPC offering")
TIER=$(idof list networkofferings name=DefaultIsolatedNetworkOfferingForVpcNetworks)
ALLOW=$(idof list networkacllists name=default_allow)
IMG=$(idof list templates templatefilter=self name=dataco-ubuntu-22.04)
SIZE=$(idof list serviceofferings name=nimbus-small)
INIT=$(idof list userdata name=dataco-init)
# Stand-in for the office: its own VPC, one tier, one server.
MAIN=$(idof list vpcs name=dataco-vpc)
OFFICE=$(cm create vpc name=dataco-office displaytext=dataco-office cidr=10.40.0.0/16 zoneid=$ZONE \
  vpcofferingid=$OFFER | jq -r '.vpc.id')
NET=$(cm create network name=dataco-office-net displaytext=dataco-office-net zoneid=$ZONE \
  networkofferingid=$TIER gateway=10.40.1.1 netmask=255.255.255.0 vpcid=$OFFICE aclid=$ALLOW \
  | jq -r '.network.id')
cm deploy virtualmachine name=dataco-office-1 zoneid=$ZONE templateid=$IMG serviceofferingid=$SIZE \
  networkids=$NET keypairs=dataco-key userdataid=$INIT >/dev/null
echo "office is up"
# A gateway on each VPC, on its source NAT address.
GW1=$(cm create vpngateway vpcid=$MAIN | jq -r '.vpngateway.id')
GW2=$(cm create vpngateway vpcid=$OFFICE | jq -r '.vpngateway.id')
nat() { cm list publicipaddresses vpcid=$1 issourcenat=true | jq -r '.publicipaddress[0].ipaddress'; }
# A customer gateway describes the far end. The same secret and settings on both.
cgw() { cm create vpncustomergateway name=$1 gateway=$2 cidrlist=$3 ipsecpsk=YOUR_IPSEC_PSK \
  ikepolicy="aes256-sha256;modp2048" esppolicy=aes256-sha256 | jq -r '.vpncustomergateway.id'; }
TO_OFFICE=$(cgw dataco-office-site "$(nat $OFFICE)" 10.40.0.0/16)
TO_MAIN=$(cgw dataco-main-site "$(nat $MAIN)" 10.30.0.0/16)
# One end waits, the other starts the tunnel.
cm create vpnconnection s2svpngatewayid=$GW1 s2scustomergatewayid=$TO_OFFICE passive=true >/dev/null
cm create vpnconnection s2svpngatewayid=$GW2 s2scustomergatewayid=$TO_MAIN >/dev/null
echo "tunnel requested"
```

### video-24-test-the-tunnel

Checks the VPN, then fetches the office server's page across it. Chapter "The office link", 13:07.

```bash
cmk list vpnconnections filter=publicip,cidrlist,passive,state
ssh ubuntu@WEB_1_PUBLIC_IP curl -s -m 8 --retry 20 --retry-all-errors --retry-delay 5 http://OFFICE_1_IP/
```

### video-24-make-dataco-busy

Copies the file, adds the line that burns CPU for fifteen minutes, and registers it. For testing only. Chapter "Autoscale for the busy page", 14:33.

```bash
cp dataco-init.yaml dataco-busy.yaml
echo '  - nohup timeout 900 sh -c "while :; do :; done" >/dev/null 2>&1 &' >> dataco-busy.yaml
tail -3 dataco-busy.yaml
cmk register userdata name=dataco-busy userdata="$(base64 -w0 dataco-busy.yaml)" filter=name
```

### video-24-dataco-busy-yaml

The file those commands produce. On camera only its last three lines show. Chapter "Autoscale for the busy page", 14:33.

```yaml
#cloud-config
write_files:
  - path: /usr/local/bin/dataco-page
    permissions: '0755'
    content: |
      #!/bin/sh
      case "$(hostname)" in
        dataco-app-*) PORT=8080 ;;
        dataco-db-*)  PORT=3306 ;;
        *)     PORT=80 ;;
      esac
      mkdir -p /srv/www
      echo "<h1>Hello from $(hostname)</h1>" > /srv/www/index.html
      exec python3 -m http.server $PORT --directory /srv/www
  - path: /etc/systemd/system/dataco-page.service
    content: |
      [Service]
      ExecStart=/usr/local/bin/dataco-page
      Restart=always
      [Install]
      WantedBy=multi-user.target
runcmd:
  - systemctl enable --now dataco-page
  - nohup timeout 900 sh -c "while :; do :; done" >/dev/null 2>&1 &
```

### video-24-burst-load-balancer

A public address and load balancer rule for the autoscale group. Chapter "Autoscale for the busy page", 14:33.

```bash
cmk associate ipaddress vpcid=VPC_ID filter=ipaddress,id
cmk create loadbalancerrule name=dataco-burst-lb publicipid=BURST_IP_ID publicport=80 privateport=80 algorithm=roundrobin networkid=WEB_NETWORK_ID openfirewall=false filter=name,publicip,state
```
