## **Capítulo 8 – Open Shortest Path First (OSPF)**

### **1. Introducción**

**OSPF (Open Shortest Path First)** es un protocolo **IGP (Interior Gateway Protocol)** de tipo **link-state**, desarrollado por el IETF y diseñado para redes IP.  
Utiliza el algoritmo **Shortest Path First (SPF)** de **Dijkstra** para calcular la ruta más corta a cada destino.

**Versiones:**

- **OSPFv2 (RFC 2328)** → IPv4
    
- **OSPFv3 (RFC 5340)** → IPv6 (y ahora también IPv4, con Address Families)
    

**Características principales:**

- Convergencia rápida
    
- Soporte de **VLSM** y **CIDR**
    
- Actualizaciones disparadas (no periódicas)
    
- Autenticación integrada (plaintext, MD5, SHA)
    
- Soporte multi-área (jerarquía de áreas)
    

---

## **2. Funcionamiento básico**

1. Cada router genera un **Link-State Advertisement (LSA)** describiendo sus enlaces.
    
2. Los LSAs se **inundan (flood)** a todos los routers dentro del área.
    
3. Cada router construye su propia **Link-State Database (LSDB)**.
    
4. Con la LSDB, ejecuta el **algoritmo SPF** para calcular las rutas más cortas.
    
5. Los resultados se instalan en la **Routing Information Base (RIB)**.
    

**Actualizaciones periódicas:**  
Cada LSA se vuelve a enviar cada 30 minutos (paranoid update) y expira a los 60 minutos si no se renueva.

---

## **3. Costo (Métrica OSPF)**

El **costo** es inversamente proporcional al ancho de banda del enlace.

**Fórmula:**

![[Pasted image 20251106191110.png]]

**Valores predeterminados:**

|Interfaz|Costo OSPF|
|---|---|
|T1|64|
|Ethernet (10 Mbps)|10|
|FastEthernet (100 Mbps)|1|
|Gigabit / 10 Gb|1 (mismo costo por defecto, por eso se ajusta manualmente)|

**Comando de ajuste:**

```
interface Gi0/0
 ip ospf cost 5
```

---

## **4. Autenticación OSPF**

OSPF soporta autenticación **plaintext**, **MD5** y **SHA/HMAC**.

**Ejemplo MD5:**

```
interface Gi0/0
 ip ospf message-digest-key 1 md5 CCNP2025
 ip ospf authentication message-digest
```

**Autenticación HMAC-SHA con key-chain:**

```
key chain OSPF_KEYS
 key 1
  key-string StrongKey!
  cryptographic-algorithm hmac-sha-256
!
interface Gi0/0
 ip ospf authentication key-chain OSPF_KEYS
```

Se puede aplicar también a nivel de área:

```
area 0 authentication message-digest
```

---

## **5. Áreas OSPF**

OSPF divide el dominio de enrutamiento en **áreas lógicas** para mejorar la escalabilidad.

**Recomendaciones:**

- El **Área 0** es el **backbone** central obligatorio.
    
- Todas las demás áreas deben conectarse directamente al Área 0.
    
- Si no hay conexión física, puede configurarse un **Virtual Link** temporal.
    

**Tipos de áreas:**

|Tipo|Descripción|
|---|---|
|**Normal**|Acepta todas las LSAs (1–5,7).|
|**Stub**|Bloquea LSAs externas (tipo 5); usa una ruta por defecto.|
|**Totally Stubby**|Bloquea LSAs tipo 3 y 5.|
|**NSSA (Not-So-Stubby)**|Permite redistribuir rutas externas (tipo 7).|

**Virtual Link (último recurso):**

```
area 113 virtual-link 1.1.1.1
```

---

## **6. Vecinos y Adyacencias**

Routers OSPF en el mismo segmento se convierten en **vecinos** usando el multicast **224.0.0.5** (AllSPFRouters).  
La relación se vuelve **adyacente** una vez que sincronizan sus LSDBs.

**Estados OSPF:**

1. **Down** – Sin paquetes recibidos.
    
2. **Init** – Recibido Hello, pero sin bidireccionalidad.
    
3. **2-Way** – Visto su propio Router ID (elección de DR/BDR).
    
4. **ExStart** – Elección del maestro y número de secuencia.
    
5. **Exchange** – Intercambio de DBD (Database Description).
    
6. **Loading** – Solicitud de LSAs faltantes.
    
7. **Full** – Sincronización completa (adyacencia establecida).
    

**Elección DR/BDR:**

- El router con **prioridad más alta** o **Router ID mayor** gana.
    
- Se puede ajustar con:
    
```
ip ospf priority 200
```
    

---

## **7. Tipos de paquetes OSPF**

|Tipo|Nombre|Función|
|---|---|---|
|1|Hello|Descubre y mantiene vecinos|
|2|DBD (Database Description)|Resume contenido de LSDB|
|3|LSR (Link-State Request)|Solicita LSAs faltantes|
|4|LSU (Link-State Update)|Envía LSAs|
|5|LSACK|Reconoce LSAs recibidos|

---

## **8. Tipos de LSAs (OSPFv2)**

|Tipo|Nombre|Generado por|Alcance|
|---|---|---|---|
|**1**|Router LSA|Todos los routers|Dentro del área|
|**2**|Network LSA|Designated Router|Dentro del área|
|**3**|Summary LSA|ABR|Entre áreas|
|**4**|ASBR Summary|ABR|Entre áreas|
|**5**|AS External|ASBR|Todo el dominio|
|**7**|NSSA External|ASBR (NSSA)|Solo NSSA (convertido a tipo 5 en ABR)|

---

## **9. Selección de rutas OSPF**

**Prioridad de instalación en la RIB:**

1. Intra-area (`O`)
    
2. Inter-area (`O IA`)
    
3. Externa tipo 1 (`O E1`)
    
4. Externa tipo 2 (`O E2`)
    

- **E1:** Métrica = costo interno + costo externo.
    
- **E2:** Métrica = solo costo externo (predeterminado).
    

**Balanceo ECMP:** hasta 4 rutas por defecto.  
Puede ajustarse con:

```
maximum-paths 8
```

---

## **10. Resumen de rutas**

Para reducir LSAs y mejorar rendimiento, se usa **summarization**:

**Entre áreas (ABR):**

```
area 1 range 10.10.0.0 255.255.0.0
```

**Externas (ASBR):**

```
summary-address 172.16.0.0 255.255.0.0
```

---

## **11. OSPFv3 (IPv6 e IPv4)**

**Diferencias clave con OSPFv2:**

- Soporta **IPv4 e IPv6** (Address Families).
    
- Configuración **por interfaz**, no con `network`.
    
- Permite múltiples instancias por enlace.
    
- Usa direcciones **link-local** como origen.
    
- LSAs no contienen direcciones IP (solo prefijos).
    

**Multicast:**

- AllSPFRouters → `FF02::5`
    
- AllDRouters → `FF02::6`
    

**LSAs adicionales:**

- **Tipo 8:** Link LSA (prefijos de enlace local).
    
- **Tipo 9:** Intra-area prefix LSA (prefijos de router o red).
    

**Configuración básica:**

```
ipv6 unicast-routing
router ospfv3 10
 router-id 1.1.1.1
!
interface Gi0/0
 ospfv3 10 ipv6 area 0
```

---

## **12. Verificación**

```
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip protocols
```

**Ejemplo:**

```
O 10.1.1.0/24 [110/20] via 10.0.0.2, GigabitEthernet0/0
O IA 172.16.0.0/16 [110/30] via 192.168.1.1, GigabitEthernet0/1
```

---

## **13. Parámetros de examen clave**

|Concepto|Valor / Detalle|
|---|---|
|Protocolo IP|**89**|
|AD predeterminado|**110**|
|Hello timer|10 s|
|Dead timer|40 s|
|Multicast|224.0.0.5 / 224.0.0.6|
|DR/BDR elección|Basada en prioridad y Router ID|
|Costo fórmula|100 Mbps / BW|
|LSA externos|Tipo 5 y 7|
|Balanceo ECMP|4 rutas (por defecto)|