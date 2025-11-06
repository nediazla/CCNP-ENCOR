## **Capítulo 6 – IP Routing Essentials**

### **1. Introducción**

El enrutamiento IP (Internet Protocol Routing) es el proceso mediante el cual un router mueve paquetes de una red a otra.  
Cuando un destino no está directamente conectado, el router debe conocer una ruta hacia él, ya sea mediante **rutas estáticas** o **protocolos dinámicos de enrutamiento**.

Los protocolos dinámicos ajustan sus tablas de enrutamiento automáticamente frente a cambios en la red, mientras que las rutas estáticas deben modificarse manualmente.

Protocolos comunes:

- **RIPv2** – vector-distancia clásico
    
- **EIGRP** – vector-distancia avanzado
    
- **OSPF** – link-state
    
- **IS-IS** – link-state
    
- **BGP** – protocolo exterior (EGP)
    

---

## **2. Algoritmos de enrutamiento**

|Tipo|Descripción|Ejemplo|
|---|---|---|
|**Distance Vector**|Envía actualizaciones periódicas solo a vecinos directos. Conocen la red “a través de otros”. Escalable de forma limitada.|RIP|
|**Advanced Distance Vector**|Usa _DUAL (Diffusing Update Algorithm)_. Forma adyacencias, soporta _equal_ y _unequal load balancing_, y converge rápidamente.|EIGRP|
|**Link-State**|Cada router crea una base de datos topológica completa. Usa SPF (Dijkstra). Escalable, preciso y con convergencia rápida.|OSPF, IS-IS|
|**Path-Vector**|Usa políticas y atributos de ruta entre sistemas autónomos (AS).|BGP|

---

## **3. Selección de rutas**

Los routers pueden aprender múltiples rutas a un mismo destino. Para decidir cuál usar, aplican tres criterios:

### **a) Longest Prefix Match**

La coincidencia más específica (más bits de máscara) tiene prioridad.  
Ejemplo:

- 10.0.5.0/24
    
- 10.0.5.0/26
    
- 10.0.5.0/28 ← _Gana la coincidencia más larga._
    

### **b) Administrative Distance (AD)**

Valor numérico que indica la “confiabilidad” de una fuente de enrutamiento.

|Fuente de ruta|AD predeterminado|
|---|---|
|Conectada directamente|**0**|
|Estática|**1**|
|EIGRP resumen|5|
|BGP externo (eBGP)|20|
|EIGRP interno|90|
|OSPF|110|
|IS-IS|115|
|RIP|120|
|EIGRP externo|170|
|iBGP / Local BGP|200|
|Desconocida|255 (no confiable)|

**Importante:**  
Modificar el AD sin planificación puede causar **loops o inconsistencias**.

### **c) Métricas**

Cada protocolo define su propio método de métrica:

- **RIP:** saltos (_hop count_)
    
- **EIGRP:** ancho de banda, retardo, carga, confiabilidad
    
- **OSPF:** costo basado en ancho de banda
    
- **IS-IS:** costo configurable
    
- **BGP:** política (AS-Path, Local Preference, MED)
    

**Equal-Cost Multipath (ECMP):**  
Permite instalar varias rutas con igual métrica y balancear carga.

**EIGRP Unequal-Cost Load Balancing:**  
Permite balanceo proporcional mediante el comando `variance`.

---

## **4. Tipos de rutas estáticas**

### **a) Directly Attached Static Route**

Referencia solo la interfaz de salida:

`ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/1`

Requiere que la interfaz esté _up_ para instalarse en la RIB.

---

### **b) Recursive Static Route**

Usa una dirección IP de siguiente salto; el router busca recursivamente la interfaz de salida:

`ip route 192.168.10.0 255.255.255.0 10.10.10.1`

---

### **c) Fully Specified Static Route**

Define tanto la interfaz como la IP del siguiente salto (útil en interfaces multiacceso):

`ip route 192.168.10.0 255.255.255.0 GigabitEthernet0/1 10.10.10.1`

---

### **d) Floating Static Route**

Ruta de respaldo con mayor AD que la principal:

`ip route 192.168.10.0 255.255.255.0 10.10.10.2 200`

Solo se instala si la ruta principal desaparece.

---

## **5. Funcionamiento de la tabla de enrutamiento (RIB)**

La **Routing Information Base (RIB)** almacena todas las rutas conocidas.  
La ruta con **mejor AD y métrica** se convierte en la ruta activa y se pasa a la **Forwarding Information Base (FIB)**, usada por CEF (Cisco Express Forwarding) para el reenvío real de paquetes.

---

## **6. Procesos de forwarding en routers Cisco**

|Modo|Descripción|Uso|
|---|---|---|
|**Process Switching**|Cada paquete se procesa por CPU. Lento.|Depuración o tráfico bajo.|
|**Fast Switching**|Usa caché de decisiones previas. Parcialmente en desuso.|Pre-CEF.|
|**CEF (Cisco Express Forwarding)**|Crea estructuras _FIB_ y _Adjacency Table_ en hardware. Rápido, escalable.|Predeterminado en Cisco IOS.|

**Estructuras clave:**

- **FIB (Forwarding Information Base):** construida a partir de la RIB.
    
- **Adjacency Table:** contiene las MAC y ARP de los vecinos para reenvío inmediato.
    

---

## **7. Buenas prácticas de diseño**

- Utilizar **rutas estáticas** solo en enlaces simples o controlados.
    
- Emplear **EIGRP u OSPF** en redes medianas/grandes.
    
- No mezclar protocolos sin redistribución planificada.
    
- Asegurar **consistencia de AD** entre rutas estáticas y dinámicas.
    
- Habilitar **CEF** para mejorar el rendimiento.