---
title: "Jeeves HTB WriteUp"
layout: post
---
![Jeeves HTB](/assets/images/Jeeves-1.png)


<h2>Presentación</h2>
Jeeves es una máquina windows que corre un servicio de jenkin que es un servidor para automatizar tareas open sources, la primera vez que hice la máquina daba con 
un archivo keepass que contenía claves, pero esta vez no fue necesaria para escalar privilegios, abusamos del privilegio SeImpersonatePrivilege para lograr convertirnos en administrador, y jugamos con JuicyPotato para explotar el mismo, y para ver las flag de root cuenta con un ADS (Alternate Data Stream) que esta es una característica propia de sistema de ficheros NTFS permite incluir metainformación en un fichero. [ADS-Alternate Data Stream][ADS-Alternate Data Stream]

<h2>Reconocimiento</h2>
En la fase de reconocimiento como siempre realizamos la identificación de puertos y servicios en dos pasos, en la primera parte exportando el output en formato grep a un archivo llamado allports:
  
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
nmap -p80,135,445,50000 -sCV -A -oN allservices 10.10.10.100
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

Aquí podemos identificar dos servicios http en diferente puertos, puerto 80 y 50000 respectivamente, web server de Microsoft IIS y Jetty 9.4.x un Java Web Server; es bueno identificar estas versiones para luego buscar en exploit-DB o seachsploit vulnerabilidades asociadas a las mismas, que quizás para nuestra version enumerada solo aparece un _Information Disclosure_ en un archivo *txt*.

Podemos observar el OS que detecta nmap _windows_server_2008 r2_ que podemos comprobar una vez ganemos accesos con el comando _system info_.

<h2>Enum SMB puerto 445</h2>

Si intentamos conectarnos al servicio de SMB haciendo uso de smbclient a través de un _null session_ para nos reporta _session setup failed: NT_STATUS_ACCESS_DENIED_; por lo que necesitaríamos credenciales para listar y enumerar dicho servicio.

{% highlight bash %}
smbclient -L //10.10.10.63 -N 
{% endhighlight %}

<h2> Puertos 80 y 50000 </h2>

Bien el servidor web en la  http://10.10.10.63/ nos muestra un buscador como la siguiente imagen y que sin importar que introducimos nos envia a _/error.html_,
que no es más que una imagen de error ASP.NET !ay ASP.NET framework, por allí inicie viendo video de [Gavilanch2][Gavilanch2] en youtube!; ok seguimos..

![Jeeves HTB](/assets/images/Jeeves-2.png)

Mencionar que aplicar fuzzing a directorio tampoco arroja nada a diferencia del servicio en el puerto 50000.

![Jeeves HTB](/assets/images/Jeeves-3.png)

Ya hablamos sobre Jetty 9.4.x y posibles vulnerabilidades asociadas; y el fuzzing arroja:

{% highlight bash %}
gobuster dir -u http://10.10.10.63:50000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html
{% endhighlight %}

Y así se ve nuestro Jenking!

![Jeeves HTB](/assets/images/Jeeves-4.png)

Como mencioné en la introdución automatiza la creación y prueba de software, y de ser asi permite la ejecución de comandos, como podemos ver en la siguiente imagen, permite ejecutar windows batch, shell, groovy entre otros, si investigamos un poco más. 

En una consola groovy podemos ejecutar una reverse shell, mientras estamos en escucha, podemos ejecutar el siguiente script de groovy, [Groovy-ReverseShell][Groovy-ReverseShell], como se ve en la siguiente imagen. 

![Jeeves HTB](/assets/images/Jeeves-5.png)

Ejecutamos y aquí esta nuestra shell;

![Jeeves HTB](/assets/images/Jeeves-6.png)

<h2>Escalando Privilegios</h2>
  
![Jeeves HTB](/assets/images/Jeeves-7.png)
 
Lo que suelo hacer antes de escalar privilegios o para iniciar, es moverme a un directorio con capacidad de escritura, cualquiera del listado de applocker bypass windows, en este caso _C:\Windows\Temp_ crear un directorio priv para comprobar capacidad de escritura y traemos el [Juicy-Potato][Juicy-Potato].
 
En esta ocasión tambien agregue el nc.exe para enviarme una consola a otro puerto ya que el priv de SeImpersonatePriv ejecuta esta tarea como el administrador como se puede ver en la siguiente imagen:

![Jeeves HTB](/assets/images/Jeeves-8.png)

recibimos la conexión mientras estamos en escucha con el comando:
{% highlight bash %}
rlwrap nc -nlvp 4444
{% endhighlight %}

<h2>Root Flag --> Alternate Data Stream</h2>

En la siguiente imagen observamos que al momento de intentar ver la flag, nos dice _look deeper_ :) 

![Jeeves HTB](/assets/images/Jeeves-9.png)

Para listar dicha flag luego de listar deeper con el comando:

{% highlight bash %}
dir /R
{% endhighlight %}

la data en stream se puede leer si la rediregimos al comando more <

{% highlight bash %}
more < hm.txt:root.txt:$DATA
{% endhighlight %}

!As simple as that!

🖱️_by:_ *@DaVinciRoot*

[Gavilanch2]: [https://www.hackingarticles.in/credential-dumping-group-policy-preferences-gpp/](https://www.youtube.com/watch?v=YzC-FYg66xA&list=PL0kIvpOlieSNWR3YPSjh9P2p43SFnNBlB)
[ADS-Alternate Data Stream]: [https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0](https://www.securityartwork.es/2015/02/23/alternate-data-stream-ads-flujo-de-datos-alternativos-en-ntfs/)
[Groovy-ReverseShell]: [https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76)
[Juicy-Potato]: [https://github.com/ohpe/juicy-potato)
