# ADR-0001: Arquitectura base del proyecto (nombre provisional: "Compañero Digital")

**Estado:** Aceptado
**Fecha:** 2026-06-12
**Responsables:** Amin (fundador / arquitecto)
**Revisión:** v1 — sujeta a revisión tras Milestone 1 del MVP

## Contexto

Se quiere construir una solución **gratuita y open source** de protección digital frente a fraude para personas con baja alfabetización tecnológica (principalmente mayores), con foco inicial en España. El objetivo no es competir en "detección" (eso lo cubren cada vez mejor Google/Apple a nivel de SO y proyectos como Guardião), sino en una **capa humana de simplicidad y escalado**: un punto único, siempre visible, que ayude al usuario a "parar y verificar" y conectarle con un familiar de confianza o con recursos oficiales (línea 017 de INCIBE).

Restricciones clave:

- Equipo inicial: 1 persona, perfil Staff/Senior backend .NET, con interés en C# pero abierto a otras tecnologías si el proyecto lo justifica.
- Sin presupuesto para infraestructura cara; debe poder operar con coste casi nulo.
- Usuarios finales: personas mayores en España, mayoritariamente con dispositivos Android de gama media/baja.
- Requisito no negociable: **privacidad por diseño** (sin vigilancia encubierta) y cumplimiento RGPD.
- Objetivo de comunidad: facilitar que terceros contribuyan (stack accesible, código auditable).
- No hay objetivo de monetización; el proyecto prioriza impacto y confianza sobre velocidad de "time to market" comercial.

## Decisión

Se adoptan las siguientes decisiones arquitectónicas para la v1 (MVP):

1. Cliente móvil: **React Native + TypeScript, Android-first**.
2. Backend: **ASP.NET Core (Minimal API, .NET 9)**.
3. Procesamiento de datos: **local-first, reglas heurísticas on-device** (sin IA/LLM en v1).
4. Notificaciones al familiar: **Firebase Cloud Messaging (FCM)**.
5. Licencia: **Apache License 2.0**.
6. Los módulos nativos críticos (widget de pantalla de inicio y recepción de "compartir") se mantienen **en el propio repositorio**, no como dependencias externas sin auditar.

A continuación se detalla el razonamiento de cada una.

---

### Decisión 1: Framework del cliente móvil

| Dimensión | Kotlin nativo | Flutter | .NET MAUI | React Native + TS |
|---|---|---|---|---|
| Complejidad inicial | Media | Media | Media-Alta | Media |
| Soporte de widgets de pantalla de inicio (Android) | Nativo, total | Vía plugins, parcial | Limitado/inmaduro | Vía `react-native-android-widget`, requiere módulo nativo |
| Soporte "compartir hacia la app" (share intent) | Nativo, total | Vía plugins | Limitado | Vía módulo nativo, bien documentado |
| Coste de pasar a iOS más adelante | Reescritura completa | Bajo | Bajo (pero ecosistema iOS MAUI menos maduro) | Bajo |
| Tamaño de comunidad para atraer contribuidores OSS | Grande (Android) pero nicho específico | Grande | Pequeña/nicho | Muy grande (JS/TS) |
| Afinidad con el perfil del fundador (.NET) | Baja | Baja | Alta | Media |

**Pros / Contras relevantes:**

- *Kotlin nativo*: control total sobre las APIs críticas (widgets, share, accesibilidad), pero limita drásticamente el pool de contribuidores y duplica esfuerzo si se quiere iOS después.
- *Flutter*: buen rendimiento y UI, pero el soporte de widgets/share en Android vía plugins es menos maduro que en RN y la comunidad hispanohablante de "impacto social" está menos presente.
- *.NET MAUI*: máxima afinidad con el fundador, pero el soporte de widgets de Android está inmaduro hoy y el ecosistema de paquetes para integraciones de sistema (share intents, accesibilidad avanzada) es notablemente más pequeño. Alto riesgo de bloqueos en justo las dos piezas más críticas del producto.
- *React Native + TS*: las piezas críticas (widget, share intent) requieren un módulo nativo Kotlin igualmente —como en cualquier framework cross-platform—, pero existen librerías de referencia maduras y, sobre todo, **la mayor base de contribuidores potenciales de cualquier opción cross-platform**, lo cual es estratégico para un proyecto open source sin presupuesto.

### Trade-off analysis

El criterio decisivo no es el rendimiento (la UI es trivial: 3 botones grandes + una checklist), sino:

1. Minimizar el riesgo en las dos integraciones nativas que son el corazón del producto.
2. Maximizar la probabilidad de atraer contribuidores externos a medio plazo.

React Native cumple ambos mejor que .NET MAUI (descartado por inmadurez en widgets) y mejor que Kotlin nativo (descartado por limitar la comunidad y duplicar esfuerzo futuro). Flutter queda como alternativa razonable de "segunda opción" si en Milestone 1 el módulo de widget en RN resultara más costoso de lo esperado.

**Decisión:** React Native (bare workflow, no Expo managed, por necesidad de módulos nativos) + TypeScript estricto. Android-first; iOS queda en Fase 3.

---

### Decisión 2: Backend

| Opción | Evaluación |
|---|---|
| ASP.NET Core Minimal API (.NET 9) | Afinidad total con el fundador (velocidad de desarrollo y calidad desde el día 1), bajo coste de hosting, control total de datos (RGPD) |
| Node/Express | Viable, pero sin ventaja sobre .NET dado el perfil del equipo |
| Solo BaaS (Firebase/Supabase, sin backend propio) | Reduce esfuerzo inicial, pero compromete el principio de "datos auditables y bajo control propio" y dificulta servir el feed de reglas versionado de forma transparente |

**Decisión:** ASP.NET Core Minimal API. Responsabilidades del backend en v1, deliberadamente mínimas:

- Servir el feed versionado de reglas de detección de fraude (`rules/`).
- Gestionar el emparejamiento (pairing) entre el dispositivo del mayor y el del familiar.
- Reenviar notificaciones push (vía FCM) cuando se pulsa "tengo una duda".

No se construye ningún sistema de cuentas de usuario tradicional (sin email/password) para minimizar superficie de riesgo y fricción.

---

### Decisión 3: Privacidad y procesamiento — local-first, sin IA en v1

**Decisión:** la checklist de "señales de alarma" se evalúa **en el dispositivo**, mediante reglas heurísticas (regex/listas, versionadas vía `rules/`). No se envía el contenido de mensajes/llamadas a ningún servidor.

Razones:

- Coste de inferencia cero.
- Privacidad por diseño: nada que filtrar, nada que un atacante (o el propio proyecto) pueda mal-usar.
- Auditable por la comunidad: cualquiera puede leer las reglas y verificar que no hay "vigilancia oculta".
- IA/LLM se evalúa como mejora opcional en Fase 3, una vez validada la confianza del producto y siempre on-device o con consentimiento explícito caso por caso.

---

### Decisión 4: Notificaciones al familiar — Firebase Cloud Messaging

**Decisión:** FCM para el aviso "tu [padre/madre] ha pulsado el botón de duda a las HH:MM". Es gratuito, multiplataforma (preparado para iOS futuro) y es el estándar de facto; evita construir infraestructura de push propia.

---

### Decisión 5: Licencia — Apache License 2.0

| Opción | Evaluación |
|---|---|
| MIT | Máxima simplicidad y permisividad, pero sin cláusula de patentes |
| Apache-2.0 | Permisiva, incluye concesión de patentes (protege al proyecto y a quien lo adopte, p. ej. ayuntamientos, ONGs, bancos), estándar habitual en proyectos de impacto social con vocación institucional |
| AGPL-3.0 | Obliga a compartir mejoras de cualquier fork-as-a-service, pero genera fricción/aversión en adopción institucional (muchas organizaciones evitan AGPL por política interna) |

**Decisión:** Apache-2.0. Dado que el objetivo declarado es maximizar impacto y prestigio (no ingresos), y que potenciales adoptantes institucionales (ayuntamientos, asociaciones de mayores, bancos) suelen evitar AGPL, Apache-2.0 ofrece el mejor equilibrio entre apertura, protección legal y adopción.

---

### Decisión 6: Módulos nativos críticos auditables in-repo

**Decisión:** los módulos nativos que implementan el widget de Android (RemoteViews) y la recepción de "compartir" (ACTION_SEND) se desarrollan y mantienen dentro del propio repositorio, aunque se inspiren en librerías existentes (`react-native-android-widget`, `react-native-receive-sharing-intent`). No se delega su mantenimiento a dependencias externas sin revisión, dado que son las piezas que tocan directamente los datos sensibles del usuario (contenido de mensajes compartidos).

---

## Consecuencias

**Se facilita:**
- Onboarding de contribuidores externos (stack JS/TS + .NET ampliamente conocidos).
- Evolución del feed de reglas sin republicar la app (actualización remota vía `rules/`).
- Cumplimiento RGPD por diseño (mínima superficie de datos).

**Se complica:**
- Los dos módulos nativos (widget, share-intent) requieren conocimiento de Android/Kotlin además de RN; es el principal riesgo técnico del MVP y debe abordarse como spike temprano (Milestone 1-2).
- Sin IA, la detección de patrones nuevos depende de actualizar manualmente `rules/`; aceptable para v1, a revisar en Fase 2/3.

**A revisar más adelante:**
- Si el módulo de widget en RN resulta inviable o muy costoso, reevaluar Flutter (Decisión 1) antes de comprometer más esfuerzo.
- Reevaluar la necesidad de IA on-device una vez se tenga feedback real de los primeros usuarios (Fase 2).

## Action Items

1. [ ] Crear el monorepo con la estructura definida en `ESPECIFICACION.md`.
2. [ ] Configurar CI básico (lint + build) para `apps/mobile` y `backend`.
3. [ ] Spike: PoC del módulo nativo de widget Android (RemoteViews) — validar viabilidad antes de construir el resto de la UI.
4. [ ] Definir el JSON Schema del feed de reglas (`rules/schema/`).
5. [ ] Redactar `docs/PRIVACIDAD.md` (mini DPIA) antes de implementar el pairing familiar.
6. [ ] Publicar `LICENSE` (Apache-2.0) y `CONTRIBUTING.md`.
