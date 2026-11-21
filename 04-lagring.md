# 04 – Lagring

## Standardopsætningen
| Storage | Type | Bruges til |
|---|---|---|
| `local` | mappe (`/var/lib/vz`) | ISO-filer, skabeloner, backup |
| `local-lvm` | LVM-thin | diske til containere og VM'er |

**Thin provisioning:** En disk på 100 GB fylder kun det, der faktisk er skrevet til den.
Derfor kan du "tildele" mere, end der er plads til.

**Vigtigt:** Bliver en thin-pool 100 % fuld, går **alle** containere på den ned. Tjek den jævnligt:
```bash
lvs    # se kolonnen Data%
pvesm status
```

## Genbrug en ekstra disk (fx gammel HDD)
Har disken en volume group fra en gammel installation, kan du tilføje den direkte:
```bash
vgs                                   # find navnet, fx pve-OLD-xxxx
pvesm add lvmthin old-storage --vgname pve-OLD-xxxx --thinpool data --content rootdir,images
```

## Ny tom disk som datadisk
```bash
lsblk                                 # find den rigtige disk først!
parted /dev/sdb --script mklabel gpt mkpart primary ext4 0% 100%
mkfs.ext4 /dev/sdb1
mkdir -p /mnt/data
blkid /dev/sdb1                       # find UUID
echo "UUID=<uuid> /mnt/data ext4 defaults,nofail 0 2" >> /etc/fstab
systemctl daemon-reload && mount -a
```
- Brug **altid UUID** i fstab, ikke `/dev/sdb1` – navnet kan skifte ved genstart.
- `nofail` gør, at serveren stadig starter, selvom disken mangler.

## Disk-sundhed
```bash
smartctl -H /dev/sda        # NVMe: /dev/nvme0n1
```
