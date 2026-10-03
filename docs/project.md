# PROJECT — qué estoy construyendo

## Objetivo

API backend para reservar un recurso (cancha, sala, turno… a definir en S1) con pago asociado, confiable ante concurrencia y fallos de pago.
Objetivo personal: aprender backend de verdad y tener un proyecto desplegado y defendible en entrevistas.

**Pitch en 30 segundos** (completar con mis palabras al terminar S1):
> …

## Alcance del MVP

- [ ] Registro y login de usuarios (roles customer/admin)
- [ ] CRUD de recursos reservables (solo admin)
- [ ] Consulta de disponibilidad
- [ ] Crear / cancelar reservas
- [ ] Pago de la reserva (sandbox) con idempotencia
- [ ] Webhooks del proveedor de pagos
- [ ] Expiración de reservas sin pagar
- [ ] Tests (unitarios, integración, concurrencia, webhooks duplicados/desordenados)
- [ ] Logs estructurados y rate limiting básico
- [ ] Docker + CI + deploy

## Fuera de alcance (por ahora)

Frontend, notificaciones, múltiples sedes/tenants, panel de administración, reembolsos.

## Definición de "terminado" (proyecto 1)

- Desplegado y accesible online, con video/GIF de respaldo.
- README con cómo correrlo, arquitectura, diagramas y decisiones.
- CI verde con tests de concurrencia y de webhooks.
- `docker compose up` levanta todo.
- Puedo explicar cada decisión de diseño sin mirar el código.
- Al menos algunos usuarios reales lo probaron.

## Decisiones pendientes (con fecha límite)

| Decisión | Cuándo | Registro |
|---|---|---|
| ¿Qué recurso concreto? ¿Slots fijos o rangos libres? | S1 | ADR-003 |
| ¿SQLAlchemy sync o async? | S1 | ADR-002 |
| ¿Qué proveedor de pagos? | S5 | ADR nuevo |
| ¿Dónde lo despliego? | S10 | ADR nuevo |

## Después

Proyecto 2: pipeline de datos económicos.