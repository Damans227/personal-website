# Ubuntu vs Oracle Linux (EL10) - CloudStack Installation

| Aspect | Ubuntu | Oracle Linux (EL10) |
|--------|--------|---------------------|
| Package Manager | apt | dnf |
| Extra Repos | universe (usually enabled) | oracle-epel-release-el10 (install manually) |
| CloudStack Repo | .deb packages in /etc/apt/sources.list.d/ | .rpm packages in /etc/yum.repos.d/ |
| Networking | Netplan (YAML in /etc/netplan/) | NetworkManager (nmcli) |
| Bridge Setup | Edit YAML, run netplan apply | nmcli connection add/modify/up |
| NFS Package | nfs-kernel-server | nfs-utils |
| NFS Service | nfs-kernel-server | nfs-server |
| Security | AppArmor | SELinux |
| Disable Security | systemctl disable apparmor | setenforce 0 + edit config |
| Firewall | ufw or iptables | firewalld |
| Libvirt Listen Config | /etc/default/libvirtd | Systemd override |
