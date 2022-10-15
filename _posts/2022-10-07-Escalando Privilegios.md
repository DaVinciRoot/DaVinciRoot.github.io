---
title: "Love HTB Escalando Privilegio"
layout: post
---
![Love HTB](/assets/images/love.png)

<h2>Presentación</h2>
En esta ocasión estaré presentando otra escalada de privilegio en windows y es abusando de: AlwaysInstallElevated, me habia tomado tiempo la preparacion de algunos writeUp, ya que estuve preparandome para el eJPT el cual logramos obtener, tesis, mudanza y otras movidas como seguir con networking y firewall desde EVE-NG, pronto..  🤯.

<h2>WinPEAS</h2>
En la fase de busqueda de privilegias, WinPeas nos presenta el siguiente setting:

![Love HTB Escalando Privilegio ](/assets/images/love-1.png)

Este key registry significa que cualquier usuario puede instalar un archivo .msi con privilegios de NT Authority\System.

<h2>Enumeración Manual</h2>

Puedes hacer la solicitud al registro de manera manual con los siguientes comandos:

> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

el regultado de ambos query debe devolver el valor 1, es decir (value is 0x1).



<h2>Initctl</h2>




<h2> ¿Qué hago ahora? </h2>
 




<h2>PoC</h2>





🖱️_by:_ *@DaVinciRoot*
