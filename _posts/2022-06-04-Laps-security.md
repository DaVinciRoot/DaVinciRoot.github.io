---
title: "LAPS Local Administrator Password Solution"
layout: post
---

<h2>LAPS Local Administrator Password Solution </h2>
Uno de los problemas con lo que lidia toda seguridad en una infraestructura es el manejo de credenciales para los distintos dispositivos de la red, credenciales de acceso para servidores, para la administración de router, firewall y switches, soluciones como servidores de autenticación remotos o Single Sign On, ayudan a la solución de esto, pero hoy quiero hablarle de la solución de Microsoft para el re-uso de credenciales por partes de los administradores, en el manejo de Window Server y usuarios del dominio.

Antes de LAPS una práctica común entre administradores era o es 🤌 utilizar la misma contraseña en cada computadora, con el objetivo de poder acceder en caso de que algún evento lo impidiera. Esto trae como resultado que el comprometer una máquina cliente, se pudiera obtener el hash del RID 500 y loguearnos con el, en cualquier computadora del dominio a través de la cuenta local del administrador. 

_Brute Forcing SID 500 in Active Directory [Mark Mo][Mark Mo]_

<h2>¿Cómo funciona?</h2>

Se instala un componente en cada computadora el cual genera cada cierto tiempo de manera _aleatoria_ contraseñas que son guardadas automáticamente en el Directorio Activo, adjuntadas como atributos de la computadora en cuestión y que solo el administrador puede ver el valor de este atributo, pero que si tienes los permisos es fácil de leer utilizando el cmdlet de PowerShell `Get-AdmPwdPassword`.

La configuración de LAPS en el dominio trae consigo la cuestión de a quien se le debería otorgar permisos para poder actualizar contraseñas y demás, como a usuarios fuera del grupo de _admins_ del dominio, como grupo de helpdesk, asunto que puede salirse de las manos con el tiempo y terminar otorgando permisos a usuarios que no deberian. 
  
*Que es justo lo que sucede en la máquina Timelapse de HTB...* una vez sea retirada traemos el WriteUp.


[Mark Mo]: https://medium.com/@markmotig/brute-forcing-sid-500-in-active-directory-c9eb7c01a8a6
[Microsoft]: https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0
