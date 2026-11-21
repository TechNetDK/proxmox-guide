# 03 – Containere og VM'er

Jeg bruger mest **LXC-containere**. De bruger langt mindre RAM og starter på sekunder.
Brug en **VM**, hvis du har brug for et andet styresystem (fx Windows) eller din egen kerne.

## Hent en skabelon
```bash
pveam update
pveam available | grep ubuntu
pveam download local ubuntu-<version>-standard_<version>_amd64.tar.zst
```

## Opret en container
```bash
pct create 100 local:vztmpl/ubuntu-<version>-standard_<version>_amd64.tar.zst \
  --hostname ct-test --ostype ubuntu --unprivileged 1 \
  --cores 2 --memory 2048 --swap 1024 --rootfs local-lvm:16 \
  --net0 name=eth0,bridge=vmbr0,ip=10.0.0.12/24,gw=10.0.0.1 \
  --nameserver 1.1.1.1 --features nesting=1 \
  --onboot 1
pct start 100
pct enter 100
```

| Valg | Hvorfor |
|---|---|
| `--unprivileged 1` | Root i containeren er **ikke** root på værten. Brug altid det. |
| `--features nesting=1` | Giver bedre kompatibilitet med nyere systemd. Proxmox anbefaler det. |
| `--onboot 1` | Starter automatisk, når serveren booter. |

## Del en mappe fra værten (bind-mount)
Fx en datamappe på værten, som containeren skal kunne se:
```bash
pct set 100 -mp0 /mnt/data,mp=/mnt/data
```
I en unprivileged container er bruger 1000 inde i containeren = **101000** på værten:
```bash
chown -R 101000:101000 /mnt/data   # så bruger 1000 i containeren ejer mappen
```

## Snapshot før ændringer
Tag altid et snapshot, før du opdaterer eller ændrer noget stort:
```bash
pct snapshot 100 foer-opdatering
pct rollback 100 foer-opdatering   # hvis noget gik galt
```

## Opret en VM
1. Upload en ISO: **local → ISO Images → Upload**.
2. Klik **Create VM** og vælg ISO'en.
3. Disk på `local-lvm`, CPU-type **host**, og netkort på `vmbr0`.
4. Start VM'en og installer styresystemet via **Console**.

## Pas på med overbooking
Du kan give containerne flere kerner og mere RAM, end serveren har.
Det går fint, så længe de ikke bruger det hele samtidig. Hold øje med swap:
```bash
free -h
```
