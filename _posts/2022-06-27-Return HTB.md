---
title: "Return HTB WriteUp"
layout: post
---
![Return HTB](/assets/images/Return 1.png)


<h2>Presentación</h2>
Return forma parte del Hack The Box printer track, junto a máquinas como Driver y Antique. La primera parte de la instrucción es muy sencilla es simplemente redirigir la ip o el (_FQDN_), que se encuentra apuntando a un url interno en el panel web y obtener credenciales de LDAP mientras te mantienes en escuchas por el puerto 389, luego en la fase de reconocimiento para  escalar privilegios vemos posibilidades de hacerlo abusando de SetBackupPrivilege o SetLoadDriverPrivilege, pero eso lo traeremos más adelante, en esta ocasión escalaremos privilegios a través del Server Operator Group, que es un privilegio con lo que cuenta el usuario, del cual obtuvimos credenciales en primer lugar.

<h2>Reconocimiento</h2>

En la fase de reconocimiento realizamos el siguiente network map, exportando el output en formato grep a un archivo llamado allports:
  
{% highlight bash %}
nmap -p- --min-rate 5000 10.10.11.108 -oG allports
{% endhighlight %}

**Output** 
[*] IP Address: 10.10.11.108

{% highlight bash %}
[*] Open ports: 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49679,49682,49697,59724
{% endhighlight %}

<h2>Puertos y Servicios</h2>
Por la cantidad de puertos y servicios involucrados luce comno un Domain Controller, puertos como 53, 88, DNS y Kerberos resepctivamente, servicos y puertos de RPC 135-139, 389 LDAP como mencionamos anteriormente. Ahora procederemos con ejecutar las opciones _sCV_ para reconocimiento básico de scripts para dichos puertos y servicios. 

{% highlight bash %}
nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49679,49682,49697,59724 -sCV 10.10.11.108 -oN targeted
{% endhighlight %}

<h2> Servicio SMB 445 </h2>
Utilizando cramapexec smb y smbclient en la fase de reconocimiento, para saber frente a que estamos:

{% highlight bash %}
SMB    10.10.11.108    445    PRINTER    [*] Windows 10.0 Build 17763 x64 (name:PRINTER) (domain:return.local) (signing:True) (SMBv1:False)
{% endhighlight %}

La máquina se llama Printer, al parecer un Windows 10.0, la cual pertene al dominio _return.local_, que añadiremos a nuestro _/etc/hosts_.

```
smbclient -L 10.10.11.108 -N                                        
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.108 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
```
smbmap -H 10.10.11.108 -u 'null'
[!] Authentication error on 10.10.11.108
```
<h2> Servicio DNS 53 </h2>
```
nslookup 
> server 10.10.11.108
Default server: 10.10.11.108
Address: 10.10.11.108#53
> 10.10.11.108
;; connection timed out; no servers could be reached
```
Tampoco funcionó el ataque de trasferencia de zona a dicho servicio.

<h2>Servicio HTTP puerto 80</h2>

En la sessión de settings dentro del panel web, encontramos:

![Return HTB](/assets/images/Return 80.png)

Que como mencioné anteriormente es simplemente hacer que el server address se dirija a nuestra dirección ip de atacante.

![Return HTB](/assets/images/Return 81.png)

**Nota** Nos ponemos en escuha por el puerto 389, siendo una maquina windows podemos utilizar rlwrap, damos clic en update y recibimos las siguientes credenciales.

![Return HTB](/assets/images/Return 82.png)

Ya con estas credenciales y validando dicho usuarios con crackmapexec, y sabiendo que el puerto de Winrm esta abierto probamos una conexión.

![Return HTB](/assets/images/Return 83.png)

Desde aquí podemos obtener la user.txt flag. 

<h2>Recon-PrivEsc </h2>

En un principio y sabiendo lo que involucra escalar privilegios con SetBackupPrivilege o SetLoadDriverPrivilege, me dije que entonces HTB hubiese etiquetado la maquina como medium al menos 💁‍♂️.

![Return HTB](/assets/images/Return 84.png)

Por lo que volvi a ver e invetigar sobre el grupo que no conocia cuando hice el comando _net user_.

![Return HTB](/assets/images/Return 85.png)

E ivestigando sobre el mismo en la web oficial de Microsoft, encontré el siguiente artículo, [Microsoft][Microsoft], y que observando sus capacidades hace sentido que por defecto no tengan miembros, _lol_ cito: _By default, the group has no members. Members of the Server Operators group can sign in to a server interactively, create and delete network shared resources, **start and stop services**, back up and restore files, format the hard disk drive of the computer, and shut down the computer. This group cannot be renamed, deleted, or moved._

 
🖱️_by:_ *@DaVinciRoot*

[Microsoft]: [https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators)
