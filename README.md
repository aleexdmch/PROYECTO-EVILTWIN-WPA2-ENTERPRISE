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

### Paso 2: Instalación de las herramientas necesarias
Para realizar la suplantación de la red corporativa y capturar los hashes, usaremos la herramienta hostapd-wpe (Wireless Pwnage Edition) disponible en los repositorios oficiales de Kali Linux.

# Comando para instalar hostapd-wpe
sudo apt update && sudo apt install hostapd-wpe -y
