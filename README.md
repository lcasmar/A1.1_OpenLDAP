# Open LDAP
## Índice
- Instalación y configuración de VM
  - Creación máquinas
  - Configuración lcmUbuntuServer
    - Configuración DHCP
    - Configuración router
  - Configuración lcmUbuntuCliente
- Configuración dominio
- Instalación y configuración de LDAP
  - Creación UO
  - Creación de grupos
  - Creación de usuarios

## Instalación y configuración de VM
Vamos a crear dos máquinas virtuales según la tabla: 
|VM|SO|red|
|--|--|-|
|A1.1-Ubuntu_Cliente|Ubuntu 24.04| 1 adaptador (red interna)|
|A1.1-Ubuntu_Server|Ubuntu Server 24.04| 2 adaptadores (red interna,adaptador puente)|

### Creación máquinas
Generamos dos máquinas nuevas en VBox    
![](img/A1.1_intC1.png) ![](img/A1.1_intS1.png)  
Configuramos ambas máquinas con suficiente RAM y CPU para la práctica, en nuestro caso asignamos 4GB de RAM y 2 CPU  
![](img/A1.1_intC2.png) ![](img/A1.1_intC3.png)

Una vez nuestras máquinas están creadas vamos a establecer la configuración de red  
Para la máquina A1.1-Ubuntu_Cliente establecemos un único adaptador de red en modo **Red interna** *ASIR_local*  
![](img/A1.1_intC4.png)  
Para la máquina A1.1-Ubuntu_Server establecemos dos adaptadores de red, uno en modo **Red interna** *ASIR_local* y otro en modo **Adaptador Puente**  
![](img/A1.1_intS4.png) ![](img/A1.1_intS4.2.png)

Iniciamos las máquinas e instalamos los correspondientes SO.
*NOTA: Es recomendable apuntar las MAC de los correspondientes adaptadores de red de cada máquina para facilitarnos el trabajo más adelante*

### Configuración lcmUbuntuServer
Para nuestra máquina server debemos gestionar varias configuraciones adicionales para poder realizar la práctica. Además la instalación del SO Ubuntu Server difiere bastante de la instalación de un Ubuntu regular escritorio, por eso vamos a pararnos un poco en la instalación del SO.  
*Destacar que los pasos que no se muestren en capturas es debido a que se deja la opción por defecto.*

Arrancamos la máquina e instalamos el so:  
Configuramos preferencias de versión, idioma y teclado.  
![](img/A1.1_intS5.1.png)  
![](img/A1.1_intS5.3.png) 
![](img/A1.1_intS5.2.png)  
Configuramos red, es importante configurar el adaptador que este en modo red interna con una IP fija, en este caso asignamos la IP *192.168.11.11*   
![](img/A1.1_intS5.4.png)   
Verificamos como para el otro adAptador de red se configura una ip de forma automática    
![](img/A1.1_intS5.5.png)  
Dejamos la configuración del proxy blanco cuando nos la solicite   
![](img/A1.1_intS5.6.png)  
Configuramos para que utilice el disco que le hemos asignado  
![](img/A1.1_intS5.7.png)  
Configuramos nombre del servidor, usuario y contraseña  
![](img/A1.1_intS5.8.png)  
Instalamos y comprobamos que podemos acceder con usuario y contraseña  
![](img/A1.1_intS5.9.png)  
Comprobamos la configuración de red con el comando *ip a* que muestra la configuración de red de los adaptadores del equipo  
![](img/A1.1_intS5.10.png)    

#### Configuración servidor DHCP
Configuramos ahora el server para que actúe como servidor DHCP:
Instalamos los paquetes necesarios *isc-DHCP-server*  
![](img/A1.1_intS6.1.png)  
Configuramos el servidor para que actúe como servidor DHCP y asigne IP's dentro del rango 192.168.11.10/192.168.11.20    
![](img/A1.1_dhcp_conf1.png)  
Ahora vamos a indicarle al servidor en que adaptador de red debe funcionar el servicio DHCP, en nuestro caso va a ser por la interfaz enp0s8 que es la que tenemos configurada en red interna y va a conectar con nuestro cliente  
![](img/A1.1_dhcp_conf2.png)  
Reiniciamos el servicio DHCP  
![](img/A1.1_dhcp_conf3.png)  
El servicio consta como *enabled* por lo que verificamos que funciona y que debería de estar asignando ip's a nuestros clientes  
![](img/A1.1_dhcp_conf4.png)  

### Configuración servidor como router
Tras comprobar que nuestra configuración de red es correcta procedemos a configurar nuestro server como router para que nuestro ubuntu cliente tenga salida a internet.  
Accedemos al archivo ip_forward en la ruta /proc/sys/net/ipv4/ip_forward y comprobamos que consta valor 0, es decir que no va a permitir pasar paquetes  
![](IMG/A1.1_routerS1.png)  

Para modificar este valor lo hacemos desde el fichero /etc/sysctl.conf  
Descomentamos la linea correspondiente  
![](img/A1.1_routerS2.png)  
Forzamos el cambio a 1 con *sysctl -p*, esta es una herramienta para modificar parámetros del kernel en tiempo real y/o mostrar sus valores actuales, la opción -p indica que cargue y aplique configuraciones desde un archivo.  
![](img/A1.1_routerS3.png)  
Configuración iptables a través del comando *iptables -L -nv -t nat*, iptables es una herramienta para configurar el firewall en sistemas Linux, -L indica que queremos las reglas de firewall actuales, -n evita que iptables resuelva direcciones IP a nombres host, -v muestra información detallada y -t nat especifica la tabla en la que estamos interesados que es la tabla NAT  
![](img/A1.1_routerS4.png)  
![](img/A1.1_routerS4.1.png)  
Para evitar que se deshagan los cambios instalamos *iptables-persistent* de forma que la configuración se guarde en un archivo /etc/iptables/rules.v4 y no se pierda con los reinicios  
![](img/A1.1_routerS4.2.png)  
![](img/A1.1_routerS4.3.png)  

### Configuración lcmUbuntuCliente
Después de instalar el so Ubuntu  desktop 24.04 LTS en la máquina cliente comprobamos la configuración de red de esta.
Por defecto la configuración de red viene predeterminada de forma automática, comprobamos que al consultar la ip de nuestro cliente esta ya ha sido asignada por nuestro servidor DHCP.  
![](img/A1.1_intC5.png)  
Verificamos así definitivamente que nuestro server DHCP funciona.    
Comprobamos de todos modos la conexión entre las máquinas mediante los respectivos pings   
![](img/A1.1_intC5.2.png)  

Comprobamos que nuestra máquina cliente tiene acceso a internet , por lo que nuestro router funciona.  
![](img/A1.1_routerS5.png)

## Configuración dominio
Para poder instalar LDAP necesitamos primero crear un dominio, en nuestro caso vamos a llamar a nuestro dominio *lcmLDAP*.
Modificamos el hostname en el server 
![](img/A1.1_dominio_S1.png)  
También modificamos el archivo /etc/hosts añadiendo las lineas correspondientes a nuestro servidor  
![](img/A1.1_dominio_S2.png)  
Actualizamos repositorios tanto en el cliente como en el servidor    
![](img/A1.1_dominio_S3.png)  
![](img/A1.1_dominio_S4.png)

## Instalación y configuración de LDAP
Instalamos *SLAPD LDAP-UTILS*  
![](img/A1.1_LDAPS1.png)
Una vez termine la instalación se va a ejecutar.  
Introducimos contraseña cuando nos lo solicite  
![](img/A1.1_LDAPS2.png)

Ahora nos va a preguntar si queremos crear un servidor ldap de cero, marcamos <No> para construir nuestro servidor ldap  
![](img/A1.1_LDAPS4.png)

Configuramos nombre de dominio DNS como lcmUbuntuServer.local
![](img/A1.1_LDAPS5.png)
Configuramos nombre organización  
![](img/A1.1_LDAPS6.png)
Configuramos contraseña  
![](img/A1.1_LDAPS7.png)
Marcamos opciones por defecto  
![](img/A1.1_LDAPS8.png)
![](img/A1.1_LDAPS9.png)

Comprobamos la info de nuestra configuración ldap con el comando *slapcat*
![](img/A1.1_LDAPS10.png)

Vamos a crear en el directorio raíz un directorio llamado ou para tener ahí nuestras plantillas de creación de servicios en LDAP.  
![](img/a1.1_ldalp_ou.png)  

Ahora vamos a crear nuestros elementos en LDAP.  
Para cada elemento que creemos en LDAP vamos a tener una plantilla en formato *ldif*, en estas plantillas definimos objetos con atributos.  
**dn: define el DN del elemento definido.  **  
**objectClass: Define de que tipo es el objeto.  **   

### Creación de OU
Creamos en el directorio **ou** un archivo *ou.ldif* para nuestras ou  
![](img/a1.1_ldalp_ou2.png)
![](img/a1.1_ldalp_ou3.png)
Importamos el fichero al servidor LDAP  
![](img/a1.1_ldalp_ou4.png)

### Creación de grupos
Creamos en el directorio **ou** un archivo *group.ldif* para nuestros grupos   
![](img/A1.1_ou_grupo.png)
![](img/A1.1_ou_grupo1.png)  
Cargamos los cambios
![](img/A1.1_ou_grupo2.png)  
Usamos la misma plantilla *group.ldif* para crear todos los grupos  
![](img/A1.1_ou_grupo3.png)    
Comprobamos que se han creado todos los grupos con el comando *ldapsearch*  
![](img/A1.1_ou_grupo4.png)    

### Creación de usuarios
Creamos en el directorio **ou** el archivo *users.ldif*  
![](img/a1.1_ldalp_ou5.png)  

Usamos la misma plantilla del archivo *users.ldif* para crear el resto de usuarios    
![](img/a1.1_ldalp_ou6.png)  
![](img/a1.1_ldalp_ou7.png)  
![](img/a1.1_ldalp_ou8.png)  

Comprobamos con el comando ldapsearch que todos los usuarios han sido creados correctamente sacando un listado.    
![](img/a1.1_ldalp_ou9.png)  

Ahora vamos a configurar LDAP en el cliente  
Instalamos libnss-ldap libpan-ldap ldap-utils -y a través de la consola de comandos  
![](img/a1.1_ldap1.png)
Añadimos la dirección de nuestro servidor ldap   
![](img/a1.1_ldap2.png)  
Añadimos el nombre DNS de nuestro servidor LDAP    
![](img/a1.1_ldap3.png)  
Marcamos las configuraciones por defecto  
![](img/a1.1_ldap4.png)
![](img/a1.1_ldap5.png)
![](img/a1.1_ldap6.png)

Configuramos los archivos /etc/nsswitch.conf para que el sistema sepa que debe buscar información de los usuarios y grupos generados en LDAP.  
![](img/a1.1_ldap6.png)

Configuramos el archivo /etc/pam.d/common-session para que se cree automáticamente el directorio de usuarios LDAP la primera vez que se autentifiquen.   
![](img/pam_session.png)  
Configuramos el archivo /etc/pam.d/common-auth que es donde se indica de que forma el sistema va a verificar la identidad de los usuarios que intentan acceder al sistema.  
![](img/pam_common_auth.png)

Configuramos el archivo /etc/ldap/ldap.conf que básicamente es el archivo de configuración de ldap para linux que permite que el cliente se conecte al servidor.  
![](img/a1.1_final2.png)

Comprobamos como al ejecutar ldapsearch desde el cliente devuelve los usuarios ldap.  
![](img/a1.1_final.png)

Comprobamos que todo funciona conectándonos a nuestra máquina cliente con un usuario de ldap que en este caso sería lcm.  
![](img/final_f.png)

