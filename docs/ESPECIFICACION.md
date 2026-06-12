# Especificación de repositorio, stack y roadmap

> Documento complementario a `ADR-0001-arquitectura-base.md`. Define la estructura concreta del monorepo, el stack detallado y el plan de fases para el MVP en España.

## 1. Resumen del producto

Aplicación móvil gratuita y open source (Android-first) que ofrece a personas mayores un punto único, simple y siempre visible (widget de pantalla de inicio) con tres acciones:

1. **Llamar a un familiar de confianza** (un toque).
2. **Comprobar un mensaje sospechoso** (compartir desde WhatsApp/SMS/correo → checklist visual de señales de alarma, basada en reglas locales específicas de España).
3. **Llamar a la línea 017 de INCIBE** (recurso oficial gratuito).

El familiar configura la app de forma remota/inicial y puede recibir un aviso opcional cuando se usa la acción 2.

## 2. Estructura del repositorio (monorepo)

```
compania-digital/                      # nombre provisional del repo
├── apps/
│   └── mobile/
│       ├── android/
│       │   └── app/src/main/java/.../widget/    # módulo nativo: AppWidgetProvider
│       │   └── app/src/main/java/.../share/     # módulo nativo: ShareReceiverActivity
│       ├── ios/                                  # placeholder, Fase 3
│       ├── src/
│       │   ├── screens/
│       │   │   ├── HomeScreen.tsx
│       │   │   ├── ChecklistScreen.tsx
│       │   │   └── FamilySetupScreen.tsx
│       │   ├── components/
│       │   ├── rules-engine/        # heurísticas TS, portables y testeables
│       │   ├── services/            # cliente API, pairing, notificaciones
│       │   └── i18n/                # es-ES (y en-US para contribuidores)
│       ├── __tests__/
│       ├── app.json
│       ├── package.json
│       └── README.md
├── backend/
│   ├── src/
│   │   ├── Api/                     # Minimal API endpoints (Program.cs, etc.)
│   │   ├── Application/             # casos de uso: Pairing, RulesFeed, AlertNotification
│   │   ├── Domain/                  # entidades: Device, FamilyLink, RuleSetVersion
│   │   └── Infrastructure/          # EF Core, FCM client, persistencia
│   ├── tests/
│   └── README.md
├── rules/
│   ├── schema/
│   │   └── ruleset.schema.json      # JSON Schema de validación
│   ├── es-ES/
│   │   └── ruleset.json             # patrones: Bizum, Seguridad Social, +44, urgencia, etc.
│   └── CHANGELOG.md
├── docs/
│   ├── adr/
│   │   └── 0001-arquitectura-base.md
│   ├── ESPECIFICACION.md            # este documento
│   ├── PRIVACIDAD.md                # mini DPIA, modelo de datos
│   └── CONTRIBUTING.md
├── .github/
│   └── workflows/
│       ├── mobile-ci.yml
│       └── backend-ci.yml
├── LICENSE                           # Apache-2.0
└── README.md
```

## 3. Stack tecnológico detallado

### 3.1 Mobile (`apps/mobile`)

| Capa | Elección | Notas |
|---|---|---|
| Framework | React Native (bare workflow) + TypeScript estricto | Sin Expo managed, por necesidad de módulos nativos |
| Navegación | React Navigation (stack simple, 3 pantallas) | Evitar complejidad: sin tabs, sin deep linking en v1 |
| Estado | React Context / hooks | Sin Redux/MobX; el estado es trivial (config familiar + caché de reglas) |
| Widget Android | Módulo nativo Kotlin (`AppWidgetProvider` + `RemoteViews`), inspirado en `react-native-android-widget` | Spike de viabilidad en Milestone 1 |
| Share intent | Módulo nativo Kotlin (`ACTION_SEND` receiver) | Inspirado en `react-native-receive-sharing-intent`, mantenido in-repo |
| Motor de reglas | TypeScript puro (`rules-engine/`), sin dependencias nativas | Permite testear con Jest sin emulador |
| Notificaciones push | `@react-native-firebase/messaging` | FCM |
| Accesibilidad | Tipografía grande por defecto, alto contraste, soporte TalkBack | Requisito no funcional crítico |
| Testing | Jest + React Native Testing Library | Cobertura mínima: `rules-engine` al 100% |
| Lint/format | ESLint + Prettier + `tsconfig strict` | CI bloqueante |

### 3.2 Backend (`backend/`)

| Capa | Elección | Notas |
|---|---|---|
| Framework | ASP.NET Core Minimal API, .NET 9 | |
| Persistencia | SQLite (MVP) → PostgreSQL (producción) vía EF Core | Migración trivial por estar en EF Core desde el inicio |
| Autenticación | Sin cuentas tradicionales: pairing por código corto de un solo uso + token de dispositivo | Minimiza superficie de datos personales |
| Push | Firebase Admin SDK (FCM HTTP v1) | |
| Hosting objetivo | Fly.io / Railway / Azure App Service (free/low tier) | Bajo coste, fácil migración a self-host |
| Testing | xUnit + WebApplicationFactory para tests de integración de endpoints | |

### 3.3 Reglas de detección (`rules/`)

- Formato JSON validado contra `ruleset.schema.json`.
- Versionado semántico (`version: "1.0.0"`) + `lastUpdated`.
- Servido por el backend con cabecera ETag para que el cliente cachee y solo descargue cambios.
- Contenido inicial `es-ES` basado en avisos públicos de INCIBE: smishing bancario, suplantación de Seguridad Social (Bizum), prefijos internacionales sospechosos (+44 y similares), patrones de "urgencia + petición de datos/dinero", enlaces acortados/sospechosos.

### 3.4 Infraestructura y herramientas

- CI: GitHub Actions (`mobile-ci.yml`: lint + test + build Android debug; `backend-ci.yml`: build + test).
- Convenciones: Conventional Commits, changelog automático (`rules/CHANGELOG.md` para reglas; changelog general por release).
- Licencia: Apache-2.0 (ver ADR-0001, Decisión 5).
- Idioma: código y comentarios en inglés; documentación de usuario y contribución en español (con traducción a inglés progresiva para atraer contribuidores internacionales).

## 4. Modelo de privacidad (resumen — detalle en `docs/PRIVACIDAD.md`)

- Por defecto, la app funciona **100% local**: la checklist se evalúa en el dispositivo, sin enviar contenido a ningún servidor.
- El emparejamiento familiar es **opt-in**, configurado por el familiar durante el setup inicial.
- Lo único que puede salir del dispositivo (si el emparejamiento está activo) es: "se pulsó el botón de duda" + marca de tiempo. **Nunca** el contenido del mensaje analizado.
- El feed de reglas (`rules/`) es público y auditable por cualquiera.

## 5. Roadmap

### Fase 0 — Completada
Investigación de mercado (España/Portugal), análisis competitivo, ADR-0001.

### Fase 1 — MVP funcional (objetivo: 6 semanas)

| Hito | Semana | Entregable |
|---|---|---|
| M1.1 | 1–2 | Scaffolding del monorepo según sección 2, CI básico, app RN con 3 pantallas (sin lógica), backend con health-check |
| M1.2 | 3 | Spike y PoC del módulo nativo de widget Android |
| M1.3 | 4 | Módulo de share-intent + `rules-engine` con ruleset `es-ES` v1 + `ChecklistScreen` funcional |
| M1.4 | 5 | Pairing familiar (código corto) + notificación FCM al pulsar "tengo una duda" |
| M1.5 | 6 | Pruebas con usuarios reales (padres), ajustes de UX (tamaño de texto, contraste, claridad del lenguaje) |

**Criterios de éxito del MVP:**
- Los padres usan el widget sin ayuda tras una explicación de ~5 minutos.
- La checklist identifica correctamente los patrones de fraude más comunes en España (smishing bancario, "Seguridad Social + Bizum", llamadas +44, "familiar en apuros").
- Cero crashes reportados durante 2 semanas de uso real.

### Fase 2 — Iteración basada en uso real (4–8 semanas)
- Ampliación de `rules/es-ES` según feedback real.
- Telemetría anónima opt-in (solo conteos agregados, sin contenido).
- Métricas de adopción y retención.

### Fase 3 — Expansión (a definir tras Fase 2)
- Soporte iOS (Action Extension; los widgets interactivos son más limitados, requiere diseño específico).
- Evaluar IA on-device para casos ambiguos no cubiertos por reglas.
- Posibles colaboraciones institucionales (INCIBE, ayuntamientos, asociaciones de mayores) para distribución — sin comprometer la naturaleza open source/gratuita del proyecto.

## 6. Convenciones de desarrollo

- **Branching:** `main` protegido, ramas `feature/*`, PRs obligatorios con CI verde.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`...).
- **Definición de "hecho" por hito:** código + tests + actualización de `docs/` si aplica + CI verde.
- **Prioridad de diseño:** ante cualquier duda, elegir la opción más simple para el usuario final, aunque sea más compleja para el desarrollador.
