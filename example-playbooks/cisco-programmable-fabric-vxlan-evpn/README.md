# cisco-nxos-ansible-vxlan-evpn
Ansible playbooks to configure MP-BGP EVPN VXLAN using IP Unnumbered with OSPF and PIM SM in the Underlay and iBGP EVPN as the control plane. Spines act as route-reflectors and PIM Anycast RPs.

Contact(s):
* Matt Tarkington (mtarking@cisco.com)

## Option 1:

Using a CentOS/RHEL machine:

```
yum -y install epel-release
```

```
yum -y install git ansible openssh-clients
```

```
cd /WORKDIR # your working directory where you want the clone of the repo to reside
```

```
git clone https://github.com/mtarking/cisco-nxos-ansible-vxlan-evpn
```

## Option 2:

Using the Dockerfile:

```
docker build -t cisco-nxos-ansible-vxlan-evpn .
```
```
docker run -it cisco-nxos-ansible-vxlan-evpn
```
