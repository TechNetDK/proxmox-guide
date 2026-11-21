# 02 – Netværk og faste IP'er

Proxmox laver en **Linux-bridge** (`vmbr0`) oven på netkortet. Alle containere
sættes på bridgen og får deres egen IP på dit netværk – som om de var fysiske maskiner.

## /etc/network/interfaces (eksempel)
```ini
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.0.0.10/24
        gateway 10.0.0.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
```
Dit netkort kan hedde noget andet, fx `enp3s0` – se `ip -br a`.

Genindlæs uden genstart: `ifreload -a`

## To routere: adskil serveren fra hjemmenettet
Jeg bruger to routere:
- **Router 1** sidder bag ISP-routeren. Alle hjemmets enheder kobler sig på den.
- **Router 2** sidder bag router 1. Bag den står kun serveren, på sit eget net.

Fordelen: Bliver en enhed på hjemmenettet hacket, kan den ikke uden videre nå serveren.

Fordi router 2 laver NAT, kan PC'er på hjemmenettet ikke nå serveren direkte.
Administrer fra en PC på servernettet, eller lav en port-forward på router 2
for port 8006 (webgrænsefladen) og 22 (SSH) – kun til din egen PC.

## Gode vaner
- **Faste IP'er uden for routerens DHCP-område.** Fx DHCP 10.0.0.100–200, servere 10.0.0.10–20.
- **Lav en liste** over, hvilken IP hver container har.
- **Sæt DNS i hver container** (felt `nameserver`), så de ikke afhænger af værten.
- Slå unødige tjenester fra, fx `rpcbind` (port 111), hvis du ikke bruger NFS:
  ```bash
  systemctl disable --now rpcbind rpcbind.socket
  ```

## Se hvad der lytter
```bash
ss -tlnp        # åbne porte på værten
ip -br a        # interfaces og IP'er
```
