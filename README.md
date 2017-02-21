# Установка и настройка Gentoo

## Установка

Конфигурация таблицы разделов:

    sda      8:0    0 447.1G  0 disk 
    ├─sda4   8:4    0 345.5G  0 part /home
    ├─sda2   8:2    0     2G  0 part [SWAP]
    ├─sda3   8:3    0  97.7G  0 part /
    └─sda1   8:1    0   2.1G  0 part /boot

Содержимое файла `/etc/fstab` после установки ОС:

    /dev/sda1       /boot   vfat    defaults                0 2
    /dev/sda2       none    swap    sw                      0 0
    /dev/sda3       /       ext4    noatime,discard         0 1
    /dev/sda4       /home   ext4    noatime,discard         0 1
    
Процесс установки Gentoo:

    Загрузиться в режиме BIOS
    boot: gentoo
    
Настройка сетевого интерфейса:
    
    # net-setup enp3s0
    # ping ya.ru

Разметка жесткого диска:
    
    # parted -a optimal /dev/sda
    
        mklabel gpt
        unit mib
        
        mkpart primary 1 129
        name 1 boot
        
        mkpart primary 129 4129
        name 2 swap
        
        mkpart primary 4129 104129
        name 3 rootfs
        
        mkpart primary 104129 -1
        name 4 home
        
        set 1 boot on
        quit

Создание файловой системы:
        
    # mkfs.fat -F 32 /dev/sda1
    # mkfs.ext4 /dev/sda3
    # mkfs.ext4 /dev/sda4
    # mkswap /dev/sda2
    # swapon /dev/sda2
    
Монтирование разделов:
    
    # mount /dev/sda3 /mnt/gentoo
    # mount /dev/sda4 /mnt/home
    # mount /dev/sda1 /mnt/boot
    
Синхронизация времени:
    
    # ntpd -q -g
    
Разворачивание образа stage3:
    
    # cd /mnt/gentoo
    # links https://www.gentoo.org/downloads/mirrors
        stage3-amd64-20170209.tar.bz2
    # tar xvjpf stage3-*.tar.bz2 --xattrs
    
Выбор зеркал:
    
    # mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
    # mkdir /mnt/gentoo/etc/portage/repos.conf
    
Дополнительные настройки:
    
    # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
    # cp -L /etc/resolv.conf /mnt/gentoo/etc
    
Монтирование системных разделов:
    
    # mount -t proc /proc /mnt/gentoo/proc
    # mount --rbind /sys  /mnt/gentoo/sys
    # mount --make-rslave /mnt/gentoo/sys
    # mount --rbind /dev  /mnt/gentoo/dev
    # mount --make-rslave /mnt/gentoo/dev
    
Смена корневой директории:
    
    # chroot /mnt/gentoo /bin/bash
    # source /etc/profile
    # export PS1="(chroot) $PS1"
    
Синхронизация и выбор профиля:
    
    # emerge-webrsync
    # eselect profile list
    # eselect profile set N
    # emerge --ask --update --deep --newuse @world
    
Настройка временной зоны и локали:
    
    # echo "Europe/Moscow" > /etc/timezone
    # emerge --config sys-libs/timezone-data
    # nano -w /etc/locale.gen
    # eselect locale list
    # eselect locale set N
    # env-update && source /etc/profile && export PS1="(chroot) $PS1"
    
Сборка ядра:
    
    # emerge --ask sys-kernel/gentoo-sources
    # emerge --ask sys-kernel/genkernel-next
    
    # nano -w /etc/fstab
        /dev/sda1 /boot vfat defaults 0 2
    
    # genkernel --loglevel=3 all
    # emerge --ask sys-kernel/linux-firmware

Конфигурация сетевого интерфейса:

    # nano -w /etc/conf.d/hostname
    # nano -w /etc/conf.d/net
        config_enp3s0="dhcp"
    # emerge --ask net-misc/dhcpcd
        
Смена пароля root:

    # passwd
    
Установка загрузчика:

    # echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
    # emerge --ask sys-boot/grub:2
    # grub-install --target=x86_64-efi --efi-directory=/boot
    # mkdir /boot/EFI/BOOT
    # cp /boot/EFI/gentoo/grubx64.efi /boot/EFI/BOOT/BOOTX64.EFI
    # grub-mkconfig -o /boot/grub/grub.cfg
    
Выход из `chroot` и перезагрузка:

    # exit
    # cd
    # umount -l /mnt/gentoo/dev{/shm,/pts,}
    # umount -R /mnt/gentoo
    # reboot

В процессе перезагрузки:

    Сменить режим загрузки с BIOS на UEFI
    
После загрузки в режиме UEFI:

    # grub-install --target=x86_64-efi --efi-directory=/boot
    # rm /boot/EFI/BOOT/BOOTX64.EFI
    # rmdir /boot/EFI/BOOT
    # reboot
    
### Восстановление процесса установки
    
    # net-setup enp3s0
    # swapon /dev/sda2
    # mount /dev/sda3 /mnt/gentoo
    # mount /dev/sda1 /mnt/gentoo/boot
    # mount /dev/sda4 /mnt/gentoo/home
    # mount -t proc /proc /mnt/gentoo/proc
    # mount --rbind /sys  /mnt/gentoo/sys
    # mount --make-rslave /mnt/gentoo/sys
    # mount --rbind /dev  /mnt/gentoo/dev
    # mount --make-rslave /mnt/gentoo/dev
    # chroot /mnt/gentoo /bin/bash
    # source /etc/profile
    # export PS1="(chroot) $PS1"
    
## Настройка

### Перезапуск ALSA

    $ sudo systemctl restart alsa-restore

### Настройка USB-наушников

Список всех звуковых карт:

    $ cat /proc/asound/cards

### Добавление нового пользователя

    # useradd -m -G users,wheel,audio,video,sudo -s /bin/bash <user>

### Настройка sudo

Установить пакет:

    # emerge --ask app-admin/sudo

Запустить редактор файла `/etc/sudoers`:

    # visudo

Добавить следующие строки, чтобы пользователи в группе `sudo` могли выполнять
любые команды:

    ## Uncomment to allow members of group sudo to execute any command
    %sudo   ALL=(ALL) ALL

Добавить группу `sudo`:

    # groupadd sudo

Добавить нужно пользователя в группу `sudo`:

    # usermod -a -G sudo <user>

Для включения логирования необходимо запустить редактор

    # visudo
    
и добавить строку:

    Defaults logfile=/var/log/sudo.log
    
### Отключение NCQ

Открыть файл:

    /etc/default/grub
    
И добавить в переменную `GRUB_CMDLINE_LINUX` следующую строку:

    libata.force=noncq

Обновить конфигурационный файл grub:

    # grub-mkconfig -o /boot/grub/grub.cfg
    
### Включение systemd

Проверить, установлен ли `systemd` в качестве менеджера:

    # ps -p 1 -o comm=

Открыть файл:

    /etc/default/grub

Раскомментировать строку:

    # Boot with systemd instead of sysvinit (openrc)
    GRUB_CMDLINE_LINUX="init=/usr/lib/systemd/systemd"

Заново сгенерировать `grub.cfg`:

    # grub-mkconfig -o /boot/grub/grub.cfg

### Автоматическая конфигурация сети через DHCP

Создать файл

    /etc/systemd/network/50-dhcp.network

со следюущим содержанием:
    
    [Match]
    Name=en*

    [Network]
    DHCP=yes

Включить автоматическую настройку сети:
    
    # systemctl enable systemd-networkd

### Проверка драйвера видеокарты

Установка и настройка драйверов:

    # emerge --ask x11-drivers/ati-drivers
    # eselect opengl list
    # eselect opengl set N

Проверка наличия устройства:

    # lspci | grep VGA

Проверка загрузки драйвера ядра:

    $ find /dev -group video

Проверка загрузки драйвера X:

    # emerge --ask x11-apps/mesa-progs
    # glxinfo | grep -i vendor

Подключение драйвера X (не сработало):

    $ nano -w /etc/X11/xorg.conf.d/10-radeon.conf
    
        Section "Device"
            Identifier  "radeon"
            Driver      "radeon"
        EndSection

Проверка наличия аппаратного 3D-ускорения:

    $ glxinfo | grep -i rendering

### Тормозит KDE

Установить параметр:

    System Settings -> Display and Monitor -> Compositor
        Rendering backend -> XRender
    
