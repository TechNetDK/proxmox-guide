# 05 – NVIDIA-GPU i en LXC-container

Målet: give en **unprivileged LXC-container** adgang til et NVIDIA-grafikkort.

## VM eller container?
| | GPU til en **VM** (VFIO) | GPU til en **container** |
|---|---|---|
| Driver på værten | `vfio-pci` (værten rører ikke kortet) | `nvidia` (værten ejer kortet) |
| Deling | Kun én VM | Kan deles mellem containere |
| Typisk brug | Windows, gaming | CUDA-beregning, video-transkodning |

Jeg startede med VFIO, men skiftede til containeren. Det er det, guiden viser.

## 1. (Kun til VFIO) Slå IOMMU til
Spring over, hvis GPU'en skal i en container.
`/etc/default/grub`:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```
```bash
update-grub && reboot
```
(Gælder ext4-installation med GRUB. Ved ZFS/systemd-boot: `/etc/kernel/cmdline` og `proxmox-boot-tool refresh`.)

## 2. Installer NVIDIA-driveren på værten
```bash
apt install proxmox-headers-$(uname -r) dkms build-essential
chmod +x NVIDIA-Linux-x86_64-<version>.run
./NVIDIA-Linux-x86_64-<version>.run --dkms
nvidia-smi        # skal vise kortet
```
- **DKMS** bygger driveren igen, når kernen opdateres.
- Har du **Secure Boot** slået til, skal modulet **signeres**. Installeren kan lave en
  nøgle, som du derefter godkender ved næste boot (MOK-skærmen).
- **Ældre kort (fx GTX 1060):** Brug 580-serien. Nyere drivere understøtter dem ikke.
  En helt ny kerne kan kræve en nyere 580-udgave eller en ældre kerne (se kapitel 08).

## 3. Giv containeren adgang
Proxmox kan selv give containeren enhederne (her container 100):
```bash
pct set 100 --dev0 /dev/nvidia0 --dev1 /dev/nvidiactl --dev2 /dev/nvidia-uvm \
  --dev3 /dev/nvidia-uvm-tools --dev4 /dev/nvidia-modeset
pct reboot 100
```

> **Vigtigt:** `/dev/nvidia-uvm` findes ikke altid efter en genstart, og så vil containeren ikke starte.
> Sørg for, at den laves ved boot:
> ```bash
> echo "nvidia-uvm" > /etc/modules-load.d/nvidia-uvm.conf
> echo 'KERNEL=="nvidia_uvm", RUN+="/usr/bin/nvidia-modprobe -c0 -u"' > /etc/udev/rules.d/70-nvidia-uvm.rules
> ```

## 4. Samme driverversion i containeren
Installer **præcis samme** driverversion i containeren, men **uden** kernemodul:
```bash
chmod +x NVIDIA-Linux-x86_64-<version>.run
./NVIDIA-Linux-x86_64-<version>.run --no-kernel-module
nvidia-smi
```
> Afviger versionerne, virker `nvidia-smi` ikke. Opdaterer du driveren på værten,
> skal containeren opdateres samtidig.
