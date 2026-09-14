# Escalación de privilegios en FiberHome HG6246R

Este repositorio documenta un método probado para obtener acceso de administrador en un **FiberHome HG6246R** de Mundo Pacífico (Chile), partiendo de una cuenta básica `user/user1234`.

## Contexto

Anteriormente, algunos equipos FiberHome de Mundo Pacífico utilizaban credenciales administrativas conocidas como `admin/operaciones`.

Actualmente, estas credenciales ya no están disponibles de la misma forma en los equipos provisionados por el ISP. Según lo documentado en investigaciones previas, **TR-069** se utiliza para aprovisionar y gestionar remotamente la configuración del dispositivo, incluyendo las credenciales administrativas.

Las nuevas credenciales no son visibles públicamente. Por ello, en lugar de depender de las credenciales administrativas conocidas anteriormente, este método parte de una cuenta básica `user/user1234` y aprovecha la exposición de la configuración para recuperar el valor utilizado por la cuenta administrativa.


La prueba se realizó sobre un dispositivo con:

* **Modelo:** FiberHome HG6246R
* **Firmware:** RP2952
* **Hardware:** WKE2.094.351A01
* **ISP:** Mundo Pacífico (Chile)

El procedimiento no requiere un reset de fábrica ni desconectar la fibra.

## Resumen

El panel web oculta determinadas funciones administrativas a los usuarios básicos mediante comprobaciones realizadas en la interfaz.

Modificando determinadas respuestas HTTP con **Burp Suite**, un usuario con privilegios básicos puede hacer que el frontend muestre funciones administrativas que normalmente permanecen ocultas.

Entre estas funciones se encuentra la opción de descargar el archivo de configuración `usrconfig_conf`.

El archivo puede ser procesado con una herramienta pública de análisis de configuraciones de FiberHome. Después de descifrar la configuración, es posible localizar el parámetro:

WebSuperPassword

Durante la prueba, el valor almacenado en este parámetro pudo ser procesado para recuperar la contraseña utilizada por la cuenta administrativa.

Finalmente, la credencial recuperada permitió iniciar sesión mediante la cuenta `admin` y obtener acceso a las funciones administrativas del router.

## Hallazgo

La cadena observada durante la prueba está compuesta por varios problemas:

1. **Controles de privilegios en el frontend:** determinadas funciones de administración dependen de valores recibidos por el cliente, que pueden ser modificados antes de llegar al navegador.
2. **Exposición del archivo de configuración:** una cuenta básica puede llegar a visualizar y utilizar la función de descarga de `usrconfig_conf`.
3. **Secreto administrativo recuperable:** el archivo de configuración contiene `WebSuperPassword` en una forma que puede ser procesada para recuperar la credencial.
4. **Acceso administrativo:** la credencial recuperada funciona para iniciar sesión mediante la cuenta `admin`.

El problema principal no es simplemente que un usuario pueda visualizar menús administrativos. El impacto aparece cuando la manipulación de la interfaz permite llegar al archivo de configuración que contiene información sensible utilizada por la cuenta administrativa.


## Herramientas

* Burp Suite Community
* `fh-config-utility-windows-x64.exe`

## Créditos

Herramienta utilizada durante el análisis:

`https://github.com/numberonedz/fiberhome-config-utility`
