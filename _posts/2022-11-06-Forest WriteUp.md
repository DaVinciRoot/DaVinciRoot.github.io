---
title: "Forest HTB WriteUp"
layout: post
---
![Forest HTB](/assets/images/forest-0.png)


<h2>Presentación</h2>
Forest es una máquina windows que figura como Domain Controller, estaremos realizando enumeración de Directorio Activo, servicios como RPC, SMB, DNS, ataques a kerberos que ya hemos tocado en una que otras ocasiones, además  hacemos uso de unas de mis herramientas de pentesting favoritas [BloodHoundAD][BloodHoundAD] para la enumeración de entorno de Directorio Activos, excelente para Red&Blue Teamers, ruidosa por los _logs_ que genera, eso si!, abusamos del Account Operator Group y finalmente realizamos un ataque de DCSync al momento de escalar privilegios, haciendo uso de secretsdump.py del poderoso impacket.  

<h2>Reconocimiento</h2>
En la fase de reconocimiento para la identificación de puertos y servicios en dos pasos, hacemos uso de la herramienta de RustScan que nos lista los puertos de manera rápida y que también podemos exportar el output en formato grep a un archivo llamado allports:
  
{% highlight bash %}
rustscan -a 10.10.10.161
{% endhighlight %}

![Forest HTB](/assets/images/forest-1.png)

Ya luego realizamos un escaneo de todos los puertos _-p-_, _-v_ para que nos represente el output por pantalla a medida que descubre puertos abiertos, _-n_ para que no aplique resolución DNS y _-Pn_ para que asuma que todos los puertos estan arriba (up). 

```
[*] IP Address: 10.10.10.63
[*] Open ports: 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,4701,49666,49671,49664,49665,49667,49677,49684,49703,49958
```

<h2>Puertos y Servicios</h2>
Con el fin de conocer un poco más de los servicios que se están ejecutando y lanzar una serie de script básico de reconocimiento para dichos puertos y servicios. 

{% highlight bash %}
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,4701,49666,49671,49664,49665,49667,49677,49684,49703,49958 -sCV -A -oN allservices 10.10.10.161
{% endhighlight %}

```
Nmap scan report for 10.10.10.161
Host is up (0.030s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain?      Simple DNS Plus
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec Microsoft Windows Kerberos 
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=10/14%Time=5DA4BD82%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version
SF:\x04bind\0\0\x10\0\x03");
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h27m32s, deviation: 4h02m30s, median: 7m31s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 281.19 seconds

```
<h2>Enum DNS puerto 53</h2>
Haciendo uso de la herramienta _dig_ podemos enumerar el servicio DNS, como son: name server, mail server e intento de transferencia de zona, y poder identificar v-host en uso para agregarlo al _/etc/hosts_ pero no hay nada que agregar mas que el nombre y dominio de la maquina _forest.htb.local_.

![Forest HTB](/assets/images/forest-2.png)

<h2>Enum SMB puerto 445</h2>
Crackmapexec nos identifica el hostname de la maquina, la posible version de windows en uso, en este caso Windows Server 2016 Standard x64, el dominio y certificado smb firmados, que impiden ataques como el SMB relay haciendo uso del Responder, pero que presentaré muy pronto 📝. 

Si intentamos conectarnos al servicio de SMB haciendo uso de smbmap a través de un _null session_ para nos reporta _Authentication error_; por lo que necesitaríamos credenciales para listar y enumerar dicho servicio.

{% highlight bash %}
smbmap -H 10.10.10.161 -u "null"
{% endhighlight %}

_smbclient_ logra un Anonymous login pero no lista ningun share o workgroup available.

{% highlight bash %}
smbclient -L //10.10.10.161 -N 
{% endhighlight %}

<h2> Enum RPC </h2>

Haciendo uso de la herramienta _rpcclient_ a través de un null session probamos enumerar usuarios del dominio, para luego proceder a validarlo con kerburte. 

![Forest HTB](/assets/images/forest-4.png)

podemos crear un users list para aplicar un ataque de kerberos un ataque de tipo AS-REP Roasting.

![Forest HTB](/assets/images/forest-5.png)

<h2>Kerbrute userenum</h2>
  
Con la herramienta kerbrute podemos validar usuarios mediante kerberos, y nos aplica el AS-REP Roasting solicitando un TGTs (Ticket Granting Ticket) para todos los usuarios.

![Forest HTB](/assets/images/forest-6.png)

Obtenemos 19 usuarios válidos del dominio, de los 32 listado a través de rpcclient.

<h2>AS-REP Roasting</h2>

Este ataque lo realizamos en la máquina [Active][Active], en el que se identifica usuarios que tienen la casilla de “Do not require Kerberos preauthentication” habilitada.

![Forest HTB](/assets/images/forest-7.png)

identificando al usuario svc-alfresco como un AS-REP Roastuser y crackeando el hash con john, obtenemos una contraseña. Mencionar que haciendo uso de GetNPUsers nos da un hash tipo: _7500	Kerberos 5, etype 23, AS-REQ Pre-Auth_ según hash wiki a diferencia de kerbrute nos da _7500	Kerberos 5, etype 23, AS-REQ Pre-Auth_ que no me fue posible crackear. 

![Forest HTB](/assets/images/forest-8.png)

<h2>Validación de usuario mediante SMB y WINRM</h2>

![Forest HTB](/assets/images/forest-9.png)

Nos logueamos a través de WINRM usando evil-winrm, ya que nos marca _Pwn3d_

<h2>Whoami</h2>

![Forest HTB](/assets/images/forest-10.png)

En la fase de reconocimiento previo a la escalar privilegios, vemos que nuestro usuario forma parte del grupo _account operators_: [Account Operators][Account Operators]

{% highlight bash %}
The Account Operators group grants limited account creation privileges to a user. Members of this group can create and modify most types of accounts, including accounts for users, Local groups, and Global groups. Group members can log in locally to domain controllers. 
{% endhighlight %}

Si contamos con este privilegios podemos como bien expresa crear usuarios y agregarlo al grupo que deseemos, y aqui es donde entra BloodHound para identificar el path para escalar privilegios. 

<h2>BloodHound</h2>
Para resumir el WriteUp, dejo el enlace de configuración de BloodHound con [John Hammond][John Hammond] y nos trasladamos al momento de subir el archivo _.zip_.

Lo primero al subir la información recolectada es marcar nuestro usuarios que han sido comprometidos.

![Forest HTB](/assets/images/forest-11.png)

si damos click en Análisis podemos listar información, como usuario kerberoastables...

![Forest HTB](/assets/images/forest-12.png)

<h2>Identificando el path para escalar privilegios en BloodHound</h2>

BloodHound utiliza gráficos y se auxilia de un colector llamado SharpHound.ps1 que se encuentra en el mismo directorio. A nuestro fines damo click "Analysis", hago clic en "Find Shorter Paths to Domain Admin", ruta más corta para convertinos en Administradores del Dominio y obtengo el siguiente gráfico:

![Forest HTB](/assets/images/forest-13.png)

1- Nuestro usuario _svc-alfresco_ es miembro del grupo _Service Accounts_ que a su vez es miembro de _Privileged it accounts_ y miembros de _Account Operator_ que ya habiamos indentificado con el comando _whoami /all_

2- Como miembro de _Account Operators_ y teneiendo _Generic All_, sobre el grupo de _Exchange Windows permissions_, podemos crear un usuario y agregar el mismo a dicho grupo, el cual tiene permisos de _WriteDacl_ sobre el dominio _htb.local_.

![Forest HTB](/assets/images/forest-14.png)
![Forest HTB](/assets/images/forest-15.png)

Ya formando parte del grupo  _Exchange Windows permissions_ y teniendo el permiso de _WriteDacl_, podemos ver la info en el mismo BloodHound y el Abuse info,q ue nos explica como abusar de este privilegio, como no amarlo!? 💘

![Forest HTB](/assets/images/forest-16.png)
![Forest HTB](/assets/images/forest-17.png)

Mencionar que los comandos que presentan el abuse info, forman parte de [PowerView.ps1][PowerView.ps1] de PowerSploit in github. 

Pues cargamos nuestra credenciales de usuarios y el PowerView.ps1 en memoria, para proceder a ejecutar el comando de abuse info, además de agregar el _-TargetIdentity_ en formato CN de ldap 'DC=htb,DC=local' y el _-PrincipalIdentity_ que es el usuario que he creado en este caso _loduynht_, en lugar de como ejemplifica el Abuse Info, como vemos en la siguiente secuencia de comandos.

3-

![Forest HTB](/assets/images/forest-18.png)

Una vez otorgado dicho privilegio _Rigts DCSync_

Con secretsdump:
impacket-secretsdump htb.local\loduynht@10.10.10.161

![Forest HTB](/assets/images/forest-19.png)

Puedes ejecutar un Pass The Hash attack y utilizar psexec.py, aqui también tienes el HASH NTLM del usuario krbtgt para un Golden Ticket Attack.

🖱️_by:_ *@DaVinciRoot*


[BloodHoundAD]: [https://github.com/BloodHoundAD)
[Active]: [https://davinciroot.github.io/Active-HTB/)
[Account Operators]: [https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#account-operators)
[John Hammond]: [https://www.youtube.com/watch?v=yp8fw72oQvY)
[PowerView.ps1]: [https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
