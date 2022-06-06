---
title: "LAPS Local Administrator Password Solution"
layout: post
---

<h2>LAPS Local Administrator Password Solution </h2>
Uno de los problemas con lo que lidia toda seguridad en una infraestructura es el manejo de credenciales para los distintos dispositivos de la red, credenciales de acceso para servidores, para la administracion de router, firewall y switches, soluciones como servidores de autenticacion remotos o Single Sign On, ayudan a la solucion de esto, pero hoy quiero hablarle de la solucion de Microsoft para el re-uso de credenciales por partes de los administradores.

Antes de LAPS una practica comun entre administradores era utilizar la misma contrasena en cada computadora, con el objetivo de poder acceder in caso de que algun evento lo impidiera. Esto trae como resultado que el comprometer una maquina cliente, se pudiera obtener el hash del RID 500 y loguearnos con el en cualquier computadora del dominio a traves de la cuenta local del administrador. 

_<h6> Brute Forcing SID 500 in Active Directory [Mark Mo][Mark Mo] </h6>_

<h2>¿Como funciona?</h2>

Se instala un componente en cada computadora el cual genera cada cierto tiempo de manera _aleatoria_ contrasenas que sonn guardadas automaticamente en el Directorio Activo, adjuntadas como atributos de la computadora en cuestion y que solo el administrador puede ver el valor de este atributo.

La configuracion de LAPS en el dominio trae consigo la cuestion de a quien se le deberia otorgar permisos para poder actualizar contrasenas y demas, como a usuarios fuera del grupo de _admins_ del dominio, como grupo de helpdesk, asunto que puede salirse de las manos con el tiempo y terminar otorgando permisos a usuarios. 
  



[Mark Mo]: https://medium.com/@markmotig/brute-forcing-sid-500-in-active-directory-c9eb7c01a8a6
[Microsoft]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0
