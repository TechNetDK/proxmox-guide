# 01 – Installation

## 1. Lav en USB-nøgle
1. Hent Proxmox VE ISO fra proxmox.com.
2. Skriv den til USB med **balenaEtcher** eller **Rufus** (DD-tilstand).

## 2. Installer
1. Boot fra USB og vælg **Install Proxmox VE (Graphical)**.
2. Vælg systemdisken (NVMe). Filsystem: **ext4** er fint til én disk.
   Installeren laver selv: EFI-partition, swap, `root` og en **LVM-thin** pool til gæsterne.
3. Land, tidszone og tastatur: Danmark, `Europe/Copenhagen`, `da`.
4. Sæt en stærk root-adgangskode og en e-mailadresse (bruges til advarsler).
5. Netværk: værtsnavn (fx `pve.home.arpa`), fast IP, gateway (routeren) og DNS.

> **Tip:** Ligger der allerede en gammel Proxmox på en anden disk, omdøber installeren
> den gamle volume group til `pve-OLD-xxxx` i stedet for at slette den. Så kan du
> genbruge disken bagefter (se kapitel 04).

## 3. Første login
Åbn `https://<server-ip>:8006` og log ind som `root`.

Du får beskeden **"No valid subscription"**. Den er normal uden abonnement og kan bare lukkes med OK.

## 4. Gratis opdateringer (uden abonnement)
Proxmox 9 bruger `.sources`-filer. Slå enterprise-repoerne fra og tilføj no-subscription:
```bash
echo "Enabled: no" >> /etc/apt/sources.list.d/pve-enterprise.sources
echo "Enabled: no" >> /etc/apt/sources.list.d/ceph.sources

cat > /etc/apt/sources.list.d/proxmox.sources <<'EOT'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOT

apt update && apt full-upgrade -y
reboot    # hvis der kom en ny kerne
```
Det samme kan gøres i web-UI'et: **Node → Updates → Repositories**.

## 5. Tjek at alt er ok
```bash
pveversion -v      # version
uname -r           # kørende kerne
timedatectl        # skal vise Europe/Copenhagen og "System clock synchronized: yes"
mokutil --sb-state # er Secure Boot slået til?
```
