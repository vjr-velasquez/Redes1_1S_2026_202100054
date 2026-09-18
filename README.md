# Proyecto 1 – SmartCity Tech Park

**Curso:** Redes de Computadoras 1  
**Universidad:** Universidad de San Carlos de Guatemala – Facultad de Ingeniería  
**Carné:** 202100054  
**Proyecto:** SmartCity Tech Park  
**Archivo Packet Tracer:** `Proyecto1_202100054.pkt`

---

## 1. Descripción general

El proyecto consiste en el diseño e implementación de una red LAN corporativa para el complejo **SmartCity Tech Park**, utilizando una arquitectura jerárquica de Capa 2 en Cisco Packet Tracer.

La solución implementa:

- Segmentación mediante VLANs.
- Administración centralizada mediante VTP.
- Enlaces troncales 802.1Q.
- VLAN nativa distinta de VLAN 1.
- Redundancia mediante STP/PVST.
- Agregación de enlaces mediante EtherChannel con LACP.
- Segmento Legacy basado en Hub.
- Red inalámbrica para visitantes.
- Aislamiento entre VLANs.
- Redundancia física en I+D y Edificio Corporativo.

La topología utilizada corresponde a una **topología jerárquica en estrella extendida**, con enlaces redundantes en las áreas críticas.

---

## 2. Parámetros derivados del carné

Carné: **202100054**

- Último dígito: **4**
- Penúltimo dígito: **5**
- Dominio VTP: `Smart_5`
- Contraseña VTP: `proyecto12S2026`
- Protocolo EtherChannel: **LACP**
- Protocolo STP: **PVST**
- VLAN nativa: **94**
- Banner MOTD: `Acceso Restringido - TechPark_202100054`

---

## 3. Diseño de la topología

La red se divide en cuatro áreas principales:

1. **Centro de Datos**
2. **Centro de Investigación y Desarrollo (I+D)**
3. **Edificio Corporativo**
4. **Planta de Producción**

El switch `SW-CORE` funciona como núcleo de la red y como servidor VTP.

### 3.1 Centro de Datos

Dispositivos:

- `SW-CORE`
- `SW-SRV`
- `SRV1`
- `SRV2`
- `SRV3`
- `SRV4`

`SW-CORE` y `SW-SRV` se encuentran unidos mediante dos enlaces físicos agrupados en un **EtherChannel LACP Po1**, permitiendo mayor disponibilidad y evitando que la granja de servidores dependa de un único enlace físico.

### 3.2 Centro de I+D

Dispositivos:

- `SW-ID-DIST`
- `SW-ID-A`
- `SW-ID-B`
- `ID-PC1` a `ID-PC8`

Los tres switches forman una topología triangular para garantizar redundancia. `SW-ID-DIST` se conecta al Core mediante dos enlaces físicos agrupados en **EtherChannel LACP Po2**.

### 3.3 Edificio Corporativo

Dispositivos:

- `SW-CORP-DIST`
- `SW-CORP-A`
- `SW-CORP-B`
- `SW-GUEST`
- `GER-PC1` a `GER-PC4`
- `AP-VISITANTES`
- `GUEST-LAP1`
- `GUEST-LAP2`

`SW-CORP-A` y `SW-CORP-B` tienen un enlace redundante entre sí, además de sus respectivos enlaces hacia `SW-CORP-DIST`.

La red de visitantes se conecta mediante `SW-GUEST`, configurado en **VTP Transparent**, y un punto de acceso inalámbrico.

### 3.4 Planta de Producción

Dispositivos:

- `SW-PROD-DIST`
- `SW-PROD-ACC`
- `HUB-LEGACY`
- `LEGACY1` a `LEGACY4`

El segmento Legacy utiliza un **Hub**, lo cual crea un dominio de colisión compartido entre todos los dispositivos conectados a dicho Hub.

---

## 4. Inventario de dispositivos

| Área | Dispositivo | Cantidad |
|---|---|---:|
| Centro de Datos | Switch Cisco 2960-24TT | 2 |
| Centro de Datos | Server-PT | 4 |
| I+D | Switch Cisco 2960-24TT | 3 |
| I+D | PC-PT | 8 |
| Corporativo | Switch Cisco 2960-24TT | 4 |
| Corporativo | PC-PT | 4 |
| Corporativo | Access Point | 1 |
| Corporativo | Laptop-PT | 2 |
| Producción | Switch Cisco 2960-24TT | 2 |
| Producción | Hub-PT | 1 |
| Producción | PC-PT Legacy | 4 |

**Total de switches:** 11

---

## 5. Tabla de VLANs

| VLAN ID | Nombre | Ubicación / Uso |
|---:|---|---|
| 14 | GERENCIA | Edificio Corporativo |
| 24 | INVESTIGACION | Centro de I+D |
| 34 | PRODUCCION | Planta de Producción |
| 44 | SERVIDORES | Centro de Datos |
| 54 | VISITANTES | Edificio Corporativo / Wi-Fi |
| 94 | NATIVA | VLAN nativa de enlaces trunk |

---

## 6. VTP

### 6.1 Configuración general

- Dominio: `Smart_5`
- Contraseña: `proyecto12S2026`
- Versión utilizada en Packet Tracer: VTP v1

### 6.2 Roles

| Switch | Modo VTP |
|---|---|
| SW-CORE | Server |
| SW-ID-DIST | Client |
| SW-ID-A | Client |
| SW-ID-B | Client |
| SW-CORP-DIST | Client |
| SW-CORP-A | Client |
| SW-CORP-B | Client |
| SW-PROD-DIST | Client |
| SW-PROD-ACC | Client |
| SW-SRV | Client |
| SW-GUEST | Transparent |

### 6.3 Justificación del servidor VTP

`SW-CORE` fue seleccionado como servidor VTP debido a que se encuentra en el núcleo de la red y centraliza la administración lógica del campus. Desde este switch se crean y nombran las VLANs que son propagadas a los switches clientes mediante enlaces troncales.

`SW-GUEST` se configuró como **VTP Transparent** para mantener aislada la administración de VLANs del segmento de visitantes.

### 6.4 Evidencia recomendada

Insertar captura de:

```bash
show vtp status
```

En `SW-CORE` se verificó:

- VTP Domain Name: `Smart_5`
- VTP Operating Mode: `Server`
- Configuration Revision: `13`

---

## 7. Enlaces troncales

Todos los trunks utilizan 802.1Q y VLAN nativa 94.

Las VLANs permitidas en los enlaces principales son:

```text
14,24,34,44,54,94
```

En el enlace hacia visitantes solo se permiten:

```text
54,94
```

### Evidencia recomendada

```bash
show interfaces trunk
```

En `SW-CORE` se validó:

- `Po1` trunk, nativa 94
- `Po2` trunk, nativa 94
- `Fa0/5` trunk, nativa 94
- `Fa0/6` trunk, nativa 94

---

## 8. EtherChannel

Debido a que el carné termina en número par, se utilizó **LACP**.

### 8.1 Port-Channel 1

Ruta:

```text
SW-CORE <== Po1 / LACP ==> SW-SRV
```

Puertos:

```text
SW-CORE Fa0/1 <-> SW-SRV Fa0/1
SW-CORE Fa0/2 <-> SW-SRV Fa0/2
```

Objetivo:

- Redundancia para la granja de servidores.
- Mayor capacidad agregada.
- Mantener conectividad ante la falla de uno de los enlaces físicos.

### 8.2 Port-Channel 2

Ruta:

```text
SW-CORE <== Po2 / LACP ==> SW-ID-DIST
```

Puertos:

```text
SW-CORE Fa0/3 <-> SW-ID-DIST Fa0/1
SW-CORE Fa0/4 <-> SW-ID-DIST Fa0/2
```

Objetivo:

- Incrementar la capacidad hacia I+D.
- Proveer redundancia en un área crítica.

### 8.3 Evidencia

```bash
show etherchannel summary
```

Resultado validado en `SW-CORE`:

```text
1  Po1(SU)  LACP  Fa0/1(P) Fa0/2(P)
2  Po2(SU)  LACP  Fa0/3(P) Fa0/4(P)
```

### 8.4 Prueba de tolerancia a fallos

Se deshabilitó temporalmente `Fa0/1` del `SW-CORE`:

```bash
interface fa0/1
shutdown
```

El resultado fue:

```text
Po1(SU) LACP Fa0/1(D) Fa0/2(P)
```

Esto demuestra que el Port-Channel permaneció operativo mediante `Fa0/2`.

Luego se restauró el puerto:

```bash
interface fa0/1
no shutdown
```

---

## 9. Spanning Tree Protocol

El proyecto utiliza **PVST**, debido a que el último dígito del carné es par.

### 9.1 Root Bridge por VLAN

| VLAN | Nombre | Root Bridge |
|---:|---|---|
| 14 | GERENCIA | SW-CORP-DIST |
| 24 | INVESTIGACION | SW-ID-DIST |
| 34 | PRODUCCION | SW-PROD-DIST |
| 44 | SERVIDORES | SW-CORE |
| 54 | VISITANTES | SW-CORP-DIST |
| 94 | NATIVA | SW-CORE |

### 9.2 Justificación

Los Root Bridge se ubicaron cerca del punto principal de concentración de tráfico de cada VLAN:

- I+D utiliza `SW-ID-DIST` como raíz para mantener una ruta lógica óptima dentro del centro de investigación.
- Gerencia y Visitantes utilizan `SW-CORP-DIST` por ser el switch de distribución principal del edificio corporativo.
- Producción utiliza `SW-PROD-DIST` como punto central de dicha área.
- Servidores y VLAN nativa utilizan `SW-CORE` por ser el núcleo del campus.

### 9.3 Comandos utilizados

Ejemplo:

```bash
spanning-tree mode pvst
spanning-tree vlan 24 root primary
```

### 9.4 Evidencia

```bash
show spanning-tree vlan 24
show spanning-tree vlan 14
show spanning-tree vlan 34
show spanning-tree vlan 44
show spanning-tree vlan 54
show spanning-tree vlan 94
```

En las VLANs configuradas se verificó la línea:

```text
This bridge is the root
```

---

## 10. Tabla de asignación de puertos

### 10.1 SW-CORE

| Puerto | Destino | Modo |
|---|---|---|
| Fa0/1 | SW-SRV Fa0/1 | Trunk / Po1 |
| Fa0/2 | SW-SRV Fa0/2 | Trunk / Po1 |
| Fa0/3 | SW-ID-DIST Fa0/1 | Trunk / Po2 |
| Fa0/4 | SW-ID-DIST Fa0/2 | Trunk / Po2 |
| Fa0/5 | SW-CORP-DIST Fa0/1 | Trunk |
| Fa0/6 | SW-PROD-DIST Fa0/1 | Trunk |

### 10.2 SW-SRV

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-CORE Fa0/1 | Trunk / Po1 |
| Fa0/2 | SW-CORE Fa0/2 | Trunk / Po1 |
| Fa0/3 | SRV1 | Access VLAN 44 |
| Fa0/4 | SRV2 | Access VLAN 44 |
| Fa0/5 | SRV3 | Access VLAN 44 |
| Fa0/6 | SRV4 | Access VLAN 44 |

### 10.3 SW-ID-DIST

| Puerto | Destino | Modo |
|---|---|---|
| Fa0/1 | SW-CORE Fa0/3 | Trunk / Po2 |
| Fa0/2 | SW-CORE Fa0/4 | Trunk / Po2 |
| Fa0/3 | SW-ID-A Fa0/1 | Trunk |
| Fa0/4 | SW-ID-B Fa0/1 | Trunk |

### 10.4 SW-ID-A

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-ID-DIST Fa0/3 | Trunk |
| Fa0/2 | SW-ID-B Fa0/2 | Trunk |
| Fa0/3 | ID-PC1 | Access VLAN 24 |
| Fa0/4 | ID-PC2 | Access VLAN 24 |
| Fa0/5 | ID-PC3 | Access VLAN 24 |
| Fa0/6 | ID-PC4 | Access VLAN 24 |

### 10.5 SW-ID-B

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-ID-DIST Fa0/4 | Trunk |
| Fa0/2 | SW-ID-A Fa0/2 | Trunk |
| Fa0/3 | ID-PC5 | Access VLAN 24 |
| Fa0/4 | ID-PC6 | Access VLAN 24 |
| Fa0/5 | ID-PC7 | Access VLAN 24 |
| Fa0/6 | ID-PC8 | Access VLAN 24 |

### 10.6 SW-CORP-DIST

| Puerto | Destino | Modo |
|---|---|---|
| Fa0/1 | SW-CORE Fa0/5 | Trunk |
| Fa0/2 | SW-CORP-A Fa0/1 | Trunk |
| Fa0/3 | SW-CORP-B Fa0/1 | Trunk |
| Fa0/4 | SW-GUEST Fa0/1 | Trunk VLAN 54,94 |

### 10.7 SW-CORP-A

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-CORP-DIST Fa0/2 | Trunk |
| Fa0/2 | SW-CORP-B Fa0/2 | Trunk |
| Fa0/3 | GER-PC1 | Access VLAN 14 |
| Fa0/4 | GER-PC2 | Access VLAN 14 |

### 10.8 SW-CORP-B

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-CORP-DIST Fa0/3 | Trunk |
| Fa0/2 | SW-CORP-A Fa0/2 | Trunk |
| Fa0/3 | GER-PC3 | Access VLAN 14 |
| Fa0/4 | GER-PC4 | Access VLAN 14 |

### 10.9 SW-GUEST

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-CORP-DIST Fa0/4 | Trunk VLAN 54,94 |
| Fa0/2 | AP-VISITANTES | Access VLAN 54 |

### 10.10 SW-PROD-DIST

| Puerto | Destino | Modo |
|---|---|---|
| Fa0/1 | SW-CORE Fa0/6 | Trunk |
| Fa0/2 | SW-PROD-ACC Fa0/1 | Trunk |

### 10.11 SW-PROD-ACC

| Puerto | Destino | Modo / VLAN |
|---|---|---|
| Fa0/1 | SW-PROD-DIST Fa0/2 | Trunk |
| Fa0/2 | HUB-LEGACY Port 1 | Access VLAN 34 |

---

## 11. Direccionamiento IP de pruebas

No se implementó routing inter-VLAN, por lo que no se configuró gateway predeterminado en los hosts.

### VLAN 14 – GERENCIA

| Equipo | Dirección IP |
|---|---|
| GER-PC1 | 192.168.14.11/24 |
| GER-PC2 | 192.168.14.12/24 |
| GER-PC3 | 192.168.14.13/24 |
| GER-PC4 | 192.168.14.14/24 |

### VLAN 24 – INVESTIGACION

| Equipo | Dirección IP |
|---|---|
| ID-PC1 | 192.168.24.11/24 |
| ID-PC2 | 192.168.24.12/24 |
| ID-PC3 | 192.168.24.13/24 |
| ID-PC4 | 192.168.24.14/24 |
| ID-PC5 | 192.168.24.15/24 |
| ID-PC6 | 192.168.24.16/24 |
| ID-PC7 | 192.168.24.17/24 |
| ID-PC8 | 192.168.24.18/24 |

### VLAN 34 – PRODUCCION

| Equipo | Dirección IP |
|---|---|
| LEGACY1 | 192.168.34.11/24 |
| LEGACY2 | 192.168.34.12/24 |
| LEGACY3 | 192.168.34.13/24 |
| LEGACY4 | 192.168.34.14/24 |

### VLAN 44 – SERVIDORES

| Equipo | Dirección IP |
|---|---|
| SRV1 | 192.168.44.11/24 |
| SRV2 | 192.168.44.12/24 |
| SRV3 | 192.168.44.13/24 |
| SRV4 | 192.168.44.14/24 |

### VLAN 54 – VISITANTES

Las laptops fueron asociadas inalámbricamente al `AP-VISITANTES`. Las direcciones IP pueden asignarse manualmente dentro del segmento `192.168.54.0/24` si se requiere realizar pruebas adicionales.

---

## 12. Dominios de broadcast

Cada VLAN representa un dominio de broadcast independiente.

| VLAN | Nombre | Dominio de Broadcast |
|---:|---|---:|
| 14 | GERENCIA | 1 |
| 24 | INVESTIGACION | 1 |
| 34 | PRODUCCION | 1 |
| 44 | SERVIDORES | 1 |
| 54 | VISITANTES | 1 |

**Total de dominios de broadcast funcionales: 5**

La VLAN 94 se utiliza como VLAN nativa de los trunks y no representa una red de usuarios finales.

---

## 13. Dominios de colisión

En una red con switches, cada puerto activo del switch constituye un dominio de colisión independiente.

### 13.1 Segmentos con switches

Los enlaces entre switches y los puertos hacia dispositivos finales representan dominios independientes debido al funcionamiento de los switches en Capa 2.

### 13.2 Segmento Legacy

El `HUB-LEGACY` constituye un único dominio de colisión compartido para:

- `LEGACY1`
- `LEGACY2`
- `LEGACY3`
- `LEGACY4`
- enlace hacia `SW-PROD-ACC`

Esto ocurre porque un Hub trabaja en Capa 1 y repite todas las señales recibidas hacia todos sus puertos.

### 13.3 Impacto del Hub Legacy

El Hub puede generar:

- mayor probabilidad de colisiones;
- menor rendimiento bajo tráfico simultáneo;
- ancho de banda compartido;
- mayor latencia y retransmisiones.

La medida de contención aplicada consiste en conectar el Hub a un único puerto de `SW-PROD-ACC` configurado como **Access VLAN 34**, evitando que el dominio de colisión del Hub se extienda físicamente al resto de la red.

---

## 14. Medios de transmisión

### 14.1 Cableado utilizado en Packet Tracer

| Segmento | Medio utilizado |
|---|---|
| Switch ↔ Switch | Copper Cross-Over |
| Switch ↔ PC | Copper Straight-Through |
| Switch ↔ Server | Copper Straight-Through |
| Switch ↔ Hub | Copper Straight-Through |
| Switch ↔ Access Point | Copper Straight-Through |
| Laptop ↔ Access Point | Enlace inalámbrico |

### 14.2 Justificación

El cableado de cobre se utilizó en la simulación para enlaces internos de corta distancia y para mantener compatibilidad directa con los puertos disponibles en los switches 2960-24TT utilizados.

En una implementación física real, los enlaces de backbone entre edificios deberían preferentemente utilizar **fibra óptica**, debido a:

- mayor distancia soportada;
- mayor ancho de banda;
- inmunidad a interferencias electromagnéticas;
- menor atenuación;
- mejor escalabilidad para enlaces troncales.

Especialmente se recomienda fibra para:

```text
SW-CORE ↔ SW-ID-DIST
SW-CORE ↔ SW-CORP-DIST
SW-CORE ↔ SW-PROD-DIST
```

---

## 15. Seguridad básica

En los switches de distribución se configuró el banner MOTD:

```text
Acceso Restringido - TechPark_202100054
```

Switches configurados:

- `SW-ID-DIST`
- `SW-CORP-DIST`
- `SW-PROD-DIST`

Comando:

```bash
banner motd #Acceso Restringido - TechPark_202100054#
```

También se cambió la VLAN nativa de los enlaces troncales desde VLAN 1 hacia **VLAN 94**.

---

## 16. Pruebas de conectividad

### 16.1 I+D

Prueba:

```bash
ID-PC1 > ping 192.168.24.15
```

Resultado:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### 16.2 Gerencia

Prueba:

```bash
GER-PC1 > ping 192.168.14.14
```

Resultado:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### 16.3 Producción

Prueba:

```bash
LEGACY1 > ping 192.168.34.14
```

Resultado:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### 16.4 Servidores

Prueba:

```bash
SRV1 > ping 192.168.44.14
```

Resultado:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## 17. Pruebas de aislamiento entre VLANs

No se implementó routing inter-VLAN, por lo que las VLANs deben permanecer aisladas.

### 17.1 INVESTIGACION → GERENCIA

```bash
ID-PC1 > ping 192.168.14.11
```

Resultado:

```text
Sent = 4, Received = 0, Lost = 4 (100% loss)
```

### 17.2 PRODUCCION → SERVIDORES

```bash
LEGACY1 > ping 192.168.44.11
```

Resultado:

```text
Sent = 4, Received = 0, Lost = 4 (100% loss)
```

Esto confirma el aislamiento entre dominios de broadcast.

---

## 18. Pruebas de redundancia

### 18.1 I+D

Se deshabilitó temporalmente el enlace:

```text
SW-ID-DIST Fa0/3
```

Comando:

```bash
interface fa0/3
shutdown
```

Posteriormente se realizó:

```bash
ID-PC1 > ping 192.168.24.15
```

Resultado:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

La comunicación se mantuvo mediante la ruta alternativa entre `SW-ID-A` y `SW-ID-B`.

Se restauró el enlace con:

```bash
interface fa0/3
no shutdown
```

### 18.2 Corporativo

Se deshabilitó temporalmente el enlace entre `SW-CORP-DIST` y `SW-CORP-A`.

La comunicación entre `GER-PC1` y `GER-PC4` se mantuvo utilizando el enlace redundante `SW-CORP-A ↔ SW-CORP-B`.

Posteriormente el enlace fue restaurado con:

```bash
interface fa0/2
no shutdown
```

---

## 19. Lista de comandos principales utilizados

### VLANs

```bash
vlan 14
name GERENCIA
vlan 24
name INVESTIGACION
vlan 34
name PRODUCCION
vlan 44
name SERVIDORES
vlan 54
name VISITANTES
vlan 94
name NATIVA
```

### VTP Server

```bash
vtp domain Smart_5
vtp password proyecto12S2026
vtp mode server
```

### VTP Client

```bash
vtp domain Smart_5
vtp password proyecto12S2026
vtp mode client
```

### VTP Transparent

```bash
vtp domain Smart_5
vtp password proyecto12S2026
vtp mode transparent
```

### Trunk

```bash
switchport mode trunk
switchport trunk native vlan 94
switchport trunk allowed vlan 14,24,34,44,54,94
```

### Puerto Access

```bash
switchport mode access
switchport access vlan <ID>
spanning-tree portfast
```

### EtherChannel LACP

```bash
interface range fa0/1 - 2
channel-group 1 mode active
```

### STP / PVST

```bash
spanning-tree mode pvst
spanning-tree vlan <ID> root primary
```

### Banner

```bash
banner motd #Acceso Restringido - TechPark_202100054#
```

### Verificación

```bash
show vlan brief
show vtp status
show interfaces trunk
show etherchannel summary
show spanning-tree
show spanning-tree vlan <ID>
```

---

## 20. Evidencias a insertar

Agregar capturas en esta sección antes de la entrega final.

### 20.1 Topología completa

```markdown
![Topología completa](./img/topologia_completa.png)
```

### 20.2 Centro de Datos

```markdown
![Centro de Datos](./img/centro_datos.png)
```

### 20.3 I+D

```markdown
![Centro I+D](./img/id.png)
```

### 20.4 Corporativo

```markdown
![Corporativo](./img/corporativo.png)
```

### 20.5 Producción

```markdown
![Producción](./img/produccion.png)
```

### 20.6 VTP

```markdown
![VTP Server](./img/vtp_server.png)
```

### 20.7 EtherChannel

```markdown
![EtherChannel](./img/etherchannel.png)
```

### 20.8 Spanning Tree

```markdown
![Spanning Tree](./img/spanning_tree.png)
```

### 20.9 Trunks

```markdown
![Trunks](./img/trunks.png)
```

### 20.10 Pruebas de ping

```markdown
![Ping I+D](./img/ping_id.png)
![Ping Gerencia](./img/ping_gerencia.png)
![Ping Producción](./img/ping_produccion.png)
![Ping Servidores](./img/ping_servidores.png)
```

---

## 21. Presupuesto estimado

> **Nota:** Los siguientes valores deben considerarse referenciales y deben actualizarse con cotizaciones reales antes de la entrega si el catedrático solicita precios actuales exactos.

| Equipo / Material | Cantidad | Precio unitario estimado | Subtotal estimado |
|---|---:|---:|---:|
| Switch administrable 24 puertos equivalente a Cisco 2960 | 11 | Q2,500 | Q27,500 |
| Access Point empresarial | 1 | Q800 | Q800 |
| Hub Legacy / equipo equivalente | 1 | Q250 | Q250 |
| Módulos/transceptores para enlaces de fibra | 6 | Q500 | Q3,000 |
| Cable UTP Cat6 | 1 lote | Q1,500 | Q1,500 |
| Fibra óptica backbone | 1 lote | Q2,500 | Q2,500 |
| Patch cords y accesorios | 1 lote | Q1,000 | Q1,000 |

**Total estimado:** **Q36,550**

Este presupuesto no incluye servidores, PCs, laptops, racks, UPS, mano de obra ni canalización.

---

## 22. Conclusiones

1. La segmentación por VLAN permite separar los dominios de broadcast y aislar lógicamente los diferentes departamentos del campus.
2. VTP facilita la administración centralizada de VLANs desde el Core.
3. EtherChannel con LACP permite combinar múltiples enlaces físicos, incrementando disponibilidad y tolerancia a fallos.
4. PVST evita bucles de Capa 2 y permite definir un Root Bridge específico por VLAN.
5. La redundancia implementada en I+D y Corporativo permite mantener comunicación ante la pérdida de determinados enlaces físicos.
6. El segmento Legacy demuestra el comportamiento de un dominio de colisión compartido generado por un Hub.
7. La VLAN nativa 94 evita utilizar VLAN 1 como VLAN nativa en los enlaces troncales.
8. Las pruebas realizadas confirmaron conectividad dentro de cada VLAN y aislamiento entre VLANs diferentes.

---

## 23. Estructura recomendada del repositorio

```text
Redes1_1S_2026_202100054/
└── Proyecto 1/
    ├── Proyecto1_202100054.pkt
    ├── README.md
    └── img/
        ├── topologia_completa.png
        ├── centro_datos.png
        ├── id.png
        ├── corporativo.png
        ├── produccion.png
        ├── vtp_server.png
        ├── etherchannel.png
        ├── spanning_tree.png
        ├── trunks.png
        ├── ping_id.png
        ├── ping_gerencia.png
        ├── ping_produccion.png
        └── ping_servidores.png
```

---

## 24. Estado final

La topología implementada cumple los requerimientos principales de Capa 1 y Capa 2 definidos para SmartCity Tech Park:

- VLANs según carné.
- VTP centralizado.
- VLAN nativa 94.
- Trunks 802.1Q.
- EtherChannel LACP.
- PVST.
- Root Bridge por VLAN.
- Segmento Legacy con Hub.
- Red de visitantes inalámbrica.
- Redundancia en áreas críticas.
- Conectividad intra-VLAN.
- Aislamiento inter-VLAN.
- Evidencia de tolerancia a fallos.

