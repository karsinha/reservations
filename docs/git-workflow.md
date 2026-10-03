# GIT WORKFLOW

Flujo de trabajo con Git para este proyecto. Los comandos y los mensajes van en inglés; las explicaciones, en español.

## Principios

- `main` siempre funcional y con CI en verde. Historial lineal y legible.
- Una rama corta por funcionalidad. Nada se escribe directo en `main`.
- Todo entra por Pull Request (PR), aunque sea yo el único que trabaja.
- Claude Code **nunca** hace `commit`, `push` ni `tag`: sugiere el mensaje y los comandos, y los ejecuto yo después de mirar el `git diff`.

## Configuración inicial (una sola vez)

- `git config --global user.name` y `user.email` (el mismo email que uso en GitHub).
- Autenticación con GitHub por SSH o por token: seguir la documentación oficial de GitHub.
- `.gitignore` con `.env`, `.venv/` y cachés **desde el primer commit**.

## Ramas

Formato: `tipo/descripcion-corta` (minúsculas, guiones). Opcional: prefijo de semana.

| Tipo | Uso | Ejemplo |
|---|---|---|
| `feat/` | funcionalidad nueva | `feat/s4-overlap-constraint` |
| `fix/` | corrección de bug | `fix/webhook-duplicate-event` |
| `test/` | solo tests | `test/booking-concurrency` |
| `docs/` | solo documentación | `docs/adr-003-slot-model` |
| `chore/` | tooling, CI, dependencias | `chore/ci-ruff-pytest` |

Una rama = un tema. Si aparece algo que no tiene que ver, va en otra rama.

## Commits

Conventional Commits, en inglés, en imperativo y en minúscula: `tipo(scope opcional): descripción`.

```text
feat(booking): add exclusion constraint for overlapping slots
fix(webhook): ignore events already processed
test(booking): add concurrent booking test
docs: add ADR-003 slot model
chore(ci): run ruff and pytest on pull requests
```

- Commits chicos: un cambio lógico por commit.
- **Código y test juntos** en el mismo commit.
- **Cambio de modelo y su migración de Alembic juntos.**
- En la rama puedo commitear cosas intermedias ("wip"): con squash merge no ensucian `main`.
- Antes de cada commit: `git status` y `git diff` (o `git diff --staged`). No hacer `git add .` sin saber qué entra.

## Pull Requests y merge

1. Subir la rama: `git push -u origin <rama>`.
2. Abrir el PR. Título en formato Conventional Commit (será el mensaje final en `main`). Completar la plantilla.
3. Esperar el CI en verde y **releer mi propio diff** en GitHub antes de mergear.
4. **Squash merge** (un PR = un commit en `main`), borrar la rama.
5. Actualizar `main` local: `git switch main && git pull`.

Si `main` avanzó mientras trabajaba: `git fetch origin && git rebase origin/main` en mi rama. Los conflictos se resuelven a mano y entendiendo cada uno; no se resuelven "aceptando todo".

### Protección de `main`

Configurar en GitHub: exigir PR y CI en verde antes de mergear. Las opciones disponibles pueden depender del plan y de si el repo es público o privado: verificar en la documentación de GitHub.

## Migraciones de Alembic

- Una migración ya mergeada **no se edita**: se crea una nueva.
- Una migración por cambio de esquema, en el mismo PR que el cambio de modelo.
- Revisar siempre el archivo autogenerado antes de commitearlo: `autogenerate` no detecta todo (por ejemplo, constraints de exclusión) y puede generar cosas de más.

## Tags y releases

- Un tag anotado por semana del roadmap: S1 = `v0.1.0`, S2 = `v0.2.0`, … S11 = `v0.11.0`; al cerrar S12, `v1.0.0`. El `roadmap.md` manda por sobre la numeración de la guía.
- Crear: `git tag -a v0.4.0 -m "S4: availability and overlap constraint"`.
- Subir: `git push origin v0.4.0` (o `git push --tags`).
- Crear el release en GitHub desde el tag, con notas cortas (qué se puede hacer ahora que antes no).
- Cierre de semana: tag + release + entrada en `docs/progress.md`.

## Secretos

- `.env` en `.gitignore` desde el principio; `.env.example` (sin valores reales) sí se commitea.
- Si una clave llegó a subirse al repo, **hay que rotarla**: borrar el commit no alcanza, porque queda en el historial y puede haber sido copiada.
- Revisar `git diff --staged` antes de commitear precisamente para atrapar esto.

## Cuando algo sale mal

- Primero **mirar**: `git status`, `git log --oneline --graph`, `git reflog`. El reflog guarda dónde estuvo `HEAD`; casi todo lo "perdido" se recupera desde ahí.
- Deshacer cambios sin commitear en un archivo: `git restore <archivo>` (descarta los cambios: confirmar antes).
- Deshacer algo ya publicado en `main`: `git revert <commit>` (crea un commit nuevo), no reescribir historia.
- Nunca `git push --force` sobre `main`. En mi propia rama, `--force-with-lease` y solo si entiendo por qué.
- `git reset --hard`, `git clean -fd` y similares: solo con confirmación explícita y sabiendo qué se pierde.

## Ciclo completo

```bash
git switch main && git pull
git switch -c feat/s4-overlap-constraint
# trabajar: commits chicos, cada uno con su test
git push -u origin feat/s4-overlap-constraint
# abrir PR → CI en verde → releer diff → squash merge → borrar rama
git switch main && git pull
# al cerrar la semana:
git tag -a v0.4.0 -m "S4: availability and overlap constraint"
git push origin v0.4.0
```