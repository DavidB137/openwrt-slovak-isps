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

## How to setup on OpenWrt

All text below applies to **IPv6 address type**. If you have IPv4 address type, you will just need to setup PPPoE. There are no additional packages needed and it's completely straightforward.

### Requirements
You will need `ds-lite` package. If you want to configure all of this in LuCi, you will need `luci-proto-ipv6` package:

```sh
opkg update
opkg install ds-lite luci-proto-ipv6
```

### Setup
PPPoE with DS-Lite on OpenWrt is pretty simple. Create `wan` interface, set it's protocol to PPPoE, firewall zone to `wan` and enter username and password. Make sure option "Obtain IPv6 address" is set to "Automatic" and save changes. If everything goes right, there will be `wan` interface (with private IPv6 address assigned) and 2 virtual interfaces created:
- `wan_6`: DHCPv6 protocol. Receives prefix delegation.
- `wan_6_4`: DS-Lite. Tunnel to Orange's AFTR server (it's hostname is configured from DHCPv6 reply).

This is the case for **OpenWrt v22.03.0-rc6 and newer**. Previously, [`odhcp6c` package contained a bug](https://git.openwrt.org/?p=project/odhcp6c.git;a=commit;h=9212bfcbab7681cd30186ac7f1ef4c47bf38c89a) which made it impossible to get DHCPv6 prefix delegation from Orange. This should be fixed now, but if you are running v19.x or v21.x, you will have to upgrade or compile `odhcp6c` package with this bug fix yourself.

#### Example configuration
**`/etc/config/network`**:
```
config interface 'wan'
    option proto 'pppoe'
    option username 'someone@orangenet.sk'
    option password 'ABCDEF'
    option device 'eth0.3'
    option ipv6 'auto'
```

**`/etc/config/firewall`**:
```
config zone
    option name 'wan'
    # ...
    list network 'wan'
```

#### Port forwarding
If you need to open ports on public IPv4 address (one you share with others on DS-Lite AFTR server), you can use PCP. [Orange allows "port forwarding" on ports 1025 - 2047.](https://www.orange.sk/uploads/tx_oskdeviceinfo/port-forward-ipv6-huawei-hg8245u.pdf) OpenWrt doesn't currently support PCP (at least I don't how about it), but you can cross compile [libpcp](https://github.com/libpcp/pcp) and use `pcp` app. I can confirm it works, but it's a little bit complicated. I'm planning to create a separate writeup on this topic later.

#### Additional scripts
There are some caveats, however.

1. **Too low MTU on DS-Lite tunnel**

   By default, DS-Lite tunnel has MTU of 1280. This is too low as PPPoE interface has MTU of 1492.
   The solution is to create a simple hotplug script:

   **`/etc/hotplug.d/iface/01-dslite`**:
   ```sh
   #!/bin/sh

   MTU=1452
   
   if [ "$ACTION" = ifup ] && [ "$INTERFACE" = wan_6_4 ] ; then
    sleep 1
    logger "Set MTU on ds-lite to $MTU"
    ip link set ds-wan_6_4 mtu $MTU
   fi
   ```

2. **DHCPv6-PD renewal**
   
   `odhcp6c` renews prefix delegation when valid lifetime expires (1 day). After this time, however, Orange considers this prefix expired and assignes you another prefix. That isn't very desirable, as this breaks DS-Lite tunnel (local IPv6 address changes, but it's not handled by anything) and IP addresses of all your downstream devices also change. Workaround is to periodically (e.g. every 4 hours) send renew requests. This can be done by sending `SIGUSR1` signal to `odhcp6c`:

   **`/root/wan-dhcpv6-renew.sh`**:
   ```sh
   #!/bin/sh

   . /lib/netifd/netifd-proto.sh

   IFACE=wan_6
   proto_kill_command "$IFACE" $(kill -l SIGUSR1) && logger "Renewed DHCPv6 PD on $IFACE"
   ```

   And appending one line to CRON configuration:
   **`crontab -e`**:
   ```
   0 */4 * * * /root/wan-dhcpv6-renew.sh
   ```

   Of course, you can place the script anywhere else.

Don't forget to `chmod +x` all scripts. :)

### Dumps
- [DHCPv6 advertise packet (including PD)](packetdump-dhcpv6-advertise.txt)

---

## Links
- https://malt3.medium.com/how-to-survive-despite-having-dual-stack-lite-44342c794707
- https://www.orange.sk/uploads/tx_oskdeviceinfo/zyxel-vmg3927-t50k-2020-11_01.pdf
- https://www.orange.sk/uploads/tx_oskdeviceinfo/TPLINK-WDR3600.pdf
- https://www.telekom.sk/info/dokumenty/referencna-ponuka-na-virtualny-lokalny-uvolneny-pristup
