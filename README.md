# DHCP

## Pasos antes de empezar

### Máquinas virtuales
Primero tendremos que, en la máquina que usaremos como servidor, tener en los ajuste de red un adaptador en modo NAT y otro en modo puente permitiendo máquinas virtuales ya que vamos a usar un cliente como prueba más adelante. 

### Apartado 1 Instalar paquete isc-dhcp-server

Tendremos que usar el comando:

$ sudo apt install isc-dhcp-server

Tendremos que tirar el adaptador enp0s3 una vez instalado el dhcp SERVER para evitar IPs conflictivas y facilitar la configuración inicial.


### Apartado 2 y 4 Editar para una configuración básica /etc/dhcp/dhcpd.conf

Tendremos que entrar a ese archivo con:

$ sudo nano /etc/dhcp/dhcpd.conf

y comentaremos con "#" las líneas de "option domain-name "example.org";" y la que está debajo de esta. Ahora tendremos que descomentar la línea de "authoritative" que esto indica que el server DHCP es la autoridad por excelencia en asignación de IPs.

Ahora tendremos que indicarle el nombre de la subred con lo siguiente:

subnet 172.16.0.0 netmask 255.255.0.0 {
    range 172.16.0.2 172.16.0.254; (Este es el rango de IPs dentro de la subred)
    option subnet-mask 255.255.0.0;
    option broadcast-addres 172.16.0.255;
    option router 172.16.0.1; 
    option domain-name-servers 8.8.8.8, 4.4.4.4;
    option domain-name "joseserver";
}

### Apartado 3 Editar para configurar la interfaz de red /etc/Default/isc-dchp-server

En este archivo tendremos que poner el nombre del adaptador "enp0s8" en la línea de INTERFACESv4="".
Se debe poner entre esas comillas.

### Apartados 5 y 6 Arranca el servicio con systemctl y comprueba el servicio con "systemctl status"

Para arrancar el servicio pondremos:
$ sudo systemctl start isc-dhcp-server
Para comprobar el estado pondremos:
$ sudo systemctl status isc-dhcp-server

Si hemos configurado bien todo lo anterior, cuando lancemos el comando de "status", debe poner ACTIVE (RUNNING)

Antes de comprobar con el cliente haremos:
$ sudo netplan apply


### 7 Prueba con el cliente que se le asigna un ip en el rango 

Para ello usaremos otra máquina virtual ubuntu con red interna y permitinedo máquinas virtuales en su configuración. Entraremos dentro y si hemos hecho todo bien veremos que tenemos internet mientras nuestra máquina que hace de server esté encendida.

Para ver el rango de IP que nos han asignado usaremos "ip a" en la terminal.

### Aprtado 8 y 9 Declarar una asignación por mac fija a 172.16.0.5 y comprobar con cliente

Modificaremos el archivo dhcpd.conf:

$ sudo nano /etc/dhcp/dhcpd.conf

Añadiremos una línea al final que será la siguiente:

host joseserver {
    hardawre ethernet 08:00:27:0d:f0:18;
    fixed-address 172.16.0.5;
}

Y haremos un $sudo service isc-dhcp-server restart.

### 10 Comprueba con el wireshark los mensajes del protocolo

Si iniciamos el wireshark en el cliente y hacemos un ping ya sea desde el servidor al cliente o del cliente al servidor, el protocolo mostrado en WIRESHARK es el ICMP. 