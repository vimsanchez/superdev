---
name: elicitar-requerimientos
description: Usar cuando llega una petición de funcionalidad o cambio en un proyecto ya en marcha ("agrégame", "implementa", "necesito que se pueda…"), antes de diseñar o codificar — para descubrir los invariantes ocultos que la petición no menciona.
---

# Elicitar requerimientos y reglas de negocio

## Overview

Cuando llega una petición de feature, las palabras describen lo que el usuario quiere; **el
riesgo está en los invariantes que no mencionó**. El movimiento valioso NO es "hacer preguntas"
(los agentes ya preguntan), sino tres cosas que se hacen de forma inconsistente:

1. **Sondear un checklist fijo** de invariantes generalizables, para que no se olvide ninguno.
2. **Separar lo que debes DECIDIR (default seguro) de lo que debes PREGUNTAR** (valor del
   dominio que no puedes adivinar).
3. **Capturar las reglas no obvias como artefacto durable**, no dejarlas en el chat.

> Principio: no preguntes al usuario lo que deberías decidir tú; no decidas lo que solo él sabe.

## Cuándo usar

- Petición de funcionalidad/cambio en un proyecto existente: "agrégame…", "implementa…",
  "necesito que se pueda…".
- Antes de diseñar o escribir código de la feature.
- **No usar** para arranque de proyecto nuevo (usa `superdev:definir-proyecto`).

## El método

### 1. Sondea el checklist de invariantes generalizables

No confíes en la memoria. Recorre esta lista en cada feature relevante:

| Invariante generalizable | Qué sondear | ¿Default o preguntar? |
|---|---|---|
| **Unicidad** | ¿Debe ser único? ¿bajo qué clave? | Default: único; el alcance exacto → preguntar |
| **Alcance de autorización** | ¿Quién puede hacerlo y sobre qué subconjunto? | **Preguntar** (regla de negocio) |
| **Criterio de "completo"** | ¿Qué hace válido/terminado un registro? | **Preguntar** |
| **Degradación ante fallo** | ¿Qué pasa si una dependencia falla? | Default: degradar seguro (0 / rechazar), no romper |
| **Idempotencia / reintentos** | ¿El cliente reintenta? ¿se puede duplicar? | Default: idempotente |
| **Concurrencia** | ¿Dos actores a la vez sobre el mismo dato? | Default: transacción + chequeo |
| **Compatibilidad con clientes en campo** | ¿Hay apps/consumidores que no se pueden actualizar? | **Preguntar** + diseñar compatible |
| **Reversibilidad** | ¿El cambio de datos necesita poder revertirse? | Default: backup + operación idempotente |

### 2. Separa DECIDIR vs PREGUNTAR

- **Decide tú** (propón el default seguro y avanza): los invariantes de ingeniería —unicidad,
  no-negativo, idempotencia, transacción, degradación a 0, reversibilidad—. Anúncialos, no los
  conviertas en tarea del usuario.
- **Pregunta** (no puedes adivinarlos): los valores del dominio —umbrales, cantidades
  esperadas, qué cuenta como "completo", quién puede actuar sobre qué (alcance de permisos),
  compatibilidad con clientes en campo—.

### 3. Valida las reglas críticas contra datos reales

Cuando una regla tiene números (umbrales, cantidades, cobertura), pruébala contra un caso real
antes de darla por buena y registra el resultado numérico.

### 4. Captura las reglas no obvias como artefacto durable

Toda regla de negocio no obvia (especialmente un invariante como "uno por X, activos e
inactivos") se escribe como memoria/registro de decisión con su porqué. → usa
`superdev:contexto-vivo`. Si se queda en el chat, se re-litiga o se re-rompe.

Para diseño genuinamente abierto, invoca `superpowers:brainstorming`. El resultado de esta
elicitación alimenta el plan (`superpowers:writing-plans`).

## Errores comunes

- **Preguntar al usuario lo que deberías decidir** (¿permito stock negativo?) en vez de proponer
  el default seguro.
- **Asumir un default donde había que preguntar** (un umbral, una cantidad, quién puede actuar).
- **Sondear de memoria** y olvidar una clase de invariante — usa el checklist.
- **Dejar la regla en el chat** en vez de capturarla como memoria con su porqué.
- **No validar contra datos reales** una regla con números.
