Tommy OS for EBOX (Vortex86DX3)
English follows Japanese.
Tommy OSは、ICOP社製、EBOX3350DX3 (Vortex86DX3) 向けに構築された超軽量・リアルタイム志向のカスタムLinux OSです。


※開発利用実機：https://icop-shop.com/ja/product/ebox-3350dx3-rca-ap/

Vortex86特有のハードウェアの癖（タイマーのバグ、USBのハンドオフパニック、ストレージドライバの欠如など）を完全に回避するため、「外部Initrdを用いたRAMブート構成」を採用しています。これにより、ストレージへの依存をなくし、高速かつ安定した起動を実現しています。
動作環境・構成
Target Hardware: EBOX3350DX3 (Vortex86DX3 SoC)
Architecture: i486 (互換性最優先)
Bootloader: Syslinux (Extlinux)
File System: FAT32 (互換性確保) / 外部Initrd (RAMブート)
Build System: Buildroot (2024.02 LTS)
EBOX起動用MicroSD作成手順
リポジトリにあるイメージファイルを使って、EBOXを起動するためのMicroSDカードを作成する手順です。
1. 必要なツールのインストール (ホストPC: Ubuntu)
軽量ブートローダ syslinux をインストールします。
sudo apt update
sudo apt install -y syslinux syslinux-common


2. MicroSDカードの初期化とフォーマット (FAT32)
Windowsでも古いBIOSでも確実に読み込める FAT32 でフォーマットします。
# マウント解除
sudo umount /dev/mmcblk0p1

# msdosパーティションテーブルの作成
sudo parted -s /dev/mmcblk0 mklabel msdos

# FAT32パーティションの確保とブートフラグ付与
sudo parted -s /dev/mmcblk0 mkpart primary fat32 1MiB 100%
sudo parted -s /dev/mmcblk0 set 1 boot on

# FAT32でフォーマット
sudo mkfs.vfat -F 32 /dev/mmcblk0p1


3. Syslinuxのインストール
# FAT32パーティションに Syslinux をインストール
sudo syslinux --install /dev/mmcblk0p1

# MBRにマスターブートレコードを書き込む
sudo dd if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/mmcblk0 bs=440 count=1

4. イメージと設定ファイルの配置
MicroSDをマウントし、リポジトリにある3つの必須ファイルをコピーします。
bzImage (Linuxカーネル)
rootfs.cpio.gz (RAMディスク / OS本体)
syslinux.cfg (ブート設定ファイル)
sudo mkdir -p /mnt/ebox_sd
sudo mount /dev/mmcblk0p1 /mnt/ebox_sd

# ファイルのコピー (本リポジトリ内のファイルを指定してください)
sudo cp bzImage /mnt/ebox_sd/
sudo cp rootfs.cpio.gz /mnt/ebox_sd/
sudo cp syslinux.cfg /mnt/ebox_sd/

【参考: syslinux.cfg の内容（Vortex86 トラブルの回避）】
DEFAULT tommyos
TIMEOUT 30
PROMPT 0
LABEL tommyos
  SAY Booting Tommy OS (External RAM Disk Mode)...
  KERNEL bzImage
  INITRD rootfs.cpio.gz
  APPEND console=tty1 nomodeset noapic nolapic acpi=off pci=conf1,realloc=off,nommconf,nomsi,noquirks libata.dma=0 ide-core.nodma=0.0 notsc clocksource=pit nousb initcall_debug ignore_loglevel earlyprintk=vga


5. 後片付け
sync
sudo umount /mnt/ebox_sd


完成したMicroSDカードをEBOXに挿入し、電源を入れます。
ログインプロンプトが表示されたら、以下のアカウントでログイン可能です。
User: root
Password: root

English
# Tommy OS for EBOX (Vortex86DX3)

Tommy OS is an ultra-lightweight, real-time-oriented custom Linux OS built specifically for the ICOP EBOX3350DX3 (Vortex86DX3).

*Actual development device used:* https://icop-shop.com/ja/product/ebox-3350dx3-rca-ap/

To completely bypass hardware quirks specific to the Vortex86 (such as timer bugs, USB handoff panics, and the lack of storage drivers), this OS adopts an **"external Initrd RAM boot configuration."** This eliminates any dependency on storage, achieving fast and highly stable boot performance.

## Operating Environment & Configuration
* **Target Hardware:** EBOX3350DX3 (Vortex86DX3 SoC)
* **Architecture:** i486 (Compatibility prioritized)
* **Bootloader:** Syslinux (Extlinux)
* **File System:** FAT32 (Ensures compatibility) / External Initrd (RAM Boot)
* **Build System:** Buildroot (2024.02 LTS)

---

## MicroSD Card Creation Guide for EBOX Boot
Follow the steps below to create a bootable MicroSD card for the EBOX using the image files provided in this repository.

### 1. Install Required Tools (Host PC: Ubuntu)
Install the lightweight bootloader `syslinux`.

```bash
sudo apt update
sudo apt install -y syslinux syslinux-common
2. Initialize and Format the MicroSD Card (FAT32)
Format the card to FAT32 to ensure it can be reliably read by Windows and older BIOS systems.

Bash
# Unmount the partition
sudo umount /dev/mmcblk0p1

# Create an msdos partition table
sudo parted -s /dev/mmcblk0 mklabel msdos

# Create a FAT32 partition and set the boot flag
sudo parted -s /dev/mmcblk0 mkpart primary fat32 1MiB 100%
sudo parted -s /dev/mmcblk0 set 1 boot on

# Format as FAT32
sudo mkfs.vfat -F 32 /dev/mmcblk0p1
3. Install Syslinux
Bash
# Install Syslinux to the FAT32 partition
sudo syslinux --install /dev/mmcblk0p1

# Write the Master Boot Record (MBR)
sudo dd if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/mmcblk0 bs=440 count=1
4. Place Image and Configuration Files
Mount the MicroSD card and copy the three essential files from the repository.

bzImage (Linux Kernel)

rootfs.cpio.gz (RAM Disk / Base OS)

syslinux.cfg (Boot configuration file)

Bash
sudo mkdir -p /mnt/ebox_sd
sudo mount /dev/mmcblk0p1 /mnt/ebox_sd

# Copy the files (please point to the files within this repository)
sudo cp bzImage /mnt/ebox_sd/
sudo cp rootfs.cpio.gz /mnt/ebox_sd/
sudo cp syslinux.cfg /mnt/ebox_sd/
[Reference: Contents of syslinux.cfg ]

Plaintext
DEFAULT tommyos
TIMEOUT 30
PROMPT 0
LABEL tommyos
  SAY Booting Tommy OS (External RAM Disk Mode)...
  KERNEL bzImage
  INITRD rootfs.cpio.gz
  APPEND console=tty1 nomodeset noapic nolapic acpi=off pci=conf1,realloc=off,nommconf,nomsi,noquirks libata.dma=0 ide-core.nodma=0.0 notsc clocksource=pit nousb initcall_debug ignore_loglevel earlyprintk=vga
5. Cleanup
Bash
sync
sudo umount /mnt/ebox_sd
Insert the completed MicroSD card into the EBOX and power it on.
Once the login prompt appears, you can log in with the following credentials:

User: root

Password: root
