# ArchInstalacion

# Guía de Instalación de Arch Linux en un Portátil (SSD 500GB)

**Paso 1: Arrancar desde el medio de instalación de Arch Linux**

1.  Asegúrate de haber creado una unidad USB de instalación de Arch Linux.
2.  Enciende tu computadora y entra en la BIOS/UEFI.
3.  Configura el orden de arranque para que la unidad USB sea la primera opción.
4.  Guarda los cambios y reinicia tu computadora. Deberías ver el menú de arranque de Arch Linux.
5.  Selecciona la primera opción ("Arch Linux install medium") y presiona Enter.

**Paso 2: Conectar a internet**

1.  **Verificar la conexión inalámbrica (si es necesario):**
    ```bash
    iwctl
    device list
    station NOMBRE_DISPOSITIVO scan  # Reemplaza NOMBRE_DISPOSITIVO con el nombre de tu interfaz inalámbrica
    station NOMBRE_DISPOSITIVO get-networks
    station NOMBRE_DISPOSITIVO connect NOMBRE_DE_TU_ROUTER
    exit
    ```
2.  **Verificar la conexión a internet:**
    ```bash
    ping archlinux.org
    ```
3.  **Activar el servicio de tiempo de red (NTP):**
    ```bash
    timedatectl set-ntp true
    ```

**Paso 3: Particionar el disco NVMe/SATA (`/dev/sda` o `/dev/nvme0n1` - verifica con `lsblk`)**

```bash
cfdisk /dev/tu_disco
Dentro de cfdisk:

EFI (512MB): New -> 512M -> EFI System -> Write (yes)
Swap (4GB): Selecciona espacio libre -> New -> 4G -> Linux swap -> Write (yes)
Raíz (80GB): Selecciona espacio libre -> New -> 80G -> Linux filesystem -> Write (yes)
Home (Resto del espacio): Selecciona espacio libre -> New -> (Enter) -> Linux filesystem -> Write (yes)
Quit
Paso 4: Formatear las particiones

Bash

mkfs.fat -F32 /dev/tu_disco1   # Partición EFI
mkswap /dev/tu_disco2          # Partición Swap
mkfs.ext4 /dev/tu_disco3         # Partición Raíz
mkfs.ext4 /dev/tu_disco4         # Partición Home
Paso 5: Montar los sistemas de archivos

Bash

mount /dev/tu_disco3 /mnt
mkdir -p /mnt/home
mount /dev/tu_disco4 /mnt/home
mkdir -p /mnt/boot/efi
mount /dev/tu_disco1 /mnt/boot/efi
swapon /dev/tu_disco2
Paso 6: Instalar el sistema base

Bash

pacstrap /mnt base linux linux-firmware nano dhcpcd
Paso 7: Generar el archivo fstab

Bash

genfstab -U /mnt >> /mnt/etc/fstab
Paso 8: Entrar en el entorno chroot

Bash

arch-chroot /mnt
Paso 9: Configurar el sistema

Zona horaria: ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime && hwclock --systohc
Localización: Editar /etc/locale.gen, generar con locale-gen, crear /etc/locale.conf (LANG=es_ES.UTF-8).
Mapa de teclado (opcional): Crear /etc/vconsole.conf (KEYMAP=es).
Nombre del host: echo "miarch" > /etc/hostname y editar /etc/hosts.
Contraseña de root: passwd.
Instalar y habilitar NetworkManager (o dhcpcd).
Instalar GRUB y efibootmgr:
Bash

pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch
grub-mkconfig -o /boot/grub/grub.cfg
Crear usuario (opcional): useradd -m usuario, passwd usuario, usermod -aG wheel,audio,video,storage usuario.
Instalar y configurar sudo (opcional): pacman -S sudo, editar /etc/sudoers (descomentar %wheel ALL=(ALL) ALL).
Paso 10: Salir del chroot, desmontar y reiniciar

Bash

exit
umount -R /mnt
reboot
