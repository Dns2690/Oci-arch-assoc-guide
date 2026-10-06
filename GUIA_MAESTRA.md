# OCI Architect Associate --- Guía Completa de Estudio

**Idioma de explicación:** Español\
**Terminología de examen:** English\
**Objetivo:** Preparación para Oracle Cloud Infrastructure (OCI)
Architect Associate\
**Enfoque:** Comprender arquitectura y seleccionar el servicio correcto
en escenarios de examen.

> **Cómo usar esta guía:** No intentes memorizar OCI como una lista de
> productos. En el examen, identifica primero el requisito: acceso
> público o privado, misma región o distinta región, alta
> disponibilidad, costo, latencia, persistencia, shared storage,
> disaster recovery, etc. Después elimina las opciones que no cumplen
> ese requisito.

------------------------------------------------------------------------

# 0. Mapa de cobertura del examen

Esta edición integra el temario de Architect Associate, la documentación oficial actual y los patrones observados en el Practice Exam. El archivo de práctica proporcionado muestra preguntas sobre Compute/autoscaling, compartments e IAM, VCN/peering/gateways/NSG, DNS, Object Storage, Block Volume, File Storage, OS Management Hub y detalles operativos. La teoría siguiente cubre esos dominios y sus variantes arquitectónicas.

| Módulo | Debes dominar | Nivel esperado |
|---|---|---|
| Foundations | Region, AD, FD, tenancy, OCID | explicar + diseñar HA |
| IAM | policies, inheritance, domains, dynamic groups, credentials | escribir/interpretar policies |
| Networking | VCN, subnet, routes, gateways, DRG, peering, VPN/FC | resolver escenarios |
| DNS | private/public, resolver endpoints, steering | distinguir dirección/policy |
| Compute | shapes, pools, autoscaling, capacity, OS updates | comportamiento exacto |
| Object Storage | tiers, lifecycle, auto-tiering, PAR, retention | números + escenarios |
| Block Volume | performance, backup, clone, replication | seleccionar por objetivo |
| File Storage | NFS, mount target, exports, snapshots | arquitectura compartida |
| Load Balancing | listener, backend set, health, LB/NLB | diseñar y diagnosticar |
| HA/DR | FD/AD/Region, RTO/RPO | trade-offs |
| Security | least privilege, encryption, Vault, network security | defense in depth |

---

# 1. OCI Foundations y arquitectura global

## 1.1 Tenancy

Una **tenancy** es la cuenta/entorno raíz de una organización en OCI.
Contiene los recursos y la estructura de **compartments**, identidades,
políticas y suscripciones a regiones.

Conceptos relacionados:

-   **OCID (Oracle Cloud Identifier):** identificador único asignado a
    la mayoría de recursos OCI.
-   **Home Region:** región seleccionada al crear la tenancy. Algunos
    recursos IAM tienen implicaciones particulares relacionadas con la
    home region.
-   **Regions:** áreas geográficas independientes.
-   **Availability Domains (ADs):** centros de datos aislados dentro de
    una región.
-   **Fault Domains (FDs):** agrupaciones de hardware dentro de un AD
    que permiten distribuir instancias para reducir fallos
    correlacionados.

### Regla de examen

**Region \> Availability Domain \> Fault Domain**

Si una aplicación necesita resistir la falla de un rack/hardware dentro
de un AD, distribuye recursos entre **Fault Domains**.\
Si debe resistir la pérdida de un AD y la región tiene varios AD,
distribuye entre **Availability Domains**.\
Para **Disaster Recovery (DR)** frente a pérdida regional, diseña entre
**Regions**.

------------------------------------------------------------------------

# 2. Identity and Access Management (IAM)

IAM responde dos preguntas:

1.  **Authentication:** ¿quién eres?
2.  **Authorization:** ¿qué puedes hacer?

## 2.1 Identity Domains

Un **Identity Domain** es un contenedor para administrar identidades y
configuraciones relacionadas. Puede contener:

-   Users
-   Groups
-   Applications
-   Authentication settings
-   Federation / identity provider integration

No confundas **Identity Domain** con **Compartment**:

  Concept           Función principal
  ----------------- ---------------------------------------------
  Identity Domain   Identidades y autenticación
  Compartment       Organización y control de acceso a recursos
  VCN               Red virtual
  Tenancy           Contenedor raíz de OCI

## 2.2 Users y Groups

Un **User** representa una identidad humana o cuenta que interactúa con
OCI.

Un **Group** agrupa users para asignar permisos mediante policies.

Diseño recomendado:

`User → Group → IAM Policy → Resource`

Evita diseñar permisos individualmente si pueden administrarse mediante
grupos.

## 2.3 Dynamic Groups

Un **Dynamic Group** agrupa **resources**, no usuarios humanos, según
matching rules.

Ejemplo conceptual:

> Todas las Compute Instances de un compartment determinado pertenecen a
> un Dynamic Group.

Esto permite usar **Instance Principals**, de modo que una Compute
Instance pueda llamar APIs de OCI sin guardar credenciales de usuario
dentro de la VM.

### Exam trap

-   Human users → **Groups**
-   OCI resources/instances → **Dynamic Groups**
-   Aplicación en una Compute Instance que necesita Object Storage sin
    credenciales locales → **Dynamic Group + Instance Principal + IAM
    Policy**

## 2.4 Compartments

Los **Compartments** son divisiones lógicas para organizar y controlar
acceso a recursos.

Pueden representar:

-   Producción / desarrollo
-   Departamentos
-   Proyectos
-   Networking / Security / Applications / Databases

Los compartments pueden ser jerárquicos.

### Herencia

Una policy aplicada en un parent compartment puede otorgar acceso a
recursos de child compartments según su alcance.

### Importante

Mover un recurso a otro compartment puede cambiar inmediatamente qué
policies se aplican a ese recurso.

## 2.5 IAM Policies

Sintaxis conceptual:

``` text
Allow <subject> to <verb> <resource-type> in <location> where <condition>
```

Ejemplo:

``` text
Allow group NetworkAdmins to manage virtual-network-family in compartment Production
```

### Verbs

De menor a mayor acceso:

``` text
inspect → read → use → manage
```

  Verb      Idea
  --------- ------------------------------------------------------
  inspect   listar/ver existencia y cierta información
  read      inspect + leer detalles/contenido según el recurso
  use       read + trabajar con recursos existentes
  manage    permisos administrativos completos del resource type

**manage** normalmente incluye los niveles inferiores.

### Resource types

Puedes dar acceso a:

-   Un tipo específico.
-   Una familia de recursos.

Ejemplos conceptuales:

``` text
instances
volumes
buckets
virtual-network-family
```

### Location

Puede ser:

``` text
in compartment <name>
```

o:

``` text
in tenancy
```

### Conditions

Permiten restringir una policy mediante variables, tags, network
sources, operaciones, etc.

## 2.6 Network Sources

Una **Network Source** representa direcciones IP o redes desde las que
se permite una operación/autenticación determinada. Puede utilizarse
para restringir acceso según el origen de la solicitud.

## 2.7 Federation

**Federation** permite usar un **Identity Provider (IdP)** externo para
autenticarse en OCI.

Escenario típico:

Empresa ya usa un proveedor corporativo de identidad y quiere evitar
administrar credenciales OCI independientes.

Piensa:

**Existing corporate identity provider → Federation**

## 2.8 Tags

Dos categorías principales:

### Free-form tags

Flexibles. No requieren definición previa estricta.

### Defined tags

Utilizan **tag namespaces** y claves definidas, ofreciendo mayor
gobierno y consistencia.

Usos:

-   Cost tracking
-   Governance
-   Automation
-   Organization
-   Conditional access

## IAM --- Lo que debes memorizar

1.  **Groups = users.**
2.  **Dynamic Groups = resources.**
3.  **Instance Principal** evita almacenar credenciales de usuario en
    una instance.
4.  Permisos: **inspect \< read \< use \< manage**.
5.  **Compartments** organizan recursos y ayudan a delimitar acceso.
6.  **Identity Domains** administran identidades.
7.  **Federation** integra un external IdP.
8.  **Defined Tags** son mejores para gobierno estandarizado.

### Exam traps IAM

**Pregunta:** Una aplicación en una VM necesita leer objetos de un
bucket sin almacenar API keys.\
**Respuesta conceptual:** Instance Principal + Dynamic Group + IAM
Policy.

**Pregunta:** El administrador quiere organizar Production y Development
y aplicar permisos distintos.\
**Respuesta:** Compartments + Policies.

------------------------------------------------------------------------

# 3. Networking --- VCN

Networking es uno de los dominios más importantes del examen.

## 3.1 Virtual Cloud Network (VCN)

Una **VCN** es una red virtual privada definida por software en OCI.

Es **regional**.

Incluye:

-   CIDR blocks
-   Subnets
-   Route Tables
-   Security Lists
-   Network Security Groups
-   Gateways
-   DHCP Options
-   VNICs
-   DNS

Ejemplo:

``` text
VCN: 10.0.0.0/16

Public subnet:  10.0.1.0/24
Private subnet: 10.0.2.0/24
Database:       10.0.3.0/24
```

## 3.2 CIDR

Ejemplo:

``` text
10.0.0.0/24
```

Un `/24` IPv4 proporciona 256 direcciones totales matemáticamente,
aunque OCI reserva direcciones dentro de cada subnet.

Evita CIDR superpuestos si vas a conectar redes mediante peering, VPN o
FastConnect.

### Exam rule

**Networks that must communicate privately → avoid overlapping CIDRs.**

## 3.3 Subnets

Una subnet divide el espacio IP de una VCN.

Puede ser:

-   **Public subnet**
-   **Private subnet**

### Public subnet

Permite que sus VNIC tengan public IPv4 addresses cuando corresponde.

Para acceso IPv4 desde/hacia Internet también necesitarás:

-   Internet Gateway
-   Route
-   Security rules
-   Public IP en el recurso que lo requiera

### Private subnet

No permite public IPv4 en las VNIC.

Puede obtener salida a Internet mediante:

**NAT Gateway**

Puede acceder privadamente a determinados servicios públicos OCI
mediante:

**Service Gateway**

## 3.4 VNIC

Una **Virtual Network Interface Card (VNIC)** conecta una Compute
Instance con una subnet.

Puede tener:

-   Primary private IP
-   Secondary private IPs
-   Public IP según configuración
-   Hostname
-   NSG memberships

Una instance puede tener varias VNIC según shape/configuración.

------------------------------------------------------------------------

# 4. Gateways y Routing

Esta tabla es esencial.

  ---------------------------------------------------------------------
  Necesidad                          Componente
  ---------------------------------- ----------------------------------
  Recurso público ↔ Internet         Internet Gateway

  Private subnet → Internet, sin     NAT Gateway
  conexiones iniciadas desde         
  Internet                           

  Private access a servicios OCI     Service Gateway
  compatibles, como Object Storage   

  VCN ↔ on-premises / otras redes    DRG
  mediante routing avanzado          

  VCN ↔ VCN same region, modelo      LPG
  legacy                             

  On-premises ↔ OCI encrypted over   Site-to-Site VPN
  Internet                           

  On-premises ↔ OCI private          FastConnect
  dedicated connectivity             
  ---------------------------------------------------------------------

## 4.1 Internet Gateway (IGW)

Permite comunicación entre una VCN y Internet.

No basta con crear el gateway.

Se necesita:

1.  Internet Gateway
2.  Route rule
3.  Public IP cuando aplica
4.  Security rules

### Exam trap

**Internet Gateway ≠ automáticamente Internet access.**

## 4.2 NAT Gateway

Permite a recursos en private subnets iniciar conexiones hacia Internet
sin asignarles public IPv4.

Ejemplo:

``` text
Private Compute → NAT Gateway → Internet
```

El tráfico de respuesta puede regresar, pero hosts de Internet no pueden
iniciar arbitrariamente una conexión hacia la private instance a través
del NAT Gateway.

**Keyword:** outbound Internet from private subnet.

## 4.3 Service Gateway

Permite acceso privado desde una VCN hacia servicios OCI compatibles sin
atravesar Internet.

Ejemplo clásico:

``` text
Private DB/Compute → Service Gateway → Object Storage
```

### Exam trap

Private instance necesita descargar patches de un repositorio público de
Internet → **NAT Gateway**.

Private instance necesita acceder a Object Storage sin Internet →
**Service Gateway**.

## 4.4 Dynamic Routing Gateway (DRG)

El **DRG** funciona como router virtual para conectar redes.

Puede participar en:

-   VCN connectivity
-   Site-to-Site VPN
-   FastConnect
-   Multi-VCN architectures
-   Transit routing

Arquitecturas modernas usan DRG para escenarios hub-and-spoke y
conectividad entre múltiples redes.

## 4.5 Route Tables

Una route rule determina:

``` text
Destination CIDR/Service → Target
```

Ejemplos conceptuales:

``` text
0.0.0.0/0 → Internet Gateway
0.0.0.0/0 → NAT Gateway
Oracle Services Network → Service Gateway
On-prem CIDR → DRG
```

### Regla

**Security rule decide si el tráfico está permitido. Route rule decide
por dónde viajará.**

Ambas pueden ser necesarias.

------------------------------------------------------------------------

# 5. Security Lists vs Network Security Groups

## Security Lists

Se asocian con **subnets**.

Las reglas afectan las VNIC de la subnet que utilizan esas listas.

## Network Security Groups (NSG)

Se asocian con recursos/VNIC específicos compatibles.

Permiten agrupar recursos según su función, independientemente de la
subnet.

Ejemplo:

``` text
NSG-Web
NSG-App
NSG-Database
```

### Comparación

  ---------------------------------------------------------------------
  Security List                      NSG
  ---------------------------------- ----------------------------------
  Subnet-oriented                    Resource/VNIC-oriented

  Más amplia                         Más granular

  Útil para reglas comunes de subnet Recomendable para application
                                     tiers
  ---------------------------------------------------------------------

Oracle recomienda generalmente **NSG** para control granular.

## Stateful vs Stateless

### Stateful

OCI mantiene connection tracking.

Si permites una conexión en una dirección, el response traffic asociado
es permitido automáticamente.

### Stateless

No se mantiene connection tracking.

Debes permitir explícitamente el tráfico requerido en ambas direcciones.

### Exam hint

High-volume traffic donde quieres evitar connection tracking puede
llevar a **stateless rules**, pero debes diseñar ingress/egress
correctamente.

------------------------------------------------------------------------

# 6. VCN Peering

## Local Peering

VCNs en la **same region** pueden comunicarse privadamente.

Las arquitecturas modernas pueden usar un upgraded DRG; **Local Peering
Gateway (LPG)** sigue apareciendo en material y escenarios legacy.

## Remote Peering

VCNs en **different regions** pueden conectarse privadamente mediante
DRG/RPC según la arquitectura.

### Regla de examen

``` text
Same region → Local connectivity / DRG (LPG en escenarios legacy)
Different regions → Remote Peering through DRG
```

------------------------------------------------------------------------

# 7. Site-to-Site VPN vs FastConnect

## Site-to-Site VPN

Conecta on-premises con OCI mediante **IPSec tunnels** a través de
Internet.

Características:

-   Encrypted
-   Rápido de implementar
-   Internet-based
-   Utiliza DRG
-   Útil como conectividad principal en ciertos escenarios o backup de
    FastConnect

## FastConnect

Conectividad privada dedicada entre on-premises y OCI.

Características:

-   No atraviesa Internet público
-   Consistent network experience
-   Apropiado para tráfico empresarial y transferencia intensiva
-   Puede utilizar BGP
-   Puede combinarse con VPN para resiliencia/diseños específicos

### Exam decision

**"Encrypted connection over Internet" → Site-to-Site VPN**

**"Dedicated/private connection; does not traverse Internet" →
FastConnect**

------------------------------------------------------------------------

# 8. Load Balancing

## 8.1 Load Balancer (LB)

Distribuye tráfico entre backend servers.

Conceptos:

-   Load Balancer
-   Listener
-   Backend Set
-   Backend Server
-   Health Check
-   Load Balancing Policy
-   SSL/TLS
-   Session Persistence

Puede ser public o private según diseño.

### Health Checks

Determinan si un backend está healthy. Un backend unhealthy puede dejar
de recibir tráfico.

### Listener

Define cómo recibe tráfico el LB:

-   Protocol
-   Port
-   SSL configuration, etc.

### Backend Set

Agrupa backend servers y configuración asociada.

## 8.2 Network Load Balancer (NLB)

Pensado para tráfico de red de alta performance y escenarios Layer
3/Layer 4.

Regla simplificada:

-   **HTTP/HTTPS features, Layer 7 behavior → Load Balancer**
-   **Very high performance / low latency / TCP-UDP network traffic →
    Network Load Balancer**

No memorices únicamente "LB = HTTP". Lee qué funciones exige el
escenario.

------------------------------------------------------------------------

# 9. DNS

OCI DNS permite administrar zones y records.

Conceptos:

-   Public DNS zones
-   Private DNS zones
-   Records
-   Resolver
-   Traffic Management

## Public DNS

Resuelve nombres públicamente.

## Private DNS

Permite nombres privados accesibles en redes autorizadas.

### Exam hint

Si una aplicación interna debe resolver nombres que no deben publicarse
en Internet → **Private DNS**.

------------------------------------------------------------------------

# 9.1 OS Management Hub y Oracle Ksplice

El Practice Exam puede preguntar por la administración del sistema
operativo, no solamente por la infraestructura que aloja la VM.

## OS Management Hub

**OS Management Hub** ayuda a administrar y aplicar software updates a
instancias administradas. Puedes crear **update jobs**, incluso para
grupos de instancias o todas las instancias administradas de un
compartment.

## Oracle Ksplice

**Oracle Ksplice** permite aplicar actualizaciones críticas del kernel
de Oracle Linux **sin reiniciar (without rebooting)** la instancia.

``` text
Latest OS / software updates
          │
          ├── Scheduled / managed update jobs ──► OS Management Hub
          │
          └── Critical kernel patches
              without reboot ───────────────────► Ksplice
```

> 🎯 **EXAM KEY:**
> `critical kernel security updates + Oracle Linux + no reboot` →
> **Ksplice**

> ⚠️ **TRAP:** Ksplice resuelve el requisito específico de actualizar el
> kernel con mínimo downtime; no confundas esta función con autoscaling,
> instance pools o una nueva custom image.

------------------------------------------------------------------------

# 10. Compute

OCI Compute proporciona:

-   Virtual Machines (VM)
-   Bare Metal
-   Flexible shapes
-   Images
-   Boot volumes
-   Instance configurations
-   Instance pools
-   Autoscaling

## 10.1 VM vs Bare Metal

### VM

Virtualized compute.

Ventajas:

-   Flexibilidad
-   Menor tamaño inicial
-   Fácil scaling
-   Múltiples opciones de shapes

### Bare Metal

Servidor físico dedicado al cliente.

Usa Bare Metal cuando el escenario requiera:

-   Direct access to physical server
-   Muy alto rendimiento
-   Workloads que no quieren hypervisor sharing
-   Determinadas necesidades de licenciamiento o aislamiento

## 10.2 Shapes

Un **shape** determina CPU, memory y otras características.

### Flexible Shapes

Permiten ajustar OCPUs y memoria dentro de rangos permitidos.

### OCPU

No confundas **OCPU** con vCPU. La relación depende de la
arquitectura/plataforma; en preguntas de diseño enfócate en que OCPU
representa capacidad de CPU OCI y flexible shapes permiten dimensionar
recursos.

## 10.3 Images

Una image es plantilla del sistema operativo usada para crear instances.

Tipos conceptuales:

-   Oracle-provided images
-   Custom images
-   Marketplace images

## 10.4 Boot Volume

Contiene el sistema usado para boot de la Compute Instance.

Es detachable y persistent según las acciones seleccionadas al terminar
la instance.

------------------------------------------------------------------------

# 11. Instance Pools, Configurations y Autoscaling

## Instance Configuration

Plantilla para crear instances con configuración consistente.

## Instance Pool

Grupo de Compute Instances creadas desde una Instance Configuration.

Puede integrarse con Load Balancer.

## Autoscaling

Ajusta el tamaño/capacidad según reglas.

Tipos de enfoque:

-   Metric-based
-   Schedule-based

### Scenario

La demanda web aumenta cuando CPU supera un threshold → **metric-based
autoscaling**.

Se sabe que todos los lunes a las 8:00 aumenta tráfico →
**schedule-based autoscaling**.

### Exam flow

``` text
Instance Configuration
        ↓
Instance Pool
        ↓
Autoscaling
        ↓
Load Balancer (cuando aplica)
```

------------------------------------------------------------------------

# 12. Capacity y opciones de Compute

## Capacity Reservations

Reserva capacidad de Compute para garantizar disponibilidad cuando se
necesite.

Keyword:

**"Need assurance that capacity will be available" → Capacity
Reservation**

## Preemptible Instances

Capacidad de menor costo que puede ser recuperada/preempted por OCI.

Adecuada para:

-   Fault-tolerant workloads
-   Batch
-   Jobs reiniciables

No ideal para workload crítico que no tolera interrupciones.

## Burstable Instances

Adecuadas para workloads con uso promedio bajo y picos ocasionales de
CPU.

## Dedicated VM Hosts

Permiten ejecutar VMs en un physical server dedicado a una tenancy, útil
para aislamiento o ciertos requisitos de licenciamiento/compliance.

------------------------------------------------------------------------

# 13. High Availability en Compute

Distribuye instances:

-   Entre **Fault Domains** para fallas de hardware/rack.
-   Entre **Availability Domains** cuando la región dispone de varios
    AD.
-   Entre **Regions** para DR regional.

Combina con:

-   Load Balancer
-   Instance Pools
-   Autoscaling
-   Replicated/appropriate storage
-   DNS / Traffic Management según arquitectura

### Anti-pattern

Dos VMs en el mismo FD no constituyen el diseño más robusto contra
fallos de infraestructura correlacionados.

------------------------------------------------------------------------

# 14. Object Storage

**Object Storage** almacena datos no estructurados como objetos.

Casos:

-   Backups
-   Logs
-   Images/video
-   Static content
-   Data lake
-   Archives

Es **regional** y no está ligado a una Compute Instance específica.

Jerarquía conceptual:

``` text
Namespace → Bucket → Object
```

## 14.1 Storage Tiers

### Standard

Hot data.

-   Acceso frecuente
-   Acceso inmediato
-   Mayor costo de almacenamiento relativo
-   Sin minimum retention

### Infrequent Access

Cool data.

-   Acceso poco frecuente
-   Sigue disponible inmediatamente
-   Menor storage cost
-   Retrieval fee
-   Minimum retention: **31 days**

### Archive

Cold data.

-   Muy poco acceso
-   Lowest storage cost
-   Debe **restore** antes de leer
-   Minimum retention: **90 days**
-   Restauración no es instantánea

### Tabla esencial

  -------------------------------------------------------------------------
  Tier            Access                  Minimum retention Retrieval
  --------------- --------------------- ------------------- ---------------
  Standard        inmediato/frecuente                  none normal

  Infrequent      inmediato, poco                   31 days retrieval fee
  Access          frecuente                                 

  Archive         restore required                  90 days offline until
                                                            restored
  -------------------------------------------------------------------------

## 14.2 Auto-Tiering

Puede mover automáticamente objetos elegibles entre **Standard** e
**Infrequent Access** según patrones de acceso.

No equivale a lifecycle policy.

### 🔢 Auto-Tiering: detalle exacto

Auto-Tiering monitorea patrones de acceso y puede mover automáticamente
**objects larger than 1 MiB** desde **Standard** hacia **Infrequent
Access** cuando cumplen los requisitos de elegibilidad. Si
posteriormente se acceden con mayor frecuencia, OCI puede devolverlos a
**Standard**.

``` text
STANDARD
   │
   │ object > 1 MiB + access pattern becomes infrequent
   ▼
INFREQUENT ACCESS
   │
   │ becomes frequently accessed
   └──────────────────────────────► STANDARD
```

> 🎯 **EXAM KEY:** `Auto-Tiering + object > 1 MiB + Standard → ?` =
> **Infrequent Access**.

> ⚠️ **TRAP:** Auto-Tiering no significa "Standard → Archive".

## 14.3 Object Lifecycle Management

Permite crear reglas automáticas como:

-   Move objects to another tier
-   Delete objects after a period
-   Gestionar versiones anteriores

Útil para optimización de costos y retención.

## 14.4 Versioning

Mantiene versiones de objetos cuando cambian o se eliminan.

Útil para recuperación de cambios accidentales.

## 14.5 Replication

Object Storage puede replicar objetos para escenarios de resiliencia/DR
según configuración.

Distingue:

-   **Replication** → mantener copia en destino.
-   **Versioning** → conservar versiones históricas.
-   **Lifecycle** → transición/eliminación automática.
-   **Retention Rule** → impedir modificación/eliminación durante
    periodo/regla.

## 14.6 Retention Rules

Protegen objetos frente a modificación/eliminación durante un periodo
requerido.

Keyword:

**Compliance / immutable retention → Retention Rules**

### 🧠 Cómo se calcula una Time-Bound Retention Rule

La duración se aplica **individualmente a cada object** tomando como
referencia su **Last-Modified timestamp**.

Ejemplo: hoy creas una retention rule de **1 year**.

``` text
Object X ─ last modified 14 months ago
          14 months > 1 year
          └─► Ya superó la duración → puede modificarse/eliminarse

Object Y ─ last modified 3 months ago
           3 months < 1 year
           └─► Quedan 9 months de protección
```

> 🎯 **EXAM KEY:** la edad del bucket no decide la retención; importa el
> **Last-Modified de cada object**.

Una rule está activa desde su creación. **Locking the rule** controla si
la propia regla puede modificarse. Para una rule que se va a bloquear
existe un **mandatory 14-day delay** antes del lock irreversible. Una
vez locked, solo puede aumentarse su duración.

> ⚠️ **TRAP:** **Retention Rule** (impedir cambios durante el periodo) ≠
> **Versioning** (guardar versiones anteriores).

## 14.7 Pre-Authenticated Requests (PAR)

Permiten acceso a un bucket/object mediante una URL especialmente
generada sin que el receptor necesite sus propias credenciales OCI,
dentro del alcance y tiempo definidos.

### Exam trap

Compartir temporalmente un object con alguien sin OCI account →
**Pre-Authenticated Request**.

## 14.8 Encryption

Object Storage cifra datos at rest. Para requisitos de control de
claves, estudia la integración con **Vault / customer-managed keys**.

------------------------------------------------------------------------

# 15. Block Volume

Block Volume proporciona almacenamiento de bloques persistente para
Compute.

Tipos:

-   **Boot Volume**
-   **Block Volume**

## Características

-   Persistent
-   Detachable
-   Encrypted at rest
-   Backups
-   Clones
-   Performance levels
-   Replication

## 15.1 Attachments

### iSCSI

Protocolo estándar sobre TCP/IP.

### Paravirtualized

Attachment virtualizado disponible para VMs compatibles.

## 15.2 Performance levels

Oracle ofrece niveles como:

-   Lower Cost
-   Balanced
-   Higher Performance
-   Ultra High Performance

El nivel puede ajustarse según requisitos compatibles.

## 15.3 Backup vs Clone

### Backup

Copia point-in-time almacenada como backup, útil para protección y
recuperación.

Puede copiarse entre regiones y restaurarse.

### Clone

Nueva copia de un volume orientada a crear rápidamente otro volume a
partir del existente.

### Exam heuristic

-   **DR / durable recovery / copy to another region → Backup or
    Replication**
-   **Quick duplicate for testing/dev → Clone**

## 15.4 Volume Groups

Permiten agrupar volumes para ciertas operaciones coordinadas, como
backups consistentes de un conjunto.

Útil cuando una aplicación utiliza varios volumes.

## 15.5 Block Volume Replication

Permite replicación automática asíncrona continua de block/boot volumes
a otro AD o región compatible.

Use cases:

-   Disaster Recovery
-   Migration
-   Business continuity

### Importante

Un block volume normalmente se adjunta a una instance en el **same
Availability Domain** del volume. Backups permiten restaurar en otro AD
dentro de la región y copiarse a otra región.

------------------------------------------------------------------------

# 16. File Storage

OCI **File Storage** es un managed network file system.

Utiliza **NFSv3**.

Ideal cuando múltiples Compute Instances necesitan acceso concurrente a
un shared file system.

Ejemplo:

``` text
VM1 ─┐
VM2 ─┼→ Mount Target → File System
VM3 ─┘
```

## 16.1 Mount Target

Endpoint NFS dentro de una subnet.

Proporciona IP/DNS usado por clientes NFS.

Un mount target puede exponer múltiples file systems mediante exports.

## 16.2 Export

Hace un file system accesible mediante un mount target.

## 16.3 Export Set

Colección de exports asociada al mount target.

## 16.4 Snapshots

Point-in-time snapshots del file system.

Útiles para recuperación rápida de archivos/datos.

## 16.5 Acceso

File Storage puede ser accesible:

-   Desde VCN
-   A través de VCN peering
-   FastConnect
-   IPSec VPN

## Storage decision table

  Requirement                                             Storage
  ------------------------------------------------------- ----------------
  Disk for VM/database-like block I/O                     Block Volume
  Shared NFS filesystem                                   File Storage
  Massive unstructured objects/backups/logs               Object Storage
  Long-term cold object data                              Archive tier
  Local ephemeral/high-performance NVMe where available   Local NVMe

------------------------------------------------------------------------

# 17. Encryption y Key Management

OCI cifra muchos servicios at rest por defecto.

Para control adicional, usa **OCI Vault / Key Management**.

Conceptos:

-   Master encryption keys
-   Customer-managed keys
-   Key rotation
-   Secrets

### Scenario

"Organization must control encryption key lifecycle" → piensa en **Vault
/ customer-managed keys**.

No confundas:

-   Encryption at rest
-   Encryption in transit
-   IAM authorization
-   Network security

Son capas distintas.

------------------------------------------------------------------------

# 18. Arquitecturas típicas

## 18.1 Public three-tier web application

``` text
Internet
   |
Internet Gateway
   |
Public Load Balancer
   |
Web/App instances (preferably controlled with NSGs)
   |
Private Database tier
```

Principios:

-   Database private.
-   Least privilege.
-   NSGs separan tiers.
-   Multiple instances/FDs/ADs para HA.
-   Load Balancer health checks.
-   NAT Gateway para outbound Internet desde private subnet cuando sea
    necesario.
-   Service Gateway para private access a supported OCI services.

## 18.2 Private application needing Object Storage

``` text
Private Compute
      |
Service Gateway
      |
Object Storage
```

No necesita Internet Gateway para este flujo.

## 18.3 Private application needing Internet updates

``` text
Private Compute
      |
NAT Gateway
      |
Internet
```

## 18.4 Hybrid connectivity

### Opción A

``` text
On-premises
    |
IPSec VPN
    |
DRG
    |
VCN
```

### Opción B

``` text
On-premises
    |
FastConnect
    |
DRG
    |
VCN
```

## 18.5 Hub-and-Spoke

Un DRG puede servir como componente central de routing para múltiples
VCN y conexiones híbridas.

Keywords:

-   Centralized routing
-   Multiple VCNs
-   Transit
-   On-premises access

→ piensa en **DRG hub-and-spoke**.

------------------------------------------------------------------------

# 19. High Availability vs Disaster Recovery

No son lo mismo.

## High Availability (HA)

Mantener servicio disponible ante fallos locales.

Herramientas:

-   Fault Domains
-   Availability Domains
-   Load Balancer
-   Multiple instances
-   Instance Pools
-   Autoscaling

## Disaster Recovery (DR)

Recuperarse de desastre mayor, potencialmente regional.

Herramientas:

-   Cross-region design
-   Object replication
-   Block Volume backups/replication
-   DNS/Traffic Management
-   Database DR technologies según servicio

### RTO

**Recovery Time Objective:** cuánto tiempo puede tardar la recuperación.

### RPO

**Recovery Point Objective:** cuánto dato puedes permitirte perder,
medido en tiempo.

Ejemplo:

-   RPO 0 → no se tolera pérdida de datos.
-   RPO 1 hour → se tolera perder hasta aproximadamente una hora de
    cambios.
-   RTO 15 min → servicio debe recuperarse aproximadamente dentro de ese
    objetivo.

### Exam logic

Lower RPO/RTO normalmente requiere más automatización, replicación y
costo.

------------------------------------------------------------------------

# 20. Cost Optimization

Busca la combinación adecuada de:

-   Right-sized flexible shapes
-   Autoscaling
-   Burstable instances
-   Preemptible instances para workloads tolerantes
-   Object Storage tiers
-   Lifecycle policies
-   Auto-Tiering
-   Appropriate Block Volume performance
-   Compartments, budgets/tags para organización y control

### Trap

La opción técnicamente más potente no siempre es la correcta. Si la
pregunta enfatiza **cost-effective**, elimina diseños sobredimensionados
que no aportan al requisito.

------------------------------------------------------------------------

# 21. Security Design Principles

## Defense in Depth

Usa varias capas:

1.  IAM
2.  Compartments
3.  Network segmentation
4.  NSGs/Security Lists
5.  Private subnets
6.  Encryption
7.  Key management
8.  Monitoring/logging
9.  Cloud Guard y servicios de seguridad cuando aplique

## Least Privilege

Otorga solo permisos necesarios.

Evita:

``` text
manage all-resources in tenancy
```

cuando un scope menor cumple el requisito.

## Public exposure

Bases de datos y servicios internos deben permanecer privados salvo
requisito explícito.

------------------------------------------------------------------------

# 22. Decisiones rápidas de Networking

Memoriza este árbol:

``` text
¿Necesita Internet?
|
+-- Recurso público bidireccional → Internet Gateway
|
+-- Private subnet, solo outbound → NAT Gateway
|
+-- Acceso privado a supported OCI public service → Service Gateway

¿Necesita conectar redes?
|
+-- On-prem encrypted over Internet → Site-to-Site VPN
|
+-- On-prem dedicated/private → FastConnect
|
+-- Multiple VCN/transit/hybrid routing → DRG
|
+-- VCNs same region legacy peering → LPG
|
+-- Cross-region VCN connectivity → DRG remote peering
```

------------------------------------------------------------------------

# 23. Decisiones rápidas de Storage

``` text
¿Datos como disk blocks?
→ Block Volume

¿Shared filesystem / NFS?
→ File Storage

¿Unstructured objects?
→ Object Storage

¿Acceso frecuente?
→ Standard

¿Poco frecuente pero inmediato?
→ Infrequent Access

¿Raramente accedido y puede esperar restore?
→ Archive
```

------------------------------------------------------------------------

# 24. Decisiones rápidas de Compute

``` text
General-purpose virtualized workload
→ VM

Need physical server
→ Bare Metal

Need dedicated physical host for multiple VMs
→ Dedicated VM Host

Fault-tolerant interruptible job
→ Preemptible

Low baseline CPU with occasional bursts
→ Burstable

Need guaranteed future compute capacity
→ Capacity Reservation

Need multiple identical instances
→ Instance Configuration + Instance Pool

Need automatic scaling
→ Autoscaling
```

------------------------------------------------------------------------

# 25. Diferencias que debes dominar

## NAT Gateway vs Service Gateway

**NAT:** Internet.\
**Service:** supported OCI services privately.

## Security List vs NSG

**Security List:** subnet-level association.\
**NSG:** selected resources/VNICs; granular.

## VPN vs FastConnect

**VPN:** encrypted over Internet.\
**FastConnect:** private dedicated connectivity.

## Object Storage vs File Storage

**Object:** object API, unstructured, regional scale.\
**File:** NFS shared filesystem.

## Block Volume vs File Storage

**Block:** disk-like block device.\
**File:** shared NFS.

## Backup vs Replication

**Backup:** point-in-time recovery copy.\
**Replication:** ongoing copy for DR/migration.

## Versioning vs Retention

**Versioning:** preserve previous object versions.\
**Retention:** prevent prohibited modification/deletion for retention
requirement.

## Load Balancer vs Network Load Balancer

**LB:** richer application/L7 load-balancing features.\
**NLB:** high-performance low-latency network traffic/L3-L4 scenarios.

------------------------------------------------------------------------

# 26. Exam Traps --- frases que delatan la respuesta

  ---------------------------------------------------------------------
  Frase del escenario                Piensa primero en
  ---------------------------------- ----------------------------------
  "without public IP" + Internet     NAT Gateway
  outbound                           

  "privately access Object Storage"  Service Gateway

  "dedicated private connection from FastConnect
  on-prem"                           

  "encrypted tunnel over Internet"   Site-to-Site VPN

  "resources rather than entire      NSG
  subnet"                            

  "shared NFS"                       File Storage

  "disk attached to VM"              Block Volume

  "unstructured data"                Object Storage

  "temporary access without OCI      PAR
  credentials"                       

  "retain immutable objects"         Retention Rule

  "keep previous versions"           Versioning

  "automatic tier based on access"   Auto-Tiering

  "31-day minimum"                   Infrequent Access

  "90-day minimum / restore first"   Archive

  "resource calls OCI API without    Instance Principal
  user credentials"                  

  "group compute resources by        Dynamic Group
  matching rule"                     

  "guarantee compute capacity"       Capacity Reservation

  "interruptible batch workload"     Preemptible

  "known traffic increase at         Schedule-based Autoscaling
  specific time"                     

  "scale based on CPU metric"        Metric-based Autoscaling

  "central routing for many VCNs"    DRG

  "resist hardware failure in same   Fault Domains
  AD"                                

  "regional disaster"                Cross-region DR
  ---------------------------------------------------------------------

------------------------------------------------------------------------

# 26.1 Visual Cheat Sheet --- reconocer la respuesta en segundos

``` text
NETWORK
Internet ↔ public resource       → IGW
Private → Internet              → NAT
Private → OCI service           → Service Gateway
On-prem ↔ OCI over Internet     → VPN
On-prem ↔ OCI dedicated/private → FastConnect
Many networks / transit         → DRG
DNS queries coming IN           → Listening endpoint
DNS queries going OUT           → Forwarding endpoint

IAM
Human identities                → Group
OCI resources                   → Dynamic Group
VM authenticates to OCI         → Instance Principal
Any compartment                 → in tenancy
Full network administration     → manage virtual-network-family

OBJECT STORAGE
Hot                             → Standard
Cool + immediate                → Infrequent Access
Cold + restore                  → Archive
Auto-Tiering destination        → Infrequent Access
Temporary URL                   → PAR
Old copies                      → Versioning
Cannot modify/delete for time   → Retention Rule

COMPUTE
Predictable time                → Schedule-based autoscaling
Performance threshold           → Metric-based autoscaling
Kernel patch without reboot     → Ksplice
Guaranteed capacity             → Capacity Reservation
Interruptible job               → Preemptible
```

### Las cuatro preguntas que debes hacerte

``` text
1. WHAT is the requirement?
          ↓
2. WHICH OCI layer owns it?
          ↓
3. WHAT exact keyword narrows the choice?
          ↓
4. WHICH distractor fails one requirement?
```

------------------------------------------------------------------------


# 26.2 Módulo de profundización - Nivel examen y nivel arquitecto

Este módulo amplía los temas que aparecen en el Practice Exam oficial y conecta cada dato puntual con la arquitectura completa. La meta es que puedas resolver tanto una pregunta directa como una variante nueva del mismo concepto.

## A. Compute, Instance Pools y Autoscaling - dominio completo

### A.1 Instance Configuration, Instance Pool y Autoscaling Configuration

Piensa en estas piezas como una cadena:

```text
IMAGE + SHAPE + VNIC + METADATA + BOOT SETTINGS
                    │
                    ▼
          Instance Configuration
                    │ plantilla
                    ▼
              Instance Pool
                    │ conjunto de instancias
                    ▼
         Autoscaling Configuration
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Metric-based             Schedule-based
```

- **Instance Configuration**: plantilla reutilizable para crear instancias de forma consistente.
- **Instance Pool**: conjunto de instancias administradas como grupo.
- **Autoscaling Configuration**: reglas que cambian el tamaño o lifecycle state del pool.
- Cada instance pool puede tener **una autoscaling configuration**.
- Una configuración puede usar **una metric-based policy** o múltiples **schedule-based policies**.
- Schedule-based admite hasta **50 policies** por autoscaling configuration.

### A.2 Metric-based autoscaling

Se usa cuando la carga es variable y una métrica indica cuándo crecer o reducir.

```text
Monitoring metric
      │
      ├── above scale-out threshold ──► add instances
      │
      └── below scale-in threshold  ──► remove instances
```

Conceptos que debes reconocer:

- **Scale-out threshold**: condición para aumentar capacidad.
- **Scale-in threshold**: condición para reducir capacidad.
- **Cooldown period**: evita que una acción nueva ocurra antes de que el sistema tenga tiempo de estabilizarse y producir métricas representativas.
- El pool respeta los límites configurados de **minimum size**, **initial size** y **maximum size**.

> 🎯 **EXAM KEY:** CPU/memory-like performance metric + threshold = **metric-based autoscaling**.

### A.3 Schedule-based autoscaling

Se usa cuando el patrón es conocido por tiempo.

- Utiliza **Quartz cron expressions**.
- El horario se expresa en **UTC**.
- Scale-out y scale-in programados normalmente requieren policies diferentes.
- El target alcanzado permanece hasta que otra policy o acción lo cambie.
- El campo cooldown puede aparecer, pero **no controla** schedule-based autoscaling.

> ⚠️ **TRAP:** una pregunta puede mencionar `initial size` para distraerte. Una schedule policy cambia el **target pool size**; el pool no regresa solo al initial size.

### A.4 Scale-in behavior

Al reducir un instance pool, OCI intenta mantener distribución equilibrada entre **Availability Domains** y luego **Fault Domains**; dentro del FD elegido, termina primero la instancia más antigua.

### A.5 Preemptible, Burstable, Capacity Reservation y Dedicated VM Host

| Opción | Cuándo pensar en ella | No usar cuando... |
|---|---|---|
| Preemptible Instance | batch, CI, rendering, workload reiniciable | no tolera interrupción |
| Burstable Instance | baseline bajo con picos ocasionales | CPU alta sostenida |
| Capacity Reservation | debes garantizar capacidad futura | buscas el menor costo para trabajo interrumpible |
| Dedicated VM Host | aislamiento físico/licenciamiento | solo necesitas una VM normal |
| Bare Metal | acceso/rendimiento de servidor físico | una VM satisface el requisito |

### A.6 OCPU, shapes y Flexible Shapes

- **Shape** define recursos de cómputo.
- **Flexible shape** permite elegir OCPUs y memoria dentro del rango permitido.
- **OCPU** es la unidad de capacidad de CPU de OCI. No asumas que OCPU y vCPU son términos idénticos en todas las arquitecturas.
- El examen puede preguntar por optimización: evita sobredimensionar si una flexible shape permite ajustar recursos.

### A.7 OS Management Hub y Ksplice

**OS Management Hub** centraliza inventario, software sources, updates, groups, lifecycle environments y jobs. Puede crear jobs one-time o recurring para instancias o grupos.

**Ksplice** aplica determinados critical security updates al kernel de Oracle Linux **without reboot / zero downtime**. También puede cubrir componentes userspace soportados como glibc/OpenSSL en Oracle Linux.

```text
Fleet patch management ─────────► OS Management Hub
Critical kernel patch, no reboot ► Ksplice
```

> ⚠️ **TRAP:** Ksplice actualiza en memoria; sigue siendo importante mantener paquetes on-disk actualizados para futuros reboots.

---

## B. Networking - de subnet a arquitectura híbrida

### B.1 VCN y subnet

Una **VCN** es regional. Las subnets actuales pueden ser regionales. Una subnet define un rango CIDR y utiliza route tables, security lists y DHCP options asociados.

Una **public subnet** permite public IPv4 en VNICs; una **private subnet** no. Que una subnet sea pública no significa que todo recurso sea automáticamente accesible desde Internet.

Para acceso público IPv4 normalmente debes tener:

```text
Public IP + Route to IGW + IGW + Security rule + Service listening
```

Si falta una pieza, el flujo puede fallar.

### B.2 Route Table vs Security

```text
ROUTE TABLE = ¿por dónde va?
NSG / SECURITY LIST = ¿está permitido?
```

Una route rule contiene destination y target. Ejemplos:

```text
0.0.0.0/0             → Internet Gateway
0.0.0.0/0             → NAT Gateway
OCI service CIDR/label → Service Gateway
On-premises CIDR       → DRG
```

### B.3 Internet Gateway, NAT Gateway y Service Gateway

| Requisito | Gateway |
|---|---|
| Public resource ↔ Internet | Internet Gateway |
| Private resource → Internet outbound | NAT Gateway |
| Private resource → supported OCI public services | Service Gateway |

> 🧠 **MEMORY HOOK:** **I-N-S** = Internet, NAT, Service: público, salida privada, servicio OCI privado.

### B.4 Stateful vs Stateless

**Stateful**: connection tracking; response traffic de una conexión permitida se acepta automáticamente.

**Stateless**: no connection tracking; debes permitir explícitamente los flujos necesarios en ambas direcciones.

El examen puede intentar hacerte elegir una route table cuando el problema es security, o una security rule cuando el problema es routing.

### B.5 Security Lists vs NSGs

- **Security List**: asociada a subnet; sus reglas afectan VNICs de esa subnet sujetas a la lista.
- **NSG**: agrupa VNICs/resources específicos y permite reglas basadas en aplicación/tier.
- Un recurso puede pertenecer a múltiples NSGs según límites del servicio.
- Para diseños de aplicación, NSG suele ofrecer segmentación más granular.

```text
Subnet: 10.0.1.0/24
  Security List ─────────► baseline subnet rules

Web VM ─► NSG-WEB
App VM ─► NSG-APP
DB VM  ─► NSG-DB
```

### B.6 VCN Peering

**Peering** permite comunicación privada entre VCNs sin enviar tráfico por Internet.

- Same region: local connectivity; arquitecturas actuales pueden usar DRG. Material legacy puede hablar de **LPG**.
- Different regions: remote peering mediante DRG/RPC según diseño.
- Los CIDR de redes que deben rutearse entre sí no deben solaparse.
- Peering no significa transitividad automática. No asumas que A↔B y B↔C implica A↔C.

### B.7 DRG

El **Dynamic Routing Gateway** es el router virtual clave para:

- VCN ↔ on-premises
- Site-to-Site VPN
- FastConnect
- Multi-VCN connectivity
- Hub-and-spoke
- Transit routing
- Remote peering

Piensa en el DRG como el centro de conectividad privada avanzada.

### B.8 Site-to-Site VPN

- Usa **IPSec**.
- Viaja por Internet.
- Se integra con DRG.
- Normalmente se configuran túneles redundantes.
- Adecuada cuando quieres cifrado y despliegue rápido sin circuito dedicado.

### B.9 FastConnect

- Private dedicated connectivity.
- No depende del Internet público para el data path.
- Útil para ancho de banda consistente, hybrid cloud y transferencia intensiva.
- Puede usar BGP según el modelo de conexión.

### B.10 VNIC, private IP y public IP

- Cada instance tiene una primary VNIC.
- Una VNIC tiene primary private IP y puede tener secondary private IPs.
- Public IP se asocia a un private IP cuando corresponde.
- **Ephemeral public IP**: temporal y administrada para escenarios simples.
- **Reserved public IP**: persistente y puede reasignarse dentro de las reglas del servicio.

### B.11 Network Load Balancer vs Load Balancer

**Load Balancer** ofrece funciones de aplicación, listeners, backend sets, SSL/TLS, policies, health checks y session persistence.

**Network Load Balancer (NLB)** se orienta a tráfico Layer 3/4, alta performance y baja latencia. Puede preservar características de red requeridas por determinados workloads.

### B.12 Health checks

Un backend que falla el health check puede retirarse de la distribución hasta recuperarse. Distingue:

- LB reachable pero backend unhealthy.
- Security rules bloqueando health checks.
- Route correcta pero puerto/listener equivocado.

---

## C. DNS - detalle que sí aparece en examen

### C.1 Public DNS y Private DNS

- **Public Zone**: nombres resolubles públicamente.
- **Private Zone**: nombres internos visibles a redes autorizadas.
- VCN resolver proporciona resolución DNS a recursos de la VCN.

### C.2 Listening vs Forwarding Endpoint

```text
Other VCN / on-prem DNS ──query──► LISTENING endpoint ──► VCN Resolver
VCN Resolver ──rule/query─────────► FORWARDING endpoint ──► external/private DNS
```

- **Listening endpoint** recibe queries.
- **Forwarding endpoint** envía queries hacia otro DNS según resolver rules.

### C.3 Traffic Management Steering Policies

Debes distinguir al menos:

- **Failover**: primary/secondary; usa health para desviar si falla el principal.
- **Load Balancer**: distribuye respuestas según pesos.
- **Geolocation**: responde según ubicación geográfica del cliente.
- **ASN**: responde según Autonomous System Number del origen.
- **IP Prefix**: responde según rango IP del cliente.

> 🎯 **EXAM KEY:** “different endpoints based on location of end user” → **Geolocation steering**.

---

## D. IAM - dominar sintaxis, herencia y credenciales

### D.1 Jerarquía

```text
TENANCY
├── Compartment A
│   ├── Compartment B
│   │   └── Compartment C
│   └── Resources
└── Identity Domains / IAM resources
```

### D.2 Policy inheritance

Una policy en un parent compartment puede aplicar a recursos en child compartments. La herencia va hacia abajo, no hacia arriba.

Ejemplo:

```text
Allow group NetworkAdmins to manage virtual-network-family in compartment A
```

Si B y C están debajo de A, el permiso puede cubrir recursos de B/C por inheritance.

### D.3 Verbs

```text
inspect < read < use < manage
```

- **inspect**: listar/ver metadata limitada.
- **read**: inspeccionar y leer detalles/contenido según resource type.
- **use**: usar recursos existentes; no equivale necesariamente a administración completa.
- **manage**: conjunto más amplio, incluye operaciones administrativas.

### D.4 Families y aggregate resource types

Oracle agrupa resource types en families. Ejemplos comunes:

- `virtual-network-family`
- `instance-family`
- `volume-family`
- `object-family`

Una pregunta puede ofrecer `all-resources` como distractor: funciona, pero viola least privilege si una family específica basta.

### D.5 Dynamic Groups e Instance Principals

Dynamic Group = recursos que coinciden con matching rules.

```text
resource attributes
      │ matching rule
      ▼
Dynamic Group
      │ IAM Policy
      ▼
OCI API access
```

**Instance Principal** permite a una instance autenticarse como principal sin guardar credenciales de usuario.

### D.6 Identity Domains

Identity Domain administra users, groups, applications, federation y políticas de autenticación/sign-on. No es un compartment.

### D.7 Federation

Permite usar un external Identity Provider. En preguntas, busca frases como:

- corporate IdP
- single sign-on
- existing enterprise identities

### D.8 Auth Tokens

Un **Auth Token** es una credencial generada para compatibilidad con servicios/protocolos que requieren autenticación tipo username/password, por ejemplo clientes que interactúan con APIs compatibles que no usan API signing keys. Trátalo como secreto.

### D.9 API Signing Keys vs Auth Tokens vs Instance Principals

| Necesidad | Mecanismo |
|---|---|
| Usuario/SDK firma llamadas OCI API | API signing key |
| Cliente compatible que requiere token/password | Auth Token |
| Compute resource llama OCI API sin user credential | Instance Principal |
| Workload resource principal dinámico | Dynamic Group + Policy |

### D.10 Network Sources

Network Source = conjunto de IPs/rangos que puede usarse en IAM/sign-in policy para restringir el origen de solicitudes.

> ⚠️ **TRAP:** Network Source no es NSG. Uno pertenece a control de identidad/origen de IAM; el otro es network security de VNICs.

---

## E. Compartments - reglas que Oracle pregunta

- Son lógicos, no redes.
- Pueden contener recursos de múltiples regiones.
- Un recurso normalmente pertenece a un compartment a la vez.
- Los compartments pueden anidarse.
- Las policies de parents se heredan hacia descendants.
- Puedes mover determinados recursos entre compartments si el servicio lo permite; al moverlos cambia el conjunto de policies que los afecta.
- No confundas compartment con Availability Domain ni Identity Domain.

### E.1 Path mental

Si tienes:

```text
A
└── B
    └── C
```

Una policy en A puede alcanzar B/C. Una policy solo en C no concede permisos automáticamente en A/B.

---

## F. Object Storage - dominio completo

### F.1 Namespace, Bucket, Object

```text
Object Storage Namespace
        └── Bucket
             ├── object-a
             ├── object-b
             └── object-c
```

Un object incluye data + metadata y se identifica por su object name/key dentro del bucket.

### F.2 Storage tiers

| Tier | Uso | Acceso | Minimum retention |
|---|---|---|---:|
| Standard | hot | inmediato | none |
| Infrequent Access | cool | inmediato | 31 days |
| Archive | cold | restore first | 90 days |

### F.3 Auto-Tiering

- Para buckets Standard con la función habilitada.
- Objetos elegibles **> 1 MiB** pueden moverse Standard → Infrequent Access según patrón de acceso.
- Si vuelven a ser frecuentes, pueden retornar a Standard.
- No mueve automáticamente a Archive.

### F.4 Lifecycle Policies

Permiten acciones automáticas basadas en edad/condiciones, por ejemplo:

- transition a otro tier permitido
- delete objects
- gestionar previous versions

**Lifecycle** responde a reglas temporales/condiciones. **Auto-Tiering** responde al patrón de acceso.

### F.5 Versioning

Cuando está habilitado, sobrescribir/eliminar un object puede generar/conservar versiones. Esto protege contra cambios accidentales.

Conceptos:

- latest version
- previous version
- delete marker

### F.6 Retention Rules

Dos ideas:

- **Time-bound**: retiene por duración definida.
- **Indefinite**: retención sin fecha de expiración hasta que la regla sea modificada/eliminada cuando esté permitido.

Para time-bound, la duración se evalúa por object usando **Last-Modified**.

Si una rule se **locks**, el lock se vuelve irreversible tras el mandatory delay. Una locked rule no puede reducirse/eliminarse; su duración puede aumentarse según las reglas del servicio.

### F.7 Pre-Authenticated Requests (PAR)

Permite acceso mediante URL a un bucket/object sin requerir que el receptor tenga credenciales OCI propias.

Puede limitarse por:

- target
- permissions
- expiration

> 🔐 Trata la PAR URL como secreto: quien la posee puede ejercer los permisos concedidos hasta expiración/revocación.

### F.8 Public access

No hagas un bucket público para resolver un requisito de acceso temporal a un object. PAR suele ser la opción más acotada.

### F.9 Replication

Replication mantiene copia en destino para DR/data distribution. No la confundas con Versioning ni Backup.

---

## G. Block Volume - rendimiento, backup y políticas

### G.1 Boot Volume vs Block Volume

- **Boot Volume** contiene el OS de la instance.
- **Block Volume** es almacenamiento adicional de bloques persistente.

### G.2 Attachments

- iSCSI
- Paravirtualized, cuando es compatible

### G.3 Performance tiers / VPUs

El examen puede usar nombres como:

- Lower Cost
- Balanced
- Higher Performance
- Ultra High Performance

La idea esencial: más **VPUs/GB** implican mayor capacidad de rendimiento y mayor costo. **Balanced** es el nivel general/default común en muchos escenarios; **Lower Cost** prioriza costo; Higher/Ultra priorizan IOPS/throughput.

> ⚠️ **TRAP:** “high throughput / intensive I/O” no apunta a Lower Cost.

### G.4 Backup vs Clone vs Replication

```text
Need restore point / durable recovery copy ─► Backup
Need quick independent duplicate           ─► Clone
Need continuously maintained DR copy       ─► Replication
```

- Clone es point-in-time disk-to-disk y no requiere backup+restore.
- Backup puede copiarse a otra región.
- Replication sirve para continuidad/DR y migración.

### G.5 Backup Policies

Oracle ofrece backup policies administradas con schedules/retention. También puedes definir políticas propias según requisitos. Debes saber diferenciar frecuencia de backup y retención.

### G.6 Volume Groups

Agrupan boot/block volumes para operaciones coordinadas, especialmente cuando una aplicación necesita consistencia entre múltiples volumes.

### G.7 IAM para volumes

El examen puede preguntar qué aggregate resource type permite crear/administrar volumes. Aprende a reconocer `volume-family` y el verbo requerido, evitando `all-resources` si no es necesario.

---

## H. File Storage - todo lo que puede confundirse

### H.1 NFS

OCI File Storage proporciona managed shared file systems usando **NFS**. En el contexto del temario del examen, reconoce **NFSv3**.

### H.2 Mount Target

Mount Target = NFS endpoint en una subnet.

```text
Client VM
   │ NFS
   ▼
Mount Target (IP/DNS in subnet)
   │ export path
   ▼
File System
```

- Proporciona IP/DNS.
- Se reutiliza para exponer múltiples file systems mediante exports.
- Requiere direcciones IP en la subnet; planifica tamaño de subnet.

### H.3 Export y Export Set

- **Export**: hace un file system accesible mediante un path en un mount target.
- **Export Set**: conjunto de exports asociado al mount target.
- Export options controlan acceso NFS de clientes.

### H.4 Connectivity

File Storage puede alcanzarse desde redes conectadas correctamente, incluyendo VCN y escenarios híbridos/peered cuando routing y security lo permiten. No asumas que “shared file system” implica Internet público.

### H.5 Snapshots

Snapshot = point-in-time view/copy lógica del file system para recuperación. No es lo mismo que Block Volume backup.

### H.6 Replication

File Storage replication permite mantener una réplica para DR según regiones/targets compatibles. Distingue replication de snapshot.

---

## I. Load Balancing - arquitectura y troubleshooting

### I.1 Componentes

```text
Client
  │
  ▼
Load Balancer
  │ Listener :443 HTTPS
  ▼
Backend Set
  ├── Backend A
  ├── Backend B
  └── Backend C
       ▲
   Health Check
```

- **Listener**: protocol + port de entrada.
- **Backend Set**: grupo lógico de backends + policy + health check.
- **Backend**: servidor destino.
- **Health Check**: determina salud.
- **Policy**: algoritmo de distribución.

### I.2 Public vs Private Load Balancer

- Public: entry point accesible públicamente según reglas.
- Private: internal applications dentro de redes privadas.

### I.3 Session persistence

Mantiene solicitudes relacionadas de un cliente hacia el mismo backend cuando el workload lo requiere. No la actives por reflejo: reduce libertad de distribución y debe responder a un requisito real.

---

## J. High Availability, Fault Domains y Disaster Recovery

### J.1 Fault Domain

Protege contra fallos correlacionados de hardware/maintenance dentro de un AD.

### J.2 Availability Domain

Datacenter/cluster aislado dentro de región según arquitectura regional. Cuando una región tiene múltiples AD, distribuir entre ellos mejora tolerancia a fallo de AD.

### J.3 Region

Para disaster regional, necesitas otra region y estrategia de datos/aplicación.

```text
Hardware/rack issue  → Fault Domain
AD outage            → Multiple ADs (where available)
Region disaster      → Cross-region DR
```

### J.4 RTO y RPO

- **RTO** = cuánto tiempo puedes estar fuera de servicio.
- **RPO** = cuánto dato, expresado en tiempo, puedes perder.

Menores RTO/RPO suelen exigir mayor replicación, automatización y costo.

---

## K. Security y Encryption - capa transversal

### K.1 Defense in Depth

```text
Identity → IAM / MFA / Federation
Network  → private subnets / NSG / Security Lists
Data     → encryption / Vault / retention
Compute  → patching / hardened images / OS Management Hub
Ops      → logging / monitoring / Cloud Guard
```

### K.2 Encryption

Distingue:

- at rest
- in transit
- Oracle-managed keys
- customer-managed keys

**Vault / Key Management** se usa cuando la organización necesita controlar lifecycle/rotation de claves compatibles.

### K.3 Least Privilege

Nunca elijas `manage all-resources in tenancy` si una policy más específica cumple exactamente el requisito.

---

## L. Tabla maestra de decisiones

| Si la pregunta dice... | Primera respuesta candidata |
|---|---|
| private subnet needs Internet | NAT Gateway |
| private access to Object Storage | Service Gateway |
| public resource needs Internet | Internet Gateway |
| on-prem encrypted over Internet | Site-to-Site VPN |
| dedicated private on-prem link | FastConnect |
| central routing / many VCNs | DRG |
| firewall by application/resource | NSG |
| firewall baseline by subnet | Security List |
| DNS queries received | Listening endpoint |
| DNS queries sent elsewhere | Forwarding endpoint |
| location-based DNS answer | Geolocation steering |
| human identities | Group |
| OCI resources as principals | Dynamic Group |
| VM calls OCI without user keys | Instance Principal |
| permissions in any compartment | tenancy scope |
| shared NFS | File Storage |
| VM disk | Block Volume |
| unstructured objects | Object Storage |
| hot object data | Standard |
| cool but immediate | Infrequent Access |
| cold + restore acceptable | Archive |
| automatic Standard↔IA | Auto-Tiering |
| preserve old object copies | Versioning |
| prevent modification/deletion | Retention Rule |
| temporary URL access | PAR |
| fast disk duplicate | Clone |
| recovery copy | Backup |
| ongoing DR copy | Replication |
| known traffic schedule | Schedule-based autoscaling |
| CPU threshold | Metric-based autoscaling |
| kernel patch without reboot | Ksplice |
| guaranteed compute capacity | Capacity Reservation |
| interruptible batch | Preemptible |
| shared physical host isolation | Dedicated VM Host |
| regional disaster | Cross-region DR |

---

## M. Cómo responder preguntas de múltiples respuestas

Oracle puede usar **Select two / Select three**. No busques “la mejor” y te detengas. Evalúa cada opción como verdadero/falso frente al enunciado.

Método:

```text
1. Marca el número exacto de respuestas solicitado.
2. Convierte cada opción en una afirmación independiente.
3. Descarta las que son técnicamente ciertas pero no responden al requisito.
4. Verifica scope, dirección y timing:
   - IN vs OUT
   - same region vs cross-region
   - user vs resource
   - subnet vs VNIC
   - now vs scheduled
5. Relee la pregunta con las respuestas elegidas.
```

### Palabras que cambian la respuesta

- **any compartment**
- **without public IP**
- **without reboot**
- **immediately available**
- **dedicated**
- **over the Internet**
- **same region**
- **different region**
- **shared**
- **NFS**
- **last modified**
- **temporary access**
- **lowest cost**
- **least operational overhead**

---

## N. Checklist de dominio real - no solo examen

Antes de presentarte, deberías poder diseñar verbalmente estos escenarios sin mirar apuntes:

1. Public three-tier web app con DB privada y HA.
2. Private app que consume Object Storage y repositorios públicos.
3. Hybrid connectivity con VPN y migración posterior a FastConnect.
4. Hub-and-spoke con múltiples VCNs y DRG.
5. Fleet de Oracle Linux con patching centralizado y Ksplice.
6. Instance Pool con metric autoscaling y otra solución con schedule autoscaling.
7. Shared NFS para varias VMs con mount target y exports.
8. Object Storage con lifecycle, Auto-Tiering, versioning, retention y PAR sin confundirlos.
9. Block Volume con clone para test, backup para recovery y replication para DR.
10. IAM hierarchy con parent/child compartments, Dynamic Groups e Instance Principals.
11. Private DNS entre VCN/on-prem con listening y forwarding endpoints.
12. Cross-region DR explicando RTO/RPO y qué datos deben replicarse.

Si puedes explicar **por qué** elegiste cada componente y **por qué no** usarías las alternativas, estás preparado para variantes nuevas de las preguntas.

---

# 27. Banco de 50 preguntas estilo OCI Architect Associate

Estas preguntas son **originales** y están calibradas al estilo
observado en el Oracle Practice Exam: algunas comprueban un detalle
exacto; otras presentan un escenario corto. Hazlas primero sin consultar
el solucionario.

## Preguntas 1--10 --- IAM y Networking

**1.** You want members of the `NetworkAdmins` group to manage OCI
networking resources in every compartment in the tenancy. Which policy
statement best meets the requirement?

A. `Allow group NetworkAdmins to use virtual-network-family in tenancy`\
B.
`Allow group NetworkAdmins to manage virtual-network-family in tenancy`\
C. `Allow group NetworkAdmins to manage all-resources in tenancy`\
D.
`Allow group NetworkAdmins to inspect virtual-network-family in tenancy`

**2.** A Compute instance must call Object Storage APIs without storing
a user's API signing key. What should you use?

A. Auth token\
B. Dynamic Group + Instance Principal + IAM Policy\
C. Public bucket\
D. Network Security Group

**3.** Which IAM verb normally grants the broadest permissions?

A. inspect\
B. read\
C. use\
D. manage

**4.** A private Compute instance needs to download updates from a
public Internet repository without receiving a public IP. Which gateway
should be used?

A. Service Gateway\
B. NAT Gateway\
C. Internet Gateway directly on the instance\
D. DRG

**5.** A private application must access Object Storage without
traversing the public Internet. Which gateway is appropriate?

A. NAT Gateway\
B. Internet Gateway\
C. Service Gateway\
D. Local Peering Gateway

**6.** Which security construct is associated with selected
VNICs/resources rather than all VNICs governed by a subnet's security
list?

A. NSG\
B. Route Table\
C. DRG\
D. DHCP Options

**7.** Which component determines the next hop for traffic destined for
an on-premises CIDR?

A. NSG\
B. Route Table\
C. Security List\
D. Dynamic Group

**8.** Your data center requires an encrypted connection to OCI over the
public Internet. Which service fits?

A. FastConnect\
B. Site-to-Site VPN\
C. Service Gateway\
D. NAT Gateway

**9.** Your data center requires private dedicated connectivity to OCI
and traffic must not traverse the public Internet. Which service fits?

A. FastConnect\
B. Site-to-Site VPN only\
C. Internet Gateway\
D. Remote Peering only

**10.** Which OCI component acts as a virtual router for hybrid and
multi-VCN connectivity?

A. DRG\
B. NSG\
C. VNIC\
D. Backend Set

## Preguntas 11--20 --- DNS, Routing y Compute

**11.** Which DNS resolver endpoint receives queries from other VCN
resolvers or an on-premises DNS server?

A. Forwarding endpoint\
B. Listening endpoint\
C. Service endpoint\
D. Internet endpoint

**12.** Which DNS resolver endpoint sends DNS queries to another VCN's
listening resolver or to an on-premises DNS server according to resolver
rules?

A. Listening endpoint\
B. Forwarding endpoint\
C. NAT endpoint\
D. Public endpoint

**13.** An Oracle Linux fleet requires critical kernel security patches
with no reboot. Which technology is specifically designed for this?

A. Oracle Ksplice\
B. Instance Pool\
C. Block Volume Clone\
D. Capacity Reservation

**14.** You know that demand increases every weekday at 08:30 and
decreases at 18:00. Which autoscaling approach best matches the
requirement?

A. Metric-based only\
B. Schedule-based\
C. Manual scaling\
D. Object Lifecycle Management

**15.** OCI schedule-based autoscaling schedules are defined using:

A. SQL expressions in local time\
B. Quartz cron expressions in UTC\
C. IAM policy conditions\
D. DNS TTL values

**16.** To schedule a recurring scale-out in the morning and scale-in in
the evening to different target pool sizes, what is normally required?

A. One route rule\
B. Two schedule-based autoscaling policies\
C. Two instance pools\
D. Two tenancies

**17.** A web tier should scale out when average CPU utilization reaches
a configured threshold. Which approach?

A. Schedule-based autoscaling\
B. Metric-based autoscaling\
C. Lifecycle policy\
D. DNS steering

**18.** Which construct is the template used to create consistent
instances in an instance pool?

A. Instance Configuration\
B. Fault Domain\
C. Backend Set\
D. Capacity Reservation

**19.** Which Compute option is best suited to a fault-tolerant batch
workload that can be interrupted and restarted?

A. Dedicated VM Host\
B. Preemptible Instance\
C. Capacity Reservation\
D. Bare Metal only

**20.** A company must ensure that Compute capacity will be available
for a planned critical deployment. Which feature addresses this?

A. Auto-Tiering\
B. Capacity Reservation\
C. PAR\
D. Security List

## Preguntas 21--30 --- Availability y Object Storage

**21.** Two instances in the same Availability Domain should be
protected from a localized hardware failure. Where should they be
placed?

A. Same Fault Domain\
B. Different Fault Domains\
C. Same VNIC\
D. Same NSG only

**22.** Which architecture is most relevant when the requirement is to
survive a total regional disaster?

A. Same Fault Domain\
B. Cross-region DR\
C. Larger single VM\
D. One public subnet

**23.** When Object Storage Auto-Tiering is enabled, eligible objects
larger than 1 MiB can be moved from Standard to which tier?

A. Archive\
B. Infrequent Access\
C. File Storage\
D. Block Volume

**24.** An object moved by Auto-Tiering to Infrequent Access later
becomes frequently accessed. What can OCI do?

A. Delete it automatically\
B. Move it back to Standard\
C. Move it only to Archive\
D. Convert it to File Storage

**25.** Which Object Storage tier has a 31-day minimum retention period
and immediate access?

A. Standard\
B. Infrequent Access\
C. Archive\
D. File Storage

**26.** Which Object Storage tier has a 90-day minimum retention period
and requires restoration before the data can be read?

A. Standard\
B. Infrequent Access\
C. Archive\
D. Balanced

**27.** You need to give a third party temporary access to download an
Object Storage object without creating an OCI identity for that person.
What should you create?

A. Dynamic Group\
B. Pre-Authenticated Request\
C. DRG\
D. Security List

**28.** You want to preserve previous versions when an Object Storage
object is overwritten or deleted. Which feature?

A. Versioning\
B. Retention Rule\
C. Auto-Tiering\
D. FastConnect

**29.** A compliance requirement says objects must not be modified or
deleted during a specified period. Which feature?

A. Versioning\
B. Retention Rule\
C. PAR\
D. Lifecycle deletion

**30.** A bucket contains Object X, last modified 14 months ago, and
Object Y, last modified 3 months ago. Today you create a 1-year
time-bound retention rule. Which statement is correct?

A. Both objects are protected for another full year from today\
B. X is already beyond the one-year duration; Y has about nine months
remaining\
C. X has ten months remaining; Y can be deleted immediately\
D. Neither object is affected because they existed before the rule

## Preguntas 31--40 --- Storage y Load Balancing

**31.** Which timestamp is used to calculate the duration of a
time-bound Object Storage retention rule for each object?

A. Bucket creation time\
B. Object Last-Modified timestamp\
C. Tenancy creation time\
D. Last login time

**32.** Before a retention rule becomes irreversibly locked, OCI imposes
what mandatory delay?

A. 24 hours\
B. 7 days\
C. 14 days\
D. 31 days

**33.** Multiple Linux Compute instances need concurrent access to the
same managed NFSv3 filesystem. Which service?

A. Object Storage\
B. Block Volume\
C. File Storage\
D. Archive Storage

**34.** Which File Storage component provides the NFS endpoint in a
subnet?

A. Mount Target\
B. Backend Set\
C. Dynamic Group\
D. Boot Volume

**35.** An application needs disk-like persistent storage attached to a
Compute instance. Which service?

A. Object Storage\
B. Block Volume\
C. DNS\
D. File Storage only

**36.** A developer needs a fast duplicate of a Block Volume for a test
environment. Which feature is the most direct fit?

A. Clone\
B. Cross-region DR only\
C. Archive\
D. PAR

**37.** An application uses several Block Volumes and needs coordinated
backup operations for the set. Which feature should be considered?

A. Volume Group\
B. NSG\
C. Identity Domain\
D. DNS Zone

**38.** Which load-balancing option is oriented toward high-performance,
low-latency Layer 3/Layer 4 traffic?

A. Network Load Balancer\
B. Application Load Balancer only\
C. Service Gateway\
D. DRG

**39.** Which Load Balancer feature determines whether a backend should
continue receiving traffic?

A. Health Check\
B. Compartment\
C. Dynamic Group\
D. Route Table

**40.** Which component defines the protocol and port on which an OCI
Load Balancer accepts incoming traffic?

A. Listener\
B. Backend Server\
C. Fault Domain\
D. Export Set

## Preguntas 41--50 --- Arquitectura integrada y detalles de examen

**41.** A three-tier application has a public web entry point and a
private database tier. Which design is most appropriate?

A. Give every database server a public IP\
B. Public Load Balancer with private application/database tiers and
controlled security rules\
C. Put all resources in one public subnet and allow all ingress\
D. Use Object Storage as the network firewall

**42.** A private Compute instance needs both Object Storage access
without public Internet and access to a public software repository.
Which pair of gateways is relevant?

A. Service Gateway + NAT Gateway\
B. Two Internet Gateways\
C. DRG + LPG only\
D. FastConnect + PAR

**43.** Which statement correctly distinguishes Security Lists and NSGs?

A. Security Lists are IAM policies; NSGs are route tables\
B. Security Lists are subnet-oriented; NSGs are resource/VNIC-oriented\
C. NSGs work only on-premises\
D. They are identical

**44.** With a stateful security rule, what happens to response traffic
associated with an allowed connection?

A. It is automatically permitted by connection tracking\
B. It always requires an explicit reverse route in IAM\
C. It is always blocked\
D. It requires a PAR

**45.** A workload has a low CPU baseline but occasionally needs short
CPU bursts. Which Compute characteristic best matches?

A. Burstable Instance\
B. Preemptible only\
C. Archive tier\
D. Dedicated FastConnect

**46.** A company wants several VMs on a physical host dedicated to its
tenancy for isolation/licensing requirements. Which option?

A. Dedicated VM Host\
B. Preemptible Instance\
C. Public Load Balancer\
D. File Storage

**47.** Which statement best describes RPO?

A. Maximum acceptable time to restore service\
B. Maximum acceptable amount of data loss measured in time\
C. Number of Fault Domains\
D. Minimum Object Storage retention

**48.** Which statement best describes RTO?

A. Target time to restore service after disruption\
B. Target number of object versions\
C. Route-table timeout\
D. Encryption key rotation period

**49.** A company requires control over the lifecycle and rotation of
encryption keys used by supported OCI services. What should the
architect consider?

A. OCI Vault / customer-managed keys\
B. NAT Gateway\
C. Security List only\
D. DNS Resolver

**50.** Which architecture best reflects least privilege?

A. Give all administrators `manage all-resources in tenancy`\
B. Grant each role only the verbs, resource families and scope required
for its job\
C. Make every bucket public\
D. Put all users in Administrators

------------------------------------------------------------------------

# 28. Solucionario razonado --- 50 preguntas

  --------------------------------------------------------------------------------------------------
                       \#             Resp.             Clave de razonamiento
  ----------------------- ----------------------------- --------------------------------------------
                        1             **B**             `manage virtual-network-family in tenancy`
                                                        cubre administración de Networking en toda
                                                        la tenancy sin otorgar `all-resources`.

                        2             **B**             Instance Principal autentica el recurso;
                                                        Dynamic Group + Policy autorizan.

                        3             **D**             Orden general: inspect \< read \< use \<
                                                        manage.

                        4             **B**             Private → public Internet outbound = NAT
                                                        Gateway.

                        5             **C**             Private → supported OCI service = Service
                                                        Gateway.

                        6             **A**             NSG es resource/VNIC-oriented.

                        7             **B**             Route Table decide el next hop.

                        8             **B**             IPSec encrypted over Internet = Site-to-Site
                                                        VPN.

                        9             **A**             Dedicated/private connectivity =
                                                        FastConnect.

                       10             **A**             DRG es el router virtual para
                                                        hybrid/transit.

                       11             **B**             Listening recibe queries.

                       12             **B**             Forwarding reenvía queries según resolver
                                                        rules.

                       13             **A**             Ksplice aplica critical kernel patches sin
                                                        reboot.

                       14             **B**             Demanda predecible por hora =
                                                        schedule-based.

                       15             **B**             OCI usa Quartz cron y UTC.

                       16             **B**             Scale-out y scale-in programados requieren
                                                        policies separadas para targets distintos.

                       17             **B**             Threshold de CPU = metric-based.

                       18             **A**             Instance Configuration es la plantilla.

                       19             **B**             Preemptible tolera reclamación/interrupción.

                       20             **B**             Capacity Reservation asegura capacidad.

                       21             **B**             Diferentes FD reducen fallos correlacionados
                                                        dentro del AD.

                       22             **B**             Fallo regional requiere arquitectura
                                                        cross-region.

                       23             **B**             Auto-Tiering mueve Standard → Infrequent
                                                        Access.

                       24             **B**             Puede regresar a Standard al aumentar la
                                                        frecuencia de acceso.

                       25             **B**             Infrequent Access = 31 days minimum,
                                                        immediate access.

                       26             **C**             Archive = 90 days minimum + restore.

                       27             **B**             PAR da acceso temporal/scoped sin identidad
                                                        OCI del receptor.

                       28             **A**             Versioning conserva versiones anteriores.

                       29             **B**             Retention Rule impide
                                                        modificación/eliminación durante la
                                                        retención.

                       30             **B**             La duración se calcula desde Last-Modified
                                                        de cada object.

                       31             **B**             Time-bound retention usa Last-Modified.

                       32             **C**             El lock irreversible tiene 14-day mandatory
                                                        delay.

                       33             **C**             Shared managed NFSv3 = File Storage.

                       34             **A**             Mount Target es el endpoint NFS en la
                                                        subnet.

                       35             **B**             Disk-like persistent attachment = Block
                                                        Volume.

                       36             **A**             Clone es la copia rápida/directa para
                                                        dev/test.

                       37             **A**             Volume Group coordina operaciones sobre
                                                        varios volumes.

                       38             **A**             NLB se orienta a L3/L4, performance y low
                                                        latency.

                       39             **A**             Health Check identifica backend
                                                        healthy/unhealthy.

                       40             **A**             Listener define protocol/port de entrada.

                       41             **B**             Expón solo el entry point necesario; mantén
                                                        tiers internos privados.

                       42             **A**             Service Gateway para Object Storage; NAT
                                                        para Internet outbound.

                       43             **B**             Security List = subnet; NSG = selected
                                                        resources/VNICs.

                       44             **A**             Stateful mantiene connection tracking para
                                                        response traffic.

                       45             **A**             Burstable = baseline bajo + picos
                                                        ocasionales.

                       46             **A**             Dedicated VM Host = host físico dedicado
                                                        para varias VMs.

                       47             **B**             RPO = cuánto dato puedes perder, expresado
                                                        en tiempo.

                       48             **A**             RTO = cuánto puede tardar la recuperación.

                       49             **A**             Vault/customer-managed keys dan control del
                                                        key lifecycle.

                       50             **B**             Least privilege = solo permisos y scope
                                                        necesarios.
  --------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 29. Patrón del Practice Exam observado

Las capturas del Practice Exam muestran una mezcla útil para orientar el
estudio:

``` text
┌─────────────────────────────────────────────────────────┐
│  1) DIRECT FACT                                        │
│     "Which endpoint receives...?"                      │
│                                                         │
│  2) EXACT BEHAVIOR                                     │
│     "When Auto-Tiering is enabled..."                  │
│                                                         │
│  3) SHORT SCENARIO                                     │
│     "You have two objects..."                          │
│                                                         │
│  4) CONFIGURATION / POLICY                             │
│     "What must NetworkAdmin be allowed to do?"         │
│                                                         │
│  5) OPERATIONAL OCI DETAIL                             │
│     kernel updates / autoscaling behavior              │
└─────────────────────────────────────────────────────────┘
```

Por eso esta guía no se limita a arquitectura de alto nivel: también
marca **EXACT DETAILS** que pueden decidir una pregunta.

------------------------------------------------------------------------

# 30. Checklist final antes del examen

Debes poder explicar sin consultar apuntes:

-   [ ] Region vs AD vs FD
-   [ ] Tenancy vs Compartment vs Identity Domain
-   [ ] User Group vs Dynamic Group
-   [ ] Instance Principal
-   [ ] inspect vs read vs use vs manage
-   [ ] Public vs Private subnet
-   [ ] Route Table
-   [ ] Internet Gateway
-   [ ] NAT Gateway
-   [ ] Service Gateway
-   [ ] DRG
-   [ ] Local vs Remote VCN connectivity
-   [ ] Site-to-Site VPN vs FastConnect
-   [ ] Security List vs NSG
-   [ ] Stateful vs Stateless
-   [ ] Load Balancer vs Network Load Balancer
-   [ ] VM vs Bare Metal vs Dedicated Host
-   [ ] Instance Configuration vs Instance Pool
-   [ ] Metric vs Schedule Autoscaling
-   [ ] Preemptible vs Burstable vs Capacity Reservation
-   [ ] Object vs Block vs File Storage
-   [ ] Standard vs Infrequent Access vs Archive
-   [ ] Auto-Tiering vs Lifecycle
-   [ ] Versioning vs Retention
-   [ ] PAR
-   [ ] Backup vs Clone vs Replication
-   [ ] Mount Target / Export / Export Set
-   [ ] Encryption and customer-managed keys
-   [ ] HA vs DR
-   [ ] RTO vs RPO

------------------------------------------------------------------------

# 31. Método para resolver preguntas OCI

Cuando leas una pregunta:

## Paso 1 --- Encuentra el requisito dominante

Busca palabras como:

-   private
-   public
-   Internet
-   on-premises
-   dedicated
-   encrypted
-   same region
-   different region
-   shared
-   NFS
-   lowest cost
-   immediate access
-   compliance
-   temporary
-   automatic
-   highly available

## Paso 2 --- Identifica la capa

¿La pregunta es de:

-   IAM
-   Network
-   Compute
-   Storage
-   Availability
-   Security
-   Cost

## Paso 3 --- Elimina respuestas de otra capa

Ejemplo:

Una **NSG** no reemplaza una **Route Table**.

-   NSG = permission/firewall.
-   Route Table = path.

## Paso 4 --- Busca la solución administrada nativa de OCI

En preguntas de arquitectura, si OCI ofrece una función administrada que
satisface exactamente el requisito, normalmente es mejor candidata que
construir manualmente la misma función en VMs.

## Paso 5 --- Comprueba costo y complejidad

Si dos opciones funcionan, revisa si la pregunta pide:

-   lowest cost
-   least operational overhead
-   highest availability
-   lowest latency
-   no public Internet
-   minimum administration

Eso suele decidir la respuesta.

------------------------------------------------------------------------

# 32. Hoja de memoria de 2 minutos

``` text
IAM
Users → Groups
Resources → Dynamic Groups
VM calling OCI → Instance Principal
inspect < read < use < manage

NETWORK
Public Internet → IGW
Private → Internet outbound → NAT
Private → OCI service → Service Gateway
Hybrid/multi-VCN router → DRG
On-prem encrypted Internet → VPN
On-prem private dedicated → FastConnect
Subnet firewall → Security List
Resource/VNIC firewall → NSG

COMPUTE
Template → Instance Configuration
Fleet → Instance Pool
CPU threshold → Metric Autoscaling
Known time → Scheduled Autoscaling
Interruptible → Preemptible
Guaranteed capacity → Capacity Reservation

STORAGE
Disk → Block
Shared NFS → File
Objects → Object Storage
Hot → Standard
Cool immediate → Infrequent Access (31d)
Cold restore → Archive (90d)
Temporary anonymous-style scoped URL → PAR
Old object copies → Versioning
Compliance lock → Retention
Automatic access-pattern tier → Auto-Tiering

AVAILABILITY
Hardware isolation → Fault Domains
Datacenter isolation → Availability Domains
Regional disaster → Cross-region DR
```

------------------------------------------------------------------------

# 33. Fuentes oficiales recomendadas

Esta guía fue construida tomando como referencia el material de Oracle
MyLearn/eKit proporcionado para el curso y contrastando conceptos con
documentación oficial actual de OCI.

Documentación oficial para profundizar:

-   OCI Documentation:
    https://docs.oracle.com/en-us/iaas/Content/home.htm
-   IAM Overview:
    https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/overview.htm
-   Networking Overview:
    https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/overview.htm
-   Security Lists:
    https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm
-   DRG:
    https://docs.oracle.com/en-us/iaas/Content/Network/Tasks/manage-drg.htm
-   Compute Overview:
    https://docs.oracle.com/en-us/iaas/Content/Compute/Concepts/computeoverview.htm
-   Object Storage: https://docs.oracle.com/en-us/iaas/Content/Object/
-   Object Storage Tiers:
    https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm
-   Block Volume Overview:
    https://docs.oracle.com/en-us/iaas/Content/Block/Concepts/overview.htm
-   File Storage Overview:
    https://docs.oracle.com/en-us/iaas/Content/File/Concepts/filestorageoverview.htm
-   Load Balancer: https://docs.oracle.com/en-us/iaas/Content/Balance/

------------------------------------------------------------------------

# 34. Última recomendación de estudio

Prioridad sugerida:

1.  **Networking** --- domínalo a nivel de escenarios.
2.  **IAM** --- especialmente policies, compartments, dynamic groups e
    instance principals.
3.  **Storage** --- diferencias y selección del servicio/tier.
4.  **Compute** --- shapes, pools, scaling, capacity y HA.
5.  **Arquitectura completa** --- combina Networking + Compute +
    Storage + IAM.
6.  Haz el simulacro sin apuntes.
7.  Revisa cada error explicando por qué las otras tres opciones no
    cumplen el requisito.

La meta no es recordar una definición. La meta es leer:

> "private subnet, no public IP, needs Object Storage"

y responder casi automáticamente:

> **Service Gateway**

o leer:

> "private subnet, no public IP, needs public Internet repository"

y responder:

> **NAT Gateway**

Cuando esas asociaciones sean automáticas, estarás resolviendo las
preguntas como arquitecto y no solo memorizando nombres.
