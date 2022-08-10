---
title: "HA-FortiAnalyzer"
layout: post
---
![Fortigate](/assets/images/Ha-fg.png)


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
<h2>VLANs en Fortigate</h2>
![Fortigate](/assets/images/interface.png)

Dentro de la seccion de Network, nos vamos al apartado de interfaces.

{% highlight bash %} [+] Create New {% endhighlight %}
{% highlight bash %} Interface {% endhighlight %}

![Fortigate](/assets/images/vlan102.png)

Vamos a asignar la direccion ip de forma manual 172.22.0.1/24
Habilitamos ping para hacer prueba inter-Vlans al final.

#WorkingOnIt




🖱️_by:_ *@DaVinciRoot*

[Hacking-Article]: https://www.hackingarticles.in/credential-dumping-group-policy-preferences-gpp/
[Microsoft]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0
