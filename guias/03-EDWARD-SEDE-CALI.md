# Guía de Edward – Sede remota 2 (Cali)

**Tu turno empieza cuando Andrés te envía `CafeMontana.pkt`.** Agregas la sede Cali, la conectas a Medellín por el enlace **WAN2**, la pruebas y dejas la red completa. Después **coordinas las pruebas finales** y armas la sección de evidencias del documento.

| Red | Dirección | Gateway (R-CALI) |
|---|---|---|
| LAN usuarios Cali | 172.20.0.192/26 (255.255.255.192) | 172.20.0.193 |
| WAN2 Medellín–Cali | 172.20.1.52/30 (255.255.255.252) | R-MEDELLIN .53 / R-CALI .54 |

**Mientras esperas el archivo**, prepara la estructura de tu parte del documento (al final de esta guía) y recibe las capturas de Mauricio y Andrés.

---

## Paso 1. Abrir y respaldar
1. Abre `CafeMontana.pkt`.
2. Ve a **File → Save As** → `CafeMontana-v3-edward.pkt`.
3. **No modifiques nada de Medellín ni de Bogotá**, excepto lo que dice el Paso 6.

---

## Paso 2. Poner los dispositivos
Ubícalos **abajo a la derecha** de R-MEDELLIN. Nombre: clic → **Config** → **Display Name**.

| Cantidad | Categoría → Dispositivo | Nombre |
|---|---|---|
| 1 | Network Devices → Routers → **1941** | `R-CALI` |
| 1 | Network Devices → Switches → **2960** | `SW-CALI` |
| 2 | End Devices → **PC** | `PC-CALI1`, `PC-CALI2` |
| 1 | End Devices → **Printer** | `IMP-CALI` |

> 💡 Encierra tu sede en un rectángulo (**Draw Rectangle**) y ponle la nota "SEDE REMOTA 2 – CALI" (**Place Note**).

---

## Paso 3. Módulo serial en R-CALI
1. Clic en **R-CALI** → pestaña **Physical**.
2. **Apágalo**.
3. Arrastra un **HWIC-2T** a una ranura vacía.
4. **Vuelve a encenderlo.** ⚠️

---

## Paso 4. Cables

### Cable serial WAN2
1. Selecciona el cable **Serial DCE** (rojo, con relojito).
2. **Primer clic en R-MEDELLIN** → elige **Serial0/3/1**. ⚠️ El **0/3/1**: el 0/3/0 ya es de Bogotá.
3. Segundo clic en **R-CALI** → elige **Serial0/0/0**.

### Cables de cobre (Copper Straight-Through)
| Desde | Puerto | Hacia | Puerto |
|---|---|---|---|
| R-CALI | **GigabitEthernet0/0** | SW-CALI | GigabitEthernet0/1 |
| PC-CALI1 | FastEthernet0 | SW-CALI | FastEthernet0/1 |
| PC-CALI2 | FastEthernet0 | SW-CALI | FastEthernet0/2 |
| IMP-CALI | FastEthernet0 | SW-CALI | FastEthernet0/3 |

Pasa el mouse sobre el cable serial: debe llegar a **Serial0/3/1** en R-MEDELLIN y a **Serial0/0/0** en R-CALI.

---

## Paso 5. Configurar R-CALI
**CLI** → responde `no` a la pregunta inicial → pega:

```
enable
configure terminal
hostname R-CALI
no ip domain-lookup
banner motd #Cafe Montana - Router Sede Cali - Solo personal autorizado#

interface GigabitEthernet0/0
 description LAN Usuarios Cali
 ip address 172.20.0.193 255.255.255.192
 ip helper-address 172.20.1.34
 no shutdown
 exit

interface Serial0/0/0
 description WAN2 hacia Medellin
 ip address 172.20.1.54 255.255.255.252
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 172.20.1.53

end
write
```

- `ip helper-address 172.20.1.34`: reenvía las solicitudes DHCP de las PCs de Cali al servidor de Medellín.
- `ip route 0.0.0.0 0.0.0.0 172.20.1.53`: ruta por defecto. Todo lo que no es de Cali se envía a R-MEDELLIN, **incluido lo que va para Bogotá**.

---

## Paso 6. Agregar lo tuyo en R-MEDELLIN
En la **CLI de R-MEDELLIN**:

```
enable
configure terminal
interface Serial0/3/1
 clock rate 64000
 exit
ip route 172.20.0.192 255.255.255.192 172.20.1.54
end
write
```
- `clock rate 64000`: el reloj del enlace serial WAN2, en el lado DCE.
- `ip route ...`: "para llegar a la LAN de Cali (172.20.0.192/26), envía los paquetes a R-CALI (172.20.1.54)".

Verifica en R-MEDELLIN:
```
show ip interface brief
```
**Serial0/3/0 y Serial0/3/1** deben estar en **up / up**. Si los dos están arriba, las 3 sedes ya están unidas.

---

## Paso 7. Configurar SW-CALI
```
enable
configure terminal
hostname SW-CALI
no ip domain-lookup
banner motd #Cafe Montana - Switch Sede Cali#
interface GigabitEthernet0/1
 description Enlace a R-CALI G0/0
interface FastEthernet0/1
 description PC-CALI1
interface FastEthernet0/2
 description PC-CALI2
interface FastEthernet0/3
 description IMP-CALI
end
write
```

---

## Paso 8. Agregar el pool CALI en el servidor DHCP
1. **SRV-DHCP-DNS** → **Services** → **DHCP**.
2. Escribe en los campos:
   - Pool Name: `CALI`
   - Default Gateway: `172.20.0.193`
   - DNS Server: `172.20.1.34`
   - Start IP Address: `172.20.0.200`
   - Subnet Mask: `255.255.255.192`
   - Maximum Number of Users: `40`
3. Haz clic en **Add** (no en Save).

En la tabla deben quedar **5 pools**: serverPool, ADMIN, WIFI, BOGOTA y **CALI**.

---

## Paso 9. Impresora y PCs

### IMP-CALI (estática)
1. **Config → Settings**: Gateway = `172.20.0.193`. DNS Server = `172.20.1.34`.
2. **Config → FastEthernet0**: **Static**, IP Address = `172.20.0.194`, Subnet Mask = `255.255.255.192`.

### PC-CALI1 y PC-CALI2 (DHCP)
**Desktop → IP Configuration → DHCP**. Debe aparecer **"DHCP request successful"** con una IP entre **172.20.0.200** y **172.20.0.239**.

> **Si falla, revisa en orden:**
> 1. R-MEDELLIN Serial0/3/1 en **up / up**.
> 2. La ruta hacia 172.20.0.192 en R-MEDELLIN (`show ip route`).
> 3. El helper en G0/0 de R-CALI.
> 4. El pool CALI con gateway `172.20.0.193`.
>
> Para reintentar: **Static** y luego otra vez **DHCP**.

---

## Paso 10. Pruebas de tu sede y capturas 📸

| # | Dónde | Comando | Resultado esperado | Nombre de la captura |
|---|---|---|---|---|
| 1 | PC-CALI1 | `ipconfig` | IP .0.200 – .0.239, gateway .0.193, DNS .1.34 | `E01-ipconfig-PC-CALI1.png` |
| 2 | PC-CALI1 | `ping` a la IP de PC-CALI2 | Responde (misma subred) | `E02-ping-misma-subred.png` |
| 3 | PC-CALI1 | `ping 172.20.0.194` | Responde (impresora) | `E03-ping-impresora.png` |
| 4 | PC-CALI1 | `ping 172.20.1.34` | Responde (servidor en Medellín) | `E04-ping-servidor.png` |
| 5 | PC-CALI1 | Web Browser → `www.cafemontana.com` | Se ve la página | `E05-web-PC-CALI1.png` |
| 6 | R-CALI | `show ip interface brief` | G0/0 y S0/0/0 up/up | `E06-show-ip-int-brief.png` |
| 7 | R-CALI | `show ip route` | `S* 0.0.0.0/0 via 172.20.1.53` | `E07-show-ip-route-CALI.png` |

---

## Paso 11. Pruebas finales de integración (red completa) 📸
Ya están las 3 sedes. Haz estas pruebas, idealmente **en llamada con Mauricio y Andrés** para que los 3 las vean.

| # | Desde | Comando | Prueba | Captura |
|---|---|---|---|---|
| F1 | PC-ADM1 | `ping` a PC-BOG1 y a PC-CALI1 | Central → cada remota | `F01-central-remotas.png` |
| F2 | PC-BOG1 | `ping` a PC-CALI1 | Remota 1 → remota 2 | `F02-bogota-cali.png` |
| F3 | PC-CALI1 | `tracert` a la IP de PC-BOG1 | Camino: R-CALI → R-MEDELLIN → R-BOGOTA | `F03-tracert-cali-bogota.png` |
| F4 | LAPTOP-1 | `ping` a PC-BOG1 y a PC-CALI1 | Wifi → sedes remotas | `F04-wifi-remotas.png` |
| F5 | LAPTOP-1 | Web Browser → `www.cafemontana.com` | Wifi → servidor web | `F05-web-laptop.png` |
| F6 | R-MEDELLIN | `show ip interface brief` | Las 5 interfaces up/up | `F06-MED-int-brief.png` |
| F7 | R-MEDELLIN | `show ip route` | C para sus 5 redes; S hacia .0.0/25 y .0.192/26 | `F07-MED-route.png` |
| F8 | SRV-DHCP-DNS | Services → DHCP (tabla de pools) | 5 pools configurados | `F08-pools-dhcp.png` |
| F9 | Vista general | Toda la topología | Todos los puntos verdes | `F09-topologia.png` |

> El TTL de un ping Bogotá → Cali debe ser **125** (128 − 3 routers). Es un buen dato para la sustentación.

Al terminar:
1. Guarda el archivo final como **`CafeMontana.pkt`**.
2. Envíalo a Mauricio y a Andrés, para que **los 3 tengan la misma versión final**.
3. Abre el archivo una vez más para comprobar que se abre bien: el profesor lo va a abrir.

---

## Tu parte del documento técnico: pruebas y cierre
Eres responsable de las secciones de **pruebas, evidencias, problemas, conclusiones y fuentes**.

1. **Evidencias de pruebas.** Organiza **todas** las capturas (M = Mauricio, A = Andrés, E = Edward, F = finales) en una tabla así:

   | # | Prueba | Desde → Hacia | Comando | Resultado | Captura |
   |---|---|---|---|---|---|
   | 1 | Misma subred | PC-ADM1 → PC-ADM2 | ping | Exitoso, 0 % de pérdida | (imagen) |

   Cubre los puntos que pide el enunciado: misma subred, subredes de la misma sede, central ↔ remotas, remota ↔ remota, acceso a servidores, DHCP y wifi.
2. **Problemas encontrados y soluciones.** Pide a Mauricio y Andrés que te cuenten qué les falló y cómo lo arreglaron. **El enunciado lo exige.** Ejemplos típicos:
   - "El serial quedó en down porque el router estaba apagado después de poner el módulo → se encendió."
   - "DHCP request failed en Bogotá porque faltaba la ruta en R-MEDELLIN → se agregó."
   - "El primer ping se perdió por la resolución ARP; los siguientes respondieron."
3. **Conclusiones** (3 a 5), por ejemplo:
   - VLSM permitió ajustar cada subred a su necesidad sin desperdiciar direcciones.
   - Centralizar DHCP, DNS y web en Medellín simplifica la administración; `ip helper-address` lo hace posible.
   - La topología en estrella es económica, pero depende de R-MEDELLIN (punto único de falla).
4. **Fuentes.** Citen lo que usaron, por ejemplo: Cisco Networking Academy (CCNA: Introduction to Networks), la documentación de Cisco Packet Tracer y el RFC 1918 (direcciones privadas).

## Lo que debes saber explicar en la sustentación
- Cómo viaja un paquete de Cali a Bogotá: ruta por defecto de R-CALI → ruta estática de R-MEDELLIN → R-BOGOTA.
- Qué muestra cada comando de prueba: `ping`, `tracert`, `ipconfig`, `show ip interface brief` y `show ip route`.
- Qué significan las letras de `show ip route`: **C** = conectada, **L** = local (la IP propia del router), **S** = estática, **S\*** = ruta por defecto.
