---
name: runbook-cambio-prod
description: >
  Armar el runbook de un cambio en producción sobre infraestructura cloud (Azure o AWS):
  ventana medida, precondiciones, pasos, verificación y rollback. Usar antes de ejecutar
  un cambio que afecte producción — resize o migración de base de datos, upgrade de
  cluster, cambio de instancia, failover, cambio de DNS o de listener. Disparadores:
  "arma el runbook", "plan de cambio en producción", "ventana de mantenimiento",
  "cómo hacemos el switchover", "necesito el plan de rollback", "vamos a bajar/subir la
  instancia", "esto toca producción".
---

# Runbook de cambio en producción

El propósito no es documentar: es **que el cambio sea reversible y que alguien más pueda
ejecutarlo o abortarlo** sin el contexto de la sesión.

## Antes de escribir nada

1. **Medir, no estimar.** La ventana sale de métricas reales — CloudWatch o Azure Monitor
   sobre los últimos días — no de intuición. Buscar el valle de conexiones y CPU, y las
   ventanas que hay que **evitar** (backups automáticos, jobs programados, picos conocidos).
2. **Buscar el precedente.** Si ya se hizo un cambio parecido sobre el mismo recurso, el
   corte medido entonces es el mejor pronóstico. Anotarlo como referencia.
3. **Verificar el estado actual con el CLI**, no asumirlo: versión, tamaño, Multi-AZ,
   almacenamiento, quién consume el recurso.

## Estructura del runbook

```markdown
# Runbook — <cambio> sobre <recurso> (<ambiente>)

**Fecha planificada:** …  **Ventana:** HH:MM–HH:MM <zona horaria>  **Ejecuta:** …

## Qué cambia
Estado actual → estado objetivo, en una tabla. Incluir el ahorro o el beneficio esperado.

## Por qué ahora
Criterio de decisión y los números que lo respaldan.

## Ventana
La ventana medida, con qué métrica la justifica y **qué horas hay que evitar y por qué**.
Corte esperado, con el precedente si existe.

## Quién se ve afectado
Aplicaciones, procesos y equipos que consumen el recurso. Nombrarlos concretamente
(instancias, procesos, vhosts), no "los servicios".

## Precondiciones
Lista verificable. Snapshot o respaldo reciente y **comprobado**, accesos vigentes,
ventana comunicada, quién está disponible durante el cambio.

## Pasos
Numerados, con el comando literal de cada uno y qué se espera ver. Marcar cuál es el
**punto de no retorno**.

## Verificación
Qué se comprueba después y con qué comando. Criterio explícito de éxito.

## Rollback
Cómo se revierte, cuánto tarda, y **a partir de qué punto ya no se puede** — decirlo
aunque la respuesta sea "no se puede, solo se restaura del snapshot".

## Si algo sale mal
A quién se avisa y por qué vía.
```

## Reglas

- **Ningún paso destructivo se ejecuta sin confirmación explícita del usuario en el chat.**
  El runbook se escribe primero; ejecutarlo es una decisión aparte.
- Si el rollback no existe o es caro, decirlo en su propia línea. Un runbook sin rollback
  honesto es peor que ninguno.
- Los comandos van completos y con los valores reales, listos para pegar. Sin
  `<placeholders>` salvo los que el usuario debe rellenar a conciencia.
- Verificar el perfil, la suscripción y el contexto activos antes del primer comando, y
  ponerlo como paso 0. La mayoría de los accidentes son ejecutar lo correcto en la cuenta
  equivocada.
- Guardar el runbook junto a la documentación del proyecto, no en un scratchpad.

## Referencia

`~/Documents/local/st/kipu-infra/docs/RUNBOOK-SWITCHOVER.md` y los demás `RUNBOOK-*.md`
de esa carpeta son ejemplos ya escritos con este criterio.
