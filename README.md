# cloud_storage_proxmox
I document here my process of making cloud storage on my server. I'm using Proxmox VE with installed Debian in it

1. My server, 2x 2 TB disks in RAID 1 and Unitek HDD Enclosure

   <img width="1536" height="2048" alt="5e8216e8-7e81-444c-9b94-84b4433912a1" src="https://github.com/user-attachments/assets/796e52ab-9f01-4a22-8d3d-29c011baad59" />
   <img width="1536" height="2048" alt="003ad90c-6e20-4e91-af15-0940d56c1f9b" src="https://github.com/user-attachments/assets/16e26f1c-a11d-4738-88b2-ed77cf3f8874" />
   <img width="1536" height="2048" alt="8636a9f3-b062-4929-afce-4c939b01d36c" src="https://github.com/user-attachments/assets/f13715fe-1683-4c77-9387-9d1a0ac71d6f" />
   <img width="1536" height="2048" alt="6530d547-aad7-479d-aa27-cafea2c2f3dd" src="https://github.com/user-attachments/assets/c815cc48-99cd-4bda-a60f-988a9c552afa" />
   <img width="1536" height="2048" alt="cb9a060b-7eca-492a-926b-ecc6f39974b6" src="https://github.com/user-attachments/assets/2cac38d3-7ba7-4359-a07e-4df9a5b34e9f" />

   Server specs:

      Dell OptiPlex 7010 SFF
      CPU : Intel Core i5-3470 @ 3.20GHz
      Memory: Kingston 16 GB (4x4 GB) RAM DDR3 1600 MT/s
      SSD System disk 240 GB
      2x 2 TB HDD disks in RAID 1
      

2. Disk partitioning

      adminus@nextcloud:~$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
      NAME    SIZE TYPE FSTYPE  MOUNTPOINTS
      sda     930G disk
      sdb      40G disk
      ├─sdb1 37.9G part ext4    /
      ├─sdb2    1K part
      └─sdb5  2.1G part swap    [SWAP]
      sr0     756M rom  iso9660
      adminus@nextcloud:~$ sudo fdisk -l /dev/sda
      Disk /dev/sda: 930 GiB, 998579896320 bytes, 1950351360 sectors
      Disk model: QEMU HARDDISK
      Units: sectors of 1 * 512 = 512 bytes
      Sector size (logical/physical): 512 bytes / 512 bytes
      I/O size (minimum/optimal): 512 bytes / 512 bytes
      Disklabel type: gpt
      Disk identifier: E7729E67-3AD4-4EFB-8A50-BF3F089A7EFD
      adminus@nextcloud:~$ sudo fdisk /dev/sda
      
      Welcome to fdisk (util-linux 2.41.5).
      Changes will remain in memory only, until you decide to write them.
      Be careful before using the write command.
      
      
      Command (m for help): n
      Partition number (1-128, default 1): 1
      First sector (2048-1950351326, default 2048):
      Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1950351326, default 1950349311):
      
      Created a new partition 1 of type 'Linux filesystem' and of size 930 GiB.
      
      Command (m for help): p
      Disk /dev/sda: 930 GiB, 998579896320 bytes, 1950351360 sectors
      Disk model: QEMU HARDDISK
      Units: sectors of 1 * 512 = 512 bytes
      Sector size (logical/physical): 512 bytes / 512 bytes
      I/O size (minimum/optimal): 512 bytes / 512 bytes
      Disklabel type: gpt
      Disk identifier: E7729E67-3AD4-4EFB-8A50-BF3F089A7EFD
      
      Device     Start        End    Sectors  Size Type
      /dev/sda1   2048 1950349311 1950347264  930G Linux filesystem
      
      Command (m for help): w
      The partition table has been altered.
      Calling ioctl() to re-read partition table.
      Syncing disks.
      
      adminus@nextcloud:~$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
      NAME    SIZE TYPE FSTYPE  MOUNTPOINTS
      sda     930G disk
      └─sda1  930G part
      sdb      40G disk
      ├─sdb1 37.9G part ext4    /
      ├─sdb2    1K part
      └─sdb5  2.1G part swap    [SWAP]
      sr0     756M rom  iso9660
      adminus@nextcloud:~$ lsblk -f
      NAME   FSTYPE  FSVER            LABEL                 UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
      sda
      └─sda1
      sdb
      ├─sdb1 ext4    1.0                                    6bb19134-b9b1-4143-ad4e-172ae96d4218   33.9G     3% /
      ├─sdb2
      └─sdb5 swap    1                                      8130c8fa-4d82-47ee-aa4c-22fd9a182955                [SWAP]
      sr0    iso9660 Joliet Extension Debian 13.7.0 amd64 n 2026-09-12-09-39-25-00
      adminus@nextcloud:~$ lsblk -l
      NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
      sda    8:0    0  930G  0 disk
      sda1   8:1    0  930G  0 part
      sdb    8:16   0   40G  0 disk
      sdb1   8:17   0 37.9G  0 part /
      sdb2   8:18   0    1K  0 part
      sdb5   8:21   0  2.1G  0 part [SWAP]
      sr0   11:0    1  756M  1 rom
      adminus@nextcloud:~$ sudo mkfs.ext4 -L nextcloud-data /dev/sda1
      mke2fs 1.47.2 (1-Jan-2025)
      Discarding device blocks: done
      Creating filesystem with 243793408 4k blocks and 60948480 inodes
      Filesystem UUID: 1a9e5b6c-85bd-402f-9600-7e1df0dfc882
      Superblock backups stored on blocks:
              32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
              4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
              102400000, 214990848
      
      Allocating group tables: done
      Writing inode tables: done
      Creating journal (262144 blocks): done
      Writing superblocks and filesystem accounting information: done
      
      adminus@nextcloud:~$ lsblk -f
      NAME   FSTYPE  FSVER            LABEL                 UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
      sda
      └─sda1 ext4    1.0              nextcloud-data        1a9e5b6c-85bd-402f-9600-7e1df0dfc882
      sdb
      ├─sdb1 ext4    1.0                                    6bb19134-b9b1-4143-ad4e-172ae96d4218   33.9G     3% /
      ├─sdb2
      └─sdb5 swap    1                                      8130c8fa-4d82-47ee-aa4c-22fd9a182955                [SWAP]
      sr0    iso9660 Joliet Extension Debian 13.7.0 amd64 n 2026-09-12-09-39-25-00
      
      
