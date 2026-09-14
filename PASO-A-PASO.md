# Paso a paso: de `user` a `admin` en FiberHome HG6246R

> **Aviso:** Los valores de credenciales y `WebSuperPassword` mostrados en este documento son **ficticios y únicamente sirven como ejemplo**. No corresponden a las credenciales reales obtenidas durante la prueba.

## Contexto

* **Modelo:** FiberHome HG6246R
* **Firmware:** RP2952
* **ISP:** Mundo Pacífico (CHL_MP)
* **Hardware:** WKE2.094.351A01
* **IP del router:** `192.168.1.1`
* **Panel web:** `http://192.168.1.1` (HTTP, puerto 80)

## Credenciales de partida

* **Usuario:** `user`
* **Contraseña:** `user1234` (por defecto de Mundo Pacífico)

## Herramientas necesarias

* **Burp Suite Community** (proxy HTTP)
* **`fh-config-utility-windows-x64.exe`** (herramienta de descifrado) https://github.com/numberonedz/fiberhome-config-utility

---

## Fase 1 — Burp Suite

### 1.1 Configurar Burp

1. Abrir Burp Suite.
2. Ir a `Proxy → Options → Proxy Listeners`.
3. Verificar que escucha en `127.0.0.1:8080`.
4. Ir a `Proxy → Intercept` y dejar **Intercept is off**.
5. Abrir el navegador de Burp (`Proxy → Intercept → Open browser`).

### 1.2 Iniciar sesión como usuario básico

1. En el navegador, ir a `http://192.168.1.1`.
2. Iniciar sesión con:

   * Usuario: `user`
   * Contraseña: `user1234`
3. Navegar por el panel para generar tráfico.

**Hallazgo:** el frontend decide qué mostrar según el valor de `login_user` que devuelve el backend. Si modificamos esa respuesta, el frontend cree que somos admin.

---

## Fase 2 — Modificar respuestas con Burp (Match and Replace)

### 2.1 Añadir 3 reglas cada una en Match and Replace

En Burp: `Proxy → Options → Match and Replace` → **Add**.

**Regla 1:**

* Type: Response body
* Match: "login_user":\s*"0"
* Replace: "login_user":"1"
* Regex match: ✅

**Regla 2:**

* Type: Response body
* Match: \{"result":"1",\s*"user":"0"\}
* Replace: {"result":"1", "user":"1"}
* Regex match: ✅

**Regla 3:**

* Type: Response body
* Match: "factory_mode":\s*0
* Replace: "factory_mode":1
* Regex match: ✅

### Debes copiar y pegar exactamente todo después de los dos puntos.

### 2.2 Verificar

1. Recargar el panel en el navegador.
2. El panel ahora **muestra los menús de admin** (`Status`, `Network`, `Security`, `Application`, `Management`).
3. **Esto es solo cosmético** — el backend sigue rechazando acciones reales de admin.
4. **Pero** ahora tenemos acceso visual a las opciones que solo el admin ve.

**Por qué funciona:** el backend **no valida rol** en las peticiones GET de lectura. El frontend sí, pero modificando las respuestas, se salta.

---

## Fase 3 — Descargar el archivo de configuración

### 3.1 Localizar la opción de descarga

Con las reglas activas:

1. Ir a `Management → Device Management → Configuration File`.
2. Hacer click en **Configuration file back up**.
3. Guardar el archivo descargado como `usrconfig_conf`.

El archivo obtenido es un binario de aproximadamente **12 KB**.

---

## Fase 4 — Descifrar `usrconfig_conf`

### 4.1 Obtener `fh-config-utility-windows-x64.exe`

Para analizar el archivo de configuración se utilizó la herramienta pública:

`https://github.com/numberonedz/fiberhome-config-utility`

La herramienta permite trabajar con archivos de configuración de FiberHome y también procesar cadenas individuales.

### 4.2 Descifrar el archivo

Con el archivo `usrconfig_conf` descargado, utilizarás la primera opción de la herramienta que permite descrifrar el archivo, lo que te guardará un nuevo archivo llamado: `usrconfig_conf_DECRYPTED` pero a diferencia del normal este es legible en texto plano

El objetivo de esta fase es localizar el parámetro:

`WebSuperPassword`

### 4.3 Localizar `WebSuperPassword`

Dentro del archivo descifrado, buscar:

`WebSuperPassword`

Por ejemplo, una configuración podría contener:

WebSuperPassword 'A1B2C3D4E5F60718293A4B5C6D7E8F90'

> **Importante:** El valor anterior es completamente ficticio y se utiliza únicamente para explicar el procedimiento. No corresponde al valor real obtenido durante la prueba.

### 4.4 Descifrar el valor

`fh-config-utility-windows-x64.exe` también permite descifrar cadenas individuales.

Con la cadena individual encontrada y copiada de `WebSuperPassword`, utilizarás la tercera opcion de la herramienta que permite descifrar esa cadena invidual, pegando la cadena te entregará la contraseña de `WebSuperPassword` en texto plano, lo cual es el equivalente a la contraseña de login como administrador del panel del router 


> **Importante:** La cadena y el resultado de ella es una contraseña inventada para esta documentación. No es la contraseña real del dispositivo analizado pero el procedimiento y el resultado siguen siendo el mismo.

---

## 4.5 Verificación del acceso a admin

Finalmente, se utilizó la credencial recuperada para iniciar sesión en el panel web mediante la cuenta:

`admin`

El inicio de sesión fue exitoso y se obtuvo acceso a las funciones administrativas del router.

Esto confirma la escalada desde la cuenta básica utilizada inicialmente hasta la cuenta administrativa.


---


## Conclusión

El procedimiento permite pasar de una cuenta con privilegios básicos a una cuenta administrativa mediante una cadena de fallos en el panel web y en el manejo de la configuración.

El problema más relevante no es únicamente que el frontend pueda mostrar menús administrativos, sino que una cuenta básica puede llegar a obtener material de configuración que contiene un secreto recuperable utilizado por la cuenta de superadministrador.

Los valores mostrados en este documento (`A1B2C3D4E5F60718293A4B5C6D7E8F90`) son ejemplos ficticios.

## Créditos

Herramienta utilizada durante el análisis:

- [numberonedz/fiberhome-config-utility](https://github.com/numberonedz/fiberhome-config-utility)