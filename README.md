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

Reemplazamos `#IP_DEL_SERVIDOR` por la IP del servidor VPS y `#USUARIO` por el usuario que utilizaremos para acceder al servidor.

Para acceder al servidor, ejecutamos el siguiente comando en la terminal de nuestro equipo local:
```bash
ssh vps
```
La primera vez que nos conectemos al servidor, se nos pedirá que confirmemos la conexión escribiendo `yes` y presionando la tecla `Enter`. Luego, se nos pedirá la contraseña de la llave SSH que asignamos al momento de generarla. Una vez ingresada la contraseña, se nos dará acceso al servidor.

