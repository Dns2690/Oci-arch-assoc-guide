# OCI Architect Associate — Simulador de escenarios, versión 2

Simulador estático con **150 casos empresariales**, 30 de selección múltiple, modo práctica, modo examen, revisión de errores y progreso local. Las cuatro opciones de cada caso y los nombres de servicios y funciones OCI se presentan en **inglés**, como en el examen; los escenarios, razonamientos y consejos están redactados en español. Las respuestas se muestran al enviar en práctica; en examen solo al terminar. Cada alternativa tiene su razonamiento, y cada caso incluye conceptos, diferencias, pistas, consejos y un enlace a documentación oficial de Oracle.

## Actualizar GitHub Pages

Descomprime el ZIP y sube **el contenido de esta carpeta** a la raíz del repositorio existente, reemplazando los archivos con el mismo nombre. Conserva `.nojekyll`. En GitHub, configura Pages desde la rama principal y la raíz. Abre la URL HTTPS en Safari. Si estaba instalada en la pantalla de inicio, vuelve a abrirla con conexión para que el Service Worker cargue la nueva versión.

Todos los recursos de la aplicación usan rutas relativas y no requieren servidor de aplicaciones ni proceso de compilación. El progreso se guarda en el navegador y dispositivo actuales. La sesión del banco anterior (36 preguntas) queda en la clave local `oci-session-v2`; la versión 2 usa `oci-session-v3` porque sus preguntas e identificadores cambiaron.

## Pruebas

`npm install` y `npm test` ejecutan la validación del banco y pruebas de navegación, botones, puntuación y persistencia con Playwright. Si faltan navegadores, ejecuta `npx playwright install chromium webkit`.

Material educativo independiente, no oficial ni garantía de aprobación. Consulta `VERIFICACIONES.md` antes de aplicar diseños a una tenancy real.
