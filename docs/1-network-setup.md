# 🖧 Network Setup & Machine Configuration

## Host Machines

The first step was making the two host machines communicate over a direct Ethernet connection. When pinging the two machines (`192.168.100.x /24`) we had trouble sending and receiving ICMP requests. We resolved this by disabling the Windows Defender firewall setting on both host machine Ethernet adapters.

> This is not a security concern since the firewall was only disabled for the direct LAN Ethernet connection between the two computers. All internet-facing adapters remained protected.

After successful host-to-host communication, we confirmed that all VMs could ping each other as well as the hosts — **6 devices total on the private LAN.**

---

## VM Specifications

### Attacker VM — KaliATTACK
| Field | Value |
|-------|-------|
| OS | Kali Linux |
| Hostname | KaliATTACK |
| Username | attacker1 |
| IP Address | 192.168.100.20 |

We assigned the attacker VM a static IP on the same subnet by editing `/etc/network/interfaces` as sudo and adding the address and netmask. We then restarted the network settings to confirm the IP was assigned.

![Attacker VM IP Config](../screenshots/blue-team/attacker-ip.png)
*Static IP successfully assigned to the attacker VM*

---

### SIEM VM — UbuSIEM
| Field | Value |
|-------|-------|
| OS | Ubuntu Desktop |
| Hostname | UbuSIEM |
| Username | ubusiem |
| IP Address | 192.168.100.11 |

We ensured both the NAT adapter and the Bridged Network adapter were properly configured. The bridged adapter was set to the correct subnet to allow communication with all devices on the LAN.

![SIEM VM Network Config](../screenshots/blue-team/siem-network.png)
*Bridged adapter configured on the correct subnet*

---

### Snort Server — snortubuntu1
| Field | Value |
|-------|-------|
| OS | Ubuntu Server |
| Hostname | snortubuntu1 |
| Username | snortserver |
| IP Address | 192.168.100.30 |

We configured the Ubuntu Server with a static IP and set up 2 NICs — one for management and one for Snort packet sniffing, both using bridged adapters. We also enabled OpenSSH to allow remote access from other machines.

![Snort Server NIC Config](../screenshots/blue-team/snort-nic.png)
*Successful NIC configuration and assigned LAN IP address*

---

### Kali Agent — KaliAgent1
| Field | Value |
|-------|-------|
| OS | Kali Linux |
| Hostname | KaliAgent1 |
| Username | kaliagent1 |
| IP Address | 192.168.100.10 |

We set a static IP by editing `/etc/network/interfaces`, then confirmed VM-to-VM communication on a single host before testing cross-host communication.

![Kali Agent IP Config](../screenshots/blue-team/kali-agent-ip.png)
*Static IP and ARP table confirmation*

---

### Windows Agent — WindowsAgent
| Field | Value |
|-------|-------|
| OS | Windows 10 |
| Hostname | WindowsAgent |
| Username | windowsagent1 |
| IP Address | 192.168.100.12 |

After launching the Windows VM we ran `ipconfig` to check the default IP, then assigned a static IP in the same subnet. We then pinged all 5 other devices and ran `arp -a` to confirm all devices were visible on the LAN.

![Windows Agent ARP Table](../screenshots/blue-team/windows-arp.png)
*ARP table showing all 6 devices on the LAN*
