---
name: entrega-defensiva
description: Usar antes de ejecutar cualquier cambio que escriba datos (carga, migración, UPDATE masivo) o que toque el contrato de un backend (campos, enums/estados, endpoints, formatos), aunque la petición no mencione "producción" ni "clientes en campo".
---

# Entrega defensiva (cambios seguros sobre lo que ya existe)

## Overview

Un cambio que toca datos o el contrato de un backend puede romper cosas que no ves: datos que
no podrás recuperar, o clientes desplegados que no puedes actualizar. El reflejo defensivo debe
dispararse **aunque el usuario no mencione "producción"**. El fallo típico: cuando la petición
no señala producción ni clientes en campo, se pierde sobre todo la **compatibilidad hacia
atrás** — se agregan estados/enums nuevos sin pensar en las apps viejas que crashean.

> Principio: asume que hay datos que no puedes recuperar y clientes que no puedes actualizar,
> hasta confirmar lo contrario.

## Cuándo usar

- Cualquier cambio que **escriba datos** (carga / migración / UPDATE masivo).
- Cualquier cambio al **contrato de un backend** (campos, enums/estados, endpoints, formatos).
- Antes de **aprobar o ejecutar**, no después.

## Los tres reflejos

### 1. Reversibilidad en todo cambio de datos

Antes de escribir: **backup** nombrado por incidente, **dry-run / diff** para revisar qué va a
cambiar, operación **idempotente** (re-ejecutable sin daño), y un **procedimiento de rollback**
explícito. Pregunta el entorno si no lo sabes. Aplica en transacción y verifica el conteo antes
de confirmar.

### 2. Compatibilidad con quien no puedes actualizar  ← el reflejo que más se olvida

Antes de cambiar el contrato del backend, pregunta: **¿quién consume esto que no puedo
actualizar?** (apps desplegadas en campo, integraciones externas).
- Cambios **aditivos** son seguros (un campo opcional nuevo). **Quitar / renombrar / cambiar
  tipo** rompe.
- **Valores de enum o estado NUEVOS pueden CRASHEAR** a clientes viejos que parsean estricto →
  no basta con agregarlos. Protege con un **gate por versión de cliente** (mapea lo nuevo a algo
  que el cliente viejo entienda) y/o **enums tolerantes** (fallback a "desconocido").
- **Nunca asumas que el cliente viejo ignora lo desconocido**: verifícalo en su código.

### 3. Blast-radius antes de aprobar

Pregunta explícita: **¿qué se puede romper y a quién afecta?** Si el cambio toca un servicio o
cliente en uso, planea no-downtime / ventana / gate, y **no reinicies ni redepliegues a ciegas**
algo que sirve a clientes vivos.

## Quick reference

| Cambio | Reflejo |
|---|---|
| Carga / UPDATE de datos | backup + dry-run + idempotente + rollback |
| Campo nuevo en respuesta | aditivo + opcional (seguro) |
| Estado / enum nuevo | gate por versión / enum tolerante (puede crashear apps viejas) |
| Quitar / renombrar / cambiar tipo | rompe — requiere versión o migración de clientes |
| Reiniciar / redeploy de un servicio en uso | confirmar blast-radius primero |

## Errores comunes

- **Hacer el cambio defensivo solo cuando el usuario dice "producción".** Debe ser el default.
- **Agregar estados/enums nuevos sin pensar en clientes viejos** que crashean al parsear.
- **UPDATE directo** sin backup ni dry-run.
- **Asumir que el cliente viejo ignora lo desconocido** sin verificarlo.
- **Reiniciar un servicio en uso** sin medir el blast-radius.

Captura las decisiones de compatibilidad y los procedimientos de rollback como memoria →
`superdev:contexto-vivo`.
