Tommy OS for EBOX (Vortex86DX3)
Tommy OSは、EBOX3350DX3 (Vortex86DX3) 向けに構築された超軽量・リアルタイム志向のカスタムLinux OSです。
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


【参考: syslinux.cfg の内容（Vortex86 究極の回避呪文）】
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
