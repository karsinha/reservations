# Backend de reservas con pagos

API REST para reservar recursos con pago asociado. Proyecto de portafolio y de aprendizaje.
Estado actual y próximo paso: `docs/progress.md`.
Las reglas de tutor (no vibecodear) están en mi `CLAUDE.md` global y rigen acá también.
Contexto del proyecto y material de referencia: `docs/guide.md` (no es fuente de verdad; ante conflicto mandan este archivo y `docs/roadmap.md`).

## Entorno de desarrollo

- **SO: Fedora 44 (Linux).** Paquetes del sistema con `dnf`. Dame comandos para Fedora; si algo difiere de Ubuntu/Debian (nombres de paquetes, rutas, servicios), avisame.
- **Docker o Podman: a confirmar en S1 (ADR).** Fedora trae Podman por defecto; Docker se instala aparte. No asumas cuál tengo: preguntame antes de darme comandos de `compose`.
- **SELinux está activo por defecto en Fedora.** Si un volumen montado falla por permisos, sospechá de SELinux (etiquetas `:z` / `:Z` en el volumen) antes de proponer `chmod 777` o desactivarlo. Nunca desactivarlo.
- **Python:** el del sistema no es el del proyecto. Versión objetivo 3.12; verificar con `python3 --version` qué trae Fedora y cómo instalar la versión que necesito. Entorno aislado por proyecto (venv u otro gestor: decisión en S1). Nunca `sudo pip install`.
- Si algo requiere `sudo dnf install`, decime qué es y por qué lo necesitamos.
- Ante la duda sobre un paquete o comando de Fedora: verificar en la documentación oficial.

## Stack (propuesto, ajustable; registrar cambios en `docs/decisions.md`)

Python 3.12 · FastAPI · PostgreSQL · SQLAlchemy 2.0 + Alembic · Pydantic v2 · pytest · Docker Compose · ruff · GitHub Actions.
Proveedor de pagos: a definir en S5 (usar sandbox/modo prueba).
Antes de asumir comportamiento de una librería o de la API del proveedor de pagos: verificar en la documentación oficial.

## Estructura

```text
app/
  main.py
  api/        # endpoints finos: validan y delegan
  core/       # config, seguridad, db
  models/     # modelos SQLAlchemy
  schemas/    # schemas Pydantic
  services/   # lógica de negocio (incluye la máquina de estados)
tests/
  unit/
  integration/
  concurrency/
alembic/      # migraciones
docs/         # roadmap, arquitectura, decisiones, aprendizaje, progreso, git
.github/      # plantilla de PR y CI
```

## Comandos (actualizar a medida que existan)

- Base de datos: `docker compose up -d db` (o el equivalente con Podman, según ADR de S1)
- Servidor: `uvicorn app.main:app --reload`
- Tests: `pytest`
- Lint/formato: `ruff check . && ruff format .`
- Migraciones: `alembic revision --autogenerate -m "msg"` / `alembic upgrade head`

## Reglas de dominio (no negociables)

- **Dinero:** enteros en centavos (o `Decimal`), nunca `float`. Siempre con moneda explícita.
- **Fechas:** UTC, `timestamptz`.
- **Doble reserva imposible a nivel base de datos** (constraint de exclusión sobre rango de tiempo por recurso), no solo en el código. Chequear disponibilidad en código y después insertar tiene una race condition.
- **Idempotencia:** los endpoints que crean reservas o pagos aceptan el header `Idempotency-Key`.
- **Webhooks:** pueden llegar repetidos, desordenados o antes de lo esperado. Guardar cada evento con id único, procesar de forma idempotente, verificar autenticidad según la doc del proveedor, responder rápido, y **no confiar en el payload**: consultar el estado real del pago al proveedor antes de confirmar.
- **Máquinas de estado explícitas** para reservas y pagos; las transiciones válidas viven en un solo lugar.
- **Transacciones:** todo lo que modifica varias tablas va en una. No llamar a APIs externas con una transacción abierta.
- **Tareas periódicas** (expiración) deben ser seguras si corren dos veces.
- **Seguridad:** passwords con argon2 o bcrypt; autorización por rol y por dueño (un customer solo ve sus reservas); nunca loguear secretos, tokens ni datos de pago.
- **Validación:** toda entrada pasa por Pydantic; no confiar en el cliente.
- **Secretos:** variables de entorno. `.env` fuera de git; `.env.example` sí.

## Convenciones

- Type hints en todo. Funciones cortas. Lógica de negocio en `services/`, no en los endpoints.
- Un solo vocabulario: `booking` (no "reservation"), estados `pending_payment | confirmed | expired | cancelled`.
- Formato de error consistente en toda la API.
- Cada funcionalidad nueva con tests. Los tests de concurrencia y de fallo de pago son obligatorios.
- Nunca modificar la base a mano: todo cambio de esquema va por Alembic.
- No cambiar el modelo de datos sin discutirlo y registrar la decisión.

## Git

Flujo completo en `docs/git-workflow.md`: leerlo antes de proponer cualquier operación de git.

- Claude **nunca** hace commit, push ni tag: sugiere el mensaje y los comandos, y los ejecuto yo.
- Ramas `feat/`, `fix/`, `test/`, `docs/`, `chore/`. Todo entra por PR con squash merge; el título del PR va en Conventional Commits, en inglés.
- Una migración de Alembic ya mergeada no se edita: se crea otra.
- Cierre de semana del roadmap: tag anotado + release + entrada en `docs/progress.md`.

## Flujo con Claude

1. Antes de empezar: leer `docs/roadmap.md` (corte vertical actual), `docs/architecture.md` y `docs/progress.md`.
2. Durante: concepto nuevo → explicarlo antes de implementar y anotarlo en `docs/learning.md`.
3. Decisión técnica relevante → `docs/decisions.md` (en inglés).
4. Al cerrar la sesión → entrada en `docs/progress.md`.