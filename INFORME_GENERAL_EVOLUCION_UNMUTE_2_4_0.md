# UNMUTE — Informe general de evolución, cambios e implementación
## Desde el diario emocional inicial hasta la versión 2.4.0

**Versión actual del proyecto:** 2.4.0  
**Último bloque integrado:** Bloque 6 — Administración, Privacidad y Accesibilidad  
**Lema:** «Tu voz también merece ser escuchada».

---

## 1. Resumen ejecutivo

UNMUTE es un diario emocional digital que ayuda a las personas a reconocer, explorar, comprender y regular mejor sus emociones. Su función principal es ofrecer un espacio personal para escribir, registrar emociones, consultar la propia historia y utilizar herramientas voluntarias de pausa y acompañamiento.

El proyecto evolucionó desde una aplicación web ligera construida con HTML, CSS y JavaScript hacia una arquitectura full stack que incorpora funciones de servidor, Supabase, PostgreSQL, autenticación administrativa, catálogo de recursos, noticias, contacto con seguimiento e integración de OpenAI.

La evolución conserva cuatro decisiones fundamentales:

1. El diario emocional es el núcleo de UNMUTE.
2. La inteligencia artificial es complementaria y voluntaria.
3. Las entradas del diario permanecen en el navegador y no se envían automáticamente a Supabase ni a OpenAI.
4. Las nuevas funciones deben respetar la identidad visual Calma orgánica, la autonomía de la persona y la privacidad.

La versión 2.4.0 reúne seis bloques de desarrollo después de la etapa inicial de evolución y estabilización.

## 2. Propósito e identidad del producto

### 2.1. Definición

UNMUTE es un diario emocional digital que permite:
- Escribir libremente y guardar entradas personales.
- Registrar emociones e intensidad de forma opcional.
- Consultar y organizar el historial propio.
- Realizar actividades voluntarias de calma.
- Recibir acompañamiento conversacional si la persona decide utilizarlo.
- Consultar datos educativos, noticias y recursos.
- Encontrar recursos humanos de ayuda.
- Contactar al equipo y consultar el estado de una solicitud.

### 2.2. Identidad visual

La línea visual **Calma orgánica** utiliza tonos suaves, componentes redondeados, espacios amplios, tipografía legible y animaciones moderadas. La experiencia incluye tema claro, oscuro y del sistema, tamaño del texto y preferencias de movimiento.

El lema se mantiene: «Tu voz también merece ser escuchada».

### 2.3. Principios de privacidad y autonomía

- El diario permanece en el almacenamiento local del navegador (`localStorage`).
- El diario no se adjunta automáticamente al chat de IA ni a los mensajes de contacto.
- La persona decide si envía mensajes a Acompañar y debe aceptar expresamente el aviso.
- La persona puede detener o abandonar las actividades de Pausa.
- El acceso a ayuda humana no se sustituye por respuestas de IA.
- La empresa o el administrador de UNMUTE no dispone de una vista del diario privado.

---

## 3. Evolución funcional y versiones

### 3.1. Primera serie de evolución: versiones 0.1.0 a 0.5.0

Esta serie organiza el crecimiento de la experiencia inicial en cinco etapas funcionales.

| Versión | Nombre | Incorporaciones principales | Motivo |
|---|---|---|---|
| 0.1.0 | Concepto y presentación | Identidad, lema, explicación de la propuesta y primera maqueta del registro. | Comunicar qué es UNMUTE antes de añadir interacción. |
| 0.2.0 | Navegación y formulario | Navegación entre Inicio y Registrar, emoción, intensidad, nota opcional y validación básica. | Probar el flujo de registro. |
| 0.3.0 | Diario interactivo | Creación de entradas e historial durante la sesión. | Hacer funcional el recorrido principal. |
| 0.4.0 | Persistencia local | `localStorage`, borradores, edición, eliminación y aviso de privacidad. | Dar continuidad al diario sin servidor. |
| 0.5.0 | Consulta y respaldo | Búsqueda, filtro por emoción, indicadores, exportación/importación JSON y eliminación local. | Facilitar la consulta y gestión de datos propios. |

### 3.2. Versión web 1.0.0

La edición web 1.0.0 amplía el diario con:
- Presentación visual más elaborada y navegación mejor organizada.
- Registro en pasos de emoción, intensidad y texto.
- Guardado y recuperación de borradores locales.
- Historial con búsqueda y filtros por emoción, intensidad y fechas.
- Orden temporal y presentación progresiva de resultados.
- Estadísticas de registros, emoción frecuente e intensidad promedio.
- Gráfica SVG del recorrido emocional y una alternativa tabular.
- Mensajes locales de apoyo.
- Exportación/importación JSON y eliminación de datos.
- Información de privacidad y mejoras de accesibilidad.

La edición mantiene el almacenamiento local; no incorpora por sí misma una base de datos remota ni sincronización entre dispositivos.

### 3.3. Calma orgánica + IA 1.0.0

Esta edición incorpora la primera integración conversacional de IA:
- Vista Acompañar.
- Consentimiento previo antes del envío de mensajes.
- Servidor Node.js con integración OpenAI.
- Clave de API en variables de entorno del servidor, no en el navegador.
- Contexto conversacional limitado.
- Conversación temporal en memoria de la página.
- `store: false` en la solicitud al proveedor.
- Validación, límites básicos, cabeceras de seguridad y renderizado de mensajes como texto.

La IA se presenta como acompañamiento voluntario, no como terapeuta ni servicio de emergencias.

### 3.4. Calma orgánica + IA 2.0.0 — Base full stack

La arquitectura se organiza alrededor de:
- Frontend estático en `public/`.
- Netlify Functions para las APIs.
- Supabase y PostgreSQL para contenidos y mensajes.
- Supabase Auth para acceso administrativo.
- RLS y validaciones de servidor.
- OpenAI invocado desde el backend.
- `server.js` como servidor local de desarrollo.

También quedan preparadas las APIs de recursos, datos curiosos, noticias, contacto, autenticación y administración. La base mantiene el diario local y no lo migra a Supabase.

---

## 4. Desarrollo por bloques

### Bloque 1 — Versión 2.0.1: consentimiento y límites de uso

**Objetivos:** corregir el consentimiento real de Acompañar y asegurar que las operaciones protegidas no continúen sin un control persistente de uso.

**Implementaciones:**
- `public/js/ai-chat.js` envía el estado verdadero de la casilla `#aiConsent`.
- Sin consentimiento no se envía el mensaje; el backend sigue exigiendo `consent === true`.
- El rate limit utiliza Supabase mediante la función SQL compartida `consume_rate_limit`.
- Si Supabase falta, falla o el RPC responde incorrectamente, las operaciones protegidas devuelven HTTP 503 `rate_limit_unavailable`.
- El contador en memoria solo puede habilitarse expresamente fuera de Netlify para desarrollo local.
- El endpoint health diferencia modo `database`, `unavailable` y `memory`.

**Base de datos:** no se añade migración.

**Pruebas registradas:** 47/47 pruebas automatizadas aprobadas y semilla correcta. Los comandos `check` y `e2e` estaban declarados pero sus archivos no venían en ese paquete.

### Bloque 2 — Versión 2.0.2: protección de recursos y acceso administrativo

**Objetivos:** retirar la semilla JSON de la carpeta pública, completar una primera página de acceso administrativo y disponer de una comprobación estática.

**Implementaciones:**
- La semilla pasa de `public/data/resources-seed.json` a `src/data/resources-seed.json`.
- El generador de migración se actualiza a la nueva ubicación.
- Se crea `public/admin.html`, con login, verificación de sesión, resumen y cierre de sesión.
- Se añaden `public/js/admin.js` y `public/css/admin.css`.
- `/admin`, `/admin/` y `/admin.html` sirven la página de administración.
- Se añaden cabeceras `noindex` y `no-store` para las rutas administrativas.
- Se incorpora `scripts/check.mjs` y el comando `npm run check`.

**Seguridad:** la verificación de permisos se mantiene en el servidor; no se crea otra autenticación ni se exponen claves privadas.

**Pruebas registradas:** 61/61 pruebas aprobadas, 40 comprobaciones estáticas correctas y semilla correcta. `npm run e2e` continuó pendiente porque no estaba incluido el archivo de prueba.

### Bloque 3 — Versión 2.1.0: datos curiosos y noticias

**Objetivos:** permitir la administración de datos curiosos y crear el sistema de noticias públicas y administrativas.

**Datos curiosos:**
- Gestión desde administración: listar, buscar, filtrar, crear, editar, guardar borrador, publicar, despublicar y eliminar.
- Requisito editorial de fuente y fecha de revisión para publicar.
- Tarjeta «Un minuto para aprender» conectada a `GET /api/facts/today`.

**Noticias:**
- Nueva tabla `news`.
- Migración `003_news.sql` con restricciones, trigger de fecha de publicación y RLS.
- API pública `GET /api/news` y `GET /api/news/:slug`, limitada a noticias publicadas.
- API administrativa protegida para crear, editar, publicar, despublicar y eliminar.
- Pestaña «Noticias del admin» y sección pública «Noticias».
- Renderizado de texto sin HTML y validación de URLs de imagen HTTPS.

**Decisión de datos:** se utiliza tabla `news` independiente de `resources`, porque la estructura de recursos no tenía los campos ni el tipo adecuados para gestionar publicaciones.

**Pruebas registradas:** 70/70 pruebas con servicios simulados. La migración y sus políticas debían verificarse en PostgreSQL/Supabase real.

### Bloque 4 — Versión 2.2.0: Inicio, Recursos y navegación

**Objetivos:** reorganizar Inicio y ofrecer una vista pública del catálogo.

**Inicio:**
- Dos tarjetas independientes: «Un minuto para aprender» y «Noticias de UNMUTE».
- Dato y noticia seleccionados aleatoriamente entre los contenidos publicados.
- Acciones «Ver otro dato» y «Ver otra noticia».
- Estados independientes de carga, vacío, error y reintento.
- Dos columnas en escritorio y tarjetas apiladas en móvil.

**Recursos:**
- Nueva vista pública conectada a `GET /api/resources`.
- Tarjetas, búsqueda y filtros por categorías disponibles.
- Exclusión de datos curiosos del catálogo general.
- Estados vacíos y errores.
- Botones únicamente cuando existe un destino real y seguro.

**Navegación:**
- Cinco accesos principales: Inicio, Diario, Acompañar, Historia y Más.
- Menú Más: Pausa, Noticias, Recursos y Ajustes.
- Mejoras de teclado, Escape, foco y navegación Atrás/Adelante.

**Pruebas registradas:** 74/74 pruebas automatizadas, 44 comprobaciones estáticas correctas y semilla al día. El changelog registra 76/76 comprobaciones temporales en Chromium, ejecutadas con servicios de contenido simulados; ese script no quedó dentro del proyecto.

**Hallazgo de contenido:** el recurso `biblioteca-informacion` tenía un destino interno que no existía. La interfaz evita mostrar un botón roto y señala que no está disponible; el recurso requiere una decisión editorial para corregir destino o despublicar.

### Bloque 5 — Versión 2.3.0: Necesito ayuda y Contacto

**Objetivos:** crear el directorio de ayuda y completar el envío y seguimiento de mensajes de contacto.

**Necesito ayuda:**
- Vista secundaria desde Más.
- Líneas 123, 106 de Bogotá, 141 y 155 tomadas del directorio colombiano existente.
- Cobertura, descripción, horario, teléfono, WhatsApp cuando está definido, fuente y fecha de verificación.
- Enlaces `tel:` y WhatsApp.
- Sin GPS, geolocalización, IP ni ubicación precisa.
- Accesos a Pausa y Acompañar con aclaraciones sobre sus límites.

**Contacto:**
- Formulario con nombre y correo opcionales, categoría, mensaje y consentimiento obligatorio.
- `POST /api/contact`.
- La referencia se presenta solo si Supabase confirmó la inserción y el servidor devolvió el UUID real.
- Antispam, validación y control de envíos simultáneos.

**Seguimiento:**
- Nueva ruta `GET /api/contact/status?reference=UUID`.
- Respuesta pública limitada a estado y fecha de recepción.
- Traducción de estados internos: `new`, `read`, `resolved`, `archived` a Recibido, En revisión, Respondido y Cerrado.
- Rate limit persistente específico para el seguimiento.
- El UUID no da acceso al contenido ni a los datos personales del mensaje.

**Administración:**
- Nueva sección «Mensajes de contacto» en el panel existente.
- Listar, filtrar, leer detalle y cambiar estados.
- Sin borrado de mensajes y sin una segunda autenticación.

**Base de datos:** no se añade migración; se reutiliza `contact_messages`, sus estados y el mecanismo de rate limit existente.

**Pruebas registradas:** 93/93 pruebas automatizadas con servicios simulados, 45 comprobaciones estáticas correctas y semilla al día. Falta reproducir el flujo completo con Supabase real y Netlify desplegado.

### Bloque 6 — Versión 2.4.0: Administración, Privacidad y Accesibilidad

**Objetivos:** mejorar la claridad de los ajustes y la robustez/usabilidad del panel administrativo.

**Ajustes:**
- Se separan Tema visual, Accesibilidad, Concentración y escritura, Privacidad y Tus datos.
- Accesibilidad reúne tamaño del texto, reducir animaciones y desactivar efectos decorativos.
- El modo sin distracciones queda en Concentración y escritura.
- Privacidad explica los flujos reales del diario local, Acompañar, Contacto, copias y eliminación.
- Se conserva el diálogo único `dlgPriv`.
- Se preservan `unmute_v2_prefs`, `PREF0` y los identificadores de controles; `public/js/app.js` no se modifica.

**Administración:**
- Se corrige el estado de botones que podía permanecer bloqueado después de un cierre o vencimiento de sesión durante una operación.
- Se limpian del DOM el resumen, datos de sesión y diálogo de confirmación al salir.
- Se bloquean operaciones simultáneas dentro de cada sección.
- Se unifican estados de carga, guardado, éxito, error y vacío.
- Se refuerza la gestión del foco después de cambios de sección, guardados, publicaciones, eliminaciones y cancelaciones.
- Los paneles administrativos utilizan secciones con nombre accesible.
- Se ajustan acciones y filtros para móvil.

**Sin cambios:** autenticación, roles, RLS, endpoints, backend, tablas y migraciones. No se añade retención, registro del consentimiento de Contacto ni borrado remoto.

**Pruebas reproducidas en el paquete 2.4.0:** 108/108 pruebas aprobadas; 45 comprobaciones estáticas correctas, sin errores y una advertencia; semilla correcta. Las pruebas nuevas del Bloque 6 son estáticas. No hubo auditoría visual ni uso de lector de pantalla en esta revisión.

---

## 5. Arquitectura e implementación actual

### 5.1. Frontend

`public/` contiene:
- `index.html`: vistas públicas.
- `admin.html`: entrada al panel administrativo.
- `css/styles.css` y `css/admin.css`: diseño y estilos.
- `js/app.js`: navegación, diario, historial, Pausa, preferencias y vistas generales.
- `js/ai-chat.js`: consentimiento y conversación con IA.
- `js/content.js`: tarjetas de datos curiosos y noticias.
- `js/resources.js`: vista pública de Recursos.
- `js/help-directory.js` y `js/help.js`: datos y vista del directorio de ayuda.
- `js/contact.js`: formulario y seguimiento.
- `js/admin.js`: panel administrativo y sus secciones.
- `js/theme-init.js`: inicialización de tema.

### 5.2. Backend

`netlify/functions/` contiene funciones para:
- Salud del servicio.
- Chat con IA.
- Catálogo de recursos.
- Contacto.
- Seguimiento de contacto.
- Autenticación administrativa.
- Gestión de recursos/datos curiosos.
- Gestión de mensajes.
- Noticias públicas y administrativas.

`server.js` facilita la ejecución local y el enrutamiento de funciones durante el desarrollo. El despliegue previsto usa Netlify Functions.

### 5.3. Servicios y librerías

`src/services/` centraliza servicios de:
- IA.
- Autenticación.
- Recursos.
- Contacto.
- Noticias.

`src/lib/` centraliza:
- Comunicación HTTP.
- Manejo de errores.
- Validación.
- Cliente de Supabase.
- Rate limit persistente.

`src/config/environment.mjs` concentra la configuración de entorno.

### 5.4. Base de datos

Las migraciones incluidas son:
- `001_initial_schema.sql`: esquema base, perfiles, recursos, mensajes de contacto, límites y RLS.
- `002_seed_resources.sql`: contenido inicial del catálogo.
- `003_news.sql`: tabla de noticias, restricciones, trigger de publicación y políticas RLS.

La semilla fuente está en `src/data/resources-seed.json`, fuera de `public/`, porque incluye borradores.

### 5.5. Integración con OpenAI

La API de OpenAI se llama desde el backend. La clave se configura como variable privada y el nombre del modelo mediante `OPENAI_MODEL`.

Acompañar:
- Requiere consentimiento.
- Envía los mensajes que la persona decide compartir y contexto limitado de esa conversación.
- No recibe automáticamente el diario.
- Utiliza `store: false`, que no debe describirse como garantía de retención cero del proveedor.

### 5.6. Almacenamiento local

El diario, borradores, emociones, etiquetas y preferencias permanecen en el navegador mediante `localStorage`. El almacenamiento local no está cifrado ni sincronizado y otras personas con acceso al mismo perfil del navegador podrían ver los datos.

La exportación JSON ofrece una copia que la persona debe guardar. La eliminación desde Ajustes borra los datos locales de UNMUTE, no los mensajes de Contacto guardados en Supabase.

---

## 6. Mapa de funcionalidades de la versión 2.4.0

| Área | Implementación actual | Estado de evidencia |
|---|---|---|
| Inicio | Bienvenida, dos tarjetas aleatorias de datos y noticias | Código y pruebas automatizadas |
| Diario | Entradas, emociones, intensidad, borradores y edición local | Código integrado; almacenamiento local |
| Historia | Historial, búsqueda, filtros, favoritos y edición | Código integrado |
| Pausa | Respiración, orientación sensorial, escritura y temporizador | Código integrado |
| Acompañar | Chat con consentimiento y backend OpenAI | Código y pruebas simuladas; validación real reportada por el propietario en una etapa previa |
| Noticias | Lectura pública y administración de noticias | Código, endpoints y pruebas simuladas |
| Recursos | Catálogo público con búsqueda y filtros | Código y pruebas automatizadas |
| Necesito ayuda | Directorio de Colombia y accesos a herramientas | Código y pruebas simuladas/estáticas |
| Contacto | Formulario y seguimiento por UUID | Código y pruebas simuladas |
| Administración | Login, resumen, datos curiosos, noticias y mensajes | Código y pruebas simuladas |
| Privacidad | Explicación de datos, flujos y límites | Texto revisado estáticamente |
| Accesibilidad | Controles, semántica, foco y responsive | Revisado estáticamente; prueba manual pendiente |

---

## 7. Evolución de las pruebas

| Versión | `npm test` | `npm run check` | Semilla | E2E |
|---|---:|---|---|---|
| 2.0.1 · Bloque 1 | 47/47 | Archivo ausente en ese paquete | Correcta | No disponible |
| 2.0.2 · Bloque 2 | 61/61 | 40 correctas | Correcta | No disponible |
| 2.1.0 · Bloque 3 | 70/70 | 44 correctas, 0 errores, 1 advertencia | Correcta | No ejecutado |
| 2.2.0 · Bloque 4 | 74/74 | 44 correctas, 0 errores, 1 advertencia | Correcta | No ejecutado; prueba temporal Chromium reportada |
| 2.3.0 · Bloque 5 | 93/93 | 45 correctas, 0 errores, 1 advertencia | Correcta | No ejecutado |
| 2.4.0 · Bloque 6 | 108/108 | 45 correctas, 0 errores, 1 advertencia | Correcta | No ejecutado |

Las pruebas automatizadas usan mocks y comprobaciones estáticas; no equivalen a certificación de Supabase, OpenAI o Netlify en producción.

En una etapa previa, el propietario indicó que había probado la base de datos, OpenAI y Netlify y que funcionaban correctamente. Esa validación es un antecedente reportado por el propietario; no se reproduce ni se acredita con registros de servicio real dentro del ZIP 2.4.0.

---

## 8. Pendientes y riesgos conocidos

1. **Prueba E2E reproducible:** `package.json` declara `npm run e2e`, pero `tests/e2e-smoke.mjs` no está incluido.
2. **Accesibilidad manual:** falta revisar contraste real, foco, escalado del texto, responsive y compatibilidad con NVDA, VoiceOver o TalkBack. No se declara conformidad WCAG.
3. **Privacidad y retención:** no hay política implementada de conservación/eliminación de mensajes de Contacto ni registro independiente del consentimiento en la base de datos.
4. **Sesión administrativa:** el token utiliza `sessionStorage`; no se implementa renovación automática ni cookie `HttpOnly`.
5. **Contenido de Recursos:** el recurso `biblioteca-informacion` fue identificado con destino interno inexistente en el Bloque 4 y se mantuvo sin botón. No hay en los bloques siguientes confirmación de que ese destino se haya corregido o el recurso despublicado.
6. **Migraciones en entorno real:** el proyecto contiene migraciones 001–003; el ZIP no certifica por sí mismo cuáles están aplicadas en la instancia desplegada.
7. **Pruebas visuales de las nuevas vistas:** se recomienda revisar Necesito ayuda, Contacto, Ajustes y el panel en dispositivos reales.
8. **Producción:** revisar variables privadas, RLS, límites, CSP, política de privacidad, incidentes y costos de OpenAI en el entorno final.

---

## 9. Recomendaciones para la sustentación

Una presentación clara puede seguir este orden:

1. **Problema y propósito:** explicar qué necesidad emocional aborda UNMUTE y por qué el diario es el núcleo.
2. **Experiencia personal:** demostrar el registro emocional, una entrada, el historial y la privacidad local.
3. **Herramientas complementarias:** mostrar Pausa, Acompañar, Recursos y Necesito ayuda.
4. **Contenido administrable:** demostrar datos curiosos y noticias públicas.
5. **Comunicación:** explicar el formulario de Contacto, la referencia y el seguimiento.
6. **Administración:** mostrar autenticación, roles, gestión de contenidos y mensajes.
7. **Arquitectura:** explicar frontend, Netlify Functions, Supabase, OpenAI y almacenamiento local.
8. **Privacidad y accesibilidad:** mostrar los apartados de Ajustes y explicar qué información ve cada servicio.
9. **Calidad:** mostrar pruebas automatizadas y distinguir mocks, revisiones estáticas y validaciones reales.

La demostración debe utilizar datos de prueba y evitar mostrar diarios personales, credenciales o mensajes privados reales.

---

## 10. Conclusión

UNMUTE 2.4.0 reúne un diario emocional local, herramientas voluntarias de calma, acompañamiento mediante IA, recursos y noticias, un directorio de ayuda para Colombia, contacto con seguimiento y un panel administrativo protegido.

Los seis bloques ampliaron la aplicación por partes y conservaron el diario como su centro. La arquitectura separa el contenido local del usuario de los servicios que requieren procesamiento externo. Las pruebas automatizadas y estáticas han crecido hasta 108 casos aprobados en la versión 2.4.0, aunque siguen pendientes una prueba E2E reproducible y una revisión manual completa de accesibilidad.

UNMUTE está técnicamente avanzado y dispone de una base defendible para la sustentación. La presentación debe comunicar con claridad lo que está implementado, qué se ha probado con simulaciones, qué se comprobó en servicios reales y qué queda como mejora futura.

**UNMUTE — Tu voz también merece ser escuchada.**
