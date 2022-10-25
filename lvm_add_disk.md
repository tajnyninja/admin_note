# Dodanie i rozszerzenie partycji z LVM2

## Wstęp

Kilka terminów, które trzeba znać podczas korzystania z LVM:

- **Physical Volume (PV)** - Wolumen fizyczny, składa się z surowych dysków, macierzy itp.
- **Volume Group (VG)** - Grupa woluminów, łączy woluminy fizyczne w grupy pamięci masowej.
- **Logical Volume (LV)** - Wolumin logiczny

## Gdy brakuje miejsca na dysku

### Przygotowanie dysku

Sprawdzamy ile pozostało miejsca na dysku maszyny wirtualnej:

```
root@shinobi:~# df -h
System plików               rozm. użyte dost. %uż. zamont. na
udev                          16G     0   16G   0% /dev
tmpfs                        3,2G  329M  2,9G  11% /run
/dev/mapper/debian--vg-root   14G   12G  2,3G  84% /
tmpfs                         16G     0   16G   0% /dev/shm
tmpfs                        5,0M     0  5,0M   0% /run/lock
tmpfs                         16G     0   16G   0% /sys/fs/cgroup
/dev/xvda1                   236M  195M   29M  88% /boot
tmpfs                        3,2G     0  3,2G   0% /run/user/1000
```

Na `/dev/mapper/debian--vg-root` pozostało nieco ponad 2GB.

W celu wyświetlenia informacji o dostępnych dyskach należy skorzystać z polecenia `fdisk`:

```
root@shinobi:~# fdisk -l
bash: fdisk: nie znaleziono polecenia
root@shinobi:~# sudo fdisk -l
Dysk /dev/xvda: 16 GiB, bajtów: 17179869184, sektorów: 33554432
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512
Typ etykiety dysku: dos
Identyfikator dysku: 0x6d480a8d

Urządzenie Rozruch Początek   Koniec  Sektory Rozmiar Id Typ
/dev/xvda1 *           2048   499711   497664    243M 83 Linux
/dev/xvda2           501758 33552383 33050626   15,8G  5 Rozszerzona
/dev/xvda5           501760 33552383 33050624   15,8G 8e Linux LVM




Dysk /dev/mapper/debian--vg-root: 13,8 GiB, bajtów: 14751367168, sektorów: 28811264
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512


Dysk /dev/mapper/debian--vg-swap_1: 2 GiB, bajtów: 2143289344, sektorów: 4186112
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512
```

> Niestety VM niedysponuje wolnym dyskiem więc należy dodać to z poziomu hypervisora.
{.is-info}

Po dodaniu nowego dysku zobaczymy nowy wpis po uzyciu `fdisk -l`:

```
root@shinobi:~# sudo fdisk -l
Dysk /dev/xvda: 16 GiB, bajtów: 17179869184, sektorów: 33554432
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512
Typ etykiety dysku: dos
Identyfikator dysku: 0x6d480a8d

Urządzenie Rozruch Początek   Koniec  Sektory Rozmiar Id Typ
/dev/xvda1 *           2048   499711   497664    243M 83 Linux
/dev/xvda2           501758 33552383 33050626   15,8G  5 Rozszerzona
/dev/xvda5           501760 33552383 33050624   15,8G 8e Linux LVM


Dysk /dev/xvdb: 256 GiB, bajtów: 274877906944, sektorów: 536870912
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512




Dysk /dev/mapper/debian--vg-root: 13,8 GiB, bajtów: 14751367168, sektorów: 28811264
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512


Dysk /dev/mapper/debian--vg-swap_1: 2 GiB, bajtów: 2143289344, sektorów: 4186112
Jednostki: sektorów, czyli 1 * 512 = 512 bajtów
Rozmiar sektora (logiczny/fizyczny) w bajtach: 512 / 512
Rozmiar we/wy (minimalny/optymalny) w bajtach: 512 / 512
```

Nowy dysk jest czysty dlatego nalezy utworzyć na nim partycję za pomocą `fdisk`

```
root@shinobi:~# sudo fdisk /dev/xvdb

Witamy w programie fdisk (util-linux 2.33.1).
Zmiany pozostaną tylko w pamięci do chwili ich zapisania.
Przed użyciem polecenia zapisu prosimy o ostrożność.

Urządzenie nie zawiera żadnej znanej tablicy partycji.
Utworzono nową etykietę dysku DOS z identyfikatorem dysku 0x6d73d142.

Polecenie (m wyświetla pomoc): n
Typ partycji
   p   główna (głównych 0, rozszerzonych 0, wolnych 4)
   e   rozszerzona (kontener na partycje logiczne)
Wybór (domyślnie p): p
Numer partycji (1-4, domyślnie 1): 1
Pierwszy sektor (2048-536870911, domyślnie 2048):
Ostatni sektor, +/-sektorów lub +/-rozmiar{K,M,G,T,P} (2048-536870911, domyślnie 536870911):

Utworzono nową partycję 1 typu 'Linux' o rozmiarze 256 GiB.
```

Następnie formatujemy pierwszą partycję na `Linux LVM`:

```
Polecenie (m wyświetla pomoc): t
Wybrano partycję 1
Kod szesnastkowy (L wyświetla listę wszystkich kodów): L

 0  Brak            24  NEC DOS         81  Minix / stary L bf  Solaris
 1  FAT12           27  Ukryta HPFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 - ukryta l c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux ext       c7  Syrinx
 5  Rozszerzona     41  PPC PReP Boot   86  NTFS volume set da  Non-FS data
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility
 8  AIX             4e  QNX4.x part. 2. 8e  Linux LVM       df  BootIt
 9  AIX startowa    4f  QNX4.x part. 3. 93  Amoeba          e1  DOS access
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus - wyrówna
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs
 f  W95 Rozsz. (LBA 54  OnTrackDM6      a6  OpenBSD         ee  GPT
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  Ukryta FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor
14  Ukryta FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor
16  Ukryta FAT16    63  GNU HURD lub Sy af  HFS / HFS+      f2  DOS secondary
17  Ukryta HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE
1b  Ukryta W95 FAT3 70  DiskSecure Mult bb  Ukryta Boot Wiz fd  Linux RAID auto
1c  Ukryta W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep
1e  Ukryta W95 FAT1 80  Stary Minix     be  Solaris boot    ff  BBT
Kod szesnastkowy (L wyświetla listę wszystkich kodów): 8e
Zmieniono typ partycji 'Linux' na 'Linux LVM'.
```

Kolejny krok to po prostu zapisanie zmian:

```
Polecenie (m wyświetla pomoc): w
Tablica partycji została zmodyfikowana.
Wywoływanie ioctl() w celu ponownego odczytu tablicy partycji.
Synchronizacja dysków.

root@shinobi:~#
```

Za pomocą `fdisk -l` mozna zobaczyć dokonania w zakresie LVM.

### Ustawienia LVM

WYświetlenie aktualnych woluminów fizycznych:

```
root@shinobi:~# sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/xvda5
  VG Name               debian-vg
  PV Size               <15,76 GiB / not usable 2,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              4034
  Free PE               6
  Allocated PE          4028
  PV UUID               wm03RT-BTIG-N0Nc-NnG7-Qx8P-4Irm-44nNQ5
```

Wyświetlenie logycznych wolumonów:

```
root@shinobi:~# sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/debian-vg/root
  LV Name                root
  VG Name                debian-vg
  LV UUID                SRXeZy-TPUI-AwiQ-qZuM-ffoN-9ZMl-p02Uog
  LV Write Access        read/write
  LV Creation host, time debian, 2020-09-04 08:54:31 +0200
  LV Status              available
  # open                 1
  LV Size                <13,74 GiB
  Current LE             3517
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0

  --- Logical volume ---
  LV Path                /dev/debian-vg/swap_1
  LV Name                swap_1
  VG Name                debian-vg
  LV UUID                EuQF1d-md7O-lr2S-WDOb-bpci-15MI-sWiitZ
  LV Write Access        read/write
  LV Creation host, time debian, 2020-09-04 08:54:31 +0200
  LV Status              available
  # open                 2
  LV Size                <2,00 GiB
  Current LE             511
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:1
```

Dodatkowe informacje:

```
root@shinobi:~# sudo lvscan
  ACTIVE            '/dev/debian-vg/root' [<13,74 GiB] inherit
  ACTIVE            '/dev/debian-vg/swap_1' [<2,00 GiB] inherit
```

Ważny krok aby dodać sformatowany dysk do grupy wolumenów LVM:

```
root@shinobi:~# sudo vgextend debian-vg /dev/xvdb1
  Physical volume "/dev/xvdb1" successfully created.
  Volume group "debian-vg" successfully extended
```

Następnie rozszerzyć logiczny wolumin `/dev/debian-vg/root`:

> Rozszerzać można rozszerzeć responzywnie oraz wskazująć o ile zwiększyć, bądź zmniejszyć:
> - `sudo lvextend -L+10G /dev/debian-vg/root`
{.is-info}


```
root@shinobi:~# sudo lvextend -l +100%FREE /dev/debian-vg/root
  Size of logical volume debian-vg/root changed from <13,74 GiB (3517 extents) to <269,76 GiB (69058 extents).
  Logical volume debian-vg/root successfully resized.
```

> Jeżeli polecenie `df -h` lub `lsblk`, nadal wskazuje na mniejszą pojemność dysków dla `/dev/debian-vg/root`. Należy powiększyć system plików.
{.is-info}

> Pamiętaj o sprawdzeniu jaki system plików pracuje na LVM, z pomocą przychodzi komenda: `lsblk -f -m`
{.is-warning}


Wylistowanie systemu plików maszyny wirtualnej:

```
root@shinobi:~# sudo lsblk -f -m
NAME                  FSTYPE      LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT   SIZE OWNER GROUP MODE
sr0                                                                                                       1024M             brw-rw----
xvda                                                                                                        16G             brw-rw----
├─xvda1               ext2              13776986-6e5e-456e-aa6d-23df709047cf     28,3M    83% /boot        243M             brw-rw----
├─xvda2                                                                                                      1K             brw-rw----
└─xvda5               LVM2_member       wm03RT-BTIG-N0Nc-NnG7-Qx8P-4Irm-44nNQ5                            15,8G             brw-rw----
  ├─debian--vg-root   xfs               5f90d93c-5cbf-4e0c-ba17-459672f57846      2,3G    83% /          269,8G             brw-rw----
  └─debian--vg-swap_1 swap              ff26e20a-7a06-4092-8ca3-4771a90f8494                  [SWAP]         2G             brw-rw----
xvdb                                                                                                       256G             brw-rw----
└─xvdb1               LVM2_member       kwzP3i-jvsD-4FdI-vEvs-pkqk-EW6F-S4Wzsf                             256G             brw-rw----
  └─debian--vg-root   xfs               5f90d93c-5cbf-4e0c-ba17-459672f57846      2,3G    83% /          269,8G             brw-rw----
```

**Na tej podstawie jest informacja jaki system plików należy rozszerzyć.**

Rozszerzenie systemu plików:

```
root@shinobi:~# df -h
System plików               rozm. użyte dost. %uż. zamont. na
udev                          16G     0   16G   0% /dev
tmpfs                        3,2G   17M  3,2G   1% /run
/dev/mapper/debian--vg-root   14G   12G  2,4G  84% /
tmpfs                         16G     0   16G   0% /dev/shm
tmpfs                        5,0M     0  5,0M   0% /run/lock
tmpfs                         16G     0   16G   0% /sys/fs/cgroup
/dev/xvda1                   236M  195M   29M  88% /boot
tmpfs                        3,2G     0  3,2G   0% /run/user/1000
root@shinobi:~# sudo xfs_growfs  /
meta-data=/dev/mapper/debian--vg-root isize=512    agcount=4, agsize=900352 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=0
data     =                       bsize=4096   blocks=3601408, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =log wewnętrzny        bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =brak                   extsz=4096   blocks=0, rtextents=0
bloki danych zmienione z 3601408 na 70715392
root@shinobi:~# df -h
System plików               rozm. użyte dost. %uż. zamont. na
udev                          16G     0   16G   0% /dev
tmpfs                        3,2G   17M  3,2G   1% /run
/dev/mapper/debian--vg-root  270G   12G  259G   5% /
tmpfs                         16G     0   16G   0% /dev/shm
tmpfs                        5,0M     0  5,0M   0% /run/lock
tmpfs                         16G     0   16G   0% /sys/fs/cgroup
/dev/xvda1                   236M  195M   29M  88% /boot
tmpfs                        3,2G     0  3,2G   0% /run/user/1000
```


> **System wstał po restarcie i jest powodzenie:**
> ```
> /dev/mapper/debian--vg-root  270G   12G  259G   5% /
> ```


