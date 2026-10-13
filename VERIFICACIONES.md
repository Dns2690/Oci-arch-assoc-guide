# Verificación técnica y límites

Los escenarios se redactaron como decisiones de arquitectura y se contrastaron con páginas de documentación oficial de Oracle. Cada pregunta enlaza la página principal que fundamenta el servicio o patrón decisivo. La fuente no convierte el escenario en una garantía de configuración válida para cualquier tenancy.

## Aspectos que requieren comprobación al implementar

- **Disponibilidad por región, availability domain y shape:** comprueba la región concreta, los límites de servicio y la capacidad real. Una capacity reservation es específica de AD/shape compatible y puede generar cargos por capacidad ociosa.
- **Precios y clases de almacenamiento:** revisa tarifas vigentes, periodos mínimos y cargos de recuperación de Infrequent Access y Archive. No se citan importes en las preguntas porque cambian.
- **Auto-Tiering:** verifica elegibilidad de objetos, permisos y posible incompatibilidad con reglas lifecycle de transición a Infrequent Access.
- **Migración:** confirma matriz de fuentes/destinos, versiones, licencias, requisitos de red y ventanas de corte para Oracle Cloud Migrations y Database Migration.
- **RPO/RTO y réplica:** son objetivos del negocio, no garantías automáticas del servicio. Hay que medir desfase, failover y recuperación de la pila completa.
- **Seguridad:** verifica las operaciones IAM exactas, el alcance del compartimento, certificados, rutas y reglas de red antes de desplegar.
- **Servicios administrados de base de datos:** valida compatibilidad de aplicación, conexiones, características y modalidad concreta de Autonomous Database o Data Guard.

## Referencias oficiales principales

- [IAM y dynamic groups](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingdynamicgroups.htm)
- [Red privada y Service Gateway](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/privateaccess.htm)
- [Load Balancer](https://docs.oracle.com/en-us/iaas/Content/Balance/Tasks/managingloadbalancer_topic-Creating_Load_Balancers.htm) y [Network Load Balancer](https://docs.oracle.com/en-us/iaas/Content/NetworkLoadBalancer/overview.htm)
- [Autoscaling](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm) y [capacity reservations](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/reserve-capacity.htm)
- [Object Storage tiers](https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/understandingstoragetiers.htm) y [File Storage replication](https://docs.oracle.com/en-us/iaas/Content/File/Tasks/FSreplication.htm)
- [Monitoring](https://docs.oracle.com/en-us/iaas/Content/Monitoring/), [Cloud Guard](https://docs.oracle.com/en-us/iaas/Content/cloud-guard/) y [Cost Analysis](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costanalysisoverview.htm)
- [Oracle Cloud Migrations](https://docs.oracle.com/en-us/iaas/Content/cloud-migration/cloud-migration-overview.htm) y [Database Migration](https://docs.oracle.com/en-us/iaas/database-migration/doc/overview-oracle-cloud-infrastructure-database-migration.html)
