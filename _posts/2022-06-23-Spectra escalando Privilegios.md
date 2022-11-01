---
title: "Spectra HTB Escalando Privilegio"
layout: post
---
![Spectra HTB](/assets/images/Spectra.png)

<h2>Presentación</h2>
En esta ocasión no estaré presentando el writeUp de la máquina Spectra, más bien solo hablaré de la parte de escalación de privilegios, que me resultó muy interesante, y así haré con algunas que otras máquinas mientras en otro les traigo el writeUp, y ya estaremos presentado contenido de Networking y Firewall 🤯.

De todas formas cuento con un repositorio privado con notas al momento de hacer la máquina, que puedo compartirlo si me escriben a mi dirección de correo, pues sin más empecemos. 

<h2>Identificando permisos de Sudoers</h2>
Pues luego de correr algunos comando de reconocimiento a los fines de escalar privilegios, al momento de ejecutar:
{% highlight bash %}  sudo -l  {% endhighlight %}

![Spectra HTB Escalando Privilegio ](/assets/images/Image 3b.png)

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
