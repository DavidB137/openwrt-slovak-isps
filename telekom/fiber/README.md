# Telekom Fiber

GPON FTTH

## General information
You have to use Slovak Telekom-owned fiber converter/router Huawei HG8145V5 (as of writing this). It's not possible to use any other router with GPON port or to use SFP GPON module, at least nobody is known to succeed with this setup. But there are two options. You can use Huawei as a router with a monthly fee or you can ask Telekom to switch it into **bridge mode** (no monthly fee). In bridge mode, all routing functionalities of Huawei router are turned off and it functions just as a bridge between fiber and LAN interfaces. Own router can be connected to **LAN1** port (untagged).

### Protocols
**PPPoE** is used. There's **no support for IPv6 yet**. When ordering the service, you can choose between **public and private dynamic IPv4 address** (they are both without additional charge). You can also pay monthly fee for static IPv4 address.

Setup is pretty straightforward and any modern router can be used. Just setup PPPoE on WAN interface and IPv4 address should be received automatically.

MTU for PPPoE is 1492.
