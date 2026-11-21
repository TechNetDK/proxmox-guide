# 07 – Backup

> Uden backup er det ikke et spørgsmål om **om**, men **hvornår** du mister data.

## 1. Lav et sted til backup
LVM-thin kan ikke gemme backup. Lav en mappe-storage på datadisken:
```bash
mkdir -p /mnt/data/backup
pvesm add dir backup --path /mnt/data/backup --content backup
```

## 2. Backup af containerne (vzdump)
Datacenter → **Backup** → Add:
- Storage: `backup`
- Tidspunkt: fx hver nat kl. 03:00
- Mode: **Snapshot**
- Retention: **Keep Daily = 7**

Eller fra kommandolinjen:
```bash
vzdump --all 1 --storage backup --mode snapshot --compress zstd --prune-backups keep-daily=7
```

## 3. Få besked, hvis noget fejler
Datacenter → **Notifications** → tilføj din e-mail.

## 4. Gendan og test
En backup, du aldrig har gendannet, er ikke en backup. Gendan til et nyt ID (fx 900)
én gang om måneden og se, at den starter:
```bash
pct restore 900 backup:backup/vzdump-lxc-100-<dato>.tar.zst --storage local-lvm
```

## 5. Gem config-filerne
Gem kopier af disse filer et andet sted (fx et **privat** Git-repo – filerne indeholder IP'er):
- `/etc/network/interfaces`
- `/etc/pve/lxc/*.conf`
- `/etc/pve/storage.cfg`
- `/etc/pve/user.cfg`
- `/etc/fstab`
