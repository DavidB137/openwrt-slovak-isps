# Orange Fiber Partner

GPON FTTH using Slovak Telekom infrastructure.

## General information

### VLAN
Connection is terminated on Slovak Telekom-owned fiber converter/router Huawei HG8145V5 (as of writing this). There are 4 LAN ports available on this router:
- **LAN1**: internet untagged
- **LAN2 - LAN4**: internet VLAN 2510, IPTV VLAN 2513 (probably VoIP VLAN 2512, management VLAN 2517)

### Protocols
Orange uses PPPoE. If you have service with IPv4 address, you get public dynamic (static if you pay a monthly fee) IPv4 address directly. If you have service with IPv6 address, you get private IPv6 address from `fe80::/10` subnet. Then you have to run DHCPv6 client over PPPoE interface to receive public dynamic /56 IPv6 subnet (prefix delegation). There is no single IPv6 address (/128) received, just the prefix. For IPv4 communication, there's [DS-Lite](https://en.wikipedia.org/wiki/IPv6_transition_mechanism#Dual-Stack_Lite_(DS-Lite)). DS-Lite AFTR server hostname is part of DHCPv6 reply, but if nothing changes, it will be `aftr1.auro.orange.sk` or `aftr2.auro.orange.sk`.

MTU for PPPoE is 1492. MTU of DS-Lite tunnel should be set to 1452 (PPPoE - IPv6 header (40 bytes)).

---

## Links
- https://malt3.medium.com/how-to-survive-despite-having-dual-stack-lite-44342c794707
- https://www.orange.sk/uploads/tx_oskdeviceinfo/zyxel-vmg3927-t50k-2020-11_01.pdf
- https://www.orange.sk/uploads/tx_oskdeviceinfo/TPLINK-WDR3600.pdf
- https://www.telekom.sk/info/dokumenty/referencna-ponuka-na-virtualny-lokalny-uvolneny-pristup
