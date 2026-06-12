# CLAUDE.md — Contexto del proyecto para Claude Code

> Este archivo se carga automáticamente al iniciar una sesión de Claude Code en este repo.
> Documentos de referencia obligatorios antes de tocar arquitectura: `docs/adr/0001-arquitectura-base.md`
> y `docs/ESPECIFICACION.md`.

## Qué es este proyecto

App móvil **gratuita y open source** (Apache-2.0) de protección frente a fraude digital para
personas mayores en España, con baja alfabetización tecnológica. No es un gestor de contraseñas,
antivirus ni autenticador. Es una capa de simplicidad y confianza: un widget con tres botones
("llamar a un familiar", "tengo un mensaje raro", "llamar al 017 de INCIBE").

## No negociables (cualquier PR que los rompa debe rechazarse)

1. **Privacidad por diseño**: el contenido de mensajes/llamadas NUNCA sale del dispositivo.
   La checklist se evalúa 100% local (`rules-engine/`). Antes de añadir cualquier llamada
   de red desde `apps/mobile`, comprobar contra `docs/PRIVACIDAD.md`.
2. **Simplicidad radical**: si una funcionalidad añade una decisión, un paso extra o texto
   técnico para el usuario final, está mal — vuelve a diseñarla.
3. **Accesibilidad**: tipografía grande por defecto, alto contraste, soporte TalkBack.
   Cualquier nueva pantalla/componente debe cumplir esto (ver checklist en `docs/CONTRIBUTING.md`).
4. **Sin IA/LLM en v1**: detección basada en reglas (`rules/es-ES/ruleset.json`), auditable.
   No introducir dependencias de inferencia ni llamadas a APIs externas de IA sin una nueva ADR.
5. **Sin cuentas tradicionales**: pairing por código corto + token de dispositivo. No añadir
   email/password ni OAuth de terceros sin discutirlo primero.

## Estado actual / fase activa

Ver `docs/ESPECIFICACION.md`, sección "Roadmap". Indica aquí manualmente el hito activo
y bórralo/actualízalo al cerrar cada milestone:

- **Hito activo:** M1.1 — Scaffolding del monorepo, CI, esqueleto RN (3 pantallas), esqueleto backend.
- **Próximo hito:** M1.2 — Spike del módulo nativo de widget Android (RemoteViews).

## Estructura y comandos

Estructura completa: ver `docs/ESPECIFICACION.md` sección 2.

### Mobile (`apps/mobile`)
```
npm install
npm run lint        # ESLint + Prettier (debe pasar antes de cualquier commit)
npm run typecheck    # tsc --noEmit, strict mode
npm test             # Jest — cobertura 100% obligatoria en rules-engine/
npm run android      # build/run debug Android
```

### Backend (`backend`)
```
dotnet build
dotnet test
dotnet run --project src/Api   # expone GET /health
```

### Reglas de fraude (`rules/`)
Antes de añadir o modificar un patrón, usa la skill `.claude/skills/add-fraud-rule/SKILL.md`
(valida contra `rules/schema/ruleset.schema.json`, actualiza versión y `rules/CHANGELOG.md`,
y añade el test correspondiente en `apps/mobile/src/rules-engine/__tests__`).

## Convenciones

- Código y comentarios: **inglés**. Documentación de usuario/contribución: **español**.
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `test:`).
- Sin gestores de estado complejos (Redux/MobX) — React Context/hooks es suficiente.
- Cualquier texto visible para el usuario final: lenguaje sencillo, sin jerga técnica,
  revisar contra el "tono" descrito en `docs/CONTRIBUTING.md`.

## Definición de "hecho" por tarea

Código + tests pasando + lint/typecheck en verde + actualización de docs si aplica.
Si la tarea toca una decisión arquitectónica nueva (no cubierta por ADR-0001), proponer
un borrador de ADR-000X antes de implementar, no después.
