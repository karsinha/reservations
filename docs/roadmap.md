# ROADMAP — cortes verticales

**Regla:** cada semana termina con algo funcionando de punta a punta (no capas a medias), con un tag de Git (`v0.X.0`) y una entrada en `PROGRESS.md`. Si una semana se atrasa, se recorta alcance, no calidad. Marcar `[x]` al completar. El corte actual se anota en `PROGRESS.md`.

## Mes 1 — Base

- [ ] **S1:** Entorno (venv, Docker Compose con Postgres), FastAPI con `/health`, repo en GitHub, **CI mínimo** (ruff + pytest, aunque haya un solo test). Definir el recurso concreto y el modelo de slots (ADR). Modelo de datos discutido.
- [ ] **S2:** Usuarios + **autenticación** (registro, login, hash de contraseña, JWT, roles `customer`/`admin`) + CRUD de recursos (solo admin) con migraciones Alembic. Tests de permisos.
- [ ] **S3:** Crear y consultar reservas (versión simple): estados, máquina de estados básica, formato de error consistente.
- [ ] **S4:** Disponibilidad + constraint de exclusión anti-solapamiento + **primer test de concurrencia** (N solicitudes simultáneas al mismo turno, una sola gana).

## Mes 2 — Lo difícil (lo que se pregunta en entrevistas)

- [ ] **S5:** `Idempotency-Key` en creación de reservas y pagos. Pagos en sandbox. Elegir proveedor (ADR).
- [ ] **S6:** Webhooks: tabla de eventos, verificación, procesamiento idempotente, consulta del estado real al proveedor. Tests con eventos duplicados y desordenados.
- [ ] **S7:** Concurrencia a fondo: transacciones, niveles de aislamiento, `SELECT ... FOR UPDATE`, carrera webhook vs. expiración. Decidir qué pasa si el pago llega después de expirar la reserva (ADR).
- [ ] **S8:** Expiración de reservas (tarea segura ante doble ejecución), reintentos, manejo de errores, tests de fallo de pago.

## Mes 3 — Producción

- [ ] **S9:** Dockerizar la app completa (un comando levanta todo).
- [ ] **S10:** CI completo (tests contra Postgres real, cobertura de la lógica crítica).
- [ ] **S11:** Deploy online + logs estructurados con request id + rate limiting básico.
- [ ] **S12:** README (diagramas de arquitectura y de estados), documento de decisiones, video/GIF de la demo, limpieza. Perfil de GitHub/LinkedIn.

## En paralelo (desde el mes 2)

- Mirar ofertas de backend junior para ver qué stack se repite y ajustar el proyecto 2.
- Dejar el README parcial y el repo público desde temprano; postularse sin esperar a "estar listo" (el proceso con poca experiencia lleva semanas).

## Después

- Primeros usuarios reales (aunque sean pocos).
- Proyecto 2: pipeline de datos económicos.