# Proyecto 1 — Backend: Sistema de reservas con pagos

> Documento pensado para dos usos: (1) guía personal de ejecución y (2) contexto/prompt para una IA que te acompañe como mentor. La sección "Instrucciones para la IA" está lista para pegar.

---

## 0. Instrucciones para la IA (pegar al inicio de cada sesión)

```text
Actuá como mentor senior de backend. Estoy construyendo el proyecto descrito en este documento
para aprender y armar portafolio para un primer empleo (backend junior).

Reglas:
1. No me des el proyecto resuelto. Explicame el concepto, dame el camino y revisá lo que escribo.
2. Cuando te pida código, dame lo mínimo necesario y explicá por qué se hace así.
3. Antes de cerrar cada fase, hacéme 5 preguntas tipo entrevista sobre lo que construí.
4. Si mi diseño tiene un problema (concurrencia, seguridad, modelado), decímelo directamente
   y explicá la alternativa y su trade-off.
5. Respetá el alcance de la fase actual. Si propongo agregar cosas fuera de alcance, recordámelo.
6. Si algo depende de una versión o de documentación externa (Mercado Pago, librerías),
   decime que lo verifique en la documentación oficial en lugar de asumir.
7. Respondé en español.

Fase actual: [COMPLETAR]
Lo que ya tengo hecho: [COMPLETAR]
Mi duda concreta: [COMPLETAR]
```

---

## 0.5 Contexto del proyecto (pegar en `docs/GUIDE.md`, después de la sección 0)

> Esto es **contexto**, no una lista de tareas. No implementar nada por leerlo.
> Lo marcado como "propuesta" lo decido yo en S1 y queda registrado como ADR (ADR-003). Si algo choca con `CLAUDE.md` o `ROADMAP.md`, mandan ellos.

### Idea en una frase

Backend de **reservas de canchas de pádel para un club** (el modelo es idéntico para fútbol 5), con pago online, que no permite dobles reservas ni cobros duplicados aunque haya concurrencia, reintentos del cliente y webhooks repetidos.

### Por qué este tema

- Se entiende en 30 segundos y todos conocen el problema: "dos personas quieren la cancha del viernes a las 21:00".
- La concurrencia es **natural**, no forzada: el test de "N personas reservan el mismo turno" tiene sentido real.
- `pending_payment` se explica solo ("te guardo la cancha 10 minutos mientras pagás"). De ahí salen la expiración, los pagos tardíos y los webhooks fuera de orden.
- Es fácil conseguir usuarios reales: un club o un grupo que juega todas las semanas.

### Quién usa el sistema

- **Jugador (`customer`)**: ve disponibilidad, reserva, paga, cancela, ve sus reservas.
- **Admin del club (`admin`)**: gestiona canchas y precios, bloquea turnos por mantenimiento, ve todas las reservas.
- **Worker (sistema)**: expira reservas sin pagar.
- **Proveedor de pagos (externo)**: confirma pagos por webhook.

### Dominio concreto (propuesta)

- Un solo club, N canchas (`resources`). Sin multi-sede.
- Turnos de **duración fija por cancha** (60 o 90 min). En el MVP, una reserva = un turno. La base guarda igual un rango de tiempo, así que pasar a rangos libres más adelante no obliga a rehacer el modelo.
- Horario de atención por cancha. Todo se guarda en UTC; la grilla de turnos se calcula en la **hora local del club**.
- El precio se congela en la reserva (`total_cents`).
- Una reserva `pending_payment` retiene el turno unos minutos (por ejemplo 10) y después expira.
- Decisión abierta: ¿qué pasa al cancelar una reserva ya confirmada? Implica reembolso, que está fuera del MVP → decidir en un ADR.

### Los 5 problemas difíciles que el proyecto debe demostrar

Esto es lo que lo diferencia. Cada uno tiene que tener código, test y explicación en el README.

1. **Doble reserva bajo concurrencia** → constraint de exclusión en la base + test (S4, S7).
2. **Reintentos del cliente** → `Idempotency-Key` (S5).
3. **Webhooks duplicados, desordenados o tardíos** → tabla de eventos, procesamiento idempotente, consulta del estado real al proveedor (S6).
4. **Expiración segura y pago tardío** → job que tolera correr dos veces; decisión documentada sobre el pago que llega después de expirar (S7, S8).
5. **Consistencia entre mi estado y el del proveedor** → qué pasa si cae el proveedor, si el pago queda pendiente, si el webhook nunca llega.

### Cómo destacar: evidencia, no features

- Tests de concurrencia y de webhooks corriendo en CI.
- Una **prueba de carga corta** (verificar qué herramienta usar) con resultados reales documentados: cuántas solicitudes simultáneas al mismo turno, cuántas ganan, cuántas se rechazan y con qué código, latencia.
- ADRs honestos, con las alternativas que descarté y por qué.
- Sección **"Failure modes"** en el README: qué pasa si cae el proveedor, si el worker corre dos veces, si el webhook llega antes que el pago.
- Usuarios reales aunque sean pocos, y lo que aprendí de ellos.

### Extensiones con identidad propia (solo después de `v1.0.0`)

Orden sugerido:

1. **Seña**: pagar un porcentaje para confirmar y el resto en el club. Cambia el modelo de pagos.
2. **Turno fijo semanal**: modelar recurrencia y excepciones. Cada ocurrencia se materializa como una reserva, así la constraint sigue protegiendo.
3. **Lista de espera**: al liberarse un turno (expiración o cancelación) se ofrece al siguiente. Mezcla concurrencia y jobs.

### Conexión con los otros proyectos

- **Proyecto 2 (pipeline de datos)** es independiente: datos públicos reales y su propio ML. Comparte base técnica (FastAPI, PostgreSQL, Docker, CI). Opcional, después de `v1.0.0`: exportar los eventos de reservas como una fuente más.
- **Proyecto 3 (agente / servidor MCP)** consume la API de este proyecto. Por eso importan desde ahora: `Idempotency-Key`, errores claros y consistentes (409, 422 con códigos), OpenAPI correcta y autenticación por token acotado. Un agente reintenta y se equivoca de parámetros más que un humano.

### Cómo tiene que usar esto Claude Code

- Es contexto para orientar preguntas, no un plan a ejecutar.
- En S1, ayudame a **decidir** (preguntas guía) el recurso, los slots y la duración; no decidas por mí.
- Respetá el modo tutor del `CLAUDE.md` global.

---

## 1. Objetivo del proyecto

Construir un backend de **reservas de recursos con pago** (turnos de canchas, salas, consultorios o similar) que resuelva problemas reales de backend:

- Evitar **dobles reservas** bajo concurrencia.
- Manejar **transacciones** y estados de una reserva.
- Integrar **pagos** con webhooks (idempotencia, duplicados, orden).
- Procesar tareas en segundo plano (**expiración** de reservas sin pagar).
- Tener calidad de producción: **tests, Docker, CI, logs, deploy y documentación**.

**Por qué este proyecto:** se entiende en 30 segundos en una entrevista y te lleva directo a preguntas técnicas valiosas.

**Qué NO es el objetivo:** un frontend lindo. El frontend puede ser mínimo (o inexistente, usando la documentación interactiva de la API).

---

## 2. Stack sugerido

Elegí el lenguaje según lo que se repita en las ofertas de backend junior que mires. Esta propuesta usa Python, pero los conceptos se trasladan a Java/Spring, Node/NestJS o .NET.

| Capa | Sugerencia | Nota |
|---|---|---|
| Lenguaje / framework | Python + FastAPI | Documentación automática (OpenAPI) |
| Base de datos | PostgreSQL | Necesaria para el control de concurrencia que se plantea abajo |
| ORM / migraciones | SQLAlchemy 2.0 + Alembic | Migraciones versionadas |
| Cola / tareas | Redis + (Celery, ARQ o RQ) | Para expiración y notificaciones |
| Autenticación | JWT (access + refresh) | Hash de contraseñas con argon2 o bcrypt |
| Pagos | Mercado Pago en modo sandbox | Verificar la documentación vigente |
| Tests | pytest + testcontainers o DB de test | Incluir tests de concurrencia |
| Contenedores | Docker + Docker Compose | Un comando para levantar todo |
| CI | GitHub Actions | Tests + lint en cada PR |
| Deploy | Render, Railway, Fly.io u otro | Verificar planes gratuitos actuales |

---

## 3. Modelo de dominio (punto de partida)

Entidades principales:

- **User**: id, email, password_hash, role (`customer` / `admin`), created_at.
- **Resource**: id, name, description, price_per_slot, active.
- **Reservation**: id, user_id, resource_id, start_at, end_at, status, expires_at, created_at.
- **Payment**: id, reservation_id, provider, provider_payment_id, status, amount, created_at.
- **WebhookEvent**: id, provider, event_id (único), payload, processed_at.
- **IdempotencyKey**: key, user_id, request_hash, response, created_at.

Estados de una reserva:

```text
pending_payment → confirmed
pending_payment → expired
pending_payment → cancelled
confirmed       → cancelled   (con reglas de reembolso, opcional)
```

Documentá en el README qué transiciones son válidas y cuáles no.

---

## 4. Decisiones técnicas clave (lo que se pregunta en entrevistas)

### 4.1 Evitar la doble reserva
Opciones, de menos a más robusta:

1. Chequear disponibilidad en código y luego insertar (**incorrecto**: tiene condición de carrera).
2. Bloqueo de fila con `SELECT ... FOR UPDATE` dentro de una transacción.
3. **Restricción de exclusión en PostgreSQL** sobre un rango de tiempo por recurso, de modo que la propia base rechace solapamientos (investigá `tstzrange`, `EXCLUDE USING gist` y la extensión `btree_gist`).

Recomendación: implementar la opción 3 y escribir un **test de concurrencia** que dispare muchas solicitudes simultáneas al mismo turno y verifique que solo una gane. Documentá por qué elegiste esa opción.

### 4.2 Idempotencia
- Los endpoints que crean reservas o pagos deben aceptar una **idempotency key** (header) para que un reintento del cliente no cree duplicados.
- Guardá la clave, un hash del request y la respuesta, y devolvé la misma respuesta si llega repetida.

### 4.3 Webhooks de pago
- Los proveedores de pago pueden enviar el mismo evento **más de una vez** y **fuera de orden**.
- Guardá cada evento recibido con su id único y procesalo de forma idempotente.
- Verificá la autenticidad del webhook según lo que indique la documentación del proveedor.
- Respondé rápido y procesá el trabajo pesado en segundo plano.
- No confíes solo en el contenido del webhook: consultá el estado del pago al proveedor antes de confirmar la reserva.

### 4.4 Expiración de reservas pendientes
- Una reserva `pending_payment` bloquea el turno. Si no se paga en N minutos debe pasar a `expired` y liberar el turno.
- Implementalo con una tarea programada o con tareas diferidas, y asegurate de que sea segura si corre dos veces.

### 4.5 Seguridad básica
- Hash de contraseñas con algoritmo moderno, nunca guardarlas en claro.
- Autorización por rol y por propietario (un cliente solo ve sus reservas).
- Validación de entrada, rate limiting en endpoints sensibles, secretos en variables de entorno.
- CORS configurado explícitamente.

---

## 5. Fases de ejecución

Cada fase termina con algo **funcionando y presentable**, un tag de Git y una sección actualizada del README.

### Fase 0 — Setup (1–3 días)
**Objetivo:** entorno reproducible y esqueleto del proyecto.
- Repositorio, estructura de carpetas, gestor de dependencias.
- Docker Compose con app + PostgreSQL + Redis.
- Linter/formatter, pre-commit, CI mínimo (corre tests y lint).
- Endpoint `/health`.

**Listo cuando:** `docker compose up` levanta todo y CI pasa en verde.
**Tag:** `v0.1.0`

### Fase 1 — Usuarios, autenticación y recursos (1 semana)
**Objetivo:** CRUD sólido con seguridad básica.
- Registro y login con JWT, hash de contraseñas.
- Roles `customer` y `admin`.
- CRUD de recursos (solo admin crea/edita).
- Migraciones con Alembic, paginación y manejo de errores consistente.
- Tests de endpoints y de permisos.

**Listo cuando:** un cliente no puede hacer acciones de admin y los tests lo prueban.
**Tag:** `v0.2.0`

### Fase 2 — Reservas y concurrencia (1–2 semanas) ⭐ núcleo del proyecto
**Objetivo:** reservar sin dobles reservas.
- Crear reserva en estado `pending_payment` con `expires_at`.
- Restricción de exclusión en la base para impedir solapamientos.
- Consulta de disponibilidad.
- Máquina de estados con transiciones validadas.
- **Test de concurrencia** (N solicitudes simultáneas, una sola gana).
- Idempotency key en la creación.

**Listo cuando:** el test de concurrencia pasa de forma consistente y está explicado en el README.
**Tag:** `v0.3.0`

### Fase 3 — Pagos y webhooks (1–2 semanas)
**Objetivo:** confirmar reservas al recibir el pago.
- Crear la preferencia/orden de pago en sandbox.
- Endpoint de webhook con verificación y tabla de eventos.
- Procesamiento idempotente (evento duplicado = no pasa nada).
- Confirmación de reserva solo tras validar el estado real del pago.
- Tests con webhooks duplicados y fuera de orden.

**Listo cuando:** el flujo reservar → pagar → confirmar funciona de punta a punta en sandbox.
**Tag:** `v0.4.0`

### Fase 4 — Trabajos en segundo plano (1 semana)
**Objetivo:** expiración y notificaciones.
- Worker que expira reservas vencidas y libera el turno.
- Notificación por email (puede ser un servicio de prueba o log) al confirmar.
- Reintentos con backoff ante fallos.
- Seguridad ante ejecución duplicada.

**Listo cuando:** una reserva sin pagar vence sola y el turno vuelve a estar disponible.
**Tag:** `v0.5.0`

### Fase 5 — Calidad de producción (1 semana)
**Objetivo:** que se vea profesional.
- Logs estructurados con id de request.
- Rate limiting, auditoría básica de cambios de estado.
- Cobertura de tests razonable en la lógica crítica.
- Documentación OpenAPI revisada, colección Postman o ejemplos con curl.
- Diagrama de arquitectura y diagrama de estados en el README.

**Tag:** `v0.6.0`

### Fase 6 — Deploy, demo y presentación (3–5 días)
**Objetivo:** que cualquiera pueda verlo funcionando.
- Deploy público con base de datos administrada.
- Video corto o GIF (1–2 minutos) del flujo completo.
- README final.
- Tag `v1.0.0` y release en GitHub.

---

## 6. Flujo de trabajo en Git

- `main` siempre en estado funcional, historial prácticamente lineal.
- Una rama corta por funcionalidad (`feat/reservas-concurrencia`), merge vía Pull Request aunque seas el único (squash o rebase).
- Commits pequeños y descriptivos (por ejemplo, formato convencional: `feat:`, `fix:`, `test:`, `docs:`).
- Cada fase se marca con un **tag/release**, no con ramas separadas.
- Un `CHANGELOG` o la sección de releases cuenta la evolución del proyecto.

---

## 7. README final (estructura sugerida)

1. Qué es y qué problema resuelve (3 líneas).
2. Demo: link en vivo + video/GIF.
3. Arquitectura (diagrama) y stack.
4. Decisiones de diseño y trade-offs (concurrencia, idempotencia, webhooks).
5. Cómo correrlo localmente (un comando).
6. Cómo correr los tests.
7. Documentación de la API.
8. Limitaciones conocidas y próximos pasos.

---

## 8. Extensiones opcionales (solo después de v1.0.0)

- Cancelaciones con política de reembolso.
- Multi-tenant (varias sedes o negocios aislados entre sí).
- Caché de disponibilidad.
- Métricas y monitoreo (Prometheus/Grafana).
- Panel mínimo de administración.

---

## 9. Preguntas de entrevista para practicar

1. ¿Cómo evitás que dos usuarios reserven el mismo turno al mismo tiempo?
2. ¿Qué pasa si el webhook de pago llega dos veces? ¿Y si llega antes de que la reserva se haya guardado?
3. ¿Qué es la idempotencia y dónde la usaste?
4. ¿Qué hace tu sistema si el worker de expiración se ejecuta dos veces?
5. ¿Cómo manejás los secretos y la autenticación?
6. ¿Qué cambiarías si tuvieras 100 veces más tráfico?
7. ¿Qué es una transacción y qué nivel de aislamiento usaste y por qué?
8. ¿Cómo probaste la concurrencia?

---

## 10. Errores comunes a evitar

- Chequear disponibilidad en código sin protección en la base.
- Confiar ciegamente en el contenido del webhook.
- No tener tests de los casos difíciles (duplicados, concurrencia).
- Alcance infinito: agregar funcionalidades antes de cerrar una fase.
- README pobre o demo caída justo cuando alguien la mira.

---

## 11. Criterio de "listo para mostrar"

- [ ] Demo pública funcionando y video de respaldo.
- [ ] README completo con decisiones de diseño.
- [ ] Tests de concurrencia y de webhooks pasando en CI.
- [ ] Docker Compose levanta todo con un comando.
- [ ] Historial de Git ordenado y tags por fase.
- [ ] Puedo explicar cada decisión técnica sin leer el código.


