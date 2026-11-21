# 06 – Sikkerhed

Grundlæggende sikring af Proxmox-værten.

## SSH med nøgler i stedet for password
```bash
# På din PC
ssh-keygen -t ed25519
ssh-copy-id root@<server-ip>

# På serveren: /etc/ssh/sshd_config
PermitRootLogin prohibit-password
PasswordAuthentication no
```
```bash
systemctl restart ssh
```
> Test i et **nyt** vindue, at du stadig kan logge ind, før du lukker det gamle.

## Tofaktor-login på webgrænsefladen
Datacenter → Permissions → **Two Factor** → tilføj TOTP (fx Google Authenticator).

## Egen bruger til daglig drift
```bash
pveum user add admin@pve
pveum passwd admin@pve
pveum acl modify / --users admin@pve --roles PVEAdmin
```
Log derefter ind som `admin` med realm **Proxmox VE authentication server**.

## Firewall
- Slå Proxmox-firewallen til i **Datacenter → Firewall**. Lav først en regel, der tillader
  port **8006** og **22** fra din admin-PC, så du ikke låser dig selv ude.
- Firewall på containere: vælg containeren → **Firewall** → slå til og tilføj kun de porte,
  du skal bruge (og sæt `firewall=1` på netkortet).
