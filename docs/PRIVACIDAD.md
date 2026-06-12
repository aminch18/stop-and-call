# Privacidad y modelo de datos (mini DPIA)

> Referenciado desde `ADR-0001` (Decisión 3) y `CLAUDE.md`. Cualquier cambio que afecte
> a qué datos salen del dispositivo debe actualizar este documento primero.

## Principio rector

La app debe funcionar de forma útil **sin enviar nunca el contenido** de mensajes, llamadas
o capturas a ningún servidor. Todo el análisis de la checklist ocurre en el dispositivo
del usuario, usando `rules-engine/` (TypeScript puro, sin red).

## Qué datos existen y dónde viven

| Dato | Dónde vive | Sale del dispositivo? | Notas |
|---|---|---|---|
| Contenido de mensajes/SMS/capturas compartidos a la app | Memoria del dispositivo, descartado tras el análisis | **No, nunca** | Procesado por `rules-engine/`, no persistido |
| Resultado de la checklist (qué señales se detectaron) | Memoria del dispositivo | No | Se muestra al usuario y se descarta |
| Evento "se pulsó el botón de duda" + timestamp | Backend (solo si pairing activo) | **Sí, opt-in** | Es lo único que puede notificarse al familiar |
| Código de pairing + token de dispositivo | Backend | Sí (necesario para el pairing) | Sin datos personales asociados (no nombre, no teléfono) |
| Versión del ruleset descargado | Backend (vía ETag) | Sí (metadato técnico) | Para servir actualizaciones de `rules/` |
| Telemetría anónima (Fase 2, opt-in) | Backend | Sí, opt-in | Solo conteos agregados (ej. "checklist usada N veces"), nunca contenido |

## Base legal (RGPD)

- **Pairing familiar y notificaciones**: consentimiento explícito del familiar durante el setup
  (Art. 6.1.a RGPD). Se explica en un único paso, en lenguaje llano, qué se comparte y qué no.
- **Funcionamiento local de la checklist**: no implica tratamiento de datos personales por
  parte del responsable del proyecto (no hay transmisión ni almacenamiento fuera del dispositivo).
- **Telemetría (Fase 2)**: opt-in separado, desactivado por defecto.

## Derechos del usuario

- El familiar puede desemparejar el dispositivo en cualquier momento (borra el registro
  de pairing en el backend).
- No existe "cuenta" que dar de baja: sin email/password, sin perfil persistente del
  usuario mayor.
- El código fuente de `rules-engine/` y del backend es público (Apache-2.0): cualquiera
  puede auditar que esta tabla se cumple.

## Medidas de seguridad mínimas (backend)

- HTTPS obligatorio en todos los endpoints.
- Tokens de dispositivo con expiración y posibilidad de revocación.
- Sin logs que contengan contenido de mensajes (nunca llega contenido al backend, así
  que esto debería ser trivialmente cierto — verificar en cada PR que toque `Api/`).
- Base de datos (SQLite/PostgreSQL) solo contiene: `DeviceId`, `PairingCode`, `FamilyDeviceToken`,
  `RuleSetVersionServed`, timestamps. Ningún campo de texto libre proveniente del usuario.

## Checklist para revisar en cada PR

- [ ] ¿Esta PR añade alguna llamada de red desde `apps/mobile` que envíe contenido de
      mensajes, capturas o resultados detallados de la checklist? → Si sí, **rechazar**
      o exigir nueva ADR.
- [ ] ¿Esta PR añade un nuevo campo a la base de datos del backend? → Verificar que no
      sea texto libre proveniente del usuario mayor.
- [ ] ¿Esta PR cambia el flujo de consentimiento del pairing? → Actualizar este documento
      y el texto mostrado al familiar en `FamilySetupScreen`.
