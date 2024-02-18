# Configuración de un servidor VPS con dominio propio

## Introducción

Este documento tiene como objetivo guiar a los usuarios a configurar un servidor VPS con un dominio propio. El servidor VPS será configurado con un sistema operativo Debian 11 y al igual que el dominio, será adquirido a través de la plataforma de Hostinger.

## Requisitos

- Cuenta en Hostinger
- Dominio adquirido en Hostinger
- Servidor VPS adquirido en Hostinger
- Conocimientos básicos de Linux

## Configuración del servidor VPS

### Acceso al servidor

Una vez adquirido el servidor VPS, se debe acceder a él a través de SSH. Para ello, debemos de generar una llave SSH en nuestro equipo local y agregarla al servidor. Para generar la llave SSH, ejecutamos el siguiente comando en la terminal de nuestro equipo local:

```bash
ssh-keygen -t rsa -b 4096 -C "email@correo.com" -f ~/.ssh/id_rsa
```
> - `ssh-keygen` es el comando para generar una llave SSH
> - `-t rsa` indica que la llave será de tipo RSA
> - `-b 4096` indica que la llave tendrá 4096 bits
> - `-C "email@correo.com"` es un comentario que se agrega a la llave para identificarla más fácilmente
> - `-f ~/.ssh/id_rsa` indica que la llave se guardará en el directorio `~/.ssh` con el nombre `id_rsa`

El comando anterior generará una llave SSH con el nombre `id_rsa` en el directorio `~/.ssh`. Es importante que reemplacemos `email@correo.com` por nuestro correo electrónico real y que asignemos una contraseña a la llave para mayor seguridad.

Una vez generada la llave SSH, debemos de agregarla al servidor VPS. Para ello, nos vamos a la sección `Claves SSH` en el panel de ajustes del servidor VPS en Hostinger y agregamos la llave pública `~/.ssh/id_rsa.pub` que acabamos de generar.

![Claves SSH](./Img/add-ssh-key.png)

Para mayor comodidad, en nuestro archivo `~/.ssh/config` agregamos la siguiente configuración:

```bash
Host vps
    HostName #IP_DEL_SERVIDOR
    User #USUARIO
    IdentityFile ~/.ssh/id_rsa
```

Reemplazamos `#IP_DEL_SERVIDOR` por la IP del servidor VPS y `#USUARIO` por el usuario que utilizaremos para acceder al servidor, que por defecto es `root`.

Para acceder al servidor, ejecutamos el siguiente comando en la terminal de nuestro equipo local:
```bash
ssh vps
```
La primera vez que nos conectemos al servidor, se nos pedirá que confirmemos la conexión escribiendo `yes` y presionando la tecla `Enter`. Luego, se nos pedirá la contraseña de la llave SSH que asignamos al momento de generarla. Una vez ingresada la contraseña, se nos dará acceso al servidor.

### Configuración inicial

Una vez dentro del servidor, ejecutamos los siguientes comandos para actualizar el sistema.

```bash
sudo apt update
sudo apt upgrade
```
> Al ser una distribución de Linux basada en Debian, utilizamos `apt` para instalar y actualizar paquetes.

Esto actualizará el sistema y nos pedirá confirmación para instalar las actualizaciones. Escribimos `y` y presionamos la tecla `Enter` para confirmar la instalación.

### Seguridad del servidor

La seguridad del servidor es un aspecto fundamental que no debemos de pasar por alto. A continuación, se presentan algunas recomendaciones para mejorar la seguridad del servidor.

#### Deshabilitar el acceso por contraseña

Para mejorar la seguridad del servidor, deshabilitamos el acceso por contraseña y permitimos el acceso únicamente por llaves SSH. Para ello, nos dirigimos al archivo `/etc/ssh/sshd_config` con el siguiente comando:

```bash
sudo nvim /etc/ssh/sshd_config
```

> `nvim` es un editor de texto que utilizaremos para editar el archivo. Si no está instalado, podemos instalarlo ejecutando `sudo apt install neovim`. Si no queremos instalar `nvim`, podemos utilizar `nano` o cualquier otro editor de texto.

Dentro del archivo, buscamos las siguientes líneas:

```bash
PasswordAuthentication yes
challengeResponseAuthentication yes
UsePAM yes
```

Y las reemplazamos por:

```bash
PasswordAuthentication no
challengeResponseAuthentication no
UsePAM no
```

Guardamos los cambios y salimos del editor. Luego, reiniciamos el servicio SHH con el siguiente comando:

```bash
sudo systemctl restart sshd
```

#### Deshabilitar el acceso al usuario root y crear un nuevo usuario

El usuario `root` tiene permisos de administrador en el servidor, por lo que es un objetivo común para los atacantes. Por esta razón, deshabilitamos el acceso al usuario `root` y creamos un nuevo usuario con permisos de administrador.

Para añadir un nuevo usuario, ejecutamos el siguiente comando:

```bash
sudo adduser #NUEVO_USUARIO
```

> Reemplazamos `#NUEVO_USUARIO` por el nombre del nuevo usuario.

Cuando le damos enter, se nos pedirá que asignemos una contraseña al nuevo usuario y que ingresemos información adicional como el nombre completo, número de teléfono, etc. Esta información es opcional, por lo que podemos presionar la tecla `Enter` para dejar los campos vacíos.

Una vez creado el nuevo usuario, le asignamos permisos de administrador con el siguiente comando:

```bash
sudo usermod -aG sudo #NUEVO_USUARIO
```

> Reemplazamos `#NUEVO_USUARIO` por el nombre del nuevo usuario.

Para verificar que el nuevo usuario se ha creado correctamente, ejecutamos el siguiente comando:

```bash
su - #NUEVO_USUARIO
```

> Reemplazamos `#NUEVO_USUARIO` por el nombre del nuevo usuario.

Si el comando se ejecuta correctamente, se nos pedirá la contraseña del nuevo usuario y se nos dará acceso a la cuenta del nuevo usuario.

Ahora vamos a copiar la llave SSH que generamos en nuestro equipo local al nuevo usuario. Para ello, creamos el directorio `~/.ssh` en la cuenta del nuevo usuario con el siguiente comando:

```bash
mkdir ~/.ssh
```

Ingresamos al directorio `~/.ssh` y creamos el archivo `authorized_keys` con el siguiente comando:

```bash
nvim ~/.ssh/authorized_keys
```

Dentro del archivo `authorized_keys`, pegamos la llave pública `~/.ssh/id_rsa.pub` que generamos en nuestro equipo local. Guardamos los cambios, salimos del editor y con un `exit` salimos de la cuenta del nuevo usuario.

Ahora que hemos creado un nuevo usuario y le hemos asignado permisos de administrador, deshabilitamos el acceso al usuario `root`. Ingresamos al archivo `/etc/ssh/sshd_config` con el siguiente comando:

```bash
sudo nvim /etc/ssh/sshd_config
```

Dentro del archivo, buscamos la siguiente línea:

```bash
PermitRootLogin yes
```

Y la reemplazamos por:

```bash
PermitRootLogin no
```

Una vez hecho esto, guardamos los cambios y salimos del editor. Luego, reiniciamos el servicio SHH con el siguiente comando:

```bash
sudo systemctl restart sshd
```

Para verificar que el nuevo usuario tiene acceso al servidor, salimos del servidor con el comando `exit` y volvemos a ingresar al servidor con el nuevo usuario. Para ello necesitamos modificar el archivo `~/.ssh/config` en nuestro equipo local con la siguiente configuración:

```bash
Host vps
    HostName #IP_DEL_SERVIDOR
    User #NUEVO_USUARIO
    IdentityFile ~/.ssh/id_rsa
```

> Reemplazamos `#NUEVO_USUARIO` por el nombre del nuevo usuario.

Ahora, para acceder al servidor, ejecutamos el siguiente comando en la terminal de nuestro equipo local:

```bash
ssh vps
```

Al ejecutar el comando, se nos pedirá la contraseña de la llave SSH que asignamos al momento de generarla. Una vez ingresada la contraseña, se nos dará acceso al servidor con el nuevo usuario.

#### Configuración del cortafuegos

El cortafuegos es una herramienta que nos permite controlar el tráfico de red que entra y sale del servidor. Para configurar el cortafuegos, utilizamos `ufw`, para instalarlo ejecutamos el siguiente comando:

```bash
sudo apt install ufw
```

Una vez instalado, habilitamos las conexiones SSH con el siguiente comando:

```bash
sudo ufw allow OpenSSH
```

Luego, habilitamos las conexiones HTTP y HTTPS con los siguientes comandos:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Finalmente, habilitamos el cortafuegos con el siguiente comando:

```bash
sudo ufw enable
```
> Nos avisa que el cortafuegos puede bloquear nuestra conexión SSH, escribimos `y` y presionamos la tecla `Enter` para confirmar.

Para verificar que el cortafuegos se ha configurado correctamente, ejecutamos el siguiente comando:

```bash
sudo ufw status
```

Si todo se ha configurado correctamente, se nos mostrará un mensaje similar al siguiente:

![UFW Status](./Img/ufw-status.png)

Lo siguiente es configurar el cortafuegos para que deniegue todo el tráfico entrante y permita todo el tráfico saliente. Para ello, ejecutamos los siguientes comandos:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Para finalizar, reiniciamos ufw con el siguiente comando:

```bash
sudo ufw reload
```

Para verificar que todo ha sido configurado correctamente, aun podemos acceder al servidor salimos del servidor con el comando `exit` y volvemos a ingresar al servidor. Si todo se ha configurado correctamente, se nos dará acceso al servidor con el nuevo usuario.



