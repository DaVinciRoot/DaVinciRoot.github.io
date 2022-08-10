---
title: "VLANS en Fortigate"
layout: post
---
![Fortigate](/assets/images/vlan00.png)


<h2>Presentación</h2>
En esta ocasión estaremos configurando un Firewall Fortigate junto a un Switch de capa 3, diferentes Vlans, para la separación logica del tráfico, en otro post estaremos presentando que es una VLAN y para que se utilizan asi como sus beneficios. 

- PC1 pertenece a la VLAN-ID 101 Alumnos --> 172.21.0.0/24
- PC2 pertenece a la VLAN-ID 102 Docentes --> 172.22.0.0/24
- PC3 pertenece a la VLAN-ID 103 Administradores --> 172.23.0.0/24

En Fortigate es posible utilizar la misma interfaz física para la creación y tráfico de diferentes VLANs, ya que pemite identificar el tráfico con el VLAN-ID, y separar así la interfaz física de manera lógica, en una o más interfaces lógicas, a través del tag frame 802.1q agregado que permite identificarlas (VLAN-ID). 

<h2>Conociendo la RED</h2>

En la imagen tenemos un Fortigate de cara al internet este fortigate haciendo la función de _edge firewall_ o firewall de perímetro se encarga de procesar el tráfico que proviene de la red interna hacia internet y vice-versa, en este caso de las diferentes VLANS _Docentes, Alumnos y Administradores_ estos nombres para las diferentes son intencionalmente seleccionados para presentar unos de los mayores beneficios de la utilización de VLANS, y es que crear VLANS nos permite crear diseños más flexibles que agrupen a los usuarios por departamento, o por grupos que trabajan en lugar de por su ubicación física, y que como medida de seguridad nos permita implementar la aplicación de diferentes políticas de seguridad por VLANs, como Private Community VLAN o VACL, acls por VLAN que estaremos presentando más adelante.
  
<h2>VLANs</h2>
Crear, nombrar y asignar vlans es un proceso sencillo, como te mostraré a continuación, y ya en otro post ampliaremos sobre más atributos configurables con relación a las vlans.

Lo primero que hay que hacer es entrar o habilitar el modo de configuración, para eso ejecutamos los siguientes comandos: 

{% highlight bash %} enable {% endhighlight %}
{% highlight bash %} configure terminal {% endhighlight %}

<h2> Creación de VLAN Switch capa 3</h2>

Una vez dentro del modo de configuración que es posible identificar porque el terminar ahora nos añade (config)#.
![Fortigate](/assets/images/all vlans.png)

Es simplemente utilizar los comandos:

{% highlight bash %} vlan 101 {% endhighlight %} Donde el 101 es simplemente el ID de las vlans que habíamos especficado al principio, y entramos al modo de configuración de vlan (config-vlan)#
{% highlight bash %} name docentes {% endhighlight %} Donde docentes es simplemente el nombre de las vlans.

<h2> Asignar VLANs a diferentes interfaces </h2>

Para esto volvemos al modo de configuración de terminal(config)#
```
interface gigabitEthernet 0/1
switchport mode acess
switchport access vlan 101

```
<h2>VLANs en Fortigate</h2>
![Fortigate](/assets/images/interface.png)

Dentro de la sección de Network, nos vamos al apartado de interfaces.

{% highlight bash %} [+] Create New {% endhighlight %}
{% highlight bash %} Interface {% endhighlight %}

![Fortigate](/assets/images/vlan102.png)

- En _Name_ seleccionamos un nombre identificativo a la VLAN que se conecta.
- Asignamos el mismo _alias_ que el nombre de la vlan configurada en el switch esto para mejor identificación, no necesariamente deben coincidir.
- Notar por favor el tipo de interface selecionado en este caso _Type:VLAN_.
- Rol LAN en este caso.
La interfaz en este caso es el port2 (interface 2), es la misma para cada vlan, ya que mencioné en Fortigate _"es posible utilizar la misma interfaz física para la creación y tráfico de diferentes VLAN, y tráfico hacia la interfaz fisica no es taggeado, pertenecen a la native vlan, tema de otro dia"_ aunque no permite separar el tráfico broadcast en _transparent mode_ ya que todas las vlan pertenecen al mismo _forward domain_ que son como broadcast domain, por lo que a VLANs distintas configuramos _forward-domain_ distintos, de la siguiente forma:

{% highlight bash %} 
config system interface
  edit VLAN102 VLAN_102
  set forward-domain 102
 end
{% endhighlight %}

De esta forma tráfico en un interface es emitido solo a interfaces in el mismo _forward domain ID_.
- Vamos a asignar la dirección ip de forma manual 172.22.0.1/24
Habilitamos ping para hacer prueba inter-Vlans al final. (_no recomendado en entorno reales_).

Por último configuramos el DHCP server en cada interfaz tipo VLAN para que nuestro equipos reciban direccionamiento. 

![Fortigate](/assets/images/dhcp.png)

En este apartado configuramos en modo servidor, dentro del mismo rango de direcciones ip de la interfaz física, y asignamos como puerta de enlace la dirección de la interfaz.

![Fortigate](/assets/images/confvlan.png)

Podemos ver como las VLANs se muestran como parte de la interfaz port2.

<h2>Inter VLAN Royting a traves del Fortigate</h2>

Para esto nos vamos a _Policy & Obejct_ --> _Firewall policy_.

![Fortigate](/assets/images/policy.png)

Asignamos un nombre, en este ejemplo solo lo mostraremos, suponiendo que las distintas redes necesitan tráfico entre sí, aqui a modo de demostración aplicamos inter-vlan routing creando reglas en ambas direcciones, permitiendo solo comunicación ICMP, pero en ambiente reales deberíamos ser mas granulares a la hora de selecionar servicios. 

Para esto luego de crear la primera regla, solo debemos de hacer clic derecho en el _interface pairview pane_ clone reverse, asignar un nombre y activar.

![Fortigate](/assets/images/pair.png)

<h2>Ping-ing</h2>

![Fortigate](/assets/images/last.png)

En la imagen observamos una máquina de VLAN102, recibiendo direccionamiento ip mediante dhcp, haciendo ping a su puerta de enlance y a otra pc en la VLAN101.

🖱️_by:_ *@DaVinciRoot*

[Hacking-Article]: https://www.hackingarticles.in/credential-dumping-group-policy-preferences-gpp/
[Microsoft]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0
