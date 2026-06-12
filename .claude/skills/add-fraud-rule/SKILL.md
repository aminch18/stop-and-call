---
name: add-fraud-rule
description: Usar cuando se añade, modifica o elimina un patrón de detección de fraude en rules/es-ES/ruleset.json. Garantiza validación contra el schema, versionado correcto, actualización del changelog, tests en rules-engine y revisión de tono/lenguaje para el usuario final.
---

# Añadir o modificar un patrón de fraude (rules/es-ES/ruleset.json)

## Cuándo usar esta skill

- Se reporta un nuevo tipo de estafa (ej. nueva variante de smishing, nuevo prefijo
  internacional sospechoso, nuevo pretexto de "familiar en apuros").
- Se necesita ajustar un patrón existente por falsos positivos/negativos detectados
  en uso real (Fase 2).
- Se traduce o adapta un patrón ya existente en otro idioma/región a `es-ES`.

## Pasos obligatorios

1. **Localizar el archivo**: `rules/es-ES/ruleset.json`. Revisar primero
   `rules/schema/ruleset.schema.json` para conocer la estructura exacta esperada
   (campos obligatorios, tipos, enums de categoría).

2. **Añadir/modificar la entrada** siguiendo la estructura del schema. Cada patrón debe incluir
   como mínimo:
   - `id`: identificador estable, en inglés, kebab-case (ej. `seguridad-social-bizum-request`).
   - `category`: una de las categorías definidas en el schema (ej. `payment-request`,
     `urgency`, `suspicious-link`, `caller-id-spoofing`, `impersonation`).
   - `pattern` / `keywords`: la señal detectable (regex o lista de palabras clave),
     en español, cubriendo variantes con/sin tildes y mayúsculas/minúsculas.
   - `userMessage`: el texto que verá el usuario mayor si se detecta este patrón.
     **Debe cumplir el checklist de tono de `docs/CONTRIBUTING.md`** (sin jerga,
     frases cortas, en segunda persona, accionable — ej. "Esto puede ser una estafa.
     No compartas ningún código. ¿Llamamos a [familiar]?").
   - `severity`: `low` | `medium` | `high`.
   - `source`: de dónde viene este patrón (ej. "Aviso INCIBE marzo 2026", "reporte de usuario").

3. **Validar contra el schema**:
   ```bash
   cd apps/mobile
   npm run validate-rules   # o: npx ajv validate -s ../../rules/schema/ruleset.schema.json -d ../../rules/es-ES/ruleset.json
   ```
   (Si el script `validate-rules` no existe todavía, créalo como parte de esta tarea
   y añádelo a `package.json` — debe ejecutarse también en `mobile-ci.yml`.)

4. **Bump de versión**: incrementar `version` en `ruleset.json` siguiendo SemVer:
   - PATCH: ajuste de un patrón existente (menos falsos positivos/negativos).
   - MINOR: nuevo patrón añadido.
   - MAJOR: cambio que rompe compatibilidad con el formato esperado por `rules-engine/`.

5. **Actualizar `rules/CHANGELOG.md`**: una línea por cambio, formato:
   ```
   ## [1.1.0] - 2026-06-12
   - Añadido: patrón `seguridad-social-bizum-request` (fuente: aviso INCIBE marzo 2026)
   ```

6. **Añadir/actualizar test** en `apps/mobile/src/rules-engine/__tests__/`:
   - Caso positivo: un texto de ejemplo que DEBE activar el patrón.
   - Caso negativo: un texto similar pero legítimo que NO debe activarlo (para detectar
     falsos positivos).

7. **Ejecutar**:
   ```bash
   npm run lint && npm run typecheck && npm test
   ```
   Todo debe pasar en verde antes de proponer el cambio.

## Principios al redactar `userMessage`

- Nunca generar alarma sin dar una acción clara. Siempre terminar con una sugerencia
  concreta: llamar al familiar, llamar al 017, o "no hagas nada y pide ayuda".
- Evitar términos como "phishing", "smishing", "spoofing" en el texto que ve el usuario
  — usar "esto parece falso", "esto puede ser una trampa", etc.
- Un patrón = un mensaje. No combinar varias señales en un único texto confuso.

## Qué NO hacer

- No añadir patrones que requieran enviar el texto analizado a un servidor para
  evaluarlo (rompe `docs/PRIVACIDAD.md` y `ADR-0001`, Decisión 3).
- No añadir patrones basados en modelos de IA/LLM en v1 (ver `CLAUDE.md`, no negociable 4).
