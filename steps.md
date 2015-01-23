# Steps to create VM

I am using a fedora 21 machine with KVM to run this, feel free to use a RHEL server or similar.
Also I have downloaded the RHEL 7.1 beta ISO from here:

https://www.redhat.com/en/about/blog/red-hat-enterprise-linux-71-beta-now-available

```
sudo virt-install --name rhel71-pcp --ram 2048 --file /var/lib/libvirt/images/rhel71-pcp.qcow2 --vcpu 2 --cdrom /home/fab/cmilsted/Downloads/rhel-server-7.1-beta-1-x86_64-dvd.iso --noautoconsole --cpu host --vnc --file-size 6 --os-variant "rhel7" --network network:default
```
Then connect to the console and start the install:

```
sudo virt-viewer rhel71-pcp
.....
reboot button
virt-viewer rhel71-pcp

```
The anaconda file responses are here for your information - it is just a minimal install for now:

```

#version=RHEL7
# System authorization information
auth --enableshadow --passalgo=sha512

# Use CDROM installation media
cdrom
# Use graphical install
graphical
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=vda
# Keyboard layouts
keyboard --vckeymap=gb --xlayouts='gb'
# System language
lang en_GB.UTF-8

# Network information
network  --bootproto=dhcp --device=eth0 --ipv6=auto
network  --hostname=localhost.localdomain
repo --name="Server-HighAvailability" --baseurl=file:///run/install/repo/addons/HighAvailability
repo --name="Server-ResilientStorage" --baseurl=file:///run/install/repo/addons/ResilientStorage
# Root password
rootpw --iscrypted $6$2fmvi0E8/VOf0pGB$3NCYolz.hV0BHNM4UjpY.juqh35/iuAT0RepD5s5jFEDl1TpGsTZULBVTryXQJxynSiGqt7yD1QL3M2bGB6au0
# System timezone
timezone Europe/London --isUtc
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=vda
autopart --type=lvm
# Partition clearing information
clearpart --none --initlabel 

%packages
@core
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end
```

# Register the system and install docker and docker-registry

```
subscription-manager register
```

Enter your portal username and password and this should attach the correct subscription and then we need to enable the repos.

``` 
subscription-manager attach --pool={pool id}
subscription-manager repos --enable=rhel-7-server-extras-htb-rpms
subscription-manager repos --enable=rhel-7-server-optional-htb-rpms
```
Now we need to install docker and the docker registry...

```
yum update -y
yum install docker docker-registry -y

```

So note that we are docker version 1.1.2-9 here in RHEL 7 - expect this to increment as the upstream version is 1.4.1 today. Lots of changes in this area too many developments to talk about.

In the interim lets just play around with the following:

1. Pull down a RHEL container.
2. Install and commit http server and show how we can spin a couple of these up
3. Have a play with privileged containers and talk about tooling.

# Pull down a RHEL container.

1. start docker
```
systemctl start docker.service
```


Here is the dockerfile

```

# My cool Docker image
# Version 1

# If you loaded redhat-rhel-server-7.0-x86_64 to your local registry, uncomment this FROM line instead:
# FROM registry.access.redhat.com/rhel 
# Pull the rhel image from the local repository
FROM registry.access.redhat.com/rhel 

MAINTAINER Chris Negus 

# Update image
RUN yum update -y || true
RUN yum install httpd -y
RUN yum clean all
RUN mkdir /run/httpd ||true

# Create an index.html file
RUN bash -c 'echo "Your Web server test is successful." >> /var/www/html/index.html' 

```




--net=host

