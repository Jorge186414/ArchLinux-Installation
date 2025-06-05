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

#### Formato y Montaje de las particiones
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