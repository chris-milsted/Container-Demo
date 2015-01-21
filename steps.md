# Steps to create VM

I am using a fedora 21 machine with KVM to run this, feel free to use a RHEL server or similar.
Also I have downloaded the RHEL 7.1 beta ISO from here:

https://www.redhat.com/en/about/blog/red-hat-enterprise-linux-71-beta-now-available

```
virt-install --name rhel7-template --ram 4000 --file /var/lib/libvirt/images/rhel7-template.img --vcpu 2 \
                --cdrom /tmp/rhel7.iso --noautoconsole --cpu host \
                --vnc --file-size 20 \ --os-variant rhel7 \
                --network network:default
```
