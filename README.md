# OracleXEUbuntu

# download link for linux:
#  https://www.oracle.com/nl/database/technologies/xe-prior-releases.html

then: 

 1.enter commands:

# unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip 

2.-Instalar los paquetes necesarios con el comando:
# sudo apt-get install alien libaio1 unixodbc

3.-Convertir el paquete de formato RPM alformato DEB (Este formato es el usado por ubuntu) usando el comando:
# sudo alien --scripts -d oracle-xe-11.2.0-1.0.x86_64.rpm

4.-Crear el script chkconfig requerido usando el comando:
# sudo pico /sbin/chkconfig

El editor de texto pico se inició y los comandos se muestran en pantalla. Ahora, copiamos y pegamos lo siguiente dentro del archivo y guardamos:

#!/bin/bash
# Oracle 11gR2 XE installer chkconfig hack for Ubuntu
file=/etc/init.d/oracle-xe
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: OracleXE' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 11g Express Edition' >> $file
echo '### END INIT INFO' >> $file
fi
# update-rc.d oracle-xe defaults 80 01

5.-Cambiamos los permisos del archivo chkconfig usando el comando:
# sudo chmod 755 /sbin/chkconfig  

6.- Establecemos los parametros del kernel. Oracle 11gR2 XE requiere de parametros adicionales del kernel los cuales necesitamos usando el comando:
# sudo pico /etc/sysctl.d/60-oracle.conf

Copiamos lo siguiente dentro del archivo y guardamos:
# Oracle 11g XE kernel parameters  
fs.file-max=6815744  
net.ipv4.ip_local_port_range=9000 65000  
kernel.sem=250 32000 100 128 
kernel.shmmax=536870912 

Verificamos el cambio usando el comando:
# sudo cat /etc/sysctl.d/60-oracle.conf 

Deberíamos ver lo que escribimos anteriormente. Ahora cargamos los parámetros del núcleo:
# sudo service procps start

Verificamos que los parametros se cargaron mediante:
# sudo sysctl -q fs.file-max

Deberíamos poder observar el valor maximo del archivo que introducimos anteriomente.

7.-Configuramos un punto de montaje /dev/shm para Oracle. Creamos el siguiente archivo usando el comando:
# sudo pico /etc/rc2.d/S01shm_load

Copiamos lo siguiente dentro del archivo y guardamos:

#!/bin/sh
case "$1" in
start) mkdir /var/lock/subsys 2>/dev/null
       touch /var/lock/subsys/listener
       rm /dev/shm 2>/dev/null
       mkdir /dev/shm 2>/dev/null
       mount -t tmpfs shmfs -o size=2048m /dev/shm ;;
*) echo error
   exit 1 ;;
esac 

Cambiamos los permisos del archivo usando el comando:
# sudo chmod 755 /etc/rc2.d/S01shm_load

*REINICIAMOS NUESTRA MAQUINA VIRTUAL*

---AHORA SI LA INSTALACIÓN DEL SISTEMA GESTOR---


1.-Instalamos el sistema gestor de oracle usando el comando:
# sudo dpkg --install oracle-xe_11.2.0-2_amd64.deb

2.-Configuramos Oracle usando el comando:
# sudo /etc/init.d/oracle-xe configure 

Introducimos la siguiente informacion:
Un puerto HTTP valido para la aplicacion Oracle Application Express (por default es: 8080).
UN puerto valido para el escuchador de la base de datos de Oracle (por default es: 1521).
Una contraseña para el administrador de cuentas de usuario de SYS y SYSTEM. 
Confirmar las contraseñas.
Seleccionar si deseamos que  el sistema gestor se inicie cuando inicie el sistema.

3.-Configuramos las variables de entorno editando nuestro archiv .bashrc :
# pico ~/.bashrc

Agregamos las siguientes lineas hasta el final del archivo:
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe
export ORACLE_SID=XE
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/u01/app/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH

Ejecutamos los cambios ejecutando nuestro perfil:
# . ~/.profile

4.-Iniciamos Oracle 11gR2 XE:
# sudo service oracle-xe start

5.-Agregamos un usuario YOURUSERNAME al grupo dba usando el comando:

# sudo usermod -a -G dba YOURUSERNAME

--- USO DEL COMAND SHELL DE ORACLE---

1.-Iniciamos los servicios de Oracle XE 11gR2 server usando el comando:
# sudo service oracle-xe start

2.-Iniciamos la linea de comando como el administrador del sistema:
# sqlplus sys as sysdba

Ingresamos la contraseña que seleccionamos durante la instalacion.

3.-Creamos una cuenta de usuario regular usando el comando SQL:
# create user USERNAME identified by PASSWORD;

Reemplazamos USERNAME y PASSWORD con el usuario y contraseña que hayamos elejido. Es importante recordar el usuario y contraseña. Si tuvimos error al ejecutar el comando, entonces ejecutamos el siguiente comando SQL y reintentamos:
alter database open resetlogs

4.-Asignar privilegios a la cuenta de usuario usando el comando SQL:
# grant connect, resource to USERNAME;
Reemplazamos USERNAME y PASSWORD con el usuario y contraseña que hayamos elejido. Es importante recordar el usuario y contraseña.

5.-Salimos con el siquiente comando SQL:
# exit;

6.-Iniciamos la linea de comando como un usuario regular usando el comando:
# sqlplus




# unistall oracle-xe
***************************************unistall oracle-xe*************************************

# /etc/init.d/oracle-xe stop
# dpkg --purge oracle-xe
# rm -r /u01/app
# rm /etc/default/oracle-xe

# update-rc.d -f oracle-xe remove
# update-rc.d -f oracle-mount remove
# update-rc.d -f oracle-shm remove
