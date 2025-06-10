# ArchLinux

<p>
Arch Linux es un sistema operativo basado en Linux, pero lo que lo hace especial es que está diseñado para usuarios que quieren tener el control total sobre su computadora. Es una distribución de "rolling release", lo que significa que siempre está recibiendo actualizaciones, en lugar de tener que esperar a nuevas versiones cada cierto tiempo.
</p>

## Como instalar Arch Linux en Dual Boot?

Si queremos instalar Arch en Dual Boot primero tenemos que asegurarnos de que nuestra computadora lo permita, para esto primero comprobamos que nuestro equipo existan los archivos de efivars, para esto en la terminal ejecutamos el siguiente comando:
 ``` 
ls /sys/firmware/efi/efivars
 ```

<p>
Ya que no contamos con una interfaz grafica, debemos realizar toda la instalacion mediante la consola. Dentro de la instalacion lo que debemos configurar sera la distribuicion del teclado de nuestro equipo y el idioma con el que queremos instarlo.
</p>

### Configuracion de teclado e Idioma

1. Si tu teclado tiene distribuicion en ingles, es decir, no tiene la letre **ñ** puedes saltarte este paso, en caso contrario ejecuta las siguientes lineas:
    - 1: Seteamos el teclado en español:
            ``` 
            loadkeys es
            ```
    - 2: Priorizamos el español para latinoamerica:
            ``` 
            loadkeys la-latin1
            ```

2. Para cambiar el idioma a español ejecutamos el seguiente comando:
        ```
        echo "es_ES.UTF-8 UTF_8" > /etc/locale.gen
        ```
    - 2.1: Genereamos todos los archivos en español:
            ```
            locale-gen
            ```
    - 2.2: Exportamos la nueva variable de idioma:
            ```
            export LANG=es_ES.UTF-8
            ```

### Conexion a Internet

<p>
Para conectarnos a internet mediante cable Etherner solo tenemos que conectarlo al puerto de red de nuestro equipo y listo. Sin embargo si queremos hacerlo con WiFi tendremos que hacer la conexion desde la terminal, para lo cual ocuparemos dos paquetes <strong>iwd</strong> y <strong>NetworkManager</strong>.
</p>

1. **iwd**
    - Obtener el nombre del adaptador de red.
        ``` 
        ls /sys/class/net | grep w
    - Inicializar iwd 
        ``` 
        systemctl start --now iwd 
    - Acceder a iwd
        ``` 
        iwctl
    - Escanear redes WiFi y listarlas
        ``` 
        station nombre_adaptador scan
        ```     
        ```
        station nombre_adaptador get-networks
    - Conexion con una red WiFi
        ```
        --passphrase "contrasena_red" station nombre_adaptador connect "nombre_red"
        ```

1. **NetworkManager**
    - Inicializar nmcli 
        ``` 
        systemctl start --now NetworkManager 
    - Escanear redes WiFi y listarlas
        ``` 
        nmcli dev wifi list
    - Conexion con una red WiFi
        ```
        nmcli device wifi connect "nombre_red" password "contrasena_red"
        ```

### Particionado del Disco
1. Listamos los discos disponibles: 
        ``` 
        fdisk -l
        ```
    
    1.1.Nos apareceran todos los discos disponibles en nuestro equipo, si ya lo tenemos particionado sera facil reconocer el disco, en caso de que no buscamos el nombre o marca de nuestro disco princi
2. Ya con el nombre del disco accedemos a el con **cfdisk**: 
        ``` 
        cfdisk /dev/sda
        ```     
3. **Particiones**. En lo que son las particiones vamos a tener que crear 2 particiones nuevas:

    3.1. Una particion de al menos 8 Gb para swap, y en tipo de particion colocamos **linux swap**

    3.2. La particion principal para linux, esta puede ser del espacio libre restante en nuestro disco y sera de tipo **Sistema de ficheros de linux**

### Formato y Montaje de las particiones
Una ves que tenemos las particiones listas tendremos que pasar a montarlas para que linux las reconozca
1. La particion con tipo **Sistema de ficheros de linux** la vamos a formatear con Fourth Extended Filesystem (Ext4) y la montaremos en /mnt/ :
    - Formateo:
        ``` 
        mkfs.ext4 /dev/particion_root
        ```
    - Montaje: 
        ``` 
        mount /dev/particion_root /mnt/
        ```
2. Con la particion para Linux Swap igualmente tenemos que formatearla y activarla:
    - Formateo:
        ``` 
        mkswap /dev/particion_swap
        ```
    - Montaje: 
        ``` 
        swapon /dev/particion_swap
        ```
3. Para la particion EFI vamos a ocupar la misma que crea Windows por defecto solo que la montaremos en una ubicacion en especifico:
    - Creacion de la carpeta de montaje:
    ```
    mkdir -p /mnt/boot/efi
    ```
    - Montaje:
    ```
    mount /dev/particion_efi /mnt/boot/efi
    ```

## Instalacion
Para comenzar con la instalacion descargaremos e instalaremos **pacstrap** y con este se hara una instalacion basica de ArchLinux.
```
pacstrap /mnt base base-devel nano
```

### Kernel de linux
Para instalar un kernel tenemos una gran variedad de kernels, para saber mas sobre estos y decidir mejor cual instalar [visita el siguiente](https://wiki.archlinux.org/title/Kernel). En mi caso instalare el kernel **zen**.

```
pacstrap /mnt linux-firmware linux-zen linux-zen-headers mkinitcpio
```

### Montaje de los discos
A pesar de que ya hallamos montado los discos para la instalacion, al momento de apagar y prender nuestra computadora estos se desmontan por lo que es necesario volver a montarlos, para lo cual usaremos una herramienta llamada **genfstab**.
```
genfstab -p /mnt >> /mnt/etc/fstab
```
Ya que ejecutamos este comando al ver el contenid0o de la carpeta **fstab** con
    ```
    cat /mnt/etc/fstab
    ```
veremos el orden en el que se van a ir montando las particiones de nuestro disco al arranque de la computadora.

## Segunda fase de instalacion.
Ya que llegamos a este punto tenemos un Arch muy basico instalado por lo que lo siguiente sera hacer una instalacion sobre esta instalacion, suena un poco confuso, pero hasta este punto seguimos en una instalacion en un disco externo de arranque ya sea una memoria usb, cd o el inicio de una maquina virtual, pero de aqui en adelante ya haremos la instalacion sobre la particion que montamos de nuestro disco por lo que esto ya empezara a darle forma a nuestra instalacion de Arch.

Para entrar a esta segunda fase vamos a entrar con **arch-chroot** a nuestro punto de montaje estandar de linux: 
    ```
    arch-chroot /mnt
    ```

Si configuramos la instalacion en español y con una distribucion de teclado en español vamos a repetir los primeros pasos en la parte de **Configuracion de teclado e Idioma**, en caso contrario podemos seguir.

### Configuracion de la Zona Horaria
Para saber la zona horaria en la que estamos con exactitud antes de hacer este paso [entra al siguiente link](https://ipapi.co/timezone), esto nos dara la zona horario que tenemos que configurar.
-  Ejecuta el siguiente comando:
    ```
    ln -sf /usr/share/zoneinfo/America/zona_horaria_obtenida
    ```
- Ahora para comprobar que este cambio se aplico ejecutamos el siguiente comando:
    ```
    ls -l /etc/localtime
    ```
    y aqui deberemos de ver el enlace que hicimos en el paso anterior junto con la zona horaria real que obtuvimos.
- Por ultimo para sincronizar los relojes utilizamos **hwclock -w**

### Nombre de usuario y de equipo: 
1. Para configurar el nombre del equipo y el nombre del usuario lo que tenemos que hacer sera mandar el nombre de estos a un archivo en especifico.
    - 1. Para el nombre del equipo mandaremos el valor del nombre al archivo **hostname**:
                ```
                echo nombre_equipo > /etc/hostname
                ```
    y vemos que exista el nombre ejecutando el comando cat hacia el archivo que acabamos de configurar:
                ```
                cat /etc/hostname
                ```
    - 2. Para agregar un nuevo usuario vamos a utilizar el comando useradd, este agregara al usuario con ciertos privilegios, puedes leer la documentacion mas detallada [aqui](https://wiki.archlinux.org/title/Users_and_groups). Para hacer esto ejecuta el siguiente comando:
                ```
                useradd -m -g users -G wheel -s /bin/zsh nombre_usuario
                ```
    - 3. Ahora lo que tenemos que hacer sera darle una contrasena al usuario que acabamos de crear y al usuario root para esto ejecutamos el siguiente comando:
                ```
                passwd nombre_usuario
                ```
    En **nombre_usuario** lo vamos a cambiar por el usuario que acabamos de agregar y por root para actualizar su contrasena.
    - 4. Para finalizar con los usuarios lo que tenemos que hacer sera darle acceso a root al nuevo usuario para esto vamos a modificar el archivo **/etc/sudoers** con algun editor de texto. Una vez abierto el archivo vamos a buscar la seccion de especificacion de privilegios de usuario, dependiendo de como seteamos el idioma nos aparecera en ingles o espanol, ya que encontremos esta seccion vamos a crear una nueva linea igual a la que tenemos con el usuario de root, solo que en lugar de poner **root** vamos a poner el usuario que creamos, ejemplo: 
                ```
                nombre_usuario ALL=(AL:ALL) ALL
                ```

### Instalacion de herramientas de red.
Para poder gestionar la conexion a internet vamos a utilizar varias herramientas, las que para instalarlas ejecutamos el siguiente comando:
```
pacman -S dhcp dhcpcd networkmanager iwd
```
Y para que estas se activen al arranque o encendido de nuestra computadora ejecutamos lo siguiente:
```
systemctl enable dhcpcd NetworkManager iwd
```

## Configuracion del Gestor de Arranque GRUB
El gestor de arranque sera una herramienta escencial si tenemos una instalacion en Dual Boot junto con Windows. Este nos permitira tener un menu de arranque al encender nuestro equipo y seleccionar entre usar Windows o ArchLinux.

Para esto instalamos los siguientes paquetes:
```
pacman -S grub efibootmgr os-prober
```
### Instalacion de GRUB
Para la instalacion y configuracion de este, para lo cual seguiremos los siguientes pasos:

1. ``` 
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
    ```
2. ``` 
    grub-install --target=x86_64-efi --efi-directory=/boot/efi removable
    ```
3. Ahora vamos a hacer una configuracion en el archivo de modificacion de arch, para esto entramos al siguiente archivo
        ```
        /etc/default/grub
        ```
        . Ya con el archivo abierto vamos a bajar hasta encontrar una opcion que diga **GRUB_DISABLE_OS_PROBER=false**, por lo comun esta opcion se encuentra comentada con un signo de gato, entonces para que empiece a afectar al sistema la descomentamos y guardamos cambios en el archivo.
4. Para usar en el arranque el archivo de configuracion que acabamos de crear ejecutamos lo siguiente:
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Creacion de directorios para los usuarios.
A pesar de que hasta este punto ya tenemos una instalacion casi completada, aun no tenemos los directorios comunes como **Escritorio**, **Descargas**, etc. Entonces para la creacion de estos seguimos los siguientes pasos.
1. Instala el paquete 
        ```
        xdg-user-dirs
        ```.
2. Para actualizar los directorios del usuario **root** solo ejecutamos el siguiente comando:
        ```
        xdg-user-dirs-update
        ```. Y para actualizar los del usuario que creamos en pasos anteriores ejecutamos lo siguiente: 
        ```
        su nombre_usuario -c ""xdg-user-dirs-update"
        ```
## Controlador de Audio y de Video.
### Instalaciones recomentadas.
Antes de proceder con la instalacion de controladores de video puedes instalar paquetes como git, alguna terminal como kitty u otra de tu preferencia, en este caso vamos a instalar git, kitty, lsb-release, fastfetch y algunas fuentes para que cuando ya tengamos un entorno grafico veamos de la forma correcta las ventanas u opciones.

```
pacman -S git kitty lsb-release fastfetch gnu-free-fonts ttf-hack ttf-inconsolata noto-fonts-emoji
```

Lo siguiente sera salir de la instalacion ejecutando 
    ```
    exit
    ``` en la terminal. Desmontamos las particiones montadas 
    ```
    umount -R /mnt 
    ``` y para finalizar reiniciamos nuestro equipo con 
    ```
    reboot
    ```.

### Controlador de Video.
Dependiendo del procesador que tengas ya sea Intel o AMD va a cambiar la forma en que instalas los controladores. Para saber mas lee la documentacion dependiendo del tipo de procesador y tarjeta grafica que tengas. Para procesadores [Intel](https://wiki.archlinux.org/title/Intel_graphics). Procesadores AMD [Codigo Abierto](https://wiki.archlinux.org/title/AMDGPU) [ATI](https://wiki.archlinux.org/title/ATI), Controladores [Privativos](https://wiki.archlinux.org/title/AMDGPU_PRO) y Tarjetas de Video [Nvidia](https://wiki.archlinux.org/title/NVIDIA).

Si estas haciendo la instalacion de prueba en alguna maquina virtual, prueba instalar los siguentes controladores.
```
sudo pacman -S xf86-video-vesa xf86-video-fbdev
```

### Controlador de Audio.
Los controladores de audio son un poco mas estandar, entonces no va a afectar el tipo de procesador que tengas 
