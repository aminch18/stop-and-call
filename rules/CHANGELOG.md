# Changelog — rules/es-ES/ruleset.json

Formato: una entrada por versión, siguiendo SemVer (ver `.claude/skills/add-fraud-rule/SKILL.md`).

## [1.0.0] - 2026-06-12

Versión inicial: 8 patrones basados en avisos públicos de INCIBE, Policía Nacional y OCU
sobre fraudes comunes dirigidos a personas mayores en España.

- Añadido: `seguridad-social-bizum-request` — suplantación de la Seguridad Social pidiendo pago por Bizum.
- Añadido: `banco-enlace-urgente` — smishing bancario (cuenta/tarjeta bloqueada + enlace).
- Añadido: `llamada-prefijo-internacional-sospechoso` — llamadas con prefijos internacionales de alto coste.
- Añadido: `familiar-en-apuros-whatsapp` — suplantación de un familiar desde un "número nuevo" pidiendo dinero.
- Añadido: `paquete-pendiente-enlace` — smishing de paquetería (tasa de aduana, enlace de pago).
- Añadido: `codigo-verificacion-solicitado` — solicitud de código SMS/Bizum por teléfono.
- Añadido: `oferta-trabajo-facil-dinero` — estafas de "mulas bancarias".
- Añadido: `soporte-tecnico-falso` — falso soporte técnico pidiendo instalar acceso remoto.
