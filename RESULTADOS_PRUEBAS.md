# Resultado de verificación — versión 2

Fecha: 12 de octubre de 2026 (Costa Rica).

| Comprobación | Resultado |
|---|---|
| Banco: 150 identificadores y escenarios únicos | Aprobado |
| 30 casos de selección múltiple; 14 áreas; cuatro opciones y cuatro justificaciones por caso | Aprobado |
| Campos de conceptos, diferencias, pistas, consejos y puntos por verificar | Aprobado |
| Puntuación exacta de selecciones múltiples y registro de errores | Aprobado (pruebas `node --test`) |
| Práctica: explicación solo después de enviar | Aprobado en Chrome |
| Navegación, guardado, recarga, reanudación y repetición de falladas | Aprobado en Chrome |
| Examen: sin puntuación ni explicaciones antes del final, incluso al volver al inicio | Aprobado en Chrome |
| Sesión completa de 150 casos y 150 entradas de revisión final | Aprobado en Chrome |
| Anchos 320, 390 y 430 px: sin desplazamiento horizontal; botones y opciones de al menos 44 px | Aprobado en Chrome |
| Enlaces a documentación oficial de Oracle | 31/31 fuentes del banco respondieron HTTP 200 |
| Archivos estáticos, rutas relativas y caché de versión actualizada | Aprobado |

Las pruebas de navegador incluidas en `tests/browser.test.mjs` están listas para ejecutarse con Playwright en un entorno local normal. En este entorno restringido, el navegador headless iniciado desde la terminal se cerró por permisos del sistema; por eso la interacción de navegador se ejecutó mediante la sesión controlada de Chrome. Las pruebas del banco y la lógica sí se ejecutaron desde la terminal y pasaron.

Límites: no hubo un iPhone físico disponible; la revisión de 320/390/430 px se hizo con viewport móvil en Chrome. La disponibilidad regional, límites, costes, licencias y RPO/RTO concretos se deben confirmar en la tenancy objetivo, como detalla `VERIFICACIONES.md`.
