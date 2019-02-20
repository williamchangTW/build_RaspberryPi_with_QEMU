# Raspberry Pi on QEMU virtual-machine 
contributed by <`williamchangTW`>

## Raspberry Pi Kernel preparing
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

#### Further information

- 額外的參考檔案可以在這個連結下找到：[wiki]

### Reference
[Raspbian image]: https://www.raspberrypi.org/downloads/raspbian/
[kernel sources]: https://github.com/raspberrypi/linux/
[xecdesign.com]: https://xecdesign.com/downloads/linux-qemu/kernel-qemu
[libvirt]: https://wiki.archlinux.org/index.php/Libvirt_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87)
[Github: dhruvvyas90]: https://github.com/dhruvvyas90/qemu-rpi-kernel
[wiki]: https://github.com/dhruvvyas90/qemu-rpi-kernel/wiki
