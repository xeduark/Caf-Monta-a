# Trabajo Final – Redes de Datos I
## Plan general del equipo – Café Montaña S.A.S.

**Integrantes y responsabilidades**

| Integrante | Sede en Packet Tracer | Parte del documento técnico |
|---|---|---|
| **Mauricio** | Sede central – Medellín (router central, servidores, wifi) | Empresa, requerimientos, topología y su justificación |
| **Andrés** | Sede remota 1 – Bogotá (+ enlace WAN1) | Plan de direccionamiento VLSM y tablas de direccionamiento |
| **Edward** | Sede remota 2 – Cali (+ enlace WAN2) | Pruebas, evidencias, problemas/soluciones, conclusiones y fuentes |

> Mauricio tiene la sede más pesada en Packet Tracer; por eso Andrés y Edward llevan las partes más largas del documento.

**Guías individuales**
- `01-MAURICIO-SEDE-CENTRAL.md`
- `02-ANDRES-SEDE-BOGOTA.md`
- `03-EDWARD-SEDE-CALI.md`

---

## 1. Reglas del equipo (léanlas todos)

1. **Todos con la misma versión de Packet Tracer.** Revisen en *Help → About*. Un archivo guardado en una versión nueva **no abre** en una versión vieja.
2. **Un solo archivo de red:** `CafeMontana.pkt`. Se trabaja **por turnos**, pasándolo por Drive o WhatsApp:

   ```
   Mauricio (Medellín) ──► Andrés (Bogotá) ──► Edward (Cali) ──► Pruebas finales (los 3)
   ```

   Packet Tracer no permite unir dos archivos `.pkt`. Si cada uno arma su sede en un archivo distinto, después habría que rehacerlo todo en uno solo.
3. **Cada vez que reciban el archivo**, guarden una copia antes de tocarlo: `CafeMontana-v1-mauricio.pkt`, `CafeMontana-v2-andres.pkt`, etc. Si algo se daña, se vuelve a la versión anterior.
4. **Usen exactamente los nombres e IPs de este documento.** El profesor revisa que la red coincida con las tablas.
5. **Cada uno toma sus capturas** (ver su guía) y se las envía a Edward, que arma la sección de pruebas.
6. **Mientras esperan su turno**, avancen en su parte del documento.

---

## 2. La empresa (resumen)

**Café Montaña S.A.S.** (empresa ficticia): tostadora y distribuidora de café.

| Sede | Ciudad | Función | Usuarios reales |
|---|---|---|---|
| Central | Medellín | Gerencia, contabilidad, TI y servidores | 50 administrativos + 20 inalámbricos |
| Remota 1 | Bogotá | Centro de distribución y ventas | 100 |
| Remota 2 | Cali | Oficina comercial | 40 |

En Packet Tracer se ponen **2 PCs por red** como muestra representativa. Las cantidades reales se usan para calcular las máscaras.

---

## 3. Topología (estrella)

```
                  ┌──────────────── SEDE CENTRAL - MEDELLÍN (Mauricio) ────────────────┐
                  │                                                                     │
  [PC-ADM1] [PC-ADM2] [IMP-MED]      [SRV-DHCP-DNS] [SRV-WEB]         ((AP-MED))        │
        \      |      /                     \       /                  )) ))            │
         SW-ADMIN                          SW-SERVIDORES         [LAPTOP-1] [LAPTOP-2]  │
            │ G0/0                              │ G0/1                │ G0/2            │
            └─────────────────────── R-MEDELLIN ┴─────────────────────┘                 │
                  │                 /           \                                       │
                  └────────────────/─────────────\──────────────────────────────────────┘
                       S0/0/0     /  WAN1   WAN2  \    S0/0/1
                                 /                 \
                     S0/0/0  R-BOGOTA            R-CALI  S0/0/0
                     (Andrés)   │ G0/0              │ G0/0   (Edward)
                             SW-BOG              SW-CALI
                            /   |   \            /   |   \
                    [PC-BOG1][PC-BOG2][IMP-BOG] [PC-CALI1][PC-CALI2][IMP-CALI]
```

---

## 4. Plan de direccionamiento VLSM

**Bloque elegido:** `172.20.0.0/23`, dentro del rango privado 172.16.0.0/12. Se necesitan 312 direcciones; un /24 (256) no alcanza, así que se usa un /23 (512). Las redes se asignan **de la más grande a la más pequeña**.

| Sede | Red | Hosts requeridos | Red/prefijo | Máscara | Rango utilizable | Gateway | Broadcast | Responsable |
|---|---|---|---|---|---|---|---|---|
| Bogotá | LAN usuarios | 100 | 172.20.0.0/25 | 255.255.255.128 | 172.20.0.1 – 172.20.0.126 | 172.20.0.1 | 172.20.0.127 | Andrés |
| Central | LAN administrativa | 50 | 172.20.0.128/26 | 255.255.255.192 | 172.20.0.129 – 172.20.0.190 | 172.20.0.129 | 172.20.0.191 | Mauricio |
| Cali | LAN usuarios | 40 | 172.20.0.192/26 | 255.255.255.192 | 172.20.0.193 – 172.20.0.254 | 172.20.0.193 | 172.20.0.255 | Edward |
| Central | LAN inalámbrica | 20 | 172.20.1.0/27 | 255.255.255.224 | 172.20.1.1 – 172.20.1.30 | 172.20.1.1 | 172.20.1.31 | Mauricio |
| Central | LAN servidores | 5 (crece a 10) | 172.20.1.32/28 | 255.255.255.240 | 172.20.1.33 – 172.20.1.46 | 172.20.1.33 | 172.20.1.47 | Mauricio |
| WAN 1 | Medellín–Bogotá | 2 | 172.20.1.48/30 | 255.255.255.252 | 172.20.1.49 – 172.20.1.50 | No aplica | 172.20.1.51 | Andrés |
| WAN 2 | Medellín–Cali | 2 | 172.20.1.52/30 | 255.255.255.252 | 172.20.1.53 – 172.20.1.54 | No aplica | 172.20.1.55 | Edward |
| — | Libre (crecimiento) | — | 172.20.1.56 – 172.20.1.255 | — | — | — | — | — |

### Interfaces WAN

| Enlace | R-MEDELLIN | IP | Router remoto | IP |
|---|---|---|---|---|
| WAN1 | Serial0/0/0 (DCE) | 172.20.1.49/30 | R-BOGOTA Serial0/0/0 | 172.20.1.50/30 |
| WAN2 | Serial0/0/1 (DCE) | 172.20.1.53/30 | R-CALI Serial0/0/0 | 172.20.1.54/30 |

### Pools DHCP (todos en SRV-DHCP-DNS, 172.20.1.34)

| Pool | Default Gateway | DNS | Start IP | Máscara | Máx. usuarios | Rango que entrega | Lo crea |
|---|---|---|---|---|---|---|---|
| ADMIN | 172.20.0.129 | 172.20.1.34 | 172.20.0.140 | 255.255.255.192 | 50 | .0.140 – .0.189 | Mauricio |
| WIFI | 172.20.1.1 | 172.20.1.34 | 172.20.1.10 | 255.255.255.224 | 20 | .1.10 – .1.29 | Mauricio |
| BOGOTA | 172.20.0.1 | 172.20.1.34 | 172.20.0.10 | 255.255.255.128 | 100 | .0.10 – .0.109 | Andrés |
| CALI | 172.20.0.193 | 172.20.1.34 | 172.20.0.200 | 255.255.255.192 | 40 | .0.200 – .0.239 | Edward |

---

## 5. Tabla detallada de dispositivos

Las IPs que dicen "DHCP" se completan al final con la IP real que muestre `ipconfig`.

| Sede | Dispositivo | Interfaz | IPv4 | Máscara | Gateway | Asignación |
|---|---|---|---|---|---|---|
| Central | R-MEDELLIN | G0/0 (LAN admin) | 172.20.0.129 | /26 | No aplica | Estática |
| Central | R-MEDELLIN | G0/1 (LAN servidores) | 172.20.1.33 | /28 | No aplica | Estática |
| Central | R-MEDELLIN | G0/2 (LAN inalámbrica) | 172.20.1.1 | /27 | No aplica | Estática |
| Central | R-MEDELLIN | S0/0/0 (WAN1) | 172.20.1.49 | /30 | No aplica | Estática |
| Central | R-MEDELLIN | S0/0/1 (WAN2) | 172.20.1.53 | /30 | No aplica | Estática |
| Central | SRV-DHCP-DNS | Fa0 | 172.20.1.34 | /28 | 172.20.1.33 | Estática |
| Central | SRV-WEB | Fa0 | 172.20.1.35 | /28 | 172.20.1.33 | Estática |
| Central | IMP-MED | Fa0 | 172.20.0.130 | /26 | 172.20.0.129 | Estática |
| Central | PC-ADM1 | Fa0 | DHCP (.0.140 – .0.189) | /26 | 172.20.0.129 | DHCP |
| Central | PC-ADM2 | Fa0 | DHCP (.0.140 – .0.189) | /26 | 172.20.0.129 | DHCP |
| Central | AP-MED | Port 0 | Sin IP (capa 2) * | — | — | — |
| Central | LAPTOP-1 | Wireless0 | DHCP (.1.10 – .1.29) | /27 | 172.20.1.1 | DHCP |
| Central | LAPTOP-2 | Wireless0 | DHCP (.1.10 – .1.29) | /27 | 172.20.1.1 | DHCP |
| Bogotá | R-BOGOTA | G0/0 (LAN) | 172.20.0.1 | /25 | No aplica | Estática |
| Bogotá | R-BOGOTA | S0/0/0 (WAN1) | 172.20.1.50 | /30 | No aplica | Estática |
| Bogotá | IMP-BOG | Fa0 | 172.20.0.2 | /25 | 172.20.0.1 | Estática |
| Bogotá | PC-BOG1 | Fa0 | DHCP (.0.10 – .0.109) | /25 | 172.20.0.1 | DHCP |
| Bogotá | PC-BOG2 | Fa0 | DHCP (.0.10 – .0.109) | /25 | 172.20.0.1 | DHCP |
| Cali | R-CALI | G0/0 (LAN) | 172.20.0.193 | /26 | No aplica | Estática |
| Cali | R-CALI | S0/0/0 (WAN2) | 172.20.1.54 | /30 | No aplica | Estática |
| Cali | IMP-CALI | Fa0 | 172.20.0.194 | /26 | 172.20.0.193 | Estática |
| Cali | PC-CALI1 | Fa0 | DHCP (.0.200 – .0.239) | /26 | 172.20.0.193 | DHCP |
| Cali | PC-CALI2 | Fa0 | DHCP (.0.200 – .0.239) | /26 | 172.20.0.193 | DHCP |

\* En Packet Tracer, el **AccessPoint-PT** trabaja en capa 2 (como un switch inalámbrico) y no permite asignarle IP. Si el profesor exige una IP para el AP, avisen para cambiar el diseño.

---

## 6. Pruebas finales (los 3 juntos, cuando Edward termine)

Abran el archivo final y ejecuten estas pruebas. Tomen captura de cada una.

| # | Desde | Comando | Prueba |
|---|---|---|---|
| 1 | PC-ADM1 | `ping` a PC-ADM2 | Misma subred |
| 2 | PC-ADM1 | `ping 172.20.1.34` | Otra subred de la misma sede (servidores) |
| 3 | LAPTOP-1 | `ping` a PC-ADM1 | Inalámbrico → cableado |
| 4 | PC-ADM1 | `ping` a PC-BOG1 y a PC-CALI1 | Central → remotas |
| 5 | PC-BOG1 | `ping` a PC-CALI1 | Remota 1 → remota 2 (pasa por Medellín) |
| 6 | PC-CALI1 | `tracert` a PC-BOG1 | Muestra el camino por R-MEDELLIN |
| 7 | PC-BOG1, PC-CALI1, LAPTOP-1 | Web Browser → `www.cafemontana.com` | Acceso a servidores (DNS + web) |
| 8 | Todas las PCs y portátiles | `ipconfig` | DHCP correcto |
| 9 | Los 3 routers | `show ip interface brief` y `show ip route` | Estado y rutas |

---

## 7. Sustentación: preguntas probables

Cada uno explica su sede, pero **todos deben poder responder esto**:

| Pregunta | Respuesta corta |
|---|---|
| ¿Por qué topología en estrella? | Medellín concentra los servicios. Es económica (2 enlaces WAN) y fácil de administrar. Desventaja: si cae R-MEDELLIN, las sedes quedan aisladas. |
| ¿Por qué 172.20.0.0/23? | Es privado (172.16.0.0/12). Se necesitan 312 direcciones: un /24 no alcanza y un /23 da 512, con espacio para crecer. |
| ¿Por qué Bogotá es /25? | Necesita 100 hosts + gateway = 101. Un /25 da 126 útiles; un /26 (62) no alcanza. |
| ¿Por qué los WAN son /30? | Solo hay 2 routers por enlace. Un /30 da exactamente 2 IPs útiles y no desperdicia direcciones. |
| ¿Qué es el gateway? | La IP del router en cada LAN. Las PCs envían ahí todo lo que va para otra red. |
| ¿Cómo llegan Bogotá y Cali a las otras redes? | Con una ruta por defecto (`0.0.0.0 0.0.0.0`) hacia R-MEDELLIN. R-MEDELLIN tiene rutas estáticas hacia cada LAN remota. |
| ¿Cómo reciben IP las PCs de Bogotá si el DHCP está en Medellín? | Con `ip helper-address 172.20.1.34` en la G0/0 de R-BOGOTA, que reenvía la solicitud DHCP (broadcast) al servidor. |
| ¿Por qué R-MEDELLIN también tiene helper? | El servidor está en la LAN de servidores, **no** en la administrativa ni en la inalámbrica. Sin helper, esas PCs no recibirían IP. |
| ¿Por qué servidores e impresoras son estáticos? | Los usuarios necesitan encontrarlos siempre en la misma IP. |
| ¿Qué hace el `clock rate`? | Da la velocidad (reloj) del enlace serial. Va en el lado DCE, que es R-MEDELLIN. |
| ¿Qué es el TTL en el ping? | Baja 1 por cada router atravesado. De Bogotá a Cali llega con 125 (3 routers). |
