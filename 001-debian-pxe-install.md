# Summary
Install Debian Linux on a new laptop using another laptop with Debian as a PXE server & network router

# Prerequisites

| devices involved        | IP                        |
|-------------------------|---------------------------|
| router (regular gateway)| 192.168.1.1               |
| server                  | 192.168.1.50, 192.168.42.2|
| client                  | 192.168.42.x              |

| server interfaces | name   | purpose                                           |
|-------------------|--------|---------------------------------------------------|
| WiFi              | wlo1   | provides internet                                 |
| Ethernet          | enp2s0 | creates a managed network that client connects to | 

# Steps

1. Configure dnsmasq adjusting [the official Debian guide](https://wiki.debian.org/PXEBootInstall#Simple_way_-_using_Dnsmasq).
Complete the guide but change `/etc/dnsmasq.conf` to:
```
interface=enp2s0
domain=pxeboot.local
dhcp-range=192.168.42.3,192.168.42.253,255.255.255.0,1h
dhcp-boot=pxelinux.0,pxeserver,192.168.42.2
enable-tftp
tftp-root=/srv/tftp
pxe-service=x86PC, "PXELINUX (BIOS)", "pxelinux.0"
pxe-service=X86-64_EFI,"PXE (UEFI)","grubx64.efi"

# other UEFI type identifier, see RFC4578 section-2.1

pxe-service=2, "PXELINUX (0002-EFI)", "grubx64.efi"
pxe-service=6, "PXELINUX (0006-EFI)", "grubx64.efi"
pxe-service=7, "PXELINUX (0007-EFI)", "grubx64.efi"
pxe-service=8, "PXELINUX (0008-EFI)", "grubx64.efi"
pxe-service=9, "PXELINUX (0009-EFI)", "grubx64.efi"
```

2. Remove the existing wired network profile (Settings GUI)
3. Set the static IP to the Ethernet interface
Add the following lines to the `/etc/network/interfaces`:
```
auto enp2s0
iface enp2s0 inet static
    address 192.168.42.2
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 127.0.0.1 1.1.1.1
```
4. Turn on IP Forwarding. Change `/etc/sysctl.conf`:
```
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```
5. Reboot
6. Configure NAT
```
iptables -t nat -A POSTROUTING -s 192.168.42.0/24 -o wlo1 -j MASQUERADE
```

Make the client boot using Network Mode by pressing F12 or whatever your button is to enter your boot menu
