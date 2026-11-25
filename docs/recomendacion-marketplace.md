# Dónde y cómo debe vivir el marketplace de Coworker

Investigado el 23/11/2026 en Experience League y en un marketplace público de ejemplo. Cada punto indica si está **confirmado** (lo dice la documentación de Adobe), **probable** (se ve en un ejemplo, no en la documentación) o **suposición**.

## Recomendación

Un repositorio de GitHub propio (privado de la organización, o público si se quiere vender como activo) con la estructura de marketplace de plugins de Claude Code: `.claude-plugin/marketplace.json` en la raíz y un `plugin.json` por plugin. Un administrador de Coworker lo añade como marketplace y decide qué plugins se instalan para la organización y si los usuarios pueden instalarlos o quitarlos por su cuenta. El mismo repositorio sirve para probar las skills en Claude Code antes de publicarlas.

## Qué está confirmado

- **Coworker tiene plugins, skills, MCP servers y marketplaces.** Glosario de Adobe: Marketplace es el "catálogo donde se descubren los plugins disponibles"; Plugin, un "paquete instalable de capacidades"; Skills, "instrucciones reutilizables"; MCP Servers, "conexiones gobernadas a herramientas y servicios externos". Un plugin puede incluir skills y MCP servers. ([qué son los plugins](https://experienceleague.adobe.com/en/docs/cx-enterprise-ai/experience-cloud-ai/coworker/customizations/plugins/what-are-plugins))
- **Un marketplace se añade desde un repositorio de GitHub.** La página de administración cubre "cómo añadir un nuevo marketplace usando un repositorio de GitHub", instalar y gestionar plugins para la organización, revisar las skills de un plugin antes de desplegarlo y decidir si los usuarios pueden instalar o quitar plugins. ([gestionar plugins](https://experienceleague.adobe.com/en/docs/coworker/content/customizations/plugins/manage-plugins-for-your-org)). La página es un vídeo con esa lista de temas; no hay especificación escrita.
- **Flujo de usuario**: Customizations > Plugins > explorar marketplaces > revisar > instalar > usar en el chat. Quitar un plugin quita lo que incluye.
- **Gobierno**: instalar un plugin no salta los controles de la organización ni da acceso a productos o acciones que el usuario no tenga ya. Coworker declara permisos heredados de la organización de Adobe, aprobaciones y registro de auditoría.
- **Las skills también se crean desde el chat**: Coworker detecta un patrón repetido, extrae el flujo y el usuario lo publica como skill. ([primera skill](https://experienceleague.adobe.com/en/docs/coworker/content/customizations/skills/create-your-first-skill))
- **Coworker ya trae skills nativas de AJO** de lectura y análisis: análisis de conflictos, fallout, custom actions, anomalías, comparación de versiones, higiene de journeys, traza de un perfil y línea de tiempo de ejecución, además de la creación de journeys. Nuestra skill debe usarlas como fuente de datos, no competir con ellas. Su valor añadido es la separación diseño/rendimiento, los controles verificables, la verificación de falsos positivos y la salida fija.

## Qué es probable, pero no está documentado

- **El formato de archivo es el de Claude Code.** El único marketplace público para Coworker que encontré ([adobe-sean/seans_marketplace](https://github.com/adobe-sean/seans_marketplace), "my plugins for CX Coworker") usa `.claude-plugin/marketplace.json` con `name`, `owner`, `metadata` y una lista `plugins` (`name`, `source`, `description`, `version`, `author`, `dependencies`), y `plugins/<tipo>/<plugin>/.claude-plugin/plugin.json`. Lo he copiado tal cual. Adobe no publica la especificación.
- **Separación `system` / `practitioner`**: el ejemplo separa plugins de sistema (acceso a datos, hooks) y de practicante (flujos de trabajo). He puesto la auditoría en `practitioner`.
- **Frontmatter de SKILL.md**: el ejemplo pide `name`, `description` y `metadata` con `version` y `lifecycle: experimental` para skills nuevas, y casos en `evals/evals.json`. Lo he incluido.
- **Disponibilidad en Claude y otros asistentes**: Adobe anuncia Coworker dentro de ChatGPT, Claude, Copilot y Gemini. No está claro si los plugins de un marketplace propio viajan a esas superficies.

## Qué sigue sin confirmar (preguntar a Adobe)

1. Especificación oficial del marketplace y de `plugin.json`: campos obligatorios, límites de tamaño de SKILL.md y de `references/`.
2. Si un repositorio privado de GitHub funciona como marketplace y con qué credencial lo lee Coworker.
3. Si un MCP declarado en el plugin (`.mcp.json`) hereda la autenticación del usuario de Coworker (ya estaba en las preguntas abiertas de la guía).
4. Qué campos de la definición de journey devuelve la herramienta de AJO (exit criteria y merge policy dieron "no expuestos" en la primera auditoría).
5. Si se puede listar un marketplace propio para clientes de otras organizaciones (modelo de venta del health check).

## Firefly y AEM Assets

- **Firefly: confirmado como capacidad nativa.** La documentación de flujos de Coworker incluye una skill `generate-image` que "genera, recorta, superpone, crea variantes y firma imágenes de campaña con Firefly".
- **AEM Assets: confirmado.** Skills nativas de descubrimiento de assets (búsqueda semántica, por metadatos, estado de aprobación y carpeta), gestión de carpetas, taxonomía de tags, formularios de metadatos y optimización de contenido (renditions, overlays, color de fondo, enfoque, espejo).
- **GenStudio for Performance Marketing**: no lo vi en el texto revisado.
- **Implicación para el pre-flight**: la edición de imágenes con brand guidelines es viable técnicamente, porque hay herramientas de Firefly y AEM Assets. Falta confirmar en la instancia de cada cliente que esas skills están activas y qué permisos exigen. Sigue teniendo sentido empezar por un bloque de verificación de marca dentro del pre-flight y añadir la corrección como paso opcional, con aprobación humana.

## Fuentes

- [Qué son los plugins de Coworker](https://experienceleague.adobe.com/en/docs/cx-enterprise-ai/experience-cloud-ai/coworker/customizations/plugins/what-are-plugins)
- [Gestionar plugins para la organización](https://experienceleague.adobe.com/en/docs/coworker/content/customizations/plugins/manage-plugins-for-your-org)
- [Qué son las skills](https://experienceleague.adobe.com/en/docs/cx-enterprise-ai/experience-cloud-ai/coworker/customizations/skills/what-are-skills)
- [Crear tu primera skill](https://experienceleague.adobe.com/en/docs/coworker/content/customizations/skills/create-your-first-skill)
- [Customizations overview](https://experienceleague.adobe.com/en/docs/coworker/content/customizations/overview)
- [Integraciones (MCP)](https://experienceleague.adobe.com/en/docs/cx-enterprise-ai/experience-cloud-ai/coworker/customizations/integrations/understanding-integrations-in-coworker)
- [Workflows y skills nativas](https://experienceleague.adobe.com/en/docs/coworker/content/chat/use-cases/workflows)
- [Chat overview y aplicaciones compatibles](https://experienceleague.adobe.com/en/docs/cx-enterprise-ai/experience-cloud-ai/coworker/chat/overview)
- [Coworker en ChatGPT](https://business.adobe.com/blog/cx-enterprise-coworker-in-chatgpt)
- [Marketplace de ejemplo: adobe-sean/seans_marketplace](https://github.com/adobe-sean/seans_marketplace)
