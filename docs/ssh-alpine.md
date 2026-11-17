# Guía rápida: Activar y configurar SSH en Alpine Linux

Esta guía detalla los pasos necesarios para instalar, configurar y habilitar el servicio de OpenSSH en Alpine Linux para permitir el acceso mediante comandos y la automatización con Ansible.

## 1. Instalar OpenSSH
Actualiza los repositorios de paquetes e instala el servidor OpenSSH:

```bash
apk update && apk add openssh
```

## 2. Configurar el acceso para Root

Por defecto, Alpine puede bloquear el acceso remoto directo para el usuario `root`. Si vas a gestionarlo mediante usuario **root**, asegúrate de permitirlo:

1. Abre el archivo de configuración con un editor (como `vi` o `nano`):
```bash
vi /etc/ssh/sshd_config
```


2. Busca la línea `#PermitRootLogin ...` o `PermitRootLogin prohibit-password`, descoméntala (elimina el `#` del inicio) y cámbiala a:
```text
PermitRootLogin yes
```


3. Guarda y cierra el archivo.

*Verificación:* El archivo contiene la directiva `PermitRootLogin yes` activa.

## 3. Habilitar e iniciar el servicio

Alpine Linux utiliza `OpenRC` como sistema de inicio. Para asegurarte de que el servicio SSH arranque automáticamente al encender el contenedor o máquina, ejecuta:

```bash
rc-update add sshd
rc-service sshd start
```

*Verificación:* Ejecuta `rc-service sshd status` y confirma que el estado devuelto sea `started`.

## 4. Verificar la conexión

Desde tu máquina de control o terminal de Ansible, comprueba que puedes alcanzar el puerto 22 y autenticarte mediante tu clave pública o contraseña:

```bash
ssh root@[IP_DEL_CONTENEDOR]
```

*Verificación:* La terminal establece la sesión interactiva exitosamente con el sistema remoto.
