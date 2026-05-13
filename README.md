# PROYECTO: Ataque Evil Twin contra WPA2 Enterprise
**Asignatura:** Hacking Ético  
**Práctica:** 3 UD 2 - Proyecto
**Link Video** https://drive.google.com/file/d/1UOKbvtcbT80VORz4m3XSZ9tKNzdvC7B8/view?usp=sharing

---

## 1. Introducción
El objetivo es realizar una auditoría de redes Wifi mediante ataque Evil Twin. En este tipo de ataque, el objetivo es conseguir que la víctima se conecte a un punto de acceso malicioso que suplanta la red wifi de la empresa que usa WPA Enterprise con RADIUS

## 2. Información sobre la vulnerabilidad
**Tipo de ataque:** Rogue Access Point / Evil Twin.  
**Dificultad:** Alta.  
**Protocolos vulnerados:** 802.11, 802.1X, RADIUS, MSCHAPv2.

**Descripción:**
El ataque consiste en levantar un punto de acceso falso con el mismo nombre (SSID) que la red corporativa. Al forzar o engañar a los clientes para que se conecten a nuestro AP malicioso en lugar del legítimo, logramos capturar el challenge/response de MSCHAPv2 de los usuarios. De esta manera se podrían robar las credenciales de acceso al dominio mediante diccionario, ya que es habitual que el RADIUS valide las credenciales contra un Active Directory.

## 3. Herramientas Utilizadas
Para la realización de esta práctica he utilizado el siguiente entorno y hardware:

* **Atacante:** Kali Linux.
* **Hardware Wi-Fi:** Tarjeta Alfa Wireless USB3.0 AWUS036ACH 
* **Software de ataque:** `hostapd-wpe`.
* **Redes objetivo en el aula:** CETI

## 4. Escenario del laboratorio

| Rol | Sistema |
|---|---|
| Atacante | Kali Linux |
| Víctima | Dispositivo Cliente (Windows 10 / Dispositivo Móvil) |

- Adaptador de red inalámbrico en modo Monitor/AP.
- Suplantación de red corporativa 802.1x.
- Interacción manual de la víctima ante alerta de certificado

## 5. Pasos para realizar el ataque

### Paso 1: Verificación de la tarjeta de red
Antes de iniciar, es obligatorio comprobar viendo la salida del comando `iw list` y observando si la tarjeta soporta modo AP. Esto es vital para que pueda funcionar como un punto de acceso malicioso.

```bash
# Comando para listar las capacidades de la interfaz
iw list | less
```
### Paso 2: Instalación de las herramientas necesarias
Para realizar la suplantación de la red corporativa y capturar los hashes, usaremos la herramienta hostapd-wpe (Wireless Pwnage Edition) disponible en los repositorios oficiales de Kali Linux.

```bash
# Comando para instalar hostapd-wpe
sudo apt update && sudo apt install hostapd-wpe -y
```

### Paso 3: Configuración del AP falso
Para que el ataque sea efectivo, nuestro punto de acceso debe suplantar exactamente la identidad de la red objetivo. Debemos editar el archivo de configuración de `hostapd-wpe` para establecer el SSID correcto (en nuestro caso, la red del aula `CETI` o `WIFI_HE`) y definir nuestra interfaz de red inalámbrica.

```bash
# Editamos el archivo de configuración principal
sudo nano /etc/hostapd-wpe/hostapd-wpe.conf
```

### Paso 4: Ejecución del ataque
Con el archivo configurado, procedemos a levantar nuestro punto de acceso falso. Al ejecutar el siguiente comando, nuestra tarjeta de red comenzará a radiar la red suplantada y la herramienta se quedará a la escucha, esperando que los clientes intenten autenticarse.

```bash
# Comando para lanzar el AP malicioso
sudo hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf
```

### Paso 5: Conexión del cliente y alerta de seguridad en iOS

Para simular el ataque, utilizamos un dispositivo cliente (iPhone 13). El objetivo es que la víctima intente conectarse a nuestro punto de acceso falso, ya sea manualmente o de forma automática si tenía la red guardada previamente.

Al intentar establecer la conexión, el sistema operativo iOS detecta que el certificado del servidor RADIUS presentado por nuestra herramienta (`hostapd-wpe`) no coincide con el legítimo de la infraestructura corporativa. Por ello, corta temporalmente la conexión y muestra a la víctima una advertencia de **"Certificado no confiable"**. 

Para que el ataque continúe y la contraseña sea enviada, la víctima debe ser engañada y pulsar en **"Confiar"** (Trust).

![Advertencia de certificado en iPhone 13](capturas/iphone_certificado.PNG)

### Paso 6: Captura del Challenge/Response (MSCHAPv2)

En el momento en que la víctima pulsa "Confiar" y acepta el certificado malicioso, el dispositivo envía sus credenciales. En la terminal del atacante podemos observar cómo se completa el proceso de autenticación.

La herramienta logra interceptar la comunicación y nos muestra por pantalla el nombre de usuario y los hashes capturados (*Challenge* y *Response* de MSCHAPv2). Además, nos proporciona directamente la estructura del comando (para `john` o `asleap`) que necesitaremos en la siguiente fase para crackear la contraseña.

![Verificación de la captura del hash](capturas/captura_mschapv2.PNG)

## 6. Captura y obtención de credenciales (Crackeo)

A diferencia de un ataque web donde las credenciales pueden viajar en texto plano, en WPA2 Enterprise capturamos un hash (MSCHAPv2). Aunque el atacante no le proporcione internet a la víctima (y esta sufra un error de conexión), el hash ya ha sido comprometido. 

Para demostrar el riesgo real, vamos a realizar un ataque de diccionario offline para obtener la contraseña en claro.

### Paso 1: Preparación del Hash
La herramienta `hostapd-wpe` nos proporciona directamente la cadena de texto formateada para herramientas de crackeo. Copiamos la línea correspondiente a `jtr NETNTLM` y la guardamos en un archivo de texto llamado `hash.txt`.

```bash
# Guardamos el hash capturado en un archivo local
 echo "testuser:$NETNTLM$5dc3a802c2365576$b53de16e508eb95891c0eb62e40bbc00f6665963cfdb3d61" > hash.txt
```

### Paso 2: Ejecución del ataque de diccionario con John The Ripper
Utilizamos la herramienta John The Ripper, indicándole el formato del hash (netntlm) y pasándole un diccionario de contraseñas comunes (en este caso, rockyou.txt predeterminado en Kali).

```bash
# Comando para crackear el hash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=netntlm hash.txt
```
### Paso 3: Verificación de credenciales obtenidas
Como la contraseña utilizada por la víctima se encontraba dentro de nuestro diccionario, la herramienta logra calcular la colisión del hash en cuestión de segundos, mostrándonos la credencial en texto plano.

Con esto, se demuestra el compromiso total de la cuenta de dominio del usuario, permitiendo a un atacante acceder a la intranet y recursos corporativos de la organización.
