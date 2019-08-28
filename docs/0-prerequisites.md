# Prerequisites

The helper node used in this document is a Fedora 29 x86_64 VM and it will use a regular user instead root user (with some exceptions)

If a regular user is not created:

```
useradd -m ocp
echo ocp:password | chpasswd
echo "ocp ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/ocp_nopassword
visudo -cf /etc/sudoers.d/ocp_nopassword
```

Allow rootless containers for the 'ocp' user:

```
sudo usermod --add-subuids 10000-75535 ocp
sudo usermod --add-subgids 10000-75535 ocp
```

Install podman and other required utils

```
dnf clean all
dnf install -y podman jq libguestfs-tools-c
dnf update -y
```

Disable all not needed interfaces in the host to avoid messing networking stuff with containers as well as IPv6 if not needed:

```
# As root
for interface in 'eno2' 'eth2' 'eth3'; do
  cat > /etc/sysconfig/network-scripts/ifcfg-${interface} << EOF
DEVICE=${interface}
BOOTPROTO=none
ONBOOT=no
NETWORKING_IPV6=no
IPV6_AUTOCONF=no
EOF
done

# Disable IPV6
cat > /etc/sysctl.d/ipv6.conf << EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF

sed -i \
  -e 's/IPV6_AUTOCONF.*/IPV6_AUTOCONF=no/g' \
  -e 's/IPV6INIT.*/IPV6INIT=no/g' \
  /etc/sysconfig/network-scripts/*

sysctl -p /etc/sysctl.d/ipv6.conf
# Disable virbr0
rm -f /etc/libvirt/qemu/networks/autostart/default.xml
systemctl stop libvirtd

nmcli connection reload
systemctl restart NetworkManager
# Or even better
# reboot
```

Configure your bootstrap/dns server

To configure your bootstrap/dns server with the data provided you can use the following commands:
```
# echo "The network interface to configure in this example is eno2 and will act as BOOTSTRAP and DNS SERVER"

# nmcli con mod eno2 ipv4.method manual
# nmcli con mod eno2 ipv4.addresses "${BOOTSTRAP_IP}/24, ${MY_IP}/24"
# nmcli con mod eno2 ipv4.gw4 ${GATEWAY}
# nmcli con up eno2 iface eno2
# nmcli con mod 'System eno1' +ipv4.dns "${DNS}"
# systemctl restart NetworkManager

```

Switch to the 'ocp' user created before:

```
su - ocp
```

Create an ssh key to be injected in the OCP hosts:

```
ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa
```

[<< Back to README](../README.md) | [Next: Variables >>](1-variables.md)
