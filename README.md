# EdgerouterX-CheatCheet
CheatCheet for cli setup of the infamous Ubiquiti Edgerouter X

WIP document follows... and assumes this setup:
P0 - Admin
P1-3 - LAN
P4 - WAN

## Basics

### Default admin
192.168.1.1/24 on Port 0. 
ssh username@192.168.1.1

### Getting info before entering configure mode:
```show interfaces
```

### Enter configure mode: 
```configure```

### Show interfaces
```show interfaces ethernet eth4```

### Identity & timezone:
```set system host-name edge-x```
```set system time-zone Europe/Oslo```

### Quick setup:
```
set Interfaces ethernet eth4 address dhcp
set Interfaces ethernet eth3 address 10.112.1.1/24
set Interfaces ethernet eth4 description "WAN"
set Interfaces ethernet eth3 description "LAN"
```

### LAN (eth1-eth3) as a switch (sw0), with gateway IP 192.168.10.1/24:
```
set interfaces switch switch0 description 'LAN'
set interfaces switch switch0 address 192.168.10.1/24
set interfaces switch switch0 mtu 1500
set interfaces switch switch0 switch-port interface eth1
set interfaces switch switch0 switch-port interface eth2
set interfaces switch switch0 switch-port interface eth3
```

### WAN (eth4): DHCP client + basic firewall:
```
set interfaces ethernet eth4 description 'WAN'
set interfaces ethernet eth4 duplex auto
set interfaces ethernet eth4 speed auto
set interfaces ethernet eth4 address dhcp
```

### Example 'LAN (eth1): static gateway IP:
```
set interfaces ethernet eth1 description 'LAN10'
set interfaces ethernet eth1 address 192.168.10.1/24
```

### Disable unused ports:
```set interfaces ethernet eth2 disable```

# Commit and saving:
```
commit
save
exit
```

as it is based on Debian Linux it supports things like:
```commit ;save```
```commit && save```

## Advanced

### DHCP server on LAN (ex. eth):
```
set service dhcp-server disabled false
set service dhcp-server shared-network-name LAN10 authoritative enable
set service dhcp-server shared-network-name LAN10 subnet 192.168.10.0/24 default-router 192.
set service dhcp-server shared-network-name LAN10 subnet 192.168.10.0/24 dns-server 192.168.
set service dhcp-server shared-network-name LAN10 subnet 192.168.10.0/24 lease 86400
set service dhcp-server shared-network-name LAN10 subnet 192.168.10.0/24 start 192.168.10.10


set service dhcp-server shared-network-name LAN subnet 192.168.10.0/24 default-router 192.168.10.1
set service dhcp-server shared-network-name LAN subnet 192.168.10.0/24 dns-server 192.168.10.1
set service dhcp-server shared-network-name LAN subnet 192.168.10.0/24 range 0 start 192.168.10.100
set service dhcp-server shared-network-name LAN subnet 192.168.10.0/24 range 0 stop 192.168.10.200
```

### Removing DCHP service
```
delete service dhcp-server shared-network-name LAN10
delete interfaces ethernet eth3 address dhcp
```

### DHCP relay (use instead of DHCP server when an external DHCP:
server is present)
### Remove/disable DHCP server for that LAN if previously enabled
### Relay requests arriving on LAN to upstream DHCP server
```
set service dhcp-relay interface eth1
set service dhcp-relay server 192.168.20.2
set service dhcp-relay relay-options hop-count 10
set service dhcp-relay relay-options max-size 576
```

### DNS forwarding (router as caching DNS for clients):
The router listens on LAN and forwards to upstream public resolvers; you can add your preferred
resolvers.
### Enable DNS forwarding (dnsmasq) and listen on LAN interface:
```set service dns forwarding cache-size 150```
```set service dns forwarding listen-on eth1```
### Forward to upstream resolvers (examples: Cloudflare and ):
```set service dns forwarding name-server 1.1.1.1```
```set service dns forwarding name-server 8.8.8.8```
### Forward local hostnames:
```set service dns forwarding system```

### Setting dnsmasq:
```set service dhcp-server use-dnsmasq enable```
```set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 domain-name ubnt.local```

### Setting a masquerade NAT Rule:
```
set service nat rule 5000 description "WAN masquerade"
set service nat rule 5000 outbound-interface eth4
set service nat rule 5000 type masquerade
```

### Minimal WAN firewall policy:
```
set firewall name WAN_LOCAL default-action drop
set firewall name WAN_LOCAL rule 10 action accept
set firewall name WAN_LOCAL rule 10 state established enable
set firewall name WAN_LOCAL rule 10 state related enable
set firewall name WAN_IN default-action drop
set firewall name WAN_IN rule 10 action accept
set firewall name WAN_IN rule 10 state established enable
set firewall name WAN_IN rule 10 state related enable
set interfaces ethernet eth4 firewall in name WAN_IN
set interfaces ethernet eth4 firewall local name WAN_LOCAL
```

### Hair-pin NAT, allowing devices on the same internal network to access other internal devices using the network's external IP address, useful when internal clients need to access services hosted on internal servers via their public domain names or external IPs:
```set port-forward hairpin-nat enable```

### Keeping the last 10 configuration commit revisions (by default none is kept):
```set system config-management commit-revisions 10```

### Rollback to specficic revision:
```rollback ? # lists commits```
```rollback {NUM}```

### Secure admin:
```set system login user admin authentication plaintext-password 'STRONG_PASSWORD'```
```delete system login user ubnt```

### Enable hardware offload for performance:
```set system offload hwnat enable```
```set system offload ipsec enable```

### Back up the config after commit via SSH/SCP
```set system config-management commit-archive location scp://user:pass@ip/Some/Path/To/Backups```

### Disable SIP ALG: 
```set system conntrack modules sip disable```

### Restart Crashed / Hanging Web GUI
1. Kill lighttpd process manually first, otherwise the delete delete service command hangs
```pkill -9 -f lighttpd```
2. Delete and re-add the GUI service
```
configure
delete service gui
commit
set service gui
commit
```



