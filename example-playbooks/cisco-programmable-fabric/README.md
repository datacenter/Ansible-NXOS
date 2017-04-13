# cisco-programmable-fabric
Ansible playbooks to configure the Cisco Programmable Fabric VXLAN solution that uses MP-iBGP EVPN, OSPF, IP Unnumbered, and PIM SM in the Underlay. Spines act as route-reflectors and PIM Anycast RPs.

* [Cisco Programmable Fabric with VXLAN BGP EVPN Configuration Guide](http://www.cisco.com/c/en/us/td/docs/switches/datacenter/pf/configuration/guide/b-pf-configuration.html)

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
git clone https://github.com/mtarking/cisco-programmable-fabric
```

## Option 2:

Using the Dockerfile:

```
docker build -t cisco-programmable-fabric .
```
```
docker run -it cisco-programmable-fabric
```

## Option 3:

Pull directly from DockerHub:

```
docker pull mtarking/cisco-programmable-fabric
```
```
docker run -it mtarking/cisco-programmable-fabric
```
