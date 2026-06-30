---
name: guia
description: Usar al organizar o arrancar el desarrollo de una aplicación con el usuario a lo largo de varias sesiones ("¿cómo nos organizamos?", "guíame en el desarrollo", planear un proyecto multi-parte) — mapa que enruta el ciclo y las skills del método.
---

# Guía de desarrollo de aplicaciones con Claude

## Overview

Punto de entrada y **mapa** del método para desarrollar aplicaciones con Claude. Dos ideas
ordenan todo el trabajo:

1. **El trabajo avanza en etapas (in crescendo): sandbox → POC → producción.** Cada etapa
   valida algo antes de pasar a la siguiente; empieza siempre por un *vertical slice*.
2. **La colaboración evoluciona:** tu rol cambia de **arquitecto → delegador → validador** a
   medida que el proyecto madura. Invertir en contexto temprano (modo arquitecto) es lo que
   habilita delegar con autonomía después.

Cada skill del método se auto-activa por su cuenta cuando aplica; esta guía sirve para
**secuenciar** y para reconocer en qué **modo** estás.

## Cuándo usar

- Al arrancar u organizar un proyecto que durará varias sesiones.
- "¿Cómo nos organizamos?", "guíame", "trabajemos juntos en esto".
- Para situarte cuando no sabes qué paso toca.

## Mapa del ciclo → qué skill usar

| Momento | Qué pasa | Skill del método | Compón con superpowers |
|---|---|---|---|
| **Génesis** (arranque) | forma del sistema, restricciones, contrato de dominio, UI, escalonado | `superdev:definir-proyecto` | `brainstorming`, `writing-plans` |
| **Cada feature/cambio** | descubrir invariantes ocultos; decidir vs preguntar | `superdev:elicitar-requerimientos` | `brainstorming` |
| **Construcción** | ejecutar por fases con checkpoints; paralelizar exploración | — | `writing-plans`, `executing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `test-driven-development` |
| **Tras decisión/bug/regla** | persistir el conocimiento (no dejarlo en el chat) | `superdev:contexto-vivo` | — |
| **Cambios a datos o al contrato** | reversibilidad, compatibilidad, blast-radius | `superdev:entrega-defensiva` | `systematic-debugging` |
| **Antes de dar por terminado** | verificar contra la realidad | — | `verification-before-completion`, `requesting-code-review` |

## Los tres modos de colaboración

| Modo | Cuándo | El usuario… | Tú (Claude)… |
|---|---|---|---|
| **Arquitecto** | el sistema aún no existe o se reestructura | co-diseña, decide arquitectura, verifica cada incremento | exploras, propones opciones en menú, planeas |
| **Delegador** | construcción intensa de features | da briefs, aprueba diseños, interrumpe con info nueva | ejecutas con autonomía, concentras decisiones, paralelizas |
| **Validador** | el sistema ya opera | reporta síntomas, decide en una línea, exige garantías | diagnosticas con evidencia, haces cambios quirúrgicos y reversibles |

Reconoce el modo y ajústate: no co-diseñes cuando el usuario solo quiere validar, ni ejecutes a
ciegas cuando aún hay que decidir arquitectura.

## Principios transversales (toda etapa)

- **Contexto como cimiento:** mantén vivo el `CLAUDE.md` y las memorias → `superdev:contexto-vivo`.
- **Verifica contra la realidad:** prueba cada incremento; valida reglas con datos reales.
- **Defensa por defecto:** asume datos irrecuperables y clientes que no puedes actualizar →
  `superdev:entrega-defensiva`.
- **Conciso para ejecutar, extenso para dar contexto:** aporta lo que solo tú sabes.

## Errores comunes

- Saltar la génesis y empezar a codificar (define primero → `superdev:definir-proyecto`).
- Trabajar siempre en el mismo modo sin notar que el proyecto maduró.
- Construir todo a la vez en vez de escalonar (sandbox → POC → producción, vertical slice).
- Olvidar persistir decisiones/bugs (se re-litigan → `superdev:contexto-vivo`).
