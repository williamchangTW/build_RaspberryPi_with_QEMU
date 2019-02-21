# Raspberry Pi on QEMU virtual-machine 
contributed by <`williamchangTW`>

## Topic 1: Raspberry Pi Kernel preparing
- Environment: Ubuntu 18.04(VirtualBox)
- 使用已經製作好的 `kernel` 用來模擬 Raspberry Pi 在 QEMU 上，可以依據這個連結[kernel sources]內已經製作好的 `kernel` 進行編譯。  
### For Raspberry Pi, OS: Raspbian
在開始之前，必須要有[Raspbian image]，可以從這個連結到官網上下載。下載下來會是一個 `.zip` 檔，用指令解開它，解壓縮成 `.img` 檔案。
    
    $ unzip <your_zip_file>

`Raspbian` 是為了 Raspberry Pi 所開發的作業系統，非常簡易，因為模擬 Raspberry Pi 的環境通常會優先使用 `Raspbian`，因為是特地為這個硬體所開發的，相對資源較多且易於開發。 

### How to choose a kernel image?
以下會包含三個類型的 `kernel` 映像檔：
- `kernel-qemu-4.*.*-stretch`:
  - 最常見的映像檔，與 `Raspbian Stretch` 和 `Raspbian Jessie` 相容性較好。若要使用這個映像檔需要 `versatile-pb.dtb`（一樣在這個路徑底下）。這個映像檔會是最符和一開始的開發使用。
- `kernel-qemu-4.4.*-jessie`:
  - 比較適合 `Raspbian Jessie` 及 `Raspbian Wheezy`。
- `kernel-qemu-3.10.25-wheezy`:
  - 這是原始的映像檔從[xecdesign.com]所開發的。只適用於 `Raspbian Wheezy`。

## Using kernel images with QEMU
建立 `QEMU` 的虛擬環境指令：

    $ qemu-system-arm \
        -M versatilepb \
        -cpu arm1176 \
        -m 256 \
        -hda 2018-11-13-raspbian-stretch-lite.img \
        -net nic \
        -net user,hostfwd=tcp::5022-:22 \
        -dtb ./qemu-rpi-kernel/versatile-pb.dtb \
        -kernel ./qemu-rpi-kernel/kernel-qemu-4.14.79-stretch \
        -append 'root=/dev/sda2 panic=1' \
        -no-reboot

路徑可以根據不同的機器做更改。
#### More detail 
- `-cpu`: CPU model.
- `-m`: Size of RAM.
- `-M`: board model(for varified ARM board architecture.
- `-serial stdio`: option redirects the boot output messages and the console to your terminal.
- `-append`: can contains all options to be passed on to the kernel at boot time, which can control the log level using `loglevel=`, `console==ttyAMA0` for ARM, `console=ttyS0` for Intel machine. 
- `root=/dev/sda2`: key option for our kernel place.
- `PARTUUID=`: means Partition Universally Unique Identifier, can replace `/dev/sda2`(if you want to boot from USB, because rely on the name and need the Unique partition identifier).
    
內容可以在 `cmdline.txt` 檔案內找到更多資訊
---

- `-drive file=2018-10-09-raspbian-stretch-lite.img, format=raw`: memory card is modeled
- `-redir tcp:5022:22`: The network is integrated in the board model, use this option
- `ssh -p 5022 localhost`: Lets you ssh inti the guset using

## Running QEMU

當你 Login 會看到

    $ raspberrypi login:

使用 `pi`, 密碼為 `raspberry`

    $ sudo systemctl ssh start
    
連接到 `ssh` 服務

    $ ssh -p 5022 pi@localhost
    

## Using kernel images with libvirt
若用 [libvirt] 建立虛擬環境如下指令：
  
    $ virt-install \
      --name pi \
      --arch armv6l \
      --machine versatilepb \
      --cpu arm1176 \
      --vcpus 1 \
      --memory 256 \
      --import \
      --disk /.../2018-11-13-raspbian-stretch-lite.img,format=raw,bus=virtio \
      --network user,model=virtio \
      --video vga \
      --graphics spice \
      --rng device=/dev/urandom,model=virtio \
      --boot 'dtb=/.../versatile-pb.dtb,kernel=/.../kernel-qemu-4.14.79-stretch,kernel_args=root=/dev/vda2 panic=1' \
      --events on_reboot=destroy

建立 `libvert` 的客戶稱為 `pi`（這邊為了簡易使用）,為了要控制這個客戶，可以自由地使用如 `virsh` 和 `virt-manager` 等工具輔助使用。 
#### More detail

### Building your own kernel image

可以參考 `tools/` 資料夾內的內容，建制自己所需的 `kernel`。

#### Origin of this repository

這篇文章最主要參考 [Github: dhruvvyas90] 的 `Github` 內容所製作。

## Further informationo
若要模擬 RPi 在 QEMU 上, 至少需要下面兩個檔案
    - RPi kernel
    - RPi rootfs(root files)(在官網的 `img` 檔案中)
若要自己準備更新的 `kernel` 會在下面提到
### Installation
- Ubuntu(generally, any OS based on Debian)
    
        $ sudo apt-get install qemu
    
- OSX:

        brew install qemu
       
- Windows:
    - 可以在[Official Download page]直接下載最新的 QEMU 版本
    
### Topic 2: Emulating 
當你下載完 QEMU 後, 可以得到所需的檔案, 根據下面步驟進行
1. 再命令列上執行:

        $ qemu-system-arm -kernel kernel-qemu \
                  -cpu arm1176 \
                  -m 256 \
                  -M versatilepb \
                  -no-reboot \
                  -serial stdio \
                  -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw init=/bin/bash" \
                  -hda image-file-name.img
                  
2.編輯 `/etc/ld.so.preload` 的檔案中, 在第一行前面加上註解標誌(`#`)

        #/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so
        #
        #rest of the file
        
3. 編輯 `/etc/udev/rules.d/90-qemu.rules` 檔案中加入下面幾行:

        KERNEL=="sda", SYMLINK+="mmcblk0"
        KERNEL=="sda?", SYMLINK+="mmcblk0p%n"
        KERNEL=="sda2", SYMLINK+="root"
        
4. 執行

        sudo poweroff
    
5. 重新模擬

        qemu-system-arm -kernel kernel-qemu \
                        -cpu arm1176 \
                        -m 256 \
                        -M versatilepb \
                        -serial stdio \
                        -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" \
                        -drive "file=image-file-name.img,index=0,media=disk,format=raw"

#### Key notes
1. RAM 無法超出 256, 除非在 `versatilepb` 檔案中存在 bug
2. 可能會需要設定 `ssh`, 可以考慮下面的選項, 當建立 QEMU 時
```
-net nic \
-net user, hostfwd=tcp::2222-:22
```

### Topic 3: Emulating Jessie image with 4.X.XX kernel
這邊會如何模擬 `Jessie image` 的 4.x.xx `kernel`, 跟 `Wheezy` 的方式有些微的不同, 假設你有下面兩個檔案
    1. 最新版本的 `Jessie` 映像檔
    2. 4.x.xx qemu-kernel for versatilepb
#### Steps
1. `fdisk -l <image-file>`
- 輸出訊息如下:
```
Disk 2015-11-21-raspbian-jessie.img: 3.7 GiB, 3934257152 bytes, 7684096 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xea0e7380

Device                          Boot  Start     End Sectors  Size Id Type
2015-11-21-raspbian-jessie.img1        8192  131071  122880   60M  c W95 FAT32 (LBA)
2015-11-21-raspbian-jessie.img2      131072 7684095 7553024  3.6G 83 Linux
```
- 檔案系統(.img2) 從 Sector: 131072, 等於 `512*131072=67108864 bytes`, 這個數字為偏移量, 實例如: `mount -v -o offset=67108864 -t ext4 your-image-file.img /path/to/mnt/`

2. `cd /path/to/mnt`
3. `sudo nano ./etc/ld.so.preload`
4. 註解每一行`Ctrl-x` >> `Y` 儲存後離開
5. `sudo nano ./etc/fstab`
6. 註解每一個含有 `/dev/mmcblk`, `Ctrl-x` >> Y 儲存離開
7. `cd ~`
8. `sudo umount /path/to/mnt`

- 上面的步驟只需做過一次後, 接下來只需要用下面指令就可以模擬 RPi:
```
qemu-system-arm \
-kernel /path/to/kernel/kernel-name \
-cpu arm1176 \
-m 256 \
-M versatilepb \
-serial stdio \
-append "root=/dev/sda2 rootfstype=ext4 rw" \
-hda /path/to/image/image-file-name.img
```

### Reference
[Raspbian image]: https://www.raspberrypi.org/downloads/raspbian/
[kernel sources]: https://github.com/raspberrypi/linux/
[xecdesign.com]: https://xecdesign.com/downloads/linux-qemu/kernel-qemu
[libvirt]: https://wiki.archlinux.org/index.php/Libvirt_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87)
[Github: dhruvvyas90]: https://github.com/dhruvvyas90/qemu-rpi-kernel
[Official Download page]: https://www.qemu.org/download/

