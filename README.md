# How to create an IPsec VPN between Unifi USG and Mikrotik firewalls
* Hardware and software
    * Unifi USG XG-8 v4.4.44.5213871
    * Unifi Controller version 5.12.35
    * Mikrotik CCR1036-8G-2S+ with RouterOS v6.44.5
    
* Sources:
    * EdgeRouter to MikroTik IPSec VPN Setup by Willie Howe
        * https://www.youtube.com/watch?v=70EIA4bF820
    * UniFi - USG/UDM VPN: How to Configure Site-to-Site VPN
        * https://help.ubnt.com/hc/en-us/articles/360002426234-UniFi-USG-UDM-VPN-How-to-Configure-Site-to-Site-VPN
        
## Mikrotik configuration in WebFig interface
### Select: IP -> IPsec -> Peers

| Add New | |
| - | - |
| Enabled | checked
| Name | USG-01
| Address |	USG WAN address
| Port | empty
| Local Address | Mikrotik WAN address
| Auth. Method | pre shared key
| Exchange Mode | main
| Passive | uncheked
| Send INITIAL_CONTACT | checked

### Select: IP -> IPsec -> Profiles
| New Profile | |
| - | - |
| Name | profile01
| Hash Algorithms | sha1
| Encryption Algorithm | aes-128
| DH Group | modp1024 (=DH Group 2), modp2048
| Proposal Check | obey
| Lifetime | 1d 00:00:00
| Lifebytes | empty
| Nat Traversal | uncheck
| DPD Interval | 60s
| DPD Maximum Failures | 5

### Select: IP -> IPsec -> Identities
| Add New | |
| - | - |
| Enabled |	checked
| Peer | USG-01
| Auth. Method |			pre shared key
| Secret | paste your secret
| Policy Template Group | default
| Notrack Chain | empty
| My ID Type | auto
| Remote ID Type | auto
| Match By | remote id
| Mode Configuration | empty
| Generate Policy | no

### Select: IP -> IPsec -> Proposals

| Check default | |
| - | - |
| Enabled | checked
| Name | default
| Auth. Algorithms | sha1
| Encr. Algorithms | aes-128 cbc
| | aes-256 cbc
| | aes-128 ctr
| Lifetime | empty
| PFS Group | modp1024

### Select: IP -> IPsec -> Policies
* Disable default

| Add New IPsec Policy | |
| - | - |
| Enabled | checked |
| Src. Address | Mikrotik internal LAN network address (the whole network e.g. 192.168.1.0/24) |
| Src. Port | empty
| Dst. Address | USG internal LAN network address
| Dst. Port | empty
| Protocol | 255 (all)
| Action | encrypt
| Level | require
| IPsec Protocols | esp
| Tunnel | checked
| SA Src. Address | Mikrotik WAN address
| SA Dst. Address | USG WAN address
| Proposal | default

### Select: IP -> Firewall -> NAT
| New NAT Rule | |
| - | - |
| Chain | srcnat
| Src. Address | LAN network of Mikrotik
| Dst. Address | LAN network of USG
| Action | Accept

* Move the rule to the top of the firewall rules.

## USG configuration (version 5.12.35)
### Settings -> VPN -> Create New VPN Connection
| Configurations | |
| - | - |
| Name| mikrotik-usg-vpn |
| Purpose | Site-to-Site VPN |
| VPN Type | Manual IPsec |
| Enabled | check, Enable this Site-to-Site VPN |
| Remote Subnets | Mikrotik subnets |
| Route Distance | 30 was default | 
| Peer IP | WAN IP of Mikrotik |
| Local WAN IP | WAN IP of USG |
| Pre-Shared Key | secret key |
| IPsec Profile | Customized | 
| **Advanced Options** | |
| Key Exchange Version | IKEv1
| Encryption | AES-128
| Hash | SHA1
| DH Group | 2
| PFS | checked, Enable prefect forward secrecy
| Dynamic Routing | unchecked, Enable dynamic routing

## Mikrotik IPsec -> Installed SAs
* Something like this should show up when connection is up

| | SPI	| Src. Address | Dst. Address |Auth. Algorithm | Encr. Algorithm | Encr. Key Size | Current Bytes
| - | - | - | - | - | - | - | - |
| EH | 4cbfd50 | 62.x.y.z | 83.x.y.z | sha1 | aes cbc | 128 | 15540	
| EH | c0a27199 | 83.x.y.z | 62.x.y.z | sha1 | aes cbc | 128 | 13440

## Ping
* You should be able to ping both ways now.
