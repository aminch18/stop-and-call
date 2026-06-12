# Guía de contribución

Gracias por tu interés en este proyecto. Su objetivo es ayudar a personas mayores a protegerse
de fraudes digitales, de forma gratuita y abierta. Toda contribución es bienvenida: código,
traducciones, patrones de fraude nuevos, diseño, documentación o pruebas con usuarios reales.

## Configuración del entorno

### Mobile (`apps/mobile`)
Requisitos: Node 20+, JDK 17, Android Studio (SDK + emulador o dispositivo físico).
```bash
cd apps/mobile
npm install
npm run android   # build y ejecución en debug
```

### Backend (`backend`)
Requisitos: .NET 9 SDK.
```bash
cd backend
dotnet build
dotnet run --project src/Api
# GET http://localhost:5000/health debería devolver 200 OK
```

## Flujo de trabajo

1. Crea una rama `feature/<nombre-corto>` desde `main`.
2. Commits siguiendo [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`,
   `fix:`, `docs:`, `chore:`, `test:`).
3. Antes de abrir el PR: `npm run lint && npm run typecheck && npm test` (mobile) y
   `dotnet test` (backend) deben pasar.
4. Abre el PR contra `main`. El CI debe estar en verde.
5. Si tu cambio afecta a una decisión de arquitectura no cubierta por `docs/adr/0001-arquitectura-base.md`,
   propón un borrador `docs/adr/000X-tu-decision.md` en el mismo PR.

## Idioma

- Código, nombres de variables/funciones, comentarios técnicos: **inglés**.
- Documentación de usuario, contribución y textos visibles en la app: **español (es-ES)**,
  con traducción progresiva a inglés para facilitar contribuciones internacionales.

## Checklist de tono y accesibilidad (obligatoria para cualquier pantalla/texto nuevo)

Este proyecto está dirigido a personas mayores con baja alfabetización tecnológica. Antes
de añadir o modificar cualquier texto o pantalla visible para el usuario final, comprueba:

- [ ] **Sin jerga técnica**: nada de "token", "caché", "sincronizar", "permisos"... Si hace
      falta explicar un concepto técnico, búscale una analogía cotidiana o elimínalo.
- [ ] **Frases cortas y directas**, en segunda persona ("Pulsa aquí para llamar a Marta").
- [ ] **Tipografía grande** (mínimo 18sp en texto normal, 24sp+ en botones principales).
- [ ] **Alto contraste** (cumple WCAG AA como mínimo).
- [ ] **Máximo 3 opciones visibles** por pantalla. Si necesitas una cuarta, replantea el flujo.
- [ ] **Soporte TalkBack**: todo elemento interactivo tiene `accessibilityLabel` descriptivo.
- [ ] **Sin "callejones sin salida"**: siempre debe haber una forma obvia de volver atrás
      o de pedir ayuda (botón 017 / familiar).

## Cómo añadir un nuevo patrón de fraude

No edites `rules/es-ES/ruleset.json` a mano sin más. Usa la skill
`.claude/skills/add-fraud-rule/SKILL.md` (si trabajas con Claude Code) o sigue manualmente
sus pasos: validar contra el schema, añadir test en `rules-engine/__tests__`, actualizar
`rules/CHANGELOG.md` y bump de versión.

## Código de conducta

Sé respetuoso. Este proyecto trata sobre proteger a personas vulnerables; el mismo cuidado
y empatía deben reflejarse en cómo colaboramos entre nosotros.
