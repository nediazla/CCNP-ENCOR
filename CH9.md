## **Capítulo 9 – Advanced OSPF**

### **1. Introducción**

El **OSPF avanzado** aborda los mecanismos que permiten **escalar, optimizar y asegurar** grandes dominios de enrutamiento.  
En esta etapa, el enfoque se centra en:

- Estructura multi-área (backbone y áreas no backbone)
    
- Control y optimización de LSAs
    
- Resumen y filtrado de rutas
    
- Tipos de áreas especiales
    
- Redistribución y rutas externas
    
- Implementación avanzada con OSPFv3
    

---

## **2. Jerarquía y diseño de áreas**

### **Por qué usar múltiples áreas**

- Reduce el tamaño de la **LSDB (Link-State Database)**
    
- Acelera el cálculo **SPF (Dijkstra)**
    
- Aísla fallos o flaps de enlace a un área específica
    
- Permite **resumen de rutas** (lo que no es posible en un solo área)
    

**Área 0 (Backbone):**  
Es el núcleo del dominio OSPF.  
Todas las demás áreas deben conectarse **directa o lógicamente** (mediante un _virtual link_) al backbone.

**Virtual Link:**  
Permite conectar un área no contigua al Área 0.

```
area 113 virtual-link 1.1.1.1
```

Usar solo como solución temporal o de contingencia.

---

## **3. Tipos de áreas OSPF**

|Tipo de área|Descripción|LSAs permitidos|
|---|---|---|
|**Normal**|Área completa; acepta LSAs 1–5,7|1–5,7|
|**Stub**|Bloquea LSAs tipo 5; recibe ruta por defecto (0.0.0.0/0) del ABR|1–3|
|**Totally Stubby**|Bloquea LSAs tipo 3 y 5; solo permite rutas intra-área + ruta por defecto|1–2|
|**NSSA (Not-So-Stubby)**|Similar al stub, pero permite redistribuir rutas externas como tipo 7|1–3,7|

**Configuración:**

```
area 2 stub
area 2 nssa
area 2 stub no-summary       # Totally stubby
```

**Conversión de LSAs:**  
Los **ABR** convierten los LSAs **Tipo 7 (NSSA)** en **Tipo 5** para el Área 0.

---

## **4. LSAs avanzados y control de flooding**

**LSA principales (OSPFv2):**

|Tipo|Generado por|Función|
|---|---|---|
|**1**|Router|Describe enlaces del router dentro del área|
|**2**|DR|Describe redes multi-acceso|
|**3**|ABR|Resume redes entre áreas|
|**4**|ABR|Anuncia ASBRs|
|**5**|ASBR|Publica rutas externas|
|**7**|ASBR NSSA|Redistribuye rutas en NSSA|

**Importante:**  
Solo los **LSAs 1 y 2** se inundan dentro del área.  
Los **3, 4, 5, 7** cruzan áreas o dominios.

**Control de flooding:**

- _LSA throttling:_ limita frecuencia de actualizaciones.
    
```
timers throttle lsa all 20 200 5000
```
    
- _Incremental SPF:_ recalcula parcialmente solo cambios relevantes.
    
- _SPF throttling:_ evita recalcular SPF por cada LSA recibido.
    
```
timers throttle spf 10 100 5000
```
    

---

## **5. OSPF Path Selection (revisión avanzada)**

Orden de preferencia en la instalación de rutas:

1. **Intra-area (O)**
    
2. **Inter-area (O IA)**
    
3. **Externa Tipo 1 (O E1)**
    
4. **Externa Tipo 2 (O E2)**
    
5. **NSSA Tipo 1 (O N1)**
    
6. **NSSA Tipo 2 (O N2)**
    

**Diferencia E1/E2:**

- **E1:** suma costo interno + externo
    
- **E2:** usa solo costo externo (predeterminado)
    

**ECMP (Equal-Cost Multipath):**  
Permite balanceo entre rutas con igual métrica.  
Por defecto: 4 rutas, ajustable con:

```
maximum-paths 8
```

---

## **6. Route Summarization**

### **a) Inter-area summarization (Type 3)**

Realizado en el **ABR**, reduce LSAs entre áreas:

```
area 1 range 10.10.0.0 255.255.0.0 [advertise | not-advertise] [cost]
```

### **b) External summarization (Type 5)**

Hecho en el **ASBR**, reduce LSAs externos:

```
summary-address 192.168.0.0 255.255.0.0
```

**Ventajas:**

- Reduce LSDB global
    
- Minimiza impacto del cálculo SPF
    
- Crea límites de propagación de fallos
    

---

## **7. OSPF Optimization**

**Parámetros que pueden ajustarse:**

|Parámetro|Comando|Descripción|
|---|---|---|
|**Hello Timer**|`ip ospf hello-interval`|Controla frecuencia de hellos|
|**Dead Timer**|`ip ospf dead-interval`|Determina tiempo antes de declarar un vecino inactivo|
|**Cost Reference Bandwidth**|`auto-cost reference-bandwidth`|Permite diferenciar enlaces 1G/10G/40G|
|**DR Priority**|`ip ospf priority`|Influye en elección DR/BDR|

**Ejemplo:**

```
router ospf 10
 auto-cost reference-bandwidth 10000
```

---

## **8. Redistribución y rutas externas**

**Tipos de rutas externas:**

- **E1/E2:** redistribuidas desde otros IGPs
    
- **N1/N2:** redistribuidas en NSSA
    

**Redistribución ejemplo:**

```
router ospf 10
 redistribute eigrp 100 subnets metric-type 1 metric 20
```

**Filtros de redistribución:**  
Para evitar loops, usar `route-map` o `distribute-list`.

---

## **9. OSPFv3 (Avanzado)**

**Diferencias clave respecto a OSPFv2:**

- Soporta **IPv4 e IPv6** en un mismo proceso.
    
- Configuración **por interfaz**, no por `network`.
    
- Usa **link-local addresses** como origen.
    
- Puede correr **múltiples instancias** por enlace.
    
- LSAs no incluyen direcciones IP — solo prefijos.
    

**Nuevos LSAs (OSPFv3):**

|Tipo|Descripción|
|---|---|
|**8**|Link LSA – Prefijos locales del enlace|
|**9**|Intra-area Prefix LSA – Prefijos de routers o redes|

**Multicast OSPFv3:**

- AllSPFRouters → `FF02::5`
    
- AllDRouters → `FF02::6`
    

**Ejemplo de configuración (dual-stack):**

```
ipv6 unicast-routing
router ospfv3 10
 router-id 1.1.1.1
 address-family ipv6 unicast
!
interface Gi0/0
 ospfv3 10 ipv6 area 0
```

---

## **10. Verificación avanzada**

```
show ip ospf database
show ip ospf border-routers
show ip ospf interface
show ip route ospf
show ospfv3 interface brief
```
**Códigos de tabla de enrutamiento:**

```
O  = Intra-area
O IA = Inter-area
O E1 / O E2 = External
O N1 / O N2 = NSSA
```

---

## **11. Puntos clave de examen**

|Concepto|Detalle|
|---|---|
|Algoritmo|Dijkstra SPF|
|Protocolo IP|89|
|AD|110|
|DR/BDR elección|Por prioridad o Router ID|
|Áreas especiales|Stub, Totally Stubby, NSSA|
|Resumen inter-área|`area range`|
|Resumen externo|`summary-address`|
|Virtual link|`area X virtual-link RID`|
|Timers|Hello 10s / Dead 40s|
|OSPFv3|Multi-instancia, IPv4/IPv6 support|