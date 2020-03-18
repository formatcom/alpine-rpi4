~~~
 1.- REF: https://www.raspberrypi.org/documentation/
 2.- REF: https://www.raspberrypi.org/documentation/configuration/boot_folder.md
 3.- REF: https://www.raspberrypi.org/documentation/configuration/config-txt
 4.- REF: https://www.raspberrypi.org/documentation/configuration/config-txt/boot.md
 5.- REF: https://www.raspberrypi.org/documentation/configuration/config-txt/memory.md
 6.- REF: https://www.raspberrypi.org/documentation/configuration/config-txt/video.md
 7.- REF: https://www.raspberrypi.org/documentation/hardware/raspberrypi/peripheral_addresses.md
 8.- REF: https://github.com/raspberrypi
 9.- REF: https://github.com/raspberrypi/documentation
10.- REF: https://github.com/raspberrypi/linux
11.- REF: https://github.com/raspberrypi/firmware
12.- REF: https://wiki.alpinelinux.org/wiki/Classic_install_or_sys_mode_on_Raspberry_Pi
13.- REF: https://wiki.alpinelinux.org/wiki/Alpine_Linux_Init_System
14.- REF: https://wiki.alpinelinux.org/wiki/Syslog
15.- REF: https://www.raspberrypi.org/documentation/linux/kernel/building.md
16.- REF: https://www.tal.org/tutorials/booting-64-bit-kernel-raspberry-pi-3
17.- REF: http://jake.dothome.co.kr/zone-types/
18.- REF: http://weng-blog.com/2015/05/Build-Linux-kernel-rootfs-from-scratch/
19.- REF: https://blog.carreralinux.com.ar/2017/07/uso-de-cpio-unix-recuerdo/
20.- REF: https://wiki.alpinelinux.org/wiki/Alpine_Linux_Init_System
21.- REF: https://www.raspberrypi.org/documentation/configuration/led_blink_warnings.md
22.- REF: https://wiki.archlinux.org/index.php/Fstab_(Espa%C3%B1ol)
23.- REF: https://wiki.artixlinux.org/Main/OpenRC
~~~

~~~
- PREPARAMOS LA SD CARD

    - Tabla de particion -> dos

        - Primera particion [    256M  ] [ boot partition  ]  W95 FAT32 (LBA) * Boot
        - Segunda particion [ EL RESTO ] [ root filesystem ]  EXT4


$ sudo fdisk /dev/sda

Command          ( m for help     ):      o

Command          ( m for help     ):      n
Select           ( default p      ):      p
Partition number ( 1-4, default 1 ):      1
First sector                       : [ENTER]
Last  sector                       :  +256M

Command          ( m for help     ):      n
Select           ( default p      ):      p
Partition number ( 2-4, default 2 ):      2
First sector                       : [ENTER]
Last  sector                       : [ENTER]

Command          ( m for help     ):      p

Command          ( m for help     ):      t
Partition number ( 1,2, default 2 ):      1
Hex code                           :      c

Command          ( m for help     ):      a
Partition number ( 1,2, default 2 ):      1

Command          ( m for help     ):      p
Command          ( m for help     ):      w
~~~

~~~
$ sudo fdisk -l /dev/sda

Disk /dev/sda: 29.74 GiB, 31914983424 bytes, 62333952 sectors
Disk model: STORAGE DEVICE  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x68cd5f83

Device     Boot  Start      End  Sectors  Size Id Type
/dev/sda1  *      2048   526335   524288  256M  c W95 FAT32 (LBA)
/dev/sda2       526336 62332927 61806592 29.5G 83 Linux
~~~

~~~
- AHORA VAMOS A DARLE FORMATO A LAS PARTICIONES
- POR ALGUNA RAZON EN RPI SI UTILIZO LABEL NO ENCUENTRA LA PARTICION DE BOOT
- MEJOR EVITAR LOS LABEL

$ sudo mkfs.vfat -F32 -I  /dev/sda1
$ sudo mkfs.ext4          /dev/sda2

$ lsblk /dev/sda -o NAME,FSTYPE,LABEL,PARTUUID,UUID
~~~


~~~
- SOLO UTILIZAREMOS EL MINIROOTFS DE ALPINE, EN RPI4 EL KERNEL DE ALPINE 3.11
- TIENE ALGUNOS PROBLEMAS, ASI QUE COMPILAREMOS NUESTRO PROPIO KERNEL

$ ALPINECDN=http://dl-cdn.alpinelinux.org/alpine/v3.11

$ mkdir apks
$ cd apks

--- ROOTFS DE FEDORA

$ curl -LO ${ALPINECDN}/releases/aarch64/alpine-minirootfs-3.11.3-aarch64.tar.gz

--- REF: https://wiki.artixlinux.org/Main/OpenRC
--- SISTEMA DE INICIO BASADO EN DEPENDENCIAS

$ curl -LO ${ALPINECDN}/main/aarch64/openrc-0.42.1-r2.apk
$ curl -LO ${ALPINECDN}/main/aarch64/busybox-initscripts-3.2-r2.apk


--- REF: https://wiki.archlinux.org/index.php/D-Bus_(Espa%C3%B1ol)

$ curl -LO ${ALPINECDN}/main/aarch64/dbus-1.12.16-r2.apk
$ curl -LO ${ALPINECDN}/main/aarch64/dbus-libs-1.12.16-r2.apk
$ curl -LO ${ALPINECDN}/main/aarch64/dbus-openrc-1.12.16-r2.apk

--- DEPENDENCIA DE dbus

$ curl -LO ${ALPINECDN}/main/aarch64/expat-2.2.9-r1.apk

--- REF: https://wireless.wiki.kernel.org/en/developers/documentation/cfg80211
--- wireless-regdb ES REQUERIDO POR cfg80211
--- CFG80211 es la API de configuración de Linux 802.11.


$ curl -LO ${ALPINECDN}/main/aarch64/wireless-regdb-2018.05.31-r1.apk

--- REF: https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/
--- POR ULTIMO VAMOS A NECESITAR EL FIRMWARE DE BCM43455 WiFi/BT chip QUE ES EL
--- QUE UTILIZA LA RPI 4

$ curl -LO ${ALPINECDN}/main/aarch64/linux-firmware-brcm-20191215-r0.apk

--- WIRELESS-TOOLS

$ curl -LO ${ALPINECDN}/main/aarch64/wireless-tools-30_pre9-r1.apk
$ curl -LO ${ALPINECDN}/main/aarch64/wpa_supplicant-2.9-r5.apk


--- DEPENDENCIAS DE wpa_supplicant

$ curl -LO ${ALPINECDN}/main/aarch64/libnl3-3.5.0-r0.apk
$ curl -LO ${ALPINECDN}/main/aarch64/pcsc-lite-libs-1.8.25-r2.apk

~~~

~~~
- PREPARAMOS LA PARTICION DEL ROOTFS 

$ udisksctl mount -b /dev/sda2

$ sudo tar xfvz alpine-minirootfs-3.11.3-aarch64.tar.gz \
			-C /run/media/formatcom/ba6bc076-a6b3-42d7-b8ad-567b691a10e3 \
			--no-same-owner

--- REF: #13 Alpine Linux Init System
--- IGNORAR LOS MENSAJES APK-TOOLS.checksum.SHA1
$ zcat openrc-0.42.1-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat busybox-initscripts-3.2-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat wireless-regdb-2018.05.31-r1.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat linux-firmware-brcm-20191215-r0.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat wireless-tools-30_pre9-r1.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat wpa_supplicant-2.9-r5.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat dbus-1.12.16-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat dbus-openrc-1.12.16-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat dbus-libs-1.12.16-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat pcsc-lite-libs-1.8.25-r2.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat libnl3-3.5.0-r0.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -
$ zcat expat-2.2.9-r1.apk | sudo tar -C /run/media/formatcom/5d3a0812-99a2-458b-ba91-6590ca97195f -xvf -

$ cd /run/media/formatcom/7425d74e-aaef-436a-914d-300c79eb6927

--- AQUI EXPLICAR WIFI Y BLUETOOTH

--- CONFIGURAR EL WIFI PARA RPI4

$ cd lib/firmware/brcm
$ sudo ln -f -s /lib/firmware/brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.txt brcmfmac43455-sdio.txt

$ cd /run/media/formatcom/7425d74e-aaef-436a-914d-300c79eb6927

--- AGREGAR SERVICIOS AL INICIO

$ sudo ln -s /etc/init.d/{modules,sysctl,hostname,bootmisc,syslog,swclock} etc/runlevels/boot/
$ sudo ln -s /etc/init.d/{devfs,dmesg,hwdrivers} etc/runlevels/sysinit/
$ sudo ln -s /etc/init.d/{mount-ro,killprocs,savecache} etc/runlevels/shutdown/
$ sudo ln -s /etc/init.d/dbus etc/runlevels/default/

--- SI QUIERES VER COMO QUEDO
--- SALEN ROTOS PORQUE DEBEN TENER EN CUENTA QUE ESTA APUNTANDO A
--- /etc/init.d DEL ROOTFS QUE NOS ESTAMOS MONTANDO, ASI QUE ESTAN
--- EN etc/init.d EN NUESTRA SD PART 2.

$ tree etc/runlevels
$ ls etc/init.d

--- AHORA VAMOS A VERIFICAR QUE NUESTRO USUARIO ROOT NO ESTE BLOQUEADO

$ sudo cat etc/shadow

root:!::0:::::

--- SI TENEMOS EL RESULTADO ANTERIOR LE QUITAMOS EL !
--- ASI DESBLOQUEAMOS EL USUARIO ROOT

root:::0:::::

--- MODIFICAR EL ARCHIVO FSTAB

$ lsblk /dev/sda -o NAME,FSTYPE,LABEL,PARTUUID,UUID

- boot   vfat UUID 179E-DC05
- rootfs ext4 UUID eefdfc78-e179-4cb2-ac38-08f964e686d3

--- REF: #22
--- QUEDARIA ASI, EL ARCHIVO FSTAB

UUID=eefdfc78-e179-4cb2-ac38-08f964e686d3 /              ext4    defaults  0  1
UUID=179E-DC05                            /boot          vfat    defaults  0  0

--- AHORA CREAMOS LA CARPETA boot QUE NECESITA LA DEFINICION ANTERIOR

$ sudo mkdir /run/media/formatcom/7425d74e-aaef-436a-914d-300c79eb6927/boot

--- PARA MI CASO DE USO HABILITARE LA TERMINAL DEL PUERTO SERIE
--- EN EL ARCHIVO etc/inittab LO DEJAMOS COMO EL SIGUIENTE
--- REF: https://wiki.artixlinux.org/Main/OpenRC

# /etc/inittab

::sysinit:/sbin/openrc sysinit
::sysinit:/sbin/openrc boot
::wait:/sbin/openrc default

# Set up a couple of getty's
tty1::respawn:/sbin/getty 38400 tty1
tty2::respawn:/sbin/getty 38400 tty2
tty3::respawn:/sbin/getty 38400 tty3
tty4::respawn:/sbin/getty 38400 tty4
tty5::respawn:/sbin/getty 38400 tty5
tty6::respawn:/sbin/getty 38400 tty6

# Put a getty on the serial port
ttyS0::respawn:/sbin/getty -L 115200 ttyS0 vt100

# Stuff to do for the 3-finger salute
::ctrlaltdel:/sbin/reboot

# Stuff to do before rebooting
::shutdown:/sbin/openrc shutdown


--- EN EL ARCHIVO etc/securetty INDICAMOS QUE PODEMOS INICIAR SESION
--- DESDE LA TERMINAL SERIAL ttyS0. QUEDANDO DE LA SIGUIENTE MANERA

console
tty1
tty2
tty3
tty4
tty5
tty6
tty7
tty8
tty9
tty10
tty11
ttyS0

~~~

~~~
--- AHORA FUERA DE LA PARTICION DE ROOTFS TRABAJAMOS EN LA COMPILACION DEL KERNEL

REF: https://www.raspberrypi.org/documentation/configuration/config-txt/boot.md
REF: https://www.raspberrypi.org/documentation/linux/kernel/building.md
REF: https://github.com/raspberrypi/linux/branches/all

- COMPILAR EL KERNEL arm64

$ cd ~
$ git clone --depth=1 -b rpi-4.19.y https://github.com/raspberrypi/linux.git
$ mkdir boot
$ cd linux
$ KERNEL=kernel8
$ make O=../boot     ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
$ make O=../boot     ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig # opcional
$ make O=../boot -j4 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-


--- COPIAMOS LOS MODULOS EN LA PARTICION ROOTFS ext4

$ sudo make O=../boot -j4 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
	INSTALL_MOD_PATH=/run/media/formatcom/ba6bc076-a6b3-42d7-b8ad-567b691a10e3  \
	modules_install

--- YA TERMINAMOS CON LA PARTICION DE ROOTFS 
--- AHORA PASAMOS A TRABAJAR EN LA PARTICION DE BOOT

$ udisksctl unmount -b /dev/sda2
$ udisksctl mount   -b /dev/sda1

--- EL device tree EN LA PARTICION BOOT fat32

$ make O=../boot -j4 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
	INSTALL_DTBS_PATH=/run/media/formatcom/9E6E-326D  \
	dtbs_install

$ FIRMWARERPI=https://github.com/raspberrypi/firmware
$ curl -o /run/media/formatcom/FC06-D87F/start4.elf -LO ${FIRMWARERPI}/raw/master/boot/start4.elf
$ curl -o /run/media/formatcom/FC06-D87F/fixup4.dat -LO ${FIRMWARERPI}/raw/master/boot/fixup4.dat
~~~

~~~
--- COPIAMOS EL KERNEL Y SU CONFIGURACION
$ cd ..
$ cp boot/arch/arm64/boot/Image /run/media/formatcom/9E6E-326D/$KERNEL.img
$ cp config.txt cmdline.txt /run/media/formatcom/9E6E-326D

--- BUENO YA TENEMOS TODO LISTO YA SOLO NOS QUEDA PROBAR LA SD EN LA RPI

$ udisksctl unmount -b /dev/sda1
~~~

~~~
- CONECTARNOS POR EL UART

$ sudo screen /dev/ttyUSB0 115200


--- CONFIGURAR DBUS

# addgroup -S messagebus
# adduser -S -D -H -h /dev/null -s /sbin/nologin -G messagebus -g messagebus messagebus

# reboot

# rc-status default

--- APAGAR

# half
~~~

~~~
--- WIFI
--- REF: https://wiki.alpinelinux.org/wiki/Connecting_to_a_wireless_access_point

# ip link set wlan0 up
# iwlist wlan0 scanning | tee /tmp/wifi.info
# iwconfig wlan0 essid vodafone2088_5G
# iwconfig wlan0

# wpa_passphrase 'vodafone2088_5G' 'ExamWifiPassw' > /etc/wpa_supplicant/wpa_supplicant.conf
# wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

# udhcpc -i wlan0
# ip addr show wlan0

--- REF: http://dl-cdn.alpinelinux.org/alpine/v3.11/main/aarch64/
--- REF: https://www.raspberrypi.org/blog/vc4-and-v3d-opengl-drivers-for-raspberry-pi-an-update/
--- VideoCore VI driver is V3D

# apk add mesa-dri-v3d
# apk add mesa-demos

# apk add xorg-server

~~~

~~~

--- SDL2_gfx
$ ./configure --build=arm --disable-dependency-tracking --disable-mmx

~~~

~~~
- SOLO ES UNA NOTA PARA MI, NO LO UTILIZARE PARA NADA
- DESCOMPRIMIR initramfs-rpi4 DE ALPINE

$ zcat initramfs-rpi4 | cpio -ivd --no-absolute-filenames
~~~

~~~
$ sudo fsck.vfat -v -a -w /dev/sda1
~~~

