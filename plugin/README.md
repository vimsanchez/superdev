# Plugin `superdev` — Desarrollo de aplicaciones con Claude

Conjunto de **6 skills** que codifican un método probado para desarrollar aplicaciones
colaborando con Claude. Invocables **juntas o por separado**. Compone con el plugin
`superpowers` (no lo duplica): donde superpowers ya cubre una disciplina, estas skills la
referencian.

## Las skills

| Skill | Cuándo se activa | Qué aporta |
|---|---|---|
| `superdev:definir-proyecto` | arranque de proyecto nuevo (greenfield) | restricciones antes que stack; forma del sistema; contrato de dominio; UI; escalonado |
| `superdev:elicitar-requerimientos` | petición de feature en proyecto existente | checklist de invariantes; separar decidir-vs-preguntar; capturar reglas |
| `superdev:contexto-vivo` | tras una decisión, bug o regla no obvia | persistir el conocimiento (CLAUDE.md + memorias), automático, no opcional |
| `superdev:entrega-defensiva` | cambios a datos o al contrato de un backend | reversibilidad, compatibilidad hacia atrás, blast-radius |
| `superdev:guia` | organizar/arrancar un proyecto multi-sesión | mapa del ciclo sandbox→POC→producción + modos arquitecto→delegador→validador |
| `superdev:visily-to-app` | convertir un diseño de Visily en una app web o móvil | de exports de Visily a pantallas/design-system/brief en el stack elegido (EN; responde en el idioma del usuario) |

## Cómo se construyó

Las 5 skills del método se escribieron con **TDD para skills** (`superpowers:writing-skills`): se
observó el comportamiento base de un agente *sin* la skill (RED), se escribió la skill para
corregir los fallos concretos observados (GREEN), y se verificó con subagentes que ahora cumplen.
No enseñan lo que el modelo ya hace bien; cierran las brechas que hace de forma inconsistente
(anclar stack antes de restricciones, no persistir el conocimiento, perder la compatibilidad
hacia atrás, etc.). La skill `visily-to-app` complementa el método con un flujo concreto de
diseño→código.

## Requisito

Estas skills **referencian** al plugin [`superpowers`](https://github.com/obra/superpowers) en
los momentos donde ya hay una disciplina probada (brainstorming, writing-plans, executing-plans,
test-driven-development, systematic-debugging, verification-before-completion). Instálalo también
para aprovechar la composición completa.

## Instalación

```
/plugin marketplace add vimsanchez/vimasamo-skills
/plugin install superdev@vimasamo-skills
```

(El plugin vive en la carpeta `plugin/`; su manifiesto es `.claude-plugin/plugin.json`. El
marketplace se llama `vimasamo-skills` y su manifiesto está en `.claude-plugin/marketplace.json`
en la raíz del repo.)

### Actualizar a una versión nueva

```
/plugin marketplace update vimasamo-skills
/plugin uninstall superdev
/plugin install superdev@vimasamo-skills
```
Luego reinicia Claude Code para que la sesión cargue las skills nuevas.

## Uso

- **Automático:** cada skill se auto-activa cuando su descripción coincide con lo que estás
  haciendo (p. ej., al arrancar un proyecto nuevo se activa `superdev:definir-proyecto`).
- **Explícito:** invócalas por nombre cuando quieras, p. ej. `superdev:guia` para organizar el
  trabajo de un proyecto, o `superdev:entrega-defensiva` antes de un cambio sensible.

## Versión

0.2.0 — 6 skills (5 del método + `visily-to-app`). El método y su caso de origen (anonimizado)
están documentados aparte.
