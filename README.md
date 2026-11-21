# Proxmox-guide: fra tom PC til homelab

En dansk trin-for-trin-guide til at sætte en lille Proxmox-server op derhjemme.
Den bygger på mit eget homelab, men alle navne og IP-adresser er eksempler.
Guiden er skrevet til **Proxmox VE 9** (Debian 13 "trixie").

## Sådan er netværket bygget op

```
Internet
   │
ISP-router
   │
Router 1  ── hjemmenetværk (PC'er, telefoner, TV)
   │
Router 2  ── servernetværk
   │
Proxmox-vært
```

Serveren står på sit **eget netværk** bag router 2, adskilt fra resten af huset (se kapitel 02).

## Hardware jeg bruger

| Del | Det jeg har |
|---|---|
| CPU | Intel i5 (6 kerner, VT-x og VT-d) |
| RAM | 16 GB |
| Systemdisk | NVMe 128 GB |
| Datadisk | HDD 1 TB |
| GPU | NVIDIA GTX 1060 (valgfri) |

## Indhold

| # | Kapitel |
|---|---|
| 01 | [Installation](01-installation.md) |
| 02 | [Netværk og faste IP'er](02-netvaerk.md) |
| 03 | [Containere og VM'er](03-containere.md) |
| 04 | [Lagring: LVM-thin og ekstra diske](04-lagring.md) |
| 05 | [NVIDIA-GPU i en LXC-container](05-gpu-i-lxc.md) |
| 06 | [Sikkerhed](06-sikkerhed.md) |
| 07 | [Backup](07-backup.md) |
| 08 | [Fejl jeg har ramt, og løsningerne](08-fejl-og-loesninger.md) |

## Før du starter

- Slå **VT-x** til i BIOS (og **VT-d**, hvis du vil give et grafikkort direkte til en VM).
- Hav en USB-nøgle på mindst 8 GB.
- Bestem på forhånd, hvilke faste IP'er serveren og containerne skal have.
