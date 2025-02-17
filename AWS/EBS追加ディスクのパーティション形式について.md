[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)

---

# EC2 EBS追加ディスクのパーティション形式について

![](/AWS/EBS追加ディスクのパーティション形式について/icon.png)

# 目次

- [EC2 EBS追加ディスクのパーティション形式について](#ec2-ebs追加ディスクのパーティション形式について)
- [目次](#目次)
- [はじめに](#はじめに)
- [気にしておくこと](#気にしておくこと)
- [WindowsServerをいざ設定へ](#windowsserverをいざ設定へ)
- [LinuxServerをいざ設定へ](#linuxserverをいざ設定へ)
- [その前に・・・](#その前に)
- [本題のLinuxServerでのGPTパーティション作成とLVMボリューム作成](#本題のlinuxserverでのgptパーティション作成とlvmボリューム作成)

# はじめに

---

何度も慣れしたんでおりますが、EBS追加ディスク追加についておさらいです。

# 気にしておくこと

---

WindowsServerやLinuxSererでEBSのボリュームを2TB以上使うか、使わないかで気にしないといけない点があります。

ディスクのパーティション形式としてMBRとGPTがあります。

MBRとはMaster Boot Recordの略で従来から使われている古い仕様ですが、2TB容量を超えることができません。

GPTとはGUID Partition Tableの略で大容量なディスクに対応することができます。

設計上でディスクサイズが2TB超えるか、超えないかは設計時に決めてください。

※ 後から変更することはできなくもないですがめっちゃめんどうです。

# WindowsServerをいざ設定へ

---

コンピュータ管理の画面からディスク管理の画面にいきます。

**ここでGPTを選択してOKをクリックします。**

![](/AWS/EBS追加ディスクのパーティション形式について/pic1.png)

GPTになっているかどうかはOKをクリック後、DISK1を右クリックでプロパティからボリュームを確認してください。

あとはフォーマットしていくだけです。

なおディスク容量追加の場合はOS再起動なしでCドライブも拡張できます。

![](/AWS/EBS追加ディスクのパーティション形式について/pic2.png)

# LinuxServerをいざ設定へ

---

詳細は割愛しますが、fdiskコマンドはGPTには対応しておりませんのでfdiskでパーティションを切るとMBR方式になります。

対してpartedコマンドはGPTに対応しているので上記でも記載した通り2TBを超える場合にはpartedで実施する必要があります。

なお、gdiskもGPTに対応しています。gdiskはfdiskのGPT版だと思ってください。

違いとしてはpartedとはコマンド実行をすぐに書き込みしますが、gdiskの場合は最終確認に回答してから書き込みをするので、

誤って既存パーティションを破損してしまう心配が少なく済むかと思います。

# その前に・・・

---

追加ディスクの前にルートディスクの拡張方法について先に触れておきます。

ルートボリュームが8GBの状態

```jsx
$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0   8G  0 disk 
├─nvme0n1p1   259:3    0   8G  0 part /
├─nvme0n1p127 259:4    0   1M  0 part 
└─nvme0n1p128 259:5    0  10M  0 part /boot/efi
nvme1n1       259:1    0   8G  0 disk 
nvme2n1       259:2    0   8G  0 disk 
```

8GBを追加して16GBの状態

```jsx
$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  16G  0 disk 
├─nvme0n1p1   259:3    0   8G  0 part /
├─nvme0n1p127 259:4    0   1M  0 part 
└─nvme0n1p128 259:5    0  10M  0 part /boot/efi
nvme1n1       259:1    0   8G  0 disk 
nvme2n1       259:2    0   8G  0 disk 
```

growpartで既存ボリュームを拡張し、16GBが認識されていることを確認します。

```jsx
# growpart /dev/nvme0n1 1
CHANGED: partition=1 start=24576 old: size=16752607 end=16777183 new: size=33529823 end=33554399
# lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  16G  0 disk 
├─nvme0n1p1   259:3    0  16G  0 part /
├─nvme0n1p127 259:4    0   1M  0 part 
└─nvme0n1p128 259:5    0  10M  0 part /boot/efi
nvme1n1       259:1    0   8G  0 disk 
nvme2n1       259:2    0   8G  0 disk 
```

dfコマンドで拡張が適用されていないこと、/パーティションがxfsタイプであることを確認します。

```jsx
# df -lhT
Filesystem       Type      Size  Used Avail Use% Mounted on
devtmpfs         devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs            tmpfs     951M     0  951M   0% /dev/shm
tmpfs            tmpfs     381M  452K  380M   1% /run
/dev/nvme0n1p1   xfs       8.0G  1.5G  6.5G  19% /
tmpfs            tmpfs     951M     0  951M   0% /tmp
/dev/nvme0n1p128 vfat       10M  1.3M  8.7M  13% /boot/efi
tmpfs            tmpfs     191M     0  191M   0% /run/user/1000
```

[XFS ファイルシステム] xfs_growfs コマンドを使用して拡張します。

※ [Ext4 ファイルシステム]はresize2fs コマンドを使用します。

```jsx
# xfs_growfs -d /
meta-data=/dev/nvme0n1p1         isize=512    agcount=2, agsize=1047040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2094075, imaxpct=25
         =                       sunit=128    swidth=128 blks
naming   =version 2              bsize=16384  ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=4 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2094075 to 4191227
```

再度確認します。nvme0n1とnvme0n1p1のサイズが一致し、dfコマンド結果でも16GBになっていることを確認します。

```jsx
# lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  16G  0 disk 
├─nvme0n1p1   259:3    0  16G  0 part /
├─nvme0n1p127 259:4    0   1M  0 part 
└─nvme0n1p128 259:5    0  10M  0 part /boot/efi
nvme1n1       259:1    0   8G  0 disk 
nvme2n1       259:2    0   8G  0 disk 

# df -lhT
Filesystem       Type      Size  Used Avail Use% Mounted on
devtmpfs         devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs            tmpfs     951M     0  951M   0% /dev/shm
tmpfs            tmpfs     381M  452K  380M   1% /run
/dev/nvme0n1p1   xfs        16G  1.6G   15G  10% /
tmpfs            tmpfs     951M     0  951M   0% /tmp
/dev/nvme0n1p128 vfat       10M  1.3M  8.7M  13% /boot/efi
tmpfs            tmpfs     191M     0  191M   0% /run/user/1000
```

# 本題のLinuxServerでのGPTパーティション作成とLVMボリューム作成

---

追加したディスクをlsblkで確認するとnvme1n1とnvme2n1があります。

この2つのディスクをVolume Groupとして1つのグループにして、Logical Volumeを作るといったことをします。

LVMを使う可能性としてあるのは

- 1台のEC2で20TBのボリュームを使いたい
- 高スループットを実現したいけど、コスト抑えたい

AWS環境でLVMを使う可能性は低い気がしますが、高スループットでコスト効率も高く16GB以上のストレージを確保できると思います。

LVMはLogical Volume Managerといい、複数のストレージを単一の記憶領域として利用する機能になります。

```jsx
# lsblk -f
NAME          FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
nvme0n1                                                                              
├─nvme0n1p1   xfs          /     81e4e009-191b-464c-8cc3-22de217d1136   14.4G     9% /
├─nvme0n1p127                                                                        
└─nvme0n1p128 vfat   FAT16       EA7D-FA7D                               8.7M    13% /boot/efi
nvme1n1                                                                              
nvme2n1    
```

パーティションを作成します。同じようにnvme2n1 でも実施します。

```jsx
# gdisk /dev/nvme1n1
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-16777182, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-16777182, default = 16777182) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8e00
Changed type of partition to 'Linux LVM'

Command (? for help): p
Disk /dev/nvme1n1: 16777216 sectors, 8.0 GiB
Model: Amazon Elastic Block Store              
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): D46A4C98-23F8-452D-B36B-7E7343356B08
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        16777182   8.0 GiB     8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y 
OK; writing new GUID partition table (GPT) to /dev/nvme1n1.
The operation has completed successfully.
```

nvme1n1p1とnvme2n1p1が作成されました。

```jsx
# lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0  16G  0 disk 
├─nvme0n1p1   259:3    0  16G  0 part /
├─nvme0n1p127 259:4    0   1M  0 part 
└─nvme0n1p128 259:5    0  10M  0 part /boot/efi
nvme1n1       259:1    0   8G  0 disk 
└─nvme1n1p1   259:7    0   8G  0 part 
nvme2n1       259:2    0   8G  0 disk 
└─nvme2n1p1   259:8    0   8G  0 part 
```

作成したパーティションごとに物理ボリュームを作成します。

```jsx
# pvcreate /dev/nvme1n1p1 
  Physical volume "/dev/nvme1n1p1" successfully created.
  Creating devices file /etc/lvm/devices/system.devices

# pvcreate /dev/nvme2n1p1 
  Physical volume "/dev/nvme2n1p1" successfully created.
```

ボリュームグループを作成します。

```jsx
# vgcreate VolumeGroup /dev/nvme1n1p1
  Volume group "VolumeGroup" successfully created
```

VolumeGroupに8GBが割り当てられていることを確認します。

```jsx
# vgs
  VG          #PV #LV #SN Attr   VSize  VFree 
  VolumeGroup   1   0   0 wz--n- <8.00g <8.00g
```

既存ボリュームグループに追加します。

```jsx
# vgextend VolumeGroup /dev/nvme2n1p1
  Volume group "VolumeGroup" successfully extended
```

VolumeGroupを再度確認して、16GBになっていることを確認します。

```jsx
# vgs
  VG          #PV #LV #SN Attr   VSize  VFree 
  VolumeGroup   2   0   0 wz--n- 15.99g 15.99g
```

最後に論理ボリュームを作成します。ここでは16GBある物理ボリュームから10GBの論理ボリュームを作成します。

```jsx
# lvcreate -n LogicalVolume -L 10G VolumeGroup
  Logical volume "LogicalVolume" created.
```

10GBの論理ボリュームがあることを確認します。

```jsx
# lvs
  LV            VG          Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogicalVolume VolumeGroup -wi-a----- 10.00g   
```

ボリュームのマウント先を用意し、xfs形式でファイルシステムを作成します。

```jsx
# mkdir /test
# mkfs -t xfs /dev/VolumeGroup/LogicalVolume 
meta-data=/dev/VolumeGroup/LogicalVolume isize=512    agcount=16, agsize=163840 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=1      swidth=1 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

nvme1n1p1とnvme2n1p1が同じ論理ボリュームであることを確認します。

```jsx
# lsblk -f
NAME                          FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
nvme0n1                                                                                                        
├─nvme0n1p1                   xfs                  /     81e4e009-191b-464c-8cc3-22de217d1136     14.4G    10% /
├─nvme0n1p127                                                                                                  
└─nvme0n1p128                 vfat        FAT16          EA7D-FA7D                                 8.7M    13% /boot/efi
nvme1n1                                                                                                        
└─nvme1n1p1                   LVM2_member LVM2 001       GRhLeb-6zzb-HRoI-92dH-6Lpx-z9zj-dTscKg                
  └─VolumeGroup-LogicalVolume xfs                        76293569-10d7-43f3-9437-70c202d64c2c                  
nvme2n1                                                                                                        
└─nvme2n1p1                   LVM2_member LVM2 001       SUeMzB-D70r-OgUS-cdv9-hxRy-WoUy-IHJM23                
  └─VolumeGroup-LogicalVolume xfs                        76293569-10d7-43f3-9437-70c202d64c2c  
```

/testにファイルシステムをマウントして、2つのEBSを論理的に結合した1つのボリュームとして扱えます。

```jsx
# mount /dev/VolumeGroup/LogicalVolume /test
# lsblk -f
NAME                          FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
nvme0n1                                                                                                        
├─nvme0n1p1                   xfs                  /     81e4e009-191b-464c-8cc3-22de217d1136     14.4G    10% /
├─nvme0n1p127                                                                                                  
└─nvme0n1p128                 vfat        FAT16          EA7D-FA7D                                 8.7M    13% /boot/efi
nvme1n1                                                                                                        
└─nvme1n1p1                   LVM2_member LVM2 001       GRhLeb-6zzb-HRoI-92dH-6Lpx-z9zj-dTscKg                
  └─VolumeGroup-LogicalVolume xfs                        76293569-10d7-43f3-9437-70c202d64c2c      9.8G     1% /test
nvme2n1                                                                                                        
└─nvme2n1p1                   LVM2_member LVM2 001       SUeMzB-D70r-OgUS-cdv9-hxRy-WoUy-IHJM23                
  └─VolumeGroup-LogicalVolume xfs                        76293569-10d7-43f3-9437-70c202d64c2c      9.8G     1% /test
```

再起動後もマウントが保持されるようにしたい場合には/etc/fstabに追記します。

```jsx
/dev/VolumeGroup/LogicalVolume /test   xfs     defaults,nofail   0   0
```

---

[⚫️ TOPへ戻る](https://actmotech.xyz/)

[⚫️ AWS Knowledgeへ戻る](/AWS/top)
