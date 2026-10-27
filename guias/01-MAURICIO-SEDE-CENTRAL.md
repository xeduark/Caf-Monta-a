# Guía de Mauricio – Sede central (Medellín)

**Tú empiezas.** Creas el archivo `CafeMontana.pkt`, armas la sede central completa, la pruebas y se la envías a **Andrés**.

Tu sede tiene 3 redes:

| Red | Dirección | Gateway (R-MEDELLIN) | Puerto del router |
|---|---|---|---|
| LAN administrativa | 172.20.0.128/26 | 172.20.0.129 | G0/0 |
| LAN servidores | 172.20.1.32/28 | 172.20.1.33 | G0/1 |
| LAN inalámbrica | 172.20.1.0/27 | 172.20.1.1 | G0/2 |

Además dejas listos los 2 puertos seriales hacia Bogotá (S0/0/0) y Cali (S0/0/1).

---

## Paso 1. Crear el archivo
1. Abre Packet Tracer y ve a **File → Save As** → `CafeMontana.pkt`.
2. Guarda con **Ctrl + S** cada vez que termines un paso.

---

## Paso 2. Poner los dispositivos

Abajo a la izquierda están las categorías de dispositivos. Arrastra cada uno al área blanca y ponle nombre: clic en el dispositivo → pestaña **Config** → **Display Name**.

| Cantidad | Categoría → Dispositivo | Nombre |
|---|---|---|
| 1 | Network Devices → Routers → **2911** | `R-MEDELLIN` |
| 2 | Network Devices → Switches → **2960** | `SW-ADMIN`, `SW-SERVIDORES` |
| 1 | Network Devices → Wireless Devices → **AccessPoint-PT** | `AP-MED` |
| 2 | End Devices → **PC** | `PC-ADM1`, `PC-ADM2` |
| 1 | End Devices → **Printer** | `IMP-MED` |
| 2 | End Devices → **Server** | `SRV-DHCP-DNS`, `SRV-WEB` |
| 2 | End Devices → **Laptop** | `LAPTOP-1`, `LAPTOP-2` |

> 💡 Deja espacio abajo del router: ahí van las sedes de Andrés (izquierda) y Edward (derecha).
> 💡 Para que se vea ordenado, puedes encerrar tu sede en un rectángulo: barra superior → **Draw Rectangle**. Ponle el texto "SEDE CENTRAL – MEDELLÍN" con la herramienta **Place Note** (ícono de hoja).

---

## Paso 3. Módulo serial en R-MEDELLIN
El 2911 no trae puertos seriales, así que hay que agregarle un módulo.

1. Clic en **R-MEDELLIN** → pestaña **Physical**.
2. **Apaga** el router: clic en el botón de encendido (la luz verde pasa a negra).
3. En la lista de la izquierda busca **HWIC-2T** y **arrástralo** a una ranura vacía.
4. **Vuelve a encender** el router. ⚠️ No lo olvides.

Ahora tienes los puertos **Serial0/0/0** y **Serial0/0/1**.

---

## Paso 4. Portátiles con tarjeta inalámbrica
Los portátiles vienen con tarjeta de cable, así que hay que cambiarla por una inalámbrica. Repite esto en **LAPTOP-1** y **LAPTOP-2**:

1. Clic en el portátil → pestaña **Physical**.
2. **Apágalo**: botón de encendido en la imagen del portátil.
3. **Arrastra la tarjeta** que tiene puesta (abajo del portátil) **hacia la lista de la izquierda** para quitarla.
4. En la lista, busca **WPC300N** y arrástrala al hueco que quedó.
5. **Enciéndelo** de nuevo.

---

## Paso 5. Cables
Todos con **Copper Straight-Through** (línea negra continua, en el ícono del rayo ⚡).

| Desde | Puerto | Hacia | Puerto |
|---|---|---|---|
| R-MEDELLIN | **GigabitEthernet0/0** | SW-ADMIN | GigabitEthernet0/1 |
| R-MEDELLIN | **GigabitEthernet0/1** | SW-SERVIDORES | GigabitEthernet0/1 |
| R-MEDELLIN | **GigabitEthernet0/2** | AP-MED | Port 0 |
| PC-ADM1 | FastEthernet0 | SW-ADMIN | FastEthernet0/1 |
| PC-ADM2 | FastEthernet0 | SW-ADMIN | FastEthernet0/2 |
| IMP-MED | FastEthernet0 | SW-ADMIN | FastEthernet0/3 |
| SRV-DHCP-DNS | FastEthernet0 | SW-SERVIDORES | FastEthernet0/1 |
| SRV-WEB | FastEthernet0 | SW-SERVIDORES | FastEthernet0/2 |

> Los portátiles **no llevan cable**: se conectan por wifi en el Paso 9.
> **No conectes los seriales.** Esos cables los ponen Andrés y Edward.

---

## Paso 6. Configurar R-MEDELLIN
Clic en R-MEDELLIN → pestaña **CLI**. Si pregunta `initial configuration dialog? [yes/no]`, escribe `no` y presiona Enter. Luego pega todo este bloque:

```
enable
configure terminal
hostname R-MEDELLIN
no ip domain-lookup
banner motd #Cafe Montana - Router Central Medellin - Solo personal autorizado#

interface GigabitEthernet0/0
 description LAN Administrativa
 ip address 172.20.0.129 255.255.255.192
 ip helper-address 172.20.1.34
 no shutdown
 exit

interface GigabitEthernet0/1
 description LAN Servidores
 ip address 172.20.1.33 255.255.255.240
 no shutdown
 exit

interface GigabitEthernet0/2
 description LAN Inalambrica
 ip address 172.20.1.1 255.255.255.224
 ip helper-address 172.20.1.34
 no shutdown
 exit

interface Serial0/0/0
 description WAN1 hacia Bogota
 ip address 172.20.1.49 255.255.255.252
 no shutdown
 exit

interface Serial0/0/1
 description WAN2 hacia Cali
 ip address 172.20.1.53 255.255.255.252
 no shutdown
 exit

end
write
```

**Qué hace cada cosa** (para la sustentación):
- `no ip domain-lookup`: si escribes mal un comando, el router no se queda "congelado" buscándolo como nombre de dominio.
- `banner motd`: el mensaje de advertencia que aparece al entrar al router.
- `ip helper-address 172.20.1.34`: el servidor DHCP está en **otra subred** (servidores). Este comando reenvía al servidor las solicitudes DHCP de las PCs administrativas y de los portátiles. **G0/1 no lo necesita**, porque los servidores tienen IP fija.
- Los seriales quedarán en **down** hasta que Andrés y Edward conecten sus routers. Es normal.
- El `clock rate` y las rutas hacia Bogotá y Cali los agregan Andrés y Edward cuando conecten sus cables.

Verifica:
```
show ip interface brief
```
G0/0, G0/1 y G0/2 deben estar en **up / up**. G0/2 puede quedar *down* si el AP no está conectado.

---

## Paso 7. Configurar los switches
En la **CLI** de cada switch.

**SW-ADMIN**
```
enable
configure terminal
hostname SW-ADMIN
no ip domain-lookup
banner motd #Cafe Montana - Switch Administrativo Medellin#
interface GigabitEthernet0/1
 description Enlace a R-MEDELLIN G0/0
interface FastEthernet0/1
 description PC-ADM1
interface FastEthernet0/2
 description PC-ADM2
interface FastEthernet0/3
 description IMP-MED
end
write
```

**SW-SERVIDORES**
```
enable
configure terminal
hostname SW-SERVIDORES
no ip domain-lookup
banner motd #Cafe Montana - Switch de Servidores Medellin#
interface GigabitEthernet0/1
 description Enlace a R-MEDELLIN G0/1
interface FastEthernet0/1
 description SRV-DHCP-DNS
interface FastEthernet0/2
 description SRV-WEB
end
write
```

---

## Paso 8. Servidores

### 8.1 SRV-WEB
1. **Desktop → IP Configuration → Static**
   - IPv4 Address: `172.20.1.35`
   - Subnet Mask: `255.255.255.240`
   - Default Gateway: `172.20.1.33`
   - DNS Server: `172.20.1.34`
2. **Services → HTTP**: revisa que **HTTP** esté en **On**.
3. En la lista de archivos, haz clic en **edit** junto a `index.html`. Borra el contenido y escribe:
   ```html
   <html><body>
   <h1>Cafe Montana S.A.S.</h1>
   <p>Intranet corporativa - Servidor web de la sede central (Medellin)</p>
   </body></html>
   ```
   Haz clic en **Save** y confirma con **Yes**.
4. **Services → DHCP**: revisa que esté en **Off**. Solo el otro servidor reparte IPs.

### 8.2 SRV-DHCP-DNS: IP fija
**Desktop → IP Configuration → Static**
- IPv4 Address: `172.20.1.34`
- Subnet Mask: `255.255.255.240`
- Default Gateway: `172.20.1.33` ← **indispensable**: sin gateway no le responde a nadie fuera de su red.
- DNS Server: `172.20.1.34`

### 8.3 SRV-DHCP-DNS: servicio DNS
1. **Services → DNS** → **DNS Service: On**.
2. Llena:
   - Name: `www.cafemontana.com`
   - Type: `A Record`
   - Address: `172.20.1.35`
3. Haz clic en **Add**.

### 8.4 SRV-DHCP-DNS: servicio DHCP
1. **Services → DHCP** → **Service: On**.
2. **serverPool** (ya existe; lo dejamos para la red de servidores, como reserva de crecimiento). Selecciónalo en la tabla y cambia:
   - Default Gateway: `172.20.1.33`
   - DNS Server: `172.20.1.34`
   - Start IP Address: `172.20.1.40`
   - Subnet Mask: `255.255.255.240`
   - Maximum Number of Users: `5`
   - Haz clic en **Save**.
3. **Pool ADMIN.** Escribe en los campos y haz clic en **Add** (no en Save):
   - Pool Name: `ADMIN`
   - Default Gateway: `172.20.0.129`
   - DNS Server: `172.20.1.34`
   - Start IP Address: `172.20.0.140`
   - Subnet Mask: `255.255.255.192`
   - Maximum Number of Users: `50`
4. **Pool WIFI.** Igual, con **Add**:
   - Pool Name: `WIFI`
   - Default Gateway: `172.20.1.1`
   - DNS Server: `172.20.1.34`
   - Start IP Address: `172.20.1.10`
   - Subnet Mask: `255.255.255.224`
   - Maximum Number of Users: `20`

En la tabla deben quedar **3 pools**: serverPool, ADMIN y WIFI. Los pools BOGOTA y CALI los agregan Andrés y Edward.

> Deja TFTP Server y WLC Address en `0.0.0.0`: significa "ninguno".

---

## Paso 9. Red inalámbrica

### 9.1 Access point (AP-MED)
1. Clic en **AP-MED** → **Config** → a la izquierda, **Port 1** (el puerto inalámbrico).
2. Configura:
   - SSID: `CAFE-MONTANA`
   - Authentication: **WPA2-PSK**
   - PSK Pass Phrase: `Montana2026` (mínimo 8 caracteres)
   - Encryption Type: **AES**

### 9.2 Portátiles (LAPTOP-1 y LAPTOP-2)
1. Clic en el portátil → **Config** → a la izquierda, **Wireless0**.
2. Configura exactamente lo mismo que en el AP:
   - SSID: `CAFE-MONTANA`
   - Authentication: **WPA2-PSK**
   - PSK Pass Phrase: `Montana2026`
   - Encryption Type: **AES**
3. En esa misma pantalla, en **IP Configuration**, marca **DHCP**.

✔️ En unos segundos aparecerá una **línea punteada (ondas)** entre el AP y el portátil. Ya está conectado.

---

## Paso 10. Impresora y PCs

### IMP-MED (estática)
1. **Config → Settings** (Global): Gateway = `172.20.0.129`. DNS Server = `172.20.1.34`.
2. **Config → FastEthernet0**: **Static**, IP Address = `172.20.0.130`, Subnet Mask = `255.255.255.192`.

### PC-ADM1 y PC-ADM2 (DHCP)
**Desktop → IP Configuration → DHCP**. Debe aparecer **"DHCP request successful"** con una IP entre **172.20.0.140** y **172.20.0.189**.

> Si falla: revisa que el DHCP esté en **On**, que el pool ADMIN tenga gateway **172.20.0.129** y que G0/0 tenga el `ip helper-address`. Para reintentar, marca **Static** y luego otra vez **DHCP**.

---

## Paso 11. Pruebas de tu sede y capturas 📸
Toma **captura de cada prueba** (Win + Shift + S) y guárdalas con el nombre indicado. Después se las envías a Edward.

| # | Dónde | Comando | Resultado esperado | Nombre de la captura |
|---|---|---|---|---|
| 1 | PC-ADM1 | `ipconfig` | IP .0.140 – .0.189, gateway .0.129, DNS .1.34 | `M01-ipconfig-PC-ADM1.png` |
| 2 | LAPTOP-1 | `ipconfig` | IP .1.10 – .1.29, gateway .1.1 | `M02-ipconfig-LAPTOP-1.png` |
| 3 | PC-ADM1 | `ping` a la IP de PC-ADM2 | Responde (misma subred) | `M03-ping-misma-subred.png` |
| 4 | PC-ADM1 | `ping 172.20.0.130` | Responde (impresora) | `M04-ping-impresora.png` |
| 5 | PC-ADM1 | `ping 172.20.1.35` | Responde (otra subred: servidores) | `M05-ping-servidor.png` |
| 6 | LAPTOP-1 | `ping` a la IP de PC-ADM1 | Responde (wifi → cableado) | `M06-ping-wifi-admin.png` |
| 7 | PC-ADM1 | Desktop → **Web Browser** → `www.cafemontana.com` | Se ve la página de Café Montaña | `M07-web-PC-ADM1.png` |
| 8 | R-MEDELLIN | `show ip interface brief` | G0/0, G0/1 y G0/2 up/up | `M08-show-ip-int-brief.png` |

> El primer ping puede salir *Request timed out* por el ARP. Repítelo.

---

## Paso 12. Entregar el turno a Andrés
1. Guarda: **Ctrl + S**.
2. Guarda también una copia: `CafeMontana-v1-mauricio.pkt`.
3. Envía `CafeMontana.pkt` a **Andrés**.

---

## Tu parte del documento técnico
1. **Descripción de la empresa:** nombre, actividad económica, sedes y número de usuarios (ver el plan general).
2. **Requerimientos tecnológicos:** por qué cada sede necesita PCs, impresoras, servidores y wifi.
3. **Topología física y lógica:** captura de la red completa y explicación de la estrella.
4. **Justificación:** por qué estrella, por qué 3 subredes en Medellín (organización y seguridad: los visitantes del wifi no comparten red con contabilidad) y la desventaja del punto único de falla.
5. **Sede central:** explica tus 3 redes, el helper y los servicios (DHCP, DNS y web).

## Lo que debes saber explicar en la sustentación
- Por qué R-MEDELLIN tiene `ip helper-address` en G0/0 y G0/2, pero no en G0/1.
- Cómo funciona el DNS: el navegador pregunta por `www.cafemontana.com`, el servidor DNS responde `172.20.1.35`, y el navegador abre la página del servidor web.
- Cómo se conectan los portátiles: SSID + WPA2-PSK + la misma clave en el AP y en el portátil.
- Por qué servidores e impresora son estáticos y las PCs usan DHCP.
