# Marketplace de skills para Adobe CX Enterprise Coworker

Plugins del "health check" de AJO/AEP. El trío se encadena de la causa al síntoma: datos, luego journey, luego contenido.

| Plugin | Estado | Qué hace |
| --- | --- | --- |
| `auditoria-journeys-ajo` | 0.2.0, experimental | Auditoría de journeys: revisión de diseño y de rendimiento, scorecard y backlog priorizado |
| `auditoria-datos-aep` | Pendiente | Fundamentos de datos en AEP: identidad, schemas, audiencias, merge policies, licencia |
| `preflight-mensajes` | Pendiente | Go/No-Go antes de publicar: personalización, enlaces, consentimiento, accesibilidad, marca |

## Estructura

```
.claude-plugin/marketplace.json          catálogo del marketplace
plugins/practitioner/<plugin>/
  .claude-plugin/plugin.json             manifiesto del plugin
  skills/<skill>/SKILL.md                instrucciones de la skill
  skills/<skill>/references/*.md         material que la skill carga cuando lo necesita
  evals/evals.json                       casos de prueba genéricos
docs/                                    notas de diseño del marketplace
```

## Instalación

- **Coworker**: un administrador añade este repositorio como marketplace desde GitHub y lo instala para la organización (Customizations > Plugins). Ver `docs/recomendacion-marketplace.md`.
- **Claude Code** (para probar las skills fuera de Coworker): `/plugin marketplace add <owner>/<repo>` y `/plugin install auditoria-journeys-ajo@coworker-skills-ajo-aep`.

## Reglas para añadir una skill

1. Solo lectura sobre la instancia del cliente, salvo que la skill diga lo contrario de forma explícita.
2. Cada hallazgo con evidencia concreta; lo que falta se declara.
3. Salida fija y documentada en `references/`.
4. Casos de prueba en `evals/evals.json`, sin identificadores de clientes.
5. Subir `version` en `plugin.json` y en `marketplace.json` con cada cambio.
