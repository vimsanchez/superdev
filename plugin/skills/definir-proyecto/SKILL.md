---
name: definir-proyecto
description: Usar al arrancar un proyecto de software nuevo desde una idea o petición vaga, antes de elegir stack o escribir código (greenfield, scoping inicial, "ayúdame a empezar", "quiero una app/sistema para...", definir la arquitectura).
---

# Definir un proyecto de software nuevo

## Overview

La génesis de un proyecto produce **cuatro definiciones y las deja por escrito** antes de
elegir stack o escribir código. El error caro no es "no preguntar" — los agentes ya preguntan
y proponen un MVP. El error caro es más sutil: **anclar un stack antes de conocer las
restricciones inamovibles**, y **dejar la génesis solo en el chat**, donde se pierde.

> Principio: si no registras la génesis, la pierdes. Las decisiones de arranque son el ADN del
> proyecto; deben quedar en archivos, no en la conversación.

## Cuándo usar

- Petición vaga de proyecto nuevo: "ayúdame a empezar", "quiero un sistema/app para…".
- Antes de recomendar arquitectura, stack o generar código.
- **No usar** para cambios dentro de un proyecto ya definido (usa `superdev:elicitar-requerimientos`
  y `superdev:contexto-vivo`).

## La receta de génesis (en orden)

### 1. Restricciones inamovibles — ANTES de cualquier stack

Pregunta qué ya está dado y no se negocia, porque define o descarta el stack:
- Base de datos / motor existente en producción
- Hosting / nube / on-premise obligado
- Autenticación, correo (SMTP), dominios disponibles
- Clientes o sistemas en campo con los que hay que ser compatible

**No recomiendes un stack hasta tener esto.** Solo después, propón opciones de stack como un
menú razonado y deja que el usuario elija (no asumas).

### 2. Forma del sistema — declarada explícitamente

Enumera y **confirma** qué piezas tendrá: cliente (¿web?, ¿móvil?), backend, base de datos,
modelo/ML, dashboard, jobs. No asumas la forma; lo que en un proyecto heredado viene implícito,
en uno nuevo hay que decidirlo a propósito. Muchos proyectos son solo una parte (solo web, solo
backend): confírmalo, no construyas de más.

### 3. Contrato de dominio — como artefacto escrito

Entidades, reglas y el contrato de interfaz que el cliente/frontend consumirá. **Escríbelo en
un archivo** (`CONTRATO.md` / spec), no como modelo de datos suelto en el chat. Es la fuente de
verdad que todo lo demás consume.

### 4. Enfoque de UI — definido o aportado

Decide cómo se especifica la UI y déjalo escrito (un `PLAN.md` de pantallas):
- Prosa estructurada + diagramas ASCII del flujo de pantallas, **o**
- Mockups que el usuario aporta (pídelos explícitamente si la estética importa).

### Luego: persistir la génesis y escalonar

- **Persiste**: crea/siembra `CLAUDE.md` (mapa + comandos + decisiones fechadas), el `PLAN.md`
  y el `CONTRATO.md`. → usa `superdev:contexto-vivo`.
- **Escalona el trabajo (in crescendo): sandbox → POC → producción**, declarando qué valida
  cada etapa. Empieza por un *vertical slice* (un flujo completo de punta a punta) antes de
  ampliar.
- Para alinear el diseño, invoca `superpowers:brainstorming`. Para convertirlo en un plan por
  fases, invoca `superpowers:writing-plans`. Para elicitar reglas de negocio a fondo, usa
  `superdev:elicitar-requerimientos`.

## Quick reference

| Definición | Pregunta clave | Artefacto que deja |
|---|---|---|
| Restricciones | "¿Qué ya está dado por producción que no se negocia?" | nota en `CLAUDE.md` |
| Forma del sistema | "¿Qué piezas tendrá y cuáles NO?" | `CLAUDE.md` |
| Contrato de dominio | "¿Qué entidades, reglas e interfaz consume el cliente?" | `CONTRATO.md` |
| Enfoque de UI | "¿Hay mockups o lo describimos en prosa?" | `PLAN.md` |
| Etapas | "¿Qué valida sandbox, qué valida el POC?" | `PLAN.md` |

## Errores comunes

- **Recomendar stack antes de confirmar restricciones de producción.** El más frecuente; ancla
  el proyecto en tecnología que quizá viola un requisito no dicho.
- **Dejar la génesis solo en el chat.** Las decisiones de arranque se pierden; el siguiente
  agente (o persona) empieza de cero.
- **Asumir la forma del sistema** en vez de declararla y confirmarla.
- **Modelo de datos inline** en vez de un contrato escrito.
- **Saltar al código sin etapa de validación** (sin vertical slice ni sandbox→POC).
