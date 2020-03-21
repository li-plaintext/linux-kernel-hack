#### linux kernel
   - sudo apt install build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache flex bison libelf-dev
   - git clone https://kernel.googlesource.com/pub/scm/linux/kernel/git/next/linux-next.git
   - make ARCH=x86_64 x86_64_defconfig 
   - make ARCH=x86_64 menuconfig
   - make -j8


#### File system
   - ##### busybox
      - cd busybox
      - make menuconfig
         - Build Options 
           - Build BusyBox as a static binary (no shared libs)
      - make -j4  

   - ##### initrd
     1. ###### Create `ramdisk` folder
        - mkdir ramdisk
     2. ###### Create folders `(bin,sbin,etc,proc,sys,usr,dev)`
        - cd ramdisk
        - cp -r ../busybox/_install/*  .
        - ln -s bin/busybox init
        - mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin},dev}

     3. ###### Create `ramdish/etc/inittab`
        - cd etc
        - vim `inittab`
        - create content in `inittab`
           ```shell
           ::sysinit:/etc/init.d/rcS   
           ::askfirst:-/bin/sh    
           ::restart:/sbin/init
           ::ctrlaltdel:/sbin/reboot
           ::shutdown:/bin/umount -a -r
           ::shutdown:/sbin/swapoff -a
           ```
        - chmod +x inittab
  
      4. ###### Create `ramdish/etc/init.d`
           - mkdir init.d
           - cd init.d
           - vim rcS
           - Create content in `rcS`
             ```bash
             #!/bin/sh

             mount proc
             mount -o remount,rw /
             mount -a    
             clear                               
             ```
           - chmod +x rcS

      5. ###### Create `ramdish/etc/fstab`
           - vim fstab
             ```bash
             # /etc/fstab

             proc            /proc        proc    defaults          0       0
             sysfs           /sys         sysfs   defaults          0       0
             devtmpfs        /dev         devtmpfs  defaults          0       0
             ```
      6. ###### Create `initramfs.img`
           - cd ramdisk
           - find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../initramfs.img

      7. ###### overall in `ramdisk`
         ```
          - bin
          - dev
          - etc
            - init.d
              - rcS (file)
            - fstab (file)
            - inittab (file)
          - proc
          - sbin
          - sys
          - usr
          - init (file)
          - linuxrc (file)
         ```

#### qemu
   - sudo apt install qemu qemu-system

   - qemu-system-x86_64 -no-kvm -kernel arch/x86/boot/bzImage -hda /dev/zero -append "root=/dev/zero console=ttyS0" -serial stdio -display none
       

#### run
   qemu-system-x86_64 -kernel ./linux-5.5.5/arch/x86/boot/bzImage -initrd ./initramfs.img -S -s -append nokaslr

----
#### gdb
   - gdb ./vmlinux
   - (gdb) target remote:1234
   - (gdb) b start_kernel
   - (gdb) c
      - ....debug information
      
Reference: https://blog.csdn.net/jasonLee_lijiaqi/article/details/80967912?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task
