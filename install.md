# ArchLinux

<p>
Arch Linux es un sistema operativo basado en Linux, pero lo que lo hace especial es que está diseñado para usuarios que quieren tener el control total sobre su computadora. Es una distribución de "rolling release", lo que significa que siempre está recibiendo actualizaciones, en lugar de tener que esperar a nuevas versiones cada cierto tiempo.
</p>

## Coneccion a Internet

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
