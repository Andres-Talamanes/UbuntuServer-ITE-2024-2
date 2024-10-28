## Indice

1. [UserDir con Apache](#userdir-con-apache)
2. [Permisos de escritura a archivos y directorios](#permisos-de-escritura-a-archivos-y-directorios)
3. [Limitar el almacenamiento con edquota](#limitar-el-almacenamiento-con-edquota)
4. [Crear tuneles utilizando Ngrok en Ubuntu server](#crear-tuneles-utilizando-ngrok-en-ubuntu-server)
5. [Crear tuneles utilizando CloudFlare en Ubuntu server](#crear-tuneles-utilizando-cloudflare-en-ubuntu-server)
6. [Cargar o descargar archivos del servidor](#cargar-o-descargar-archivos-del-servidor)


# UserDir con Apache
Para que cada usuario en el servidor pueda alojar la carpeta public_html a trabes de la URL, se debe habilitar y configurar el modulo `userdir` de apache

## 1. Instalar Apache
```
sudo apt update
sudo apt install apache2
```

## 2. Habilitar el modulo `userdir`

El modulo `userdir` de Apache permite acceder a los archivos almacenados en una capreta especifica dentro del directorio home de cada usuario. Lo habilitaremos ejecutando:

```
sudo a2enmod userdir
```
## 3. Crear las carpetas public_html

Cada usuario debera tener la carpeta `public_html` dentro de su directorio home dandole el permiso `755` que asegura que otros usuarios y el servidor Apache pueda leer el los archivos.

```
mkdir /home/usuario/public_html
chmod 755 /home/usuario/public_html/*
```
Cambia usuario por el nombre del usuario correspondiente. 

## 4. Configurar el uso de `userdir`

Por defecto la configuracion de `userdir` debe estar habilitada correctamente, verificamos ingresando al archivo de configuracion:
```
sudo nano /etc/apache2/mods-enabled/userdir.conf
```

Nos aseguramos de que la configuracion permita el acceso adecuado:

```
<IfModule mod_userdir.c>
    UserDir public_html
    <Directory /home/*/public_html>
        AllowOverride FileInfo AuthConfig Limit Indexes
        Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
        Require all granted
    </Directory>
</IfModule>
```
## 5. Reiniciamos Apache

Para aplicar las configuraciones es necesario reiniciar Apache
```
sudo systemctl restart apache2
```

## 6. Probamos el acceso

Si todo está configurado correctamente, ahora deberías poder acceder a los archivos en la carpeta `public_html` del usuario mediante la URL:
```
http://<tu-ip>/~usuario
```
Esto mostrará el contenido de la carpeta `public_html` del usuario especificado.

**Nota:** Asegúrate de que el firewall de tu servidor esté configurado para permitir el tráfico HTTP (puerto 80) y HTTPS (si vas a usar SSL, puerto 443).

---
# Permisos de escritura a archivos y directorios

## Permisos de escritura a un archivo
Para quitar permisos de escritura a un archivo escribimos:
```
sudo chmod a-w /home/usuario/nombre-del-archivo
```
cambiamos `usuario` por el nombre del usuario y `nombre-del-archivo` por el nombre del archivo al que le queremos quitar estos permisos

Para dar permisos de escritura a un archivo escribimos:
```
sudo chmod 755 nombre_del_archivo
```

## Permisos de escritura a un directorio
Para quitar permisos de escritura a un directorio escribimos:
```
sudo chmod a-w /home/usuario/nombre-del-directorio
```
cambiamos `usuario` por el nombre del usuario y `nombre-del-directorio` por el nombre del directorio al que le queremos quitar estos permisos

Para dar permisos de escritura a un directorio escribimos:
```
sudo chmod 755 nombre_del_directorio
```

**Nota:** 755 Permite al propietario leer, escribir y ejecutar; los demás pueden solo leer y ejecutar.

---
# Limitar el almacenamiento con edquota

## 1. Instalar edquota

```
sudo apt update && sudo apt install quota
```

## 2. Habilitar cuotas en el sistema de archivos
1. Debemos habilitar las cuotas en el sistema de archivos donde deseamos aplicar las restricciones. Editamos el archivo `/etc/fstab` y añadimos las opciones `usrquota` y/o `grpquota` a la partición correspondiente (por ejemplo, en `/home` si quieremos limitar el almacenamiento en los directorios de usuario).
```
sudo nano /etc/fstab
```

2. Nos aseguramos que la linea de particion tenga algo como esto:
```
/dev/sda1   /home   ext4    defaults,usrquota,grpquota   0   2
```

3. Montar la partición con las nuevas opciones: Después de editar el archivo `/etc/fstab`, remonta la partición:
```
sudo mount -o remount /home
```

## 3. Creacion de los archivos de cuota
Ahora crearemos los archivos necesarios para las cuotas
```
sudo quotacheck -cum /home
sudo quotaon /home
```

## 4. Editar las cuotas para los usuarios

1. Para establecer las cuotas de almacenamiento de un usuario, usa el comando `edquota` seguido del nombre de usuario
```
sudo edquota usuario
```
Esto abrirá un archivo donde puedes especificar los límites. Verás algo como esto:
```
Disk quotas for user usuario (uid 1001):
  Filesystem  blocks   soft   hard    inodes  soft  hard
  /dev/sda1   0        0      0       0       0     0
```
- **Soft limit:** el límite que el usuario puede exceder temporalmente.
- **Hard limit:** el límite absoluto que el usuario no puede exceder.

En este caso nosotros estableseremos un limite de 2 GB para cada usuario

```
/dev/sda1   0        2000000   2048000  0       0     0
```

2. Ver las cuotas asignadas

Para verificar las cuotas de un usuario escribimos:
```
sudo quota usuario
```

3. Aplicar cuotas a varios usuarios.
Para aplicar la misma cuota para varios usuarios podemos usar `edcuota -p` para copiar las cuotas de un usuario a otros:
```
sudo edquota -p usuario_origen usuario1 usuario2 usuario3
```

---
# Crear tuneles utilizando Ngrok en Ubuntu server
## 1. Instalar Ngrok
1. Descarga Ngrok para linux desde el sitio oficial
```bash
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
```

2. Descomprime el archivo descargado:
```bash
unzip ngrok-stable-linux-amd64.zip
```
3. Mueve el archivo ngrok a /usr/local/bin para que esté disponible globalmente:
``` bash
sudo mv ngrok /usr/local/bin/
```
4. Asegúrate de que está instalado correctamente verificando la versión:
```
ngrok --version
```
## 2. Crear una cuenta en Ngrok
1. Si no tienes una cuenta de Ngrok, regístrate en [Ngrok](https://ngrok.com/).
2. Configura ngrok con tu token (esto es necesario para usar ngrok en su versión gratuita):
```bash
ngrok config add-authtoken YOUR_AUTHTOKEN
```
Reemplaza YOUR_AUTHTOKEN con el token proporcionado en tu cuenta de ngrok.
## 3 Crer el tunel
Para exponer un puerto de tu servidor local, puedes usar el siguiente comando. Por ejemplo, si tienes un servidor en el puerto 80 (HTTP):
```bash
ngrok http 80
```
Este comando iniciará el túnel y automáticamente generará una URL pública temporal bajo un subdominio de Ngrok, algo como:
```
http://xxxxxxxx.ngrok.io
```
## Mantener el tunel activo
Mientras el túnel esté activo, podrás acceder a tu servicio local a través de la URL generada. Una vez que cierres el terminal o detengas el proceso, la URL dejará de funcionar.

**Nota:** La URL generada automáticamente es temporal y se eliminará cuando termines la sesión de cloudflared. Si deseas un acceso más permanente o personalizado, en ese caso necesitarías un dominio propio o configuraciones adicionales.

Este método es ideal para pruebas rápidas, demostraciones o situaciones donde no deseas configurar un dominio completo.

---
# Crear tuneles utilizando CloudFlare en Ubuntu server
## 1. Crear una cuenta en Cloudflare
Si no tienes una cuenta de Cloudflare, regístrate en [Cloudflare](https://www.cloudflare.com/).
## 2. Instalar CloudFlare
1. Actualiza los paquetes en tu sistema:
```bash
sudo apt update && sudo apt upgrade 
```
2. Añade el repositorio a la lista de fuentes:
```bash
echo "deb http://pkg.cloudflare.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare.list
```
3. Instala el paquete:
```
sudo apt update && sudo apt install cloudflared
```
4. Verifica la instalación:
```
cloudflared --version
```


Si el sistema no puede localizar el paquete CloudFlare puede deverse a algun problema de la version de ubuntu o que el repositorio no se agrego correctamente. Si se experimento este error instalaremos manualmente: 

1. Descarga el binario de cloudflared:
```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```
2. Instala el paquete manualmente:
```
sudo dpkg -i cloudflared-linux-amd64.deb
```
3. Verifica la instalación:
```
cloudflared --version
```
## 3 Crer el tunel
1. Ejecuta el siguiente comando para iniciar sesión en tu cuenta de Cloudflare:
```bash
cloudflared login
```
Esto abrirá tu navegador y te pedirá que inicies sesión en Cloudflare.

2. Supongamos que tienes una aplicación corriendo en el puerto 8000 en tu máquina local. Usa el siguiente comando para crear el túnel sin dominio:
```bash
cloudflared tunnel --url http://localhost:8000
```
3. Si el paso anterior se hace de manera remota, usaremos el mismo comando sustituyebdo 'localhost' por la direccion ip del host.
```bash
cloudflared tunnel --url http://localhost:8000
```
Este comando iniciará el túnel y automáticamente generará una URL pública temporal bajo un subdominio de Cloudflare, algo como:

``` 
https://nombre-generado-al-azar.trycloudflare.com 
```
## Mantener el tunel activo
Mientras el túnel esté activo, podrás acceder a tu servicio local a través de la URL generada. Una vez que cierres el terminal o detengas el proceso, la URL dejará de funcionar.

**Nota:** La URL generada automáticamente es temporal y se eliminará cuando termines la sesión de cloudflared. Si deseas un acceso más permanente o personalizado, en ese caso necesitarías un dominio propio o configuraciones adicionales.

Este método es ideal para pruebas rápidas, demostraciones o situaciones donde no deseas configurar un dominio completo.

---

# Cargar o descargar archivos del servidor

## Con STFP

1. Conectarse al servidor con sftp:

```
sftp usuario@servidor_remoto
```

Sustituyendo `usuario` por el nombre del usuario y `servidor-remoto` con la direccion ip del servidor

2. Subir un archivo local al servidor:

```
put /ruta/del/archivo/archivo_local /ruta/donde/se/aloja/archivo_remoto
```

3. Descargar un archivo del servidor:

```
get /ruta/del/archivo/archivo_remoto /ruta/donde/se/aloja/archivo
```

## Con SCP

1. Subir:

```
scp archivo_local usuario@servidor_remoto:/ruta/destino
```

Para subir una carpeta poner `-r` entre spc y archivo_local

```
scp -r archivo_local usuario@servidor_remoto:/ruta/destino
```

2. Descargar: 

```
scp usuarui@servidor_remoto:/ruta/del/archivo archivo_local
```
Para descargar una carpeta poner `-r` entre spc y usuario

```
scp usuario@servidor_remoto:/ruta/del/archivo archivo_local
```

Sustituyendo `usuario` por el nombre del usuario y `servidor-remoto` con la direccion ip del servidor


---
# Bloque de comandos
En este apartado bloquearemos ciertos comandos en los cuales los usuarios no podran utilizarlos los cuales son:

**wall:** Este comando muestra un mensaje las terminales de todos los usuarios conectados.

**rm:** Este comando funciona para borrar archivos, directorios, etc.

**reboot:** Este comando sirve para reiniciar el sistema que estan usando.

1. Abrir el archivo souders utilizando visudo
```
sudo visudo
```

2. Agregaremos una regla para poder bloquear el primer comando wall.

```
ALL ALL = ALL, !/usr/bin/wall
```

3. Otorgaremos permisos para que solo los usuarios autorizados puedan ejecutarlo.
```
sudo chmod 750 /usr/bin/wall
```

4.Crearemos un grupo especial para los usuarios autorizados.
```
sudo groupadd wallusers
```

5. Asignaremos el grupo binario de wall.
```
sudo chown root:wallusers /usr/bin/wall
```
## Pasos para negar permisos.

1. utilizaremos el comando para eliminar al usuario del grupo wallusers.

**Nota**. Cambiar usuario por la que lo esta utilizando.

```
sudo gpasswd -d usuario wallusers
```

## Comando rm
1. Crearemos el grupo para comando rm

```
sudo chown root:wallusers !/bin/rm
```

2.Asignaremos el grupo binario de wall.
```
sudo chown root:wallusers
```

## Comando Reboot
1. Crearemos el grupo para comando rm

```
sudo chown root:wallusers !/sbin/shutdown 

o 

sudo chown root:wallusers !/sbin/reboot
```

2. Asignaremos el grupo binario de wall.
```
sudo chown root:wallusers
```