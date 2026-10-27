# Guía de Andrés – Sede remota 1 (Bogotá)

**Tu turno empieza cuando Mauricio te envía `CafeMontana.pkt`.** Agregas la sede Bogotá, la conectas a Medellín por el enlace **WAN1**, la pruebas y le envías el archivo a **Edward**.

| Red | Dirección | Gateway (R-BOGOTA) |
|---|---|---|
| LAN usuarios Bogotá | 172.20.0.0/25 (255.255.255.128) | 172.20.0.1 |
| WAN1 Medellín–Bogotá | 172.20.1.48/30 (255.255.255.252) | R-MEDELLIN .49 / R-BOGOTA .50 |

**Mientras esperas el archivo**, avanza con tu parte del documento (al final de esta guía).

---

## Paso 1. Abrir y respaldar
1. Abre `CafeMontana.pkt`.
2. Ve a **File → Save As** → `CafeMontana-v2-andres.pkt`. Esta será tu copia de trabajo.
3. **No modifiques nada de Medellín**, excepto lo que dice el Paso 6.

---

## Paso 2. Poner los dispositivos
Ubícalos **abajo a la izquierda** de R-MEDELLIN. Para ponerles nombre: clic → **Config** → **Display Name**.

| Cantidad | Categoría → Dispositivo | Nombre |
|---|---|---|
| 1 | Network Devices → Routers → **1941** | `R-BOGOTA` |
| 1 | Network Devices → Switches → **2960** | `SW-BOG` |
| 2 | End Devices → **PC** | `PC-BOG1`, `PC-BOG2` |
| 1 | End Devices → **Printer** | `IMP-BOG` |

> 💡 Encierra tu sede en un rectángulo (**Draw Rectangle**) y ponle la nota "SEDE REMOTA 1 – BOGOTÁ" (**Place Note**).

---

## Paso 3. Módulo serial en R-BOGOTA
1. Clic en **R-BOGOTA** → pestaña **Physical**.
2. **Apágalo** con el botón de encendido.
3. Arrastra un **HWIC-2T** a una ranura vacía.
4. **Vuelve a encenderlo.** ⚠️ Si lo olvidas, el enlace WAN nunca sube.

---

## Paso 4. Cables

### Cable serial WAN1
1. Selecciona el cable **Serial DCE** (rojo, con un relojito).
2. **Primer clic en R-MEDELLIN** → elige **Serial0/0/0**.
3. Segundo clic en **R-BOGOTA** → elige **Serial0/0/0**.

> Empezar por R-MEDELLIN deja ese lado como **DCE**, que es el que lleva el `clock rate`.

### Cables de cobre (Copper Straight-Through)
| Desde | Puerto | Hacia | Puerto |
|---|---|---|---|
| R-BOGOTA | **GigabitEthernet0/0** | SW-BOG | GigabitEthernet0/1 |
| PC-BOG1 | FastEthernet0 | SW-BOG | FastEthernet0/1 |
| PC-BOG2 | FastEthernet0 | SW-BOG | FastEthernet0/2 |
| IMP-BOG | FastEthernet0 | SW-BOG | FastEthernet0/3 |

Pasa el mouse sobre el cable serial para confirmar que llega a **Serial0/0/0** en los dos routers.

---

## Paso 5. Configurar R-BOGOTA
**CLI** → responde `no` a la pregunta inicial → pega:

```
enable
configure terminal
hostname R-BOGOTA
no ip domain-lookup
banner motd #Cafe Montana - Router Sede Bogota - Solo personal autorizado#

interface GigabitEthernet0/0
 description LAN Usuarios Bogota
 ip address 172.20.0.1 255.255.255.128
 ip helper-address 172.20.1.34
 no shutdown
 exit

interface Serial0/0/0
 description WAN1 hacia Medellin
 ip address 172.20.1.50 255.255.255.252
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 172.20.1.49

end
write
```

**Qué hace cada cosa:**
- `ip helper-address 172.20.1.34`: las PCs piden IP con un **broadcast**, y los routers no dejan pasar broadcasts. Este comando toma la solicitud y la reenvía al servidor DHCP de Medellín.
- `ip route 0.0.0.0 0.0.0.0 172.20.1.49` (**ruta por defecto**): Bogotá tiene una sola salida, así que todo lo que no sea su propia red lo envía a R-MEDELLIN.

---

## Paso 6. Agregar lo tuyo en R-MEDELLIN
R-MEDELLIN necesita dos cosas para el enlace con Bogotá: el **reloj del serial** y la **ruta hacia tu LAN**. En la **CLI de R-MEDELLIN**:

```
enable
configure terminal
interface Serial0/0/0
 clock rate 64000
 exit
ip route 172.20.0.0 255.255.255.128 172.20.1.50
end
write
```
- `clock rate 64000`: da el reloj del enlace serial, en el lado DCE.
- `ip route ...`: "para llegar a la LAN de Bogotá (172.20.0.0/25), envía los paquetes a R-BOGOTA (172.20.1.50)".

> Si al poner el `clock rate` sale un error que dice que solo aplica a DCE, el cable quedó al revés. Bórralo y conéctalo otra vez empezando por R-MEDELLIN.

Verifica en R-MEDELLIN:
```
show ip interface brief
```
**Serial0/0/0** debe estar en **up / up**.

---

## Paso 7. Configurar SW-BOG
```
enable
configure terminal
hostname SW-BOG
no ip domain-lookup
banner motd #Cafe Montana - Switch Sede Bogota#
interface GigabitEthernet0/1
 description Enlace a R-BOGOTA G0/0
interface FastEthernet0/1
 description PC-BOG1
interface FastEthernet0/2
 description PC-BOG2
interface FastEthernet0/3
 description IMP-BOG
end
write
```

---

## Paso 8. Agregar el pool BOGOTA en el servidor DHCP
1. Clic en **SRV-DHCP-DNS** → **Services** → **DHCP**.
2. Escribe en los campos:
   - Pool Name: `BOGOTA`
   - Default Gateway: `172.20.0.1`
   - DNS Server: `172.20.1.34`
   - Start IP Address: `172.20.0.10`
   - Subnet Mask: `255.255.255.128`
   - Maximum Number of Users: `100`
3. Haz clic en **Add** (no en Save: Save sobrescribe el pool que esté seleccionado).

En la tabla deben quedar 4 pools: serverPool, ADMIN, WIFI y **BOGOTA**.

> El servidor decide qué pool usar según el **Default Gateway**. Si escribes mal `172.20.0.1`, las PCs de Bogotá no reciben IP.

---

## Paso 9. Impresora y PCs

### IMP-BOG (estática)
1. **Config → Settings**: Gateway = `172.20.0.1`. DNS Server = `172.20.1.34`.
2. **Config → FastEthernet0**: **Static**, IP Address = `172.20.0.2`, Subnet Mask = `255.255.255.128`.

### PC-BOG1 y PC-BOG2 (DHCP)
**Desktop → IP Configuration → DHCP**. Debe aparecer **"DHCP request successful"** con una IP entre **172.20.0.10** y **172.20.0.109**.

> **Si dice "DHCP request failed", revisa en orden:**
> 1. R-MEDELLIN Serial0/0/0 en **up / up**.
> 2. La ruta `ip route 172.20.0.0 255.255.255.128 172.20.1.50` en R-MEDELLIN (compruébalo con `show ip route`).
> 3. El `ip helper-address 172.20.1.34` en G0/0 de R-BOGOTA.
> 4. El pool BOGOTA con gateway `172.20.0.1`.
>
> Para reintentar: **Static** y luego otra vez **DHCP**.

---

## Paso 10. Pruebas de tu sede y capturas 📸

| # | Dónde | Comando | Resultado esperado | Nombre de la captura |
|---|---|---|---|---|
| 1 | PC-BOG1 | `ipconfig` | IP .0.10 – .0.109, gateway .0.1, DNS .1.34 | `A01-ipconfig-PC-BOG1.png` |
| 2 | PC-BOG1 | `ping` a la IP de PC-BOG2 | Responde (misma subred) | `A02-ping-misma-subred.png` |
| 3 | PC-BOG1 | `ping 172.20.0.2` | Responde (impresora) | `A03-ping-impresora.png` |
| 4 | PC-BOG1 | `ping 172.20.1.34` | Responde (servidor en Medellín) | `A04-ping-servidor.png` |
| 5 | PC-BOG1 | `ping` a la IP de PC-ADM1 | Responde (Bogotá → Medellín) | `A05-ping-medellin.png` |
| 6 | PC-BOG1 | Web Browser → `www.cafemontana.com` | Se ve la página | `A06-web-PC-BOG1.png` |
| 7 | R-BOGOTA | `show ip interface brief` | G0/0 y S0/0/0 up/up | `A07-show-ip-int-brief.png` |
| 8 | R-BOGOTA | `show ip route` | `S* 0.0.0.0/0 via 172.20.1.49` | `A08-show-ip-route-BOG.png` |
| 9 | R-MEDELLIN | `show ip route` | `S 172.20.0.0/25 via 172.20.1.50` | `A09-show-ip-route-MED.png` |

Envía todas las capturas a Edward.

---

## Paso 11. Entregar el turno a Edward
1. Guarda con **Ctrl + S**, y guarda también como `CafeMontana.pkt`.
2. Envía `CafeMontana.pkt` a **Edward**.

---

## Tu parte del documento técnico: plan de direccionamiento
Eres el responsable de las secciones 5, 6 y 7 del enunciado. Usa las tablas del plan general (`00-PLAN-GENERAL.md`).

1. **Selección del bloque:** por qué 172.20.0.0/23. Es privado, del rango 172.16.0.0/12. Suma de direcciones: 128 + 64 + 64 + 32 + 16 + 4 + 4 = **312**. Un /24 (256) no alcanza; un /23 (512) sí, y deja espacio para crecer.
2. **Cálculo VLSM paso a paso.** Para cada red, muestra este razonamiento:
   - Hosts necesarios + 1 (gateway).
   - La potencia de 2 que alcanza: 2ⁿ − 2 ≥ hosts.
   - Prefijo = 32 − n.
   - Ejemplo Bogotá: 100 + 1 = 101 → 2⁷ − 2 = 126 ≥ 101 → n = 7 → **/25** → máscara 255.255.255.128.
   - Orden de asignación: de la más grande a la más pequeña (Bogotá /25 → Admin /26 → Cali /26 → Wifi /27 → Servidores /28 → WAN /30 → WAN /30).
3. **Tabla general de subneteo:** red, máscara, rango, gateway y broadcast.
4. **Tabla detallada de dispositivos:** al final, pide a Mauricio y Edward las IPs que recibieron sus PCs por DHCP (las de `ipconfig`) y reemplaza los "DHCP (rango…)" por la IP real.
5. **Verificación de no solapamiento:** cada red empieza justo después del broadcast de la anterior.

## Lo que debes saber explicar en la sustentación
- El cálculo VLSM de **cualquier** red, no solo Bogotá.
- Por qué los WAN son /30.
- Qué hacen la ruta por defecto de R-BOGOTA y la ruta estática que agregaste en R-MEDELLIN.
- Cómo una PC de Bogotá recibe IP de un servidor que está en Medellín (helper).
