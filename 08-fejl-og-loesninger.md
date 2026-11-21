# 08 – Fejl jeg har ramt, og løsningerne

| Problem | Årsag | Løsning |
|---|---|---|
| NVIDIA-modul vil ikke indlæses | **Secure Boot** afviser usigneret modul | Installer med `--dkms`, signer modulet og godkend nøglen i MOK ved boot |
| Driveren bygger ikke på den nyeste kerne | Ny kerne understøttes ikke af driveren endnu | `proxmox-boot-tool kernel list`, derefter `proxmox-boot-tool kernel pin <version>` (fjern igen med `kernel unpin`) |
| `nvidia-smi` virker på værten, men ikke i containeren | Forskellige driverversioner | Samme version i begge (containeren med `--no-kernel-module`) |
| Containeren vil ikke starte efter genstart | `/dev/nvidia-uvm` findes ikke endnu | Se kapitel 05, afsnit 3 |
| GPU forsvinder fra værten (hvis du tidligere har prøvet VFIO) | Kortet er bundet til `vfio-pci` | Fjern `options vfio-pci ids=...` fra `/etc/modprobe.d/` og kør `update-initramfs -u` |
| "Permission denied" på delt mappe | UID-forskydning i unprivileged container | `chown -R 101000:101000 <mappe>` på værten (= 1000 i containeren) |
| Containeren er langsom/hænger | Disken næsten fuld | `df -h` i containeren, `pct resize <id> rootfs +10G` på værten |
| Datadisk mangler efter genstart | `/dev/sdX` skiftede navn | Brug **UUID** i `/etc/fstab` |
| `apt update` giver fejl 401 | Enterprise-repo er slået til uden abonnement | Slå det fra (se kapitel 01) |

## Nyttige kommandoer
```bash
pct list                 # containere
pct config 100           # en containers opsætning
journalctl -b -p err     # fejl siden sidste boot
dmesg | grep -i nvidia   # GPU-meddelelser
lvs                      # thin-pool forbrug
```
