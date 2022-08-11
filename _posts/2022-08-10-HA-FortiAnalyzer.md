---
title: "HA-FortiAnalyzer"
layout: post
---
![Fortigate](/assets/images/Ha-fg.png)


<h2>Presentación</h2>
Esto es una red que disfruté bastante al momento de su creación y configuración, consiste en la configuración de _High Availability_ en los fortigate enviando sus logs a el FortiAnalyzer a través de un fortigate en _transparent mode_ para escenificar un switch de capa 2 con un extra de funciones, las configuraciones fueron hechas en GUI en su mayoría, estaré mostrando parte de la misma y concluyendo con el recibimiento de logs en el FortiAnalyzer, además del registro de los fortigate en FortiAnalyzer. 

<h2>La RED</h2>

En la imagen tenemos dos fortigate en esta ocasión lo vamos a configurar en modo activo-pasivo, haremos una prueba de failover para ver que sucede en nuestro equipos mientras observamos los logs en el FortiAnalyzer, tenemos los pruertos 9 y 10 de cada Fortigate son heartbeat links, ambos fortigates envian logs por la interfaz 4, el fortigate en transparent mode tambien agregado a el Forti-Analyzer y finalmente el FortiAnalyzer directamente conectado a la PC interna de configuración, la cual sería nuestra PC de configuración, en un ambiente real, nos permite aplicar configuración tanto de los fortigates, el FG en transparent mode, como el FortiAnalyzer.

{% highlight bash %} {% endhighlight %}
{% highlight bash %} l {% endhighlight %}

#WorkingOnIt


🖱️_by:_ *@DaVinciRoot*


