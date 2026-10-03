# LEARNING — conceptos que tengo que dominar

Marcar `[x]` **solo yo**, cuando pueda explicarlo con mis palabras sin mirar nada (el tutor me pregunta; yo decido).

## Bases de datos

- [ ] Claves primarias y foráneas
- [ ] JOINs (INNER, LEFT)
- [ ] GROUP BY / HAVING
- [ ] Índices: qué son y cuándo ayudan
- [ ] Transacciones y ACID
- [ ] Niveles de aislamiento y bloqueos (`SELECT ... FOR UPDATE`)
- [ ] Constraints (UNIQUE, CHECK, exclusión)
- [ ] Migraciones
- [ ] ORM vs SQL directo; sesiones; problema N+1

## Backend / APIs

- [ ] HTTP: métodos, códigos de estado
- [ ] REST: recursos, idempotencia
- [ ] Validación y serialización
- [ ] Autenticación: hash vs cifrado, JWT o sesiones
- [ ] Autorización: roles y propiedad del recurso
- [ ] Webhooks y reintentos
- [ ] Manejo de errores y logging
- [ ] Inyección de dependencias en FastAPI

## Pagos y concurrencia

- [ ] Idempotency keys
- [ ] Condiciones de carrera (race conditions)
- [ ] Máquinas de estado
- [ ] Tareas en segundo plano y ejecución duplicada

## Seguridad básica

- [ ] SQL injection y por qué los parámetros lo evitan
- [ ] Secretos y variables de entorno
- [ ] Rate limiting y CORS

## Calidad y operación

- [ ] Tests unitarios vs integración; fixtures; mocks vs base real
- [ ] Docker y Docker Compose
- [ ] CI
- [ ] Git: ramas, PRs, rebase vs merge
- [ ] Deploy

## Práctica diaria

- 30–40 min de SQL por día (ejercicios cortos).
- 10 min leyendo documentación oficial en inglés (PostgreSQL, FastAPI, SQLAlchemy).

## Preguntas de entrevista para practicar

- ¿Cómo evitás que dos usuarios reserven lo mismo a la vez?
- ¿Qué pasa si el webhook de pago llega dos veces? ¿Y si llega antes de que la reserva se haya guardado?
- ¿Qué es la idempotencia y dónde la usaste?
- ¿Qué hace tu sistema si el worker de expiración corre dos veces?
- ¿Qué es una transacción y qué nivel de aislamiento usaste y por qué?
- ¿Por qué no usar `float` para dinero?
- ¿Cómo testeaste la concurrencia?
- ¿Qué cambiarías con 100 veces más tráfico?

## Glosario propio

| Term (EN) | Definición con mis palabras (ES) | Fecha |
|---|---|---|
| | | |